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

## Technical Overview of Dex Explorer: Peeking Under the Hood <a name="Dex"></a>

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

We are going to look more deeply into property viewer since it's very rooted with what we have used for CTF challenge. 

So, how does a powerful tool like Dex Explorer actually work its magic inside a Roblox game? The `loadstring()` function I used was just the tip of the iceberg. Let's delve into the common principles behind how these in-game explorers operate. It's important to note that this is a general overview, as specific implementations can vary.

Dex Explorer, at its core, is a suite of Lua scripts that execute *within the Roblox client's environment*. It's not an external program in the traditional sense but rather an advanced `LocalScript` with a very fancy GUI.

Here's a breakdown of its likely operational principles:

1.  **Entry Point & Execution Context:**
    *   **`loadstring()` is Key:** The journey begins when the client executes the Dex loader script via `loadstring(game:HttpGet("...", true))`. This function compiles and runs a string as Lua code.
    *   **Client-Side Privileges:** Once loaded, Dex operates with the same level of access as any other `LocalScript` in the game. This means it can directly interact with client-side instances, services, and the game's environment (e.g., the `game` global, `workspace`, `Players.LocalPlayer`, `StarterGui`).

2.  **Roblox API Interaction & Introspection:**
    *   **Leveraging Core Services:** Dex extensively uses Roblox's built-in services to gather information. For example:
        *   `game:GetService("Players")` to get player information.
        *   `game:GetService("Workspace")` to access the 3D game world.
        *   `game:GetService("StarterGui")` to see UI elements (where our flag was!).
    *   **Instance Hierarchy Traversal:** To build its familiar explorer tree view, Dex needs to navigate the `Instance` hierarchy. It achieves this by:
        *   Recursively calling `Instance:GetChildren()` on objects starting from `game`.
        *   Potentially listening to signals like `Instance.ChildAdded` and `Instance.ChildRemoved` to keep its view dynamically updated if new objects are created or destroyed while Dex is running.
    *   **Property Access:** This is fundamental. Dex reads the properties of any selected `Instance` using standard Lua table indexing. For example, to get an object's name, it would essentially do `selectedObject.Name`. To get a script's source, it would read `someScript.Source`.
    *   **Function Hooking (Advanced Usage):** While not strictly necessary for just viewing properties (as in our CTF case), more comprehensive versions of Dex or similar tools might employ function hooking. They can replace or wrap common Roblox functions (e.g., `Instance.new`, methods on services) by manipulating Lua environments or metatables. This allows them to log activities, intercept data, or even modify behavior, offering much deeper analytical capabilities.

3.  **Reflection Metadata (RMD) - The Secret Sauce for Detail:**
    *   Roblox's engine contains internal data known as "Reflection Metadata" (RMD). This metadata is a comprehensive database describing all engine classes (like `Part`, `TextLabel`, `Script`, etc.), their properties, methods, events, and signals. It includes information about data types and security contexts.
    *   Dex likely fetches, parses, or has a pre-compiled version of this RMD. This is what enables it to:
        *   Display a complete list of *all* properties an `Instance` has, often more than what's visible in Roblox Studio.
        *   Show the correct data type for each property (e.g., `string`, `number`, `bool`, `Vector3`, `Instance` reference).
        *   Potentially identify and display hidden, deprecated, or internal engine properties (though accessing truly protected ones would still be blocked by engine security).

4.  **User Interface (UI) Integration:**
    *   The entire Dex Explorer interface (the tree view, property pane, script editor, command bar, etc.) is built using Roblox's own UI elements: `ScreenGui`, `Frame`, `TextLabel`, `TextBox`, `ScrollingFrame`, and so on.
    *   The Dex scripts dynamically create, position, and manage these UI objects to present the gathered information to the user and to handle user interactions like clicks and text input.

5.  **Modular Design & Data Fetching:**
    *   Complex tools like Dex are often modular. The initial script loaded via `loadstring()` might be a lightweight "loader."
    *   This loader then fetches other, larger Lua modules containing the core logic for the explorer, property viewer, script editor, etc. These modules might be fetched from an external URL (as seen with the `RepoURL` in the `SynSaveInstance` example) or embedded as strings within the loader itself.
    *   The "Load & Fetch API Data" and "Load and Fetch RMD" steps you outlined in your initial understanding refer to Dex obtaining the necessary class and property definitions, either by downloading them or having them bundled.

### The Property Viewer: Finding Our Flag

For this CTF, the property viewer was the star. When you select an object in Dex's explorer tree:

1.  **Instance Selection:** The clicked `Instance` (e.g., an object inside `StarterGui`) becomes the target for the property viewer.
2.  **RMD Lookup:** Dex consults its Reflection Metadata for the class of the selected `Instance` (e.g., if you selected a `TextLabel`, it looks up "TextLabel").

In our CTF case, the flag `bronco{n0th1ng_1s_2oo_h4rd_4_m3!}` was likely stored as a simple string in a property like `.Name`, `.Text` (if it was a `TextLabel` or similar), or `.Value` (if it was a `StringValue` object). Since `StarterGui` objects are client-side and the game wasn't heavily obfuscated, Dex could easily traverse to this object and read that property. Hyperion's defenses either weren't triggered by this type of passive information reading in this specific CTF setup or were not fully active.


## Roblox Hyperion <a name="hyperion"></a>

This section was heavily relied on a forum post on Unkowncheats. I had no time to verify about this section which I will try to attach HyperDbg later and make new blog post about it. 

Instrumentation Callback (IC): This is Hyperion's central control point. It intercepts exceptions (like page faults when trying to execute encrypted code), system calls, and potentially other events. The IC is used for code decryption, integrity checks, and generally managing protected memory regions. It's essentially a custom exception handler prior to the standard Windows Exception Handling (WEH).

Dynamic Code Encryption/Decryption: Hyperion encrypts parts of the game's .text section (where executable code resides) to prevent static analysis and tampering. When encrypted code needs to be executed, the IC intercepts the resulting page fault, decrypts the page on-the-fly, marks it executable, executes the code, and then re-encrypts it. This makes reverse engineering significantly harder.

Dual Memory Views: Hyperion cleverly avoids race conditions and simplifies memory management by using two views of the same physical memory:

Read-Only, Encrypted View: This is the "default" view, presented to most of the process. Attempts to write to this view would trigger a protection fault.

Writable, Decrypted View: This view is likely used internally by the IC to perform decryption and modifications. This separation prevents one thread from modifying the memory while another thread is reading/executing it, a classic race condition vulnerability.

TEB Flag Manipulation: The Thread Environment Block (TEB) is a per-thread data structure in Windows that holds crucial information, including details about exception handling. Hyperion reportedly uses specific flags within the TEB, potentially to prevent recursive calls to its Instrumentation Callback or to enable/disable certain checks. If an attacker can identify and manipulate these flags (e.g., by writing to the TEB from an injected DLL or a debugger), they might be able to effectively bypass or disable parts of Hyperion's checking mechanisms. The challenge lies in pinpointing the exact flags and understanding their purpose.

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











