# 编译资源 (Laravel Elixir)

- [简介](#introduction)
- [安装 & 设置](#installation)
- [运行 Elixir](#running-elixir)
- [处理样式表](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [原生 CSS](#plain-css)
    - [Source Maps](#css-source-maps)
- [处理脚本](#working-with-scripts)
    - [Webpack](#webpack)
    - [Rollup](#rollup)
    - [脚本](#javascript)
- [拷贝文件 & 目录](#copying-files-and-directories)
- [版本号 / 缓存刷新](#versioning-and-cache-busting)
- [BrowserSync](#browser-sync)

<a name="introduction"></a>
## 简介

Laravel Elixir 提供了一套干净、平滑的 API 用于为 Laravel 应用定义基本的 [Gulp](http://gulpjs.com) 任务。Elixir 支持一些通用的 CSS，例如 [Sass](http://sass-lang.com) 和 JavaScript 预处理器，例如 [Webpack](https://webpack.github.io/)，甚至测试工具。使用方法链，Elixir 允许你平滑的定义资源管道。例如：

```javascript
elixir(function(mix) {
    mix.sass('app.scss')
       .webpack('app.js');
});
```

如果你曾经对如何使用 Gulp 和编译前端资源感到困惑，那么你会爱上 Laravel Elixir。不过，并不是强制要求在开发期间使用它。你可以自由选择使用任何前端资源管道工具，或者压根不使用。

<a name="installation"></a>
## 安装 & 设置

#### 安装 Node

在开始 Elixir 之前，必须首先确保 Node.js 和 npm 在机器上已经安装：

    node -v
    npm -v

默认情况下，Laravel Homestead 包含你需要的一切；然而，如果你不使用 Vagrant，你也可以通过访问 Node 的 [下载页面](http://nodejs.org/en/download/) 轻松的安装 Node。

#### Gulp

接下来，需要安装 [Gulp](http://gulpjs.com) 作为全局 NPM 包：

    npm install --global gulp-cli

#### Laravel Elixir

剩下的唯一步骤是安装 Laravel Elixir。 在一个全新安装的 Laravel 中，你会在目录结构的根目录中找到一个 `package.json` 文件。 默认的 `package.json` 文件包括 Elixir 和 Webpack 的模块。 该文件和 `composer.json` 一样，只不过是用来定义 Node 依赖而非 PHP 依赖，你可以通过运行如下命令来安装需要的依赖：

    npm install

如果你正在 Windows 系统上开发，需要在运行 `npm install` 命令时带上 `--no-bin-links`：

    npm install --no-bin-links

<a name="running-elixir"></a>
## 运行 Elixir

Elixir 基于 [Gulp](http://gulpjs.com)，所以要运行 Elixir 命令你只需要在终端中运行 `gulp` 命令即可。添加 `--production` 标识到命令将会最小化 CSS 和 JavaScript 文件：

    // Run all tasks...
    gulp

    // Run all tasks and minify all CSS and JavaScript...
    gulp --production

运行这个命令的时候，你会看到一个显示刚刚发生事件的格式良好的表格。

#### 监控前端资源改变

`gulp watch` 命令将持续在终端运行，并监视资源的任何更改。 如果在 `watch` 命令运行时资源发生改变，Gulp 将自动重新编译：

    gulp watch

<a name="working-with-stylesheets"></a>
## 处理样式表

项目根目录下的 `gulpfile.js` 文件包含了所有的 Elixir 任务。Elixir 任务可以使用方法链的方式链接起来用于定义前端资源如何被编译。

<a name="less"></a>
### Less

要将 [Less](http://lesscss.org/) 编译成 CSS，可以使用 `less` 方法。`less` 方法假定你的 Less 文件都放在 `resources/assets/less`。默认情况下，该任务会将编译后的 CSS 放到 `public/css/app.css`：

```javascript
elixir(function(mix) {
    mix.less('app.less');
});
```

你还可以将多个 Less 文件编译成单个 CSS 文件。同样，该文件会被放到 `public/css/app.css`：

```javascript
elixir(function(mix) {
    mix.less([
        'app.less',
        'controllers.less'
    ]);
});
```

如果你想要自定义编译后文件的输出位置，可以传递第二个参数到 `less` 方法：

```javascript
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets');
});

// Specifying a specific output filename...
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets/style.css');
});
```

<a name="sass"></a>
### Sass

`sass` 方法允许你将 [Sass](http://sass-lang.com/) 编译成 CSS。假定你的 Sass 文件存放在 `resources/assets/sass`，你可以像这样使用该方法：

```javascript
elixir(function(mix) {
    mix.sass('app.scss');
});
```

同样，和 `less` 方法一样，你可以将多个脚本编译成单个 CSS 文件，甚至自定义结果 CSS 的输出路径：

```javascript
elixir(function(mix) {
    mix.sass([
        'app.scss',
        'controllers.scss'
    ], 'public/assets/css');
});
```

#### 自定义路径

尽管我们推荐你使用默认的前端资源目录，但是如果你确实需要设置其它目录，可以在相应文件路径前加上 `./` ，这表明 Elixir 将会从项目根目录开始寻找文件，而不是使用默认根目录。

例如，要编译 `app/assets/sass/app.scss` 然后将编译后文件输出到 `public/css/app.css` ，可以这样调用 `sass` 方法：

```javascript
elixir(function(mix) {
    mix.sass('./app/assets/sass/app.scss');
});
```

<a name="stylus"></a>
### Stylus

`stylus` 方法可用于将 [Stylus](http://stylus-lang.com/) 编译成 CSS。假设你的 Stylus 文件存放在 `resources/assets/stylus` ，那么可以这样调用该方法：

```javascript
elixir(function(mix) {
    mix.stylus('app.styl');
});
```

> {tip} 该方法的调用和 `mix.less()` 、`mix.sass()` 类似。

<a name="plain-css"></a>
### 原生 CSS

如果你只想要将多个原生 CSS 样式文件合并到一个文件，可以使用 `styles` 方法。传递给该方法的路径相对于 `resources/assets/css`  目录，结果 CSS 被存放在 `public/css/all.css`：

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ]);
});
```

当然，你还可以通过传递第二个参数到 `styles` 方法来输出结果文件到一个自定义目录或文件：

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ], 'public/assets/css/site.css');
});
```

<a name="css-source-maps"></a>
### Source Maps

在Elixir中，源地图默认被启用，以便在编译前端资源的时候为浏览器开发者工具提供更好的调试信息。对于每一个被编译过的文件都可以在同一目录下找到一个对应的 `*.css.map` 或 `*.js.map` 文件。

如果你不想让应用 生成源地图，可以通过 `sourcemaps` 配置选项关闭它们：

```javascript
elixir.config.sourcemaps = false;

elixir(function(mix) {
    mix.sass('app.scss');
});
```

<a name="working-with-scripts"></a>
## 处理脚本

Elixir 还提供了多个函数帮助你处理 JavaScript 文件，例如编译 ECMAScript 2015，模块管理，最小化以及合并原生JavaScript文件。

使用模块编写ES2015的时候，可以选择 [Webpack](http://webpack.github.io) 或 [Rollup](http://rollupjs.org/)，如果你对这些工具很陌生，别担心，Elixir 会为你处理所有背后的复杂逻辑。默认情况下，Laravel 的 `gulpfile` 使用 `webpack` 来编译 JavaScript，当然，你也可以选择使用自己喜欢的模块管理器。

<a name="webpack"></a>
### Webpack

`webpack` 方法用于将 [ECMAScript 2015](https://babeljs.io/docs/learn-es2015/) 编译打包成原生 JavaScript，该方法接收一个相对于 `resources/assets/js` 目录的文件路径，然后在 `public/js` 目录下生成单个打包文件：

```javascript
elixir(function(mix) {
    mix.webpack('app.js');
});
```

要选择其他输出目录，只需在目标路径前加上 `.` 前缀，然后指定一个相对于应用根目录的相对路径。例如，要编译 `app/assets/js/app.js` 到 `public/dist/app.js`：

```javascript
elixir(function(mix) {
    mix.webpack(
        './app/assets/js/app.js',
        './public/dist'
    );
});
```

如果你想要使用 Webpack 的更多功能，可以利用应用根目录下的 `webpack.config.js` 文件，Elixir 将会读取该文件并将其中的 [配置](https://webpack.github.io/docs/configuration.html) 用于构建过程。


<a name="rollup"></a>
### Rollup

和 Webpack 相似，Rollup 是为 ES2015 准备的下一代模块管理器，该方法接收一个相对于 `resources/assets/js` 目录的文件数组，然后在 `public/js` 目录下生成单个文件：

```javascript
elixir(function(mix) {
    mix.rollup('app.js');
});
```

和 `webpack` 方法一样，你也可以自定义传递给 `rollup` 方法的输入输出文件路径：

    elixir(function(mix) {
        mix.rollup(
            './resources/assets/js/app.js',
            './public/dist'
        );
    });

<a name="javascript"></a>
### 脚本

如果你有多个 JavaScript 文件想要编译成单个文件，可以使用 `scripts` 方法。

`scripts` 方法假定所有路径相对于 `resources/assets/js` 目录，而且所有结果 JavaScript 默认存放在 `public/js/all.js`：

```javascript
elixir(function(mix) {
    mix.scripts([
        'order.js',
        'forum.js'
    ]);
});
```

如果你需要将多个脚本集合合并到不同的文件，需要多次调用 `scripts` 方法。该方法的第二个参数决定每个合并的结果文件名：

```javascript
elixir(function(mix) {
    mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

如果你需要将多个脚本合并到给定目录，可以使用 `scriptsIn` 方法。结果 JavaScript 会被存放到 `public/js/all.js`：

```javascript
elixir(function(mix) {
    mix.scriptsIn('public/js/some/directory');
});
```

> {tip} 如果你要合并多个已经最小化的 vendor 库文件，例如 jQuery，可以考虑使用 `mix.combine()` ，这将会合并这些文件，略过源地图和最小化步骤。最终，编译消耗时间将会大幅减小。


<a name="copying-files-and-directories"></a>
## 拷贝文件 & 目录

你可以使用 `copy` 方法拷贝文件/目录到新路径，所有操作都相对于项目根目录：

```javascript
elixir(function(mix) {
	mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
});
```

<a name="versioning-and-cache-busting"></a>
## 版本号 / 缓存刷新

很多开发者会给编译的前端资源添加时间戳或者唯一令牌后缀以强制浏览器加载最新版本而不是代码的缓存副本。Elixir 可以使用 `version` 方法为你处理这种情况。

`version` 方法接收相对于 `public` 目录的文件名，附加唯一hash到文件名，从而实现缓存刷新。例如，生成的文件名看上去是这样——`all-16d570a7.css`：

```javascript
elixir(function(mix) {
    mix.version('css/all.css');
});
```

生成版本文件后，可以在 [视图](/laravel/{{version}}/views) 中使用 Elixir 的全局帮助函数 `elixir` 方法来加载相应的带哈希值的前端资源，`elixir` 函数会自动判断哈希文件名：

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

#### 给多个文件加上版本号

你可以传递一个数组到 `version` 方法来为多个文件添加版本号：

```javascript
elixir(function(mix) {
    mix.version(['css/all.css', 'js/app.js']);
});
```

文件被加上版本号后，就可以使用辅助函数 `elixir` 来生成指向该哈希文件的链接。记住，你只需要传递没有哈希值的文件名到 `elixir` 方法。该帮助函数使用未加希值值的文件名来判断文件当前的希值版本：

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

    <script src="{{ elixir('js/app.js') }}"></script>

<a name="browser-sync"></a>
## BrowserSync

BrowserSync 会在你修改前端资源后自动刷新浏览器， `browserSync` 方法接收一个 JavaScript 对象，该对象有一个包含本地应用URL的属性—— `proxy`。当你运行 `gulp watch` 命令时，可以通过3000端口（ `http://project.dev:3000` ）享受浏览器同步：

```javascript
elixir(function(mix) {
    mix.browserSync({
        proxy: 'project.dev'
    });
});
```
