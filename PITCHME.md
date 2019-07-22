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

上記リンク先の文書の中身にお任せ。

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

なお、このコードの実行結果は、以下みたいになるかと。

```
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

### LINQ実践例

+ Select
+ Where -> Select
+ 思いついたもの色々

---

---

### まとめ

+ もう2019年ですよ？
  + 2008年から導入されているこの機能を使わずに、いつ使うの？
+ 「関数型パラダイムとかなんだよ！？　俺はオブジェクト指向で忙しい！」
  + LINQで雰囲気はつかめるぞ。
+ LINQの知識がPowerShellにも活かせる？
  + PowerShell編に続く……
  
### Thank's for listening

+ このスライドは[Git Pitch](https://gitpitch.com/)で作成しました。
+ 再配布は [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/deed.ja)に基づいて許可いたします。
