# vue-layer-loading

> vue-layer 主要是用于显示一个加载的状态，默认是一个CSS3写的loading图，可以在组件内容区添加业务需要显示的加载过程中显示的内容.

## Build Setup

``` bash
# 安装
npm install vue-layer-loading --save

# 使用
// 引入组件
import Layer from 'vue-layer-loading'
// 引入样式，只需要在开始的地方引用一次就可以了,避免重复引入
import LayerCss from 'vue-layer-loading/dist/layer.css'

<layer
  v-model="visible" // 是否显示蒙层
  maskColor="rgba(0, 0, 0, 0.5)" // 蒙层的颜色
  :wholeWindow="true" // 是否全屏显示，默认全屏
  :click-mask2-close="false" // 点击蒙层是否消失 默认不消失
></layer>
```
