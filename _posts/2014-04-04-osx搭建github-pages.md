---
layout: post
tags : [osx github jekyll 搭建博客] 
---

## 一句话
主要介绍下mac下搭建github pages,针对用mac终端开发的用户


### 建立github page
1.由于mac终端已经集成很github,就不介绍github的安装了

2.下面正式开始，建立**github page**，我推荐大家用[jekyll bootstrap](http://jekyllbootstrap.com/usage/jekyll-quick-start.html),非常方便，我的博客也是用过这个建的，大家可以直接这里面看介绍，一步一步操作，我就不累赘介绍了

### 购买域名
这个可选,如果不买的话，可以直接跳过此步骤,买了域名后，你需要修改两处地方：

1.新增一个github的CNAME文件,里面填写自己的域名即可,像我的是:`cnedwan.com`

2.在域名购买的网站修改域名映射，映射到github

### 本地写博客
到了这一步，大家的线上博客应该都已经搭好了，在实际写博客的时候，不可能总是push到线上去看，所以必需得有一个本地写博环境，在前面这2步，我也没用多少时间，我在搭建github page时主要时间都耗在了这上面，因为工作比较忙，一直拖到了昨天有空才解决了，下面废话不多说，开始吧

1.我们用的本地写博客工具是jekyll，前面说的[jekyll bootstrap](http://jekyllbootstrap.com/usage/jekyll-quick-start.html),也是在这个基础上搭建了很多东西，方便开发者们，快速使用

2.安装jekyll:`sudo gem install jekyll`，直接安装的话，应该会报错，所以需要按以下几步走 唉--！		
		
		curl -L https://get.rvm.io | bash -s stable --ruby
		rvm pkg install openssl
		rvm reinstall all --force
		rvm remove 2.0.0
		rvm install 1.9.3 -C --with-openssl-dir=$HOME/.rvm/usr
		sudo gem install jekyll

3.接下来用`jekyll serve`试试

4.至此jekyll应该已经装好了，再装不好，那就是人品实在太差了


怎么样是不是非常方便，赶紧开始写博吧:)

#####注：在用博客的时候，可能不想用`jekyll bootstrap`的主题，想自己捣鼓下，没有关系，如果有node环境的话，用npm安装一个less编译工具就可以了`npm install -g less`

如果你喜欢ED的文章就赶紧点`赞`吧~~
