<title>Property-based testing</title>

Property-based testing
======================

---

API design は *algebra*. その *laws* or *property* を自動でテストしたいよねという話.

### ref
- [ScalaCheck Documentation](http://www.scalacheck.org/documentation.html)
- [jessitron/scalacheck-prisoners-dilemma](https://github.com/jessitron/scalacheck-prisoners-dilemma)
- [mog project: Scala: Property-Based Testing with ScalaTest and ScalaCheck](http://mogproject.blogspot.jp/2014/10/scala-property-based-testing-with.html)
- [Property-based testing | ScalaTest](http://www.scalatest.org/user_guide/property_based_testing)
- [QuviQ | amazing test tools](http://www.quviq.com/index.html)
  - [*QuickCheck* A Silver Bullet for testing?](http://htmlpreview.github.io/?https://raw.github.com/strangeloop/lambdajam2013/master/slides/Norton-QuickCheck.html)
- [Real World Property Based Testing](http://blog.charleso.org/property-testing-preso/#1)

---

8.4 Using the library and improving its usability
-------------------------------------------------
8.3 まででreasonable なAPI考えた． そのライブラリーを使ってみることによって, 考えてきたAPIの抜け漏れや表現力，使いやすさを見てみましょう．

---

### Usablity
- convenient syntax
- appropriate helper functions
- common usage pattern

expressive かつ pleasant to use にしよう．

---

#### 例

```scala
val smallInt = Gen.choose(-10,10)
val maxProp = forAll(listOf(smallInt)) { ns =>
val max = ns.max
  !ns.exists(_ > max)
}
```

```scala
case class Prop(run: (MaxSize,TestCases,RNG) => Result)
```

いざ`maxProp`をrun しようとすると`(MaxSize,TestCases,RNG)` の3つを渡すの面倒．
-> companion objectに `run`メソッド持たせて, デフォルト引数使おう．


```scala
def run(p: Prop,
        maxSize: Int = 100,
        testCases: Int = 100,
        rng: RNG = RNG.Simple(System.currentTimeMillis)): Unit =
  p.run(maxSize, testCases, rng) match {
    case Falsified(msg, n) =>
      println(s"! Falsified after $n passed tests:\n $msg")
    case Passed =>
      println(s"+ OK, passed $testCases tests.")
  }
```

---

ところで`run(maxProp)` すると実は空のリストが渡されたケースで落ちる.

```
scala> val maxProp = Prop.forAll(listOf(smallInt)) { ns =>
     |   val max = ns.max
     |   !ns.exists(_ > max)
     | }
maxProp: fpinscala.testing.Prop = Prop(<function3>)

scala> Prop.run(maxProp)
! Falsified after 0 passed tests:
 test case: List()
generated an exception: empty.max
```

Property-based testing によって

1. コードに隠された暗黙の前提を明らかに
2. その前提を explicit にすることを強いられる

---

8.4.2 Writing a test suite for parallel computations
----------------------------------------------------
7章でみた，parallel computations についてのlawsをこのライブラリで満たせる?
-> 満たせるけど適切なhelper functionsと貧困なsyntaxによって意図がぼやけてしまう．

#### テストケース

```scala
map(unit(1))(_ + 1) == unit(2)
```

#### Parallelに

```scala
val ES: ExecutorService = Executors.newCachedThreadPool
val p1 = Prop.forAll(Gen.unit(Par.unit(1)))(i =>
  Par.map(i)(_ + 1)(ES).get == Par.unit(2)(ES).get)
```

---

### わかりやすくしていく

```scala
def forAll[A](as: Gen[A])(f: A => Boolean): Prop
// Generator とか使わわないケースは既存のユニットテストライブラリと同じように書きたい
def check(p: => Boolean): Prop = {
  lazy val result = p
    forAll(unit(()))(_ => result)
}
// 同じ計算を何回もしてるのは無駄なのでテストケース数を無視.
// 基本ここではResult型 Passed or Falsified を返せばよいのでこうなる
def check(p: => Boolean): Prop = Prop { (_, _, _) =>
  if (p) Passed else Falsified("()", 0)
}
// このままだと1回しかやっていなくても100回やったと表示されるので, `Proved`型
// を作って`Passed`の代わりに返す
case object Proved extends Result
```

---

### 結果

これが

```scala
// 書き方は改善後にあわせる
// val p1 = Prop.forAll(Gen.unit(Par.unit(1)))(i =>
//   Par.map(i)(_ + 1)(ES).get == Par.unit(2)(ES).get)
//                   ____________________ <- 主にこの辺
val p1 = Prop.forAll(Gen.unit(Par.unit(1))) { i =>
  val p = Par.map(i)(_ + 1)(ES)
  val p2 = Par.unit(2)
  p(ES).get == p2(ES).get
}
```

こうなった

```scala
val p2 = Prop.check {
  val p = Par.map(Par.unit(1))(_ + 1)
  val p2 = Par.unit(2)
  p(ES).get == p2(ES).get
}
```

もちろんシンタックスだけでなく, テスト結果の表示や中身の無駄な実行回数が削減されてる

---

Testing higher-order functions and future directions
----------------------------------------------------

higher-order functions をテストするいい方法が今のところなかった

`data` を generate はできるが `functions` をgenerate するいい手段がまだない

- `s: List[A]`
- `f: A => Boolean`
- `s.takeWhile(f).forall(f)` <- Property

のようなことがしたい. 

---

具体的にはこんなかんじ. しかし `isEven` 部分の関数をテスティングフレームワークに
よしなに作ってもらいたい

```scala
val isEven = (i: Int) => i%2 == 0
val takeWhileProp =
Prop.forAll(Gen.listOf(int))(ns => ns.takeWhile(isEven).forall(isEven))
```

例えば `Gen[String => Int]` を作りたければこうなる

```scala
def genStringIntFn(g: Gen[Int]): Gen[String => Int] =
  g map (i => (s => i))
```

ただこれでは入力された`Int`は無視される constant な関数を作ってるだけなので十分じゃない.
`takeWhile` の例に当てはめると常に`true` / `false` が返ってくる関数を作ってることになり無意味.

Excercise 8.19 で考えましょう

---

8.6 The laws of generators
--------------------------
ここで作ってきた `Gen` が`Par`や`List`, `Stream`, `Option` に似てるの面白くない???

```scala
def map[A,B](a: Par[A])(f: A => B): Par[B]
def map[A,B](l: List[A])(f: A => B): List[B]
def map[A,B](s: Rand[A])(f: A => B): Rand[B]
def map[B](f: A => B): Option[B]
def map[B](f: A => B): Gen[B]
def map[B](f: A => B): Stream[B]
```

ただ似てるだけ or 何らかの 同じ**laws**を満たしてる?

---

chapter 7 でのParのlawsがこれ

```scala
map(x)(id) == x
// 7.4.1 The law of mapping
map(unit(x))(f)  == unit(f(x))  // Initial law
map(unit(x))(id) == unit(id(x)) // Substitute identity function for f
map(unit(x))(id) == unit(x)     // Simplify
map(y)(id)       == y           // Substitute y for unit(x) on both sides
```

このlawsを`Gen`や`List`, `Stream`も確かに満たしてる..!

...んですが, この詳細はPart 3で見ていきましょう

※ `def id[A](a: A): A = a.`

---

8.7 Summary
-----------

このChapterではproperty-based testing を使って functional library design に取り組んだ.


1. abstract algebraとconcrete representation を行き来することによって両者から情報を与えあうことができた
2. `map` や `flatMap` といったこれまでにみてくたcombinatorはsignatureが一緒なだけでなく laws も満たしていたことを発見した
  - Part 3で見ていきましょう
