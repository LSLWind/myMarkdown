### 使用Github Pages+jekyll

好处是开源，免费，不限流量，只用写而不用太关注管理，不用租服务器自己部署，方便

### 创建Github Pages项目

在github上新建一个仓库，名称为 github昵称.github.io，之后进入settings，这里面提供整体设置，在GitHub Pages下可以选择主题：

![img](https://cdn.sspai.com/20190506142607.jpg?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1) 

此时你可以在settings下的GitHub Pages中看到：  Your site is published at https://lslwind.github.io/ 这样的提示，代表你的GitHub Pages已经可以访问，将项目clone到本地，此时要进行博客搭建

### 使用jekyll

#### 安装ruby、gem以及jekyll

使用jekyll就要安装ruby，进入官网： https://rubyinstaller.org/downloads/ 进行下载，因为是已经整合好的，直接运行exe，安装即可。

之后下载gem，gem是ruby的包管理工具，进入官网： https://rubygems.org/pages/download?locale=zh-CN 下载即可。

下载之后要安装gem：

```ruby
//在RubyGems官网上下载压缩包，解压到你的本地任意位置
//在Terminal中
cd yourpath to RubyGems //你解压的位置
ruby setup.rb
```

有了gem就可以轻松安装jekyll，直接

```ruby
gem install jekyll 
```

这一步就相当于配置开发环境

#### 安装使用git bash

git bash是一个git命令行工具，不过也可以把git bash当成一个windows下的shell来用，当然用windows下的powershell也行，不过用git bash更顺手，git bash 安装： http://www.git-scm.com/download/ 

安装的时候会提示配置一些东西，具体参考为：

![lijoT0.png](https://s2.ax1x.com/2019/12/25/lijoT0.png)

![lijHYT.png](https://s2.ax1x.com/2019/12/25/lijHYT.png)

​                   ![lijz01.png](https://s2.ax1x.com/2019/12/25/lijz01.png)

后面还有一些配置，看英文说明即可，一般使用默认

