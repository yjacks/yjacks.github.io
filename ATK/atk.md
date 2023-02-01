============================================================================

##### 1 引言

```go
atk：Another Golang Tcl/Tk binding GUI ToolKit

myguitk.go

go mod init mygui
go run mygui
go build -ldflags -H=windowsgui 运行时隐藏golang程序自己的cmd窗口
go build -o mygui.exe -ldflags -H=windowsgui
go build -o mygui.exe -ldflags="-H windowsgui -w -s"

success : mygui.exe,lib\tk8.6,lib\tcl8.6,tk86.dll,tcl86.dll,zlib1.dll

usage: go build [-o output] [-i] [build flags] [packages]

// 通过键盘快捷键Win+Ctrl+O键可以打开或者关闭虚拟键盘

```

myguitk.go

```go
package main

import "github.com/visualfc/atk/tk"

type Window struct { *tk.Window }

func NewWindow() *Window {
	mw := &Window{}
	mw.Window = tk.RootWindow()

	lbl := tk.NewLabel(mw, "Hello ATK", tk.LabelAttrForground("blue"))
	btn := tk.NewButton(mw, "Quit")
    btn.SetTakeFocus(false)
	btn.OnCommand(func() { tk.Quit() })
	
    tk.NewVPackLayout(mw).AddWidgets(lbl, btn)
	return mw
}

func main() {
	win := func() {
		mw := NewWindow()
		mw.SetTitle("ATK Sample")
		mw.Center(nil)
		mw.ShowNormal()
		mw.ResizeN(300, 200)
	}
	tk.MainLoop(win)
}

```

基本框架

```go
package main

import "github.com/visualfc/atk/tk"

type Window struct { *tk.Window }

func NewWindow() *Window {
	mw := &Window{}
	mw.Window = tk.RootWindow()
	... // 设置控件及布局
	return mw
}
func main() {
	win := func() {
		mw := NewWindow()
		mw.SetTitle("ATK Sample") // 主窗口标题
		mw.Center(nil)            // 主窗口居中
		mw.ShowNormal()           // 主窗口显示
		mw.ResizeN(300, 200)      // 主窗口尺寸
	}
	tk.MainLoop(win)
}

```

##### 基类控件

###### 主窗口控件

```
// WindowWidget接口，包括Widget接口
type WindowWidget interface {
	Widget
	SetGeometry(v Geometry) error
	Geometry() Geometry
	SetGeometryN(x int, y int, width int, height int) error
	GeometryN() (x int, y int, width int, height int)
}

// Widget接口
type Widget interface {
	Id() string, Info() *WidgetInfo, Type() WidgetType, TypeName() string
	Parent() Widget, Children() []Widget, IsValid() bool, Destroy() error,
    DestroyChildren() error
	
	// attribute
	NativeAttribute(key string) (value string), 
	SetNativeAttribute(key string, value string) error
	SetAttributes(attributes ...*WidgetAttr) error
	
	// event
	BindEvent(event string, fn func(*Event)) error, 
	BindKeyEvent(fn func(e *KeyEvent)) error
	BindKeyEventEx(fnPress func(e *KeyEvent), 
				   fnRelease func(e *KeyEvent)) error
	BindInfo() []string, ClearBind(event string) error
	
	// grab
	SetGrab() error, ReleaseGrab() error, IsGrab() bool
	
	// focus
	SetFocus() error, IsFocus() bool, Lower(below Widget) error, 
    Raise(above Widget) error
} // widget.go

type Window struct { BaseWidget }
// BaseWidget实现了Widget接口，Window实现了WindowWidget接口

func (w *Window) SetTitle(title string) error
func (w *Window) Title() string
w.SetAlpha(alpha float64) error; w.Alpha() float64
w.SetFullScreen(full bool) error; w.IsFullScreen() bool
w.SetTopmost(full bool) error; w.IsTopmost() bool
w.SetGeometryN(x int, y int, width int, height int) error; 
w.GeometryN() (x int, y int, width int, height int)
w.SetGeometry(v Geometry) error; w.Geometry() Geometry
w.MoveN(x int, y int) error; w.Move(pos Pos) error
w.SetPosN(x int, y int) error; SetPos(pos Pos) error
w.PosN() (x int, y int); w.Pos() Pos
w.ResizeN(width int, height int); w.Resize(sz Size) error
w.SetSizeN(width int, height int) error; w.SetSize(sz Size) error
w.SizeN() (width int, height int); w.Size() Size
w.SetWidth(width int) error; w.Width() (width int)
w.SetHeight(height int) error; w.Height() (height int)
w.SetNaturalSize() error;
w.SetResizable(enableWidth bool, enableHeight bool) error
w.IsResizable() (enableWidth bool, enableHeight bool)
w.Iconify() error; w.IsIconify() bool
w.ShowModal() error; w.EndModal() error; 
w.Wait() error; w.ShowFullScreen(); w.ShowMinimized() error
w.IsMinimized() bool; w.ShowNormal() error; w.Hide() error
w.SetVisible(b bool) error; w.IsVisible() bool
w.Deiconify() error;
w.SetMaximumSizeN(width int, height int)
w.SetMaximumSize(sz Size) error
w.MaximumSizeN() (int, int); w.MaximumSize() Size
w.SetMinimumSizeN(width int, height int) error
w.SetMinimumSize(sz Size) error
w.MinimumSizeN() (int, int); w.MinimumSize() Size
w.ScreenSizeN() (width int, height int); w.ScreenSize() Size
w.Center(parent WindowWidget) error;w.Center(nil) error
w.OnClose(fn func() (accept bool)) error

tk.RootWindow() *Window // 主窗口
tk.NewWindow(attributes ...*WidgetAttr) *Window // 用来创建非主窗口

```



###### Base控件

```go
type BaseWidget struct {
	id   string
	info *WidgetInfo
}

type WidgetInfo struct {
	Type      WidgetType // type WidgetType int
	TypeName  string
	IsTtk     bool
	MetaClass *MetaClass
}

type MetaClass struct {
	Command    string   // "tk::button"
	Class      string   // "Button"
	Attributes []string // "background","foreground",...
}

type MetaType struct {
	Type string    // "Button"
	Tk *MetaClass  // &MetaClass{"tk::button", "Button", []string{...}}
	Ttk *MetaClass // &MetaClass{"ttk::button","TButton",[]string{...}}

typeMetaMap = make(map[WidgetType]*MetaType)

func (w *BaseWidget) Id() string
func (w *BaseWidget) Info() *WidgetInfo
func (w *BaseWidget) Type() WidgetType
func (w *BaseWidget) TypeName() string

func (w *BaseWidget) Parent() Widget
func (w *BaseWidget) Children() []Widget
func (w *BaseWidget) Destroy()
func (w *BaseWidget) DestroyChildren()

w.NativeAttribute(key string) string
w.NativeAttributes(keys ...string) (attributes []NativeAttr)
	fmt.Println(btn.NativeAttributes("cursor","width"))
	[{cursor hand1} {width 42}]
	
w.SetNativeAttribute(key string, value string) // 设置属性
    
type NativeAttr struct { Key string; Value string } // NativeAttr属性
w.SetNativeAttributes(attributes ...NativeAttr) // 设置属性
	btn.SetNativeAttribute("cursor","hand1")
	btn.SetNativeAttributes(tk.NativeAttr{"width","42"},
	tk.NativeAttr{"text","按钮abc456"})

type WidgetAttr struct { key string; value interface{} } // WidgetAttr属性
w.SetAttributes(attributes ...*WidgetAttr) // 设置属性

tk.WidgetAttrInitUseTheme(use bool) *WidgetAttr// 是否使用ttk控件
tk.WidgetAttrFont(font Font) *WidgetAttr // 控件字体
tk.WidgetAttrWidth(width int) *WidgetAttr // 控件宽度
tk.WidgetAttrHeight(height int) *WidgetAttr // 控件高度[px/行数]
tk.WidgetAttrText(text string) *WidgetAttr // 控件文本
tk.WidgetAttrImage(image *Image) *WidgetAttr // 控件图像
tk.WidgetAttrReliefStyle(style ReliefStyle) *WidgetAttr // 控件风格
tk.WidgetAttrBorderWidth(width int) *WidgetAttr // 控件边界宽度
tk.WidgetAttrPadding(pad Pad) *WidgetAttr // 控件padding 
tk.WidgetAttrPaddingN(padx int, pady int) *WidgetAttr // 控件padding


w.BindEvent(event string, fn func(e *Event))
w.BindKeyEvent(fn func(e *KeyEvent))
w.BindKeyEventEx(fnPress func(e *KeyEvent), fnRelease func(e *KeyEvent))
w.ClearBind(event string)
w.SetGrab(), w.ReleaseGrab()


```



##### 常见控件

###### 普通按钮

```go
// 按钮
func NewButton(parent,text,attributes) *Button
parent:父控件; text:按钮文本; attributes:按钮其它属性
btn := NewButton(parent,text,attributes)

attributes例子: 
tk.ButtonAttrText(text string),tk.ButtonAttrWidth(width int),
tk.ButtonAttrImage(image *Image),tk.ButtonAttrCompound(compound Compound),
tk.ButtonAttrState(state State),tk.ButtonAttrTakeFocus(takefocus bool),
btn方法:
btn.SetText(text string),btn.Text()
btn.SetWidth(width int),btn.Width() // 单位:平均字符长度
btn.SetImage(image *Image), btn.Image()
btn.SetCompound(compound Compound), btn.Compound()
btn.SetPaddingN(padx int, pady int), btn.PaddingN() (int, int), 
btn.SetPadding(pad Pad), btn.Padding()
btn.SetState(state State), btn.State()
btn.SetTakeFocus(takefocus bool), btn.IsTakeFocus()
btn.OnCommand(fn func())

tk.Pad{ X, Y int }
tk.Compound: tk.CompoundNone, tk.CompoundTop, tk.CompoundBottom
	tk.CompoundLeft, tk.CompoundRight, tk.CompoundCenter
tk.State: tk.StateNormal,tk.StateActive,tk.StateDisable,tk.StateReadOnly

btn := tk.NewButton(mw,"Quit",tk.ButtonAttrState(2))
btn.SetState(tk.StateDisable)
btn.SetPadding(tk.Pad{10,10})
btnInfo := tk.NewLabel(mw,"Info")
fninfo := func(btn *tk.Button) { btnInfo.SetText(btn.Text()) }
btn.OnCommand(func() { fninfo(btn1) }) // OnCommand方法==command

img, _ := tk.LoadImage("./btn01.png") // 设置按钮图像
btn := tk.NewButton(mw,"Quit",tk.ButtonAttrImage(img),
					tk.ButtonAttrCompound(5))
btn.OnCommand(func() { tk.Quit() }) // command=quit, 退出主窗口

```

###### 标签

```
NewLabel(parent,text,attributes) *Label
parent:父控件; text:标签文本; attributes:标签其它属性
label := NewLabel(parent,text,attributes)

attributes例子:
tk.LabelAttrBackground(color string)
tk.LabelAttrForground(color string)
tk.LabelAttrBorderWidth(width int)
tk.LabelAttrReliefStyle(relief ReliefStyle)
tk.LabelAttrFont(font Font)
tk.LabelAttrAnchor(anchor Anchor)
tk.LabelAttrJustify(justify Justify)
tk.LabelAttrWrapLength(wraplength int)
tk.LabelAttrImage(image *Image)
tk.LabelAttrCompound(compound Compound)
tk.LabelAttrText(text string)
tk.LabelAttrWidth(width int)
tk.LabelAttrPadding(padding Pad)
tk.LabelAttrState(state State)
tk.LabelAttrTakeFocus(takefocus bool)

label方法:
label.SetBackground(color string), label.Background()
label.SetForground(color string), label.Forground()
label.SetBorderWidth(width int), label.BorderWidth()
label.SetReliefStyle(relief ReliefStyle), label.ReliefStyle()
label.SetFont(font Font), label.Font()
label.SetAnchor(anchor Anchor), label.Anchor()
label.SetJustify(justify Justify), label.Justify()
label.SetWrapLength(wraplength int), label.WrapLength()
label.SetImage(image *Image), label.Image()
label.SetCompound(compound Compound), label.Compound()
label.SetText(text string), label.Text()
label.SetWidth(width int), label.Width()
label.SetPaddingN(padx int, pady int), label.PaddingN() (int, int)
label.SetPadding(pad Pad), label.Padding()
label.SetState(state State), label.State()
label.SetTakeFocus(takefocus bool), label.IsTakeFocus()

tk.ReliefStyle: tk.ReliefStyleFlat, tk.ReliefStyleGroove
	tk.ReliefStyleRaised, tk.ReliefStyleRidge, tk.ReliefStyleSolid
	tk.ReliefStyleSunken
tk.Anchor: tk.AnchorCenter, tk.AnchorNorth, tk.AnchorEast, tk.AnchorSouth
	tk.AnchorWest, tk.AnchorNorthEast, tk.AnchorNorthWest
	tk.AnchorSouthEast, tk.AnchorSouthWest
tk.Justify: tk.JustifyCenter, tk.JustifyLeft, tk.JustifyRight

lbl := tk.NewLabel(mw, "Hello ATK 01",
				   tk.LabelAttrForground("blue"),
				   tk.LabelAttrBackground("red"),
				   tk.LabelAttrReliefStyle(5),
)

```

###### 字体

```
参考【font.go】
type Font interface {
	Id() string
	IsValid() bool
	String() string
	Description() string
	Family() string
	Size() int
	IsBold() bool
	IsItalic() bool
	IsUnderline() bool
	IsOverstrike() bool
}
【FontAttr】: [bold,italic,underline,overstrike]
tk.FontAttrBold(),tk.FontAttrItalic(),tk.FontAttrUnderline()
tk.FontAttrOverstrike()

BaseFont实现了Font接口, UserFont继承了BaseFont

func (w *BaseFont) MeasureTextWidth(text string) int
func (w *BaseFont) Clone() *UserFont
func (f *UserFont) Destroy() error // 删除字体
func NewUserFont(family,size,attributes) *UserFont
func NewUserFontFromClone(font Font) *UserFont

func (w *UserFont) SetFamily(family string) *UserFont 
func (w *UserFont) SetSize(size int) *UserFont
func (w *UserFont) SetBold(bold bool) *UserFont 
func (w *UserFont) SetItalic(italic bool) *UserFont 
func (w *UserFont) SetUnderline(underline bool) *UserFont
func (w *UserFont) SetOverstrike(overstrike bool) *UserFont

func FontFamilieList() []string // 返回所有可用字体
tk.FontFamilieList()

font := NewUserFont(family,size,attributes) // 创建一个font
family:string, size:int, attributes:【FontAttr】
font1 := tk.NewUserFont("", 36,tk.FontAttrItalic(),tk.FontAttrUnderline())
font2 := NewUserFontFromClone(font1) // 从已有字体进行复制创建

weidget.SetFont(font1)

```

###### 控件常见属性及类型

```
控件公共属性, attributes, 参考【widget_attr.go】:
tk.WidgetAttrInitUseTheme(use bool) // 设置控件是否使用ttk theme
tk.WidgetAttrFont(font Font)
tk.WidgetAttrWidth(width int)
tk.WidgetAttrHeight(height int)
tk.WidgetAttrText(text string)
tk.WidgetAttrImage(image *Image)
tk.WidgetAttrReliefStyle(style ReliefStyle)
tk.WidgetAttrBorderWidth(width int)
tk.WidgetAttrPadding(pad Pad)
tk.WidgetAttrPaddingN(padx int, pady int)

控件属性类型, 参考【misc.go】:
tk.Pos{ X, Y int }; tk.Pad{ X, Y int }; tk.Size{ Width, Height int }
tk.Geometry{ X, Y, Width, Height int }

tk.Orient: tk.Vertical, tk.Horizontal

tk.Justify: tk.JustifyCenter, tk.JustifyLeft, tk.JustifyRight

tk.Side: tk.SideLeft, tk.SideRight, tk.SideTop, tk.SideBottom

tk.BorderMode: 
	tk.BorderModeInside, tk.BorderModeOutside, tk.BorderModeIgnore

tk.Fill: tk.FillNone, tk.FillX, tk.FillY, tk.FillBoth

tk.ReliefStyle: ReliefStyleFlat, tk.ReliefStyleGroove
	tk.ReliefStyleRaised, tk.ReliefStyleRidge, tk.ReliefStyleSolid
	tk.ReliefStyleSunken

tk.Anchor: tk.AnchorCenter, tk.AnchorNorth, tk.AnchorEast, tk.AnchorSouth
	tk.AnchorWest, tk.AnchorNorthEast, tk.AnchorNorthWest
	tk.AnchorSouthEast, tk.AnchorSouthWest

tk.Sticky: tk.StickyCenter, tk.StickyN, tk.StickyS, tk.StickyE, tk.StickyW
	tk.StickyNS, tk.StickyEW, tk.StickyAll

tk.Direction: tk.DirectionBelow, tk.DirectionAbove, tk.DirectionLeft
	tk.DirectionRight

tk.Compound: tk.CompoundNone, tk.CompoundTop, tk.CompoundBottom
	tk.CompoundLeft, tk.CompoundRight, tk.CompoundCenter

tk.State: tk.StateNormal,tk.StateActive,tk.StateDisable,tk.StateReadOnly

tk.ListSelectMode: tk.ListSelectSingle, tk.ListSelectBrowse
	tk.ListSelectMultiple, tk.ListSelectExtended

tk.DisplyCursor: tk.DisplyCursorNone, tk.DisplyCursorHollow
	tk.DisplyCursorSolid

tk.LineWrapMode: tk.LineWrapNone, tk.LineWrapChar, tk.LineWrapWord

tk.TreeSelectMode: tk.TreeSelectExtended, tk.TreeSelectBrowse
	tk.TreeSelectNode
	
```

###### 控件布局-pack

```
mw := tk.RootWindow()
hbox1 := tk.NewHPackLayout(mw)
hbox2 := tk.NewHPackLayout(mw)
vbox := tk.NewVPackLayout(mw)
vbox.AddWidget(hbox1, tk.PackAttrExpand(false))
vbox.AddWidget(frm)
vbox.AddWidget(lbl2)


type PackLayout struct {
	*LayoutFrame
	side  Side
	pad   *Pad
	items []*LayoutItem
}

func NewPackLayout(parent Widget, side Side) *PackLayout
func NewHPackLayout(parent Widget) *PackLayout // side = SideLeft
func NewVPackLayout(parent Widget) *PackLayout // side = SideTop
*PackLayout方法:
hbox.SetSide(side Side), 
hbox.SetPadding(pad Pad), hbox.SetPaddingN(padx int, pady int)
hbox.SetBorderWidth(width int), hbox.BorderWidth()
hbox.Repack()

hbox.removeItem(widget Widget)
hbox.RemoveWidget(widget Widget)
hbox.indexOfWidget(widget Widget)

// 布局添加控件
hbox.AddWidget(widget Widget,attributes ...*LayoutAttr) 
hbox.AddWidgetEx(widget Widget, fill Fill, expand bool, anchor Anchor)
hbox.InsertWidgetEx(index int,widget,fill,expand,anchor)

attributes:【*LayoutAttr】【参考pack.go】
tk.PackAttrSide(side Side), tk.PackAttrFill(fill Fill)
tk.PackAttrSideLeft(), tk.PackAttrSideRight()
tk.PackAttrSideTop(), tk.PackAttrSideBottom()
tk.PackAttrPadx(padx int), tk.PackAttrPady(pady int)
tk.PackAttrIpadx(padx int), tk.PackAttrIpady(pady int)
tk.PackAttrAnchor(anchor Anchor), tkPackAttrExpand(b bool)
tk.PackAttrFillX(),tk.PackAttrFillY(),tk.PackAttrFillBoth()
tk.PackAttrFillNone()
tk.PackAttrBefore(w Widget), tk.PackAttrAfter(w Widget)

hbox.SetWidgetAttr(widget Widget, attributes ...*LayoutAttr)

hbox.InsertWidget(index int, widget Widget, attributes ...*LayoutAttr)
hbox.AddWidgets(widgets ...Widget)
hbox.AddWidgetList(widgets []Widget, attributes ...*LayoutAttr)

```

###### 控件布局-grid

```
mw := tk.RootWindow()
grid := NewGridLayout(mw)

type GridLayout struct {
	*LayoutFrame
	items []*LayoutItem
}

func NewGridLayout(parent Widget) *GridLayout
*GridLayout方法:
grid.AddWidgetEx(widget,row,column,rowspan,columnspan,sticky Sticky)
grid.AddWidget(widget Widget, attrs ...*LayoutAttr)
grid.AddWidgets(widgets ...Widget)
grid.AddWidgetList(widgets []Widget, attrs ...*LayoutAttr)
grid.RemoveWidget(widget Widget)
grid.Repack(), grid.SetBorderWidth(width int), grid.BorderWidth()
grid.SetRowAttr(row int, pad int, weight int, group string)
grid.SetColumnAttr(column int, pad int, weight int, group string)

attrs： 【*LayoutAttr】【参考grid.go】
tk.GridAttrColumn(n int), tk.GridAttrColumnSpan(n int)
tk.GridAttrRow(n int), tk.GridAttrRowSpan(n int)
GridAttrIpadx(padx int), GridAttrIpady(pady int)
GridAttrPadx(padx int), GridAttrPady(pady int)
GridAttrSticky(v Sticky), 

```

控件布局

```
mw := tk.RootWindow()
spacer := NewLayoutSpacer(mw,10,false)

type LayoutSpacer struct {
	BaseWidget
	space  int
	expand bool
}

func NewLayoutSpacer(parent Widget, space int, expand bool) *LayoutSpacer

*LayoutSpacer方法:
func (w *LayoutSpacer) SetSpace(space int) *LayoutSpacer
func (w *LayoutSpacer) Space() int
func (w *LayoutSpacer) SetExpand(expand bool) *LayoutSpacer
func (w *LayoutSpacer) IsExpand() bool
func (w *LayoutSpacer) SetWidth(width int) *LayoutSpacer
func (w *LayoutSpacer) Width() int
func (w *LayoutSpacer) SetHeight(height int) *LayoutSpacer
func (w *LayoutSpacer) Height() int

```

###### 事件

```
【basewidget.go】
func (w *BaseWidget) BindEvent(event string, fn func(e *Event))
func (w *BaseWidget) BindKeyEvent(fn func(e *KeyEvent))
func (w *BaseWidget) BindKeyEventEx(fnPress func(e *KeyEvent), fnRelease func(e *KeyEvent))
func (w *BaseWidget) ClearBind(event string)


mw := tk.RootWindow()
mw.BindKeyEvent(mw.OnKeyEvent)
mw.BindEvent(event.Motion, mw.OnMotion)
mw.BindEvent(event.ButtonPress, mw.OnButtonClick)
mw.BindEvent("<Double-ButtonPress>", mw.OnButtonDbclick)

func (w *Window) OnMotion(e *tk.Event) {
	w.posLabel.SetText(fmt.Sprintf("global: %v-%v\tpos: %3d-%3d",
    e.GlobalPosX, e.GlobalPosY, e.PosX, e.PosY))
}

func (w *Window) OnButtonClick(e *tk.Event) {
	w.posLabel.SetText(fmt.Sprintf("mouse click: %v\tpos: %v-%v",
    e.MouseButton, e.PosX, e.PosY))
}

func (w *Window) OnButtonDbclick(e *tk.Event) {
	w.posLabel.SetText(fmt.Sprintf("mouse double click: %v\tpos: %v-%v",
    e.MouseButton, e.PosX, e.PosY))
}
e.PosX, e.PosY【参考\tk\event.go】
type Event struct { ... }
type KeyEvent struct { *Event, KeyModifier KeyModifier }
e *tk.Event: 
e.Widget Widget, e.State string
e.PosX int, e.PosY int, e.GlobalPosX int, e.GlobalPosY int
e.KeyCode int, e.KeySym  string
e.KeyText string, e.KeyRune rune
e.Width  int, e.Height int

e *tk.KeyEvent:
e.KeyModifier

event.Motion, event.ButtonPress 【参考\event\event.go】

event.Activate = "<Activate>", event.Deactivate = "<Deactivate>"
MouseWheel = "<MouseWheel>", event.Motion = "<Motion>"
event.Configure = "<Configure>", event.Destroy = "<Destroy>"
event.Enter = "<Enter>", event.Leave = "<Leave>"

event.KeyPress = "<KeyPress>", event.KeyRelease = "<KeyRelease>"
event.ButtonPress = "<ButtonPress>", eventButtonRelease = "<ButtonRelease>"
event.DoubleButtonPress = "<Double-ButtonPress>"

VIRTUAL EVENTS
....

```

##### 其它按钮

###### CheckButton

```
type CheckButton struct {
	BaseWidget
	command *Command
}
func NewCheckButton(parent,text,attributes ...*WidgetAttr) *CheckButton

mw := tk.RootWindow()
ctn := tk.NewCheckButton(mw,"CheckBtn",)

*WidgetAttr:
tk.CheckButtonAttrText(text string)
tk.CheckButtonAttrWidth(width int)
tk.CheckButtonAttrImage(image *Image)
tk.CheckButtonAttrCompound(compound Compound)
tk.CheckButtonAttrPadding(padding Pad) 
tk.CheckButtonAttrState(state State) 
tk.CheckButtonAttrTakeFocus(takefocus bool)

CheckButton方法:
ctn.SetText(text string), ctn.Text()
ctn.SetWidth(width int), ctn.Width()
ctn.SetImage(image *Image), Image(), 
ctn.SetCompound(compound Compound), Compound()
ctn.SetPaddingN(padx int, pady int), PaddingN() (int, int)
ctn.SetPadding(pad Pad), Padding()
ctn.SetState(state State), State()
ctn.SetTakeFocus(takefocus bool), IsTakeFocus()
ctn.SetChecked(check bool), IsChecked()
ctn.OnCommand(fn func())

```

###### RadioButton

```
type RadioButton struct {
	BaseWidget
	command *Command
}
func NewRadioButton(parent,text,attributes ...*WidgetAttr) *RadioButton

mw := tk.RootWindow()
rtn := tk.NewRadioButton(mw,"CheckBtn",)

*WidgetAttr:
RadioButtonAttrText(text string)
RadioButtonAttrImage(image *Image)
RadioButtonAttrWidth(width int)
RadioButtonAttrCompound(compound Compound)
RadioButtonAttrPadding(padding Pad)
RadioButtonAttrState(state State)
RadioButtonAttrTakeFocus(takefocus bool)

RadioButton方法:
rtn.SetText(text string), ctn.Text()
rtn.SetWidth(width int), ctn.Width()
rtn.SetImage(image *Image), Image(), 
rtn.SetCompound(compound Compound), Compound()
rtn.SetPaddingN(padx int, pady int), PaddingN() (int, int)
rtn.SetPadding(pad Pad), Padding()
rtn.SetState(state State), State()
rtn.SetTakeFocus(takefocus bool), IsTakeFocus()
rtn.SetChecked(check bool), IsChecked()
rtn.OnCommand(fn func())

rtn使用【参考D:\谷歌下载\atk-master\atk_demo-master\button\button.go】
rtn.SetValue(value int)

```

##### 输入框控件

###### Entry

```
type Entry struct {
	BaseWidget
	xscrollcommand *CommandEx
}

func NewEntry(parent Widget, attributes ...*WidgetAttr) *Entry

mw := tk.RootWindow()
ent := tk.NewEntry(mw,)

attributes: *WidgetAttr【参考tk\entry.go】
tk.EntryAttrForeground(color string)
tk.EntryAttrBackground(color string)
tk.EntryAttrWidth(width int)
tk.EntryAttrFont(font Font)
tk.EntryAttrJustify(justify Justify)
tk.EntryAttrShow(show string)
tk.EntryAttrState(state State)
tk.EntryAttrTakeFocus(takefocus bool)
tk.EntryAttrExportSelection(export bool)

Entry方法:
ent.SetForeground(color string), Foreground()
ent.SetBackground(color string), Background()
ent.SetWidth(width int), Width()
ent.SetFont(font Font), Font()
ent.SetJustify(justify Justify), Justify()
ent.SetShow(show string), Show()
ent.SetState(state State), State()
ent.SetTakeFocus(takefocus bool), IsTakeFocus()
ent.SetExportSelection(export bool), IsExportSelection()
ent.SetText(text string), Text()
ent.SetXViewArgs(args []string), OnXScrollEx(fn func([]string) error)
ent.BindXScrollBar(bar *ScrollBar)
ent.SetCursorPosition(pos int) *Entry, CursorPosition()
ent.OnUpdate(fn func()), OnEditReturn(fn func())
ent.Copy(), Paste(), Cut(), Clear()
ent.Delete(index int), DeleteRange(start int, end int)
ent.TextLength(), Insert(index int, str string), Append(str string)
ent.ClearSelection(), HasSelectedText(), SelectedText() string
ent.SetSelection(start int, end int), SelectAll()
ent.SelectionStart() int, SelectionEnd() int

```

###### Text

```
type Text struct {
	BaseWidget
	xscrollcommand *CommandEx
	yscrollcommand *CommandEx
}

type TextEx struct {
	*ScrollLayout
	*Text
}

func NewText(parent Widget, attributes ...*WidgetAttr) *Text
func NewTextEx(parent Widget, attributs ...*WidgetAttr) *TextEx // 带滚动条

mw := tk.RootWindow()
txt := tk.NewText(mw,)
stxt := tk.NewTextEx(mw,)

attributes: *WidgetAttr【参考tk\text.go】
tk.TextAttrBackground(color string), TextAttrForeground(color string)
tk.TextAttrSelectBackground(color string)
tk.TextAttrSelectforeground(color string)
tk.TextAttrBorderWidth(width int), TextAttrFont(font Font)
tk.TextAttrHighlightBackground(color string)
TextAttrHighlightColor(color string)
TextAttrHighlightthickness(width int)
TextAttrInsertBackground(color string)
TextAttrInsertBorderWidth(width int)
TextAttrInsertOffTime(offtime int)
TextAttrInsertOnTime(ontime int)
TextAttrInsertWidth(width int)
TextAttrPadding(padding Pad)
TextAttrReliefStyle(relief ReliefStyle)
TextAttrSelectborderwidth(width int)
TextAttrInactiveSelectBackground(color string)
TextAttrTakeFocus(takefocus bool)
TextAttrAutoSeparatorsOnUndo(autoseparators bool)
TextAttrBlockCursor(blockcursor bool)
TextAttrStartLine(startline int), TextAttrEndLine(endline int)
TextAttrWidth(width int), TextAttrHeight(height int)
TextAttrInsertUnfocussed(style DisplyCursor)
TextAttrMaxUndo(maxundo int)
TextAttrLineAboveSpace(spacing int), TextAttrLineBelowSpace(spacing int) 
TextAttrLineWrapSpace(spacing int)
TextAttrLineWrap(wrap LineWrapMode)
TextAttrEnableUndo(undo bool)

Text方法:
SetBackground(color string), Background()
SetForeground(color string), Foreground()
SetBorderWidth(width int), BorderWidth()
SetFont(font Font), Font()
SetHighlightBackground(color string), HighlightBackground()
SetHighlightColor(color string), HighlightColor()
SetHighlightthickness(width int), Highlightthickness()
SetInsertBackground(color string), InsertBackground()
SetInsertBorderWidth(width int), InsertBorderWidth()
SetInsertOffTime(offtime int), InsertOffTime() 
SetInsertOnTime(ontime int), InsertOnTime()
SetInsertWidth(width int), InsertWidth()
SetPaddingN(padx int, pady int), PaddingN() (int, int) 
SetPadding(pad Pad), Padding()
SetReliefStyle(relief ReliefStyle), ReliefStyle()
SetSelectBackground(color string), SelectBackground()
SetSelectborderwidth(width int), Selectborderwidth()
SetSelectforeground(color string), Selectforeground()
SetInactiveSelectBackground(color string), InactiveSelectBackground()
SetTakeFocus(takefocus bool), IsTakeFocus()
SetAutoSeparatorsOnUndo(autoseparators bool), IsAutoSeparatorsOnUndo() 
SetBlockCursor(blockcursor bool), IsBlockCursor()
SetStartLine(startline int), StartLine()
SetEndLine(endline int), EndLine()
SetWidth(width int), Width(), SetHeight(height int), Height()
SetInsertUnfocussed(style DisplyCursor), InsertUnfocussed()
SetMaxUndo(maxundo int), MaxUndo()
SetLineAboveSpace(spacing int), LineAboveSpace()
SetLineWrapSpace(spacing int), LineWrapSpace()
SetLineBelowSpace(spacing int), LineBelowSpace() 
SetLineWrap(wrap LineWrapMode), LineWrap()
SetEnableUndo(undo bool), IsEnableUndo()
SetReadOnly(b bool), IsReadOnly()
TextLength(),LineCount(),PlainText() // 返回文本字符数,文本行数,文本内容:string
InsertText(pos int, text string), AppendText(text string), Clear()
Length() // 返回end索引
SetText(text string) // 删除原有文本并设置新文本
SetTabSize(size int)
SetXViewArgs(args []string), SetYViewArgs(args []string)
OnXScrollEx(fn func([]string) error), OnYScrollEx(fn func([]string) error)
BindXScrollBar(bar *ScrollBar), BindYScrollBar(bar *ScrollBar)

```

###### ComboBox

```
type ComboBox struct {
	BaseWidget
}

func NewComboBox(parent Widget, attributes ...*WidgetAttr) *ComboBox

mw := tk.RootWindow()
cobo := tk.NewComboBox(mw,)

attributes: *WidgetAttr【参考tk\combobox.go】
tk.ComboBoxAttrFont(font Font)
tk.ComboBoxAttrBackground(color string)
tk.ComboBoxAttrForground(color string)
tk.ComboBoxAttrJustify(justify Justify)
tk.ComboBoxAttrWidth(width int)
tk.ComboBoxAttrHeight(height int)
tk.ComboBoxAttrEcho(echo string)
tk.ComboBoxAttrState(state State)
tk.ComboBoxAttrTakeFocus(takefocus bool)
tk.ComboBoxAttrValues(values []string)

ComboBox方法:
SetFont(font Font), Font()
SetBackground(color string), Background()
SetForground(color string), Forground()
SetJustify(justify Justify), Justify() 
SetWidth(width int), Width()
SetHeight(height int), Height()
SetEcho(echo string), Echo()
SetState(state State), State()
SetTakeFocus(takefocus bool), IsTakeFocus()
SetValues(values []string), Values()
OnSelected(fn func())
OnEditReturn(fn func())
SetCurrentText(text string), CurrentText()
SetCurrentIndex(index int), CurrentIndex()


```

###### ListBox

```
type ListBox struct {
	BaseWidget
	xscrollcommand *CommandEx
	yscrollcommand *CommandEx
}

type ListBoxEx struct {
	*ScrollLayout
	*ListBox
}

func NewListBox(parent Widget, attributes ...*WidgetAttr) *ListBox
func NewListBoxEx(parent, attributs ...*WidgetAttr) *ListBoxEx // 带滚动条

mw := tk.RootWindow()
libo := tk.NewListBox(mw,)
slibo := tk.NewListBoxEx(mw,)

attributes: *WidgetAttr【参考tk\listbox.go】
tk.ListBoxAttrBackground(color string)
tk.ListBoxAttrForground(color string)
ListBoxAttrBorderWidth(width int)
ListBoxAttrReliefStyle(relief ReliefStyle)
ListBoxAttrFont(font Font)
ListBoxAttrJustify(justify Justify)
ListBoxAttrWidth(width int)
ListBoxAttrHeight(height int)
ListBoxAttrPadding(padding Pad)
ListBoxAttrState(state State)
ListBoxAttrSelectMode(mode ListSelectMode)
ListBoxAttrTakeFocus(takefocus bool)

ListBox方法:
SetBackground(color string), Background()
SetForground(color string), Forground()
SetBorderWidth(width int), BorderWidth()
SetReliefStyle(relief ReliefStyle), ReliefStyle()
SetFont(font Font), Font()
SetJustify(justify Justify), Justify()
SetWidth(width int), Width()
SetHeight(height int), Height()
SetPaddingN(padx int, pady int), PaddingN() (int, int)
SetPadding(pad Pad), Padding()
SetState(state State), State()
SetSelectMode(mode ListSelectMode), SelectMode() 
SetTakeFocus(takefocus bool), IsTakeFocus()
SetItems(items []string), Items(), ItemCount()
InsertItem(index int, item string), AppendItem(index int, item string)
AppendItems(items []string)
SetItemText(index int, item string), ItemText(index int)
RemoveItem(index int), RemoveItemRange(start int, end int)
SetSelectionRange(start int, end int), SelectedIndexs(), SelectedItems()
ClearSelection(), 
OnSelectionChanged(fn func())
SetXViewArgs(args []string), SetYViewArgs(args []string)
OnXScrollEx(fn func([]string), OnYScrollEx(fn func([]string) error)
BindXScrollBar(bar *ScrollBar), BindYScrollBar(bar *ScrollBar)

```

###### Scale

```
type Scale struct {
	BaseWidget
	command *Command
}

func NewScale(parent, orient Orient, attributes ...*WidgetAttr) *Scale

mw := tk.RootWindow()
sca := tk.NewScale(mw,)

attributes: *WidgetAttr【参考tk\scale.go】
tk.ScaleAttrOrient(orient Orient)
ScaleAttrTakeFocus(takefocus bool)
ScaleAttrFrom(from float64)
ScaleAttrTo(to float64)
ScaleAttrValue(value float64)
ScaleAttrLength(length int)

Scale方法:
SetOrient(orient Orient), Orient()
SetTakeFocus(takefocus bool), IsTakeFocus()
SetFrom(from float64), From(), SetTo(to float64), To()
SetRange(from, to float64), Range() (float64, float64)
SetValue(value float64), Value()
SetLength(length int), Length()
OnCommand(fn func())

```

###### SpinBox

```
type SpinBox struct {
	BaseWidget
	command        *Command
	xscrollcommand *CommandEx
}

func NewSpinBox(parent Widget, attributes ...*WidgetAttr) *SpinBox

mw := tk.RootWindow()
ent := tk.NewSpinBox(mw,)

attributes: *WidgetAttr【参考tk\spinbox.go】
tk.SpinBoxAttrTakeFocus(takefocus bool)
tk.SpinBoxAttrFrom(from float64), SpinBoxAttrTo(to float64)
tk.SpinBoxAttrIncrement(increment float64)
tk.SpinBoxAttrWrap(wrap bool)
tk.SpinBoxAttrTextValues(values []string)

SpinBox方法:
SetTakeFocus(takefocus bool), IsTakeFocus()
SetFrom(from float64), From(), SetTo(to float64), To()
SetRange(from, to float64), Range() (float64, float64)
SetValue(value float64), Value()
SetIncrement(increment float64), Increment()
SetWrap(wrap bool), IsWrap()
SetTextValues(values []string), TextValues()
SetTextValue(value string), TextValue()
OnCommand(fn func())
OnXScrollEx(fn func([]string) error)
OnEditReturn(fn func())

Entry() *Entry // 返回spinbox控件中的entry控件
spin := tk.NewSpinBox(w)
spin.SetRange(0, 100)
spin.SetValue(0)
spin.Entry().SetWidth(5)

```

##### 容器控件

###### Frame

```
type Frame struct {
	BaseWidget
}

func NewFrame(parent Widget, attributes ...*WidgetAttr) *Frame

mw := tk.RootWindow()
frame := tk.NewFrame(mw,)

attributes: *WidgetAttr【参考tk\frame.go】
tk.FrameAttrBorderWidth(width int)
tk.FrameAttrReliefStyle(relief ReliefStyle)
tk.FrameAttrWidth(width int)
tk.FrameAttrHeight(height int)
tk.FrameAttrPadding(pad Pad)
tk.FrameAttrTakeFocus(takefocus bool)

Frame方法:
SetBorderWidth(width int), BorderWidth()
SetReliefStyle(relief ReliefStyle), ReliefStyle()
SetWidth(width int), Width(), SetHeight(height int), Height()
SetPaddingN(padx int, pady int), PaddingN() (int, int)
SetPadding(pad Pad), Padding()
SetTakeFocus(takefocus bool), IsTakeFocus()

```

###### LabelFrame

```
type LabelFrame struct {
	BaseWidget
}

func NewLabelFrame(parent, attributes ...*WidgetAttr) *LabelFrame

mw := tk.RootWindow()
labframe := tk.NewLabelFrame(mw,)

attributes: *WidgetAttr【参考tk\labelframe.go】
tk.LabelFrameAttrLabelText(text string)
tk.LabelFrameAttrLabelAnchor(anchor Anchor)
tk.LabelFrameAttrBorderWidth(width int)
tk.LabelFrameAttrReliefStyle(relief ReliefStyle)
tk.LabelFrameAttrWidth(width int)
tk.LabelFrameAttrHeight(height int)
tk.LabelFrameAttrPadding(pad Pad)
tk.LabelFrameAttrTakeFocus(takefocus bool)

LabelFrame方法:
SetLabelText(text string), LabelText()
SetLabelAnchor(anchor Anchor), LabelAnchor()
SetBorderWidth(width int), BorderWidth()
SetReliefStyle(relief ReliefStyle), ReliefStyle()
SetWidth(width int), Width(), SetHeight(height int), Height()
SetPaddingN(padx int, pady int), PaddingN() (int, int)
SetPadding(pad Pad), Padding()
SetTakeFocus(takefocus bool), IsTakeFocus()

```

###### Notebook

```
type Notebook struct {
	BaseWidget
}

func NewNotebook(parent Widget, attributes ...*WidgetAttr) *Notebook

mw := tk.RootWindow()
notb := tk.NewNotebook(mw,)

attributes: *WidgetAttr【参考tk\notebook.go】
tk.NotebookAttrWidth(width int)
tk.NotebookAttrHeight(height int)
tk.NotebookAttrTakeFocus(takefocus bool)
tk.NotebookAttrPadding(padding Pad)

Notebook方法:
SetWidth(width int), Width()
SetHeight(height int), Height()
SetTakeFocus(takefocus bool), IsTakeFocus()
SetPaddingN(padx int, pady int), PaddingN() (int, int)
SetPadding(pad Pad), Padding()
AddTab(widget Widget, text string, attributes ...*WidgetAttr)
InsertTab(pos int, widget, text string, attributes ...*WidgetAttr)
SetTab(widget Widget, text string, attributes ...*WidgetAttr)
    *WidgetAttr： 
    tk.TabAttrState(state State), TabAttrSticky(sticky Sticky)
    tk.TabAttrPadding(padding Pad), TabAttrText(text string)
    tk.TabAttrImage(image *Image), TabAttrCompound(compound Compound)
RemoveTab(widget Widget)
SetCurrentTab(widget Widget), CurrentTab()
CurrentTabIndex(), TabCount(), TabIndex(widget Widget)

```

##### 高级控件

###### Canvas

```
type Canvas struct {
	BaseWidget
	xscrollcommand *CommandEx
	yscrollcommand *CommandEx
}

func NewCanvas(parent Widget, attributes ...*WidgetAttr) *Canvas

mw := tk.RootWindow()
canv := tk.NewCanvas(mw,)

attributes: *WidgetAttr【参考tk\canvas.go】
tk.CanvasAttrBackground(color string)
tk.CanvasAttrBorderWidth(width int)
tk.CanvasAttrHighlightBackground(color string)
tk.CanvasAttrHighlightColor(color string)
tk.CanvasAttrHighlightthickness(width int)
tk.CanvasAttrInsertBackground(color string)
tk.CanvasAttrInsertBorderWidth(width int)
tk.CanvasAttrInsertOffTime(offtime int)
tk.CanvasAttrInsertOnTime(ontime int)
tk.CanvasAttrInsertWidth(width int)
tk.CanvasAttrReliefStyle(relief ReliefStyle)
tk.CanvasAttrSelectBackground(color string)
tk.CanvasAttrSelectborderwidth(width int)
tk.CanvasAttrSelectforeground(color string)
tk.CanvasAttrTakeFocus(takefocus bool)
tk.CanvasAttrCloseEnough(closeenough float64)
tk.CanvasAttrConfine(confine bool)
tk.CanvasAttrWidth(width int), CanvasAttrHeight(height int)
tk.CanvasAttrState(state State)
tk.CanvasAttrXScrollIncrement(value int)
tk.CanvasAttrYScrollIncrement(value int)

Canvas方法:
SetBackground(color string), Background()
SetBorderWidth(width int), BorderWidth()
SetHighlightBackground(color string), HighlightBackground()
SetHighlightColor(color string), HighlightColor()
SetHighlightthickness(width int), Highlightthickness()
SetInsertBackground(color string), InsertBackground()
SetInsertBorderWidth(width int), InsertBorderWidth()
SetInsertOffTime(offtime int), InsertOffTime()
SetInsertOnTime(ontime int), InsertOnTime()
SetInsertWidth(width int), InsertWidth()
SetReliefStyle(relief ReliefStyle), ReliefStyle()
SetSelectBackground(color string), SelectBackground()
SetSelectborderwidth(width int), Selectborderwidth()
SetSelectforeground(color string), Selectforeground()
SetTakeFocus(takefocus bool), IsTakeFocus()
SetCloseEnough(closeenough float64), CloseEnough()
SetConfine(confine bool), IsConfine()
SetWidth(width int), Width(), SetHeight(height int), Height()
SetState(state State), State()
SetXScrollIncrement(value int), XScrollIncrement(), 
SetYScrollIncrement(value int), YScrollIncrement()

```

###### TreeView

```
type TreeView struct {
	BaseWidget
	xscrollcommand *CommandEx
	yscrollcommand *CommandEx
}

type TreeViewEx struct {
	*ScrollLayout
	*TreeView
}

func NewTreeView(parent Widget, attributes ...*WidgetAttr) *TreeView
func NewTreeViewEx(parent,attributs ...*WidgetAttr) *TreeViewEx // 带滚动条


mw := tk.RootWindow()
treev := tk.NewTreeView(mw,)

attributes: *WidgetAttr【参考tk\treeview.go】
tk.TreeViewAttrTakeFocus(takefocus bool)
TreeViewAttrHeight(row int)
TreeViewAttrPadding(padding Pad)
TreeViewAttrTreeSelectMode(mode TreeSelectMode)

TreeView方法:
SetTakeFocus(takefocus bool), IsTakeFocus()
SetHeight(row int), Height()
SetPaddingN(padx int, pady int), PaddingN() (int, int)
SetPadding(pad Pad), Padding()
SetTreeSelectMode(mode TreeSelectMode), TreeSelectMode()
SetHeaderHidden(hide bool), IsHeaderHidden()
SetColumnCount(columns int), ColumnCount()
SetHeaderLabels(labels []string)
SetHeaderLabel(column int, label string), HeaderLabel(column int)
SetHeaderImage(column int, img *Image), HeaderImage(column int)
SetHeaderAnchor(column int, anchor Anchor), HeaderAnchor(column int)
SetColumnWidth(column int, width int), ColumnWidth(column int)
SetColumnMinimumWidth(column int, width int)
ColumnMinimumWidth(column int)
SetColumnAnchor(column int, anchor Anchor), ColumnAnchor(column int)
SetColumnStretch(column int, stretch bool), ColumnStretch(column int)
IsValidItem(item *TreeItem), RootItem() *TreeItem
ToplevelItems() []*TreeItem
InsertItem(parent *TreeItem, index, text, values []string) *TreeItem
DeleteItem(item *TreeItem), DeleteAllItems()
MoveItem(item *TreeItem, parent *TreeItem, index int)
ScrollTo(item *TreeItem)
SetCurrentIndex(item *TreeItem), CurrentIndex() *TreeItem
SelectionList() (lst []*TreeItem)
SetSelections(items ...*TreeItem)
RemoveSelections(items ...*TreeItem)
AddSelections(items ...*TreeItem)
ToggleSelections(items ...*TreeItem)
SetSelectionList(items []*TreeItem)
RemoveSelectionList(items []*TreeItem)
AddSelectionList(items []*TreeItem)
ToggleSelectionList(items []*TreeItem)
ExpandAll(), CollepseAll()
Expand(item *TreeItem), Collapse(item *TreeItem)
SetExpanded(item *TreeItem, expand bool), IsExpanded(item *TreeItem)
SetFocusItem(item *TreeItem), FocusItem() *TreeItem
OnSelectionChanged(fn func())
OnItemExpanded(fn func())
OnItemCollapsed(fn func())
OnDoubleClickedItem(fn func(item *TreeItem))
ItemAt(x int, y int) *TreeItem
SetXViewArgs(args []string)
SetYViewArgs(args []string)
OnXScrollEx(fn func([]string) error), OnYScrollEx(fn func([]string) error)
BindXScrollBar(bar *ScrollBar)
BindYScrollBar(bar *ScrollBar)

```

TreeItem

```
type TreeItem struct {
	tree *TreeView
	id   string
}

mw := tk.RootWindow()
treev := tk.NewTreeView(mw,)
item := tk.TreeItem{ treev,"id" }

TreeItem方法:
Id(), IsValid()
InsertItem(index int, text string, values []string) *TreeItem
Index(), IsRoot(), Parent() *TreeItem, Children() (lst []*TreeItem)
Next() *TreeItem, Prev() *TreeItem
SetExpanded(expand bool), IsExpanded()
ExpandAll(), CollapseAll(), Expand(), Collapse()
SetText(text string), Text()
SetValues(values []string), Values() []string
SetImage(img *Image), Image() *Image
SetColumnText(column int, text string), ColumnText(column int)

```



##### 菜单控件

###### Menu

```
type Menu struct {
	BaseWidget
}

func NewMenu(parent Widget, attributes ...*WidgetAttr) *Menu

mw := tk.RootWindow()
menu := tk.NewMenu(mw,)

attributes: *WidgetAttr【参考tk\menu.go】
tk.MenuAttrFont(font Font)
MenuAttrActiveBackground(color string)
MenuAttrActiveForground(color string)
MenuAttrBackground(color string)
MenuAttrForground(color string)
MenuAttrSelectColor(color string)
MenuAttrDisabledForground(color string)
MenuAttrActiveBorderWidth(width int)
MenuAttrBorderWidth(width int)
MenuAttrReliefStyle(relief ReliefStyle)
MenuAttrTearoffTitle(title string)
MenuAttrTearoff(tearoff bool)
MenuAttrTakeFocus(takefocus bool)

Menu方法:
SetFont(font Font), Font()
SetActiveBackground(color string), ActiveBackground()
SetActiveForground(color string), ActiveForground()
SetBackground(color string), Background()
SetForground(color string), Forground()
SetSelectColor(color string), SelectColor()
SetDisabledForground(color string), DisabledForground()
SetActiveBorderWidth(width int), ActiveBorderWidth()
SetBorderWidth(width int), BorderWidth()
SetReliefStyle(relief ReliefStyle), ReliefStyle()
SetTearoffTitle(title string), TearoffTitle()
SetTearoff(tearoff bool), IsTearoff()
SetTakeFocus(takefocus bool), IsTakeFocus()
AddSubMenu(label string, sub *Menu)
AddNewSubMenu(label string, attributes ...*WidgetAttr) *Menu
InsertSubMenu(index int, label string, sub *Menu)
InsertNewSubMenu(index, label, attributes ...*WidgetAttr) *Menu
AddAction(act *Action), InsertAction(index int, act *Action)
AddActions(actions []*Action)
AddSeparator(), InsertSeparator(index int)
SetMenuTearoff(enable bool)
PopupMenu(menu *Menu, xpos int, ypos int)

```

###### MenuButton

```
type MenuButton struct {
	BaseWidget
}

func NewMenuButton(parent,text,attributes ...*WidgetAttr) *MenuButton

mw := tk.RootWindow()
menubtn := tk.NewMenuButton(mw,)

attributes: *WidgetAttr【参考tk\menubutton.go】
tk.MenuButtonAttrText(text string)
tk.MenuButtonAttrWidth(width int)
tk.MenuButtonAttrImage(image *Image)
tk.MenuButtonAttrCompound(compound Compound)
tk.MenuButtonAttrPadding(padding Pad)
tk.MenuButtonAttrState(state State)
tk.MenuButtonAttrTakeFocus(takefocus bool)
tk.MenuButtonAttrDirection(direction Direction)
tk.MenuButtonAttrMenu(menu *Menu)

MenuButton方法:
SetText(text string), Text()
SetImage(image *Image), Image()
SetWidth(width int), Width()
SetCompound(compound Compound), Compound()
SetPaddingN(padx int, pady int), PaddingN() (int, int)
SetPadding(pad Pad), Padding()
SetState(state State), State(), SetTakeFocus(takefocus bool), IsTakeFocus()
SetDirection(direction Direction), Direction()
SetMenu(menu *Menu), Menu()

```



##### 其它控件

###### Separator

```
type Separator struct {
	BaseWidget
}

func NewSeparator(parent,orient,attributes ...*WidgetAttr) *Separator

mw := tk.RootWindow()
sep := tk.NewSeparator(mw,)

attributes: *WidgetAttr【参考tk\separator.go】
tk.SeparatorAttrOrient(orient Orient)
tk.SeparatorAttrTakeFocus(takefocus bool)

Separator方法:
SetOrient(orient Orient), Orient()
SetTakeFocus(takefocus bool), IsTakeFocus()

```

###### Paned

```
type Paned struct {
	BaseWidget
}

func NewPaned(parent,orient Orient,attributes ...*WidgetAttr) *Paned

mw := tk.RootWindow()
paned := tk.NewPaned(mw,)

attributes: *WidgetAttr【参考tk\paned.go】
tk.PanedAttrWidth(width int)
tk.PanedAttrHeight(height int)

Paned方法:
SetWidth(width int), Width()
SetHeight(height int), Height()
AddWidget(widget Widget, weight int)
InsertWidget(pane int, widget Widget, weight int)
SetPane(pane int, weight int)
RemovePane(pane int)

```

###### ProgressBar

```
type ProgressBar struct {
	BaseWidget
}

func NewProgressBar(parent,orient,attributes ...*WidgetAttr) *ProgressBar

mw := tk.RootWindow()
probar := tk.NewProgressBar(mw,)

attributes: *WidgetAttr【参考tk\progressbar.go】
tk.ProgressBarAttrOrient(orient Orient)
tk.ProgressBarAttrTakeFocus(takefocus bool)
tk.ProgressBarAttrLength(length int)
tk.ProgressBarAttrMaximum(maximum float64)
tk.ProgressBarAttrValue(value float64)

ProgressBar方法:
SetOrient(orient Orient), Orient()
SetTakeFocus(takefocus bool), IsTakeFocus()
SetLength(length int), Length()
SetMaximum(maximum float64), Maximum()
SetValue(value float64), Value(), Phase()
SetDeterminateMode(b bool), IsDeterminateMode()
Start(), StartEx(ms int), Stop(), Pause()


```

scrollbar.go

##### 2 theme主题

```go
btn := tk.NewButton(mw,"Quit",
					tk.WidgetAttrWidth(20),
					tk.WidgetAttrInitUseTheme(false))
// tk.WidgetAttrInitUseTheme(false)	使用tk::button, 默认true使用ttk::button	

fmt.Println(tk.TtkTheme.IsTtk(),
			tk.TtkTheme.ThemeIdList(),tk.TtkTheme.ThemeId())
// true [xpnative clam alt classic default winnative vista] vista

tk.TtkTheme.SetThemeId("alt") // 设置ttk主题
// true [xpnative clam alt classic default winnative vista] alt
fmt.Println(tk.TtkTheme.IsTtk(),
			tk.TtkTheme.ThemeIdList(),tk.TtkTheme.ThemeId())
// true [xpnative clam alt classic default winnative vista] vista

// ttk::button .b; winfo class .b  // ==> TButton
// ttk::style configure style ?-option ?value option value...? ?
// ttk::style map style ?-option { statespec value... }?
// ttk::button .b -text "Hello" -style "Fun.TButton"
// ttk::style configure Emergency.TButton -foreground red -padding 10
// ttk::style configure TButton -font "helvetica 24"
// ttk::style lookup TButton -font  // [---> helvetica 24]
// ttk::style map TButton \
// -background [list disabled #d9d9d9 active #ececec] \
// -foreground [list disabled #a3a3a3] \
// -relief [list {pressed !disabled} sunken]

// ttk::style map 可用state
active/!active 		 The mouse is currently within the widget.
alternate/!alternate This state is reserved for application use.
disabled/!disabled 	 The widget will not respond to user actions.
focus/!focus 		 The widget currently has focus.
pressed/!pressed 	 The widget is currently being pressed 
					 (e.g., a button that is being clicked).
readonly/!readonly   The widget will not allow any user actions to 
					 change its current value. For example, a 
					 read-only Entry widget 
					 will not allow editing of its content.
selected/!selected   The widget is selected. Examples are checkbuttons and 
					 radiobuttons that are in the “on” state. 

// % ttk::style theme styles vista
// Label TScale Horizontal.TScale TMenubutton TLabelframe.Label
// Vertical.TProgressbar TEntry TRadiobutton TButton Heading
// Toolbutton TNotebook.Tab ComboboxPopdownFrame Treeview 
// Vertical.TScale TCombobox TNotebook TProgressbar 
// Horizontal.TProgressbar . TCheckbutton Item TSpinbox Tab
// % ttk::style theme styles clam
// TMenubutton TEntry TLabelframe Vertical.Sash 
// TRadiobutton Heading TButton TNotebook.Tab Toolbutton
// Treeview ComboboxPopdownFrame TCombobox TProgressbar 
// TCheckbutton Tab TSpinbox Horizontal.Sash Sash

```

##### 3 ATK补充

```
// 设置控件属性，获取控件属性
// 使用控件方法，补充控件方法

// 异步更新

// 程序图标



```

