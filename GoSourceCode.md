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



