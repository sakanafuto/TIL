# Flutter を MVVM で実装する

![](https://miro.medium.com/max/1400/1%2AUj5dnm3RTQ89uidDpcObHw.jpeg)

Repository pattern はデータソースへのアクセスを抽象化するためのデザインパターン。
Repository 採用の理由は 3 つ。

- データアクセスを再利用するため
- データの取得時に、「どこからデータを取ってくるか？」という記述を Presentation レイヤーから遮断するため
- データアクセスにおいてライブラリ依存を解決するため。

「Presentation レイヤーはどこからデータを取ってくるか知る必要はなくて、データさえ取れればよい」という考え方。ライブラリに破壊的な変更があった倍位にも、Ripository のレイヤーの修正だけで済み、保守性も高まる。

UseCase は？

![](https://img.logmi.jp/article_images/XyBh9eUzXdT7FMNCu5CHtj.png)

![](https://img.logmi.jp/article_images/LPMe8e36asQqurG4U2L8J9.png)

クリーンアーキテクチャの中では「アプリケーション固有のビジネスロジックを担当するレイヤー」とある。

![](https://img.logmi.jp/article_images/16YGGqgLo7WuazEikPo5qN.png)

Repository をデータアクセスのためのレイヤーとしてしまうと、中規模以上の開発の場合、マッピングやバリデーションを ViewModel に記述せざるを得なくなり、FatViewModel となってしまう。

![](https://img.logmi.jp/article_images/W3JVWXUHK6xW1b1mnGfvoD.png)

## Flutter Architecture Blueprints

[Flutter Architecture Blueprints](https://github.com/wasabeef/flutter-architecture-blueprints)

_ChangeNotifier_

- 値が変更されたことを通知することができる Observable。
- ViewModel が Model を複数持つような場合、StateNotifier だと 1 つのクラスしか持てない。

### ViewModel

ChangeNotifier を継承している。

HomeViewModel は ChangeNotifier を継承しているため、ChangeNotifierProvider を使用することで HomeViewModel を生成し、後述する View 側で読み取ることができる。

扱うデータは news で、データが更新されたことを通知したい場合に notifyListener メソッドを呼ぶことで View 側に通知することができる Observable。

NewsRepository の getNews というメソッドを使って、news データを取ってきている。

```dart
import 'package:app/data/app_error.dart';
import 'package:app/data/model/news.dart';
import 'package:app/data/provider/news_repository_provider.dart';
import 'package:app/data/repository/news_repository.dart';
import 'package:app/ui/change_notifier_with_error_handle.dart';
import 'package:flutter/material.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

final homeViewModelNotifierProvider = ChangeNotifierProvider(
    (ref) => HomeViewModel(ref, repository: ref.read(newsRepositoryProvider)));

class HomeViewModel extends ChangeNotifier {
  HomeViewModel({@required NewsRepository repository})
      : _repository = repository;

  final NewsRepository _repository;

  News _news;
  News get news => _news;

  Future<void> fetchNews() async {
    return _repository
        .getNews()
        .then((value) => _news = value;)
        .catchError((dynamic error) { /* Error Handling */ )
        .whenComplete(() => notifyListeners());
  }
}
```

### View

useProvider は、ConsumerWidget の場合 watch することで再現可能。

HookBuilder も Flutter Hooks のクラスである。このビルダーの中で useProvider を生成して、その値が変更された場合にはこの中だけでリビルドが起こる。Flutter Hooks を使わずに Riverpod だけでやる場合には、Comsumer で同様のことが可能。

useFuture は FutureBuilder と同じ動作をする。

:::details useFuture の例

Future や Stream を扱うもので、FutureBuilder や StreamBuilder が簡単に書けるようになる。以下は package_info を用いてアプリ情報を取得する場合の記述の仕方。

_Hooks なし_

```dart
class NotUseHooksSample extends HookWidget {
  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: PackageInfo.fromPlatform(),
      builder: (context, snapshot) {
        if (snapshot.hasData) {
          return Text('appName = ${snapshot.data.appName}');
        } else {
          return Container();
        }});}}
```

_Hooks あり_

ネストが浅くなりシンプルになる。

```dart
class UseFutureSample extends HookWidget {
  @override
  Widget build(BuildContext context) {
    final packageInfo = useMemoized(PackageInfo.fromPlatform);
    final snapshot = useFuture(packageInfo);
    if (snapshot.hasData) {
      return Text('appName = ${snapshot.data.appName}');
    } else {
      return Container();
    }}
}
```

:::

useMemoized は AsyncMemoizer と似たよう動作で、配列で渡している Keys の値が同じならば fetchNews メソッド部分のキャッシュを返してリビルドしない。ここでは news データを toString したものをキーに含めている。

> 参考: [useXx](https://qiita.com/mkosuke/items/f88419d0f4d41ed6d858)

```dart
import 'package:app/ui/app_theme.dart';
import 'package:app/ui/component/article_item.dart';
import 'package:app/ui/component/loading.dart';
import 'package:app/ui/component/toast.dart';
import 'package:app/ui/error_notifier.dart';
import 'package:app/ui/home/home_view_model.dart';
import 'package:app/util/ext/context.dart';
import 'package:flutter/material.dart';

import 'package:flutter_hooks/flutter_hooks.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

class HomePage extends HookWidget {
  @override
  Widget build(BuildContext context) {
    final error = useProvider(errorNotifierProvider);
    if (!error.hasBeenHandled) {
      toast(context, error.getErrorIfNotHandled().message);
    }

    return Scaffold(
        appBar: AppBar(
          title: Text(context.localized.home),
          actions: [
            // action button
            IconButton(
              icon: const Icon(Icons.color_lens),
              onPressed: () async =>
                  context.read(appThemeNotifierProvider).toggle(),
            ),
            IconButton(
              icon: const Icon(Icons.refresh),
              onPressed: () =>
                  context.read(homeViewModelNotifierProvider).fetchNews(),
            ),
          ],
        ),
        body: Center(
          child: HookBuilder(
            builder: (context) {
              final homeViewModel = useProvider(homeViewModelNotifierProvider);
              final snapshot = useFuture(useMemoized(
                  () => homeViewModel.fetchNews(),
                  [homeViewModel.news.toString(), error.peekContent()?.type]));

              return snapshot.connectionState == ConnectionState.waiting
                  ? const Loading()
                  : ListView.builder(
                      itemCount: homeViewModel.news.articles.length,
                      itemBuilder: (_, index) {
                        return ArticleItem(
                            article: homeViewModel.news.articles[index]);
                      },
                    );
            },
          ),
        ));
  }
}
```

### Repository

Freezed を使用し Model を自動生成している。
NewsRepository と NewsRepositoryImpl がいる。Repository は不要そうに見えるが MVVM と Repository pattern 実現する上で、ViewModel がどこからデータを取得・更新するのかを意識させないためにビジネスロジックとデータ操作を分離することを目的としているため必要である。たとえば、ネットワークの状況によってサーバから取得するか、ローカルキャッシュを使うかの決定は Repository で行っている。

```dart

/// news_repository.dart
import 'package:app/data/model/news.dart';

abstract class NewsRepository {
  Future<News> getNews();
}
///

/// news_repository_impl.dart
import 'package:app/data/model/news.dart';
import 'package:app/data/remote/news_data_source.dart';
import 'package:app/data/repository/news_repository.dart';
import 'package:flutter/material.dart';

class NewsRepositoryImpl implements NewsRepository {
  NewsRepositoryImpl({@required NewsDataSource dataSource})
      : _dataSource = dataSource;

  final NewsDataSource _dataSource;

  @override
  Future<News> getNews() async {
    return _dataSource.getNews();
  }
}
///

/// news_repository_provider.dart
import 'package:app/data/provider/news_data_source_provider.dart';
import 'package:app/data/repository/news_repository.dart';
import 'package:app/data/repository/news_repository_impl.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

final newsRepositoryProvider = Provider<NewsRepository>(
    (ref) => NewsRepositoryImpl(dataSource: ref.read(newsDataSourceProvider)));
///

```

### DataSource

DataSource は実際にサーバーの API を叩いたり、ローカルから取得したりする処理を書く。DataSource を取得先ごとに作ることが一般的。

```dart

/// news_data_source.dart
import 'package:app/data/model/news.dart';

abstract class NewsDataSource {
  Future<News> getNews();
}
///

/// news_data_source_impl.dart
import 'package:app/constants.dart';
import 'package:app/data/model/news.dart';
import 'package:app/data/remote/news_data_source.dart';
import 'package:app/util/ext/date_time.dart';
import 'package:dio/dio.dart';
import 'package:dio_http_cache/dio_http_cache.dart';
import 'package:flutter/foundation.dart';

class NewsDataSourceImpl implements NewsDataSource {
  NewsDataSourceImpl({@required Dio dio}) : _dio = dio;

  final Dio _dio;

  @override
  Future<News> getNews() async {
    return _dio
        .get<Map<String, dynamic>>(
          '/v2/everything',
          queryParameters: <String, String>{
            'q': 'anim',
            'from': DateTime.now()
                .subtract(
                  const Duration(days: 28),
                )
                .toLocal()
                .formatYYYYMMdd(),
            'sortBy': 'publishedAt',
            'language': 'en',
            'apiKey': Constants.of().apiKey,
          },
          options: buildCacheOptions(const Duration(hours: 1)),
        )
        .then((response) => News.fromJson(response.data));
  }
}
///

/// news_data_source_provider.dart
import 'package:app/data/remote/news_data_source.dart';
import 'package:app/data/remote/news_data_source_impl.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

import 'dio_provider.dart';

final newsDataSourceProvider = Provider<NewsDataSource>(
    (ref) => NewsDataSourceImpl(dio: ref.read(dioProvider)));
///

```

## 参考

- [「minne」はなぜ「MVVM ＋ UseCase ＋ Repository」なのか。3 つのアーキテクチャを選んだ 5 つの理由。](https://logmi.jp/tech/articles/325433)

- [Flutter Hooks の useXXX の使い方](https://qiita.com/mkosuke/items/f88419d0f4d41ed6d858)
- [【Flutter】dio + freezed で API レスポンスを Result<T>で受け取る](https://qiita.com/muttsu-623/items/2fa68fb6689c76f5415f)
- [Flutter : dio のみでの通信基盤作成](https://qiita.com/Morisan/items/0b90505000d5c3183b9b)
- [Flutter 通信ライブラリ選定 ~ 選ばれたのは Retrofit でした ~](https://qiita.com/Morisan/items/9688ab92ff94024353c5)
