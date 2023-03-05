---
layout: post
title: "使用virtualenv创建隔离的python环境"
date: 2013-07-28 18:05
comments: true
categories: python
---

python各种版本之间的不兼容性着实让人头疼, 工作中使用python常常需要一个团队中统一python以及各种库的版本. 在ruby中有一个好用的工具[rvm](<rvm.io>)可以在用户的家目录下安装一个(或多个)本地的ruby, 然后各个工程可以使用自己的ruby和gem的版本.Python里面有没有类似的工具呢? 有的, 这就是virtualenv. 

## 本地安装python 

虽然virtualenv可以直接从系统本身的python中使用, 但是这偏离了我们的初衷: 使用完全隔离的, 和系统默认的python无关的环境. 为什么要这样呢? 因为系统中有很多程序依赖于系统的python, 如果我们替代掉系统的python, 会导致一些程序出错. 例如在centos上, 如果把系统默认的python给换掉, ibus和yum就很有可能起不来了. 如果使用系统的python版本, 又可能让我们开发的应用程序跑不起来, 比较头疼. 所以最好的解决方案还是本地安装一个和系统无关的python.

如果在ubuntu上编译安装python, 建议先安装几个依赖包, 否则编译出来的python会缺少一些功能, 例如没有readline的话, 命令行就不能补全了, 这很蛋疼, 必须预防:

	apt-get install build-essential libncursesw5-dev libreadline5-dev libssl-dev libgdbm-dev libc6-dev libsqlite3-dev tk-dev libgdbm-dev libsqlite3-dev libbz2-dev

安装python可以直接从源代码去安装:

	mkdir ~/src
	wget http://www.python.org/ftp/python/2.7.2/Python-2.7.2.tgz
	tar -zxvf Python-2.7.2.tar.gz
	cd Python-2.7.2
	mkdir ~/.localpython
	./configure --prefix=/home/<user>/.localpython
	make
	make install

把2.7.2换成你需要的python版本. 另外, 执行完configure之后不要急着make, 先确认一下输出中有没有提示你缺少某些模块. 如果有的话, 把这些依赖装上吧, 免得以后麻烦.

## 安装virtualenv

	cd ~/src
	wget http://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.5.2.tar.gz#md5=fbcefbd8520bb64bc24a560c6019a73c
	tar -zxvf virtualenv-1.5.2.tar.gz
	cd virtualenv-1.5.2/ 
	~/.localpython/bin/python setup.py install

注意把1.5.2换成最新的virtualenv版本. 装好以后, 就可以从`~/.localpython`目录执行virtualenv了.

## virtualenv的基本使用

使用`virtualenv`命令可以创建一个隔离环境. 后面可以带参数, 例如:

	virtualenv <envname> --prompt=test

其中`--prompt`后的参数表示在使用该环境后的提示符, 也就是在该环境被激活以后, shell提示符前面会多出来的一串, 用来提示你当前用的是哪个虚拟环境. 执行之后, 会在`<envname>`目录下创建一个虚拟环境, 把需要的脚本文件都拷贝到这个目录下. 在激活这个环境之后, 各种python包就会安装到这个目录下.

要激活一个虚拟环境, 使用 `source env_dir/bin/activate`. 这个脚本会帮你设置一些环境变量, 把python映射到这个目录下. 运行之后会看到shell提示符的变化.

要退出一个虚拟环境, 使用 `deactivate`命令. 仔细观察一下, 这是个在`activate`脚本中定义的函数, 会帮你还原各种相关环境变量.

## 自动安装项目依赖

有了virtualenv之后, 使用pip安装各种包就会安装到当前的虚拟环境下, 从而跟系统的python无关(跟自己在家目录下本地安装的python也无关!). 因此, 可以在不同的虚拟环境下装不同的包, 而不用担心冲突. 不过还有一点不方便的是, python里面没有类似于bundler这种东西, 不能比较方便的自动安装依赖. 不过好在virtualenv可以自定义bootstrap脚本, 方便用户在创建环境时自动安装依赖. 例如:

{% highlight python %}
import virtualenv, textwrap
output = virtualenv.create_bootstrap_script(textwrap.dedent("""
import os, subprocess
packages = [
    "MySQL-python==1.2.4",
    "pycrypto==2.6",
    "pyflakes==0.6.1",
    "pymongo==2.4.1",
    "python-memcached==1.48",
    "redis==2.7.1",
    "simplejson==3.1.3",
    "SQLAlchemy==0.7.4",
    "thrift==0.9.0",
    "tornado==2.4.1",
    "xpinyin==0.4.5",
]
def after_install(options, home_dir):
    for p in packages:
        while True:
            ret = subprocess.call([join(home_dir, 'bin', 'pip'), 'install', p])
            if ret == 0:
                break
            else:
                print "ret = ", ret
                print "Try again: " + p
"""))
f = open('create_env.py', 'w').write(output)
{% endhighlight %}

在`after_install`函数中, 我们使用pip安装指定的包(注意这里有个重试的功能, 天朝的网络你懂的), 也可以做任何其他的操作, 十分灵活(当然, 也十分土). 
把以上代码保存为`boot.py`, 运行一下会产生一个`create_env.py`的脚本, 然后使用这个脚本去代替virtualenv来创建环境, 就可以在环境建好之后自动装好这些依赖.
