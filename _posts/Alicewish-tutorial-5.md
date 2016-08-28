---
title: 墨问非名-汉化教程5——自动分页填字
date: 2016-05-26 07:03:22
categories: 教程
tags: [汉化,填字]
toc: true
---
<span id="busuanzi_container_page_pv">
  本文总阅读量<span id="busuanzi_value_page_pv"></span>次
</span>

## 前言

我知道我做的美漫汉化教程挺欺负人的，该写清楚的地方云里雾里。唉，可是懒癌上来挡不住啊。

## 准备

首先要有格式准确的稿子。

![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_231655.png)
<!-- more -->
意思就是每页页码标清楚，**这是重点！**

![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_233314.png)



然后在保留格式信息的情况下转成纯文本：

### 把换行替换成の

```vbscript
Sub 缩行()
With Selection.Find
	.Text = "^p" '查找
	.Replacement.Text = "の" '替换
	.Wrap = wdFindContinue
	.MatchByte = True
	.MatchWildcards = False '不使用通配符
End With
Selection.Find.Execute Replace:=wdReplaceAll '全部替换
End Sub
```

![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_232610.png)

### 转换成Markdown

因为上一步处理，不会多出奇怪的换行。

![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_232715.png)

### 扔进谷歌表格拿正则表达式过一过

![屏幕快照 2016-05-25 下午11.30.37](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-屏幕快照 2016-05-25 下午11.30.37.png)

### 变成可以用来填字并且有格式指示的文档

![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_233123.png)

### 扔进Word

![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_233218.png)



## 重要概念

知道`软换行`和`硬换行`的区别：

### 软换行

1. `Shift`+`Enter`
2. 不另起段落
3. 在Word查找替换中以`^l`表示
4. 符号形如↓![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_232131.png)

### 硬换行

1. `Enter`
2. 另起段落
3. 在Word查找替换中以`^p`表示
4. 符号形如↵![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_232333.png)

## 根据页码标记转换换行类型

```vbscript
Sub 填字转分页()
With Selection.Find
.Text = "^p" '查找硬换行
.Replacement.Text = "^l" '替换为软换行
.Wrap = wdFindContinue
.MatchByte = True
.MatchWildcards = False '不使用通配符
End With
Selection.Find.Execute Replace:=wdReplaceAll '全部替换
With Selection.Find
.Text = "^11^11([0-9])([0-9])^11^11" '查找[软换行*2][两位数字][软换行*2]格式的页码
.Replacement.Text = "^p^l" '替换为[硬换行][软换行]
.Wrap = wdFindContinue
.MatchWildcards = True '不使用通配符
End With
Selection.Find.Execute Replace:=wdReplaceAll '全部替换
End Sub
```

![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_234554.png)

注意行末符号：

![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_234723.png)

这样一页就变成对应一个段落。

Word中选取下一段落的快捷键是`Ctrl`+`Shift`+`↓`，

然后交给`AutoHotKey`吧：

![](http://o7ru3d96x.bkt.clouddn.com/2016-05-26-2016-05-25_235322.png)

```autohotkey
;==================Word快捷键==================
#IfWinActive, ahk_class OpusApp, 填
Esc::Exit
^+#!K:: ;Word填字
{
WinGetTitle, Title ;获取窗口名
FormatTime, TimeStringStart, yyyy/MM/dd hh:mm:ss tt R
SetKeyDelay, 100
Loop, 22
{
Send ^+{Down} ;选择下一段落
Send ^c ;复制
Send {Down} ;下
Send #3 ;切换到记事本
Sleep 1000 ;延时1秒
Send ^a ;全选
Send ^v ;粘贴
Send ^s ;保存
Send #4 ;切换到PS
Sleep 1000 ;延时1秒
Send {f10} ;运行脚本
Sleep 1000 ;延时1秒
Loop ;判断脚本是否执行完
    {
        Sleep, 1000
        IfExist, \\Mac\Host\Volumes\Mack\汉化\-.txt
            break
    }
FileDelete, \\Mac\Host\Volumes\Mack\汉化\-.txt ;删除小文档
Sleep 1000 ;延时1秒
Send ^{Tab} ;切换到下一页
Sleep 1000 ;延时1秒
Send #2 ;切换到WORD
Sleep 1000 ;延时1秒
}
FileDelete, \\Mac\Home\Documents\填字完成.txt ;删除填字完成文档
Sleep 1000 ;延时1秒
SoundBeep, 750, 500 ;以较高的音高进行发音并持续半秒.
Sleep 1000 ;延时1秒
FormatTime, TimeStringEnd, yyyy/MM/dd hh:mm:ss tt R
FileAppend,
(

填字项目：%Title%
开始时间：%TimeStringStart%
完成时间：%TimeStringEnd%

), \\Mac\Home\Documents\填字完成.txt
return
}
```
## 后记

视频背后大概就是这样。

有问题评论~