## `__proto__ 指向 [[Prototype]]` 


- __proto__实际上指向的是js的内部属性`[[prototype]]`，由于js的机制无法直接访问内部属性，可以通过Object.getPrototypeOf()和Object.setPrototypeOf()访问器来访问

- `__proto__`是一个非javascript标准但许多浏览器都实现的属性

- `__proto__`是隐式的原型属性，每个**实例对象**都有`__proto__`私有属性，当**实例对象**创建时`__proto__`属性会被自动创建

- **`__proto__`指向了创建该实例对象的<font color="#ff0000">构造函数的原型</font>**

- **该原型对象也有一个自己的原型对象（`__proto__`）,层层向上直到一个对象的原型对象为null**

- **通过`__proto__`将对象和原型联系起来组成原型链**


- 根据定义null没有原型，并作为这个原型链的最后一个环节


- `[[prototype]]`机制就是存在于对象中的一个内部链接，它会引用其它对象；通常来说，这个链接的作用是：如果在对象上没有找到需要的属性或者方法引用，引擎就会继续在`[[Prototype]]`关联的对象上进行查找。同理，如果在后者中也没有找到需要的引用就会继续查找它的`[[Prototype]]`,以此类推。这一系列对象的链接被称为“原型链”

---
- __proto__看起来很像是一个属性，但是实际上它更像是一个getter/setter
- __proto__的实现大致是这样的：
  ```
  Object.defineProperty(Object.prototype,"__proto__",{
    get:function(){
      retrun Object.getPrototypeOf(this);
    },
    set:function(){
      //ES6中的setPrototypeOf(..)
      Object.setPrototypeOf(this,o);
      return o;
    }
  })
  ```