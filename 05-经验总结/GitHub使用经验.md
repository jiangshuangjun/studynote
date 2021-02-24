## GitHub 使用经验

<span id = "0"/>

- [常用词含义](#1)
- [in 限制搜索](#2)
- [stars 和 fork 范围搜索](#3)
- [awesome 加强搜索](#4)
- [高亮显示某一行代码](#5)
- [项目内搜索](#6)
- [搜索区域活跃用户](#7)

---

<span id = "1"/>

<br/>

### 常用词含义

- watch

  > 会持续受到该项目的动态

- fork

  > 复制某个项目到自己的 GitHub 仓库中

- star

  > 可以理解为点赞

- clone

  > 将项目下载至本地

- follow

  > 关注你感兴趣的作者，会收到他们的动态

<span id = "2"/>

<br/>

### in 限制搜索

- 公式

  > xxx 关键词 in:name 或 description 或 readme

- xxx in:name

  > 项目名包含 xxx 的

- xxx in:description

  > 项目描述包含 xxx 的

- xxx in:readme

  > 项目的 readme 文件中包含 xxx 的

- 组合使用

  > 搜索项目名或者 readme 中包含秒杀的项目
  >
  > seckill in:name,readme

<span id = "3"/>

<br/>

### stars 和 fork 范围搜索

- 公式

  - xxx 关键词 stars 通配符

    > :> 或者 :>=

  - 区间范围数字

    > 数字1..数字2

- 查找 stars 数大于等于 5000 的 springboot 项目

  > springboot stars:>=5000

- 查找 forks 数大于 500 的springcoud 项目

  > springcloud forks:>500

- 组合使用

  > 查找 fork 在 2000 到 4000 之间并且 stars 数在 6000 到 8000 之间的 springboot 项目
  >
  > springboot forks:2000..4000 stars:6000..8000

<span id = "4"/>

<br/>

### awesome 加强搜索

- 公式

  > awesome 关键字
  >
  > awesome 系列一般是用来收集学习、工具、书籍类相关的

- 搜索优秀的 redis 相关的项目，包括框架、教程等

  > awesome redis

<span id = "5"/>

<br/>

### 高亮显示某一行代码

- 公式

  - 1行

    > 地址后面紧跟 #L数字

  - 多行

    > 地址后面紧跟 #L数字1-L数字2

<span id = "6"/>

<br/>

### 项目内搜索

- 英文字母 t
- [更多快捷键-帮助文档](https://help.github.com/en/articles/using-keyboard-shortcuts)

<span id = "7"/>

<br/>

### 搜索区域活跃用户(某个地区内的大佬)

- 公式

  - location:地区
  - language:语言
  - pushed:>YYYY-MM-DD // 最后更新时间大于 YYYY-MM-DD

- 例：地区北京的 Java 方向的用户

  > location:beijing language:java