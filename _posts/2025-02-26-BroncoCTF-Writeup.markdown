---
layout: post
title:  "Bronco CTF World's Hardester Flag Writeup "
date:   2025-02-26 02:10:10 +0100
categories: CTF 
usemathjax: true
---

# World's Hardester Flag - misc

```
                             =*%%@@@@@%%+.                        
                         =@@@@@@@@@@@@@@@@@@*.                    
                      =@@@@@@@@@@@@@@@@@@@@@@@@%.                 
                   =%@@@@@@@@@@@@@@@@@@@@@@@@@@@@@+               
                #@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@:            
            .*@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@+          
          -@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@#-:-=@@@@@@@@@+        
        +@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@+::+#@@@@@@@@@@@@@@@:      
      +@@@@@@@@@*-*%@@@@@@@@@@@@@@@%=:#@@@@@@@@@@@@@@@@@@@@@=     
     %@@@@@@@@@@@@@%*:#@@@@@@@@@@#*+-:-%@@@@@@@@@@@@@@@@@@@@@+    
    -@*=+#%@@@@@@@@%*@@@@@@@@@@@@@@@@@@@+:=#@@@@@@@@@@@@@@@@@@:   
     @+-+**#@@@@@@+#@@@@@@@@@@@@@@@@@@@@@@@%- +@@@@@@@@@@@@@@@=   
     :@%*:-==-%@@*@@@@@@@@@@**@@@@@@@@@@@@@@@@#-%@@@@@@@@@@@@@+   
       .#*:--:.=@@@@@@@@@@@   =@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@+   
          -=---:.#@@@@@@@@@=  =@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@:   
           .+:--*%#=--%@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@=    
             *#*=----: .#@@@@@@@@@@@@@@@@@@@@%%=  ::------:.      
             =*=-:----:  =@@@@@@@@@@@@@@@%%@@@@@@#                
             -=::--=-. ::::*@@@@@@@@@%#%@%@@@@@@@@@%.             
             .+. .::        -@@@@%#%@@@@@@@@@@@@@@@@@%            
               +:.. . -.     .+%@@@@@@@@@@@@@@@@@@@@@@@*          
                :.   *:+-.   .*@@@@@@@@@@@@@@@@@@@@@@@@@%:        
               -@@:  -*+*:-==:+@@@@@@@@@@@@@@@@@@@@@@@@@@@#       
               :@@@#:    .-=. .-@@@@@@@@@@@@@@@@@@@@@@@@@@@%.     
                .%@@@*..   .   ---%@@@@@@@@@@@@@@@@@+@@@@@@@+     
                   :=+=:.   .-=--==+#%@@@@@%%%%%@@@@@@@@@@@@*     
                        :. .:-==+#%@@@@@@@@@@@@@@@@@@%#*+=-:.     
                         :+:.:+%*:.                               
                           =+*-                                   
                        Too Dumb To Have Imposter Syndrome        
                                Mard aka. Mardcelo                
```
# Table of contents
1. [Introduction](#introduction)
2. [Game Overview](#overview)
3. [Speedrun](#speedrun)
    1. [Exploit Time](#exploit)
    2. [Technical Overview of Dex Explorer](#Dex)
4. [Roblox Hyperion](#hyperion) 
5. [After the BroncoCTF 2025](#aftermath)
6. [Prevention](#prevention) 
7. [Obfuscation](#obfuscation)  

## Introduction <a name="introduction"></a>

I came back from Ireland, I had 1 hour and 30 minutes left. I had to speed run to finish a few challenges that I thought it would be fun.

I managed to rush back to my Desktop and then rushed for 10 minutes and solved the flag.
I tried rather than the intended approach of “lol learn Lua and figure out commands”, I did “this game is really exploitable beyond the scope of this challenge”. I started to write this write-up.

## Game Overview <a name="overview"></a>

The game is based on a ported version of “The World’s Hardest Game 4” in Roblox.
The game is about you controlling a red square, who must collect all yellow circles and avoid the blue circles while making it to the Green exit. The game might be simple, but it is very difficult.

This is version 4 of the game since the game introduces water obstacles, which you can drown in. and ice obstacles, which makes movement difficult, etc.

There is also a Settings menu, and Command bar, where what challenge the developer was expecting was trying to use the command to somehow reveal the flag.

## Speedrun <a name="speedrun"></a>

Since if we see in the console, we can see that the client side isn’t obfuscated.

![Game Console](http://ctfnote.frogcouncil.team/pad/uploads/447063e6-7961-46b5-b4b0-abc66b217f08.png)

I thought that if I attached a debugger to the client side I would be able to retrieve the flag and the source code.

## Exploit Time <a name="exploit"></a>

Using the DLL injection, I used `loadstring()` to load an existing tool called Dex Explorer, Which is a debugger suite for Roblox cheat development.

Using this I was able to have user side code view

![](http://ctfnote.frogcouncil.team/pad/uploads/4d64d33a-6044-4bb1-ba67-14e67d19c8a6.png)

![](http://ctfnote.frogcouncil.team/pad/uploads/00dca873-12cf-4a22-b1a4-85b9003b9d32.jpg)

Boom, `bronco{n0th1ng_1s_2oo_h4rd_4_m3!}` Got the flag in under 2 minutes

So, how does this script work? 

Great question, since people are lazy, I did some analysis of it

## Technical Overview of Dex Explorer [TODO]  <a name="Dex"></a>

Dex Explorer loader works like this:

1. Define Key Variables
2. Define Default Settings
3. Define Core Service Hooking
4. Get Local Player
5. Define Helper Functions
6. Initialize Main Object
7. Define Module System
8. Load & Fetch API Data from repo above 
9. Load and Fetch RMD (Reflection Metadata)
10. Create & Show UI
11. Initialize Explorer & Properties
12. Start the Main Execution

We are going to look more deeply into property viewer since it's very rooted with what we have used for CTF challenge. [TODO]

## Roblox Hyperion <a name="hyperion"></a>

This section was heavily relied on a forum post on Unkowncheats. I had no time to verify about this section which I will try to attach HyperDbg later and make new blog post about it. 

Instrumentation Callback (IC): This is Hyperion's central control point. It intercepts exceptions (like page faults when trying to execute encrypted code), system calls, and potentially other events. The IC is used for code decryption, integrity checks, and generally managing protected memory regions. It's essentially a custom exception handler prior to the standard Windows Exception Handling (WEH).

Dynamic Code Encryption/Decryption: Hyperion encrypts parts of the game's .text section (where executable code resides) to prevent static analysis and tampering. When encrypted code needs to be executed, the IC intercepts the resulting page fault, decrypts the page on-the-fly, marks it executable, executes the code, and then re-encrypts it. This makes reverse engineering significantly harder.

Dual Memory Views: Hyperion cleverly avoids race conditions and simplifies memory management by using two views of the same physical memory:

Read-Only, Encrypted View: This is the "default" view, presented to most of the process. Attempts to write to this view would trigger a protection fault.

Writable, Decrypted View: This view is likely used internally by the IC to perform decryption and modifications. This separation prevents one thread from modifying the memory while another thread is reading/executing it, a classic race condition vulnerability.

TEB Manipulation: Hyperion uses flags in the Thread Environment Block (TEB) to prevent recursion within its own IC. This is a defensive measure to avoid infinite loops or stack overflows if the IC itself causes an exception. The TEB is a per-thread data structure that contains information about the thread, including exception handling details.

Exception Handling Hijacking Prevention (Limited): The description notes that they validate event handlers and return addresses to prevent simple APC (Asynchronous Procedure Call) redirection, a common technique for injecting code. However, this doesn't mean exception handling is completely immune, as we'll see.

### Bypass Methods 

>Hyperion is a robust anti-tamper but we must remember that they are running at CPL3 aka being usermode, so they have all the usermode constraints. This is why they are unable to detected rootkits and kernel manipulations or even hardware based hypervisors (which is why most people debug and trace with a hypervisor), another example, i said that they check for named objects and registry entries. Anyone can hide or remove these objects from hyperion's view by just hooking and intercepting calls to NtQueryDirectoryObject. Same for their syscalls, you can just unmap hyperion module, hook needed sycalls and remap their module.

Instrumentation Callback Abuse: This is the most obvious and direct attack vector. Since everything goes through the IC, controlling it is the holy grail. The description suggests several sub-approaches:

Direct IC Manipulation: If you can find and modify the IC's code or data directly, you can completely disable or subvert its checks. This is likely very difficult due to the encryption.

TEB Flag Manipulation: The description explicitly states that manipulating TEB flags can disable the IC or skip checks. This is a prime target. Finding where and how these flags are set and cleared is key.

Exception Context Manipulation: When the IC handles an exception, it modifies the thread's context (specifically, the RIP register, which points to the next instruction to execute) to resume execution after decryption. The description states that you could cause an exception and then, within the IC's handling routine, modify the RIP in the exception context to point to your own code. This is a sophisticated but powerful attack, essentially hijacking the exception handling flow.

Syscall Hooking (after Hyperion Unmapping): The description mentions unmapping Hyperion's memory, hooking system calls (functions provided by the operating system kernel), and then remapping Hyperion. This would allow you to intercept and modify the results of system calls that Hyperion uses for its checks (e.g., memory queries, process enumeration, etc.). The challenge here is the "unmapping/remapping" process; you'd need to understand Hyperion's memory layout and protection mechanisms very well.

Page Decryption Abuse: While they decrypt on-the-fly, the decrypted page must exist in memory, however briefly. A sufficiently fast and well-timed memory scan might be able to capture the decrypted code. This is a race condition, and likely unreliable, but worth exploring. More realistically, you could focus on understanding the decryption routines themselves. If you can reverse engineer the decryption algorithm or key derivation, you could decrypt pages yourself.


## After BroncoCTF <a name='aftermath'></a>

After the BroncoCTF ended, I was talking with my team members about the ability to copy the game. Where I found out, it is very possible.

I used `SynSaveInstance()` with  function to basically copy the game. 

```lua
local Params = {
	RepoURL = "https://raw.githubusercontent.com/luau/SynSaveInstance/main/",
	SSI = "saveinstance",
}

local synsaveinstance = loadstring(game:HttpGet(Params.RepoURL .. Params.SSI .. ".luau", true), Params.SSI)()

local CustomOptions = { SafeMode = true, timeout = 15, SaveBytecode = true }

synsaveinstance(CustomOptions)
```

![](http://ctfnote.frogcouncil.team/pad/uploads/cf70d9b9-ea1e-4290-a9fa-11d82cfca502.png)

This also allowed me to get the flag `bronco{n0th1ng_1s_2oo_h4rd_4_m3!}`
This was quite an interesting method

Yoshi made a mistake on putting the flag on `StarterGui`, Which to stop this you will have to put place your GUI’s in `Replicated Storage` unless they are used almost the entire time the player is in the game, main GUI’s that are constantly shown can just be in `Starter GUI`, no point in not having them there.

Or just like I said, obfuscate the shit out of the game, so nobody knows what's going on, but this challenge was meant to be open-end. 

## Prevention <a name='Prevention'></a>

Only forum/blog post for this issue or anything related to exploits in Roblox is linked with this [forum post](https://devforum.roblox.com/t/exploiting-explained/170977). I think it's very general, and the game developer should also make anti-cheats. 

Is there any way to stop people using Dex? 

Answer is no, but there are ways such as obfuscation to make them miserable about their life choices, and use `PackageLink` making it impossible for them to publish their own game with stolen assets, but that's absolutely a pain in the ass. 

To stop codes getting stolen, keep everything in server side, rather trusting the client side.

Still, if the object loads into client side, there is no way to actually block exploiters to steal assets. But you can at least stop them stealing your code that is "made with love".

> After Roblox added `filtering enabled` this was also been mitigated somewhat. 

Also, it's evident that popular games like Phantom Forces uses obfuscation to stop people who managed to save the place using `saveinstance()` It's appearing in the game's console 

![](http://ctfnote.frogcouncil.team/pad/uploads/9f7e4c32-ecbb-4531-936c-cf9ad1227d5a.png)

### Obfuscation <a name='obfuscation'></a>

For obsfucating a game in Roblox, there is a lot of different tools. Let me list a few: 

1. [Prometheus](https://github.com/prometheus-lua/Prometheus) 
2. [Synapse Secure](https://github.com/SnowyShiro/Synapse-SecureLua-Deobfuscator/tree/main)
3. [Ironbrew 2](https://github.com/Trollicus/ironbrew-2)











