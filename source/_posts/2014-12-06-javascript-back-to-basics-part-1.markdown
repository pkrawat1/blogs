---
layout: post
title: "Javascript Back to Basics part 1"
date: 2014-12-06 17:47:23 +0530
comments: true
categories: [javascript]
---

#Overview
JavaScript is an object oriented dynamic language, it has types and operators, core objects, and methods.

One of the key differences is that JavaScript does not have classes, instead, the class functionality is accomplished by object prototypes.

The other main difference is that functions are objects, giving functions the capacity to hold executable code and be passed around like any other object.

#What is an Object?
An object is a dynamic collection of properties. Every Property has a unique key string within the object.

``` javascript
//Fundamental operations
get
    object.name
    object['name']
set
    object.name = value;
    object['name'] = value;
delete
    delete object.name
    delete object['name']
```

#Property
A property is named collection of attributes.

**Types**

1. **Data Properties**
2. **Accessor Properties**

### Different Properties
* value: anything
* writable: boolean ;  For true, the property value can be modified.
* enumerable: boolean ; For true, property can be enumerated by a for…in statement.
* congigurable: bollean ; For true, property attributes can be changed, and the property can be deleted.
* get: function(){...return value}
* set: funtion(value){...}

### Data Properties
A data property is a property that can get and set a value. Data properties contain the value and writable properties in their descriptors.

Data Descriptor attributes are:
**value, writable, enumerable, configurable**

### Data Properties Added Without Using defineProperty
If you add a data property without using the **Object.defineProperty**, **Object.defineProperties**, or** Object.create functions**, the writable, enumerable, and configurable attributes are all set to true. After the property is added, you can modify it by using the **Object.defineProperty** function.

You can use the following ways to add a data property:

* An assignment operator (=), as in obj.color = "white";
* An object literal, as in obj = { color: "white", height: 5 };
* A construction function
``` javascript
function Circle (xPoint, yPoint, radius) {
    this.x = xPoint;
    this.y = yPoint;
    this.r = radius;
}
//Invoking constructor
var aCircle = new Circle(5, 11, 99);
//The type of all objects created with a custom constructor is object.
//There are only six types in JavaScript:
//object, function, string, number, boolean, and undefined
```

# Creating Object
```javascript
// writable, enumerable, congigurable is set true
var test = {foo: bar};
//Also written as
var test = Object.defineProperties(
    Object.create(Object.prototype), {
        foo: {
            value: bar,
            writable: true,
            enumerable: true,
            congigurable: true
        }
    }
);
//Here
Object.create(Object.prototype) gives an Object {}
```

### Accessor Properties
An accessor property calls a user-provided function every time that the property value is set or retrieved. The descriptor for an accessor property contains a get attribute, a set attribute, or both.
**Accessor descriptor attributes**
* **get**, A function that returns the property value. The function has no parameters.
* **set**, A function that sets the property value. It has one parameter that contains the value to be assigned.
* **enumerable**
* **configurable**

``` javascript
Object.defineProperty(height_to_weight,
    'height', {
        get: function(){
            return this.weight * 2.7;
        },
        set: function(value){
            this.weight = value / 2.7;
        },
        enumerable: true
    }
);

height_to_weight.height= 175 // {height: 175, weight: 65}
```

