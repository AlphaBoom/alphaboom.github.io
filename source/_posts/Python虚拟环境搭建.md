---
title: Python虚拟环境搭建
date: 2017-02-02 21:18:41
tags: 笔记
---
> 简单做下记录，毕竟python不咋用。用的时候还要到处找资料。转载至[Python 虚拟环境：Virtualenv](http://liuzhijun.iteye.com/blog/1872241)
<!--more-->

## virtualenv
virtualenv用于创建独立的Python环境，多个Python相互独立，互不影响，它能够：

1. 在没有权限的情况下安装新套件
2. 不同应用可以使用不同的套件版本
3. 套件升级不影响其他应用

### 安装

```bash
sudo apt-get install python-virtualenv
```
### 使用方法
```
virtualenv [虚拟环境名称]
```
如，创建**ENV**的虚拟环境

```bash
virtualenv ENV
```

默认情况下，虚拟环境会依赖系统环境中的site packages，就是说系统中已经安装好的第三方package也会安装在虚拟环境中，如果不想依赖这些package，那么可以加上参数 --no-site-packages建立虚拟环境

```bash
virtualenv --no-site-packages [虚拟环境名称]
```
### 启动虚拟环境
```bash
cd ENV
source ./bin/activate
```
注意此时命令行会多一个(ENV)，ENV为虚拟环境名称，接下来所有模块都只会安装到该目录中去。

### 退出虚拟环境
```bash
deactivate
```
### 在虚拟环境安装Python套件
Virtualenv 附带有pip安装工具，因此需要安装的套件可以直接运行：

```bash
pip install [套件名称]
```
如果没有启动虚拟环境，系统也安装了pip工具，那么套件将被安装在系统环境中，为了避免发生此事，可以在~/.bashrc文件中加上：

```bash
export PIP_REQUIRE_VIRTUALENV=true
```
或者让在执行pip的时候让系统自动开启虚拟环境：

```bash
export PIP_RESPECT_VIRTUALENV=true
```
## Virtualenvwrapper
Virtaulenvwrapper是virtualenv的扩展包，用于更方便管理虚拟环境，它可以做：

1. 将所有虚拟环境整合在一个目录下
2. 管理（新增，删除，复制）虚拟环境
3. 切换虚拟环境
4. ...

### 安装
```
sudo easy_install virtualenvwrapper  
```
此时还不能使用virtualenvwrapper，默认virtualenvwrapper安装在/usr/local/bin下面，实际上你需要运行virtualenvwrapper.sh文件才行，先别急，打开这个文件看看,里面有安装步骤，我们照着操作把环境设置好。

1. 创建目录用来存放虚拟环境

	```bash
	mkdir $HOME/.virtualenvs
	```
2. 在~/.bashrc中添加行： export WORKON_HOME=
$HOME/.virtualenvs

3. 在~/.bashrc中添加行：source /usr/local/bin/virtualenvwrapper.sh

4. 运行： source ~/.bashrc

此时virtualenvwrapper就可以使用了。

### 列出虚拟环境列表

```bash
workon
```
也可以使用

```bash
lsvirtualenv
```
### 新建虚拟环境

```bash
mkvirtualenv [虚拟环境名称]
```
### 启动/切换虚拟环境

```bash
workon [虚拟环境名称]
```
### 删除虚拟环境

```bash
rmvirtualenv [虚拟环境名称]
```
### 离开虚拟环境

```bash
deactivate
```
## 参考：
http://www.virtualenv.org/en/latest/
http://stackoverflow.com/questions/11372221/virtualenvwrapper-not-found
http://www.openfoundry.org/tw/tech-column/8516-pythons-virtual-environment-and-multi-version-programming-tools-virtualenv-and-pythonbrew
http://virtualenvwrapper.readthedocs.org/en/latest/index.html
