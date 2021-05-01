# 使用手册

## 如何从零开始搭建 Hexo 博客

1. 参考 [Hexo 安装](https://hexo.io/zh-cn/docs/)， 添加 Hexo，接着初始化项目，且命名为 `book`（可以是其他名称）：

```sh
$ hexo init book
```

2. 可以替换原有的主题 `landscape`，比如选择 [matery](https://github.com/xinetzone/matery.git)：

```shell
$ cd themes
$ git clone https://github.com/xinetzone/matery.git
$ cd ..
```

3. 修改配置文件 `book/_config.yml`，以匹配所需。创建 `book/_config.matery` 并删除原有文件 `_config.landscape.yml`。

4. 安装一键部署工具：

```sh
$ cd book
$ npm install https://github.com/xinetzone/hexo-deployer-git.git
```

5. 生成静态网页，并部署到 GitHub Pages：

```sh
$ hexo clean && hexo g
$ hexo d
```

## 直接使用 `design` 作为你的博客模板

1. 进入 [xinetzone/design](https://github.com/xinetzone/design)（修改自 [blinkfox/hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery)） 选择按钮 `Use this template` 即可直接使用。
2. 启动 hexo：

```shell
$ cd book
$ npm i
```

3. 当然，仍需要修改 `book/_config.yml` 和 `book/_config.matery`，以匹配所需。
4. 将此仓库克隆到本地，进入 `book/themes`，添加所需要的主题插件（你也可以替换为其他主题插件）：

```sh
$ git clone https://github.com/xinetzone/matery.git
```
5. 这样便可以开箱即用。
6. 修改 `book/themes/matery/layout/_partial/social-link.ejs` 可以添加新的联系方式。
7. 添加新的目录或者子目录，可以修改 `book/themes/matery/layout/_partial/navigation.ejs` 和 `book/_config.matery.yml` 的 `menu`。、

注意：本项目被维护在 [xinetzone/dao](https://github.com/xinetzone/dao) 的 `template` 分支里面。