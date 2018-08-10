---
layout:     post
title:      "levenshtein距离"
subtitle:   ""
date:       2018-08-10 10:52:29
author:     "Kai"
header-img: "img/post-bg-e2e-ux.jpg"
catalog: true
tags:
    - ng6
    - cli
    - 纠错
---



在 ng6 的cli源码中（command-runner.js）有个识别错误命令的方法，用来给用户返回相似的命令并以此来纠正用户的误输入。里面用到了Levenshtein距离理论：
> 在信息论，语言学和计算机科学中，levenshtein距离是用于测量两个序列之间差异的字符串度量。非正式地，两个单词之间的Levenshtein距离是将一个单词更改为另一个单词所需的单字符编辑（插入，删除或替换）的最小数量。它以苏联数学家弗拉基米尔·莱文斯坦（Vladimir Levenshtein）的名字命名，他在1965年考虑过这个距离。Levenshtein距离也可以称为编辑距离，尽管该术语也可以表示更大的距离度量系列。它与成对字符串对齐密切相关。

### 举个例子

这两个单词`kitten`and`sitting`的levenshtein距离是 3。
1. __k__itten → sitten (第一步 `k` 替换成 `d`)
2. sitten → sittin (第二步 `e` 替换成 `i`)
3. sittin → sitting (最后加上`g`)

这是最少的步数<br>
我一家之言 肯定有人不信 所以这个牛逼的数学家才弄了个算法计算出这最少的步数。

```js
function levenshtein(a, b) {
    /* base case: empty strings */
    if (a.length == 0) {
        return b.length;
    }
    if (b.length == 0) {
        return a.length;
    }
    // 测试最后的字符是否相等
    const cost = a[a.length - 1] == b[b.length - 1] ? 0 : 1;
    /* 从a去除一个字符、从b去除一个字符、都去除一个字符之间取最小 */
    return Math.min(
        levenshtein(a.slice(0, -1), b) + 1, 
        levenshtein(a, b.slice(0, -1)) + 1, 
        levenshtein(a.slice(0, -1), b.slice(0, -1)) + cost
    );
}
```
这里用的递归来实现的，效率较低
[codepen](https://codepen.io/kavil/pen/ZjVvob?editors=1111),
但angular-cli只是用他来简单纠错，并没运用在代码项目里还是可以接受的。
levenshtein方法更适用于机器学习纠错等领域，相信以后会有更多小伙伴关注levenshtein。

> 参考 [https://en.wikipedia.org/wiki/Levenshtein_distance](https://en.wikipedia.org/wiki/Levenshtein_distance)