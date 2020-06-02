# Uni-app 小程序

* swiper显示多个item的时候。设置了next和previous margin 。对于第一个和最后一个也会有额外的margin。需要消除 的话，需要动态去设置滑到最后一个时候的next和previous。
* 使用小程序input组件时，若使用@input去监听，或者用v-model（实质也是写了一个@input事件），会发生抖动现象，即在快速输入的时候，光标会不及时跟着输入的内容走，甚至在延时之后才出现。需要去除这种抖动。可以使用form表单，不给input组件绑定@input事件。最终用bindSubmit的事件去获取值
* fix弹窗的滑动穿透问题可以添加 touch-action: none;来解决
* 行内滚动 ，超过一屏，需要设置元素为inline-block ，父元素white-space: no wrap；
* h5中。图片的缩放可以通过max-width 设置宽或者高来设置缩放的基准值。小程序里是通过mode来设置。

* 小程序。需要设置placeHolder 的样式的话，需要放在顶层css文件中。

