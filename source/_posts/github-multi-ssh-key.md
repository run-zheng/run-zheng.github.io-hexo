---

title: github配置ssh及多ssh key问题处理
date: 2018-09-16 22:49:51
tags: [github,ssh,ssh key]
categories: github 博客


---

## 一、生成ssh公私钥
用ssh-keygen生成公私钥。
``` shell
$ssh-keygen -t rsa -C "你的邮箱" -f ~/.ssh/id_rsa_mult
```
在~/./ssh目录下会生成一对文件id_rsa_mult和id_rsa_mult.pub文件 

![ssh-keygen-mult][1]

## 二、编辑config文件，增加多用户支持

在ssh用户的配置文件~/.ssh/config增加github-mult.com的配置
``` shell 
$touch config 
$vi config 
```
![ssh-config][2]

## 三、 解决Enter passphrase for key 问题

在后续使用id_rsa_mult过程中，会出现输入私钥的key， 在事先可以将key加入，解决该问题

``` shell 
$ssh-agent bash 
$ssh-add -l    #列出已经添加的key  
$ssh-add -D   #清理下 
$ssh-add ~/.ssh/id_rsa   #添加id_rsa秘钥
$ssh-add ~/.ssh/id_rsa_mult  #添加id_rsa_mult秘钥 
$ssh-add -l 
```

![ssh-add-key][3]

## 四、配置github的公钥
![github-add-ssh-key][4]

通过ssh -T git@github-mult.com 确认是否配置正确： 
``` shell
$ssh -T git@github-mult.com 
```
![test-github-connect-key][6]

注意： 是git@github-mult，不是git@github.com， git仓库地址复制过来后也要改一下  
测试clone仓库：
``` shell 
$git clone git@github-mult.com:xxxx/xxxx.github.io.git 
```
![test-github-clone][5]


  [1]: github-multi-ssh-key/ssh-keygen-mult.png
  [2]: github-multi-ssh-key/ssh-config.png
  [3]: github-multi-ssh-key/ssh-add-key.png
  [4]: github-multi-ssh-key/github-add-ssh-key.png
  [5]: github-multi-ssh-key/test-github-clone.png
  [6]: github-multi-ssh-key/test-github-connect-key.png