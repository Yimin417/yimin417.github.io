# 利用Travis CI实现博客的自动部署

操作之前看了网上的教程，原本以为会很快搞定，结果还是折腾了好久。有关具体部署的步骤网上有很多文章，我把我踩雷的一些注意点记录一下。

## 主要思路
- 设置好github，为的是告知Travis CI发生的变化
- 设置Travis CI，将构建好的文章部署到GitHub pages
- 连接github与Travis CI，这两者之间是如何通信的，互相如何通知的

***
两者之间要想实现通信，主要通过配置文件，解析配置文件时会"教"他们如何操作
***

### 第一步：获取 GitHub Personal Access Token

点击 Github 用户settings页面 最下方的 Developer setting ，然后选择 Personal access tokens 来生成一个新的token。

**注意：** 

1. generate new token时，token的名字可以随便起，但是要记住，因为后面要用。
2. select scopes时只勾选repo即可（下面的四个子选项就会自动选择），其他不勾选。然后点击generate token即可。
3. 生成的token一定要记在某个地方，比如我把token发到自己微信了，算是备份，因为后面要用到，不然刷新后，token就不能再看了，只会显示刚才创建token时生成的名字，就还得重新生成一遍。


### 第二步：Travis CI相关的配置


主要包括两部分：环境变量配置与.travis.yml文件的修改

公有仓库使用<https://travis-ci.org/>进行操作，私有仓库需要使用收费的<https://travis-ci.com/>


**配置环境变量**

在Legacy Services Integration里面打开yimin417.github.io这个仓库，然后到settings里面设置，其他选项保持默认即可，除了在Environment Variables里面添加如下：
name  添加你前面在github里面创建的token名
value 添加在github里面生成的token
然后保存就好

**配置.travis.yml文件**


#指定构建环境是Node.js，当前版本是稳定版
```
language: node_js
node_js: stable
```
#设置缓存文件
```
cache:
  directories:
    - node_modules

```
#设置钩子只检测blog-source分支的push变动
```
branches:
  only:
    - sourcode
```


#在构建之前安装hexo环境
```
before_install:
   - npm install -g hexo-cli
```

#安装git插件和搜索功能插件
```
install:
 - npm install
  - npm install hexo-deployer-git --save
 ``` 
#执行清缓存，生成网页操作
```
script:
  - hexo clean
  - hexo generate
```

#设置git提交名，邮箱；替换真实token到_config.yml文件
```
after_script:
  - git config user.name "yimin417"
  - git config user.email "1414876235@qq.com"
#替换同目录下的_config.yml文件中github_token字符串为travis后台刚才配置的变量，注>意此处sed命令用了双引号。单引号无效！
  - sed -i "s/access_token/${ACCESS_TOKEN}/g"./_config.yml
  - hexo deploy
```

### 第三步：对配置文件_config.yml修改

只修改最后
```
deploy:
  type: git
  repository: https://access_token@github.com/yimin417/yimin417.github.io
  branch: master
```
#这里面的branch是写博客的分支

*注：博客源代码在新创建的分支sourcode下*

然后，所有的配置就完成了。

## git提交相关命令及问题说明

1. git checkout 分支名  切换分支
2. git checkout -b 分支名 首先创建分支然后再切换到新建分支上
3. push时提交到的是存放博客源码的分支，因为Travis监听的是这个分支的变化，我的该分支是sourcode



