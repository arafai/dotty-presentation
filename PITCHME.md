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

- path-dependent, abstract type and refinement types
- union and intersection types 
- first class functions, type and method declaration etc

+++ 

```scala
trait SeqModule {
  type Elem; type Seq
  def isEmpty(xs:Seq): Boolean
  def empty:Seq
}

def listSeq[A]: SeqModule { type Elem = A } = 
    new SeqModule {
      type Elem = A; type Seq = List[A]
      def isEmpty(xs:  Seq)  = xs.isEmpty
      def empty  = Nil
}

def isAnyEmpty(s:SeqModule)(a: s.Seq, b:s.Seq) = 
    s.isEmpty(a) && s.isEmpty(b)
```
---
