- [「Functional Programming in Scala」第7回読書会 - connpass](http://fpscala-osaka.connpass.com/event/14585/)
- 16:30〜17:00 12.6〜12.8章 まとめ資料共有 @haya14busa

Links
-----
- [独習 Scalaz — The Essence of the Iterator Pattern](http://eed3si9n.com/learning-scalaz/ja/The+Essence+of+the+Iterator+Pattern.html)


CODE
----

```scala
def traverse[F[_],A,B](as: List[A])(f: A => F[B]): F[List[B]]
def sequence[F[_],A](fas: List[F[A]]): F[List[A]]
```

```scala
def sequenceMap[K,V](ofa: Map[K,F[V]]): F[Map[K,V]]
```
