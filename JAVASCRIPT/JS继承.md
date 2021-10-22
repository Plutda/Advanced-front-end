- 组合继承
```
function Father(name, age) {
  this.name = name
  this.age = age
}
Father.prototype.sayHi = function () {
  console.log('hi')
}
function Son(name, age) {
  Father.call(this, name, age)
}
Son.prototype = new Father()
Son.prototype.constructor = Son
const son = new Son('wang', 26)
```
组合继承的缺点：调用了两次父类函数(new Father()和Father.call(this))
<br/>
组合继承的优点：1、可以继承父类原型上的属性，可以传参，可复用。 2、每个新子类对象实例引入的构造函数属性是私有的。

**扩展：函数的constructor**
```
// a.js
let instance = new sonFn()

// b.js 中导入a模块，这时候如果没有constructor，获取构造函数就比较困难，有constructor可以像下面这样获取构造函数:
let con = instance.constructor
```
因此每次重写函数的prototype都应该修正一下constructor的指向，以保持读取constructor指向的一致性
<br/>

- Object.create()
```
/* Object.create() 的实现原理 */
// cloneObject()对传入其中的对象执行了一次浅拷贝，将构造函数F的原型直接指向传入的对象。
function cloneObject(obj){
  function F(){}
  F.prototype = shallowCopy(obj); // 将传进来obj对象作为空函数的prototype
  return new F(); // 此对象的原型为被继承的对象, 通过原型链查找可以拿到被继承对象的属性
}

var father = {
  name: 'jacky',
  age: 22,
  courses: ['前端']
}

// var son1 = Object.create(father); // 效果一样
var son1 = cloneObject(father);
son1.courses.push('后端');

var son2 = cloneObject(father);
son2.courses[2] = '全栈';

console.log(father.courses); //  ["前端", "后端", "全栈"]
```
优点: 不需要创建新类型，方便<br>
<br>
缺点: 和原型链继承一样，引用类型被修改会互相影响

- 寄生组合继承
```
function Father(name, age){
    this.name = name;
    this.age = age;
}
Father.prototype.getFatherName = function(){
    console.log('父类的方法')
}

function Son(name, age, job){
    Father.call(this,name,age); // 借用构造继承: 继承父类通过this声明属性和方法至子类实例的属性上
    this.job = job;
}

// 寄生组合式继承：封装了son.prototype对象原型式继承father.prototype的过程，并且增强了传入的对象。
function inheritPrototype(son,father){
    // 写法1，通过Object.create
    var clone = Object.create(father.prototype); // 原型式继承：浅拷贝father.prototype对象
    clone.constructor = son;  // 增强对象，弥补因重写原型而失去的默认的constructor 属性
    son.prototype = clone; // 指定对象，将新创建的对象赋值给子类的原型
    // 写法2，通过new操作符
    function clone() {};
    //clone()的原型指向的是father
    clone.prototype = father.prototype; 
    //son原型指向的是clone()
    son.prototype = new clone(); 
    // 修正构造函数
    son.prototype.constructor = son;
}
inheritPrototype(Son,Father); // 将父类原型指向子类

// 新增子类原型属性
Son.prototype.getSonName = function(){
    console.log('子类的方法')
}

```
寄生组合继承优点：<br>
  1、只调用一次父类Father构造函数。不必为了指定子类的原型而调用构造函数，而是间接的让 Son.prototype 访问到 Father.prototype。<br>
  2、寄生组合式继承是最成熟的继承方法, 也是现在最常用的继承方法，众多JS库采用的继承方案也是它。<br>
寄生组合式继承缺点：<br>
  3、给子类原型添加属性和方法的时候，一定要放在inheritPrototype()方法之后。

- ES6 extends继承（最优方式）

**核心： 类之间通过extends关键字实现继承，清晰方便。 class 仅仅是一个语法糖，它的核心思想仍然是寄生组合式继承。**
```
class Father {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  skill() {
    console.log('父类的技能');
  }
}

class Son extends Father { 
  constructor(name, age, job){
    // 相当于Father.prototype.constructor.call(this, name, age) 
    super(name, age); // 调用父类的constructor,只有调用super之后，才可以使用this关键字
    this.job = job;
  }

  getInfo() {
    console.log(this.name, this.age, this.job);
  }
}


let son = new Son('jacky',22,'前端开发');
son.skill(); // 父类的技能
son.getInfo(); // jacky 22 前端开发
```

如果子类没有定义constructor方法，这个方法会被默认添加，代码如下。也就是说，不管有没有显式定义，任何一个子类都有constructor方法。
<br>
**子类必须在constructor方法中调用super方法，否则新建实例时会报错。**<br>
这是因为子类自己的this对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用super方法，子类就得不到this对象。

**ES5继承与ES6继承的区别**<br>
1、ES5的继承实质上是先创建子类的实例对象，再将父类的方法添加到this上( Father.call(this) )。<br>
2、ES6的继承是先创建父类的实例对象this，再用子类的构造函数修改this。<br>
3、ES6的继承因为子类没有自己的this对象，所以必须先调用父类的super()方法。