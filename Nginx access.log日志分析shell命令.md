Liunx && Nginx access.log日志分析shell命令
===
## Nginx access.log日志分析shell命令
##### Nginx日志格式:
```
 $remote_addr – $remote_user [$time_local]  $request $status $apache_bytes_sent $http_referer $http_user_agent
127.0.0.1 - - [24/Mar/2011:12:45:07 +0800] "GET /fcgi_bin/xxx.fcgi?id=xxx HTTP/1.0" 200 160 "-" "Mozilla/4.0"
```
* 查询的 文件前5信息
``` Liunx
> tail -f access.log | head -n 5
> cat access.log| head -n 5
```
输出结果


![WX20180513-211051]($res/WX20180513-211051.png)

*  access.log 字段说明：

	1.客户端（用户）IP地址。如：上例中的 201.158.69.116
	
	2.访问时间。如：上例中的 [03/Jan/2013:21:17:20 -0600]
	
	3.访问端口。如：上例中的 127.0.0.1:9000
	
	4.响应时间。如：上例中的 0.007
	
	5.请求时间。如：上例中的 0.007
	
	6.用户地理位置代码（国家代码）。如：上例中的 MX（墨西哥）
	
	7.请求的url地址（目标url地址）的host。如：上例中的 pythontab.com
	
	8.请求方式（GET或者POST等）。如：上例中的 GET
	
	9.请求url地址（去除host部分）。如：上例中的 /html/test.html
	
	10.请求状态（状态码，200表示成功，404表示页面不存在，301表示永久重定向等，具体状态码可以在网上找相关文章，不再赘述）。如：上例中的 "200"
	
	11.请求页面大小，默认为B（byte）。如：上例中的 2426
	
	12.来源页面，即从哪个页面转到本页，专业名称叫做“referer”。如：上例中的 "http://a.com"
	
	13.用户浏览器语言。如：上例中的 "es-ES,es;q=0.8"
	
	14.用户浏览器其他信息，浏览器版本、浏览器类型等。如：上例中的  "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.97 Safari/537.11"

*  查找全部访问最多的IP并获取前十个
   根据上次，我们要用到 '去重'、'统计'、'排序'、'条件'、'limit （条数）'.
   去重统计 - sort | nuiq -c 
   排序        - sort -rn ( rn 倒序 n 正序)
   条件 awk '{print $1}'
   limit(条件) head -n 10 （正序10条 ） （）
```Liunx
 cat access.log |awk '{print $1}' |sort| uniq -c |sort -rn | head -n 10
```
输出的结果
``` liunx 
3508 58.132.171.126
1424 36.102.208.80
1356 91.230.47.3
1293 67.216.199.63
1273 180.97.106.37
1090 125.64.94.206
 897 121.42.0.77
 852 121.42.0.75
 820 36.110.71.26
 793 124.205.149.181
```
上面没有加上条件
* 通过日志查看**当天访问页面排前10的url:**
```liunx
>  cat access.log | grep "13/May/2018" | awk '{print $7}'|sort|uniq -c | sort -rn | head -n 10

- 返回结果
  72 /
   8 /phpmyadmin/index.php
   7 /xampp/phpmyadmin/index.php
   7 /www/phpMyAdmin/index.php
   7 /wls-wsat/CoordinatorPortType
   7 /web/phpMyAdmin/index.php
   7 /typo3/phpmyadmin/index.php
   7 /tools/phpMyAdmin/index.php
   7 /robots.txt
   7 /pmd/index.php
```
* 通过日志查看当天**访问次数最多的10个IP**
```liunx 
awk '{print $1}' access.log |grep "24/Mar/2017"|sort |uniq -c|sort -nr|head -n 10
```
*  通过日志查看当天访问次数最多的时间段
``` Liunx
> awk '{print $4}' access.log | grep "24/Mar/2017" |cut -c 14-18|sort|uniq -c|sort -nr|head -n 10
返回结果
  51 01:43
  31 22:58
  29 00:26
  23 22:59
  20 01:34
  19 02:13
  18 23:00
  17 21:20
  13 22:57
   9 02:14
```
* 通过日志查看状态等于400 的 IP 和 地址
```liunx 
> awk '{print $1,$7, $10= 400}' access.log |sort|uniq -c|sort -nr|head -n 10

输出结果
 1356 91.230.47.3 / 400
 473 119.57.159.17 / 400
 392 119.57.159.183 / 400
 354 139.162.88.63 http://clientapi.ipip.net/echo.php?info=1234567890 400
 351 123.151.42.61 http://www.baidu.com/ 400
 330 202.108.211.56 / 400
 259 106.120.173.108 / 400
 223 119.57.159.17 /robots.txt 400
 200 120.76.54.183 / 400
 184 106.120.173.108 /robots.txt 400

```

