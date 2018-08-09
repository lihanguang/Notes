### AppBarLayout的5种ScrollFlag
#### scroll
Child View 伴随着滚动事件而滚出或滚进屏幕。注意两点：第一点，如果使用了其他值，必定要使用这个值才能起作用；第二点：如果在这个child View前面的任何其他Child View没有设置这个值，那么这个Child View的设置将失去作用。

#### enterAlways
快速返回模式。其实就是向下滚动时Scrolling View和Child View之间的滚动优先级问题。对比scroll和scroll | enterAlways设置，发生向下滚动事件时，前者优先滚动Scrolling View，后者优先滚动Child View，当优先滚动的一方已经全部滚进屏幕之后，另一方才开始滚动。


#### enterAlwaysCollapsed
enterAlways的附加值。这里涉及到Child View的高度和最小高度，向下滚动时，Child View先向下滚动最小高度值，然后Scrolling View开始滚动，到达边界时，Child View再向下滚动，直至显示完全。

#### exitUntilCollapsed
这里也涉及到最小高度。发生向上滚动事件时，Child View向上滚动退出直至最小高度，然后Scrolling View开始滚动。也就是，Child View不会完全退出屏幕。

#### snap
简单理解，就是Child View滚动比例的一个吸附效果。也就是说，Child View不会存在局部显示的情况，滚动Child View的部分高度，当我们松开手指时，Child View要么向上全部滚出屏幕，要么向下全部滚进屏幕，有点类似ViewPager的左右滑动。

