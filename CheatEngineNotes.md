# Cheat Engine Notes
[TOC]

## 0x1. Cheater and C++

### 1. 获取进程信息

* 首先查找窗口，使用 spy++ 获取窗口标题和窗口类

  ![image-20231201214650469](CheatEngineNotes.assets/image-20231201214650469.png)

* 使用 `FindWindow()` 函数获取对应窗口句柄

  ```c++
  HWND w = FindWindowA(getGameMainClass(), getGameName());
  ```

* 使用函数 `GetWindowThreadProcessId()` 根据窗口句柄获取进程ID

  ```c++
  DWORD processId;
  GetWindowThreadProcessId(w, &processId);
  ```

* 根据进程 ID 获取**进程句柄** `OpenProcess()`

  函数文档：[OpenProcess function](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess)，需要三个参数：

  * `dwDesiredAccess`：需要访问进程的权限，一般申请所有权限 `PROCESS_ALL_ACCESS`
  * `bInheritHandle`：如果想要打开的进程继承当前进程则设置为 TRUE，一般为 FALSE
  * `dwProcessId`：本地进程ID

  需要开启所有权限：`PROCESS_ALL_ACCESS`

  ```c++
  pHandle = OpenProcess(PROCESS_ALL_ACCESS, false, proccessID);
  ```


* 获取进程中模块的基址

  在一些游戏中，可能是根据`XXX.DLL`为基址的

  > 需要使用到头文件：`TlHelp32.h`

  ```c++
  std::pair<QWORD, QWORD> ProcessUtil::GetModule(DWORD processId, char* moduleName)
  {
  	HANDLE mod = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, processId);
  	MODULEENTRY32 mEntry;
  	mEntry.dwSize = sizeof(mEntry);
  	do {
  		if (!strcmp(mEntry.szModule, moduleName))
  		{
  			return { (QWORD)mEntry.hModule, mEntry.modBaseSize };
  		}
  	} while (Module32Next(mod, &mEntry));
  	return { 0, 0 };
  }
  ```

* 从进程内存中读取数值：

  需要使用函数：`ReadProcessMemory()`

  ```c++
  template<typename T>
  static T Read(HANDLE pHandle, QWORD address) {
      T _read = 0;
      ReadProcessMemory(pHandle, (LPVOID)address, &_read, sizeof(T), NULL);
      return _read;
  }
  ```

* 向内存中写入数值：

  需要使用函数：`WriteProcessMemory()`

  ```c++
  template<typename T>
  static BOOL Write(HANDLE pHandle, QWORD address, T value) {
      return WriteProcessMemory(pHandle, (LPVOID)address, &value, sizeof(T), NULL);
  }
  ```

### 2. DLL注入

* 线程注入：

  强制创建一个目标进程的线程，将目标 DLL 加载到进程空间中。主要使用到三个函数：

  1. `LoadLibrary()`：动态加载 DLL 使用的函数
  2. `CreateRemoteThread()`：在另一个进程空间中创建线程，参考[文档](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread)参数为：
     * `hProcess`：对应进程句柄
     * `lpThreadAttributes`: 安全描述符，默认 NULL 即可
     * `dwStackSize`：线程的初始栈大小，如果为0使用默认大小。
     * `lpStartAddress`：指向类型为 `LPTHREAD_START_ROUTINE ` 的函数指针，代表函数的起始地址，并且该地址必须存在于目标进程空间中。
     * `lpParameter`：传给对应函数的指针。
     * `dwCreationFlags`：创建线程的标志位，默认 0 即可，表示立即执行。
     * `lpThreadId`：用于接收线程标识符 ID，可以为 NULL
  3. `VirtualAllocEx()`：用于为某个进程保留、分配内存空间，参考[文档](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex)其参数为：
     * `hProcess`：进程的句柄
     * `lpAddress`：如果为空，则表示分配内存区域；不为空则保留该内存地址
     * `dwSize`：分配的内存空间大小，单位为字节数。为空则分配一个页的大小，不为空则具体大小的。
     * `flAllocationType`：分配类型**提交**还是**保留**，选提交 `MEM_COMMIT`，会为虚拟空间映射具体的物理地址。
     * `flProtect`：可读可写使用 `PAGE_READWRITE`
     * 返回值是分配内存区域的基址。

  ```c++
  void injectDLL(int pid, char* dllPath) 
  {
      // 获取对应进程的句柄
      HANDLE h = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
  
      // 申请参数大小的内存
      LPVOID paramAddr = VirtualAllocEx(h, NULL, strlen(dllPath) + 1, MEM_COMMIT, PAGE_READWRITE);
      // 向内存中写入数据
      WriteProcessMemory(h, paramAddr, dllPath, strlen(dllPath) + 1, NULL);
  
      // 获取加载LoadLibraryA函数在 KERNEL32.DLL 中
      HMODULE hModule = LoadLibrary("KERNEL32.DLL");
      LPTHREAD_START_ROUTINE func = (LPTHREAD_START_ROUTINE)GetProcAddress(hModule, "LoadLibraryA");
  
      CreateRemoteThread(h, NULL, 0, func, paramAddr, 0, NULL);
  }
  ```

  

* 

## 0x2. CE Demo 第九关(共享代码块)

参考资料：
* [Auto Assemble文档](https://wiki.cheatengine.org/index.php?title=Cheat_Engine:Auto_Assembler)

应用场景：
* 比如在扣减血量的时候，我方和敌人扣减血量使用的是同一个函数，如果将扣减血量的指令 nop 掉之后，自己虽然不掉血了，但是敌人也不掉血了。因此在 patch 的时候需要进行判断扣减的对象。

在关键语句的修改脚本：

* 使用lua脚本的 `define` 函数来定义基址：`define(base,"Tutorial-i386.exe"+2566F0)`
* 在使用 `registersymbol` 函数将定义好的符号，可以让汇编代码进行识别，结束时候使用 `unregistersymbol` 函数解除符号。
* 可以使用 `label` 函数定义多个代码块地址。
* 所有代码从上往下执行，需要开发者定义好流程控制。


```lua
define(address,"Tutorial-i386.exe"+28E89)
define(bytes,89 43 04 D9 EE)
define(base,"Tutorial-i386.exe"+2566F0)

[ENABLE]
//code from here to '[DISABLE]' will be used to enable the cheat

assert(address,bytes)
alloc(newmem,$1000)

label(code)
label(return)
label(me)
label(enemy)
registersymbol(base)

newmem:
  //push eax
  mov eax, [base]
  cmp ebx, [eax+500]
  je enemy
  cmp ebx, [eax+504]
  je enemy
  jne me

enemy:
  mov eax, 0
  jmp code

me:
  mov eax, 459c4000

code:
  mov [ebx+04],eax
  fldz
  jmp return

address:
  jmp newmem
return:

[DISABLE]
//code from here till the end of the code will be used to disable the cheat
address:
  db bytes
  // mov [ebx+04],eax
  // fldz

dealloc(newmem)
UNREGISTERSYMBOL(base)

```