---
title: 搜狗输入法自定义时间代码片段
date: 2022-08-17 11:39:42
updated: 2022-08-17 11:39:42
tags:
  - 输入法
  - snippet
categories:
  - 收藏安利
  - 代码片段
---

### 浅谈

什么是输入法片段？旨在通过特殊的字符串来输出自定义好的文本，达到便捷输出的目的。[https://pinyin.sogou.com/help.php?list=1&q=9](https://pinyin.sogou.com/help.php?list=1&q=9)

当然除了自己定义外，还有一些内置的短语片段，例如`sj`会输出时间，`xq`会输出星期内容，`rq`会输出具体日期..如图：

[![keyboard-input-snippet-p1](/images/posts/keyboard-input-snippet/p1.png)](/images/posts/keyboard-input-snippet/p1.png)

然后来说说怎么自定义，就拿时间举例...~~因为我觉得内置的`sj`短语不太能满足我的格式需求~~

<!-- more -->

### 添加时间短语

> 输入法-> 属性设置 -> 高级 -> 候选扩展 -> 自定义短语设置 -> 添加新短语

短语除了固定性的内容以外，还可以是模板... `#开头`形如`#xxxxxx`,了解模板的关键字函数：

### 函数公式

| 函数         |       含义       |                                           举例 |
| :----------- | :--------------: | ---------------------------------------------: |
| $year        |     年(4 位)     |                                     2006、2008 |
| $year_yy     |     年(2 位)     |                                         06、08 |
| $month       |        月        |                                       12、8、3 |
| $month_mm    |        月        | 12、08、03 此函数在输入法 3.1 版之后（含）有效 |
| $day         |        日        |                                      3、13、22 |
| $day_dd      |        日        | 03、13、22 此函数在输入法 3.1 版之后（含）有效 |
| $weekday     |       星期       |                                  0、1、2、5、6 |
| $fullhour    |  时(24 小时制)   |                                 02、08、13、23 |
| $halfhour    |   (12 小时制)    |                                 02、08、01、11 |
| $ampm        |    AM、PM(英)    |                                 AM、PM（大写） |
| $minute      |        分        |                                 02、08、15、28 |
| $second      |        秒        |                                 02、08、15、28 |
| $year_cn     |  年(中文 4 位)   |                                       二〇〇六 |
| $year_yy_cn  |  年(中文 2 位)   |                                           〇六 |
| $month_cn    |     月(中文)     |                                   十二、八、三 |
| $day_cn      |     日(中文)     |                               三、十三、二十二 |
| $weekday_cn  |    星期(中文)    |                             日、一、二、五、六 |
| $fullhour_cn | 时(中文 24 时制) |                           二、八、十三、二十三 |
| $halfhour_cn | 时(中文 12 时制) |                               二、八、一、十一 |
| $ampm_cn     |  上午下午(中文)  |                                     上午、下午 |
| $minute_cn   |     分(中文)     |                       零二、零八、十五、二十八 |
| $second_cn   |     秒(中文)     |                       零二、零八、十五、二十八 |

`eg.`设置形如`2022-08-17 11:31:14`格式的短语：

> `#$year-$month_mm-$day_dd $fullhour:$minute:$second`

效果如下：

[![keyboard-input-snippet-p2](/images/posts/keyboard-input-snippet/p2.png)](/images/posts/keyboard-input-snippet/p2.png)
