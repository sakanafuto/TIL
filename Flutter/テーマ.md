# Theme

`Theme Data クラス`をいじってデフォルトのフォントサイズやカラースキーマを変更した。
外出しした`AppThemeData クラス`を、`MaterialApp`で指定する。

```dart
return MaterialApp(
  ...
  theme: AppThemeData.mainThemeData,
);
```

```bash
 lib
├──  main.dart
├──  theme
│   └──  app_theme.dart
```

テーマの例

```dart 
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

class AppThemeData {
  static const _fillColor = Colors.black;
  static final Color _focusColor = Colors.black.withOpacity(0.2);

  static ThemeData mainThemeData = themeData(mainColorScheme);

  static ThemeData themeData(ColorScheme colorScheme) {
    return ThemeData(
      useMaterial3: true,
      colorScheme: colorScheme,
      textTheme: _textTheme,
      appBarTheme: AppBarTheme(
        backgroundColor: colorScheme.primaryContainer,
        titleTextStyle: TextStyle(color: colorScheme.secondary),
        elevation: 0,
        iconTheme: IconThemeData(color: colorScheme.primary),
      ),
      bottomNavigationBarTheme: BottomNavigationBarThemeData(
        type: BottomNavigationBarType.fixed,
        selectedItemColor: colorScheme.primaryContainer,
        unselectedItemColor: colorScheme.secondary,
      ),
      iconTheme: IconThemeData(color: colorScheme.onPrimary),
      // canvasColor: colorScheme.background,
      highlightColor: Colors.transparent,
      focusColor: _focusColor,
    );
  }

  static const ColorScheme mainColorScheme = ColorScheme(
    brightness: Brightness.light,
    primary: Color.fromRGBO(200, 200, 200, 1),
    primaryContainer: Color.fromRGBO(150, 150, 150, 1),
    secondary: Color.fromRGBO(230, 230, 230, 1),
    secondaryContainer: Color.fromRGBO(210, 210, 210, 1),
    background: Color(0xFFFAFBFB),
    surface: Color(0xFFFAFBFB),
    onBackground: Color(0xFFFAFBFB),
    error: _fillColor,
    onError: _fillColor,
    onPrimary: _fillColor,
    onSecondary: _fillColor,
    onSurface: _fillColor,
  );

  static final TextTheme _textTheme = TextTheme(
    headline1: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w700, fontSize: 24.0),
    headline2: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w700, fontSize: 22.0),
    headline3: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w700, fontSize: 20.0),
    headline4: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w500, fontSize: 18.0),
    headline5: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w500, fontSize: 16.0),
    headline6: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w500, fontSize: 16.0),
    bodyText1: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w400, fontSize: 14.0),
    bodyText2: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w400, fontSize: 16.0),
    subtitle1: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w500, fontSize: 16.0),
    subtitle2: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w500, fontSize: 14.0),
    caption: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w600, fontSize: 14.0),
    button: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w600, fontSize: 14.0),
    overline: GoogleFonts.sawarabiGothic(fontWeight: FontWeight.w500, fontSize: 12.0),
  );
}
```


## 参考
[[Flutter]ThemeData classについての個人的まとめ
](https://zenn.dev/t_fukuyama/articles/ea5a424433f2ff)