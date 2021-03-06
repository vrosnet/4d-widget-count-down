タイマーウィジェット
---

簡単なカウントダウンタイマーです。(v13+)

![](https://github.com/miyako/4d-widget-count-down/blob/master/images/1.png)

使い方
---
* データベースのComponentsフォルダーに```count-down.4dbase```またはエイリアスをインストールします。
* フォームにサブフォームオブジェクトを追加し，《出力サブフォーム》プロパティを外します。
* 縦横スクロールバーは消してください。
* リサイズオプション・境界線スタイル・フォーカスはどれでも構いません。
* サブフォームの《詳細フォーム》リストの中から```count-down (count-down)```を選択します。
* サブフォームの変数タイプを《時間》に設定します。
* 変数名は省略することができます《フォームローカル変数》。
* 変数名を省略しない場合，C_TIMEで宣言することが必要です。
* フォームイベントは```On Data Change```以外，すべて外してください。
* オブジェクト名は必要に応じて変更してください。

![](https://github.com/miyako/4d-widget-count-down/blob/master/images/2.png)

* タイマーを開始する
```
CD_START ("Timer1";?00:00:30?)
```
$1: サブフォーム（ウィジェット）のオブジェクト名

$2: スタートする時間（1時間を超える場合は```59:59```からスタートします。負の値は無視されます。）

* タイマーを停止する
```
CD_PAUSE ("Timer1")
```

* タイマーを再開する
```
CD_RESUME ("Timer1")
```

* コールバック
時間が経過する（1秒単位）ごとに```On Data Change```イベントが発生します。

たとえば，残り時間に合わせて色を変えたり，```00:00```に達したらメッセージを表示するようなことができます。

```
$thisWidget:=OBJECT Get name(Object current)

$event:=Form event

$RED:=0x00FF0000
$YELLOW:=0x00FFD400
$GREEN:=0xC23B

Case of
: ($event=On Data Change)

Case of 
: (Self->=?00:00:00?)

CANCEL

: ((Self->)<?00:00:10?)

If ($RED#CD_Get_color ($thisWidget))
CD_SET_COLOR ($thisWidget;$RED)
End if 

End case 

End case 
```
![](https://github.com/miyako/4d-widget-count-down/blob/master/images/3.png)

論考
---

***イベント連鎖***

通常，ウィジェットはコンテナ側の```On Data Change```イベントを外し，**キャプチャリング**，つまり親から子に対するイベント連鎖は，バインド変数とウィジェット側の```On Bound Variable Change```イベントで処理します。また，**バブリング**，つまりこから親に対するイベント連打は，```CALL SUBFORM CONTAINER```コマンドで処理します。

*通常のイベント連鎖*:

* キャプチャリング: ```On Bound Variable Change``` (コンテナの```On Data Change```は外すこと)
* バブリング: ```CALL SUBFORM CONTAINER```


しかし，```ON TIMER```イベントは例外的であり，この方式ではイベントがうまく連鎖しません。そのため，イベント連鎖は別の方法でコーディングする必要があります。

*例外的なイベント連鎖*:

* キャプチャリング: ```On Data Change``` (コンテナの```On Data Change```は外さないこと)
* バブリング: コンテナ変数をタッチする（e.g. ```$Container->:=$Container->```）

なお，ウィジェットのタイマーイベントは，親のタイマーイベントからは独立しています。今回の場合，ウィジェット内のタイマーイベントは1ティック毎にコールされていますが，実際に時間変数が更新されたときのみ，コンテナにバブリングイベントを送信しています。1秒単位でタイマーをコールしていないのは，更新間隔の誤差を少なくするため，またイベントが飛ばされた場合にも，正確な経過時間を反映するためです。4Dのタイマーイベントは，他のイベントと競合した場合には発生しません。ウィジェットは，タイマー開始からの経過時間に基づき，表示を更新しています。

***SVGのアニメーション***

デジタル時計は，SVG画像です。時計を構成する各オブジェクト（XML要素）には，IDが振られています。画像を更新するときには，```SVG SET ATTRIBUTE```を使用しています。このコマンドが使用できるのは，すでにレンダリングされた画像だけです。つまり，ファイルから読み込まれたばかりで，まだレンダリングされていないSVG画像には使用できません。また，SVG画像には，バックエンドのDOM（内部DOMツリー）がなければなりません。

下記の方法で作成されたSVGにはバックエンドのDOMが存在します。

*  ```SVG EXPORT TO PICTURE``` (```Get XML Data Source```以外のオプション)
* ```BLOB TO PICTURE```

下記の方法で作成されたSVGにはバックエンドのDOMが存在しません。

* READ PICTURE FILE
* スタティックピクチャ
* ライブラリピクチャ

```DOM SET XML ATTRIBUTE BY NAME (INDEX) ```とは違い，```SVG SET ATTRIBUTE/SVG GET ATTRIBUTE```は，レンダリングされたDOMを直接，書き換えることができます。ですから， <def>要素で定義されたCSSクラスや```<g>```要素，```<use>```要素など，さまざまな仕方で継承された最終的な属性値を読み書きすることができます。また，DOMを再解析する必要がありません。一方，```<def>```やCSSクラスなど，宣言型の定義を書き換えることはできません。そのような定義を書き換えるためには，DOMコマンドでXMLそのものを書き換えてから，再度，```SVG EXPORT TO PICTURE```で画像を作成する必要があります。

Acknowledgments
---
Digital Clock (www.Mathew.Callaghan.ws) / CC BY 4.0
