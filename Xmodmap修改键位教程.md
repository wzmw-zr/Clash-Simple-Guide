# Xmodmap修改键位教程

之所以打算使用Xmodmap来修改键位，是因为使用了一段时间vim之后，虽然一开始我就使用Xmodmap来修改了esc和caps lock键位，但是我实际上是不会使用Xmodmap的，因为最近发现shift键非常的烦人，所以打算再次更改键位，因此准备仔细研究Xmodmap的用法。

使用Xmodmap进行按键映射，其原理是我们按下键盘上的一个键，对应输入一个keycode，因此，我们只需要修改keycode对应的映射就可以达到修改键位的效果。

首先，我们可以通过`xmodmap -pke`来查看当前所有keycode及其映射，下面我的方法就是根据`xmodmap -pke`的结果来进行相应的keycode映射的修改。

```
keycode 66 = Escape NoSymbol Escape
keycode 9 = Caps_Lock Nonsymbol Caps_Lock
keycode 64 = Shift_L NoSymbol Shift_L
keycode 50 = Alt_L NoSymbol Alt_L

clear Shift
clear Lock
clear Mod1
add Shift = Shift_L Shift_R
add Lock = Caps_Lock
add Mod1 = Alt_L Alt_R Meta_L
```

此外，对于出现在`xmodmap -pm`中的键位，我们需要对其进行更新，就是先clear，之后在重新添加进去。

至此，Xmodmap的修改键位的流程就完成了。