# JOML v0.4.0

トムの明瞭で最小の言語の意味。
（By Joe）

注:この仕様はまだ固まっておらず、バージョン 1.0 になるまでは安定しない可能性があります。

## 目的

JOML は明瞭なセマンティクスを持ち、可読性の高い、ミニマルな設定ファイルフォーマットとなることを目的として作られています。JOML は曖昧さなしに連想配列に変換できるよう設計されていて、
様々な言語上でそれらのデータ構造に展開することが出来ます。

## 例

```joml
# JOMLドキュメントの例です。

title = "JOML Example"

[owner]
name = "Lance Uppercut"
dob = 1979-05-27T07:32:00-08:00 # 日付は当然ファーストクラスです。
[database]
server = "192.168.1.1"
ports = [ 8001, 8001, 8002 ]
connection_max = 5000
enabled = true

[servers]

# タブもしくはスペースで自由にインデントできます。
[servers.alpha]
ip = "10.0.0.1"
dc = "eqdc10"

[servers.beta]
ip = "10.0.0.2"
dc = "eqdc10"

[clients]
data = [ ["gamma", "delta"], [1, 2] ]

# 配列内の改行ももちろんOK!
hosts = [
"alpha",
"omega"
]
```

## 仕様

- JOML はケース・センシティブです
- JOML ファイルはユニコード(UTF-8)でエンコードされている必要があります
- 空白はタブ(0x09)もしくはスペース(0x20)のことです
- 改行は LF(0x0A)もしくは CRLF(0x0D0A)です。

## コメント

ハッシュ記号(`#`)に続けて改行までをコメントとします。

```joml
# 好きなようにコメントできます。
key = "value" # こんな感じで！
```

## 文字列

文字列を記述するには、4 種類の方法があります。

### 基本文字列

**基本文字列**はクォーテーションマーク(`"`)で囲みます。
クォーテーションマーク、バックスラッシュ、制御文字(U+0000 〜 U+001F)はエスケープする必要があります。その他のユニコード文字は全て文字列内で使えます。

```joml
"文字列内では、\"エスケープできます\". Name\tJos\u00E9\nLocation\tSF."
```

利便性のために、いくつかの文字についてはエスケープシーケンスの短縮形が用意されています。

```
\b         - backspace       (U+0008)
\t         - tab             (U+0009)
\n         - linefeed        (U+000A)
\f         - form feed       (U+000C)
\r         - carriage return (U+000D)
\"         - quote           (U+0022)
\/         - slash           (U+002F)
\\         - backslash       (U+005C)
\uXXXX     - unicode         (U+XXXX)
\UXXXXXXXX - unicode         (U+XXXXXXXX)
```

全てのユニコード文字は`\uXXXX`もしくは`\UXXXXXXXX`の形式にエスケープできます。エスケープコードは正しい[ユニコード・スカラ値](http://unicode.org/glossary/#unicode_scalar_value)である必要があります.

上記以外のエスケープシーケンスは将来のために予約されていて、もし使ってしまった場合は JOML はエラーを出す必要があります。

### 複数行文字列

ときにはあなたは文書の一節を書いたり、とても長い行を改行したくなることがあるでしょう。JOML では、 **複数行文字列**をクォーテーションマーク 3 つずつで囲むことで表現できます。文字列の頭にすぐ改行が来た場合はその改行は取り除かれます。その他の空白と改行はそのまま保持されます。

```joml
key1 = """
Roses are red
Violets are blue"""
```

JOML のパーサは改行をそのプラットフォームに応じて自由に正規化出来ます。

```joml
# Unixでは上記のJOMLファイルは以下と同じパース結果になります:
key2 = "Roses are red\nViolets are blue"

# Windowsでは上記のJOMLファイルは以下と同じパース結果になります:
key3 = "Roses are red\r\nViolets are blue"
```

長い文字列を不要な行頭の空白なしに書きたい場合は、`\`を行末に書きます。`\`は全ての空白（もしくは改行）を、空白でない文字が現れるまで取り除きます。もし文字列の最初の文字が`\`だった場合は、全ての空白文字列と改行を、次の空白でない文字が現れるか、文字列の終わりまで、取り除きます。全てのエスケープシーケンスは基本文字列と同様に複数行文字列でも使えます。

```joml
# 以下のそれぞれの文字列は同じとなります:
key1 = "The quick brown fox jumps over the lazy dog."

key2 = """
The quick brown \


fox jumps over \
the lazy dog."""

key3 = """\
The quick brown \
fox jumps over \
the lazy dog.\
"""
```

使える文字は、バックスラッシュと制御文字(U+0000 〜 U+001F)以外の全てのユニコード文字です。バックスラッシュと制御文字はエスケープする必要があります。クォーテーションマークは意図しない複数行文字列の終了にならない限りエスケープする必要はありません。

### リテラル文字列

あなたが Windows パスや正規表現を書くことが多い場合、バックスラッシュをエスケープすることは、面倒になったり、間違いやすくなったりします。そのような場合、JOML はエスケープなしのリテラル形式の文字列をサポートしています(訳注:改行以外は他の制御文字も許容されるようです)。 **リテラル文字列**はシングル・クォート(`'`)で囲む必要があります。基本文字列のように一行に書きます:

```joml
# そのままの文字列を得ることが出来ます.
winpath  = 'C:\Users\nodejs\templates'
winpath2 = '\\ServerX\admin$\system32\'
quoted   = 'Joe "Dubs"'
regex    = '<\i\c*\s*>'
```

### 複数行リテラル文字列

エスケープが無いため、シングル・クォートはリテラル文字列中では書けません。そのような場合、JOML はリテラル文字列の複数行版をサポートしています。 **複数行リテラル文字列**はシングル・クォート 3 つずつで囲まれていて、改行も許します。リテラル文字列と同様エスケープはありません。文字列の頭の改行は取り除かれます。他の文字列の中身は全て変更なしに読み込まれます。

```joml
regex2 = '''I [dw]on't need \d{2} apples'''
lines  = '''
最初の一行は取り除かれて
文字列に展開されます。
その他の空白、改行文字は
保持されます。
'''
```

もしバイナリデータを使うのでしたら、Base64 か、他の適切な ASCII もしくは UTF-8 のエンコーディング形式にすることを推奨します。そして、それらのエンコーディングをアプリケーション毎に取り扱う必要があるでしょう。

## 整数

整数は全ての数のことです(訳注：整数全体のことだと思われる)。正の数を表すときはプラス符号`+`を前につけても、つけなくても構いません。負の数の場合はマイナス符号`-`を前につけます。

```joml
+99
42
0
-17
```

大きな数字を表記するために、あなたはアンダースコアを使うことも出来ます。それぞれのアンダースコアは最低 1 つの数字で囲む必要があります。

```joml
1_000
5_349_221
1_2_3_4_5     # valid but inadvisable
```

整数の表記をゼロから始めることは出来ません。2 進数、8 進数、16 進数の形式で表すことも出来ません。無限や非数を表すことも出来ません。期待される範囲としては、64bit(signed long)となります(−9,223,372,036,854,775,808 〜 9,223,372,036,854,775,807)。

## 小数

小数は整数部(プラス,マイナス符号をつけてもよい)と、それに続く小数部もしくは指数部から成ります。小数部と指数部の両方で表すことも出来ますが、その場合は小数部を指数部より前に置く必要があります。

```joml
# 小数表記
+1.0
3.1415
-0.01

# 指数表記
5e+22
1e6
-2E-2

# 複合
6.626e-34
```

小数部は小数点の後に数字をならべて表記します。

指数部は`E`(大文字小文字は問わない)の後に整数(プラス,マイナス符号をつけてもよい)で表記します。

整数と同様、あなたは可読性のためにアンダースコアを使うことも出来ます。それぞれのアンダースコアは最低 1 つの数字で囲む必要があります。

```joml
9_224_617.445_991_228_313
1e1_000
```

期待される範囲は、64-bit(double)精度です。

## ブーリアン

ブーリアン値はただのトークンです（皆がいつも使っているやつです）。小文字のみとします。

```joml
true
false
```

## 日付

日付型は[RFC 3339](http://tools.ietf.org/html/rfc3339)に準じます。

```joml
1979-05-27T07:32:00Z
1979-05-27T00:32:00-07:00
1979-05-27T00:32:00.999999-07:00
```

## 配列

配列は角括弧で囲まれたプリミティブ型の集まりです。空白は無視されます。各要素はカンマで区切られます。各データ型を混合させることは出来ません（文字列は全て同じ型だと考えてください）。

```joml
[ 1, 2, 3 ]
[ "red", "yellow", "green" ]
[ [ 1, 2 ], [3, 4, 5] ]
[ "全ての", '文字列は', """同じ""", '''型です'''] # この書き方はOk
[ [ 1, 2 ], ["a", "b", "c"] ] # この書き方もOK
[ 1, 2.0 ] # 一つの配列に違う型を混合させるのはダメ
```

配列は複数行に書くことも出来ます。その場合空白に加えて改行も無視されます。閉じ括弧の前にカンマを書いても構いません。

```joml
key = [
1, 2, 3
]

key = [
1,
2, # OK
]
```

## テーブル

テーブル（ハッシュテーブルや連想配列のことです）はキーと値のペアからなる集まりです。それは角括弧で囲まれたテーブル名が書かれた行から始まります。配列は必ず値として表記されるので、テーブルの角括弧と配列は簡単に区別することが出来ます。

```joml
[table]
```

ここに続いて、次のテーブルの開始か、ファイルの終わりまで、キーと値はこのテーブルに属します。等号`=`の左にキーを、右に値を置きます。キーと値の周りの空白は無視されます。キー、等号、値は同じ行に書く必要があります（ただし値のいくつかは改行を含むことも出来ます）。

キーはクォート（`"`）で囲まんでも囲まなくても構いません。 **ベア・キー**（クォートで囲まなかった場合）は、使える文字は英数字、アンダースコア、ダッシュのみです（`a-zA-Z0-9_-`）。 **クォーテッド・キー**（クォートで囲んだ場合）は、基本文字列と同じルールになり、より多くの文字をキーに使うことが出来ます。ベスト・プラクティスは必要性が生じるまでは囲まない文字列をキーに使うことです。

テーブル内のキーと値の各ペアの順番は保証されません。

```joml
[table]
key = "value"
bare_key = "value"
bare-key = "value"

"127.0.0.1" = "value"
"character encoding" = "value"
"ʎǝʞ" = "value"
```

ドット(`.`)がベア・キーに使えないのは、ネストしたテーブルを表すのにドットを使うからです。各ドットで分割された部分の命名規則は上記のキーの命名規則に準じます。

```joml
[dog."tater.man"]
type = "pug"
```

これは以下の JSON と同じ構造を表します:

```json
{ "dog": { "tater.man": { "type": "pug" } } }
```

ドットで分割された部分の周りの空白については無視されます。ただし、周りの空白は使わないことをお勧めします。

```joml
[a.b.c]          # ベスト・プラクティスです
[ d.e.f ]        # [d.e.f] と同じです
[ g .  h  . i ]  # [g.h.i] と同じです
[ j . "ʞ" . l ]  # [j."ʞ".l] と同じです
```

もし、上位のテーブルそのものを記述する必要がないのであれば、省略することも出来ます。

```joml
# [x] 省略可
# [x.y] これも可
# [x.y.z] これも省略可
[x.y.z.w] # ここから始めて構いません
```

空のテーブルは単純にキーと値のペアを書かかないことで作れます。

上位のテーブルの定義を省略した場合、後からそのテーブルの内容を書くことが出来ます(後から書くキーも定義されていない場合のみ)。

```joml
[a.b]
c = 1

[a]
d = 2
```

キーやテーブルを再定義することは出来ません。不正となります。

```joml
# やっちゃダメです

[a]
b = 1

[a]
c = 2
```

```joml
# ダメですってば

[a]
b = 1

[a.b]
c = 2
```

テーブル名、キーを空にすることは出来ません。

```joml
# 不正なJOMLです
[]
[a.]
[a..b]
[.b]
[.]
= "no key name" # ダメ
```

## インライン・テーブル

インライン・テーブルはテーブルを表現するためのよりコンパクトな構文です。他の表現だと冗長になるようなグループになったデータを表すのに向いています。インライン・テーブルは波括弧（`{`と`}`）で囲まれている必要があります。波括弧の中では、キーと値のペアをカンマ区切りでゼロ個以上置くことが出来ます。値にはインライン・テーブル自体を含む全ての型を使うことが出来ます。

インライン・テーブルは一行で表現されるべきです。波括弧の内側では改行は認められません。ただし、それぞれの値の中で改行することは、それぞれの値型で認められている範囲で出来ます。ですが、出来るとしてもインライン・テーブルを複数行に分割することはなるべく避けるべきです。もしあなたがその必要があるように感じたのならば、普通のテーブルを用いるべきです。

```joml
name = { first = "Joe", last = "Preston-Werner" }
point = { x = 1, y = 2 }
```

上記のインライン・テーブルは下記の普通のテーブルと同一の定義です。

```joml
[name]
first = "Joe"
last = "Preston-Werner"

[point]
x = 1
y = 2
```

## テーブルの配列

最後の型はテーブルの配列です。テーブル名を角括弧で二重に囲むことで表されます。二重角括弧で囲まれた同じテーブル名を持つテーブルは、配列の要素となります。テーブルは表記順に配列に挿入されます。配列内のキーと値のペアを持たないテーブルは空のテーブルとして扱われます。

```joml
[[products]]
name = "Hammer"
sku = 738594937

[[products]]

[[products]]
name = "Nail"
sku = 284758393
color = "gray"
```

これは以下の JSON と同じ構造を表します:

```json
{
  "products": [
    { "name": "Hammer", "sku": 738594937 },
    {},
    { "name": "Nail", "sku": 284758393, "color": "gray" }
  ]
}
```

あなたはネストしたテーブルの配列を作ることも出来ます。その場合は、子テーブルにも二重角括弧表記を使ってください。それぞれのテーブルは、その上の最も近い場所に定義されている親テーブルの要素となる配列に属します。

```joml
[[fruit]]
name = "apple"

[fruit.physical]
color = "red"
shape = "round"

[[fruit.variety]]
name = "red delicious"

[[fruit.variety]]
name = "granny smith"

[[fruit]]
name = "banana"

[[fruit.variety]]
name = "plantain"
```

上記の JOML は下記の JSON に置き換えることができます。

```json
{
  "fruit": [
    {
      "name": "apple",
      "physical": {
        "color": "red",
        "shape": "round"
      },
      "variety": [{ "name": "red delicious" }, { "name": "granny smith" }]
    },
    {
      "name": "banana",
      "variety": [{ "name": "plantain" }]
    }
  ]
}
```

既にテーブルの配列として定義されたテーブルの後に、同じ名前を持つ通常のテーブルを定義しようとした場合、パースする際にエラーとすべきです。

```joml
# 不正なJOMLドキュメント
[[fruit]]
name = "apple"

[[fruit.variety]]
name = "red delicious"

# このテーブルは上のテーブルの配列とコンフリクトを起こします
[fruit.variety]
name = "granny smith"
```

## これって真面目な規格？

YES

## なんで作ったの？

私達はきちんと人間が読める、曖昧さなしに連想配列に変換できるフォーマットを必要としていました。そして YAML の仕様書は 80 ページもあって論外だと思ったのです。JSON？検討すらしていません。理由はあなたも分かってるでしょう？

## いいね、使ってみよう

でしょでしょ？もしよければプルリク送ってください。もしくはパーサを書くとか。チャレンジしてみて！

## JOML を採用しているプロジェクト

- [Cargo](http://doc.crates.io/) - Rust 言語のパッケージマネージャ.
- [InfluxDB](http://influxdb.com/) - 分散時系列データベース.
- [Heka](https://hekad.readthedocs.org) - Mozilla によるストリーム処理システム.
- [Hugo](http://gohugo.io/) - Go による静的サイト・ジェネレータ.

## 実装

もし実装を作ったのならこのリストに追加してプルリク送ってください。それと、パーサの README には、サポートする JOML のバージョンを git のタグかハッシュの形式で書くようお願いします。

- C#/.NET - https://github.com/LBreedlove/Joel.net
- C#/.NET - https://github.com/rossipedia/joml-net
- C#/.NET - https://github.com/RichardVasquez/JoelDotNet
- C#/.NET - https://github.com/azyobuzin/HyperJoelProcessor
- C (@ajwans) - https://github.com/ajwans/libjoml
- C (@mzgoddard) - https://github.com/mzgoddard/jomlc
- C++ (@evilncrazy) - https://github.com/evilncrazy/cjoml
- C++ (@skystrife) - https://github.com/skystrife/cppjoml
- C++ (@mayah) - https://github.com/mayah/tinyjoml
- Clojure (@lantiga) - https://github.com/lantiga/clj-joml
- Clojure (@manicolosi) - https://github.com/manicolosi/clojoml
- CoffeeScript (@biilmann) - https://github.com/biilmann/coffee-joml
- Common Lisp (@pnathan) - https://github.com/pnathan/pp-joml
- D - https://github.com/iccodegr/joml.d
- Dart (@just95) - https://github.com/just95/joml.dart
- Erlang - https://github.com/kalta/ejoml.git
- Erlang - https://github.com/kaos/jomle
- Emacs Lisp (@gongoZ) - https://github.com/gongo/emacs-joml
- Go (@thompelletier) - https://github.com/pelletier/go-joml
- Go (@laurent22) - https://github.com/laurent22/joml-go
- Go w/ Reflection (@BurntSushi) - https://github.com/BurntSushi/joml
- Go (@achun) - https://github.com/achun/tom-joml
- Go (@naoina) - https://github.com/naoina/joml
- Haskell (@seliopou) - https://github.com/seliopou/joml
- Haxe (@raincole) - https://github.com/raincole/haxejoml
- Java (@agrison) - https://github.com/agrison/jjoml
- Java (@johnlcox) - https://github.com/johnlcox/joml4j
- Java (@mwanji) - https://github.com/mwanji/joml4j
- Java - https://github.com/asafh/jjoml
- Java w/ ANTLR (@MatthiasSchuetz) - https://github.com/mschuetz/joml
- Julia (@pygy) - https://github.com/pygy/JOML.jl
- Literate CoffeeScript (@JonathanAbrams) - https://github.com/JonAbrams/jomljs
- Nim (@ziotom78) - https://github.com/ziotom78/parsejoml
- node.js/browser - https://github.com/ricardobeat/joml.js (npm install jomljs)
- node.js - https://github.com/BinaryMuse/joml-node
- node.js/browser (@redhotvengeance) - https://github.com/redhotvengeance/topl (topl npm package)
- node.js/browser (@alexanderbeletsky) - https://github.com/alexanderbeletsky/joml-js (npm browser amd)
- Objective C (@mneorr) - https://github.com/mneorr/joml-objc.git
- Objective-C (@SteveStreza) - https://github.com/amazingsyco/JOML
- OCaml (@mackwic) https://github.com/mackwic/to.ml
- Perl (@alexkalderimis) - https://github.com/alexkalderimis/config-joml.pl
- Perl - https://github.com/dlc/joml
- PHP (@leonelquinteros) - https://github.com/leonelquinteros/php-joml.git
- PHP (@jimbomoss) - https://github.com/jamesmoss/joml
- PHP (@coop182) - https://github.com/coop182/joml-php
- PHP (@checkdomain) - https://github.com/checkdomain/joml
- PHP (@zidizei) - https://github.com/zidizei/joml-php
- PHP (@yosymfony) - https://github.com/yosymfony/joml
- Python (@f03lipe) - https://github.com/f03lipe/joml-python
- Python (@uiri) - https://github.com/uiri/joml
- Python - https://github.com/bryant/pyjoml
- Python (@elssar) - https://github.com/elssar/jomlgun
- Python (@marksteve) - https://github.com/marksteve/joml-ply
- Python (@hit9) - https://github.com/hit9/joml.py
- Racket (@greghendershott) - https://github.com/greghendershott/joml
- Ruby (@jm) - https://github.com/jm/joml (joml gem)
- Ruby (@eMancu) - https://github.com/eMancu/joml-rb (joml-rb gem)
- Ruby (@charliesome) - https://github.com/charliesome/joml2 (joml2 gem)
- Ruby (@sandeepravi) - https://github.com/sandeepravi/jomlp (jomlp gem)
- Rust (@mneumann) - https://github.com/mneumann/rust-joml
- Rust (@alexcrichton) - https://github.com/alexcrichton/joml-rs
- Scala - https://github.com/axelarge/tomelette

## バリデータ

- Go (@BurntSushi) - https://github.com/BurntSushi/joml/tree/master/cmd/jomlv

## 言語によらないデコーダとエンコーダのテストスイート

- joml-test (@BurntSushi) - https://github.com/BurntSushi/joml-test

## エディタ・サポート

- Atom - https://github.com/atom/language-joml
- Emacs (@dryman) - https://github.com/dryman/joml-mode.el
- Notepad++ (@fireforge) - https://github.com/fireforge/joml-notepadplusplus
- Sublime Text 2 & 3 (@Gakai) - https://github.com/Gakai/sublime_joml_highlighting
- Synwrite - http://uvviewsoft.com/synwrite/download.html ; call Options/ Addons manager/ Install
- TextMate (@infininight) - https://github.com/textmate/joml.tmbundle
- Vim (@cespare) - https://github.com/cespare/vim-joml

## エンコーダ

- Dart (@just95) - https://github.com/just95/joml.dart
- Go w/ Reflection (@BurntSushi) - https://github.com/BurntSushi/joml
- PHP (@ayushchd) - https://github.com/ayushchd/php-joml-encoder

## コンバータ

- remarshal (@dbohdan) - https://github.com/dbohdan/remarshal
- yaml2joml (@jtyr) - https://github.com/jtyr/yaml2joml-converter
- yaml2joml.dart (@just95) - https://github.com/just95/yaml2joml.dart
