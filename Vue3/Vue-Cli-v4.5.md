# Vue Cli
## 目标浏览器
在`package.json`里的`browserslist`字段或者一个单独的`.browserlistrc`文件指定项目的目标浏览器范围。
## 配置
### vue.config.js
vue-cli把webpack的配置文件隐藏了，接受vue.config.js作为可选的配置文件。在项目根目录下（与`package.json`同级）的vue.config.js会被`@vue/cli-service`自动加载。也可以使用`package.json`中的`vue`字段（未尝试验证）。
- publicPath (vue-cli 3.3 以前：baseUrl)
- outputDir
- assetsDir 静态资源目录（相对于`outputDir`）
- indexPath index.html的输出路径，相对于`outputDir`或者使用绝对路径。
- filenameHashing 文件名中包含hash，用来控制缓存。
- pages 多页面应用配置
- lintOnSave 安装了`@vue/cli-plugin-eslint`后才生效。
- 