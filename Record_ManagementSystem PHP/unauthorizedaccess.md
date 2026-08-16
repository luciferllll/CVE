1. login.php — Login succeeds but never sets a session (broken authentication)
```
$result = $conn->prepare("SELECT * FROM user WHERE username= :hjhjhjh AND password= :asas");
$result->execute();
$rows = $result->fetch(PDO::FETCH_NUM);
if($rows > 0) {
    header("location: main/index.php");   // ← only redirects; no $_SESSION['loggedin'] = true / $_SESSION['user'], etc.
}
```
On wrong credentials it only stores an error message in $_SESSION['ERRMSG_ARR']. No login state is ever persisted — whether the login succeeds or not has zero effect on whether the business pages can be reached.

2. /main/index.php — Business pages never validate the session (missing access control)

I grepped all 25 PHP files under main/ for session_ / SESSION / logged / auth — zero hits. No authentication checks at all.

main/index.php goes straight to business logic:
```
include('connect.php');
$result = $db->prepare("SELECT * FROM transaction ORDER BY id DESC");
$result->execute();
for($i=0; $row = $result->fetch(); $i++){ ... }
```
No session_start(), no login check. Anyone hitting the URL directly can read all transaction/record data (date, received by, description, office, status, forwarded to…). Worse, connect.php connects to MySQL as root with an empty password.

3. Chain impact (same root cause affects the whole main/)

- add.php / editform.php / delete.php / savedoc.php / saveoffice.php — also zero authentication → unauthorized Create / Update / Delete all work too (the full CRUD you asked about earlier).
- delete.php deletes by GET id param, combined with no auth → records can be deleted by an unauthenticated attacker.
- doctype.php / offices.php — same, readable without login.
Set up the relevant environment
<img width="2532" height="1836" alt="image" src="https://github.com/user-attachments/assets/7e7822b0-5989-4d11-8dde-d1bb34add9c9" />
Directly accessing http://10.166.209.241:8081/main/index.php allows unauthorized access (Critical) 
<img width="1848" height="1334" alt="image" src="https://github.com/user-attachments/assets/3e672f90-aa72-4fb8-a441-b5f2cda5f110" />
<img width="2230" height="1278" alt="image" src="https://github.com/user-attachments/assets/eb08890c-ac82-4b7c-a7ca-273777f3e8df" />
Can add, delete, modify, and query.
<img width="2240" height="1188" alt="image" src="https://github.com/user-attachments/assets/f9910bf1-ed5b-4e5f-9087-a68a4b02cbc2" />
