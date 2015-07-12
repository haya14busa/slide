- [「Functional Programming in Scala」第6回読書会 - connpass](http://fpscala-osaka.connpass.com/event/13530/)
- 16:30〜17:00 11.4〜11.6章 まとめ資料共有 @haya14busa

Links
-----
- [Functors, Applicatives, And Monads In Pictures - adit.io](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)
- [独習 Scalaz — Functor則](http://eed3si9n.com/learning-scalaz/ja/Functor-Laws.html)
- [Modegramming Style: 圏論デザインパターン](http://modegramming.blogspot.jp/2012/03/blog-post_05.html)
- [はじめての圏論 その第1歩：しりとりの圏 - 檜山正幸のキマイラ飼育記](http://d.hatena.ne.jp/m-hiyama/20060821/1156120185)
- Scala で圏論入門: [Home · scalajp/introduction-to-category-theory-in-scala-jp Wiki](https://github.com/scalajp/introduction-to-category-theory-in-scala-jp/wiki)
- [射 (圏論) - Wikipedia](http://ja.wikipedia.org/wiki/%E5%B0%84_(%E5%9C%8F%E8%AB%96))


compose(compose(f, g), h) == compose(f, compose(g, h))


```scala
def map2[A,B,C](ma: F[A], mb: F[B])(f: (A, B) => C): F[C]
def replicateM[A](n: Int, ma: F[A]): F[List[A]]
def sequence[A](lma: List[F[A]]): F[List[A]]
def traverse[A,B](la: List[A])(f: A => F[B]): F[List[B]]
def join[A](mma: F[F[A]]): F[A]
```

コンビネータ論理は、コンビネータまたは引数のみからなる関数適用によって結果が定義されている高階関数、コンビネータに基づいている。

[Scalaで型クラス入門 — still deeper](http://chopl.in/blog/2012/11/06/introduction-to-typeclass-with-scala.html)
