---
layout: page
title: 文字列処理
---
# 文字列の表記（文字列リテラル）

プログラム中に直接文字列を埋め込む場合は、引用符で囲みます。引用符の種類はいくつかあります。

- 一重引用符 `'hoge'`
- 二重引用符 `"hoge"`
- バッククォート（テンプレート文字列） `` `My name is ${name}.` ``
    - 他の表記と異なり、 `${}` でJavaScriptの式を途中に挿入できる。
    - ECMAScript 6 で導入された新しい機能である。

引用符自体を文字列の中に埋め込むには、バックスラッシュを使ったエスケープシーケンスを使います。
```
> console.log("\"")
"
undefined
> console.log('\'')
'
undefined
> console.log(`\``)
`
undefined
```

バックスラッシュを文字列の中に埋め込みたいときは、バックスラッシュを2つ重ねます。
```
> console.log("\\")
\
undefined
```

改行文字を埋め込むには、 `\n` というエスケープシーケンスを使う必要があります。ただし、テンプレート文字列の場合は、複数行にわたって文字列リテラルを書くこともできます。
```javascript
// hello.js
console.log("Hello\nworld")
console.log(`Hello
world`)
```
```shell
$ nodejs hello.js
Hello
world
Hello
world
```

# 文字列の操作
文字列は、 + 演算子で連結することができます。
```
> "Hello " + "world"
Hello world
```

メソッド呼び出しの構文を使って、他にも様々な操作を行えます。
```
> "Hello".repeat(5) // 同じ文字列を繰り返す
'HelloHelloHelloHelloHello'
> "Hello".slice(1,4) // 部分文字列の切り出し
'ell'
> "1,2,3".split(",") // 文字列を区切り文字で分割（配列が得られる）
[ '1', '2', '3' ]
.substring
> "Hello world!".toLowerCase() // 小文字に変換
'hello world!'
> "Hello world!".toUpperCase() // 大文字に変換
'HELLO WORLD!'
> " Goodbye world! \n".trim() // 先頭と末尾の空白・改行文字を取り除く
'Goodbye world!'
.replace
> "Hello world!".indexOf("l") // 部分文字列を先頭から探し、その位置を返す（見つからなかった場合は -1 を返す）
2
> "Hello world!".lastIndexOf("l") // 部分文字列を末尾から探し、その位置を返す（見つからなかった場合は -1 を返す）
9
> "Hello world!".includes("!") // 部分文字列を含むかどうかを判定する。 .indexOf(...) !== -1 とほぼ等価。
true
> "Hello world!".startsWith("Hello") // 文字列の先頭が
true
> "Hello world!".endsWith("!")
true
```

## 文字単位のアクセス
文字列の構成要素である、個別の文字にアクセス方法には以下のようなものがあります：
```
> str = "A\u03B1\u3042"
'Aαあ'
> str.length // 長さ（文字数）
3
> str.charAt(1) // n文字目を取得（取得結果は文字列）
'α'
> str.charAt(2)
'あ'
> str.charCodeAt(1) // n文字目の文字コード (Unicode) を取得（取得結果は整数）
945
> str.charCodeAt(2)
12354
> str.charCodeAt(2).toString(16)
'3042'
> String.fromCharCode(0x03B1, 0x3042) // 文字コードから文字列を作る
'αあ'
```

# 発展：UnicodeとUTF-16
JavaScriptの文字列は、実際には16ビット整数の列です。Unicodeの、UTF-16を使って、文字を整数に対応させます。ひらがなの「あ」はUnicodeだとU+3042に対応します：

さっき「文字数」と書いたのは正確には「UTF-16で数えた時の長さ」で、扱う文字によっては「文字数」と .length の値が一致しなくなります。

例えば、絵文字を含む文字列を扱ってみましょう。絵文字「🐱」のUnicodeコードポイントはU+1F431です。\u記法で5桁以上のコードポイントを埋め込むには、コードポイントを波カッコで括ります。
```
> str = "A\u3042\u{1F431}"
'Aあ🐱'
> str.length
4
```


U+1F431は、UTF-16でコードすると D83D DC31 というふうに、16ビット整数2個を使って表すことになります。 このように、5桁以上のコードポイントで表されるUnicode文字をUTF-16で符号化した際に使う2個の整数値を、サロゲートペアと呼びます。
```
> str.charCodeAt(0).toString(16)
'41'
> str.charCodeAt(1).toString(16)
'3042'
> str.charCodeAt(2).toString(16) // サロゲートペアの1個目
'd83d'
> str.charCodeAt(3).toString(16) // サロゲートペアの2個目
'dc31'
```

サロゲートペアから元々のコードポイントを取り出したい場合に、直接計算することもできますが、ECMAScript標準に用意された.codePointAt関数を使うこともできます：
```
> str.codePointAt(1).toString(16) // サロゲートペアでない位置を指定した場合は、UTF-16の値がそのまま返る
'3042'
> str.codePointAt(2).toString(16) // サロゲートペアの位置を指定した場合は、コードポイントが返る
'1f431'
```
.codePointAt関数に与える「文字列中の位置」は、.charCodeAt関数と同様に、UTF-16単位で数えます。
コードポイントから文字列を作る場合、String.fromCharCode関数は4桁以下のコードポイントしか扱えませんが、String.fromCodePoint関数を使うと5桁以上のコードポイントも扱うことができます。
```
> String.fromCodePoint(0x1f431)
'🐱'
```
