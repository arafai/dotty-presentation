# **Dotty**

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

Side note: Scala migration

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

+++

```scala
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
- first class functions

+++ 

```scala
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

```scala 3
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

Dotty - Essentials foundations # Intersection types  
<br />

```scala 3
  trait Barkable {
    def bark(): Unit = {}
    def animals(): List[Int]
  }
  trait Growlable {
    def growl(): Unit = {}
    def animals(): List[String]
  }

  def both(x: Barkable & Growlable) = {
    x.bark()
    x.growl()
    val res:List[Int & String] = x.animals()
  }
  class Both extends Barkable with Growlable {
    def animals(): List[Int & String] = List.empty
  }
 
  both(Both())
 
```

---

Dotty - Essentials foundations # Union types  
<br />

```scala 3
  trait Barkable {
    def bark(): Unit = {}
    def animals(): List[Int]
  }
  trait Growlable {
    def growl(): Unit = {}
    def animals(): List[String]
  }

  def both(x: Barkable & Growlable) = {
    x.bark()
    x.growl()
    val res:List[Int & String] = x.animals()
  }
  class Both extends Barkable with Growlable {
    def animals(): List[Int & String] = List.empty
  }
 
  both(Both())
 
```






```

#### END

- [Martin Odersky - Compilers are Databases](https://www.youtube.com/watch?v=WxyyJyB_Ssc)  
- [Nada Amin - DOT: Scala Types from Theory to Practice](https://www.youtube.com/watch?v=fjj_fv346lY)  
- [Paolo Giarrusso — The DOT Calculus: An Introduction for Scala Programmers](https://www.youtube.com/watch?v=OVu7XzHY5U0)  
- [Guillaume Martres - Future proofing Scala the TASTY intermediate rep](https://www.youtube.com/watch?v=OVu7XzHY5U0)  


---
