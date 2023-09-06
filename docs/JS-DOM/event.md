---
nav: js dom
title: 事件
---

## 事件类型一览表

[事件类型一览表](https://developer.mozilla.org/zh-CN/docs/Web/Events)

## 事件注册、删除

### 方式 1，通过元素属性

**两种注册方式**
1 标签属性

```html
<button onclick="alert(1);">test</button>
```

2 对象属性

```js
document.querySelector('button').onclick = function () {
  alert(2);
};
```

_这两种方式某种意义上性质一样_。上例 2 种注册方式都写，那么 1 方式将被覆盖

**删除**

```js
// 等于null即可(未注册时的默认值就是null)
document.querySelector('button').onclick = null;
```

**判断元素是否具有此类型事件**  
未注册情况，事件属性值为 `null` ，说明有此事件。  
为 `undefined` 说明没有此事件  
兼容性：ie6 all

### 方式 2，标准方法

addEventListener  
参数 3 可不带，默认为 false

removeEventListener  
attachEvent  
detachEvent

## 多个事件触发顺序、冒泡原理

**概述**  
一个元素可以拥有多个事件，包括自身的和继承过来的。  
自身的即为目标阶段，先触发。继承过来的即为冒泡阶段，后触发，且冒泡(向上)触发。  
参数 3 可以将冒泡阶段提升为捕获阶段，将最先触发(先与目标阶段触发)

**详细**  
下面的序号值(阶段值)可通过 event.eventPhase 获取

1-捕获：addEventListener 参数 3 为 true  
只会提升继承事件(所以只会提升冒泡阶段的事件)，将在自身事件之前触发(目标阶段)。(如果还有爷爷级的，先触发爷爷。。。)  
自身事件的参数 3 是否为 true 没有影响(即对目标阶段无效)  
此阶段 ie678 不支持

2-目标：自身事件的触发阶段。  
元素自身也可注册多个事件，先注册则先触发

3-冒泡：继承事件的触发阶段  
如果一个元素，除了自身事件，还有继承过来的事件，那么先触发自身，再触发继承事件(如果不止有父，还有爷爷，先触发父。。。)

```html
<div class="parent">
  <button>test</button>
</div>

<script>
  var eParent = document.querySelector('.parent'),
    eChild = eParent.children[0];

  document.body.addEventListener(
    'click',
    function () {
      console.log(6);
    },
    false,
  );
  document.body.addEventListener(
    'click',
    function () {
      console.log(5);
    },
    true,
  );
  document.body.addEventListener(
    'click',
    function () {
      console.log(4);
    },
    true,
  );

  eParent.addEventListener(
    'click',
    function () {
      console.log(3);
    },
    true,
  );
  eParent.addEventListener(
    'click',
    function () {
      console.log(2);
    },
    false,
  );

  eChild.addEventListener(
    'click',
    function () {
      console.log(1);
    },
    false,
  );
  eChild.addEventListener(
    'click',
    function () {
      console.log(0);
    },
    true,
  );

  // 依次输出：5、4、3、1、0、2、6
</script>
```

## dispatchEvent 手动触发事件

实现创建自定义事件和触发内置事件

https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/dispatchEvent

https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Creating_and_triggering_events

## 阻止冒泡

阻止继承过来的事件，详情见[多个事件触发顺序、冒泡原理](#js-dom-webapis/4078345507)

```js
e.stopPropagation();
```

**兼容性写法**

```js
if (e.stopPropagation) e.stopPropagation();
else e.cancelBubble = true; // ie678阻止方式
```

## 阻止默认动作

```js
if (e.cancelable) e.preventDefault(); //cancelable、preventDefault结合使用
```

**input[type=radio] 阻止选中。**  
可在 click、touchstart、ouchend 事件中通过 preventDefault 阻止选中。mousedown，mouseup 不能阻止选中

**关于 mousedown 阻止 click 疑问**  
mousedown 不会阻止由 click 触发的默认动作，只能由 click 自己阻止

## useCapture 触发顺序影响

useCapture 特性，跟元素层级有关系, 跟注册先后没关系，越顶层越先触发。
比如 body 和子元素 div，useCapture 为 true 情况，body 先触发

## 键盘输入事件

### 常用的几个 keyCode

- 回车键：13
- shift：16
- ctrl：17

### keypress 字符键触发

- 先触发后输入
- 可以阻止输入触发

问题：

- 删除键，这种能改变字符串的不触发
- 使用中文输入法输入时也不触发

是否是先触发后输入？
兼容性如何？

不建议当做 input 事件触发前的监听事件：

### input 有字符输入则触发

实现输入字符 后触发
改变后触发，即输入发生后触发

**1.Firefox、Safari、Chrome**
除了 input、textarea，还适用于 div 文本框。所有符号键 （包括删除键）

div 文本框 添加元素居然不触发，手动添加文本是否也不会触发呢

```js
Text1.oninput = function (e) {
  alert('');
  console.log(1);
};
```

**2.IE**
只适用于固有的 input、textarea 输入框

ie6 ie7 ie8 ie10 值变动就会触发

ie9 字符键触发，删除键不会触发

```js
Text1.onpropertychange = function (e) {
  alert('');
  console.log(1);
};
```

**3.Opera**
没有

## 触摸事件

`touchcancel`：没有可能再触发触摸事件时触发。比如，触发默认的页面滚动后，将停止监听，即触发此事件

### 特性

**多点 touch，元素外的点也会增加 TouchList：**
touch 事件元素外增加触摸点，不会触发 touch 事件，但是原事件的 TouchList 会多一个。

### e.touches 与 e.targetTouches 区别

touches 是所有在屏幕上的点，targetTouches 是当前事件元素上的点

## 鼠标事件

### 点下、松开

mousedown、mouseup

鼠标左右键都将触发

**ie678，点击失效问题(点击过快情况)**

```js
$('#div_test')
  .mousedown(function () {
    console.log('-----------------点下');
  })
  .mouseup(function () {
    console.log('松开');
  });
// 出现如下日志：
// 日志: -----------------点下
// 日志: 松开
// 日志: 松开

// 也就是执行了两次 松开事件了，
// 如下写法可以解决：
function eUp() {
  console.log('松开');
  $('#div_test').unbind('mouseup', eUp);
}
$('#div_test').mousedown(function () {
  console.log('-----------------点下');
  $('#div_test').mouseup(eUp);
});
```

### 右键菜单

```js
el.oncontextmenu = function () {
  e.preventDefault(); // 阻止默认菜单弹出
};
```

### 双击

```js
// 兼容性：所有浏览器
el.ondblclick = function () {};
el.addEventListener('dblclick', function () {});
```

### 移入事件

```js
//1、正常的 移入移出 事件：onmouseover移入、onmouseout移出】
//兼容性：all浏览器

/* ----------------------------------------------
 * 事件中特有属性——relatedTarget属性
 *
 *获取 触发事件 后 鼠标
 * 之前所处 元素对象——onmouseover移入事件
 *当前所处 元素对象——onmouseout移出事件
 *
 * event对象的属性，只对这两个事件有效：onmouseout移出事件、onmouseover移入事件
 *
 * 兼容性： 所有ie中 只有ie9支持。非ie浏览器没问题
 */
alert(e.relatedTarget.className);

/***********************************************************/
//2、特殊的 移入移出 事件
//【onmouseenter移入、onmouseleave移出】
//事件 根元素 及其后代 看成一个整体 来触发此事件
//兼容性：所有ie，Opera支持。fox、Chrome现在已经支持了
```

### 滚轮事件

[滚轮事件#浏览器兼容性 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/Events/wheel#浏览器兼容性)

目前标准 `wheel`：

```js
elem.addEventListener('wheel', function (e) {
  e.preventDefault();
  console.log(e.deltaY > 0);
});
```

## 滚动条事件

scroll

```js
// 给元素绑定
el.onscroll = function () {};
// 给浏览器窗口绑定
window.onscroll = function () {};
```

### 新版 chrome 窗口滚动条默认无法阻止解决

https://segmentfault.com/a/1190000007913386

即给 document、document.body 绑定的 touchstart、touchmove 默认无法通过 preventDefault 阻止浏览器窗口滚动条滚动

注：给其他元素绑定可阻止浏览器窗口滚动条

需通过如下方式

```js
document.addEventListener('touchmove', touchmove, { passive: false });
```

> passive 支持检测。引自 [addEventListener | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener)

```js
var passiveSupported = false;

try {
  var options = Object.defineProperty({}, 'passive', {
    get: function () {
      passiveSupported = true;
    },
  });

  window.addEventListener('test', null, options);
} catch (err) {}
```

但旧版浏览器不支持此种参数

## 设备旋转事件

chrome 模拟的是旋转前触发

android 机未测

ios 11 旋转后触发，可获取真实的浏览器宽度

```js
window.addEventListener('orientationchange', function () {
  // screen.orientation ios 11 不支持
  document.body.innerHTML = screen.orientation.angle + '--' + window.innerWidth;
});
```

## resize 窗口变化事件

ios 旋转有动画，resize 是否能获取真实浏览器宽？

ios 11 可以，也就是说旋转后触发。动画前触发。动画不影响宽度获取，估计动画是假象，动画前就已经渲染完成

印象中之前某次测试的是旋转前，难道是修复了？？

## 阻止失焦

鼠标情况在 `mousedown` 中阻止

## 问题

### 移动端(android) touchend 中 focus() 获焦失败

```js
el.addEventListener('touchend', function (e) {
  e.stopPropagation(); // 要让获焦成功，android 必须加这个
  ipt.focus();
});
```

android 在 touchend 后有默认动作触发，如果在 touchend 中通过 focus 使某文本框获焦，会立马失焦。ios 没此问题

### 移动端 chrome touchmove 卡顿现象

chrome 在触发默认滚动条时，move 事件卡顿，触发频率很低

### ios document click 无效问题

https://stackoverflow.com/questions/3705937/document-click-not-working-correctly-on-iphone-jquery

```css
html {
  cursor: pointer;
}
```
