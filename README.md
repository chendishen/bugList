# bugList
项目中遇到的较神奇的bug及解决方法

> Q:cube-ui里的Scroll组件，当把它的<style lang="stylus" rel="stylesheet/stylus">改成自己项目sass的<style lang="scss" type="text/css" scoped>,
  会导致滚动效果失效</br>
> A:原因排查了一下，cube-scroll-list-wrapper 里本应该设置了display:inline-block，但是由于scoped的隔绝传递，所以UI组件的样式设置没有起作用。去掉scoped可以解决，但是这样会违背了原来组件样式相互独立的初衷。可以用深度作用选择器**>>>**来进行加深传递，不支持">>>"的sass可以用"**/deep/**，例如</br>
```js
.example /deep/ {
  display:inline-block;
}
```
