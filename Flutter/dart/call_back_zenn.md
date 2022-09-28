サンプルを用いて、タイトルの通り別のWidgetでコールバック関数を呼び出しています。
おまけでコールバック関数について少し触れています。

:::message
- コードを読みやすくするために一部省略しています。
- 勉強中のコードになります。もっと良い書き方があればコメントください🙇‍♂️
:::

---

# 寿司屋

``SushiShop``という``StatefulWidget``内で個別の``SushiWidget``をいくつも表示させたいとします。
また寿司は作ったり食べたりできるので、``makeSushi``と``eatSushi``という関数も用意したいです。

``makeSushi``は新しく寿司を作り一覧に追加し、``eatSushi``は選んだ（タップした）寿司を消す挙動にします。

``SushiWidget``を一覧化するのはかんたんで、``SushiShop``内のビルドの通り``Widget``の``children``としてリストを渡してあげればよいです。リスト内のオブジェクトの型は``Sushiクラス``として、インスタンスを格納します。

``Sushiクラス``のプロパティには、ネタとして``neta``（サンプルコードは"鮪"確定ですが…）と、あとはどの寿司を食べたか識別するためのidを持っています。

``SushiShop``内で寿司をつくる``makeSushi``関数を作成し、``id``と``neta``を初期化します。あとは寿司を食べる``eatSushi``関数を…とここで問題がでてきます。

``void eatSushi(int netaId) {sushiList.removeAt(netaId);}``で一見よさそうにみえますが、``eatSushi``を発動させるのは寿司がタップされたとき、すなわち``SushiShop``クラスとは違う``Sushiクラス``のウィジェット（``TextButton``）がタップされたときになります。

そのため``SushiWidget``内で``onPressed: _eatSushi()``みたいなことはできないんです。

これを解決するために、``Sushiクラス``では``Function(int) sushiCallback（引数にint型を必要とする関数）``をプロバティとして持たせています。

寿司自身がタップされたときには``onPressed: () => sushiCallback(id)``として``id（＝識別するためのもの）``を引数に渡してあげることで、``SushiShop``でどの寿司がタップされたのかを判断することができるのです。

```dart:sample.dart
class SushiShop extends StatefulWidget {
	...
}

class SushiShopState extends State<SushiShop> {

	List<Sushi> sushiList = [];
	int _sushiCounter = 0;

	void _makeSushi() {
		Sushi sushi = Sushi(
			id: _sushiCounter,
			neta: "鮪",
			sushiCallback: (sushiId) => _eatSushi(sushiId),
		);
		_sushiCounter++;
		setState(() {
			sushiList.add(sushi);
		});
	}

	void _eatSushi(int netaId) {
		_sushiCounter--;
		setState(() {
			// 指定した添字のオブジェクトを削除
			sushiList.removeAt(netaId);
		});
	}

	@override
	Widget build(BuildContext context) {
		return Column(
			children: [
				IconButton(
					...,
					onPressed: () => _makeSushi(),
				),
				Column(
					...
					children: sushiList,
				),
			],
		),
	}
}
```

```dart:sample.dart
class Sushi extends StatelessWidget {
	Sushi({
		required this.id,
		required this.neta,
		required this.sushiCallback,
		Key? key
	}) : super(key: key);

	int id;
	String neta;
	Function(int) sushiCallback;

	@override
	Widget build(BuildContext context) {
		return Center(
			child: TextButton(
				onPressed: () => sushiCallback(id),
				child: Text(
					"$neta",
				),
			),
		);
	}
}

```

寿司を並べるくらいなら``Text``とかを羅列すれば良かったのですが、任意にタップした寿司を消すという動的な仕様を実装しようと思ったらここまでややこしくなりました。

※``sushiCounter``のインクリメントなどはもうちょっといじらないとうまく動作しません。

## おまけ

寿司クラスも、JSでかんたんにまとめると少しわかりやすいように思えます。

```javascript:sample.js
// 寿司関数を定義（高階関数）
function sushi(id, neta, func) {
	func(id)
}

// コールバック関数
const eatSushi = function(id) {
	... // [id]の寿司をたべる処理
}

sushi(1, "サーモン", eatSushi);
```

コールバック関数は直接引数にできるので…

```javascript:sample.js
function sushi(id, neta, func) {
	func(id)
}

sushi(1, "サーモン", function(id){
	...
});
```

これにアロー関数を用いると…

```javascript:sample.js
function sushi(id, neta, func) {
	func(id)
}

sushi(1, "サーモン", (id) => {...});
```
サンプルコードのこれも

```dart:sample.dart
// 高階関数
Sushi({
	required this.id,
	required this.neta,
	required this.sushiCallback,
	Key? key
}) : super(key: key);

// コールバック関数
void _eatSushi(int netaId) {
	_sushiCounter--;
	setState(() {
		sushiList.removeAt(netaId);
	});
}

Sushi sushi = Sushi(
	id: _sushiCounter,
	neta: "鮪",
	sushiCallback: (sushiId) => _eatSushi(sushiId),
);
```

つまりはこういうこと（一部省略）

```dart:sample.dart
Sushi(id, neta, sushiCallback)

void _eatSushi(int netaId) {
	sushiList.removeAt(netaId);
}

Sushi sushi = Sushi(_sushiCounter, "鮪", (sushiId) => _eatSushi(sushiId));
```

``Widget``で見かけるコロンや名前付き引数の波括弧がいい感じにややこしくしてますね…。

書き方や考え方にいろいろと甘い部分はあると思います。
``Flutter``は楽しいのでもっと勉強してパワーアップしたいです🦋
