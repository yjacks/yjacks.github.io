# go atk教程01

## 前言:

golang下面有一些GUI库,但是有文档的几乎没有(我只看到go gtk有一个第三方的具体文档，但那玩意多少年了),于是我将目光对准了atk,作者有一个(可能)是基于它写的程序 LiteIDE,这个库跟Python中的tkinter一样,是一个对tcl/tk的binder(原文),库最近一次更新是一年前,相对比较新,而且没有那么复杂。

    本人需要从源代码和Demo中理清逻辑,所以更新估计以月为单位.(幸运的是,函数名与定义方式都比较明了,可以看出功能)

0.安装

我以linux为基础,Arch下安装tk:sudo pacman -S tk

tcl自带

之后go get github.com/visualfc/atk

## 启动一个空窗口

### 代码:

`package main

import (
	"github.com/visualfc/atk/tk"
)

type Window struct {
	*tk.Window
}

func NewWindow() *Window {
	mw := &Window{tk.RootWindow()}
	mw.ResizeN(300, 200)
	return mw
}

func main() {
	tk.MainLoop(func() {
		mw := NewWindow()
		mw.SetTitle("ATK Sample")
		mw.ShowNormal()
	})
}
`
### 解析
`tk.Window` 指向一个基础框架
`tk.RootWindow`创建一个基础窗口
`mw.ResizeN`定义窗口大小
`tk.Mainloop`主循环
`mw.setTitle`设置一个标题
`mw.ShowNormal`显示窗口
