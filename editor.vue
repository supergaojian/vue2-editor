<template>
  <div class="text-editor" ref="editorContainer" @mousedown="mousedown">
    <div class="editor-opera-panel" :class="[focus && 'editor-opera-panel-active']" :style="{ top: scrollTop + 'px' }">
      <div class="blod" @click="bold">B</div>
      <div class="italic" @click="italic">I</div>
      <div class="font-size">
        <div class="font-size-box">{{ size }}</div>
        <ul class="size-list">
          <li class="size" v-for="(size, index) in sizes" @click="changeSize(size, index)">
            <span :style="{ fontSize: size + 'px' }">{{ size }}px&nbsp;&nbsp;字体</span>
          </li>
        </ul>
      </div>
      <div class="text-align">
        <div class="current" :class="align">
          <div class="top-line"></div>
          <div class="middle-line"></div>
          <div class="bottom-line"></div>
        </div>
        <ul class="align-list">
          <li class="left" @click="changeAlign('left')" v-if="align != 'left'">
            <div class="top-line"></div>
            <div class="middle-line"></div>
            <div class="bottom-line"></div>
          </li>
          <li class="right" @click="changeAlign('right')" v-if="align != 'right'">
            <div class="top-line"></div>
            <div class="middle-line"></div>
            <div class="bottom-line"></div>
          </li>
          <li class="center" @click="changeAlign('center')" v-if="align != 'center'">
            <div class="top-line"></div>
            <div class="middle-line"></div>
            <div class="bottom-line"></div>
          </li>
          <li class="normal" @click="changeAlign('left')" v-if="align != 'normal'">
            <div class="top-line"></div>
            <div class="middle-line"></div>
            <div class="bottom-line"></div>
          </li>
        </ul>
      </div>
      <div class="color">
        <div class="current" :style="{ backgroundColor: currentColor }"></div>
        <ul class="color-list">
          <li class="color-detail" v-for="color in colors" @click="changeColor(color)">
            <div :style="{ backgroundColor: color }">
              <div class="color-cover"></div>
            </div>
          </li>
        </ul>
      </div>
      <!--<div class="symbol">符</div>-->
      <div class="clear-btn" @click="clear">清除格式</div>
    </div>
    <div class="editor-text-box">
      <div class="editor-text" ref="editor" contenteditable="true" @focus="focus = true" @blur="blur" @mouseup="mouseup">
        <div>{{ content }}</div>
      </div>
    </div>
  </div>
</template>

<script>
  /*
   * props {
   *   content: 传入文本（string）
   *   sizes：字号数组（Array int）
   *   colors：字体颜色（Array string）
   * }
   */
  export default {
    name: 'text-editor',

    props: [ 'content', 'sizes', 'colors' ],

    data () {
      return {
        editing: false,
        scrollTop: -38,
        range: null, // 当前选择range
        selectType: '', // 当前用户选择状态
        focus: false, // 焦点集中状态，false为未选中，true未选中态
        size: 18, // 默认字体大小
        currentSize: 18, // 当前字号
        align: 'normal', // 文本对其方式，normal全文对齐、left左对齐、right右对齐、center居中
        currentColor: '#000000' // 字体颜色
      }
    },

    methods: {
      mousedown (e) {
        e.stopPropagation()
        e.path.map(path => {
          if (path.className && path.className.indexOf('editor-opera-panel') > -1) {
            this.editing = true
          }
        })
      },

      blur () {
        if (!this.editing) {
          this.focus = false
          this.editing = false
//        console.log(document.querySelector('.editor-text').innerHTML)
        }
      },

      mouseup () {
        let selection = document.getSelection()
        this.range = selection.getRangeAt(0)
        this.selectType = selection.type
        let startContainer = selection.anchorNode
        if (startContainer.nodeType !== 1 && startContainer.tagName !== 'DIV') {
          startContainer = startContainer.parentNode
        }
        this.scrollTop = startContainer.offsetTop - 38
      },

      bold () {
        // 字体加粗
        if (!this.range) {
          return
        }
        if (this.selectType === 'Range') {
          this.$refs.editor.focus()
          let selection = document.getSelection()
          selection.removeAllRanges()
          selection.addRange(this.range)
          document.execCommand('bold', false, null)
          this.editing = false
        }
      },

      italic () {
        // 字体加斜体
        if (!this.range) {
          return
        }
        if (this.selectType === 'Range') {
          this.$refs.editor.focus()
          let selection = document.getSelection()
          selection.removeAllRanges()
          selection.addRange(this.range)
          document.execCommand('italic', false, null)
          this.editing = false
        }
      },

      changeSize (size, index) {
        // 修改字号大小
        if (!this.range) {
          return
        }
        this.size = size
        const { editor } = this.$refs
        editor.focus()
        let selection = document.getSelection()
        selection.removeAllRanges()
        if (this.selectType === 'Range') {
          // 修改选择文本的字号
          selection.addRange(this.range)
          document.execCommand('fontsize', false, index)
        } else if (this.selectType === 'Caret') {
          // 修改当前行的字号
          let range = document.createRange()
          let { startContainer } = this.range
          range.selectNodeContents(startContainer)
          selection.addRange(range)
          document.execCommand('fontsize', false, index)
          selection.removeAllRanges()
        }
        let fonts = editor.querySelectorAll('font')
        fonts.forEach(font => {
          if (font.hasAttribute('size')) {
            font.removeAttribute('size')
            font.style.fontSize = size + 'px'
          }
        })
        this.editing = false
      },

      changeAlign (type) {
        // 修改对齐方式，仅支持单行修改
        if (!this.range) {
          return
        }
        const { editor } = this.$refs
        editor.focus()
        let selection = document.getSelection()
        selection.removeAllRanges()
        if (this.selectType === 'Caret') {
          // 当前行做修改
          this.align = type
          let range = document.createRange()
          let { startContainer } = this.range
          range.selectNodeContents(startContainer)
          selection.addRange(range)
          document.execCommand('justify' + type, false, null)
          this.editing = false
        }
      },

      changeColor (color) {
        // 修改字体颜色，仅在选中文本中可以使用
        if (!this.range) {
          return
        }
        if (this.selectType === 'Range') {
          this.currentColor = color
          this.$refs.editor.focus()
          let selection = document.getSelection()
          selection.removeAllRanges()
          selection.addRange(this.range)
          document.execCommand('forecolor', false, color)
          this.clearRange()
          this.editing = false
        }
      },

      clear () {
        // 清除格式
        if (!this.range) {
          return
        }
        const { editor } = this.$refs
        editor.focus()
        let selection = document.getSelection()
        selection.removeAllRanges()
        if (this.selectType === 'Range') {
          // 清除选择范围的格式
          selection.addRange(this.range)
        } else if (this.selectType === 'Caret') {
          // 清除当前行的格式
          let range = document.createRange()
          let { startContainer } = this.range
          if (startContainer.nodeType !== 1 && startContainer.tagName !== 'DIV') {
            startContainer = startContainer.parentNode
          }
          range.selectNodeContents(startContainer)
          selection.addRange(range)
        }
        document.execCommand('removeformat', false, null)
        this.clearRange()
        this.editing = false
      },

      clearRange () {
        // 清除range
        this.range = null
      }

    }

  }
</script>

<style scoped>
  /* reset */
  * {
    box-sizing: border-box;
  }
  ul,li {
    padding: 0px;
    list-style: none;
  }
  .text-editor {
    position: relative;
    width: 100%;
    min-height: 30px;
    font-size: 18px;
  }
  /* 操作面板 */
  .editor-opera-panel {
    z-index: -1;
    transform: scaleX(0);
    position: absolute;
    top: -38px;
    right: 0px;
    padding: 0px 10px;
    width: auto;
    height: 30px;
    line-height: 30px;
    font-size: 16px;
    color: #ffffff;
    border-radius: 4px;
    background-color: #000000;
    box-shadow: 0 2px 1px rgba(0, 0, 0, 0.2);
    transition: all 0.1s linear;
    transform-origin: 0 100%;
  }
  .editor-opera-panel:after {
    clear: both;
    height: 0;
    display: block;
    visibility: hidden;
    content: "";
  }
  .editor-opera-panel-active {
    z-index: 999;
    transform: scaleX(1);
  }
  /* 显示中列表样式 */
  .editor-opera-panel .acitive-list {
    transform: scaleY(1) !important;
    opacity: 1 !important;
  }
  /* 加粗 */
  .editor-opera-panel .blod {
    float: left;
    width: 15px;
    font-weight: bold;
    cursor: pointer;
  }
  /* 斜体 */
  .editor-opera-panel .italic {
    float: left;
    margin-left: 8px;
    width: 15px;
    font-weight: bold;
    font-style: italic;
    cursor: pointer;
  }
  /* 字号 */
  .editor-opera-panel .font-size {
    position: relative;
    float: left;
  }
  .editor-opera-panel .font-size:hover .size-list {
    transform: scaleY(1) !important;
    opacity: 1 !important;
  }
  .editor-opera-panel .font-size-box {    
    position: relative;
    margin-top: 7px;
    margin-left: 10px;
    width: 50px;
    height: 16px;
    line-height: 16px;
    font-size: 14px;
    color: #000000;
    border-radius: 3px;
    background-color: #FFFFFF;
    cursor: pointer;
  }
  .editor-opera-panel .font-size-box:before {
    position: absolute;
    top: 5px;
    right: 5px;
    content: "";
    border-left: 4px solid transparent;
    border-right: 4px solid transparent;
    border-top: 6px solid #000000;
  }
  .editor-opera-panel .font-size .size-list {
    transform: scaleY(0);
    opacity: 0.5;
    position: absolute;
    right: -1px;
    top: 8px;
    padding: 0px;
    min-width: 50px;
    border-radius: 4px;
    border: 1px solid #333333;
    background-color: #FFFFFF;
    box-shadow: 0 2px 1px rgba(0, 0, 0, 0.2);
    transition: all 0.1s linear;
    transform-origin: 0 0;
  }
  .editor-opera-panel .font-size .size-list .size {
    padding: 0px 10px;
    min-width: 50px;
    min-height: 30px;
    line-height: 30px;
    color: #000000;
    text-align: left;
    white-space: nowrap;
    cursor: pointer;
    transition: all 0.2s linear;
  }
  .editor-opera-panel .font-size .size-list .size:hover {
    color: #2db7f5;
    background-color: #efefef;
  }
  .editor-opera-panel .font-size .size-list .size span{
    display: block
  }
  /* 对齐方式 */
  .editor-opera-panel .text-align {
    position: relative;
    margin-left: 8px;
    float: left;
    padding-top: 8px;
    width: 20px;
    height: 30px;
    cursor: pointer;
  }
  .editor-opera-panel .text-align:hover .align-list {
    transform: scaleY(1) !important;
    opacity: 1 !important;
  }
  .editor-opera-panel .text-align .top-line,
  .editor-opera-panel .text-align .middle-line,
  .editor-opera-panel .text-align .bottom-line {
    margin-left: 10%;
    margin-bottom: 3px;
    width: 80%;
    height: 2px;
    background-color: #FFFFFF;
    transition: all 0.2s linear;
  }
  .editor-opera-panel .text-align .left .top-line,
  .editor-opera-panel .text-align .left .bottom-line {
    width: 60%;
  }
  .editor-opera-panel .text-align .right .top-line,
  .editor-opera-panel .text-align .right .bottom-line {
    width: 60%;
    margin-left: 30%;
  }
  .editor-opera-panel .text-align .center .top-line,
  .editor-opera-panel .text-align .center .bottom-line {
    width: 60%;
    margin-left: 20%;
  }
  .editor-opera-panel .text-align .align-list {
    transform: scaleY(0);
    opacity: 0.5;
    position: absolute;
    top: 10px;
    right: -2px;
    padding: 5px 2px 0px;
    width: 24px;
    border-bottom-left-radius: 3px;
    border-bottom-right-radius: 3px;
    background-color: #000000;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
    transition: all 0.1s linear;
    transform-origin: 0 0;
  }
  .editor-opera-panel .text-align .align-list li {
    margin-bottom: 7px;
    padding-top: 4px;
    height: 20px;
  }
  /* 字体颜色 */
  .editor-opera-panel .color {
    position: relative;
    float: left;
    margin-left: 8px;
    width: 20px;
    height: 30px;
  }
  .editor-opera-panel .color:hover .color-list {
    transform: scaleY(1) !important;
    opacity: 1 !important;
  }
  .editor-opera-panel .color .current{
    margin-top: 5px;
    width: 20px;
    height: 20px;
    border: 3px solid #FFFFFF;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.2s linear;
  }
  .editor-opera-panel .color .color-list {
    transform: scaleY(0);
    opacity: 0.5;
    position: absolute;
    top: 10px;
    right: -4px;
    padding: 0px 4px;
    width: 28px;
    border-bottom-left-radius: 3px;
    border-bottom-right-radius: 3px;
    background-color: #000000;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
    transition: all 0.1s linear;
    transform-origin: 0 0;
  }
  .editor-opera-panel .color .color-list .color-detail {    
    margin: 5px 0px;
  }
  .editor-opera-panel .color .color-list .color-detail div {
    position: relative;
    width: 20px;
    height: 20px;
    border: 3px solid #FFFFFF;
    border-radius: 3px;
    cursor: pointer;
  }
  .editor-opera-panel .color .color-list .color-detail div .color-cover {
    opacity: 0;
    position: absolute;
    width: 100%;
    height: 100%;
    background-color: rgba(255, 255, 255, 1);
    transition: all 0.2s linear;
  }
  .editor-opera-panel .color .color-list .color-detail div:hover .color-cover {
    opacity: 0.3;
  }
  .editor-opera-panel .symbol {
    float: left;
    margin-left: 8px;
    line-height: 30px;
    font-size: 15px;
    font-weight: bold;
    cursor: pointer;
  }
  .editor-opera-panel .clear-btn {
    float: left;
    margin-left: 8px;
    line-height: 30px;
    font-size: 13px;
    font-weight: bold;
    cursor: pointer;
  }
  .editor-text {
    outline: none;
  }
</style>
