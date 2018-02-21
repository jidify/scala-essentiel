[TOC]



##  Objects et Classes (ch 3)

### Scala type hierarchy

![scala type hierachy](\img\scala-type-hierarchy.png)

There are **two special types** at the ***bottom*** of the hierarchy:

-  `Nothing` is the type of `throw` expressions, and 


- `Null` is the type of the value `null`. 

These special types are subtypes of everything else, which helps us **assign types to `throw` and `null`** while **keeping other types in our code sane**. 

The following code illustrates this:

```scala
def badness = throw new Exception("Error")
null
// res: Null = null

if(true) 123 else badness
// res: Int = 123

if(false) "it worked" else null
// res: String = null
```

Although the types of `badness` and `// res` are `Nothing` and `Null` respectively, **the types of `res2` and `res3` are still sensible**. This is because **`Int` is the least common supertype of `Int` and `Nothing`**, and **`String` is the least common supertype of `String` and `Null`**.



### Méthodes versus Champs

- un champs donne un nom à une valeur


- une méthode donne un nom à un calcul qui donne une valeur 

### Case classes (et Case objects)

TODO

#### Case objects

If you define **a case class with no constructor arguments** => define a ***case object***. 
It has the same default methods as a case class.

```scala
case object Citizen {
  def firstName = "John"
  def lastName  = "Doe"
  def name = firstName + " " + lastName
}
```

The `case object` keyword defines **a class and an object**:

```scala
class Citizen { /* ... */ }
object Citizen extends Citizen { /* ... */ }
```

### Pattern matching

syntaxe:

```scala
expr0 match {
  case pattern1 => expr1
  case pattern2 => expr2
  ...
}
```



## Traits (ch 4)

Utiliser des ***def*** dans les traits.
Les implémentations concretes peuvent utiliser soit des ***def*** ou des ***vals***.

### Modelling Data with Traits

#### Algebraic Data Types

An algebraic data type is any data that uses the above two patterns. In the functional programming literature, data using:

-  the "**has-a and**" pattern is known as a ***product type***, and 


- the "**is-a o**r" pattern is a ***sum type***.

  ​

####  Product Type pattern

`A`  ***has*** a `B` ***and*** `C`

```scala
trait A {
  def b: B
  def c: C
}
```
or
```scala
case class A(b: B, c: C)
```



#### Sum Type pattern

`A` ***is*** a `B` ***or*** `C`

```scala
trait A {
  def b: B
  def c: C
}
```

or

```scala
case class A(b: B, c: C)
```



#### The Missing Patterns

##### is-a and

```scala
trait B
trait C
trait A extends B with C
```



##### has-a or

The "has-a or" patterns means that `A` has a `B` or `C`. **There are two ways** we can implement this. 

We can say that `A` has a `d`of type `D`, where `D` is a `B` or `C`. We can mechanically apply our two patterns to implement this:

```
trait A {
  def d: D
}
sealed trait D
final case class B() extends D
final case class C() extends D
```



Alternatively we could implement this as `A` is a `D` or `E`, and `D` has a `B` and `E` has a `C`. Again this translates directly into code

```
sealed trait A
final case class D(b: B) extends A
final case class E(c: C) extends A
```



### Working with data

#### structural-recursion

Structural-recursion answers the question: "How can we do **different things** for **different data types** that belong to a **common trait**?"

#### Structural Recursion using Polymorphism

it's just the well known ***overriding*** of **POO**:

```scala
sealed trait A {
  def foo: String
}
final case class B() extends A {
  def foo: String = "It's B!"
}
final case class C() extends A {
  def foo: String = "It's C!"
}
```

If we define **an implementation in a trait**, and change the implementation in an extending class, we must use the **`override`keyword.**

```scala
sealed trait A {
  def foo: String = "It's A!"
}
final case class B() extends A {
  override def foo: String = "It's B!"
}
final case class C() extends A {
  override def foo: String = "It's C!"
}
```



#### Structural Recursion using Pattern Matching

 Define a **case** for **every subtype**, and each pattern matching case **must extract the fields we're interested in**:

```scala
def f(a: A): F = a match {
    case A(b, c) => ??? //use b and c
    case B(_, c) => ??? //use only c
  }
```



##### Where to implement pattern matching

<u>2 possibilities :</u> 

1. pattern matching **in** a method of **the base trait** 

2. pattern matching **in** a method of an **external object**

   ​

<u>The general rule is:</u>

-  if a method only depends on other fields and methods **the base trait** it is a good candidate to be implemented **inside the trait**. 


- If the method depends on other **data outside of the trait**, consider implementing it using pattern matching **in an external object** . 

- If we want to have **more than one implementation** we should use pattern matching and implement it **in  external(s) object(s).**

  ​

#### Object-Oriented vs Functional Extensibility

TODO



### Recursive Data

A particular use of algebraic data types that comes up very often is defining *recursive data*. This is data that is defined in terms of itself, and allows us to create data of potentially unbounded size (though any concrete instance will be finite).

#### Recursive Algebraic Data Types Pattern

<u>When defining</u> recursive algebraic data types, **there must be at least two cases**: 

- one **that is recursive**, and 


- one **that is not**. Cases that are not recursive are known as **base cases**. 

In code, the general skeleton is:

```
sealed trait RecursiveExample
final case class RecursiveCase(recursion: RecursiveExample) extends RecursiveExample
final case object BaseCase extends RecursiveExample

```



<u>When writing structurally recursive code</u> on a recursive algebraic data type:

- whenever we encounter **a recursive element** in the data  => we **make a recursive call to our method**.

- whenever we encounter **a base case** in the data =>  we return **the identity for the operation we are performing.**

  ​



#### Tail Recursion

##### An easy mistake about tail recursion

```scala
def sum(list: IntList): Int = list match {
    case End => 0
    case Pair(hd, tl) => hd + sum(tl)
  }
```

**Isn’t the "last action" `sum(tl)`  a call to itself, making it tail-recursive?**

Although sum(tl) is at the end of the second case expression, the **last two actions of this function are**:

1. Call `sum(tl)`

2. When that function call returns, add its value to `hd` and return that result

When we make that code more explicit and write it as a series of one-line statements: 

```scala
val s = sum(tl)
val result = hd + s
return result
```

the last calculation that happens before the return statement is the **sum of `hd` and `s`** is calculated and **not the recursive call**.



##### @tailrec annotation

One way **to be certain that a  function is tail recursive** is to annotate it with the **@tailrec** annotation to **ask the compiler** to check that methods we believe are tail recursion really are.




##### So, how do I write a tail-recursive function?

Now that you know the current approach isn’t tail-recursive, the question becomes, “How do I make it tail-recursive?” A common pattern used to make a recursive function that “accumulates a result”
into a tail-recursive function is to follow a series of simple steps:

1. Keep the original function signature the same (i.e., sum’s signature).
2. Create a second function by 
  1. copying the original function 
  2. giving it a new name 
  3. making it private 
  4. giving it a new “accumulator” input parameter
  5. adding the @tailrec annotation to it.
3. Modify the second function’s algorithm so it uses the new accumulator
4. Call the second function from inside the first function. When you do this you give the second function’s accumulator parameter an “seed” value which correspond to the identity value of the function.

```
@tailrec
def sum(list: IntList): Int = {
    @tailrec
    def sumWithAccumulator(list: IntList, total: Int = 0): Int =
      list match {
        case End => total
        case Pair(hd, tl) => sum(tl, total + hd)
    }
    sumWithAccumulator(list, 0) // '0' is the identity of 'sum' function
}
  
```



##### Remark on tail recursion

In Scala we tend **not to work directly with tail recursive** functions as there is a **rich collections library that covers the most common cases where tail recursion is used**. 
Should you need to go beyond this, because you're implementing your own datatypes or are optimising code, it is useful to know about tail recursion.