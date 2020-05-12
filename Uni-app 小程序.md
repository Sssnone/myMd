# Uni-app 小程序

* swiper显示多个item的时候。设置了next和previous margin 。对于第一个和最后一个也会有额外的margin。需要消除 的话，需要动态去设置滑到最后一个时候的next和previous。
* 使用小程序input组件时，若使用@input去监听，或者用v-model（实质也是写了一个@input事件），会发生抖动现象，即在快速输入的时候，光标会不及时跟着输入的内容走，甚至在延时之后才出现。需要去除这种抖动。可以使用form表单，不给input组件绑定@input事件。最终用bindSubmit的事件去获取值