---
title: Ubuntu上的包管理：dpkg，apt和aptitude
excerpt: "最开始的时候，Linux上的软件以源代码的方式发布，用户下载源代码包（通常打包为.tar.gz），然后自行编译。dpkg是Debian软件包管理器的基础，它被伊恩&middot;默多克创建于1993年。dpkg与RPM十分相似，同样被用于安装、卸载和供给和.deb软件包相关的信息。本身是一个底层的工具，本身并不能从远程包仓库下载包以及处理包的依赖的关系，基于dpkg的上层工具，如APT，则用于从远程获取软件包以及处理复杂的软件包关系。"
date: '2011-10-23 12:34:20 +0800'
categories:
- Linux
tags:
- linux
- ubuntu
---

### 简述

最开始的时候，Linux上的软件以源代码的方式发布，用户下载源代码包（通常打包为.tar.gz），然后自行编译。
dpkg是Debian软件包管理器的基础，它被伊恩&middot;默多克创建于1993年。dpkg与RPM十分相似，同样被用于安装、卸载和供给和.deb软件包相关的信息。
dpkg本身是一个底层的工具，本身并**不能**从远程包仓库下载包以及处理包的依赖的关系，基于dpkg的上层工具，如APT，则用于**从远程获取**软件包以及**处理复杂的软件包关系**。

APT全称Advanced Packaging Tool，可以自动下载，配置，安装二进制或者源代码格式的软件包，因此简化了Unix系统上管理软件的过程。现在Debian和其衍生发行版（如Ubuntu）中都包含了APT。
APT最早是基于dpkg的开发的，只用来处理deb格式的软件包。现在经过APT-RPM组织修改，APT已经可以安装在支持RPM的系统管理RPM包。
而aptitude是一个APT的文本界面客户端，现在也逐渐加入了GUI的界面，详见<a title="Aptitude 有了 GTK+ GUI" href="http://linuxtoy.org/archives/gtk-gui-for-aptitude.html" target="_blank">http://linuxtoy.org/archives/gtk-gui-for-aptitude.html</a>
Synaptic是Ubuntu中自带的APT的GUI客户端，也就是传说中的新立得。

### dpkg命令

(来自：<a href="http://linuxtoy.org/archives/dpkg_reference.html" target="_blank">http://linuxtoy.org/archives/dpkg_reference.html</a>)
<table>
<tbody>
<tr>
<th width="200">命令</th>
<th>作用</th>
</tr>
<tr>
<td>dpkg -i package.deb</td>
<td>安装包</td>
</tr>
<tr>
<td>dpkg -r package</td>
<td>删除包</td>
</tr>
<tr>
<td>dpkg -P package</td>
<td>删除包（包括配置文件）</td>
</tr>
<tr>
<td>dpkg -L package</td>
<td>列出与该包关联的文件</td>
</tr>
<tr>
<td>dpkg -l package</td>
<td>显示该包的版本</td>
</tr>
<tr>
<td>dpkg --unpack package.deb</td>
<td>解开 deb 包的内容</td>
</tr>
<tr>
<td>dpkg -S keyword</td>
<td>搜索所属的包内容</td>
</tr>
<tr>
<td>dpkg -l</td>
<td>列出当前已安装的包</td>
</tr>
<tr>
<td>dpkg -c package.deb</td>
<td>列出 deb 包的内容</td>
</tr>
<tr>
<td>dpkg --configure package</td>
<td>配置包</td>
</tr>
</tbody>
</table>
注意：更多选项可通过 dpkg -h 查询，有些指令需要超级用户权限才能执行

### APT命令

(来自：<a href="http://linuxtoy.org/archives/apt_reference.html" target="_blank">http://linuxtoy.org/archives/apt_reference.html</a>)
<table>
<tbody>
<tr>
<th width="200">命令</th>
<th>作用</th>
</tr>
<tr>
<td>apt-cache search package</td>
<td>搜索包</td>
</tr>
<tr>
<td>apt-cache show package</td>
<td>获取包的相关信息，如说明、大小、版本等</td>
</tr>
<tr>
<td>sudo apt-get install package</td>
<td>安装包</td>
</tr>
<tr>
<td>sudo apt-get install package --reinstall</td>
<td>重新安装包</td>
</tr>
<tr>
<td>sudo apt-get -f install</td>
<td>强制安装</td>
</tr>
<tr>
<td>sudo apt-get remove package</td>
<td>删除包</td>
</tr>
<tr>
<td>sudo apt-get remove package --purge</td>
<td>删除包，包括删除配置文件等</td>
</tr>
<tr>
<td>sudo apt-get autoremove</td>
<td>自动删除不需要的包</td>
</tr>
<tr>
<td>sudo apt-get update</td>
<td>更新源</td>
</tr>
<tr>
<td>sudo apt-get upgrade</td>
<td>更新已安装的包</td>
</tr>
<tr>
<td>sudo apt-get dist-upgrade</td>
<td>升级系统</td>
</tr>
<tr>
<td>sudo apt-get dselect-upgrade</td>
<td>使用 dselect 升级</td>
</tr>
<tr>
<td>apt-cache depends package</td>
<td>了解使用依赖</td>
</tr>
<tr>
<td>apt-cache rdepends package</td>
<td>了解某个具体的依赖</td>
</tr>
<tr>
<td>sudo apt-get build-dep package</td>
<td>安装相关的编译环境</td>
</tr>
<tr>
<td>apt-get source package</td>
<td>下载该包的源代码</td>
</tr>
<tr>
<td>sudo apt-get clean &amp;&amp; sudo apt-get autoclean</td>
<td>清理下载文件的存档</td>
</tr>
<tr>
<td>sudo apt-get check</td>
<td>检查是否有损坏的依赖</td>
</tr>
</tbody>
</table>
备注：package 为软件包名称。

### aptitude命令
（来自<a href="http://linuxtoy.org/archives/aptitude_quick_reference.html" target="_blank">http://linuxtoy.org/archives/aptitude_quick_reference.html</a>）
aptitude是基于APT的又一个包管理的前端，aptitude似乎在处理依赖问题上更佳一些。据说aptitude 另外用一份数据量很小的扩展标记来实现所谓更佳的管理，我没有发现有什么比apt命令更加牛逼的功能，不过看起来命令比apt要简洁。
<table>
<tbody>
<tr>
<th width="200">命令</th>
<th>作用</th>
</tr>
<tr>
<td>aptitude update</td>
<td>更新可用的包列表</td>
</tr>
<tr>
<td>aptitude upgrade</td>
<td>升级可用的包</td>
</tr>
<tr>
<td>aptitude dist-upgrade</td>
<td>将系统升级到新的发行版</td>
</tr>
<tr>
<td>aptitude install pkgname</td>
<td>安装包</td>
</tr>
<tr>
<td>aptitude remove pkgname</td>
<td>删除包</td>
</tr>
<tr>
<td>aptitude purge pkgname</td>
<td>删除包及其配置文件</td>
</tr>
<tr>
<td>aptitude search string</td>
<td>搜索包</td>
</tr>
<tr>
<td>aptitude show pkgname</td>
<td>显示包的详细信息</td>
</tr>
<tr>
<td>aptitude clean</td>
<td>删除下载的包文件</td>
</tr>
<tr>
<td>aptitude autoclean</td>
<td>仅删除过期的包文件</td>
</tr>
</tbody>
</table>
当然aptitude也是text-based，也就是命令行模式的

### Synaptic

由于synaptic是GUI界面的，没啥命令好说的，其实这个在Ubuntu里面也挺少用到的，更多的还是通过apt-get命令就可以搞定。
