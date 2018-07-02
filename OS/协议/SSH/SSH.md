### SSH(安全外壳协议)

[百度百科](https://baike.baidu.com/item/ssh/10407?fr=aladdin)

简介：（摘自百度百科）
SSH 为 Secure Shell 的缩写，由 IETF 的网络小组（Network Working Group）所制定；SSH 为建立在应用层基础上的安全协议。SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。SSH最初是UNIX系统上的一个程序，后来又迅速扩展到其他操作平台。SSH在正确使用时可弥补网络中的漏洞。SSH客户端适用于多种平台。几乎所有UNIX平台—包括HP-UX、Linux、AIX、Solaris、Digital UNIX、Irix，以及其他平台，都可运行SSH。

功能：（摘自百度百科）
传统的网络服务程序，如：ftp、pop和telnet在本质上都是不安全的，因为它们在网络上用明文传送口令和数据，别有用心的人非常容易就可以截获这些口令和数据。而且，这些服务程序的安全验证方式也是有其弱点的， 就是很容易受到“中间人”（man-in-the-middle）这种方式的攻击。所谓“中间人”的攻击方式， 就是“中间人”冒充真正的服务器接收你传给服务器的数据，然后再冒充你把数据传给真正的服务器。服务器和你之间的数据传送被“中间人”一转手做了手脚之后，就会出现很严重的问题。通过使用SSH，你可以把所有传输的数据进行加密，这样"中间人"这种攻击方式就不可能实现了，而且也能够防止DNS欺骗和IP欺骗。使用SSH，还有一个额外的好处就是传输的数据是经过压缩的，所以可以加快传输的速度。SSH有很多功能，它既可以代替Telnet，又可以为FTP、PoP、甚至为PPP提供一个安全的"通道" [1]  。

SSH协议安全性较高，故很多系统采用此协议做认证。认证方式一般有两种，一种是口令认证，一种是秘钥认证。

关于秘钥的生成（需要先安装ssh）：

```
ssh-keygen -t rsa
```
```
使用帮助：
$ ssh-keygen --help
ssh-keygen: invalid option -- -
usage: ssh-keygen [-q] [-b bits] [-t dsa | ecdsa | ed25519 | rsa]
                  [-N new_passphrase] [-C comment] [-f output_keyfile]
       ssh-keygen -p [-P old_passphrase] [-N new_passphrase] [-f keyfile]
       ssh-keygen -i [-m key_format] [-f input_keyfile]
       ssh-keygen -e [-m key_format] [-f input_keyfile]
       ssh-keygen -y [-f input_keyfile]
       ssh-keygen -c [-P passphrase] [-C comment] [-f keyfile]
       ssh-keygen -l [-v] [-E fingerprint_hash] [-f input_keyfile]
       ssh-keygen -B [-f input_keyfile]
       ssh-keygen -D pkcs11
       ssh-keygen -F hostname [-f known_hosts_file] [-l]
       ssh-keygen -H [-f known_hosts_file]
       ssh-keygen -R hostname [-f known_hosts_file]
       ssh-keygen -r hostname [-f input_keyfile] [-g]
       ssh-keygen -G output_file [-v] [-b bits] [-M memory] [-S start_point]
       ssh-keygen -T output_file -f input_file [-v] [-a rounds] [-J num_lines]
                  [-j start_line] [-K checkpt] [-W generator]
       ssh-keygen -s ca_key -I certificate_identity [-h] [-U]
                  [-D pkcs11_provider] [-n principals] [-O option]
                  [-V validity_interval] [-z serial_number] file ...
       ssh-keygen -L [-f input_keyfile]
       ssh-keygen -A
       ssh-keygen -k -f krl_file [-u] [-s ca_public] [-z version_number]
                  file ...
       ssh-keygen -Q -f krl_file file ...
```

```
生成过程记录:
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/data/data/com.termux/files/home/.ssh/id_rsa): ./mykey  // 这里输入了需要保存key的文件名
Enter passphrase (empty for no passphrase):   // 这里可以不用输入，直接回车
Enter same passphrase again:   // 和上一步输入一致，再次回车
Your identification has been saved in ./mykey.  // 这里是私钥保存路径
Your public key has been saved in ./mykey.pub.  // 这里是公钥保存路径
The key fingerprint is:
SHA256:bw0Ps0uf1G3OCDCg1V4WJ7JABjDFNJZLcbToYrTc6Ys u0_a236@localhost
The key's randomart image is:
+---[RSA 2048]----+
|    o=B** . o .  |
|     o+= + o +   |
|    ....+ o o    |
|   o +.+ o o     |
|    = = S B      |
|   . o   . X . . |
|      .   = = . o|
|     . . o + o = |
|    E .   . o . o|
+----[SHA256]-----+
$ cat mykey.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDAGtYfGTm7Mg5lqvTkv/QhvssxffKOdGNK9fKM9LwOiZiN0/07+dfS88GdZDu8Zi5leRPWPPTtPhyWXzUbhZ5KV33xk8da2c48vOcTYHloxv3mva8zm3ZgkCFGSoN8dlWb20Wy7wcmOrkxH1v71N9/i0v50LFhMRYHHAel1z+jGKd/xMITg5CC62G7V8eoGsZDw48DhTzAUab469R/gbTxWsoQTJIZCU0T0k4zOmJpDFs717j2D9pGz0Llok6cQ0oIH4hrG8d529SPznD9KBUzyY2a1Y4GhkK9mNgz2UvwGcFpcNo7f9sMXVTl73+q9hNV+If5BRVK9OGO1GHB4G1r u0_a236@localhost
```

关于公私钥使用：

就拿Linux使用SSH秘钥登录：我们需要将自己的公钥放置到Linux的公钥库中，当我们使用私钥登录时，Linux系统会产生随机数给登录客户端，客户端用私钥加密后发给Linux系统，Linux系统用我们的公钥解密得到和发送一致的随机数，则认证通过。

ssh使用时一般需要的参数有：主机地址，端口，口令/秘钥