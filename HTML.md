# HTML

* 事件委托的作用：

  * 一是用于像要给ul下面的多个li进行事件绑定。可以直接绑定在ul上。
  * 同样还是已经给ul下的li添加事件，但是在新增li的时候，事件需要重新绑定，但是如果绑定在父元素上，就可以不需要重新绑定。
* bind&on各种绑定事件的区别、jq对象与原生对象间的关联转换
* 微信计算高度的时候会把上面的状态栏也计算进去。
* pointer-events: none;可以用于阻止鼠标事件，包括点击事件。
* 立即执行函数（IIFE）

```javascript
立即执行函数写法

使用”立即执行函数”（Immediately-Invoked Function Expression，IIFE），可以达到不暴露私有成员的目的

var module = (function() {
    var _count = 0;
    var m1 = function() {
        alert(_count)
    }
    var m2 = function() {
        alert(_count + 1)
    }
    
    return {
        m1: m1,
        m2: m2
    }
})()
使用上面的写法，外部代码无法读取内部的_count变量。

console.info(module._count); //undefined
module就是Javascript模块的基本写法。
```

*  ```
   scrollIntoView()    出现在可视区内
   ```

*  出现遮罩禁止页面的方式

     ```	

     ```
   1. 正常浏览器只需要给body加上 overflow:hidden; height:100%
        但是在手机端还需要加上 position:fixed;
   2. 
      ```

      ```

*  页面导航栏 向下滑隐藏，向上滑时fixed  //  主要是判断方向 利用scrollTop

   ```javascript
      $(document).ready(function(){  
          var nowScorllTop=0,lastScorllTop=0;  
        
          $(window).scroll(function(e){  
                  nowScorllTop = $(this).scrollTop();  
                    
                  if(lastScorllTop<=nowScorllTop){//down
                     $("nav").css("display","none")
                  }  
                    
                  else{//up
                      $("nav").css("display","block")  
                  }  
                  lastScorllTop=nowScorllTop         
          });  
      }); 
   ```

*  垂直居中

   ```javascript
   1. top:50%; margin-top:-height/2;
   2. top:50%;transform: translateY(-height/2);
   3. 弹性布局 即flex布局给父元素加上这些样式 :
             align-items: center; /*定义body的元素垂直居中*/   
             justify-content: center; /*定义body的里的元素水平居中*/
   ```

*  移动端调试

     ```
     1.vconsole.js   这个js文件加载之后，可以将控制台中的任意console.log给打印出来，帮忙调试
     2.https://github.com/nupthale/weinre、
     3.https://github.com/wuchangming/spy-debugger
     4.安卓就用chrome   apple就要用mac的Safari
     ```

*  绑定this的方法

     ```

     ```
   1. es6下面 this.handleClick = ::this.handleClick
   2. this.handleClick.bind(this)
   3. ()=>{}  箭头函数
      ```

      ```

*  disabled加在input属性上时，无法写，同时通过name属性提交也会无法提交。而替换成readonly属性之后，虽然也无法写，但是可以提交。

*  回退到上一步的操作

   ```javascript
    //history.back(-1):直接返回当前页的上一页，数据全部消息，是个新页面

    //history.Go(-1):也是返回当前页的上一页，不过表单里的数据全部还在

    //history.back(0) 刷新 history.back(1) 前进 history.back(-1) 后退
   ```

    ​

*  移动端下focus()在ios和安卓的差异

   ``````javascript
   ios下focus()如果发生在异步操作下面，不起效果，例如 setTimeout(function(){
     document.querySelector("#btn").focus()
   },0)，将不起效果。而在安卓下面是正常的可以起效果
   ``````

*  移动端的局部滚动问题。

   ``````javascript
   //局部滚动区域到达顶部或者底部这种临界区域的时候，由于移动端会无法判断此时是局部滚动还是全局滚动，默认会冒泡，使得全局也发生滚动事件。
   //解决方案：可以使得局部区域永远滚不到顶部或者底部的方案。 https://zhuanlan.zhihu.com/p/24837233
   var ScrollFix = function(elem) {
       var startY, startTopScroll;
       elem.addEventListener('touchstart', function(event){
           startY = event.touches[0].pageY;
           startTopScroll = elem.scrollTop;
           //当滚动条在最顶部的时候
           if(startTopScroll <= 0)
               elem.scrollTop = 1;
           //当滚动条在最底部的时候
           if(startTopScroll + elem.offsetHeight >= elem.scrollHeight)
               elem.scrollTop = elem.scrollHeight - elem.offsetHeight - 1;
       }, false);
   };
   http://www.zhangxinxu.com/study/201612/mobile-scroll-prevent-window-scroll.html这个完美解决了。但是需要完全一致的布局，不能忽略display:none;和  visibility: hidden;之间的差异，以及html body的overflow:hidden；
   这里的smartScroll存在一个兼容性问题，由于这个插件是基于jq或者zepto的，在zepto或者原生js上面，可以读到event.touches[0],即此时的手指列表。但是在jq中，由于重新封装，所以需要读取下面的这个属性:      event.originalEvent.touches[0]。
           $.smartScroll = function(container, selectorScrollable) {
               // 如果没有滚动容器选择器，或者已经绑定了滚动时间，忽略
               if (!selectorScrollable || container.data('isBindScroll')) {
                   return;
               }

               // 是否是搓浏览器
               // 自己在这里添加判断和筛选
               var isSBBrowser;

               var data = {
                   posY: 0,
                   maxscroll: 0
               };

               // 事件处理
               container.on({
                   touchstart: function (event) {
                       var events = event.originalEvent.touches[0] || event.touches[0] || event;

                       // 先求得是不是滚动元素或者滚动元素的子元素
                       var elTarget = $(event.target);

                       if (!elTarget.length) {
                           return;
                       }

                       var elScroll;

                       // 获取标记的滚动元素，自身或子元素皆可
                       if (elTarget.is(selectorScrollable)) {
                           elScroll = elTarget;
                       } else if ((elScroll = elTarget.parents(selectorScrollable)).length == 0) {
                           elScroll = null;
                       }

                       if (!elScroll) {
                           return;
                       }

                       // 当前滚动元素标记
                       data.elScroll = elScroll;

                       // 垂直位置标记
                       data.posY = events.pageY;
                       data.scrollY = elScroll.scrollTop();
                       // 是否可以滚动
                       data.maxscroll = elScroll[0].scrollHeight - elScroll[0].clientHeight;
                   },
                   touchmove: function () {
                       // 如果不足于滚动，则禁止触发整个窗体元素的滚动
                       if (data.maxscroll <= 0 || isSBBrowser) {
                           // 禁止滚动
                           event.preventDefault();
                           return;
                       }
                       // 滚动元素
                       var elScroll = data.elScroll;
                       // 当前的滚动高度
                       var scrollTop = elScroll.scrollTop();

                       // 现在移动的垂直位置，用来判断是往上移动还是往下
                       var events = event.touches[0] || event;
                       // 移动距离
                       var distanceY = events.pageY - data.posY;

                       if (isSBBrowser) {
                           elScroll.scrollTop(data.scrollY - distanceY);
                           elScroll.trigger('scroll');
                           return;
                       }

                       // 上下边缘检测
                       if (distanceY > 0 && scrollTop == 0) {
                           // 往上滑，并且到头
                           // 禁止滚动的默认行为
                           event.preventDefault();
                           event.stopPropagation();
                           return;
                       }

                       // 下边缘检测
                       if (distanceY < 0 && (scrollTop + 1 >= data.maxscroll)) {
                           // 往下滑，并且到头
                           // 禁止滚动的默认行为
                           event.preventDefault();
                           event.stopPropagation();
                           return;
                       }
                   },
                   touchend: function () {
                       data.maxscroll = 0;
                   }
               });

               // 防止多次重复绑定
               container.data('isBindScroll', true);
           };
   ``````

*  任意滚动区域的滚动事件

   ``````javascript
           $(".scrollContainer").scroll(function(){
               var viewHeight =$(this).height();//可见高度
               var contentHeight =$(this).get(0).scrollHeight;//内容高度
               var scrollHeight =$(this).scrollTop();//滚动高度
               var reachBottomValue = contentHeight-viewHeight;
               var offset = 100;     //距离底部多少的偏移值

               if(scrollHeight+offset >= reachBottomValue&&!isPull&&reachBottomValue){  //到达预计的偏移值点 触发加载
                   states.searchPage++;
                   getList(true)
               }

           });
   ``````

*  解决移动端fixed布局下面，键盘弹起时部分老的webview会影响布局（较新的webview影响比较小）

   ``````
   可以使用absolute布局解决。
   ``````

   ​