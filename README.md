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
* 所有服务类配置更改后必须重启 **否则出现任何问题概不负责**
* Linux基础命令不懂的，请跳转链接进行查看，本文对基础指令不做过多解释
* 配置SSH **如已配置过SSH,勿重复配置，由此导致的之前仓库连接不上，后果自负 `.ssh文件夹`**

   > ssh-agent: 对解密的私钥进行高速缓存
   > ssh-add:   将用户使用的私钥添加到高速缓存的用户列表中
   ```
   打开git bash <本地git环境即可>
   mkdir .ssh
   ssh-keygen -t rsa -b 4096 -C "email" <email:填自己的邮箱 -> 新手将几个问题直接回车跳过即可，有兴趣自行查看资料进行配置>
   eval "$(ssh-agent -s)" <开启SSH代理>
   cd .ssh
   ssh-add ~/.ssh/id_rsa <添加专用密钥>
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
### [git basic command intriduce](https://github.com/ajun568/git_basic_command)
### [Linux basic command introduce](https://github.com/ajun568/linux_basic_command)
