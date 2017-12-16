---
title: sublime安装packageControl
tags: [sublime]
---

原来Subl3安装Package Control很麻烦，现在简单的方法来了

一、简单的安装方法

使用Ctrl+`快捷键或者通过View->Show Console菜单打开命令行，粘贴如下代码：
```
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```
如果顺利的话，此时就可以在Preferences菜单下看到Package Settings和Package Control两个菜单了。

顺便贴下Sublime Text2 的代码：
```
import urllib2,os; pf='Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler( ))); open( os.path.join( ipp, pf), 'wb' ).write( urllib2.urlopen( 'http://sublime.wbond.net/' +pf.replace( ' ','%20' )).read()); print( 'Please restart Sublime Text to finish installation')
```
二、手动安装

可能由于各种原因，无法使用代码安装，那可以通过以下步骤手动安装Package Control：

1.点击Preferences > Browse Packages菜单

2.进入打开的目录的上层目录，然后再进入Installed Packages/目录

3.下载[Package Control.sublime-package](https://sublime.wbond.net/Package%20Control.sublime-package)并复制到Installed Packages/目录

4.重启Sublime Text。

