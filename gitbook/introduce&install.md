## GitBook介绍以及安装

### Gitbook介绍

GitBook 是一个基于 Node.js 的命令行工具，支持 Markdown 和 AsciiDoc 两种语法格式，可以输出 HTML、PDF、eBook 等格式的电子书。所以我更喜欢把 GitBook 定义为文档格式转换工具。

所以，GitBook 不是 Markdown 编辑工具，也不是 Git 版本管理工具。

### Gitbook安装

因为 GitBook 是基于 Node.js，所以我们首先需要安装 [Node.js](https://nodejs.org/en/download/)，找到对应平台的版本安装即可。

现在安装 Node.js 都会默认安装 npm（node 包管理工具），所以我们不用单独安装 npm，打开命令行，执行以下命令安装 GitBook：

> npm install -g gitbook-cli
>
> 查看gitbook是否成功安装
>
> gitbook -V 
>
> 列出gitbook所有命令
>
> gitbook help
>
> 输出gitbook-cli的帮助信息
>
> gitboot --help

