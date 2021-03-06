---
layout: post
title: 型クラス
---

# 導入

Haskellでは多相的な関数を書けます。例えば、

```haskell
map_int_int :: (Int -> Int) -> [Int] -> [Int]
map_double_double :: (Double -> Double) -> [Double] -> [Double]
map_int_string :: (Int -> String) -> [Int] -> [String]
```

というような似た関数をたくさん定義しなくても、

```haskell
map :: (a -> b) -> [a] -> [b]
```

という一つの関数を多数の型について適用できます。ここでの型変数 `a` には任意の型を代入できます。

今度は、リストの要素の和を計算する関数を考えましょう（実際に標準ライブラリで提供されている sum 関数のことは一旦忘れます）。ここでも

```haskell
sum_int :: [Int] -> Int
sum_double :: [Double] -> Double
```

という似たような関数を書く代わりに

```haskell
sum :: [a] -> a
```

という関数を書けないでしょうか。さっきの例と違うのは、この場合の型変数 `a` には任意の型が入るわけではなく、「足し算と0が定義された」型である必要がある、という点です。

別の例を考えてみましょう。特定の値がリストに含まれているか調べる関数 `elem :: a -> [a] -> Bool` においては、 `a` が等値性を判断できる型（`(==)` が定義された型）である必要があります。（例えば、関数同士の等値性は定義されていないので、型 a は関数型であってはいけません）

このように、「任意の型」について使えるわけではないが、「ある性質を持った型」なら何にでも使えるような関数を書きたい時があります。Haskellにはそのための仕組みがあり、**型クラス**と呼ばれています。（他の言語では、関数や演算子のオーバーロードと呼ばれる仕組みを使って似たようなことを実現しています）

# 型クラスの例と型クラスを使う関数の例

Haskellには組み込みでいくつかの型クラスが用意されています。 sum 関数で必要になるような算術演算は、 Num という名前のクラスで定義されています。elem 関数に必要になるような等値性 `(==)` は Eq という名前の型クラスで定義されています。比較演算（全順序）は Ord という型クラスで定義されています。

このような型クラスを使うと、 sum 関数や elem 関数の型は次のように書けます：

```haskell
sum :: (Num a) => a -> a
elem :: (Eq a) => a -> [a] -> Bool
```

型クラスの制約を記述するには、二重矢印 `=>` を使います。型クラスが1つの場合は、型クラスの周りのカッコ `( )` は省略できます。

実は、これまでに見てきた関数にも型クラスを使って定義されているものが多数あります。ghci の `:t` コマンドで、いくつか確かめて見ましょう。

```haskell
Prelude> :t (+)
(+) :: Num a => a -> a -> a
Prelude> :t (==)
(==) :: Eq a => a -> a -> Bool
```

実は整数リテラルの型も、型クラスを使って多相的になっています。

```haskell
Prelude> :t 123
123 :: Num t => t
```

このおかげで、整数でも浮動小数点数でも同じように `x+1` と書くことができます。

課題：小数リテラル（`123.45` など）の型はどうなっているでしょうか。調べてみましょう。

「入出力入門」で使った `(>>)` や `(>>=)` も、

```haskell
Prelude> :t (>>)
(>>) :: Monad m => m a -> m b -> m b
Prelude> :t (>>=)
(>>=) :: Monad m => m a -> (a -> m b) -> m b
```

という風に、 IO という具体的な型ではなくて Monad という型クラスに対して定義されています。

前のページ（「入出力入門」）で紹介した readIO 関数も Read という型クラスを使って定義されています。

```haskell
Prelude> :t readIO
readIO :: Read a => String -> IO a
```

複数の型クラスを使う関数も定義できます。冪乗 `(^)` は、底は掛け算が定義されていれば何でもよくて、指数の方は整数っぽい型 (Int や Integer) を取れるようになっています。（なお、指数が負の場合はエラーとなります。負の整数乗を扱いたい場合は `(^^)`, 実数乗を扱いたい場合は `(**)` を使います。）

```haskell
Prelude> :t (^)
(^) :: (Num a, Integral b) => a -> b -> a
```

余談：実は、ここまで述べたルールでは、 `x ^ 2` と書いた場合に「2」の型が一意に定まりません。つまり、 `x ^ 2` は `x ^ (2 :: Int)` とも `x ^ (2 :: Integer)` とも解釈できます。このような曖昧さはHaskellの闇の一つで、「型が曖昧な場合はIntegerまたはDoubleを使う」という風な場当たり的な規則で実際の型を決定しています（このような規則は defaulting と呼ばれています）。

# インスタンスとメソッド

それぞれの型クラスに対して、その型クラスの「要件」を満たしている型は、その型クラスのインスタンス（実例、具体例）である、と呼ばれます。例えば、 Int 型や Double 型は Num クラスのインスタンスですが、 String 型は Num クラスのインスタンスではありません。

型クラスの「要件」は、「メソッド」と呼ばれるいくつかの関数で表されます。ある型をある型クラスのインスタンスにしたい時は、その型について型クラスのメソッドを実装しなければなりません。例えば、 Eq クラスのメソッドは `(==) :: a -> a -> Bool` と `(/=) :: a -> a -> Bool` の2つなので、型 T を Eq クラスのインスタンスにしたかったら `(==) :: T -> T -> Bool` という関数と `(/=) :: T -> T -> Bool` という関数の2つを自分で実装する必要があります（実際の実装のやり方は、このページの下の方を見てください）。

# 型クラスの定義

型クラスの定義の仕方を見ていきます。

```haskell
class Eq a where
  (==), (/=) :: a -> a -> Bool
```

型クラスを定義するには、 “`class`” キーワードを使います。class キーワードに続けて、型クラスの名前（大文字から始めます）、型変数（小文字から始めます）を書きます。その後に `where` キーワードを書き、メソッドの型定義を並べていきます（定義が型クラスに属することを明示するために、行頭に空白を入れてインデントしています）。Eq クラスのメソッドは `(==)` と `(/=)` の2つですが、この2つは同じ型を持つので、一行にまとめて書いています。

これだけで定義を終えてもいいのですが、実際の Eq クラスの定義はもうちょっと続きます。 `(/=)` は `(==)` の否定なので、 `(==)` さえあれば `(/=)` は自動的に定義できます。このような場合は、型クラスの定義の中でデフォルト実装を用意できます。

```haskell
  x /= y = not (x == y)
```

逆に、 `(/=)` があった場合に `(==)` を自動的に定義するためのデフォルト実装も用意されています（これを使うような状況が思いつきませんが…）。

```haskell
  x == y = not (x /= y)
```

まとめると、 Eq クラスの定義は次のようになります：

```haskell
class Eq a where
  (==), (/=) :: a -> a -> Bool
  x /= y = not (x == y)
  x == y = not (x /= y)
```

Num クラスの定義も見てみましょう：

```haskell
class Num a where
  (+), (-), (*) :: a -> a -> a
  negate :: a -> a
  abs, signum :: a -> a
  fromInteger :: Integer -> a
  x - y = x + negate y
  negate x = 0 - x
```

abs （絶対値）と signum （符号）がなければ、 Num クラスは数学的な環（単位元付き）と言っても良さそうです。（abs と signum さえなければ…！）

ちなみに、Haskellの型クラスには、演算の交換法則や結合法則のような性質（公理）を記述する機能はありません。型クラスを設計した人がそういう公理を明文化したいと思っても、利用者向けのドキュメントにそう書いておくのが関の山です。

# スーパークラスとサブクラス

型クラス A を定義する際に、その型クラスのインスタンスは別の型クラス B のインスタンスでもある、ということを要請できると便利です。

例えば、「割り算の定義された数」を表す型クラス Fractional のインスタンスに対しては、「足し算と掛け算が定義された数」を表す型クラス Num の演算も定義されているのが自然でしょう。

このような状況を、「A は B のサブクラスである」または「B は A のスーパークラスである」と言います。

サブクラス・スーパークラスの関係は、クラスを定義する際に記述します。具体的には、 “`class`” と型クラス名の間に、 `(スーパークラス1 a, スーパークラス2 a, ...) =>` という感じで書きます。

例：

```haskell
class (Num a) => Fractional a where
  (/) :: a -> a -> a -- 割り算
  recip :: a -> a -- 逆数
  fromRational :: Rational -> a -- 有理数から変換（小数リテラルで使われる）
  recip x = 1 / x
  x / y = x * recip y
```

別の例：

```haskell
class (Eq a) => Ord a where
  compare :: a -> a -> Ordering
  (<), (<=), (>=), (>) :: a -> a -> Bool
  max, min :: a -> a -> a
```

# インスタンスの定義
次は、インスタンスの定義のやり方を見ていきます。

自分で定義した型 T を Eq クラスのインスタンスにしたい場合は次のように書きます：

```haskell
instance Eq T where
  a == b = （省略）
```

インスタンスの定義は “`instance`” キーワードから始めます。class と似ていますが、型変数の部分は実際の型で置き換えます。

メソッドの定義は、普通の関数定義と同じように書いていきます（ただし、その定義がインスタンスに属することを示すために、行頭に空白を入れてインデントしています）。ここでは関数の中身は書いていません。

`(/=)` に関しては、デフォルト実装があるので省略しています。

型パラメーターを持つ型で、パラメーターに関する制約が満たされる場合のみインスタンスが定義できる場合は、 “`instance`” と型クラス名の間に制約を書きます。
例：

```haskell
instance (Eq a) => Eq (Maybe a) where
  Just x  == Just y  = x == y  -- この右辺で、型 a が Eq のインスタンスであることを使っている
  Nothing == Nothing = True
  _       == _       = False
```
