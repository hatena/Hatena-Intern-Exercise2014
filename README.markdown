Hatena-Intern-Exercise2014
========================================

以下基本的な教材は Hatena::Textbook など

# Perl 課題

## 基礎編

LTSV (Labeled Tab-separated Values) とはラベル付きのTSVフォーマットです。
LTSVの1レコードは、`label:value` という形式で表されたラベル付きの値がタブ文字区切りで並びます。

以下に LTSV の例を示します。

* ltsv.log

```
host:127.0.0.1	user:frank	epoch:1372694390	req:GET /apache_pb.gif HTTP/1.0	status:200	size:2326	referer:http://www.hatena.ne.jp/
host:127.0.0.1	user:john	epoch:1372794390	req:GET /apache_pb.gif HTTP/1.0	status:200	size:1234	referer:http://b.hatena.ne.jp/hotentry
host:127.0.0.1	user:-	epoch:1372894390	req:GET /apache_pb.gif HTTP/1.0	status:302	size:9999	referer:http://www.example.com/start.html
```

例えば、1レコード目の host の値は 127.0.0.1 であり、2レコード目の referer の値は http://b.hatena.ne.jp/hotentry になります。LTSV についてより詳しくは、以下を参照して下さい。

* http://blog.stanaka.org/entry/2013/02/05/214833
* http://ltsv.org/

### 課題 Perl-1

上記の LTSV の1レコードを表す Log クラスを実装して下さい。
(Perl のオブジェクト指向については Hatena::Textbook などを参考にして下さい。)

Log クラスは以下のメソッドを持ちます。
* `req` の値に含まれている HTTP メソッド名を返す `method` メソッド
* `req` の値に含まれているリクエストパスを返す `path` メソッド
* `req` の値に含まれているプロトコル名を返す `protocol` メソッド
* `host` と `req` の値からリクエストされた uri を組み立てて返す `uri` メソッド
* `epoch` が表している日付を `YYYY-MM-DDThh:mm:ss` というフォーマットの文字列に変換して返す `time` メソッド
    * 日付を扱うモジュールを用いてかまいません
    * タイムゾーンは GMT として下さい


* main.pl

```perl
use strict;
use warnings;

use Log;

my $log = Log->new(
    host    => '127.0.0.1',
    user    => 'frank',
    epoch   => '1372694390',
    req     => 'GET /apache_pb.gif HTTP/1.0',
    status  => '200',
    size    => '2326',
    referer => 'http://www.hatena.ne.jp/',
);
print $log->method . "\n";
print $log->path . "\n";
print $log->protocol . "\n";
print $log->uri . "\n";
print $log->time . "\n";
```

```shell
$ perl -Ilib main.pl
```

* 出力

```perl
$ perl -Ilib main.pl
GET
/apache_pb.gif
HTTP/1.0
http://127.0.0.1/apache_pb.gif
2013-07-01T15:59:50
```

また、後述するリポジトリに含まれているテストを実行して、課題が正しく動作していることを確認してください。

* Perl のテストの実行に関しては、[Hatena-Textbook](https://github.com/hatena/Hatena-Textbook/blob/master/oop-for-perl.md#-58) を参照して下さい。

### 課題 Perl-2

上記 LTSV フォーマットのログファイルを読み込み、以下のLog オブジェクトの配列を返すパーザーを実装してください

* main2.pl

```perl
use strict;
use warnings;

use Data::Dumper;

use Parser;

my $parser = Parser->new( filename => 'log.ltsv' );
warn Dumper $parser->parse;
```

```shell
$ perl -Ilib main2.pl
```

* 出力フォーマット

```perl
$VAR1 = [
          bless( {
                   'epoch' => '1372694390',
                   'req' => 'GET /apache_pb.gif HTTP/1.0',
                   'status' => '200',
                   'user' => 'frank',
                   'referer' => 'http://www.hatena.ne.jp/',
                   'size' => '2326',
                   'host' => '127.0.0.1'
                 }, 'Log' ),
          ...
        ];
```

実装に関しては、以下の条件を守って下さい。

* LTSV をパースするCPANモジュールを用いないこと
* LTSVでは値が定義されていない場合は`host:-`のように値に`-`が入ります。値が定義されていなかった場合には、そのラベルをハッシュリファレンスに含めないようにして下さい
* map や grep といった関数を積極的に利用するとよいでしょう (必須ではない)


課題1と同じように、テストを実行して正しく動作していることを確認して下さい


* Perl でファイルを読み込む方法はいろいろあって混乱しやすいため、以下にサンプルコードを載せておきます

```perl
open my $fh, '<', 'log.ltsv' or die $!;
my $line = <$fh>; # スカラーコンテキストなので一行読み込む
my @lines = <$fh>; # リストコンテキストなので(残りの)すべての行を配列として読み込む
print $line;
print "----\n";
print @lines;
```

### 課題 Perl-3

Log を集計する LogCounter を実装して下さい
LogCounter は、HTTP サーバーエラー (500番台) の数を数える `count_error` とユーザーごとにログをまとめる `group_by_user` メソッドを持ちます。

* main3.pl

```perl
my $parser = Parser->new( filename => 'log.ltsv' );
my $counter = LogCounter->new($parser->parse);
print 'total error size: ' . $counter->count_error . "\n";
print Dumper $counter->group_by_user;
```

```shell
$ perl -Ilib main3.pl
```

* 出力

```perl
$VAR1 = {
          'guest' => [
                       bless( {
                                'epoch' => '1372894390',
                                'req' => 'GET /apache_pb.gif HTTP/1.0',
                                'status' => '503',
                                'referer' => 'http://www.example.com/start.html',
                                'size' => '9999',
                                'host' => '127.0.0.1'
                              }, 'Log' )
                     ],
          'frank' => [
                       bless( {
                                'epoch' => '1372694390',
                                'req' => 'GET /apache_pb.gif HTTP/1.0',
                                'status' => '200',
                                'user' => 'frank',
                                'referer' => 'http://www.hatena.ne.jp/',
                                'size' => '2326',
                                'host' => '127.0.0.1'
                              }, 'Log' ),
                       ...
                     ],
          ...
        };
```

* CPAN には List::Util や List::UtilsBy といった便利なモジュールがありますが、今回は利用せずに実装してみましょう
* user の値がない(値が `-` である)場合は `guest` という名前にして集計して下さい
* いろいろな集計処理が考えられます。余裕があれば自分で考えて実装し、テストも追加してみて下さい。



## 応用編

課題3までで作った LTSV パーザーを用いてこれまでに使ったデータより大きなデータから可視化を行い、標準出力にダイアグラム (グラフ) を出力してください。

サンプルデータは sample_data/log.ltsv にあるのでこれを利用してください。

様々なデータの可視化が考えられます。

例:

* 日毎のリクエスト URI の分布
* リクエスト URI 毎のステータスコード (`status`) の分布

ダイアグラムの例:

```
/
---:        100       200
200:=================*
403:==*
404:===*
500:=*

/bookmark
---:        100       200
200:=================*
403:==*
404:=======*
500:===*
```

上記の例に限らず、自由に可視化の対象・方法を考えてみてください。

## JavaScript 課題

この JavaScript 課題は、JavaScript の言語コア部分や、簡単な DOM 操作 (DOM 操作は、HTML 文書の構造などをプログラムで変更するもので、web 開発においては主にユーザーインターフェイスの動的な変更のために使用されます) といった基本的な部分の理解を確かめるためのものです。
JavaScript の処理系として [Node.js](http://nodejs.org/) なども存在しますが、ここでは
[Firefox](http://www.mozilla.jp/firefox/) や [Chrome](http://www.google.co.jp/intl/ja/chrome/browser/) といった最近のブラウザを使用するようにしてください。
(クロスブラウザについて考慮する必要はありません。 Firefox や Chrome といった最近のブラウザの最新バージョンで動作するようにしてください。)

課題 JS-1 から JS-3 までは、必ず行ってください。
これらの課題では、JavaScript を使って Web サーバーのアクセスログ (を模した文字列) を解析し、HTML ドキュメント上にアクセスログの表を表示する、という処理を実装していきます。
なお、これらの課題 (課題 JS-3 まで) の解答には、外部ライブラリを使用しないようにしてください。

課題 JS-4 以降は応用課題ですので、必ずしも行わなくても構いません。

### 課題 JS-0

各自、自分が主に使用しているブラウザにおける開発者用のツールを使ってみてください。
Firefox や Chorome がおすすめですが、Opera や Internet Explorer (10 以降)、Safari といったブラウザでも大丈夫です。
多くの開発者用のツールでは、次のような機能があります。

* ページの HTML 文書構造を動的に表示する
* JavaScript 用のコンソールで JavaScript 実行時のエラーを確認する
* デバッガで JavaScript の処理を対話形式で進める
  * その際、スコープ内の変数の値を確認できる
* イベント発火に対してブレイクポイントを設定する

使いこなせるようになる必要はありませんが、少なくとも JavaScript のエラーが発生しているのかどうかを確認できるようになっていれば、課題を進めやすいと思います。

例として Firefox や Chrome に付属している Web 開発用のツールを紹介します。

* Firefox では、メニューバーの 「ツール」 中の 「Web 開発」 から、web 開発向けの便利なツールを開くことができます。
  * Web コンソール: [Web コンソール - Tools | MDN](https://developer.mozilla.org/ja/docs/Tools/Web_Console)
  * デバッガ: [デバッガ - Tools | MDN](https://developer.mozilla.org/ja/docs/Tools/Debugger)
  * また、Firefox 拡張機能として [Firebug](https://addons.mozilla.org/ja/firefox/addon/firebug/) というものもあります
* Chrome には、デベロッパー ツールという開発者向けのツールが付属しています。
  * [Chrome DevTools — Google Developers](https://developers.google.com/chrome-developer-tools/)

### 課題 JS-1

Web サーバーへのアクセスログを表す LTSV 形式の文字列を引数として受け取り、その文字列をパースしてオブジェクトの配列にして返す関数 `parseLTSVLog` を js/main.js ファイルに記述してください。
アクセスログを表す LTSV 形式の文字列の例を次に示します。

注意) タブがスペースになっています。

```
path:/	reqtime_microsec:200000
path:/bookmark	reqtime_microsec:123456
```

1 行が 1 つのリクエストに対応しています。
1 行を 1 つの JavaScript のオブジェクトにし、アクセスログ中のラベルと値をプロパティとして持たせるようにしてください。
基本的に、アクセスログ中の値は文字列としてプロパティの値にすれば良いですが、「reqtime_microsec」 というラベルの値についてのみは数値にしてください。

すなわち、

```javascript
var logStr =
    "path:/\treqtime_microsec:200000\n" +
    "path:/help\treqtime_microsec:300000\n" +
    "path:/\treqtime_microsec:250000\n";
```

という文字列を引数として受け取り、

```javascript
[
  { path: "/",     reqtime_microsec: 200000 },
  { path: "/help", reqtime_microsec: 300000 },
  { path: "/",     reqtime_microsec: 250000 },
]
```

という構造の配列を返すように `parseLTSVLog` 関数を実装してください。

実際のアクセスログには様々なデータが含まれますが、課題では簡単のため 「path」 というラベルと 「reqtime_microsec」 というラベルのみを考慮すればよいものとします。
「path」 というラベルの値には、URL として有効な文字列が与えられます。
「reqtime_microsec」 というラベルの値としては、整数値とみなせる文字列が与えられます。

課題で指定されていない仕様は、自由に決めてよいものとします。
例えば、引数が与えられずに関数が呼び出された場合は、例外を送出しても良いですし、空の配列を返すようにしても構いません。

なお、この関数のテストは、js/test.html、js/test.js に記述されています。
ブラウザで js/test.html を表示すると、自動的にテストが走るようになっています。
最低限、最初から書かれているテストに通過するように `parseLTSVLog` 関数を実装してください。
余裕があれば、最初から書かれているテストにさらにテストを追加してみてください。
(例えば、予期せぬ値が引数として渡された場合に関するテストなど。)
テスト用のフレームワークとしては、[QUnit](http://qunitjs.com/) を使用しています。

### 課題 JS-2

次の 2 つの値を引数として受け取り、

* 第 1 引数: `div` 要素を表す DOM オブジェクト
* 第 2 引数: 課題 JS-1 で作成した関数 `parseLTSVLog` の返り値と同じ形式の配列

次のような表を (DOM 操作によって) 第 1 引数の `div` 要素の直下に生成する関数 `createLogTable` を js/main.js ファイルに記述してください。

```html
<table>
  <thead><tr><th>path</th><th>reqtime_microsec</th></tr></thead>
  <tbody>
    <tr><td>/</td><td>200000</td></tr>
    <tr><td>/help</td><td>300000</td></tr>
    <tr><td>/</td><td>250000</td></tr>
  </tbody>
</table>
```

`tbody` 要素内の `tr` 要素は、引数として受け取った配列の各要素に対応します。
表は 2 カラムにして、最初のカラムには `path` の値を、次のカラムには `reqtime_microsec` の値を表示するようにしてください。

また、引数として受け取った配列の最初の要素から順に、表の上から表示するようにしてください。

```javascript
// `createLogTable` 関数の使用例
var containerElem = document.getElementById("table-container"); // table-container という ID の div 要素が HTML ドキュメント中にあるとする
createLogTable(containerElem, [
    { path: "/",     reqtime_microsec: 200000 },
    { path: "/help", reqtime_microsec: 300000 },
    { path: "/",     reqtime_microsec: 250000 }
]);
```

課題 JS-1 と同じく、この関数のテストも js/test.html、js/test.js に記述されています。
最低限、最初から書かれているテストを通過するように実装し、余裕があれば新たにテストを追加してみてください。

参考になりそうな MDN (Mozilla Developer Network) のページを以下に挙げておきます。
(ここで紹介しているメソッド以外を使用しても構いません。)

* ノードに子ノードを追加: [Node.appendChild - Web API リファレンス | MDN](https://developer.mozilla.org/ja/docs/Web/API/Node.appendChild)
* Element ノードの生成: [document.createElement - Web API リファレンス | MDN](https://developer.mozilla.org/ja/docs/Web/API/document.createElement)
* ノードの子孫のテキストを設定したり読み取ったりする: [Node.textContent - Web API リファレンス | MDN](https://developer.mozilla.org/ja/docs/Web/API/Node.textContent)

### 課題 JS-3

課題 JS-1 と JS-2 で作成した関数を用いて、js/js-3.html のページが次のような動作をするように js/js-3.js に JavaScript の処理を記述しなさい。
js/main.js には課題 JS-1 と JS-2 で作成した関数が記述されているものとします。

* js/js-3.html の動作: ユーザーが `textarea` 要素に LTSV 形式のアクセスログを入力し、「表に出力する」 ボタンをクリックすると、`access-log-table-container` という `id` 属性をもつ要素の直下にアクセスログの表を表す `table` 要素が作られる。

js/js-3.html の中身は次のとおりです。

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>Hatena-Intern-Exercise2014 : JavaScript</title>
  <link rel="stylesheet" href="js-3.css" />
</head>
<body>
  <section>
    <h1>JavaScript 事前課題</h1>
    <h2>課題 JS-3 : イベントリスナを使用する</h2>
    <p>「表に出力する」 ボタンをクリックすると、テキストエリアに入力したログデータが表として表示されるようにしてください。 その際、課題 JS-1 と JS-2 で作成した関数を使用してください。</p>
    <p>また、表 (table 要素) は table-container を id として持つ div 要素の子要素として作成するようにしてください。</p>
    <div>
      <textarea id="log-input" cols="64" rows="10">path:/&#x0009;reqtime_microsec:123456
path:/uname&#x0009;reqtime_microsec:500000
path:/help&#x0009;reqtime_microsec:234222
path:/&#x0009;reqtime_microsec:94843
</textarea>
      <div><input value="表に出力する" type="button" id="submit-button" /></div>
    </div>
    <div id="table-container"></div>
  </section>
  <script src="main.js"></script>
  <script src="js-3.js"></script>
</body>
</html>
```

参考になりそうな MDN のページを以下に挙げておきます。
(ここで紹介しているメソッド以外を使用しても構いません。)

* id を指定して文書中の Element ノードを取得: [document.getElementById - Web API リファレンス | MDN](https://developer.mozilla.org/ja/docs/Web/API/document.getElementById)
* イベントリスナを追加: [EventTarget.addEventListener - Web API リファレンス | MDN](https://developer.mozilla.org/ja/docs/Web/API/EventTarget.addEventListener)

### 課題 JS-4 (応用課題)

課題 JS-3 で作成したものに、ログの検索機能を実装してください。

例えば、以下の様な機能が考えられます。
- `path` を検索できる機能。例えば `bookmark` と入力して「検索」ボタンを押すと、`path` の `uri` に `bookmark` が含まれるログが表示される
- 指定した数値以上の `reqtime_microsec` を含むログを検索できる機能。例えば `170000` と入力すると、170000以上の `reqtime_microsec` が含まれるログが表示される
- `POST`のログしか表示しない機能
- 2つ以上の条件を組み合わせて検索できる機能
- インクリメンタルサーチする機能

どのような検索機能にするかどうかや、UIは全て自由とします。複数の機能を作ってもらっても構いません。またこの課題では jQuery や Underscore.js など、好きなライブラリを使って構いません。

この課題のための HTML ファイルや JS ファイルは、新たに作成してください (ファイル名は特に指定しませんが、js/js-4.html や js/js-4.js など、わかりやすい名前にしてください)。
また、この課題で使うログデータは `sample_data/log.ltsv` にあるデータを `<textarea>` に入れて利用するようにしてください。実行の度に貼り付けても良いし、予めDOMに埋め込んでおいても構いません。
