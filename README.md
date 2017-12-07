# buildServer
* **Write a document about my problems on the build server**   
* **Git bash command and Linux bash command will be introduced in this document**   
* **Here is the document**   
> tool
```
git bash
putty <windows environment>
node environment
mongo environment
```
> 详细步骤 <该括号内是说明>
* 购买域名和服务器(阿里云 学生认证后9.9/月) **本教程以阿里云Ubuntu 16.04 64位 学生版进行讲解 建议所有账号密码记下来，切勿忘记**
* 未添加安全组前不能使用服务器，请先添加安全组
* 配置所需项步骤：停止服务器 -> 更换系统盘 **友情提示：此操作会重置掉所有服务器之前部署内容，请谨慎操作**
* 所有服务类配置更改后必须重启 **否则出现任何问题概不负责，再次强调，每次对配置进行vi操作都要重启服务**
* Linux基础命令不懂的，请跳转链接进行查看，本文对基础指令不做过多解释
* 配置SSH **如已配置过SSH,勿重复配置，由此导致的之前仓库连接不上，后果自负 `.ssh文件夹`**

   > ssh-agent: 对解密的私钥进行高速缓存
   > ssh-add:   将用户使用的私钥添加到高速缓存的用户列表中
```
   打开git bash <本地git环境即可>
   mkdir .ssh
   ssh-keygen -t rsa -b 4096 -C "email" <email:填自己的邮箱->新手将几个问题直接回车跳过即可，有兴趣自行查看资料进行配置>
   eval "$(ssh-agent -s)" <开启SSH代理>
   cd .ssh
   ssh-add ~/.ssh/id_rsa <添加专用密钥>
   cat id_rsa.pub <查看本地公钥>
```
* 打开putty,输入账号密码登录，首次登录默认用户名为root，密码为三种不同字符那个密码
* 退出puTTy ^D **^代表ctrl，仅解释一遍**
* 增加新用户并给用户升权 **用脚想也知道直接用root不安全对不对**
```
   adduser username <username自己命名，需要记住，以后的登录名,下面所有的username是什么自行体会>
   gpasswd -a username sudo <给该用户升权>
   sudo visudo <进入编辑页>
   在 root ALL=(ALL:ALL) ALL 下加一行 username ALL=(ALL:ALL) ALL <升级用户权限为root权限>
   按提示操作 ^X <退出>
   shift + Y <保存> 
```
* 服务器识别本机密钥 **再次提醒，不会vim操作的请跳转Linux链接进行查看，Linux在文档下方，同为中文文档**
```
   vi authorized_keys <授权公钥>
   编辑 - >将获取的本地公钥右键自动粘贴到此 -> 退出保存
   chmod 600 authorized_keys
   sudo service ssh restart
```
* 修改默认端口22 **修改默认端口来增加安全性**
```
   sudo vi /etc/ssh/sshd_config
   修改配置项{
     port： xxx <范围1~65535，切勿设置1024内的端口，顺便提一嘴，25是个被禁用的端口>
     UseDNS: no
     PermitRootLogin: no <禁用root登录>
     PermitEmptyLogin: no
     结尾追加 AllowUsers username <username是什么你应该清楚吧>
   }
```
* 更新Ubuntu，以便后续配置防火墙
```
   sudo apt-get update && sudo apt-get upgrade <是否继续：你猜呢>
```
* 配置iptables防火墙 **防火墙是个需要反复更改的配置，每次有需要放行的端口都要更改防火墙，切记**
```
   sudo iptables -F <该命令为清空之前配置，想好了加个端口需不需要这步，嗯，就这样>
   sudo vi /etc/iptables.up.rules
   配置文件编写：{
     *filter

     -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT  <允许所有建立起来的链接>

     -A OUTPUT -j ACCEPT <允许所有出去的流量>

     -A INPUT -p tcp --dport 443 -j ACCEPT <允许HTTPS请求的连接>
     -A INPUT -p tcp --dport 80 -j ACCEPT <允许80端口请求的连接>

     -A INPUT -p tcp -m state --state NEW --dport 上文设置的端口号(没改过为22) -j ACCEPT <只能从该端口登录，其他端口登录会被防火墙拦截>

     -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT <允许ping>

     <mongoDB允许端口>
     -A INPUT -s 127.0.0.1 -p tcp --destination-port 27017 -m state --state NEW,ESTABLISHED -j ACCEPT
     -A OUTPUT -d 127.0.0.1 -p tcp --source-port 27017 -m state --state ESTABLISHED -j ACCEPT

     -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied:" --log-level 7 <记录被拒绝的请求>

     <对恶意访问的IP加以拦截>
     -A INPUT -p tcp --dport 80 -i eth0 -m state --state NEW -m recent --set
     -A INPUT -p tcp --dport 80 -i eth0 -m state --state NEW -m recent --update --seconds 60 --hitcount 150 -j DROP <如果某个IP在80端口在60s内发出了150次请求就拦截掉（60，150可更改）>

     <拒绝所有其他进到服务器的流量 注意后续引入新的接口/新的协议/新的数据库/外网/某个IP段要来加规则，切记>
     -A INPUT -j REJECT
     -A FORWARD -j REJECT

     COMMIT
   }
   sudo iptables-restore < /etc/iptables.up.rules <将配置文件位置吐给服务器，每次更改防火墙后这步也要执行>
   sudo ufw status <看防火墙有没有成功被建立起来>
   没有： sudo ufw enable
```
* 防火墙开机自启，避免每次手动启动
```
   sudo vi /etc/network/if-up.d/iptables
   文件配置：{
     #!/bin/sh <这句代表本配置文件为脚本>
     iptables-restore /etc/iptables.up.rules
   }
   sudo chmod +x /etc/network/if-up.d/iptables <给脚本执行的权限>
```
* Fail2Ban
```
   sudo apt-get install fail2ban <安装Fail2Ban>
   sudo vi /etc/fail2ban/jail.conf
   更改配置文件：{
     bantime: xxx <可以设置的大一些，如3600>
			  destemail: xxx <设置为自己的邮箱>
			  action = %(action_mv)s
			  ssh: {
				  enabled = true
				  port = ssh
				  filter = sshd
				  logpath = /var/log/auth.log
				  maxretry = 6
			  }
   }
   sudo service fail2ban status <查看fail2ban有没有运行>
   sudo service fail2ban start <开启fail2ban>
   sudo service fail2ban stop <停止fail2ban>
```
### [git basic command intriduce](https://github.com/ajun568/git_basic_command)
### [Linux basic command introduce](https://github.com/ajun568/linux_basic_command)
