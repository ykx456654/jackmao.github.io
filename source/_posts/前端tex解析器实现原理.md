# 实现一个前端tex解析器
tex文本本质上是一串有很多反斜杠`\`的有规律的字符串，由于其可读性较差，就有了mathjax、katex等tex解析库，把tex语法编译成html+css。

## 基本的tex词法
TeX中通常每个字符视为一个标记，`\`与后面的尽可能多的字母（直到遇到 空格 或者 [,{ 等标识符）组成宏

一般是这样的格式 `\command[arg1]{arg2}{arg3}` 前面的一部分是指宏，方括号中是可选参数，花括号中是必选参数

这些控制序列是有特殊含义的命令。比如 `\frac{1}{2}`表示一个分式: 
    $$\frac{1}{2}$$
`\sqrt{100}`表示一个根式
    $$\sqrt{100}$$

## 词法分析
我们拿到一个tex文本，第一步是把输入一个词一个词的拆分开。

这一步被叫做 `词法分析`,或者说是分词。这一步的关键就在于 我们把字符组合成我们需要的单词、标识符、符号等等。

词法分析大多都不需要处理逻辑运算像是算出 1+2: 
    其实这个表达式只有三种 标记：一个数字：1,一个加号+，另外一个数字：2。

而对于tex文本，我们第一步就是要通过[正则]把tex文本解析成一个个命令 与 文本:
    `\frac{1}{2}` => 

```javaScript
[
    {
        "text": "\\frac"
    },
    {
        "text": "{"
    },
    {
        "text": "1"
    },
    {
        "text": "}"
    },
    {
        "text": "{"
    },
    {
        "text": "2"
    },
    {
        "text": "}"
    }
]
```

### 建立正则来提取tex命令与文本
```javaScript
const spaceRegexString = "[ \r\n\t]";
const controlWordRegexString = "\\\\[a-zA-Z@]+";
const controlSymbolRegexString = "\\\\[^\uD800-\uDFFF]";
const controlWordWhitespaceRegexString =
    `${controlWordRegexString}${spaceRegexString}*`;
const controlWordWhitespaceRegex = new RegExp(
    `^(${controlWordRegexString})${spaceRegexString}*$`);
const combiningDiacriticalMarkString = "[\u0300-\u036f]";
export const combiningDiacriticalMarksEndRegex =
    new RegExp(`${combiningDiacriticalMarkString}+$`);
const tokenRegexString = `(${spaceRegexString}+)|` +  // whitespace
    "([!-\\[\\]-\u2027\u202A-\uD7FF\uF900-\uFFFF]" +  // single codepoint
    `${combiningDiacriticalMarkString}*` +            // ...plus accents
    "|[\uD800-\uDBFF][\uDC00-\uDFFF]" +               // surrogate pair
    `${combiningDiacriticalMarkString}*` +            // ...plus accents
    "|\\\\verb\\*([^]).*?\\3" +                       // \verb*
    "|\\\\verb([^*a-zA-Z]).*?\\4" +                   // \verb unstarred
    `|${controlWordWhitespaceRegexString}` +          // \macroName + spaces
    `|${controlSymbolRegexString})`;

```

## 解析成语法树
经过词法解析的tex文本，虽然是一个数组，但是并没有太多的语义信息跟模式，我们要在这些项中找出有特殊意义的宏命令，并将它们解析成一个有层级结构的语法树。有了语法树，我们才能用html + css把这段tex文本展示出来。


### 递归解析子式
根据tex语法，第一个`{}`中的是分子， 第二个`{}`内的是分母。其中，每个`{}`内包含的式子都是一个独立的子式，如果说子式里面还有这种结构式的情况，就需要我们递归来处理这些子式，递归的结束条件就是，当前子式里面所有项都是同级元素，不存在更深的子式。

首先我们得找出独立的子式的部分，将独立的子式部分划分为一个数组，代表这这个数组里每个节点都是同级渲染的。

例如： 
$$ \frac {\sqrt {123 + \sqrt[9] {2}} } {2_2^2} $$



```[{"text":"\\frac"},{"text":"{"},{"text":"\\sqrt"},{"text":"{"},{"text":"1"},{"text":"2"},{"text":"3"},{"text":"+"},{"text":"\\sqrt"},{"text":"["},{"text":"9"},{"text":"]"},{"text":"{"},{"text":"2"},{"text":"}"},{"text":"}"},{"text":"}"},{"text":"{"},{"text":"2"},{"text":"_"},{"text":"2"},{"text":"^"},{"text":"2"},{"text":"}"}]```

这个公式解析的结构化子式如下：

```javaScript
[
    {                         // 命令
        "text": "\\frac"
    },
    [                         // 独立的子式
        {
            "text": "\\sqrt"
        },
        [
            {
                "text": "1"
            },
            {
                "text": "2"
            },
            {
                "text": "3"
            },
            {
                "text": "+"
            },
            {
                "text": "\\sqrt"
            },
            [
                {
                    "text": "9"
                }
            ],
            [
                {
                    "text": "2"
                }
            ]
        ]
    ],
    [                       // 独立的子式
        {
            "text": "2"
        },
        {
            "text": "_"
        },
        {
            "text": "2"
        },
        {
            "text": "^"
        },
        {
            "text": "2"
        }
    ]
]

```


### 分析关键词，解析成语法树

虽然有了结构化的式子，但是这只是一个单纯的数据结构，我们还不能直接拿这个数据渲染成可视公式，我们还需要进一步的根据式子里面的关键词（命令），来生成一颗更加清晰的树，好方便我们渲染成HTML。

#### 关键词的处理


latex的数学公式数目是有限的，每一个命令都对应着相应的语法，就有一个处理这个命令与其子式的函数。
    就像之前有提到过的分式[\frac]， 它后面的第一个子式是分子， 第二个子式是分母，那么我们就可以取上面得到的数据结构里面
    `{ text: '\\frac' }`
    后面的两个子式， 来与它合成一个节点, 这个节点就是这个公式的语法树上的一个节点。
    类似的， 根式[\sqrt]后面的 [[]] 子式是根指数， [{}] 内子式的根底数,  我们也可以把他们合成为一个树节点。

    上面的式子解析成树如下：
```
[
    {
        "type": "frac",
        "numer": [   //  分子子式
            {
                "type": "sqrt"   // 根式
                "index": null,   //  没有根指数
                "body": [     // 跟底数子式
                    {
                        "body": {    //  普通的文本节点
                            "text": "1"
                        },
                        "type": "text"
                    },
                    {
                        "body": {
                            "text": "2"
                        },
                        "type": "text"
                    },
                    {
                        "body": {
                            "text": "3"
                        },
                        "type": "text"
                    },
                    {
                        "body": {
                            "text": "+"
                        },
                        "type": "text"
                    },
                    {
                        "type": "sqrt"    // 根式
                        "index": [   //  根指数子式
                            {
                                "text": "9"
                            }
                        ],
                        "body": [  // 跟底数子式
                            {
                                "text": "2"
                            }
                        ],
                    }
                ],
            }
        ],
        "denom": [  // 分母子式
            {
                "body": {
                    "text": "2"
                },
                "type": "subsup",
                "sub": {   //   下标子式
                    "text": "2"
                },
                "sup": {  //   上标子式
                    "text": "2"
                }
            }
        ]
    }
]
```


## 渲染treeNode

得到上面的语法树后，我们只需要将相应的节点，按规则渲染成dom就行了。
比如`\frac`是分式结构，上下各一个子式，再加一条线就行了。
$$\frac{分子子式}{分母子式}$$

### builder:

`\frac`:

```javaScript
export default class Frac extends Component {
    render() {
        const { denom, numer } = this.props;
        // console.log(denom)
        return (
            <span className="frac">
                <span className="frac-numer">{numer}</span>
                <span className="frac-line" />
                <span className="frac-denom">{denom}</span>
            </span>
        );
    }
}
```

`\sqrt`:

```javaScript


export default class Sqrt extends Component {
    render() {
        return (
            <span className="sqrt-wrapper">
                {/* 根指数 */}
                {rootIndex &&
                    <span className="sqrt-root-index" ref={(rootIndexEl) => this.rootIndexEl = rootIndexEl}>
                    {rootIndex}
                    </span>
                }
                <span className="sqrt">
                    {/* 根式 */}
                    <svg className="sqrt-svg" viewBox="0,0,400000,1080" 
                        style={{ height: '1.08em', width: '400em' }}>
                        <path d={sqrtPath} stroke="#000" strokeWidth="1" />
                    </svg>
                    {/* 根底数 */}
                    <span className="sqrt-root-base"> {rootBase}</span>
                </span>
            </span>
        );
    }
}
```

