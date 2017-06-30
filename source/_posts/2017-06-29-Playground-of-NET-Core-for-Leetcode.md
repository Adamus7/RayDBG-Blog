---
title: Playground of .NET Core for LeetCode
date: 2017-06-29 16:10:35
tags:
- VSCode
- .NET Core
- LeetCode
---
I always like to practice coding skills and algorithm on [LeetCode](leetcode.com). Since I'm not a genius who can write code in white paper and debug the code in the air, a local test before submitting is really important for me. So, everytime, I need to read the descriptions of the problem on LeetCode, create a local project with a main function, implement a class with functions for the problem, create test inputs and check the output manually. It is terrible because I have to repeat those steps every time. In this article, I would like to introduce how I solved the above problem with .NET Core, XUnit and PowerShell Scripts.
<!-- more -->

# Get the problem from LeetCode
Haochen shared a project on Github which can retrieve description of a problem from LeetCode and help to generate corresponding Class with comments and even test Class. 

