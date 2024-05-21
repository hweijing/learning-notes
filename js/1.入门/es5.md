# es5

## 函数调用

### q1：call()，apply()，bind()的用法是什么？

>答：
>
>官方解释是这样的
>
>- call()：调用对象的方法，用另一个对象替换当前对象
>- apply()：调用函数，用指定的对象替换函数的this值，用指定的数组替换函数的参数
>- bind()：对于给定的函数，创建与原始函数具有相同主体的绑定函数，绑定函数的this对象与指定对象关联，并具有指定的初始参数
>
>大致看一下意思好像没什么区别，总结一句话：改变当前函数对象this指向
>
>我们都知道如果在没有模块化划分的前提下，函数对象中的this是指向window对象的，即全局对象
>
>```javascript
>function fn() {
>   console.log(this);
>};
>
>fn(); // Window {window: Window, …}
> ```
>
>又或者在一个一般对象中声明一个函数对象，此时函数对象中的this是指向这个一般对象的
>
>```javascript
>var obj = {
>   fn: function() {
>       console.log(this);
>   };
>};
>
>obj.fn(); // {fn: ƒ}
>```
>
>而call()，apply()，bind()的作用就是改变this的指向对象，下面我们分别执行这三个方法试一下
>
>```javascript
>function fn() {
>   console.log(this);
>};
>
>var obj = {};
>
>fn.call(obj); // {}
>
>fn.apply(obj); // {}
>
>fn.bind(obj)(); // {}
>```
>
>我们发现call()，apply()方法都重复执行了一遍它们自己的调用对象（fn函数对象）中的语句，而bind()的官方定义为返回一个新的函数，所以就在后面添加了一个调用的语句，结果依然是执行了fn函数对象中的语句，而他们的共同点也显而易见，在fn单独执行时，函数中的this指向Window对象，而增加了这三个方法后，并且在方法中传了一个obj普通对象作为参数，fn中的this指向了obj普通对象，这样就证明了这三个方法不仅会先改变原函数对象中this的指向，而且又会执行一遍原函数对象的语句，下面贴一个伪代码增强理解
>
>```javascript
>var fn1 = fn.call(obj);
>
>var fn2 = (function obj() {
>   console.log(this);
>})();
>
>console.log(fn1 === fn2); // true
>```
>
>那么如果我们在fn方法对象中增加几个形参呢？在执行这三个方法时会有影响吗？下面我们来仔细分析一下
>
>```javascript
>function fn(a, b) {
>   console.log(this + '参数：' + a + ' ' +  b);
>};
>
>var obj = {};
>
>fn.call(obj); // [object Object]参数：undefined undefined
>
>fn.apply(obj); // [object Object]参数：undefined undefined
>
>fn.bind(obj)(); // [object Object]参数：undefined undefined
>```
>
>我们在fn函数对象中传了a和b两个形参，但是在调用这三个方法时仅仅传了一个obj作为参数，没有对这两个参数进行赋值，所以打印出undefined，下面我们按照官方定义的方式对这三个方法的参数进行赋值
>
>```javascript
>function fn(a, b) {
>   console.log(this + '参数：' + a + ' ' +  b);
>};
>
>var obj = {};
>
>fn.call(obj, '我是a', '我是b'); // [object Object]参数：我是a 我是b
>
>fn.apply(obj, ['我是a', '我是b']); // [object Object]参数：我是a 我是b
>
>fn.bind(obj, '我是a', '我是b')(); // [object Object]参数：我是a 我是b
>```
>
>我们发现除了apply()外，其他两个方法都可以依次为a和b进行赋值，并且同样重复执行了一遍fn函数对象中的语句，而apply()的官方要求第二个参数必须为一个数组，当传入实参时，该方法会把这个数组拆开分别赋给对应的形参
