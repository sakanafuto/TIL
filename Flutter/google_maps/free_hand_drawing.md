# 【Flutter】Riverpod / Google Mapsでマップをなぞる

# はじめに

flutter上でGoogleマップを表示させ、なぞった範囲内を塗りつぶす機能を実装してみました。

![](https://storage.googleapis.com/zenn-user-upload/875851d3b361-20220820.gif)

実装方法は[こちら](https://medium.com/@shadyboshra2012/flutter-free-hand-polygon-drawing-on-google-maps-796e8bd47e85)の記事を参考にさせていただきました。`Dart`のバージョンが古く`Null Safety`に対応しておらず、また状態管理にライブラリを用いていなかったため、そのあたりを本記事でアレンジしております。

## バージョン
```bash 
❯ fvm flutter --version

Flutter 3.0.1 • channel stable • https://github.com/flutter/flutter.git
Framework • revision fb57da5f94 (3 months ago) • 2022-05-19 15:50:29 -0700
Engine • revision caaafc5604
Tools • Dart 2.17.1 • DevTools 2.12.2
```

## ライブラリ

```dart
hooks_riverpod: ^1.0.4
flutter_hooks: ^0.18.4
google_maps_flutter: ^2.1.10 // mapを表示する
flutter_config: ^2.0.0 // 環境変数を設定する
geolocator: ^9.0.1 // 現在地情報を取得する
```

## 用語

実装当初、知らなくて戸惑った単語たち…

| 用語 | 説明 | 本記事における扱い |
| ---- | ---- | ---- |
| LatLng | 緯度経度のこと | PolylineとPolygonを構成する要素で、この座標の集まりで描画を表現する |
| Polyline | マップ上に線を描くためのもの | なぞっている間、線が動的に指についてくる動作を描画するのに用いる |
| Polygon | マップ上に座標で囲まれた領域を多角形で表すもの | なぞり終えて、指が離れた際に多角形を描画するのに用いる |

# 事前準備

[google_maps_flutter](https://pub.dev/packages/google_maps_flutter)などを参考に、Google Maps APIと表示させたいプラットフォームとを連携します。これをしないとマップが表示されません。

また本記事では解説しませんが、APIキーを直接書かないように`flutter_config`を用いました。
環境変数をDartだけでなくiOSやAndroidのコードに公開できる便利なライブラリです。

今回はiOSとAndroidで実装したかったので、このあたりのファイルを編集しました。
- `/ios/Runner/Info.plist`
- `/ios/Runner/AppDelegate.swift`
- `/android/app/src/main/AndroidManifest.xml`
- `/android/app/build.gradle`

参考： [AndroidManifest.xml、AppDelegate.swiftからenvにアクセス](https://zenn.dev/taitai9847/articles/0be7298408b7c8)

# 実装

コード全体は[まとめ](#まとめ)に記載しています。

## build

ビルド部分の要点を抜き出したコードは次のとおりです。

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: ...
      GestureDetector(
        onPanUpdate: (drawPolygonEnabled) ? _onPanUpdate : null,
        onPanEnd: (drawPolygonEnabled) ? _onPanEnd : null,
        child: GoogleMap(
          ...
          polylines: _polylineSet,
          polygons: _polygonSet,
          ...
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _toggleDrawing,
        ...
      ),
    ...
```

`GestureDetecter`（childに対してドラッグ操作やピンチ操作などを付与する）のchildに`GoogleMap`というWidgetを指定しています。

`GoogleMap`ウィジェットのみだと、普段私たちが使っているGoogle Mapsアプリのような拡大縮小やマップ移動の操作しかできません。これをなぞれるようにするため、`FloatingActionButton`に『マップ操作』と『なぞる操作』を切り替える機能を実装しています。

`GestureDetector`のプロパティとして`onPanUpdate`と`onPanEnd`を定義しています。どちらも『なぞる』状態ではたらく関数を定義していて、中身は後述します。

`onPanUpdate` => ドラッグ操作で位置が変化したときにコール
`onPanEnd` => ドラッグ操作が終了したときにコール

`GoogleMap`のプロパティには`polylines`と`polygons`があり、`Polyline / Polygon`のインスタンスを`value`とするハッシュセットを指定することでマップ上に線分や図形を表示できます。

## _onPanUpdate

つづいて`_onPanUpdate`関数の中身です。

コメントにそれぞれの解説が書いてあるとおりですが、こちらの関数の役割は”触った部分の座標をもとにPolylineインスタンスを生成し、_polylineSetに追加する”です。

```dart
_onPanUpdate(DragUpdateDetails details) async {
  // 常に新しいポリゴンを描画する
  if (ref.watch(clearDrawingProvider.state).state) {
    ref.read(clearDrawingProvider.state).state = false;
    _clearPolygons();
  }

  double? x, y;

  // Androidの場合、描画距離が明らかに短い
  // 恐らくGoogleMapsのバグ
  if (Platform.isAndroid) {
    x = details.localPosition.dx * 3;
    y = details.localPosition.dy * 3;
  } else if (Platform.isIOS) {
    x = details.localPosition.dx;
    y = details.localPosition.dy;
  }

  if (x != null && y != null) {
    int xCoordinate = x.round();
    int yCoordinate = y.round();

    // 2本指描画を防止するため
    if (_lastXCoordinate != null && _lastYCoordinate != null) {
      var distance = math.sqrt(math.pow(xCoordinate - _lastXCoordinate!, 2) +
          math.pow(yCoordinate - _lastYCoordinate!, 2));
      // 点と点の距離が大きいかどうかをチェック
      if (distance > 80.0) return;
    }

    // 座標をキャッシュする
    _lastXCoordinate = xCoordinate;
    _lastYCoordinate = yCoordinate;

    // GoogleMapにおける座標に変換
    ScreenCoordinate screenCoordinate =
        ScreenCoordinate(x: xCoordinate, y: yCoordinate);

    final GoogleMapController controller = await _controller.future;
    LatLng latLng = await controller.getLatLng(screenCoordinate);

    try {
      // 新しいポイントをリストに追加する
      _latLngList.add(latLng);

      // ポリラインのセットを初期化
      _polylineSet.removeWhere(
          (polyline) => polyline.polylineId.value == 'user_polyline');
      // 新しいポリラインを追加
      _polylineSet.add(
        Polyline(
          polylineId: const PolylineId('user_polyline'),
          points: _latLngList,
          width: 2,
          color: Colors.blue,
        ),
      );
    } catch (e) {
      if (kDebugMode) {
        print(" error painting $e");
      }
    }
    // なぞっている間ポリラインが描画され続ける
    ref.read(polygonSetProvider.state).state = {..._polygonSet};
  }
}
```
## _onPanEnd

つづいて`_onPanEnd`関数の中身です。

こちらの関数の役割は”なぞり終えた際にPolygonインスタンスを生成し、_polygonSetに追加する。”です。

```dart
onPanEnd(DragEndDetails details) async {
  // 最後にキャッシュされた座標をリセット
  _lastXCoordinate = null;
  _lastYCoordinate = null;

  // ポリゴンのセットを初期化
  _polygonSet
      .removeWhere((polygon) => polygon.polygonId.value == 'user_polygon');
  // 新しいポリゴンの追加
  _polygonSet.add(
    Polygon(
      polygonId: const PolygonId('user_polygon'),
      points: _latLngList,
      strokeWidth: 2,
      strokeColor: Colors.blue,
      fillColor: Colors.blue.withOpacity(0.4),
    ),
  );

  // 描画状態を終了
  ref.read(drawPolygonEnabledProvider.state).update((state) => !state);
}
```

## _toggleDrawing / _clearPolygons

`_toggleDrawing`と`_clearPolygons`です。

```dart
/// 「なぞる」の切り替えを行う関数
_toggleDrawing() {
  _clearPolygons();
  ref.read(drawPolygonEnabledProvider.state).update((state) => !state);
}

/// PolylineとPolygonとLatLngをクリアする関数
_clearPolygons() {
  _latLngList.clear();
  _polylineSet.clear();
  _polygonSet.clear();
}
```

## _determinePosition

デバイスの位置情報周りの確認を行い、現在地を返す関数です。
『なぞる』が目的であればこちらはオプションになります。

```dart
/// デバイスの現在位置を決定する
/// 位置情報サービスが有効でない場合 or 許可されていない場合エラー
Future<Position> _determinePosition() async {
  bool serviceEnabled;
  LocationPermission permission;

  serviceEnabled = await Geolocator.isLocationServiceEnabled();
  if (!serviceEnabled) {
    return Future.error('位置情報サービスが無効です。');
  }

  permission = await Geolocator.checkPermission();
  if (permission == LocationPermission.denied) {
    permission = await Geolocator.requestPermission();
    if (permission == LocationPermission.denied) {
      return Future.error('位置情報を取得する権限がありません。');
    }
  }

  if (permission == LocationPermission.deniedForever) {
    return Future.error('位置情報サービスの権限が永久に拒否されています。権限を要求することができません。');
  }

  Position position = await Geolocator.getCurrentPosition(
      desiredAccuracy: LocationAccuracy.high);
  _loading = false;
  ref.read(getUserLocationProvider.state).state =
      LatLng(position.latitude, position.longitude);
  return position;
}
```

これにより位置情報サービスを許可するか確認した上で、現在地を取得できるようになります。
`GoogleMap`ウィジェットにおいて`myLocationButtonEnabled: true`（デフォルトでtrue）とすることでマップにボタンが表示され、押下することで現在地まで移動してくれます。

![](https://storage.googleapis.com/zenn-user-upload/e06163b302db-20220822.gif)

# まとめ
コードの全体像です。

```dart
import 'package:flutter/material.dart';
import 'package:flutter/foundation.dart';
import 'dart:io';
import 'dart:async';
import 'dart:collection';
import 'dart:math' as math;

import 'package:hooks_riverpod/hooks_riverpod.dart';

import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'package:geolocator/geolocator.dart';


final drawPolygonEnabledProvider = StateProvider<bool>((ref) => false);
final clearDrawingProvider = StateProvider<bool>((ref) => false);
final polygonSetProvider = StateProvider<Set<Polygon>>((ref) => {});
final getUserLocationProvider =
    StateProvider<LatLng>((ref) => const LatLng(0, 0));

class MyApp extends StatefulHookConsumerWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  MyAppState createState() => MyAppState();
}

class MyAppState extends ConsumerState<MyApp> {
  static final Completer<GoogleMapController> _controller = Completer();

  bool _loading = true;

  @override
  void initState() {
    super.initState();
    _loading = true;
    _determinePosition();
  }

  int? _lastXCoordinate;
  int? _lastYCoordinate;

  get _polygonSet => ref.watch(polygonSetProvider);

  final Set<Polyline> _polylineSet = HashSet<Polyline>();
  final List<LatLng> _latLngList = [];

  @override
  Widget build(BuildContext context) {
    final drawPolygonEnabled = ref.watch(drawPolygonEnabledProvider);
    final currentPosition = ref.watch(getUserLocationProvider);
    return Scaffold(
      body: _loading
          ? const CircularProgressIndicator()
          : GestureDetector(
              onPanUpdate: (drawPolygonEnabled) ? _onPanUpdate : null,
              onPanEnd: (drawPolygonEnabled) ? _onPanEnd : null,
              child: GoogleMap(
                mapType: MapType.normal,
                initialCameraPosition: CameraPosition(
                  target: currentPosition,
                  zoom: 14.4746,
                ),
                polygons: _polygonSet,
                polylines: _polylineSet,
                myLocationEnabled: true,
                myLocationButtonEnabled: false,
                onMapCreated: (GoogleMapController controller) {
                  _controller.complete(controller);
                },
              ),
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: _toggleDrawing,
        backgroundColor: Theme.of(context).colorScheme.background,
        child: Icon(drawPolygonEnabled ? Icons.cancel : Icons.edit),
      ),
    );
  }

  _onPanUpdate(DragUpdateDetails details) async {
    if (ref.watch(clearDrawingProvider.state).state) {
      ref.read(clearDrawingProvider.state).state = false;
      _clearPolygons();
    }

    double? x, y;

    if (Platform.isAndroid) {
      x = details.localPosition.dx * 3;
      y = details.localPosition.dy * 3;
    } else if (Platform.isIOS) {
      x = details.localPosition.dx;
      y = details.localPosition.dy;
    }

    if (x != null && y != null) {
      int xCoordinate = x.round();
      int yCoordinate = y.round();


      if (_lastXCoordinate != null && _lastYCoordinate != null) {
        var distance = math.sqrt(math.pow(xCoordinate - _lastXCoordinate!, 2) +
            math.pow(yCoordinate - _lastYCoordinate!, 2));
        if (distance > 80.0) return;
      }

      _lastXCoordinate = xCoordinate;
      _lastYCoordinate = yCoordinate;

      ScreenCoordinate screenCoordinate =
          ScreenCoordinate(x: xCoordinate, y: yCoordinate);

      final GoogleMapController controller = await _controller.future;
      LatLng latLng = await controller.getLatLng(screenCoordinate);

      try {
        _latLngList.add(latLng);

        _polylineSet.removeWhere(
            (polyline) => polyline.polylineId.value == 'user_polyline');
        _polylineSet.add(
          Polyline(
            polylineId: const PolylineId('user_polyline'),
            points: _latLngList,
            width: 2,
            color: Colors.blue,
          ),
        );
      } catch (e) {
        if (kDebugMode) {
          print(" error painting $e");
        }
      }
      ref.read(polygonSetProvider.state).state = {..._polygonSet};
    }
  }

  _onPanEnd(DragEndDetails details) async {
    _lastXCoordinate = null;
    _lastYCoordinate = null;

    _polygonSet
        .removeWhere((polygon) => polygon.polygonId.value == 'user_polygon');
    _polygonSet.add(
      Polygon(
        polygonId: const PolygonId('user_polygon'),
        points: _latLngList,
        strokeWidth: 2,
        strokeColor: Colors.blue,
        fillColor: Colors.blue.withOpacity(0.4),
      ),
    );

    ref.read(drawPolygonEnabledProvider.state).update((state) => !state);
  }

  _toggleDrawing() {
    _clearPolygons();
    ref.read(drawPolygonEnabledProvider.state).update((state) => !state);
  }

  _clearPolygons() {
    _latLngList.clear();
    _polylineSet.clear();
    _polygonSet.clear();
  }

  Future<Position> _determinePosition() async {
    bool serviceEnabled;
    LocationPermission permission;

    serviceEnabled = await Geolocator.isLocationServiceEnabled();
    if (!serviceEnabled) {
      return Future.error('位置情報サービスが無効です。');
    }

    permission = await Geolocator.checkPermission();
    if (permission == LocationPermission.denied) {
      permission = await Geolocator.requestPermission();
      if (permission == LocationPermission.denied) {
        return Future.error('位置情報を取得する権限がありません。');
      }
    }

    if (permission == LocationPermission.deniedForever) {
      return Future.error('位置情報サービスの権限が永久に拒否されています。権限を要求することができません。');
    }

    Position position = await Geolocator.getCurrentPosition(
        desiredAccuracy: LocationAccuracy.high);
    _loading = false;
    ref.read(getUserLocationProvider.state).state =
        LatLng(position.latitude, position.longitude);
    return position;
  }
}
```

現在`Riverpod`を勉強中でして、急ごしらえで`StateProvider`で実装したものになります。
もっといい方法がありましたらコメントいただけますと幸いです🙇‍♂️

# 参考
**なぞる**
- [Flutter —Free Hand Polygon Drawing on Google Maps](https://medium.com/@shadyboshra2012/flutter-free-hand-polygon-drawing-on-google-maps-796e8bd47e85)（[GitHub](https://github.com/ShadyBoshra2012/free_hand_polygon_drawing_map/blob/master/lib/main.dart)）
- [Google Maps Flutter: Marker, circle and polygon.](https://medium.com/@zeh.henrique92/google-maps-flutter-marker-circle-and-polygon-c71f4ea64498)（[GitHub](https://gist.github.com/josehenriqueroveda/a7be466fa6ae06e526ee29cf02db7676)）
- [【Flutter】GestureDetectorを使って、スワイプしたときの位置を取得する](https://note.com/hatchoutschool/n/n4f70764cc83b)

**flutter_config**
- [AndroidManifest.xml、AppDelegate.swiftからenvにアクセス](https://zenn.dev/taitai9847/articles/0be7298408b7c8)

**geolocator**
- [【Flutter】スマホの位置情報を取得するやり方](https://zenn.dev/namioto/articles/3abb0ccf8d8fb6)
- [FlutterでGoogleMapが開くときに現在地で表示する](https://zenn.dev/t_sakurai/articles/97b4ee8952e9c7)