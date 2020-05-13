# **Dotty**
### Let's make Scala great again

---

### Dotty goals
<br />
<br />

- strengthen Scala's foundations
- make Scala easier and safer to use
- improve the consistency and expressiveness of Scala's language constructs

Note:
Strengthen Scala's foundations - Make the full programming language compatible with the foundational work on the DOT calculus and apply the lessons learned from that work.
Make Scala easier and safer to use - Tame powerful constructs such as implicits to provide a gentler learning curve. Remove warts and puzzlers.

---

### How ?
<br />
<br />

- Dotty language (**D**ependent **O**bject **T**ypes Calculus)
- Dotty compiler (dotc)
- TASTY

---

#### TASTY (Typed Abstract Syntax Trees) - goals
<br />
<br />

- maintain Scala binary compatibility
- store output of compiler (intermediate representation)
- extra information for tooling, IDE 
- macros

+++

#### Side note: Scala migration


<br />

- Scala 2 compiler: Pickle format
- Scala 3/Dotty compiler: TASTY format
- Scala 2 is Tasty and Scala 3 eats Pickles

Note: Some feature in Scala 2 are deprecated in Scala 3 -> migration  

---

#### Dotty compiler (dotc) - goals
<br />
<br />

- inspired by temporal databases (runId, phaseId)
- new compiler architecture, mostly functional
- reliability 

---

#### DOT calculus - goals
<br />
<br />

- foundation for Dotty's type system
- type soundness
- formative
- formal verification

+++

```scala
// scala2
import scala.collection.mutable.{Map => MMap, Set => MSet}

val ms:MMap[Int, MSet[Int]] = MMap.empty

if(!ms.contains(1)) ms += 1 -> MSet(1) else ms(1) += 1

res0: scala.collection.mutable.Iterable[_ >: (Int, scala.collection.mutable.Set[Int]) with Int] 
with scala.collection.mutable.Builder[(Int, scala.collection.mutable.Set[Int])
with Int,scala.collection.mutable.Iterable[_ >: (Int, scala.collection.mutable.Set[Int]) 
with Int] with scala.collection.mutable.Builder[(Int, scala.collection.mutable.Set[Int]) 
with Int,scala.collection.mutable.Iterable[_ >: (Int, scala.collection.mutable.Set[Int]) 
with Int] with scala.collection.mutable.Builder[(Int, scala.collection.mutable.Set[Int]) 
with Int,scala.collection.mutable.Iterable[_ >: (Int, scala.collection.mutable.Set[Int])
with Int] with scala.collection.mutable.Builder[(Int, scala.collection.mutable.Set[Int]) 
with Int,scala.collection.mutable.Clearable with scala.collection.mutable.Shrinkable[Int...
```

+++

#### DOT calculus - how ?
<br />
<br />

- path-dependent type, abstract type members and structural typing 
- union and intersection types 
- small set of features, no inheritance, no traits

+++ 

#### DOT calculus 
<br />

```scala
// scala2
trait SeqModule {
  type Elem; type Seq
  def isEmpty(xs: Seq): Boolean
  def empty: Seq
}

def listSeq[A]: SeqModule { type Elem = A } = 
    new SeqModule {
      type Elem = A; type Seq = List[A]
      def isEmpty(xs:  Seq)  = xs.isEmpty
      def empty  = Nil
}

def isAnyEmpty(s: SeqModule)(a: s.Seq, b: s.Seq) = 
    s.isEmpty(a) && s.isEmpty(b)

val intSeqModule: SeqModule { type Elem = Int } = listSeq[Int]

//intSeqModule.Seq is abstract
def listInt(xs: List[Int]): intSeqModule.Seq = xs // error
```

+++

#### DOT calculus 
<br />

```scala 3
// dot-ish
type SeqModule = {
  type Elem; type Seq
  val isEmpty: Seq => Boolean
  def empty: Seq
}

def listSeq[A]: SeqModule & { type Elem = A } = 
    new {
      type Elem = A; type Seq = List[A]
      val isEmpty: Seq => Boolean = _.isEmpty
      def empty  = Nil
}
//val listSeq: (tyArg: { type A }) =>
//  SeqModule & { type Elem = tyArg.A }


def isAnyEmpty(s: SeqModule)(a: s.Seq, b: s.Seq) = 
    s.isEmpty(a) && s.isEmpty(b)

```

---

#### Dotty 
<br />

- Dotty - Essentials foundations 
- Dotty - New constructs
- Dotty - Restrictions

---

Dotty - Essentials foundations # Intersection types (GLB) 
<br />

```scala 3
  trait Printer[-A]
  trait Barkable {
    def bark() = {}
    def animals(): List[Int]
    def print(): Printer[Barkable] 
  }
  trait Growlable {
    def growl() = {}
    def animals(): List[String]
    def print(): Printer[Growlable]
  }
  class Both extends Barkable with Growlable {
    def animals(): List[Int & String] = ???
    def print(): Printer[Barkable | Growlable] = ???
  }
 
  def both(x: Barkable & Growlable) = {
    x.bark()
    x.growl()
  }
  both(Both())
```

---

Dotty - Essentials foundations # Union types (LUB) 
<br />
<br />

```scala 3
  case class Barkable(bark:String)
  case class Growlable(growl:String)

  def both(x: Barkable | Growlable) = {
    x match {
      case b: Barkable => b.bark
      case g: Growlable => g.growl
    }
  }
  
  if(true) Barkable("dog") else Growlable("lion")
// Object & Product & Serializable
  val res: Barkable | Growlable =  if(true) Barkable("dog") else Growlable("lion")
```

+++

Dotty - Essentials foundations # Union types (LUB) 
<br />
<br />

```scala 3
trait C[+T]
trait D
trait E
class A extends C[A] with D
class B extends C[B] with D with E

val x = if(true) A() else B()
// C[A | B] & D

trait A { def hello: String }
trait B { def hello: String }

def test(x: A | B) = x.hello // error: value `hello` is not a member of A | B
```

---

Dotty - Essentials foundations # Type Lambdas 
<br />
<br />

```scala
// scala2
trait Functor[A, +M[_]]{
  def map[B] (f: A => B): M[B]
}

case class SeqFunctor[A](seq: Seq[A])
  extends Functor[A, Seq]{
 
  override def map[B](f: A => B): Seq[B] =
    seq.map(f)
  }

SeqFunctor(List(1,2)).map(_ * 2)

case class MapFunctor[K,V](mapKV:Map[K,V])
  extends Functor[V, ({type L[a] = Map[K,a]})#L]{
 
  override def map[V2](f: V => V2): Map[K, V2] =
    mapKV.map { case (k,v) => k -> f(v)}   
  }
```

+++

Dotty - Essentials foundations # Type Lambdas 
<br />
<br />

```scala 3
trait Functor[A, +M[_]]{
  def map[B] (f: A => B): M[B]
}
type mapF[K] = [V] =>> Map[K,V]

case class MapFunctor[K,V](mapKV:Map[K,V])
  extends Functor[V, mapF[K]]{

  override def map[V2](f: V => V2): Map[K, V2] =
    mapKV.map { case (k,v) => k -> f(v)}
}

MapFunctor(Map("one" -> 2)).map(_ * 2)
//val res0: Map[String, Int] = Map(one -> 4)

type MF = [X, Y] =>> Map[Y, X]

type T[X] = R // shorthand type T = [X] =>> R
type F2[A, +B] = A => B // shorthand  type F2 = [A, B] =>> A => B
```

---

Dotty - Essentials foundations # Match types
<br />
<br />

```scala 3
type Elem[X] = X match {
  case String => Char
  case Array[t] => t
  case Iterable[t] => t
}

Elem[String]       =:=  Char
Elem[Array[Int]]   =:=  Int
Elem[List[Float]]  =:=  Float
Elem[Nil.type]     =:=  Nothing
```

+++

Dotty - Essentials foundations # Match types
<br />
<br />

```scala 3
type LeafElem[X] = X match {
  case String => Char
  case Array[t] => LeafElem[t]
  case Iterable[t] => LeafElem[t]
  case AnyVal => X
}
```
---

Dotty - Essentials foundations # Dependent Function Types 
<br />
<br />

```scala 3

trait Entry { type Key; val key: Key }

def extractKey(e: Entry): e.Key = e.key          // a dependent method
val extractor: (e: Entry) => e.Key = extractKey  // a dependent function value

/*
Function1[Entry, Entry#Key] {
  def apply(e: Entry): e.Key
}
*/
```

---

Dotty - New Constructs # Enums 
<br />
<br />

```scala 3
enum Color(val rgb: Int) {
  case Red   extends Color(0xFF0000)
  case Green extends Color(0x00FF00)
  case Blue  extends Color(0x0000FF)
}

val red = Color.Red
//val red: Color = Red
red.ordinal
//val res0: Int = 0

Color.valueOf("Blue")
// val res0: Color = Blue
Color.values
//val res1: Array[Color] = Array(Red, Green, Blue)

enum Color extends java.lang.Enum[Color] { case Red, Green, Blue }

```

---

Dotty - New Constructs # (Generalized) Algebraic Data Types 
<br />
<br />

```scala 3
enum Option[+T] {
  case Some(x: T) extends Option[T] // extends can be omitted
  case None       extends Option[Nothing] // if extends omitted, Nothing is inferred 
}

Option.Some("hello")
//val res1: t2.Option[String] = Some(hello)

Option.None
//val res2: t2.Option[Nothing] = None

new Option.Some(2)
//val res3: t2.Option.Some[Int] = Some(2)
```

---

Dotty - New Constructs # Kind Polymorphism
<br />
<br />

```scala 3
Int // first level, type value
List // higher kinded, type constructor

Int <: Any // * 
List <: [+X] =>> Any // * -> * 
Map <: [X, +Y] =>> Any// * -> * -> * 
List[Int] <: Any // *

def f[T <: AnyKind] = () 
f[Int]
f[List]
f[Map]
f[[X] =>> String]
```

---

Dotty - New Constructs # Polymorphic functions
<br />
<br />


```scala
// scala 2
def rank1[A](a: A) = List(a)
// rank 1 polymorphism, OK
def rank2[A, B, C](f: A => List[A], b: B, c:C):(List[B], List[C]) = (f(b), f(c)) 
// rank 2 polymorphism, error
// not valid scala code
// def rank2[A, B, C](f: A => List[A] forAll {A}, b: B, c:C):(List[B], List[C]) = (f(b), f(c)) 
// f = Function1 which is monomorphic even though the method is polymorphic in its arguments

type Id[A] = A
trait ~>[F[_],G[_]] {
  def apply[A](a: F[A]): G[A]
}

def rank2[B, C](f: Id ~> List, b:B, c:C) = (f(b), f(c))
// rank 2 using FunctionK ( natural transformation)
 
```

+++

Dotty - New Constructs # Polymorphic functions
<br />
<br />


```scala 3
//scala 3
val id = [A] => (a: A) => a

def rank2[A, B, C](f: [A] => A => List[A], b: B, c:C):(List[B], List[C]) = (f(b), f(c)) 

```

---
- Dotty - Restrictions
---


#### END

- [Martin Odersky - Compilers are Databases](https://www.youtube.com/watch?v=WxyyJyB_Ssc)  
- [Nada Amin - DOT: Scala Types from Theory to Practice](https://www.youtube.com/watch?v=fjj_fv346lY)  
- [Paolo Giarrusso — The DOT Calculus: An Introduction for Scala Programmers](https://www.youtube.com/watch?v=OVu7XzHY5U0)  
- [Guillaume Martres - Future proofing Scala the TASTY intermediate rep](https://www.youtube.com/watch?v=OVu7XzHY5U0)  


---
