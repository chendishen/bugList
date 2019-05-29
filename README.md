# bugList
项目中遇到的较神奇的bug及解决方法

#### ---------------------------------------------萌得掉血的分割线------------------------------------------------

#### Q:UI组件失效
cube-ui里的Scroll组件，当把它的<style lang="stylus" rel="stylesheet/stylus">改成自己项目sass的<style lang="scss" type="text/css" scoped>,会导致滚动效果失效</br>
#### A:
原因排查了一下，cube-scroll-list-wrapper 里本应该设置了display:inline-block，但是由于scoped的隔绝传递，所以UI组件的样式设置没有起作用。去掉scoped可以解决，但是这样会违背了原来组件样式相互独立的初衷。可以用**深度作用选择器**">>>"来进行加深传递，不支持">>>"的sass可以用"/deep/"，例如</br>
```css
.example /deep/ {
  display:inline-block;
}
```

#### ---------------------------------------------萌得掉血的分割线------------------------------------------------

#### Q:实现点击切换样式的效果
1、从后端获取到的数组adrList里遍历，给每个数组对象加一个check属性，只有一个是true,点击不同的对象的时候，会把当前对象变为true,其他的为false。（这样做是因为本身也需要把指定的check属性作为是否选中的标识传给后台，所以需要动态给数组adrList添加）  
```js
checkList:function(index){
   var Tindex = index;
   for(let i=0;i<this.adrList.length;i++){
       this.adrList[i].check = false;
   }
   this.adrList[Tindex].check = true;
}
```
2、接着使用三元表达式进行动态判断更改class，以达到更改选中样式的效果
```html
:class = "[item.check==true?"checkbox active":"checkbox"]"
```
然后，不行。好像无法动态改变（据我测试哈，如有不对，请大佬务必指正）
#### A:
“Object.defineProperty 的实现所存在的很多限制：无法监听属性的添加和删除、数组索引和长度的变更”,我觉得原因来自于这里。那如何去实现点击切换样式的效果呢？
```js
data(){
  return{
    flag:0,
  }
}
checkList:function(index){
   this.flag = index;
}
```
```html
:class="{active:index==flag}"
```
然后问题就解决了。


#### ---------------------------------------------萌得掉血的分割线------------------------------------------------

#### Q:微信小程序忽然间无法编译
昨天还好好的，今天忽然间就无法编译了！吓死了有没有！也没有报错！
#### A:
一般来说，微信小程序不报错却不能编译都是app.json的问题。  以下都是会报错的。  

1、app.json后，绝不可多加多余的逗号！如
```json
"sitemapLocation": "sitemap.json",
```
2、app.json里，不可以添加注释！


#### ---------------------------------------------萌得掉血的分割线------------------------------------------------

#### Q:cube-ui框架的input组件，为啥“支持一键清空内容”不起作用？输入栏的“X”
文档里的显示示例也不起作用，这什么破组件？但是我们项目需要做到“一键清空输入框内容”，我们如何去解决呢？
#### A:
这里有一个深坑，cube-ui绝对是我见过写文档写得最不明确的，大部分东西都需要我们去看源码来写。  
要解决这问题，首先看下源码的这个位置cube-ui-dev=>src=>components=>input=>input.vue  
然后在他的_showClear()方法里可以看到如下
```js
_showClear() {
  let visible = this.formatedClearable.visible && this.inputValue && !this.readonly && !this.disabled
  if (this.formatedClearable.blurHidden && !this.isFocus) {
    visible = false
  }
  return visible
}
```
看到重点没？？？这杀千刀的，竟然除了文档里写的“是否显示一键清空内容”外，还有好几个&&，而且没在文档说明，甚至示例你点击都不奏效！  
那么，我们想要实现“一键清空内容”，我们需要给input组件绑定的，除了文档写的clearable之外，还需要添加readonly=false和disabled=false...  
正确加上必要属性的input组件写法如下：
```html
<cube-input
  v-model="yzmLogin.value"
  :clearable="yzmLogin.clearable"
  :readonly = "yzmLogin.readonly"
  :disabled = "yzmLogin.disabled"
  class="phone"
></cube-input>
```
