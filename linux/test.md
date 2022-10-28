-e で`ファイルがある場合`という意味になる。

```shell
for i in `seq 1 15`
do
if [ $(($i%3)) -eq 0 ] && [ $(($i%5)) -eq 0 ]; then
  echo 'FizzBuzz'
elif [ $(($i%3)) -eq 0 ]; then
  echo 'Fizz'
elif [ $(($i%5)) -eq 0 ]; then
  echo 'Buzz'
else
  echo $i
fi
done

# 結果
1
2
Fizz
4
Buzz
Fizz
7
8
Fizz
Buzz
11
Fizz
13
14
FizzBuzz
```

- [Bash における括弧類の意味](https://qiita.com/yohm/items/3527d517768402efbcb6)
- https://atmarkit.itmedia.co.jp/ait/articles/1807/05/news041.html
