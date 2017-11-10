---
title: Token Privileges in PowerShell
date: 2017-11-08 17:06:51
tags:
---
One common task in my daily work is to create a PowerShell script to modify the ACL(Access Control List) of Folders or Registrys on Windows. When I run some commands to take the ownership of the Folders or Registrys, I sometimes got permission denied error, even I have ran it as Administrator. What happens?
<!-- more --> 
# Access Tokens and Privileges
According the article [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms721532(v=vs.85).aspx#_security_access_token_gly): An access token contains the security information for a logon session. The system creates an access token when a user logs on, and every process executed on behalf of the user has a copy of the token. The token identifies the user, the user's groups, and the user's **privileges**.
A **privileges** is the right of an account, such as a user or group account, to perform various system-related operations on the local computer, such as shutting down the system, loading device drivers, or changing the system time.

