title: Go Interfaces
date: 2017-02-25 14:13:51
tags:
  - go
  - oop
---

## Go interfaces

If you come from a C# background, you will find Go Interfaces like entities from another planet. My recommendation is to think about the interface in the literal sense, regardless of the programming language constructs. 

I came up with this list to better describe Go interfaces:

* Go has no classes. Let that sink in a little bit; in C#/Java an interface is almost always attached to an implementor class.
* Go only has structs.
* There is no syntax to implement an interface. There is no `class Foo : IFoo` or `Foo implements Foo`
* For a struct to be a correct implementation of an interface, it only has to have those methods. 

Imagine you have these couple of classes in C#: 

```
public class Elephant 
{
   public void Eat()
   {
       Console.WriteLine("Eating grass...");
   }
}

public class Tiger
{
    public void Eat()
    {
        Console.WriteLine("Eating humans...");
    }
}
```

At some point you think, "hey those are the same method, I can make an abstraction and create:"

```
public interface IAnimal 
{
     void Eat();
}
```

And then you do:

```
public class Elephant : IAnimal
....
public class Tiger : IAnimal
....
```

And you're being a great OOP Engineer™. In Go, you don't need to go back and add that interface to those classes. When you created those classes with `Eat`, they fulfilled `IAnimal` interface in Go's compiler eyes.

![](http://res.cloudinary.com/hyeomans-blog/image/upload/c_scale,w_842/v1488061413/gopher_ossceo.png)

This is how it looks in Go:

```
package main

import (
    "fmt"
)

type Animal interface {
    Eat()
}

type Elephant struct {}

type Tiger struct { }

func (e Elephant) Eat() {
    fmt.Println("I eat grass")
}

func (t Tiger) Eat() {
    fmt.Println("I eat humans")
}

func main() {
    var animal Animal
    animal = Elephant{}
    animal.Eat()
    
    animal = Tiger{}
    animal.Eat()
}

```

Because both Tiger and Elephant have a method declaration, in Go land they are implementers of an Animal interface.

This idea __implicit__ interface implementation is so powerful in Go; you will encounter interfaces with only one method, and this will be normal.

I created the Gopher image with [https://gopherize.me/](https://gopherize.me/)
