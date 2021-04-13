---
title: "sudo chmod 700 /* 补救措施"
date: 2021-04-13T13:01:36+08:00
draft: false
tags: ["linux"]
---

今天使用`sudo chmod 700 ./*`修改文件权限的时候, 不小心少打了一个`.`, 变成了`sudo chmod 700 /*`, 导致所有文件都无法访问, 所有命令都使用不了.

在[Reddit](https://www.reddit.com/r/linuxquestions/comments/c578cl/accidentally_did_sudo_chmod_700/)上找到解决方案, 通过`sudo su`登陆为root, 然后把权限改回来. 

然而现在所有的命令我都用不了, 包括`sudo`. 于是直接打开服务器主机, 在本地进行修改, 但是在服务器上, 登录页面都出不来. 最后想到, 可以直接通过`recovery mode`, 以root身份登录命令行, 进而恢复文件权限.

#### 具体步骤:

如果是以root用户来运行`sudo chmod 700 ./*`的, 直接以root身份恢复文件权限即可 (见3). 如果遇到上述情况(即sudo命令无法使用, 也无法通过root登录), 重启服务器, 执行下述步骤. 

1. 进入GNU GRUB界面 (开机一直按着`Shift`键, 或者BIOS引导界面之后按`Esc`建), 选择`Advanced options for Ubuntu`, 回车.

   ![GNU GRUB](/images/2021-04-13-1.png)

2. 选择最新版本的`recovery mode`, 回车
   
    ![recovery mode](/images/2021-04-13-2.png)

3. 选择`root Drop to root shell prompt`, 回车, 即可以root身份登录命令行.
    
    ![root](/images/2021-04-13-3.png)

4. 输入如下指令, 恢复文件权限.
   ```
    chmod 755 $(find / -maxdepth 1 -type d)
    chmod 777 $(find / -maxdepth 1 -type l)
    chmod 777 /tmp
    chmod 750 /root
    chmod 700 /lost+found
    ```
    
5. 输入`reboot`重启.