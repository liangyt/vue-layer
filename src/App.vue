<template>
  <div id="app">
    <button @click="visible=!visible">Visible</button>
    <div class="parent">
      <layer v-model="visible" :click-mask2-close="true">
        <div><h1>Just Test Layer</h1></div>
      </layer>
    </div>
  </div>
</template>

<script>
import Layer from './components/layer'
// 以下两个引用是引用编译以后的文件,以就是发布以后的文件
// import Layer from '../dist/layer'
// import LayerCss from '../dist/layer.css'

export default {
  name: 'app',
  components: {
    Layer
  },
  data () {
    return {
      visible: false
    }
  },
  watch: {
    // 监控蒙层是否显示
    visible (val) {
      // if (val) {
      //   setTimeout(() => {
      //     this.visible = false
      //   }, 2000)
      // }
    }
  }
}
</script>

<style>
.parent {
  width: 300px;
  position: relative;
}
</style>
