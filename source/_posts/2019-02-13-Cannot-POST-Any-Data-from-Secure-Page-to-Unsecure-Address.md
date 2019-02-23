---
layout: '[post]'
title: Cannot POST Any Data from Secure Page to Unsecure Address
date: 2019-02-13 17:20:50
tags:
---

<!DOCTYPE html>
<html>
<body>

<h2>POST data to a HTTP endpoint</h2>

<p>Bad case: submit to a HTTP://test.contoso.com/testurl</p>
<form action="http://test.contoso.com/testurl" method="POST">
  First name:<br>
  <input type="text" name="firstname" value="Mickey">
  <br>
  Last name:<br>
  <input type="text" name="lastname" value="Mouse">
  <br><br>
  <input type="submit" value="Submit">
</form> 
<br/>

<p>Good case: submit to a HTTPS://test.contoso.com/testurl</p>
<form action="https://test.contoso.com/testurl" method="POST">
  First name:<br>
  <input type="text" name="firstname" value="Mickey">
  <br>
  Last name:<br>
  <input type="text" name="lastname" value="Mouse">
  <br><br>
  <input type="submit" value="Submit">
</form>

</body>
</html>