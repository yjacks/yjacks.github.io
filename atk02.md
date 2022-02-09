# 02

## 标签显示、输入与按钮

### 代码

```
// main
package main

import (
	"fmt"

	"github.com/visualfc/atk/tk"
)

type Window struct {
	*tk.Window
	edit *tk.Entry
}

func NewWindow() *Window {
	mw := &Window{}
	mw.Window = tk.RootWindow()
	mw.edit = tk.NewEntry(mw)
	mw.edit.SetFocus()
	lbl := tk.NewLabel(mw, "Hello ATK")
	btn := tk.NewButton(mw, "Quit")
	ipt := tk.NewButton(mw, "Show input")
	btn.OnCommand(func() {
		tk.Quit()
	})
	ipt.OnCommand(func() {
		text := mw.edit.Text()
		fmt.Println(text)
	})
	tk.NewVPackLayout(mw).AddWidgets(lbl, tk.NewLayoutSpacer(mw, 0, true), btn, mw.edit, ipt)
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
```

## 涉及函数

`tk.NewLabel`定义一个标签

`tk.NewButton`定义一个按钮

`btn.OnCommand`为按钮绑定函数

`tk.quit`退出

`tk.NewVPackLayout(mw).AddWidgets`放置组件

`mw.ResizeN`设置窗口大小

`mw.SetTitle`设置窗口题目
