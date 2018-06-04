---
title: Debugging ASP.NET Core Application with Framework Source Code
date: 2018-06-04 16:37:07
tags:
- ASP.NET Core
- Debugging
- Dotnet Core
---
Since .Net Core and ASP.NET Core is open sourced, developers are able to step into the source code of framework for different purposes. 
As you may know, the source code of the dotnet runtime (CoreCLR), foundational libraries(CoreFX) and ASP.NET Core are hosted GitHub. With all the source code available, it that possible to have a different debugging experience with framework code?
<!-- more -->
# Pre Story
You may familiar with PDB(Program Database) file which contains debugging information about a program(EXE) or a module(DLL). A PDB file is created from source files during compilation and it stores a list of symbols of a module. With the help of the PDB file, the debugger is able to locate symbols or execution state of the source code.
.NET Core introduces a new symbol file (PDB) format - portable PDBs. Unlike traditional PDBs which are Windows-only, portable PDBs can be created and read on all platforms. [Taggart Software](https://github.com/ctaggart/SourceLink) introduced a new feature, SourceLink, into dotnet world. Later, it becomes a .NET Foundation project. SourceLink is a developer productivity feature that allows unique information about an assembly's original source code to be embedded in its PDB during compilation.
By enabling SourceLink v2 in dotnet core application, developers can embed online source code path (remote Git/TFS repository) into portable PDB file. Visual Studio 15.3+ supports reading SourceLink information from PDB file while debugging and helps to download the source code. Then developers can debug into the source code of the referenced DLL.

