# Riverpod + GoRouter

`Initial Route`と`LoginPage`への Rout だけの場合。

```dart
final loginInfo = LoginInfo();

GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomePage(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginPage(),
    ),
  ],
  redirect: (state) {
		// ログインしていない場合
    if (!loginInfo.isLoggedIn) {
			// ログインページにいるかどうか
      return state.subloc == '/login' ? null : '/login';
    }
		// nullを返すと、もともと遷移しようとしていたページへ向かう
		// reirect関数はnullを返すまで処理を呼び続ける
    return null;
  },
)
```

上記の実装方法だと、`LoginInfo`が持つログイン状態が変更されても、`GoRouter`は検知することができない。そのため、ログイン後に別画面に遷移したい場合は、`Login`したことを`LoginPage`で検知して、遷移させる必要がある。
これをログイン状態が変わった際に自動的に`redirect関数`を再度呼び出させて別画面に遷移させたい場合、`refreshListenable`を使用することで可能になる。

`refreshListenable: loginInfo,`

こうすることで、渡した値が変化したとき再度`redirect関数`が呼ばれるようになる。

`redirect関数`内も、`loginInfo`が更新された際に呼び出されることを考慮して、ログイン後は`Initial Routeにredirect`させるコードに修正します。

```dart
redirect: (state) {
  if (!loginInfo.isLoggedIn) {
    return state.subloc == '/login' ? null : '/login';    }
  if (state.subloc == '/login') {
    return '/';
  }
  return null;
},
refreshListenable: loginInfo,
```

subloc が`/login`の場合に`Initial Route`へ redirect させている。
`LoginPage`で何かしらのログイン処理を行い、`LoginInfo`の値が更新されると、`refreshListenable`に`LoginInfo`を渡しているため`redirect関数`が呼ばれる。
`LoginPage`でログイン処理を行なっており、`/login`にいる状態で`redirect関数`が呼ばれるため、subloc が`/login`となります。

これで`LoginInfo`が変更された際に、自動的に遷移させることができるようになる。

`Riverpod`を使用している場合、`LoginInfo`に該当する部分を、`Provider`などで使用していることもあると思いますが、そのままでは`Provider`を参照できないので、`GoRouter`自体を`Provider`で囲んで参照できるようにします。

```dart
final routerProvider = Provider(
  (ref) => GoRouter(
    routes: [
      GoRoute(
        path: '/',
        builder: (context, state) => HomePage(),
      ),
      GoRoute(
        path: '/login',
        builder: (context, state) => LoginPage(),
      ),
    ],
    redirect: (state) {
      final isLoggedIn = ref.read(loginInfoProvider).isLoggedIn;
      if (!isLoggedIn) {
        return state.subloc == '/login' ? null : '/login';
      }

      return null;
    },
    refreshListenable: ref.watch(loginInfoProvider),
  ),
);
```

`Provider`で囲んだことで、`routeInformationParser`と`routerDelegate`を使用している箇所でも変更が必要になるので修正します。

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  return MaterialApp.router(
    routeInformationParser: ref.watch(routerPrivider).routeInformationParser,
    routerDelegate: ref.watch(routerPrivider).routerDelegate,
  );
}
```

これで、`GoRouter`内でも`riverpod`を使用した`Provider`などの値を使用できるようになる。

## TODO（続きまとめる）

- StateNotifierProvider の場合、`refreshListenable`が使えない
- 強制リダイレクト直前の目的の画面に遷移させる

## 参考

- [riverpod + go_router の備忘録](https://zenn.dev/mkikuchi/articles/cc87c84e1404c4)
- [Umigishi-Aoi/go_router_sample](https://github.com/Umigishi-Aoi/go_router_sample/blob/master/go_router_sample/lib/main.dart)
- [【Flutter】go_router で BottomNavigationBar の永続化に挑戦する](https://zenn.dev/heyhey1028/articles/d64564e6fd1df4)
- [【Flutter】Riverpod 入門](https://zenn.dev/naoya_maeda/articles/a8bbf40a202c74)
