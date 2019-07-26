### LINQでなんちゃって関数型プログラミングやってみるか

2019-07-26

CLR/H #109

Hidetsugu TAKAHASHI a.k.a manzyun

---

### はじめに - ご容赦ください

+ *就業の**合間**クオリティ*です
+ 関数型プログラミング とは という話はしません
  + ノリと雰囲気だけでやります
  + 圏論, モナドとかは触れません

---

### スピーカー紹介

高橋秀羅（たかはし　ひでつぐ）

+ しがない放浪（派遣）プログラマ
+ C♯, VB.net, PHP, Python, JavaScript, Go とか
+ 2019年にもなって、作れるWebページは2000年代なデザイン
+ [CoderDojo](https://coderdojo.jp)の駄メンター

インターネッツでは「まんじゅ（́ん｀）」というふざけた名前で出没。

---

### LINQ知ってます？

よろしければ挙手お願いします。

---

### 「女性アイドルグループ」の方の「LinQ」と思って手を挙げた方

すみませんorz

今回は「統合言語クエリ」の方です。

（Googleの検索結果で上の方に出てくるので、場を和ませるネタとして）

---

### LINQの使いこなしに自信あります？

自信に溢れた挙手をしてくださった方には、僕のバトンタッチをしていただきます（嘘）

以降の内容に問題があれば挙手の上、訂正・補足お願いします。

---

### What's LINQ

+ Language INtegrated Query
  + 統合言語クエリ
+ SQLにおけるクエリの様な書き方で、以下のデータへの問い合わせや変更ができる
  + SQLデータベース
  + ADO.NETデータセット
  + XMLドキュメントとストリーム
  + .NETコレクション内のデータ

詳しくは、[統合言語クエリ - MSDN](https://docs.microsoft.com/ja-jp/dotnet/csharp/linq/)

---

### LINQ2つの書き方

+ クエリ構文

+ メソッド構文

---

### 今回はメソッド構文推しで

+ チェーンメソッドバリバリな .NET的な気がするので
+ スピーカーの脳内アウトプットはこちらの方が比較的スムーズなので

---

### では早速LINQを……

その前に。

---

### ラムダ式

乱暴に言ってしまうと、

`\[f(x) = x\]`

の違う書き方。

`\[\lambda x.x\]`

と書ける。

---

### もうちょっとラムダ式

この例に挙げるラムダ式では、すでに演算子　「`*`」 に「乗算」の処理が割り当てられていることとして、

`\[f(x) = x * x\]` → `\[\lambda x. x * x\]`

`\[f(x, y) = x * y\]` → `\[\lambda xy.x * y\]`

とも書けなくもない。

---

### 【Appendix】ラムダ式のもっと深い話は、

ラムダ計算の深い話（簡約, チャーチ数, 不動点コンビネーター, その他、ラムダ式での計算機の実装）は、

[ラムダ計算基礎文法最速マスター - 貳佰伍拾陸夜日記](https://tarao.hatenablog.com/entry/20100208/1265605429)に任せる。

更にこれを体感するには、SchemeかLispか、LINQでやってみよう。

---

### .NETでのラムダ式

```csharp
int x => x;
string s => Hello + "s";
float fl => fl * fl

var hoge => hoge.ToString();
```

みたいに書ける **デリゲート** です。

---

### 良さが伝わりにくいデリゲートの使い方

非同期で関数やメソッド呼び出すときに使うやつ。

```csharp
public static class Program
{
    delegate void CallSay(string know_name);
    
    private void Main()
    {
        CallSay sayer = Say;  // 「Sayする何か」じゃなく「セイヤーッ！」
        sayer("manzyun");
    }
    private void Say(string name)
    {
        Console.WriteLine("hello, {0} !", name);
    }
}
```

この実行結果、分かる方。

---

### 良さが伝わりにくいデリゲートの使い方

ですが今回は「メソッド・関数の中でしか使わないメソッド・関数を定義」するために、

```csharp
public static class Program
{
    private void Main()
    {
        Func<int, int> pow = n => n * n;
        Action<string> hello = delegate(string name, int seed)
            {
                Console.WriteLine("Hello, {0}. Your number is {1} !", name, pow(seed));
            }
        Console.WriteLine("Welcome new game");
        hello("manzyun", 8);
    }
}
```

という風に使います。

---

なお、このコードの実行結果は、以下みたいになるかと。

```shell-session
Welcome new game
Hello, manzyun. Your number is 16 !
```

---

### 【Appendix】デリゲートのもっと深い話

というより「デリゲート」の部分を書いた時の参考資料は、

[デリゲート - ++C++; //未確認飛行 C](https://ufcpp.net/study/csharp/sp_delegate.html)

です。

こちらにもっと詳しいことが書いてある。

---

### LINQでデリゲートやラムダ式をどう使うか

ふつーにこう使います。

```csharp
// 3の倍数のものを抽出し、それらを更にそれぞれ2乗したものを代入
IEnumerable<int> pow_only_3 = Enumerable.Range(0, 10)
                              .Where( x => x % 3 == 0 )
                              .Select( x => x * x );
```

---

### 「え、`x` に入る値って、どこからくるの？」

```csharp
// 便宜上先ほどの続き
pow_only_3.Select( x => x / 3 );
```

乱暴に言うと、  
「 `pow_only_3` のそれぞれの要素がひとつずつ入る」

やってることはforeachとほぼ同じ。

---

### 【補足】「ラムダ式の変数は一文字じゃなければだめなのか？」

そんなことないよ！

```csharp
// 便宜上先ほどのソースの書き換え
pow_only_3.Select( shan => shan / 3 );
pow_only_3.Select( int shan => shan / 3);
```
どちらもOK。

---

### 【Appendix】変数名について

LISPという言語が「複数の文字からなる一つの変数」というアイディアをだすまでは、
プログラミングにおいても「一つの変数は一つの文字で表す」のが通例だったそうです。

ソース: 
[The Idea of Lisp](https://dev.to/ericnormand/the-idea-of-lisp)

---

### LINQ、どんな時に使うの？

気軽に `List` や `Dictionary` や `HashSet` で使えばいいと思うよ。

---

### すっごく重要な情報

その行で値が欲しいときは、最後に `ToList()`, `ToArray()`, `ToDictionary()` を実行します。

---

### 使用例1

Listに格納されているそれぞれのデータに対して処理を行いたい。

`Select` を使います。

```csharp
// 0から始まる100個のそれぞれの値に2を乗算する。
Enumerable.Range(0, 100)
          .Select( x => x * 2 );

// 5から始まる100個のそれぞれの値を、テキストフォーマットで出力する。
Enumerable.Range(5, 100)
          .Select( (x, i) =>
              Console.WriteLine(
                  "index: {0}, value:{1}", i, x
              )
          );
```

---

### 使用例2

Listに格納されているデータから、ある条件のものを抽出する。

`Where` を使います。

```csharp
// 0から始まる100個の数値の中から、3で割り切れるものを取得。
Enumerable.Range(0, 100)
          .Where( x => x % 3 == 0 );
```

---

### 【補足】Whereなどに入れる関数について

演算結果が真偽値（ `boolean` ）になる様に書ければOK。

`true` になるものが抽出されます。

```csharp
// 50超のものかつ3で割り切れる数を抽出
Enumerable.Range(0, 100)
          .Where( x => x > 50 && x % 3 == 0 );
          
// 50超のものを抽出し、その中から3で割り切れるものを抽出;
Enumerable.Range(0, 100)
          .Where( x => x > 50 )
          .Where( y => y % 3 == 0 ) // 上のWhereとの区別のため、あえて変えた。
```

---

### 使用例3

合計を求めてみる。

```csharp
// 1から始まる101個の数値を乗算する
Enumerable.Range(1, 101)
          .Aggregate( (n, m) => n * m );

// 1から始まる101個の数値の合計を求める
Enumerable.Range(1.101)
          .Aggregate( (n, m) => n + m );
```

---

### 【補足】使用例3について

大抵の集計演算はすでにLINQで準備されています。

```csharp
// 1から始まる101個の数値の合計を求める
Enumerable.Range(1, 101).Sum()
```

+ Average
+ Sum
+ Max
+ Mix
+ etc...

---

### 他にもLINQはSQLみたいな命令がいっぱいありますが……

今回はこの辺で。

詳しくは [C# の統合言語クエリ - Microsoft](https://docs.microsoft.com/ja-jp/dotnet/csharp/linq/)を参照。

なお、VB.netでも使えるので、ご安心ください。

---

### ここまでのまとめ

+ `x => ` は、リストの中の一つのデータに対する関数を書いてあげる。
+ LINQの引数（関数）の中を手続き型にしか書けそうにないなら、デリゲートを使ってしまうのも手

---

### 具体例1 「在庫管理システム」

商品の集合（ `HashSet` ）に対して、
「商品IDが `0E-` から始まる商品で、商品在庫が5未満のものを取得する」

なお、商品のデータは何らかのRDBMSに格納されているものとする。

---

### 具体例1 「在庫管理システム」

こんな感じでしょうか？
（使ったことがないのですが、LINQ to SQLを想像で使ってみました）

```csharp
// dbpath は事前にDBファイルのパスを入れているものとする
AwesomeGoods db = new AwesomeGoods(dbpath);

IQueryable<Goods> near_empty_goods =
    db.stock.Where( goods => goods.Id.IndexOf("0E-") > 0 )
            .Where( e => e.stock < 5 )
```

参考資料: [LINQ to SQL - Microsoft](https://docs.microsoft.com/ja-jp/dotnet/framework/data/adonet/sql/linq/)

---

### 具体例2「オープンデータのXMLファイルの集計」

ある技術者試験の合格情報について、XML形式で公開されている。

XMLのデータ（`/Examinees` 要素をルートとし、複数の `Examinee` 要素を持つ)から、
2019年9月 （`Examinee/Date`）に、
ネットワーク技術者試験(`Examinee/Subject` の値が `Network` )に
合格した(`Examinee/Clear` の値が `OK` )ものを抽出し、その合計を求めよ。

---

### 具体例2「オープンデータのXMLファイルの集計」

LINQ to XMLでは、XPathも使うと、とても便利。

```csharp
using System.Xml.Linq;
using System.Xml.XPath;

/* ...中略... */

XDocument examinees = XDocument.Load(@"path/to/the/directory/2019.xml);

int passed_examinees_count =
    examinees.XPathSelectElements("/Examinees/Examinee")
    .Where( examinee => examinee.Date == "2019-09" )
    .Where( examinee => examinee.Subject == "Network" )
    .Where( examinee => examinee.Clear == "OK" )
    .Count()
```

実務でLINQ to XMLを使いましたが、うろ覚え……。

参考資料:[LINQ To XML とXPathを利用したXMLドキュメント検索 (C#プログラミング) - iPentec](https://www.ipentec.com/document/csharp-linq-to-xml-and-xpath-xml-retrieve)

---

### 【Appendix】Functional Reactive Programmingもやってみる。

[Reactive Extension](https://github.com/dotnet/reactive)というライブラリを使います。

Unity Game EngineやGUIプログラミングで大活躍のはず。
イベントやストリームデータに対してLINQが使えます。

```csharp
/* ...前略 */

// 0から始まるそれぞれ1つの値を持つ10個のイベントから、偶数のイベントだけ取り出して、ダイアログ出力。
Observable.Range(0, 10).Where( x => x % 2 == 0 )
                       .Subscribe(MessageBox.Show("偶数キター（°∀°）－！"));

// 最初から5番目以降のイベント以降からダイアログを出力。
Observable.Range(0, 10).Take(5).Subscribe(MessageBox.Show("おくれてじゃじゃじゃじゃーん");
```

参考資料: [5000兆年ぶりにReactive Extensions再入門 - こっちみないで(´・ω・｀)](http://kmycode.hatenablog.jp/entry/2017/09/30/181312)

---

### 言いたかったこと

+ もう2019年ですよ？
  + 2008年から導入されているこの機能を使わずに、いつ使うの？
+ 「関数型パラダイムとかなんだよ！？　俺はオブジェクト指向で忙しい！」
  + LINQで雰囲気はつかめるぞ。
  + Reactive Extensionで、GUIプログラミングでも応用可能
+ LINQの感覚がPowerShellにも活かせる？
  + PowerShell編に続く……

---

### Thank's for listening

+ このスライドは[Git Pitch](https://gitpitch.com/)で作成しました。
+ 再配布は [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/deed.ja)に基づいて許可いたします。
