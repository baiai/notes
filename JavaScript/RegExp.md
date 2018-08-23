## 创建一个RegExp对象

```js
let pattern = /ja/gim;
let pattern = new RegExp('ja', 'gim');
```

## 常用方法

exec：返回一个数组arr，arr[0]是匹配的字符，arr[1...n]是第i个子表达式匹配的文本。返回的数组还有两个属性：index匹配字符的起始位置，input表示正在检索的字符串。而正则表达式对象也有五个属性：

`lastIndex`：表示表达式从待检索的字符串那个位置开始检索，一个字符串对象；

`ignoreCase`：boolean值，表示是否忽略大小写；

`global`：boolean值，表示是否全局搜索；

`multiline`：boolean值，表示是否多行搜索；

`source`：RegExp表示的文本

示例：

```js
var re = /quick\s(brown).+?(jumps)/ig;
var result = re.exec('The Quick Brown Fox Jumps Over The Lazy Dog');
```

| Object   | Property/Index   | Description                                                  | Example                                       |
| -------- | ---------------- | ------------------------------------------------------------ | --------------------------------------------- |
| `result` | `[0]`            | The full string of characters matched                        | `Quick Brown Fox Jumps`                       |
|          | `[1], ...[*n* ]` | The parenthesized substring matches, if any. The number of possible parenthesized substrings is unlimited. | `[1] = Brown[2] = Jumps`                      |
|          | `index`          | The 0-based index of the match in the string.                | `4`                                           |
|          | `input`          | The original string.                                         | `The Quick Brown Fox Jumps Over The Lazy Dog` |
| `re`     | `lastIndex`      | The index at which to start the next match. When "g" is absent, this will remain as 0. | `25`                                          |
|          | `ignoreCase`     | Indicates if the "`i`" flag was used to ignore case.         | `true`                                        |
|          | `global`         | Indicates if the "`g`" flag was used for a global match.     | `true`                                        |
|          | `multiline`      | Indicates if the "`m`" flag was used to search in strings across multiple lines. | `false`                                       |
|          | `source`         | The text of the pattern.                                     | `quick\s(brown).+?(jumps)`                    |

test()：返回一个boolean，表示待检索字符串是否含有符合RegExp的字符串。

示例：

```js
var str = 'hello world!';
var result = /^hello/.test(str);
console.log(result); // true
```

正则表达式还可以用于string类型的变量的模式匹配：

`str.search(regexp)`：返回字符串中第一个匹配正则的位置，没找到返回-1；

`str.match(regexp)`：没有找到返回null，如果找到则返回数组，如果regexp带有全局flag，返回的是匹配正则表达式的所有字符串。如果没有全局flag，但会的是匹配的字符串以及index和input。和exec一样；

`str.replace(regexp, replacement)`：找到匹配regexp的字符串并用replacement替换掉；

`str.split(regexp, num)`：以匹配正则的子字符串分裂字符串，返回一个数组。num是可选，表示的是返回的数组的长度。



## 书写规则

### 方括号

| 表达式             | 描述                               |
| ------------------ | ---------------------------------- |
| [abc]              | 查找方括号之间的任何字符。         |
| [^abc]             | 查找任何不在方括号之间的字符。     |
| [0-9]              | 查找任何从 0 至 9 的数字。         |
| [a-z]              | 查找任何从小写 a 到小写 z 的字符。 |
| [A-Z]              | 查找任何从大写 A 到大写 Z 的字符。 |
| [A-z]              | 查找任何从大写 A 到小写 z 的字符。 |
| [adgk]             | 查找给定集合内的任何字符。         |
| [^adgk]            | 查找给定集合外的任何字符。         |
| (red\|blue\|green) | 查找任何指定的选项。               |

### 元字符

| 元字符 | 描述                                        |
| ------ | ------------------------------------------- |
| .      | 查找单个字符，除了换行和行结束符。          |
| \w     | 查找单词字符。                              |
| \W     | 查找非单词字符。                            |
| \d     | 查找数字。                                  |
| \D     | 查找非数字字符。                            |
| \s     | 查找空白字符。                              |
| \S     | 查找非空白字符。                            |
| \b     | 匹配单词边界。                              |
| \B     | 匹配非单词边界。                            |
| \0     | 查找 NUL 字符。                             |
| \n     | 查找换行符。                                |
| \f     | 查找换页符。                                |
| \r     | 查找回车符。                                |
| \t     | 查找制表符。                                |
| \v     | 查找垂直制表符。                            |
| \xxx   | 查找以八进制数 xxx 规定的字符。             |
| \xdd   | 查找以十六进制数 dd 规定的字符。            |
| \uxxxx | 查找以十六进制数 xxxx 规定的 Unicode 字符。 |

### 量词

| 量词   | 描述                                        |
| ------ | ------------------------------------------- |
| n+     | 匹配任何包含至少一个 n 的字符串。           |
| n*     | 匹配任何包含零个或多个 n 的字符串。         |
| n?     | 匹配任何包含零个或一个 n 的字符串。         |
| n{X}   | 匹配包含 X 个 n 的序列的字符串。            |
| n{X,Y} | 匹配包含 X 至 Y 个 n 的序列的字符串。       |
| n{X,}  | 匹配包含至少 X 个 n 的序列的字符串。        |
| n$     | 匹配任何结尾为 n 的字符串。                 |
| ^n     | 匹配任何开头为 n 的字符串。                 |
| ?=n    | 匹配任何其后紧接指定字符串 n 的字符串。     |
| ?!n    | 匹配任何其后没有紧接指定字符串 n 的字符串。 |

