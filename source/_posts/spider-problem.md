---
title: Python + Scrapy 过坑记录
date: 2017-06-02 09:38:39
thumbnail: /img/mine/tianfu_5th_bike.jpeg
tags: 
  - scrapy
  - spider
  - python
  - linux
---
最近用Scrapy写了一个爬虫，开发和部署中遇到了许多坑。此处记录一二。

### 前置：Python的升级（2.6.6 -> 2.7.13）
实例上自带的Python是2.6.6，太老了点，要升级到2.7.13。感觉很麻烦，因为直接yum install我怀疑会污染环境，所以还是选择编译source code安装。
source code安装升级原理很简单：**下载安装**，**改usr/bin下面的引用**
#### 详细步骤:

```bash
cd {python-source-code-root}
./configure
make
make install
```

以上操作，会安装好python2.7.13。和普通source code安装无异。
然后backup现有的python2.6.6，再软连接新的python过去

```bash
mv /usr/bin/python /usr/bin/python2.6.6
ln -s {python2.7.13_bin}/python /usr/bin/python
```

>*如果出现yum出现无法使用，可以打开yum然后把python的引用改成python2.6.6*

### 坑一：安装pip失败

Python准备好了，因为要用scrapy，那自然需要pip来安装一下。
安装pip其实也很简单，同样的首先是**下载安装**，然后**改usr/bin下的引用**

#### 详细步骤
首先在[packaging.python.org](https://packaging.python.org)找到[get-pip.py](https://bootstrap.pypa.io/get-pip.py)的最新下载地址并下载。
然后安装：

```bash
python get-pip.py
```

如果正常安装就好了，可是这个时候坑出现了，输出了这么一顿报错：

```
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
Collecting pip
Could not fetch URL https://pypi.python.org/simple/pip/: 
There was a problem confirming the ssl certificate: Can't connect to HTTPS URL because the SSL module is not available. - skipping
Could not find a version that satisfies the requirement pip (from versions: )
No matching distribution found for pip
```
这个问题的产生，是因为configure source code 前环境中没有安装openssl-devel，并且需要移除一些ssl的注释。

#### 解决方案
1. 安装openssl-devel
2. configure 源代码
3. 打开`{python_source_code_root}/Module/Setup`，并移除以下内容的注释:
```
#SSL=/usr/local/ssl
#_ssl _ssl.c \
#       -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
#       -L$(SSL)/lib -lssl -lcrypto
```
4. `make & make install`

### 坑二：安装Twisted失败

准备好了python，准备好了pip，接下来就要安装一下scrapy了。

#### 详细步骤
```bash
pip install scrapy
```

如果正常安装，就好了...但是还是看见如下报错

```
Could not find a version that satisfies the requirement Twisted>={version} (from scrapy) (from versions: )
No matching distribution found for Twisted>={version} (from scrapy)
```

#### 解决方案
1. 安装bzip2-devel
2. 重新configure你的source code，带上参数 `--enable-ipv6`
3. `make & make install`

### 坑三：import MySQLdb 失败

安装好了scrapy，然后我的爬虫会把数据扔进MySQL，所以需要安装一下`MySQL-python`

#### 详细步骤
```bash
pip install MySQL-python
```
嗯就这一步，一定不会失败的！

但是还是失败了。。。：
```
error: command ‘gcc’ failed with exit status 1
```

#### 解决方案
1. 安装MySQL-python之前，先安装mysql-devel

### 坑四：跑爬虫写数据库的时候失败

环境准备好了，爬虫跑起来。没有详细步骤，因为每个人可能跑的爬虫都不太一样。
但是，我的爬虫又报错了。

```
No module named _sqlite3
```
*为什么，我用的MySQL，可能这个链接库要用到什么吧*

#### 解决方案
1. 安装sqlite-devel
2. 太惨了又要重装。。先configure吧，这次记着再带上一个参数 `--enable-loadable-sqlite-extensions`
3. `make & make install`

### 坑五：MySQL存进去的中文变问号？

爬了一些内容，往数据库一扔，中文变成了问号。明显encode的问题。

#### 解决方案
1. 改掉MySQL的default character，在my.cnf中更新如下内容
```
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```
2. 创建数据库的时候charset setting：`character set utf8 collate utf8_general_ci;`
3. 创建表的时候加入charset setting: `default charset=utf8`
4. 代码中加入charset setting: `# -- coding: UTF-8 --`
5. 如果是使用`MySQLdb`链接库，在创建链接时设置`charset="utf8"`


### 总结

如果你跟着走下来的话，重装挺多遍的啊。
总结一下完整的流程，应该是这样的。

1. 下载python源代码
2. 确保系统中已经安装 `openssl-devel`, `bzip2-devel`, `mysql-devel`, `sqlite-devel`
3. configure源代码，并带上参数`--enable-ipv6`, `--enable-loadable-sqlite-extensions`
4. 移除`Module/Setup`中SSL部分的注释：
```
#SSL=/usr/local/ssl
#_ssl _ssl.c \
#       -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
#       -L$(SSL)/lib -lssl -lcrypto
```
5. `make & make install`
6. 备份/usr/bin中的python，并软链新的python
7. 下载`get-pip.py`并安装`python get-pip.py`
8. 备份/usr/bin中的pip，并软链新的pip
9. `pip install scrapy`
10. `pip install MySQL-python`
11. 修改MySQL的库、表，还有代码的charset
12. 运行代码