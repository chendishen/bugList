# bugList
项目中遇到的较神奇的bug及解决方法

#Q:cube-ui里的Scroll组件，当把它的<style lang="stylus" rel="stylesheet/stylus">改成自己项目sass的<style lang="scss" type="text/css" scoped>,
会导致滚动效果失效
#A:原因排查了一下，cube-scroll-list-wrapper 里本应该设置了display:inline-block，但是由于scoped的隔绝，所以没有起作用。
