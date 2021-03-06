# maya_exprespy
Maya で Python によるエクスプレッション機能を提供するノードのプラグインです。
exprespy（エクスプレスパイ）と呼びます。

![SS](/exprespy.png)


##特徴
本プラグインでは速度を重視し、ノードは C++ で実装されています。
また、それによって、Python によるエクスプレッションコードは一度だけコンパイルされメモリ上に保持され、
以降はコンパイル済みのコードオブジェクトを実行する仕組みを実現しています。
つまり、実行時の効率が良いものとなっています。

また、標準のエクスプレッション機能と同様に、エディタ（アトリビュートエディタ）上では、
実際のノード・アトリビュート名でのコーディングが可能です。
それらは、標準機能と同様に、実際はノードのコネクションに置き換えられるようになっています。

ただし、単位変換の機能はなく、内部単位で直接扱われます。
少し初心者向けでない部分ですが、これも効率重視たる所以です。

さらに、Python API 2.0 の型に対応し、double3 や matrix アトリビュートを直接入出力することが可能です。


##類似技術
Python でエクスプレッションを書けるようにするプラグインは
[SOuP](http://www.soup-dev.com/)
に含まれる
[pyExpression](http://www.soup-dev.com/wiki/PyExpression.html)
ノードが有名です。
しかし、それは全て Python で実装されており、
Python のエクスプレッションコードは exec() 関数によって実行される仕組みで、
実行するたびにコードをコンパイルするため効率的ではありません。

また、標準で付属する MASH には Python ノードがあり、Python でモーショングラフィックスを制御することが出来ます。
ただ、それは particle データを制御する仕組みで、それに特化された性能は素晴らしいですが、汎用的ではありません。


##ディレクトリ構成
* [examples](/examples): Mayaシーンの例。
* [plug-ins](/plug-ins): ビルド済みプラグインバイナリ。
* [python](/python): サポート python モジュール。
* [scripts](/scripts): サポート mel スクリプト。
* [srcs](/srcs): C++ソースコード。
* [viewTemplates](/viewTemplates): Node Editor のためのテンプレート。


##インストール方法
* plug-ins フォルダにあるプラットフォームとバージョンごとのフォルダに収められているファイルを
  MAYA_PLUG_IN_PATH の通ったフォルダにコピーします。
  (My Documents)\maya\\(version)\plug-ins でも OK です。

* python フォルダにあるファイルを PYTHONPATH の通ったフォルダにコピーします。
  (My Documents)\maya\scripts でも OK です。

* scripts フォルダにあるファイルを MAYA_SCRIPT_PATH の通ったフォルダにコピーします。
  (My Documents)\maya\scripts でも OK です。

* viewTemplates フォルダにあるファイルを MAYA_CUSTOM_TEMPLATE_PATH の通ったフォルダにコピーします。
  設定されていなければ (My Documents)\maya\\(version)\prefs\viewTemplates で OK です。


##使用方法
エクスプレッションを記述するには、まず exprespy ノードを生成します。
以下のどちらかの方法が利用できます。

* プラグインをロードしてから、以下の mel コードを実行。

  ```
  createNode exprespy;
  ```

* 以下の python コードを実行（プラグインはロードされていなくてもOK）。

  ```
  import exprespy
  exprespy.create()
  ```

その後、生成された exprespy ノードのアトリビュートエディタにコードを入力します。


##サンプルシーン
* [constraints.ma](/examples/constraints.ma)

  様々なコンストレイン機能をエクスプレッションで実装した例。
  position、orient、アップオブジェクト無しの aim、アップオブジェクト有りの aim を実装しています。

* [ripple.ma](/examples/ripple.ma)

  [SOuP](http://www.soup-dev.com/) の pyExpression のサンプルを速度比較のために置き換えたてみたもの。
  dgtimer でノードの処理速度のみを比較すると、なんと約15倍高速化されています。
  フレームレート比較だと3.5倍程度の高速化となり、描画負荷などもあるため純粋なノードの性能比較ほどの差はありません。

  また、アトリビュート名でコーディングせず、``IN`` や ``OUT`` での参照方法の例でもあります。

  ![SS](/ripple.png)

##詳細仕様

* 内部単位での扱い

  単位付きアトリビュートは入出力とも内部単位で扱われます。
  time は秒、
  doubleLinear(distance) は centimeter 、
  doubleAngle は radian
  での扱いとなります。

* 入力アトリビュートの型は以下に対応しています。

  - 様々なスカラー数値型（double, float, time, doulbeLinear, doubleAngle, int, bool, enum 等）

    Python 上では float か int になります（Python では float と double の区別はありません）。
    入力数値型の判別が厳密には出来ないため bool は int になります。

  - string 型

    Python 上では unicode になります。

  - double3 型

    Python 上では API 2.0 の MVector になります。
    その後、MPoint や MEulerRotation 等に変換するのも自由自在です。

  - matrix 型

    Python 上では API 2.0 の MMatrix になります。

* 出力アトリビュートの型は以下に対応しています。

  - 様々なスカラー数値型

    Python の型に応じて、以下のようにアトリビュート型が決まります。

    - bool -> bool
    - int -> int
    - long int -> int
    - float -> double

  - string 型

    Python 上の str や unicode が string 型になります。

  - double3 型

    Python 上の API 2.0 の MVector, MPoint, MEulerRotation が double3 になります。
    そのため MPoint の w や MEulerRotation の order はそのままだと捨てられますので、
    必要なら別アトリビュートに出してください。

  - matrix 型

    Python 上の API 2.0 の MMatrix が matrix 型になります。

* モジュール

  Python エクスプレッションコードでは、以下のモジュールがインポート済みで使える状態となっています。

  ```
  import math
  import maya.api.OpenMaya as api
  import maya.cmds as cmds
  import maya.mel as mel
  ```

  他に必要なモジュールがあれば自由に import してください
  （一般的に import 文の負荷は高めなので、毎実行単位で行うのは少し注意）。

* スコープ

  Python エクスプレッションコードでは、Maya Python のグローバルスコープ上の名前に参照出来ます。
  ただし、新たに使用した変数などで、Maya のグローバルスコープは汚れません。

  ローカルスコープは、コードが編集されコンパイルし直されるまで保持されます。
  それによって、例えば、状態を記憶して最初の一回だけ実行する処理を記述したりすることも可能です。

* 入出力と依存関係

  入力として参照したアトリビュートは exprespy ノードの input[] アトリビュートに、
  出力として参照した（ = で代入した）アトリビュートは output[] アトリビュートに接続されます。
  それらの型は generic で、何を繋いでも unitConversion ノードは挟まりません（仕様で許可されている型しか繋がりません）。

  全ての出力は全ての入力に依存することになります。コード上の論理的な依存関係は関係ありません。
  よって、シーン中の全ての処理を一つの exprespy ノードに書くのは良くありません。
  こういった考え方や仕組みは標準の expression ノードと全く同じものです。

* コードはどのように保存されるのか

  コードは exprespy ノードの code という string アトリビュートに保存されます。
  コード内のアトリビュート参照は ```IN[index]``` や ```OUT[index]``` のようなプレースホルダに置き換わります。
  実際のアトリビュートは、それらに対応した input[] と output[] に接続されます。

  アトリビュートエディタ上に見えているコードは、プレースホルダを置き換えた仮のコードです。
  この処理は exprespy の Python モジュールが行っています。
  
  ``IN`` と ``OUT`` は python エクスプレッションコードでは dict となっています。
  list ではない理由は、実際のマルチアトリビュートと同様に欠番のある疎な配列に対応するためです。
  この仕様を理解していれば、ノードエディタやスクリプト等でコネクションを直接編集し、
  エディタ上のコードに直接 ``IN`` や ``OUT`` とインデックスを直接コーディングしても構いません。
  また、標準エクスプレッション機能では不可能な exprespy ノード同士を連結することも出来るでしょう。

  エディタがコネクションを識別すると ``IN`` や ``OUT`` を直接記述したコードもアトリビュート名に置き換えられて見えるようになり、
  以降は自動的なコネクション管理がなされます。
  ただし、インデックスをハードコードせずに変数にして参照するようなコードの場合は、エディタには識別されません。
  識別されないコネクションは維持され続けますので、何も問題ありません。


##制限事項
* コード内に書いたアトリビュート参照はパースされて実際のコネクションとなるわけですが、
  コードのコメントを識別していないため、
  コメント中にも実際のプラグにマッチする名前があるとコネクションに置き換えられてしまいます。
  実行結果がおかしくなるわけではありませんが、余計な参照が増えるため、無駄にはなります。

* GUI（アトリビュートエディタ）は、あまり気合入れて書いていないため、時におかしな挙動をするかも知れません。

* 仕様に書いたアトリビュートの入出力にしか対応していませんが、
  将来的には mesh、nurbsCurve、nurbsSurface 等の入出力も可能にしたいと考えてはいます。
  ただ、実装難度が高いので、まずは非対応です。
  （データオブジェクトを C++ API から Python API への効率的に変換する手法が思いつかない…）


##改訂履歴
* 2016.10.6:
  - アトリビュートエディタを改善。識別していないコネクションが維持されるようにした。
  - サンプルシーンに [ripple.ma](/examples/ripple.ma) 追加。
  - [viewTemplates](/viewTemplates) 追加。Node Editor で扱いやすくなる。
  - プラグインを更新。compiled アトリビュートを hidden にした。
  
* 2016.10.2: 初版

