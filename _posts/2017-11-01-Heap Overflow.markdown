---
layout: post
comments: true
title:  "Buffer Overflow Attack"
excerpt: "A Comprehensive Investigation of the Equifax Security Incident."
date:   2017-10-07 11:00:00
mathjax: false
---

- The Stack Overflow

The buffer overflow attack has a long history. A stack overflow attack is a common type of this kind of attack. The notorious Morris worm is a classic example of a stack overflow attack. The way it works is simple. For instance, when a character array of length 256 is defined, a 256 byte space will be allocated on the program call stack. If more than 256 bytes are written into the name buffer, using gets, then other values on the stack will be destroyed. Among them, important values like the frame pointer and the return address are overwritten.

Apparently, this will cause damage to the program data, but the problem of buffer overflow is more serious: they often result in code execution [1]. This happens when these overflowed data deliberately overwrite important values staying on the stack—return addresses. The return address determines which instruction the CPU will execute when it returns from the current function. If the overflowed buffer overwrite the return address, it could point to any address. If attackers control the return address, they can decide what code to execute next.

In 1988, the Morris worm virus exploited exactly the gets function used in finger [1]. finger is an old UNIX program used to query user info on a remote system. When queried about a specific username, finger will tell you some information about that user. When fetching the username, it would read, using gets, a 512-byte buffer onto the stack from the network. In a normal operation, the data is just a possible username and 512-byte buffer is more than enough to store it. No one is likely to have that long a username. But, using the gets function, the system did nothing to enforce any constraint. Sending more than 512 bytes over the network would overflow its buffer. And this is exactly what Robert Morris did. He sent a 537-byte buffer which rewrote the return address to point a short piece of code in that buffer on the stack. The code launched a shell, a common choice of an attack. The finger program ran as root, so when it was manipulated to launch a shell, that shell also ran as root. Then the root shell was now controlled remotely by the attacker. Now, the dangerous gets function is rarely used. And safer functions typically require the size of a buffer as a parameter.

- The Heap Overflow

The heap overflow is another common attack type. Actually, the winner of 2017 “Pwn2Own” take advantage of a heap overflow bug of Microsoft Edge to remotely take control [2]. When a website is opened, the JavaScript code from the site will be running on local machines. This introduces potential secure threats. As a result, Web Browsers like Chrome, Safari and Edge have become targets of many heap overflow attacks.

While there is no return address stored on the heap, other important values like function pointers can be found there. For instance, in Object Oriented Languages, class instances are normally located on the heap. As a result, the pointers to class methods, function pointers, are also on the heap. Through the overflowed buffer, attackers could change pointers to direct to the shellcode they want to execute. 

Heap spraying is a classic attack technique. It works by spraying shellcodes in the heap. Pieces of shellcode always come with a NOP slides before it. Once a function pointer directing to a NOP slide is called, the shellcode following the slide will be executed, either to download a malicious software or to open a remote connection.

“Heap sprays for web browsers are commonly implemented in JavaScript and spray the heap by allocating very long strings” [3]. Depending on how the browser encoding characters, an attacker can put in corresponding encoded NOP slides and shellcode. The heap spraying code keeps copying strings until the heap is filled with enough shellcode to ensure they will be invoked.

- Bibliography

[1] Bright, Peter. “How security flaws work: The buffer overflow” Ars Technica, 25 Aug. 2015

[2] Brook, Chris. “VM Escape Earns Hackers $105K at Pwn2Own” Threat Post, 17 Mar. 2017

[3] “Heap spraying” Wikipedia.org, Retrieved Sept 12, 2017
