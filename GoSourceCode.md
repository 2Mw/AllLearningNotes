# Go Source Code Reading

[TOC]

## net

Doc: [Doc: net package](https://pkg.go.dev/net)

Time: (2022-09-03)

Overview: Package net provides a portable interface for network I/O, including TCP/IP, UDP, domain name resolution and Unix domain sockets.

Demo:

* Dial to a server:

  ```go
  conn, err := net.Dial("tcp", "golang.org:80")
  if err != nil {
  	// handle error
  }
  fmt.Fprintf(conn, "GET / HTTP/1.0\r\n\r\n")
  status, err := bufio.NewReader(conn).ReadString('\n')
  // ...
  ```

* Create a server:

  ```go
  ln, err := net.Listen("tcp", ":8080")
  if err != nil {
  	// handle error
  }
  for {
  	conn, err := ln.Accept()
  	if err != nil {
  		// handle error
  	}
  	go handleConnection(conn)
  }
  ```

How `net` resolve name: It looks up operating system.

* On Unix systems,  there are 2 ways to resolve servers

  1. Pure go resolver: Send domain information to DNS servers in `/etc/resolv.conf`. (Default)
  2. Use cgo-based resolver in C library such `getaddrinfo` and `getnameinfo`. (Blocked)

  We set the resolver by setting the netdns value of GODEBUG environment variable:

  ```sh
  export GODEBUG=netdns=go    # force pure Go resolver
  export GODEBUG=netdns=cgo   # force native resolver (cgo, win32)
  ```

* On windows, in GO 1.18.x and earlier, the resolver always used C lib functions, such as `GetAddrInfo` and `DnsQuery`.

### Dial

What is the process of dialing remote address?

```go
func Dial(network, address string) (Conn, error) {
	var d Dialer
	return d.Dial(network, address)
}

func DialTimeout(network, address string, timeout time.Duration) (Conn, error) {
	d := Dialer{Timeout: timeout}
	return d.Dial(network, address)
}
```

The `network` is the type of networks: such as tcp, tcp4, tcp6, udp, ip, unix and so on.

The `address` string contains IP and Port: such as 192.168.1.1:8080 or [2001:db8::1]:53

There is Dialer used to dial remote address, let's see details.

```go
func (d *Dialer) Dial(network, address string) (Conn, error) {
	return d.DialContext(context.Background(), network, address)
}
```

The method `Dial` of `Dialer` call function `DialContext`, which needs 3 parameters. The first one is father Context, which is used to cancel connection if timeout (Default no timeout unless using `DialTimeout`).

Firstly, we read the structure of `Dialer`:

```go
type Dialer struct {
    // The amount of time a dial will wait for a connect to complete.
    Timeout time.Duration
    // Absolute time point. 0 means no timeout or dependent on OS.
    Deadline time.Time
    LocalAddr Addr
    DualStack bool
    FallbackDelay time.Duration
    // Specify the interval for an active network connction. Negative means no heartbeats.
    KeepAlive time.Duration
    Resolver *Resolver
    Cancel <-chan struct{}
    // It is called after creating network connection before actually dialing.
    Control func(network, address string, c syscall.RawConn) error
}
```

The core part of dialing is function `DialContext`:

```go
func (d *Dialer) DialContext(ctx context.Context, network, address string) (Conn, error) {
    if ctx == nil {
        panic("nil context")
    }
    // .....
    // Context Setting, Omit

    // if address contains domain, then return the list of ips.
    addrs, err := d.resolver().resolveAddrList(resolveCtx, "dial", network, address, d.LocalAddr)
    if err != nil {
        return nil, &OpError{Op: "dial", Net: network, Source: nil, Addr: nil, Err: err}
    }

    sd := &sysDialer{
        Dialer:  *d,
        network: network,
        address: address,
    }
    
    // primaries contains the ipv4 type address
    // fallbacks contains the ipv6 type address
    var primaries, fallbacks addrList
    if d.dualStack() && network == "tcp" {
        primaries, fallbacks = addrs.partition(isIPv4)
    } else {
        primaries = addrs
    }

    var c Conn
    if len(fallbacks) > 0 {
        c, err = sd.dialParallel(ctx, primaries, fallbacks)
    } else {
        c, err = sd.dialSerial(ctx, primaries)
    }
    if err != nil {
        return nil, err
    }
    
    // For TCP Connection, it needs keep alive
    if tc, ok := c.(*TCPConn); ok && d.KeepAlive >= 0 {
        setKeepAlive(tc.fd, true)
        ka := d.KeepAlive
        if d.KeepAlive == 0 {
            ka = defaultTCPKeepAlive
        }
        setKeepAlivePeriod(tc.fd, ka)
        testHookSetKeepAlive(ka)
    }
    return c, nil
}
```

There are mainly three point of dialing:

1. Set Context config if set timeout.
2. Lookup the IP list, if the type of address is domain then return the IPs list of the domain.
3. IP list are partitioned to two parts according if the type of IP is IPv4.
4. If there are both IPv4 and IPv6 address, dial parallelly for them. Other with dial serially.
5. If the type of connection if TCP, the file descriptor Keep Alive (Long Connection).
