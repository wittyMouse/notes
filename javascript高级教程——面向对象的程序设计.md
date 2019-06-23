### 目录

***

#### <a href="#创建对象">创建对象</a>
1. <a href="#工厂模式">工厂模式</a>
2. <a href="#构造函数模式">构造函数模式</a>
3. <a href="#原型模式">原型模式</a>
4. <a href="#组合模式">组合使用构造函数和原型模式</a>
5. <a href="#动态原型模式">动态原型模式</a>
6. <a href="#寄生构造函数模式">寄生构造函数模式</a>
7. <a href="#稳妥构造函数模式">稳妥构造函数模式</a>

#### <a href="#继承">继承</a>
1. <a href="#原型链">原型链</a>
2. <a href="#借用构造函数">借用构造函数（伪造对象或经典继承）</a>
3. <a href="#组合继承">组合继承（伪经典继承）</a>
4. <a href="#原型式继承">原型式继承</a>
5. <a href="#寄生式继承">寄生式继承</a>
6. <a href="#寄生组合式继承">寄生组合式继承</a>

***

#### <a name="创建对象">创建对象</a>
1. <a name="工厂模式">工厂模式</a>

```javascript
function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        alert(this.name);
    };
    return o;
}

var person1 = createPerson("Nicholas", 29, "Software Engineer");
var person2 = createPerson("Greg", 27, "Doctor");
```

存在问题：无法得知对象类型

2. <a name="构造函数模式">构造函数模式</a>

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function() {
        alert(this.name);
    };
}

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

与`createPerson()`的不同之处：
- 没有显式地创建对象；
- 直接将属性和方法赋给了this对象；
- 没有`return`语句。

通过`new`操作符调用构造函数实际会经历以下4个步骤：
1. 创建一个新对象；
2. 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）；
3. 执行构造函数中的代码（为这个新对象添加属性）；
4. 返回新对象。

对象实例中存在一个`constructor`属性指向构造函数。

```javascript
alert(person1.constructor == Person);  //true
alert(person2.constructor == Person);  //true
```

对象的`constructor`属性最初是用来标识对象类型的。但是，提到检测对象类型，还是`instanceof`操作符更可靠一些。

```javascript
alert(person1 instanceof Object);  //true
alert(person1 instanceof Person);  //true
alert(person2 instanceof Object);  //true
alert(person2 instanceof Person);  //true
```

创建自定义的构造函数意味着将来可以将它的实例标识为一种特定的类型。

`person1`和`person2`既是`Person`的实例，同时又是`Object`的实例，因为所有对象均继承自`Object`。

构造函数与其他函数的唯一区别，就在于调用它们的方式不同。

```javascript
//  当作构造函数使用
var person = new Person("Nicholas", 29, "Software Engineer");
person.sayName();  //"Nicholas"

// 作为普通函数调用
person("Greg", 27, "Doctor");
window.sayName();  //"Greg"

// 在另一个对象的作用域中调用
var o = new Object();
Person.call(o, "Kristen", 25, "Nurse");
o.sayName();  //"Kristen"
```

存在问题：每个方法都要在每个实例上重新创建一遍（无法共享相同方法）。

```javascript
alert(person1.sayName == person2.sayName);  //false
```

可以通过把函数定义转移到构造函数外部来解决这个问题

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = sayName;
}

function sayName() {
    alert(this.name);
}

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

新的问题：
- 在全局作用域中定义的函数实际上只能被某个对象调用；
- 如果对象需要定义很多方法，那么就要定义很多个全局函数。

3. <a name="原型模式">原型模式</a>

```javascript
function Person() {
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
    alert(this.name);
}

var person1 = new Person();
person1.sayName();  //"Nicholas"

var person2 = new Person();
person2.sayName();  //"Nicholas"

alert(person1.sayName == person2.sayName);  //true
```

```javascript
alert(Person.prototype.isPrototypeOf(person1));  //true
alert(Person.prototype.isPrototypeOf(person2));  //true
```

```javascript
alert(Object.getPrototypeOf(person1) == Person.prototype);  //true
alert(Object.getPrototypeOf(person1).name);  //"Nicholas"
```

存在问题：
- 它省略了为构造函数传递初始化参数这一环节，所有实例在默认情况下都将取得相同的属性值；
- 共享引用类型值的属性。

4. <a name="组合模式">组合使用构造函数和原型模式</a>

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ["Shelby", "Court"];
}

Person.prototype = {
    constructor : Person,
    sayName : function() {
        alert(this.name);
    }
}

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");

person1.friends.push("Van");
alert(person1.friends);  //"Shelby", "Court", "Van"
alert(person2.friends);  //"Shelby", "Court"
alert(person1.friends == person2.friends);  //false
alert(person1.sayName == person2.sayName);  //true
```

5. <a name="动态原型模式">动态原型模式</a>

```javascript
function Person(name, age, job) {
    //属性
    this.name = name;
    this.age = age;
    this.job = job;
    //方法
    if (typeof this.sayName != "function") {
        Person.prototype.sayName = function() {
            alert(this.name);
        };
    }
}

var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();  //"Nicholas"
```

*使用动态原型模式时，不能使用对象字面量重写原型。如果在已经创建了实例的情况下重写原型，那么就会切断现有实例与新原型之间的联系。*

6. <a name="寄生构造函数模式">寄生构造函数模式</a>

```javascript
function Person(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function () {
        alert(this.name);
    };
    return o;
}

var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();  //"Nicholas"
```

```javascript
function SpecialArray() {

    //创建数组
    var values = new Array();

    //添加值
    values.push.apply(values, arguments);

    //添加方法
    values.toPipedString = function() {
        return this.join("|");
    };

    //返回数组
    return values;
}

var colors = new SpecialArray("red", "blue", "green");
alert(colors.toPipedString());  //"red|blue|green"
```

7. <a name="稳妥构造函数模式">稳妥构造函数模式</a>

```javascript
function Person(name, age, job) {
    
    //创建要返回的对象
    var o = new Object();

    //可以在这里定义私有变量和函数

    //添加方法
    o.sayName = function () {
        alert(name);
    };

    //返回对象
    return o;
}
```

注意，在这种模式创建的对象中，除了使用`sayName()`方法外，没有其他办法访问`name`的值。可以像下面使用稳妥的`Person`构造函数。

```javascript
var friend = Person("Nicholas", 29, "Software Engineer");
friend.sayName();  //"Nicholas"
```

#### <a name="继承">继承</a>
1. <a name="原型链">原型链</a>
```javascript
function SuperType() {
    this.property = true;
}

SuperType.prototype.getSuperValue = function() {
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

//继承了SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function() {
    return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue());  //true
```

所有函数的默认原型都是`Object`的实例

可以通过两种方式来确定原型和实例之间的关系。

- 使用`instanceof`操作符，只要用这个操作符来测试实例与原型链中出现过的构造函数，结果都会返回`true`。

```javascript
alert(instance instanceof Object);  //true
alert(instance instanceof SuperType);  //true
alert(instance instanceof SubType);  //true
```

- 使用`isPrototypeOf()`方法，只要是原型链中出现过的原型，都可以说是该原型链所派生的实例的原型。因此`isPrototypeOf()`方法也会返回`true`。

```javascript
alert(Object.prototype.isPrototypeOf(instance));  //true
alert(SuperType.prototype.isPrototypeOf(instance));  //true
alert(SubType.prototype.isPrototypeOf(instance));  //true
```

*子类型有时候需要重写超类型中的某个方法，或者需要添加超类型中不存在的某个方法。但不管怎么样，给原型添加方法的代码一定要放在替换原型的语句之后。*

```javascript
function SuperType() {
    this.property = true;
}

SuperType.prototype.getSuperValue = function() {
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

//继承了SuperType
SubType.prototype = new SuperType();

//添加新方法
SubType.prototype.getSubValue = function() {
    return this.subproperty;
};

//重写超类型中的方法
SubType.prototype.getSuperValue = function() {
    return false;
};

var instance = new SubType();
alert(instance.getSuperValue());  //false
```

*在通过原型链实现继承时，不能使用对象字面量创建原型方法。因为这样做就会重写原型链。*

```javascript
function SuperType() {
    this.property = true;
}

SuperType.prototype.getSuperValue = function() {
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

//继承了SuperType
SubType.prototype = new SuperType();

//使用字面量添加新方法，会导致上一行代码无效
SubType.prototype = {
    getSubValue : function() {
        return this.subproperty;
    },
    
    someOtherMethod : function() {
        return false;
    },
};

var instance = new SubType();
alert(instance.getSuperValue());  //error!
```

存在问题：
- 包含引用类型值的原型（原型模式的问题，包含引用类型值的原型属性会被所有实例共享）
- 在创建子类型的实例时，不能向超类型的构造函数中传递参数。实际上，应该说是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。

2. <a name="借用构造函数">借用构造函数（伪造对象或经典继承）</a>

```javascript
function SuperType() {
    this.colors = ["red", "blue", "green"];
}

function SubType() {
    //继承了SuperType
    SuperType.call(this);
}

var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors);  //"red, blue, green, black"

var instance2 = new SubType();
alert(instance1.colors);  //"red, blue, green"
```

```javascript
function SuperType(name) {
    this.name = name;
}

function SubType() {
    //继承了SuperType，同时还传递了参数
    SuperType.call(this, "Nicholas");

    //实例属性
    this.age = 29;
}

var instance = new SubType();
alert(instance.name);  //"Nicholas"
alert(instance.age);  //29
```

存在问题：
- 方法都在构造函数中定义，因此函数复用就无从谈起了。（构造函数模式的问题）
- 在超类型的原型中定义的方法，对子类型而言也是不可见的，结果所有类型都只能使用构造函数模式。

3. <a name="组合继承">组合继承（伪经典继承）</a>

```javascript
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    
    //继承属性
    SuperType.call(this, name);

    this.age = age;
}

//继承方法
SubType.prototype = new SuperType();

SubType.prototype.sayAge = function() {
    alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors);  //"red, blue, green, black"
instance1.sayName();  //"Nicholas"
instance1.sayAge();  //29

var instance2 = new SubType("Greg", 27);
alert(instance2.colors);  //"red, blue, green"
instance1.sayName();  //"Greg"
instance1.sayAge();  //27
```

存在问题：无论什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。

```javascript
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    SuperType.call(this, name);  //第二次调用SuperType()

    this.age = age;
}

SubType.prototype = new SuperType();  //第一次调用SuperType()
SubType.prototype.constructor = SuperType;
SubType.prototype.sayAge = function() {
    alert(this.age);
};
```

4. <a name="原型式继承">原型式继承</a>
由道格拉斯·克罗福德提出
```javascript
function() {
    function F(o) {}
    F.prototype = o;
    return new F();
}
```

```javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

alert(person.friends);  //"Shelby, Court, Van, Rob, Barbie"
```

`ECMAScript5`通过新增`Object.create()`方法规范化了原型式继承。这个方法接收两个参数：一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象。

```javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = Object.create(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = Object.create(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

alert(person.friends);  //"Shelby, Court, Van, Rob, Barbie"
```

`Object.create()`方法的第二个参数与`Object.defineProperties()`方法的第二个参数格式相同：每个属性都是通过自己的描述符定义的。以这种方式指定的任何属性都会覆盖原型对象上的同名属性。

```javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = Object.create(person, {
    name: {
        value: "Greg"
    }
});

alert(anotherPerson.name);  //"Greg"
```

*在没有必要兴师动众地创建构造函数，而只想让一个对象与另一个对象保持类似的情况下，原型式继承时完全可以胜任的。不过别忘了，包含引用类型值的属性始终都会共享相应的值，就像使用原型模式一样。*

5. <a name="寄生式继承">寄生式继承</a>

```javascript
function createAnother(original) {
    var clone = object(original);  //通过调用函数创建一个新对象
    clone.sayHi = function() {  //以某种方式来增强这个对象
        alert("hi");
    };
    return clone;  //返回这个对象
}
```

```javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
}

var anotherPerson = createAnother(person);
anotherPerson .sayHi();  //"hi"
```

*使用寄生式继承来为对象添加函数，会由于不能做到函数复用而降低效率；这一点与构造函数模式类似。*

6. <a name="寄生组合式继承">寄生组合式继承</a>

```javascript
function inheritPrototype(subType, superType) {
    var prototype = object(superType.prototype);  //创建对象
    prototype.constructor = subType;  //增强对象
    subType.prototype = prototype;  //指定对象
}
```

```javascript
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    SuperType.call(this, name);
    
    this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function() {
    alert(this.age);
}
```