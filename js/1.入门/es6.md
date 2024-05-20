# es6

## 面向对象

### q1：父类的构造函数中声明的属性跟子类的构造函数有什么联系？

>答：
>
>为什么子类中还要再写一个构造函数？不写这个构造函数的话也能用啊，子类中的构造函数肯定有一些用，但是不知道它到底是干什么的？网上说的都是如果不写constructor或者不调用super方法的时候会报错，但是我这也没报错啊，该传值还是传值，该用还是可以用。
>
>无论是父类还是子类，都会默认自带一个constructor(){}函数，即使不写，这个类中也有，就像是每个对象都有一个原型一样，在父类中，如果需要在父类中自定义属性，就必须在父类的constructor(){}中声明，否则是真的会报错，而父类中的constructor(){}的作用不只是声明，它还有一个初始化父类中自定义属性的作用，虽然可以在自定义属性时直接定义值，但是通用语法是要写在constructor(){}中，在子类中，则不需要去写constructor(){}函数，因为extends已经把父类中的所有属性都继承过来了，但是并没有为其赋值，除非在父类中已经定义好了，这时候在子类的constructor(){}中就可以为其继承过来的属性初始化值，初始化之后，即使在创建子类实例时传入了值，也仍然不起作用

### q2：对象原型实现继承的方式是什么？

>答：
>
>- **`__proto__`实现继承**
>
>每个对象都有一个特殊的属性：`__proto__`，可以用这个属性去关联另一个对象实现继承
>
>```javascript
>var animal = {
>   name: "animal",
>   eat: function() {
>       console.log(this.name + " is eating");
>   }
>};
>
>var dog = {
>   name: "dog",
>   __proto__: animal
>};
>
>dog.eat(); // dog is eating
>```
>
>但是这个方式的缺点是`__proto__`属性直到ES6才添加到规范中，存在兼容性问题，并且直接使用`__proto__`来改变原型链对性能不太友好
>
>- **构造函数实现继承**
>
>在`__proto__`之上做了一些变通，让JS可以像Java那样new出来对象
>
>```javascript
>function Animal(name, sex) { // 构造函数
>   this.name = name;
>   this.sex = sex;
>   this.eat = function() {
>       console.log(this.name + ' is eating', 'sex：' + this.sex);
>   };
>}
>
>var dog = new Animal('dog', 'male');
>
>dog.eat(); // dog is eating sex：male
>
>var cat = new Animal('cat', 'female');
>
>cat.eat(); // cat is eating sex：female
>
>console.log(dog.eat === cat.eat); // false
>```
>
>但是这个方法的缺点是构造函数的每个属性和方法都会在实例对象中重新创建一遍，我们实际需要的只是name属性和sex属性不同而已，在new的时候dog实例和cat实例都会创建（this.eat = new Function('')）一个eat方法，但两个方法并不相等，不同实例上的同名函数是不相等的，这样会同时占用两块内存，造成内存浪费，或许我们也可以把eat方法设置成全局函数试一下
>
>```javascript
>function Animal(name, sex) { // 构造函数
>   this.name = name;
>   this.sex = sex;
>   this.eat = eat;
>}
>
>function eat() {
>   console.log(this.name + ' is eating', 'sex：' + this.sex);
>}
>
>var dog = new Animal('dog', 'male');
>
>dog.eat(); // dog is eating sex：male
>
>var cat = new Animal('cat', 'female');
>
>cat.eat(); // cat is eating sex：female
>
>console.log(dog.eat === cat.eat); // true
>```
>
>这时候虽然只占用了一块内存，并且两个实例对象上的方法相等了，但是又出现了一个新问题，在全局作用域上定义的函数对象应该只能被某个唯一的对象调用，这样做就造成了全局污染问题，如果实例对象需要定义很多方法，那么就要定义很多个全局函数，那么又怎么能保证方法名不会重复呢？
>
>我们先来分析一下new的时候到底做了什么事
>
>```javascript
>function Animal(name, sex) { // 构造函数
>   this.name = name;
>   this.sex = sex;
>   this.eat = function() {
>       console.log(this.name + ' is eating', 'sex：' + this.sex);
>   };
>   console.log(this); // 没有new出实例对象前，this指向window，实例化后会改变this指向，指向实例对象
>}
>```
>
>1. 创建一个新的对象
>
>       ```javascript
>       var dog = {};
>       ```
>
>       在我们正常声明一个一般对象时，对象中有`__proto__`属性但是没有prototype属性，但是当我们声明一个函数对象时，对象中不仅有`__proto__`属性同时还有了一个prototype属性
>
>       ```javascript
>       var obj = {};
>
>       console.log(obj.__proto__) // {constructor: ƒ, …}
>
>       console.log(obj.prototype); // undefined
>
>       var fn = function() {}
>
>       console.log(fn.__proto__); // ƒ () { [native code] }
>
>       console.log(fn.prototype); // {constructor: ƒ}
>       ```
>
>2. 将构造函数的作用域赋给新对象并执行构造函数中的代码
>
>       ```javascript
>       dog.__proto__ = Animal.prototype;
>
>       Animal.call(dog);
>       ```
>
>       Animal.prototype相当于是`__proto__`实现继承中的父类
>
>3. 返回新对象
>
>       ```javascript
>       return dog;
>       ```
>
>下面拟写一个new方法
>
>```javascript
>function Animal(name, sex) { // 构造函数
>   this.name = name;
>   this.sex = sex;
>   this.eat = function() {
>       console.log(this.name + ' is eating', 'sex：' + this.sex);
>   };
>   console.log(this); // 没有new出实例对象前，this指向window，实例化后会改变this指向，指向实例对象
>}
>
>function myNew(Parent, ...prop) {
>   var Child = {};
>   Child.__proto__ = Parent.prototype;
>   Parent.apply(Child, prop);
>   return Child;
>}
>
>var dog = myNew(Animal, 'dog', 'male'); // Animal {name: "dog", sex: "male", eat: ƒ}
>
>dog.eat(); // dog is eating sex：male
>```
>
>- **原型实现继承**
>
>上面提到无论是把方法放在构造函数内部还是在构造函数外部声明一个全局方法，都有相应的缺点和弊端，有没有更好一些的方法呢？在构造函数实现继承中我们提到当创建一个函数对象时，函数对象中是会有一个prototype属性的，而一般对象中是没有这个属性的，所以我们可以通过函数对象的原型去找到对应的方法和属性
>
>```javascript
>function Animal(name, sex) {
>   this.name = name;
>   this.sex = sex;
>}
>
>Animal.prototype = {
>   eat: function() {
>       console.log(this.name + ' is eating', 'sex：' + this.sex);
>   };
>};
>
>var dog = new Animal('dog', 'male');
>
>dog.eat(); // dog is eating sex：male
>```
