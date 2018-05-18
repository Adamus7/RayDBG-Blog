---
title: Debugging .Net Core on Linux with LLDB
date: 2018-05-18 15:18:33
tags:
- .Net Core
- LLDB
- Debugging
---
.Net Core is designed to be cross-platform, modular and optimized for cloud. What if there is a exception or a memory issue of the .Net Core application on Linux platform?
On Windows, we have a set of tools to do different analysis. For example, I can take a process dump by ProcDump and feed it to WinDBG for exception or memory analysis.
Actually, we can do similar things on Linux for .Net Core application now.
<!-- more -->

# The Tools
Simply, here is a cheat sheet of difference tools on Windows and Linux:

| Windows | Linux |
| --- | --- |
| ProcDump | ProcDump, gcore |
| WinDBG | gdb, LLDB |

The LLDB debugger is conceptually similar to the native Windows debugging tools in that it is a low level and command live driven debugger. It is available for a number of different *NIX systems as well as MacOS. Part of the reason the .NET Core team chose the LLDB debugger was for its extensibility points that allowed them to create the SOS plugin which can be used to debug .NET core applications. The SOS LLDB plugin contains the same commands that we have grown accustomed to in the Windows world. Therefore, LLDB is the ideal debugger for .Net Core on Linux.

Microsoft shipped ProcDump to Linux which provides a convenient way for Linux developers to create core dumps of their application based on performance triggers. Eventually, the ProcDump will call gcore on Linux to generate the core dump.


