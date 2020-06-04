# bugList
前端项目中遇到的较神奇的bug及解决方法,  
ps:bug主要来源于 vue的cube-ui框架,react的antd-mobile框架，以及微信公众号

#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

#### Q:UI组件失效
cube-ui里的Scroll组件，当把它的<style lang="stylus" rel="stylesheet/stylus">改成自己项目sass的<style lang="scss" type="text/css" scoped>,会导致滚动效果失效</br>
#### A:
原因排查了一下，cube-scroll-list-wrapper 里本应该设置了display:inline-block，但是由于scoped的隔绝传递，所以UI组件的样式设置没有起作用。去掉scoped可以解决，但是这样会违背了原来组件样式相互独立的初衷。可以用**深度作用选择器**">>>"来进行加深传递，不支持">>>"的sass可以用"/deep/"，例如</br>
```css
.example /deep/ {
  display:inline-block;
}
```

#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

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


#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

#### Q:微信小程序忽然间无法编译
昨天还好好的，今天忽然间就无法编译了！吓死了有没有！也没有报错！
#### A:
一般来说，微信小程序不报错却不能编译都是app.json的问题。  如下：     

1、app.json后，绝不可多加多余的逗号！如
```json
"sitemapLocation": "sitemap.json",
```
2、app.json里，不可以添加注释！


#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

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

#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

#### Q:cube-ui框架的时间选择器datePiker，重新选择第二次后竟然不会触发数据的实时刷新
我用得实在是肝疼，直接上栗子🌰
```html
<div class="timeStart">
   <span class="tit-tc-span">起始点</span>
   <div class="tit-tc-div" @click="timeStart">
       <span v-if="timeBegin==''">点击选择</span>
       <span v-else>{{timeBegin}}</span>
   </div>
   <date-picker
       ref="datePickerStart"
       :min="[2008, 8, 8]"
       :max="[2020, 10, 20]"
       :maskClosable="false"
       @select="dateSelectHandlerBegin"
       @cancel="_cancel"
    ></date-picker>
</div>
```
下边是methods里的选择方法
```js
dateSelectHandlerBegin(selectedVal) {
      console.log(selectedVal);
      console.log("timeBegin:",this.timeBegin);
      this.timeBegin = selectedVal;
      this.modelStartDateValue = new Date(
        selectedVal[0],
        selectedVal[1] - 1,
        selectedVal[2]
      ).toDateString();
      console.log("modelStartDateValue:",this.modelStartDateValue)
},
```
然后就没有然后了，{{timeBegin}}只会显示时间选择器首次选择了的值，没有进行数据刷新，注意：console.log是完全正常的，也就是会显示选择后的最新的data的值，只是页面没有进行数据绑定刷新。
#### A:
我也暂时不知道这是不是个奇怪的解决方法，目前只是发现的奇怪的思路
```html
<div class="timeStart">
   <span class="tit-tc-span">起始点</span>
   <div class="tit-tc-div" @click="timeStart">
       <span v-if="timeBegin==''">点击选择</span>
       <span v-else>{{timeBegin}}{{modelStartDateValue}}</span>
   </div>
   <date-picker
       ref="datePickerStart"
       :min="[2008, 8, 8]"
       :max="[2020, 10, 20]"
       :maskClosable="false"
       @select="dateSelectHandlerBegin"
       @cancel="_cancel"
    ></date-picker>
</div>
```
如上，当同时把{{timeBegin}}{{modelStartDateValue}}输出的时候，这两者都会同时刷新，但是只放{{timeBegin}}重新选择时间是不会进行刷新的，只放{{modelStartDateValue}}却也是可以刷新的，这问题预计是他封装的框架的方法的问题，导致watch事件没有监听到。
到目前为止，我弃用了他第一个参数传递去当data数据更新，用了第三个参数，该方法没有这种问题
```js
dateSelectHandlerBegin(selectedVal, selectedIndex, selectedText) {
      this.timeBegin = selectedText.join("");
      this.modelStartDateValue = new Date(
        selectedVal[0],
        selectedVal[1] - 1,
        selectedVal[2]
      ).toDateString();
},
```



#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

#### Q:安装了postcss-px-to-viewport，引用cube-ui（之类）的UI框架后，会导致框架样式失效
postcss-px-to-viewport是我在项目里用vw单位的时候，用来把px直接转vw的一个webpack插件，但是引入UI框架和该插件的时候，会有冲突。
#### A:
在.postcssrc.js里给postcss-px-to-viewport加一个参数就可以了
```js
"postcss-px-to-viewport": {
      //设计图宽度为750
      viewportWidth: 750,
      // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置
      viewportHeight: 1334,
      //小数点位数
      unitPrecision: 3,
      viewportUnit: 'vw',
      selectorBlackList: ['.ignore', '.hairlines','cube-picker'],
      //px的最小值
      minPixelValue: 1,
      //允许在媒体查询中转换`px`
      mediaQuery: false,
    }
```
如上，类名里带有cube-picker的，就会被此插件给忽略。但是这里我先设想到另一个情况，cube-picker这个被封装的UI组件，我很可能会进行自定义样式，一旦进行了对包含cube-picker的类改值，会不会直接被忽略了，然后沿用的还是老单位呢？这情况应该是存在的。暂时设想是如果实在遇到少数，那就用rem去改写吧。别的法子等碰到了在去想。


#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

#### Q:vue项目里使用bus总线(EventBus)在执行后，每次跳转后会绑定多一次事件
A页面绑定bus事件并跳转到B：
```js
methods: {
  getDetailed() {
      var data = {
        orderId: "1238919959125"
      };
      this.bus.$emit("orderDT", data);
      this.$router.push("/user/orderDetails");
    },
 }
```
B页面获取bus内容：
```js
methods: {
  getOrderId() {
      this.bus.$on("orderDT", content => {
        console.log("content", content);
      });
    },
  }
```
#### A:
需要在销毁前注销掉该用完的bus总线内容
```js
beforeDestroy() {
   this.bus.$off("orderDT");
},
```



#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

#### Q:vue项目里使用bus总线(EventBus)在首次进行路由跳转的时候总是不传递数据，后续跳转却可以正常传参
A页面绑定bus事件并跳转到B：
```js
methods: {
  getDetailed() {
      var data = {
        orderId: "1238919959125"
      };
      this.bus.$emit("orderDT", data);
      this.$router.push("/user/orderDetails");
    },
 }
```
B页面获取bus内容：
```js
methods: {
  getOrderId() {
      this.bus.$on("orderDT", content => {
        console.log("content", content);
      });
    },
  }
```
#### A:
这主要是生命周期的影响，执行A页面的方法时，B页面还未生成，也就是B页面的created中监听A事件传递数据的方法还没有被触发，所以即使A页面传递数据，B页面也无法取消。所以我们A页面的bus总线需要在B页面生成后才绑定。beforedestroy和destroy是在B生成后才执行的，所以我们把总线的绑定放在A页面销毁前。

A页面绑定bus事件并跳转到B：
```js
methods: {
  getDetailed() {
      this.$router.push("/user/orderDetails");
    },
 }，
beforeDestroy() {
    var data = {
      orderId: "1238919959125"
    };
    this.bus.$emit("orderDT", data);
 },
```
B页面获取bus内容：
```js
  beforeCreate() {
    this.bus.$on("orderDT", content => {
      console.log("content", content);
      this.orderDT = content
    });
  },
  beforeDestroy() {
    this.bus.$off("orderDT");
  },
```


#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

#### Q:vue使用history模式开发公众号。在一个自定义分享的需求里，如首页main进入详情页detail,在detail进行自定义分享。
一、安卓里会出现，分享首次成功之后，返回main,再进入其他detail页（参数不同）会导致自定义分享的失败。因为微信文档说了：‘所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用，同一个url仅需调用一次，对于变化url的SPA的web app可在每次url变化时进行调用’。但是我们的detail页（参数不同，但是同为detail页）要每次定义不同的分享内容（参数，比如分享人，再比如当时的detail的id）。  

二、IOS里会出现，一次在detail页的自定义分享都不成功。因为IOS里的链接，无论怎么进行push的路由变化，实际链接都还是首次打开该项目的链接。比如main是首页。进入/detail页时，url实际上还是main。故而造成连简单的一次自定义分享都不成功。

#### A:
经过调试，发现进入/detail页，先刷新一次，再进行分享，ios和安卓都没有发现自定义分享失败的情况。这是因为刷新后，安卓重新配置了配置信息。苹果重新获取了首次进入url为当前url。那么我们的解决方法就是对detail页进行刷新。经过测试，苹果可以在beforeEach进行配置，安卓则需要在afterEach里进行配置。苹果进入detail页非常正常，但是安卓需要额外刷新一次的视觉。location.replace会直接替换url，试过用location.assign,会有问题，中途凭空产生了额外一个空白的detail历史记录，导致使用返回键返回的时候，会进入一个没有数据的detail页。所以苹果的用location.replace。
在router/index.js里对这个事情进行解决：
```js
function isIOSEnv() { //判断是否为ios
  return /(iPhone|iPad|iPod|iOS)/i.test(navigator.userAgent)
}
router.beforeEach((to, from, next) => {
  // 1. 判断是不是登录页面
  // 是登录页面
  if (to.path === '/login' || to.path === '/' || to.path === '/detail') {
    // next()
    // 苹果
    if (from.path === '/' && to.path === '/detail' && isIOSEnv()) {
      location.replace(BaseUrl+'/detail')
    }
    else{
      next()
    }
  } else {
    // 不是登录页面
    // 2. 判断 是否登录过
    let token = StorageUtils.getItem(
      StorageUtils.storageKey.user_token
    );
    token ? next() : next('/')
    document.title = '登录'
  }
})

router.afterEach((to, from) => {
  // 安卓
  if (from.path === '/' && to.path === '/detail' && !isIOSEnv()) {
    window.location.reload();
  }
})
```


#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

#### Q:http-proxy-middleware的一个比较罕见的bug，独立安装该模块，进行反向代理时，某种配置会导致无法打开项目。打开http://localhost:3000/ 时显示为404

```js
const { createProxyMiddleware } = require('http-proxy-middleware');
module.exports = function (app) {
  app.use(createProxyMiddleware('/', {
    target: 'http://api.baidu.com/', // 转发到的服务器的域名/IP
    pathRewrite: {
      "^/": ""  // 重写path
    },
    changeOrigin: true
  }));
};
```

#### A:
代理没有指定网关（如/test）时，他会对所有的链接进行代理，包括了npm run start打开的服务器链接，需要把网关输入进行匹配。如下：

```js
const { createProxyMiddleware } = require('http-proxy-middleware');
module.exports = function (app) {
  app.use(createProxyMiddleware('/test', {
    target: 'http://api.baidu.com/', // 转发到的服务器的域名/IP
    pathRewrite: {
      "^/test": ""  // 重写path
    },
    changeOrigin: true
  }));
};
```



#### ---------------------------------------------🍉萌得掉血的分割线🍉------------------------------------------------

#### Q:当组合使用antd-mobile的Tabs 和 ListView的时候，发现ReactDOM.findDOMNode(ref.current).offsetTop查找ListView该dom节点的offsetTop为0，导致上拉的时候让本该固定的Tabs却被隐藏掉了

```js
<Tabs onChange={(tab, index) => { changeTabs(tab, index); }} tabs={tabs} tabBarUnderlineStyle={{ borderBottomWidth: 0.7, borderBottomColor: '#004CDF', width: '10%', textAlign: 'center', left: '5%' }} renderTabBar={props => <Tabs.DefaultTabBar {...props} page={5} />}>
          <ListView
            key={useBodyScroll ? '0' : '1'}
            ref={ref}
            dataSource={dataSource}
            // renderHeader={() => <span>Pull to refresh</span>}
            renderFooter={() => (<div style={{ padding: 30, textAlign: 'center' }}>
              {isLoading ? 'Loading...' : 'Loaded'}
            </div>)}
            renderRow={row}
            // renderSeparator={separator}
            useBodyScroll={useBodyScroll}
            style={useBodyScroll ? {} : {
              height: height,
              border: '1px solid #ddd',
              margin: '5px 0',
            }}
            pullToRefresh={<PullToRefresh
              refreshing={refreshing}
              onRefresh={onRefresh}
            />}
            onEndReached={onEndReached}
            pageSize={5}
          />
</Tabs>
```

#### A:
尚未解决

