# VUE3

* ref和reactive的区别 

  ```javascript
  主要是在解构后是否仍然具有响应式能力（对于子组件来讲很重要），reactive类似于对象的解构，类似于 cosnt pos = {
    x: 1,
    y: 2
  }。。
  在使用时，如果 const { x, y } = pos 这种用法会丢失响应式能力，所以只能用pos.x，pos.y来保持，
  而ref的话，类似于直接定义 let x, let y;所以这种不会丢失响应式能力。当然，vue3也提供了torefs的方法来将reactive的属性转换成ref的属性。
  ```

  

