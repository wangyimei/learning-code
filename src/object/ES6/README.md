知识点对应的小demo为`class.js`， 使用Code Runner查看对应输出。
# class 基本语法
```
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }

    toString() {
        return `(${this.x},${this.y})`;
    }
}
```
- 其中`constructor`就是构造方法，`this`关键字代表实例对象。
- 类的所有方法都是定义在类的`prototype`属性上的。
```
// 等同于
Point.prototype = {
    constructor(){},
    toString() {}
}
```
- 但是类内部定义的所有方法都是不可枚举的。（这一点与ES5）行为不一致。
```
Object.keys(Point.prototype);  // []
```

## constructor
一个类必须有`constructor`方法，若是没有显示定义，一个空的`constructor`方法会被默认添加。`constructor`方法默认返回实例对象`this`，也可以指定返回其他对象。若果返回新对象，那么实例的对象将不是(`Foo`)类的实例。
```
class Foo {
    constructor() {
        return Object.create(null);
    }
}

new Foo() instanceof Foo; // false
```
类必须使用`new`调用，否则会报错。
## getter and setter
与ES5写法一致，并且都是设置在属性的描述符对象上的。
```
class Point {
    constructor() {
    }

    set prop(val) {
        console.log('setter:' + val);
    }

    get prop() {
        console.log('getter');
    }
}
const instance = new Point();
instance.prop = 123; // setter:123
instance.prop; // getter
const descriptor = Object.getOwnPropertyDescriptor(Point.prototype, 'prop');
console.log('get' in descriptor);
console.log('set' in descriptor);
```
## 注意点
1. 类和模块内部默认就是严格模式；
2. 不存在提升。（为了保证在继承的时候，子类在父类之后定义。）
3. name属性，本质上ES6的类知识ES5构造函数的一层包装，所以函数的许多特性都被`class`继承，包括`name`。
4. Generator方法，如果在某个方法的前面加上（`*`） ，就表示该方法是Generator方法。
5. this的指向，类的方法内部如果包含`this`， 它默认指向类的实例。如果要单独使用内部有`this`的方法，则需手动将 `this`绑定到类的实例上，否则很可能报错。因为类内部使用的是严格模式，所以`this`实际上指向的是`undefined`。

## 静态方法
如果一个方法前加上了`static`关键字，则该方法不会被实例继承，而是直接通过类调用。静态方法内部的`this`指向类而不是实例。父类的静态方法，可以被子类继承。并且静态方法也是可以从`super`对象上调用。
```
class Foo {
    static bar() {
      this.baz();
    }
    static baz() {
      console.log('hello');
    }
    baz() {
      console.log('world');
    }
  }
  
Foo.bar();// hello
```
## 实例属性的新写法
实例属性可以直接定义在类的最顶层。
```
class Foo {
    _count = 0;
    ...
}
```
## `new.target`属性
`new.target`返回`new`作用于的那个构造函数的名称。需要注意的是子类继承父类时，`new.target`返回的是子类的名称。
```
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error('本类不能实例化');
    }
  }
}

class Rectangle extends Shape {
  constructor(length, width) {
    super();
    // ...
  }
}

var x = new Shape(); // 报错
var y = new Rectangle(3, 4); // 正确
```

# class继承
Class通过`extends`关键字，继承父类所有的属性和方法。子类`constructor`中需调用`super`方法，否则在创建实例的时候会抛错。如果子类不调用`super`方法，就得不到`this`对象。
与ES5不同， ES6继承机制是先将父类的实例对象的属性和方法，添加到`this`上（所以必须先调用`super`方法），然后用子类的构造函数修改`this`。
只有`super`可以调用父类实例。在子类构造函数中，只有调用了`super`之后，才可以使用`this`关键字。
## super
`super`既可以当作函数使用，又可以当作对象使用。
- `super`作为函数调用时，**代表父类的构造函数。但返回的是子类的实例**。
- `super`作为对象时，在普通方法中，**指向父类的原型对象；在静态方法中，指向父类**。

- `super()`作为函数时，只能用在子类的构造函数之中。
-  由于`super`在普通方法中指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过`super`调用的。
- 普通方法通过`super`调用父类方法时，方法内部的`this`指向当前子类的实例。
- `super`在静态方法中指向父类，但通过`super`调用父类方法时，方法内部的`this`指向当前的子类。  而不是子类的实例。
- 注意`super`必须显示指定作为函数或者对象使用，否则会报错。