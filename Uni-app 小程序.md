# Uni-app 小程序

* swiper显示多个item的时候。设置了next和previous margin 。对于第一个和最后一个也会有额外的margin。需要消除 的话，需要动态去设置滑到最后一个时候的next和previous。

* 使用小程序input组件时，若使用@input去监听，或者用v-model（实质也是写了一个@input事件），会发生抖动现象，即在快速输入的时候，光标会不及时跟着输入的内容走，甚至在延时之后才出现。需要去除这种抖动。可以使用form表单，不给input组件绑定@input事件。最终用bindSubmit的事件去获取值

* fix弹窗的滑动穿透问题可以添加 touch-action: none;来解决

* 行内滚动 ，超过一屏，需要设置元素为inline-block ，父元素white-space: no wrap；

* h5中。图片的缩放可以通过max-width 设置宽或者高来设置缩放的基准值。小程序里是通过mode来设置。

* 小程序。需要设置placeHolder 的样式的话，需要放在顶层css文件中。

  * 监听是否到可视区，是否到某个部分，可以用[intersection-observer](https://www.colabug.com/goto/aHR0cHM6Ly9kZXZlbG9wZXIubW96aWxsYS5vcmcvZW4tVVMvZG9jcy9XZWIvQVBJL0ludGVyc2VjdGlvbl9PYnNlcnZlcl9BUEk=)  。并且存在兼容性问题（ios 12.2以下不行），github有w3c group的[intersection-observer](https://www.colabug.com/goto/aHR0cHM6Ly9kZXZlbG9wZXIubW96aWxsYS5vcmcvZW4tVVMvZG9jcy9XZWIvQVBJL0ludGVyc2VjdGlvbl9PYnNlcnZlcl9BUEk=) 的polyfill。

* 小程序的canvas组件。源代码来自于https://github.com/kuckboy1994/mp_canvas_drawer。里面

  `````javascript
  <template>
    <view>
      <canvas
        canvas-id="canvasdrawer"
        :style="{
          width: width + 'px',
          height: height + 'px',
          position: 'absolute',
          left: '0',
          top: '0'
        }"
        class="board"
      />
      <!-- <canvas
        canvas-id="canvasdrawer"
        :style="{
          width: width + 'px',
          height: height + 'px',
          position: 'absolute',
          left: '-10000000px',
          top: '-10000000px'
        }"
        class="board"
      /> -->
    </view>
  </template>
  <script>
  export default {
    props: {
      painting: {
        type: Object,
        default: () => {},
      },
    },
    data() {
      return {
        showCanvas: false,
        isPainting: false,
        tempFileList: [],
        width: 100,
        height: 100,
        index: 0,
        imageList: [],
        ctx: null,
        cache: {},
      };
    },
    watch: {
      painting: {
        handler(newVal, oldVal) {
          console.log(newVal);
          if (JSON.stringify(newVal) !== JSON.stringify(oldVal)) {
            this.readyPigment();
          }
        },
        deep: true,
      },
    },
    onReady() {
      console.log(this);
      this.ctx = uni.createCanvasContext('canvasdrawer', this);
      if (JSON.stringify(this.painting) !== '{}') {
        this.readyPigment();
      }
    },
    methods: {
      readyPigment () {
        const { width, height, views } = this.painting;
        this.width = width || 0;
        this.height = height || 0;
        const inter = setInterval(() => {
          if (this.ctx) {
            clearInterval(inter);
            this.ctx.clearActions();
            this.ctx.save();
            this.ctx.setFillStyle('#fff'); // 这里是绘制白底，让图片有白色的背景
            this.ctx.fillRect(0, 0, this.width, this.height);
            this.getImagesInfo(views);
          }
        }, 100);
      },
      getImagesInfo (views) {
        const imageList = [];
        for (let i = 0; i < views.length; i++) {
          if (views[i].type === 'image') {
            imageList.push(this.getImageInfo(views[i].url));
          }
        }
        const loadTask = [];
        for (let i = 0; i < Math.ceil(imageList.length / 8); i++) {
          loadTask.push(new Promise((resolve, reject) => {
            Promise.all(imageList.splice(i * 8, 8)).then((res) => {
              resolve(res);
            }).catch((res) => {
              reject(res);
            });
          }));
        }
        Promise.all(loadTask).then((res) => {
          let tempFileList = [];
          for (let i = 0; i < res.length; i++) {
            tempFileList = tempFileList.concat(res[i]);
          }
          this.tempFileList = tempFileList;
          if (this.tempFileList.length) {
            this.startPainting();
          }
        });
      },
      startPainting () {
        const tempFileList = this.tempFileList;
        const views = this.painting.views;
        for (let i = 0, imageIndex = 0; i < views.length; i++) {
          if (views[i].type === 'image') {
            this.drawImage({
              ...views[i],
              url: tempFileList[imageIndex],
            });
            imageIndex++;
          } else if (views[i].type === 'text') {
            if (!this.ctx.measureText) {
              uni.showModal({
                title: '提示',
                content: '当前微信版本过低，无法使用 measureText 功能，请升级到最新微信版本后重试。',
              });
              this.$emit('getImage', { errMsg: 'canvasdrawer:version too low' });
            } else {
              this.drawText(views[i]);
            }
          } else if (views[i].type === 'rect') {
            this.drawRect(views[i]);
          }
        }
        this.ctx.draw(false, () => {
          // uni.setStorageSync('canvasdrawer_pic_cache', this.cache)
          const system = uni.getSystemInfoSync().system;
          if (/ios/i.test(system)) {
            this.saveImageToLocal();
          } else {
            // 延迟保存图片，解决安卓生成图片错位bug。
            setTimeout(() => {
              this.saveImageToLocal();
            }, 800);
          }
        });
      },
      drawImage (params) {
        this.ctx.save();
        const {
          url,
          top = 0,
          left = 0,
          width = 0,
          height = 0,
          borderRadius = 0,
          deg = 0,
        } = params;
        if (borderRadius) {
          this.ctx.beginPath();
          this.ctx.arc(left + borderRadius, top + borderRadius, borderRadius, 0, 2 * Math.PI);
          this.ctx.clip();
          this.ctx.drawImage(url, left, top, width, height);
        }
        if (!borderRadius) {
          if (deg !== 0) {
            this.ctx.translate(left + width / 2, top + height / 2);
            this.ctx.rotate(deg * (Math.PI / 180));
            this.ctx.drawImage(url, -width / 2, -height / 2, width, height);
          } else {
            this.ctx.drawImage(url, left, top, width, height);
          }
        }
  
        this.ctx.restore();
      },
      drawText (params) {
        this.ctx.save();
        const {
          MaxLineNumber = 2,
          breakWord = false,
          color = 'black',
          content = '',
          fontSize = 16,
          top = 0,
          left = 0,
          lineHeight = 20,
          textAlign = 'left',
          width,
          bolder = false,
          textDecoration = 'none',
        } = params;
        this.ctx.beginPath();
        this.ctx.setTextBaseline('top');
        this.ctx.setTextAlign(textAlign);
        this.ctx.setFillStyle(color);
        this.ctx.setFontSize(fontSize);
  
        if (!breakWord) {
          this.ctx.fillText(content, left, top);
          this.drawTextLine(left, top, textDecoration, color, fontSize, content);
        } else {
          let fillText = '';
          let fillTop = top;
          let lineNum = 1;
          for (let i = 0; i < content.length; i++) {
            fillText += [content[i]];
            if (this.ctx.measureText(fillText).width > width) {
              if (lineNum === MaxLineNumber) {
                if (i !== content.length) {
                  fillText = `${fillText.substring(0, fillText.length - 1)}...`;
                  this.ctx.fillText(fillText, left, fillTop);
                  this.drawTextLine(left, fillTop, textDecoration, color, fontSize, fillText);
                  fillText = '';
                  break;
                }
              }
              this.ctx.fillText(fillText, left, fillTop);
              this.drawTextLine(left, fillTop, textDecoration, color, fontSize, fillText);
              fillText = '';
              fillTop += lineHeight;
              lineNum++;
            }
          }
          this.ctx.fillText(fillText, left, fillTop);
          this.drawTextLine(left, fillTop, textDecoration, color, fontSize, fillText);
        }
        this.ctx.restore();
  
        if (bolder) {
          this.drawText({
            ...params,
            left: left + 0.3,
            top: top + 0.3,
            bolder: false,
            textDecoration: 'none',
          });
        }
      },
      drawTextLine (left, top, textDecoration, color, fontSize, content) {
        if (textDecoration === 'underline') {
          this.drawRect({
            background: color,
            top: top + fontSize * 1.2,
            left: left - 1,
            width: this.ctx.measureText(content).width + 3,
            height: 1,
          });
        } else if (textDecoration === 'line-through') {
          this.drawRect({
            background: color,
            top: top + fontSize * 0.6,
            left: left - 1,
            width: this.ctx.measureText(content).width + 3,
            height: 1,
          });
        }
      },
      drawRect (params) {
        this.ctx.save();
        const {
          background, top = 0, left = 0, width = 0, height = 0,
        } = params;
        this.ctx.setFillStyle(background);
        this.ctx.fillRect(left, top, width, height);
        this.ctx.restore();
      },
      getImageInfo (url) {
        return new Promise((resolve, reject) => {
          if (this.cache[url]) {
            resolve(this.cache[url]);
          }
          if (!this.cache[url]) {
            if (url) {
              const objExp = new RegExp(/^http(s)?:\/\/([\w-]+\.)+[\w-]+(\/[\w- ./?%&=]*)?/);
              if (objExp.test(url)) {
                uni.getImageInfo({
                  src: url,
                  complete: (res) => {
                    if (res.errMsg === 'getImageInfo:ok') {
                      this.cache[url] = res.path;
                      resolve(res.path);
                    } else {
                      this.$emit('getImage', { errMsg: 'canvasdrawer:download fail' });
                      reject(new Error('getImageInfo fail'));
                    }
                  },
                  fail: (err) => {
                    console.log(err);
                  },
                });
              } else {
                this.cache[url] = url;
                resolve(url);
              }
            }
          }
        });
      },
      saveImageToLocal () {
        const width = this.width;
        const height = this.height;
        uni.canvasToTempFilePath({
          x: 0,
          y: 0,
          width,
          height,
          canvasId: 'canvasdrawer',
          complete: (res) => {
            if (res.errMsg === 'canvasToTempFilePath:ok') {
              // this.showCanvas = false;
              // this.isPainting = false;
              // this.tempFileList = [];
              this.$emit('getImage', { tempFilePath: res.tempFilePath, errMsg: 'canvasdrawer:ok' });
            } else {
              this.$emit('getImage', { errMsg: 'canvasdrawer:fail' });
            }
          },
        }, this);
      },
    },
  };
  </script>
  <style lang="scss" scoped>
  .board {
    position: fixed;
    top: 2000rpx;
  }
  </style>
  
  `````

  

* 小程序设置border 为1rpx的时候，有些机型会在某些方向上展示不出来，比如iphonex 的右边border不显示。，此时加上 transform: translateY(0px) translateX(0px) 即可解决