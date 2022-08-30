# Flutter を MVVM で実装する

![](https://miro.medium.com/max/1400/1%2AUj5dnm3RTQ89uidDpcObHw.jpeg)

Repository patternはデータソースへのアクセスを抽象化するためのデザインパターン。
Repository採用の理由は3つ。

- データアクセスを再利用するため
- データの取得時に、「どこからデータを取ってくるか？」という記述をPresentationレイヤーから遮断するため
- データアクセスにおいてライブラリ依存を解決するため。

「Presentationレイヤーはどこからデータを取ってくるか知る必要はなくて、データさえ取れればよい」という考え方。ライブラリに破壊的な変更があった倍位にも、Ripositoryのレイヤーの修正だけで済み、保守性も高まる。

UseCaseは？

![](https://img.logmi.jp/article_images/XyBh9eUzXdT7FMNCu5CHtj.png)

![](https://img.logmi.jp/article_images/LPMe8e36asQqurG4U2L8J9.png)

クリーンアーキテクチャの中では「アプリケーション固有のビジネスロジックを担当するレイヤー」とある。

![](https://img.logmi.jp/article_images/16YGGqgLo7WuazEikPo5qN.png)

Repositoryをデータアクセスのためのレイヤーとしてしまうと、中規模以上の開発の場合、マッピングやバリデーションをViewModelに記述せざるを得なくなり、FatViewModelとなってしまう。

![](https://img.logmi.jp/article_images/W3JVWXUHK6xW1b1mnGfvoD.png)



## Flutter Architecture Blueprints

[Flutter Architecture Blueprints](https://github.com/wasabeef/flutter-architecture-blueprints)

*ChangeNotifier*
- 値が変更されたことを通知することができるObservable。
- ViewModelがModelを複数持つような場合、StateNotifierだと1つのクラスしか持てない。

### ViewModel

ChangeNotifierを継承している。

HomeViewModelはChangeNotifierを継承しているため、ChangeNotifierProviderを使用することでHomeViewModelを生成し、後述するView側で読み取ることができる。

扱うデータはnewsで、データが更新されたことを通知したい場合にnotifyListenerメソッドを呼ぶことでView側に通知することができるObservable。

NewsRepositoryのgetNewsというメソッドを使って、newsデータを取ってきている。

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

useProviderは、ConsumerWidgetの場合watchすることで再現可能。

HookBuilderもFlutter Hooksのクラスである。このビルダーの中でuseProviderを生成して、その値が変更された場合にはこの中だけでリビルドが起こる。Flutter Hooksを使わずにRiverpodだけでやる場合には、Comsumerで同様のことが可能。

useFutureはFutureBuilderと同じ動作をする。

:::details useFutureの例

FutureやStreamを扱うもので、FutureBuilderやStreamBuilderが簡単に書けるようになる。以下はpackage_infoを用いてアプリ情報を取得する場合の記述の仕方。

*Hooksなし*

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

*Hooksあり*

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

useMemoizedはAsyncMemoizerと似たよう動作で、配列で渡しているKeysの値が同じならばfetchNewsメソッド部分のキャッシュを返してリビルドしない。ここではnewsデータをtoStringしたものをキーに含めている。

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

Freezedを使用しModelを自動生成している。
NewsRepositoryとNewsRepositoryImplがいる。Repositoryは不要そうに見えるがMVVMとRepository pattern実現する上で、ViewModelがどこからデータを取得・更新するのかを意識させないためにビジネスロジックとデータ操作を分離することを目的としているため必要である。たとえば、ネットワークの状況によってサーバから取得するか、ローカルキャッシュを使うかの決定はRepositoryで行っている。

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

DataSourceは実際にサーバーのAPIを叩いたり、ローカルから取得したりする処理を書く。DataSourceを取得先ごとに作ることが一般的。

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

- [Flutter HooksのuseXXXの使い方](https://qiita.com/mkosuke/items/f88419d0f4d41ed6d858)
- [【Flutter】dio + freezed でAPIレスポンスをResult<T>で受け取る](https://qiita.com/muttsu-623/items/2fa68fb6689c76f5415f)
- [Flutter : dioのみでの通信基盤作成](https://qiita.com/Morisan/items/0b90505000d5c3183b9b)
- [Flutter通信ライブラリ選定 ~ 選ばれたのはRetrofitでした ~](https://qiita.com/Morisan/items/9688ab92ff94024353c5)