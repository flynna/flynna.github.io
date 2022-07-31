---
title: git-commit 添加emoji的message显示
date: 2022-07-31 19:00:58
updated: 2022-07-31 19:00:58
tags:
  - git-commit
  - emoji
categories:
  - 土豆の收藏安利
---

### 先看效果

[![git-commit-emoji-p1](/images/posts/git-commit-emoji/p1.png)](/images/posts/git-commit-emoji/p1.png)

<!-- more -->

> 在执行`git commit`的时候，为这次内容提交打上标记（仅 emoji 显示）,凸显此次提交的内容主体类型，也使得在整个历史提交中易于区分查找。

> 当然，这种`emoji`的代码并不强制使用 ~~可以不用，但不能不知道~~

### 格式规范

`:code:`**注意这里的冒号是英文，且与 code 之间没有空格**.

`eg. git commit -m ':bug: 修复了xxx功能'`

### `emoji`代码

| emoji                                     | emoji 代码                   | 说明                  |
| ----------------------------------------- | ---------------------------- | --------------------- |
| `:art:` (调色板)                          | `:art:`                      | 改进代码结构/代码格式 |
| `:zap:` (闪电) `:racehorse:` (赛马)       | `:zap :racehorse:`           | 提升性能              |
| `:fire:` (火焰)                           | `:fire:`                     | 移除代码或文件        |
| `:bug:` (bug)                             | `:bug:`                      | 修复 bug              |
| `:ambulance:` (急救车)                    | `:ambulance:`                | 重要补丁              |
| `:sparkles:` (火花)                       | `:sparkles:`                 | 引入新功能            |
| `:memo:` (备忘录)                         | `:memo:`                     | 撰写文档              |
| `:rocket:` (火箭)                         | `:rocket:`                   | 部署功能              |
| `:lipstick:` (口红)                       | `:lipstick:`                 | 更新 UI 和样式文件    |
| `:tada:` (庆祝)                           | `:tada:`                     | 初次提交              |
| `:white_check_mark:` (白色复选框)         | `:white_check_mark:`         | 增加测试              |
| `:lock:` (锁)                             | `:lock:`                     | 修复安全问题          |
| `:apple:` (苹果)                          | `:apple:`                    | 修复 macOS 下的问题   |
| `:penguin:` (企鹅)                        | `:penguin:`                  | 修复 Linux 下的问题   |
| `:checkered_flag:` (旗帜)                 | `:checked_flag:`             | 修复 Windows 下的问题 |
| `:bookmark:` (书签)                       | `:bookmark:`                 | 发行/版本标签         |
| `:rotating_light:` (警车灯)               | `:rotating_light:`           | 移除 linter 警告      |
| `:construction:` (施工)                   | `:construction:`             | 工作进行中            |
| `:green_heart:` (绿心)                    | `:green_heart:`              | 修复 CI 构建问题      |
| `:arrow_down:` (下降箭头)                 | `:arrow_down:`               | 降级依赖              |
| `:arrow_up:` (上升箭头)                   | `:arrow_up:`                 | 升级依赖              |
| `:construction_worker:` (工人)            | `:construction_worker:`      | 添加 CI 构建系统      |
| `:chart_with_upwards_trend:` (上升趋势图) | `:chart_with_upwards_trend:` | 添加分析或跟踪代码    |
| `:hammer:` (锤子)                         | `:hammer:`                   | 重大重构              |
| `:heavy_minus_sign:` (减号)               | `:heavy_minus_sign:`         | 减少一个依赖          |
| `:heavy_plus_sign:` (加号)                | `:heavy_plug_sign:`          | 增加一个依赖          |
| `:whale:` (鲸鱼)                          | `:whale:`                    | Docker 相关工作       |
| `:wrench:` (扳手)                         | `:wrench:`                   | 修改配置文件          |
| `:globe_with_meridians:` (地球)           | `:globe_with_meridians:`     | 国际化与本地化        |
| `:pencil2:` (铅笔)                        | `:pencil2:`                  | 修复 typo             |
