---

title: hexo安装、生成blog并deploy到github
date: 2018-09-16 23:23:01
tags: [github,hexo,github pages]
categories: github 博客


---


## 一、安装 nodejs
从[https://nodejs.org/en/download/](https://nodejs.org/en/download/)下载nodejs, 安装一路下一步就ok
![install-nodejs][1]


## 二、安装hexo 
安装完nodejs后， 用npm安装hexo。  

``` shell
$npm install -g hexo-cli  
$hexo -v 
```

![install-hexo][2]


## 三、用hexo初始化并生成blog

#### 1、用hexo初始化并生成blog

``` shell
$hexo init jack-demo 
```

![hexo-init-blog][3]

#### 2、安装依赖，然后用hexo generate， 也可以用缩写hexo g生成静态页面

``` shell 
$cd jacky-demo 
$ls -l 
$npm install 
$hexo generate 
```

![install-hexo-deps][4]

#### 3、生成静态页面后，可以用hexo server启动服务器，并通过[http://localhost:4000](http://localhost:4000)访问，默认主题比较丑。

``` shell
$hexo server 
```

![hexo-server-blog][5]
![blog-index][6]



## 四、更换主题成indigo
#### 1、从github clone indigo主题， clone后，安装主题需要的依赖。 

``` shell
$git clone git@github.com:yscoder/hexo-theme-indigo.git themes/inidgo
$npm install hexo-renderer-less --save   #安装less,作为css的预处理工具
$npm install hexo-generator-feed --save #安装rss的feed生成工具
$npm install hexo-generator-json-content --save #用于生成静态站点数据，用于站内搜索的数据源
$npm install hexo-helper-qrcode --save #用于生成微信分享二维码
```

![install-indigo-theme   ][7]   


#### 2、开启标签页

``` shell 
$hexo new page tags 
```

![new-tags-page          ][8]   


编辑source/tags/index.md，增加layout和comments
![edit-tags-page         ][9]   


#### 3、开启分类页

``` shell 
$hexo new page categories 
```

![new-categories-page    ][10]  

编辑source/categories/index.md，增加layout和comments
![edit-categories-page   ][11]  

#### 4、修改_config.yml，使用indigo主题
![edit-config-yml        ][12]  

注意： _config.yml的author改成自己，indigo会用来显示昵称， themes/indigo/_config.yml里的email改成自己的email 

#### 5、重新生成静态页面并启动服务器(需要调试信息可以使用，hexo s --debug)

``` shell 
$hexo clean & hexo g & hexo s 
```

![clean-generate-server  ][13]  

#### 6、效果如下：
![indigo-theme-index     ][14]  


## 五、注册github并配置ssh
到https://github.com/ 注册账号，然后配置ssh登录。 
1、配置git的登录信息

``` shell
$git config --global user.name "你的git用户名"
$git config --global user.email "你的git登录邮箱"
```

![github-ssh-config      ][15]  

2、生成ssh公私钥

``` shell
$ssh-keygen -t rsa -C "你的git登录邮箱"
```

![ssh-keygen-rsa         ][16]  


3、设置github的ssh key
将id_rsa.pub的内容拷贝到github的ssh key中
![id_rsa_pub             ][17]  
![set-github-ssh-key     ][18]  

4、测试链接github设置的ssh key免登陆是否生效

``` shell
$ssh -T git@github.com
```

![test-key-connect-github][19]  


## 六、上传博客到github 
#### 1、新增git仓库
github上新建一个以注册的昵称开头的repository。 比如演示用的昵称是jacky-dmeo， repository的名称是jacky-demo.github.io 。 

#### 2、配置deploy的地址
type为git， repository配置为1新增的git仓库地址
![edit-config-deploy       ][20]  


#### 3、安装hexo deploy插件

``` shell
$ npm install hexo-deployer-git --save
```

![install-hexo-deployer-git][21]  

#### 4、上传到github

``` shell 
$hexo deploy 
```

![hexo-deploy-1            ][22]  
![hexo-deploy-2            ][23]  

#### 5、查看github的博客，看下效果
![github-blog-index        ][24]  


  [1]: use-hexo-gen-blog-deploy-github/install-nodejs.png
  [2]: use-hexo-gen-blog-deploy-github/install-hexo.png
  [3]: use-hexo-gen-blog-deploy-github/hexo-init-blog.png
  [4]: use-hexo-gen-blog-deploy-github/install-hexo-deps.png
  [5]: use-hexo-gen-blog-deploy-github/hexo-server-blog.png
  [6]: use-hexo-gen-blog-deploy-github/blog-index.png
  [7]: use-hexo-gen-blog-deploy-github/install-indigo-theme.png   
  [8]: use-hexo-gen-blog-deploy-github/new-tags-page.png
  [9]: use-hexo-gen-blog-deploy-github/edit-tags-page.png
  [10]: use-hexo-gen-blog-deploy-github/new-categories-page.png
  [11]: use-hexo-gen-blog-deploy-github/edit-categories-page.png
  [12]: use-hexo-gen-blog-deploy-github/edit-config-yml.png
  [13]: use-hexo-gen-blog-deploy-github/clean-generate-server.png
  [14]: use-hexo-gen-blog-deploy-github/indigo-theme-index.png
  [15]: use-hexo-gen-blog-deploy-github/github-ssh-config.png 
  [16]: use-hexo-gen-blog-deploy-github/ssh-keygen-rsa.png 
  [17]: use-hexo-gen-blog-deploy-github/id_rsa_pub.png 
  [18]: use-hexo-gen-blog-deploy-github/set-github-ssh-key.png 
  [19]: use-hexo-gen-blog-deploy-github/test-key-connect-github.png 
  [20]: use-hexo-gen-blog-deploy-github/edit-config-deploy.png 
  [21]: use-hexo-gen-blog-deploy-github/install-hexo-deployer-git.png 
  [22]: use-hexo-gen-blog-deploy-github/hexo-deploy-1.png 
  [23]: use-hexo-gen-blog-deploy-github/hexo-deploy-2.png 
  [24]: use-hexo-gen-blog-deploy-github/github-blog-index.png 




