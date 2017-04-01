---
layout: post
title: 上手 Webpack ? 这篇就够了！
categories: [web前端]
tags: [前端, Webpack]
description: JavaSript 模块化打包已混迹江湖许久。2009年，RequireJS 就提交了它的第一个版本，Browserify 接踵而至，随后其他打包工具也开始大行其道。最终，Webpack 从其中脱颖而出。如果你对它不甚了解，希望我的文章能让你上手这件强力打包工具。
---

>   作者：小boy （沪江前端开发工程师）    
>   本文原创翻译，转载请注明作者及出处。    
>   原文地址：https://www.smashingmagazine.com/2017/02/a-detailed-introduction-to-webpack
> 


JavaSript 模块化打包已混迹江湖许久。2009年，RequireJS 就提交了它的第一个版本，Browserify 接踵而至，随后其他打包工具也开始大行其道。最终，Webpack 从其中脱颖而出。如果你对它不甚了解，希望我的文章能让你上手这件强力打包工具。

## 什么是模块化打包工具？

在大多数语言（JS 的最新版本 ECMAScript 2015+ 也支持，但并非支持所有浏览器）中，你可以将代码拆分至多个文件，并且通过在业务代码中引用这些文件来使用它们包含的方法。可惜的是浏览器并不拥有这个能力。因此，模块化打包工具应运而生，它以两种形式为浏览器提供这个能力：1.异步加载模块，并且在加载结束后运行它们。2.将需要用到的文件拼凑成单一 JS 文件，最终在 HTML 中使用 `<script>` 标签加载该 JS 文件。

如果没有模块化加载及打包工具的话，你就得手动拼凑文件或者在 HTML 中加载无数的 `<script>` 标签了，而且这样干有一些不好的地方：

 - 你需要关心文件的加载顺序，包括哪些文件依赖于其他文件；还需要确认是否引入了不需要的文件。
 - 多个 `<script>` 标签意味着用多个网络请求来加载代码，同时也意味着更差的性能。
 - 显然，其中有很多本可以交给计算机完成的手工活。

大多数模块化打包工具直接跟 npm 或者 Bower（译者注：两者都是包管理工具）整合，这样可以让你更容易地在业务代码中添加第三方依赖包（dependencies）。你仅需要安装一下，然后写一行代码引入它们，接着运行模块化打包工具，这样就已经将第三方代码整合进自己的业务代码了。或者，如若配置正确，你可以将所有要用的三方代码整合进一个分开的文件，这样一来，当你更新业务代码，用户需要更新缓存的时候，他们就无需重新下载公共库代码（vendor code）了。

## 为什么选择 Webpack ?

至此，你已对 Webpack 的愿景有了基础的认知，然而为什么在各路豪杰中选择 Webpack 呢？在我看来有这样一些理由：

- 小鲜肉的特性助了它一臂之力，借此特点它可以绕开或者避免前辈们遇到的问题。
- 上手简单。如果只是想打包一些JS文件，而没有其他需求的话，你甚至都不需要一份配置文件。
- 它的插件体系使其能做更多事情，从而十分强大，所以它可能是你需要的唯一构建工具。

据我所知，少有其他的模块打包和构建工具也能做到这些。但 Webpack 仍胜一筹：当你踩坑的时候有庞大的社区支持。
Browserify 的社区可能只是大，如果它不大的话，就会缺少一些 Webpack 的潜在必要特性。说了这么多 Webpack 的优点，估计你就等上代码了吧？那么我们开始。

## 安装 Webpack

在使用 Webpack 前，我们首先要先把它安装好。为此我们需要 Node.js 和 npm ，我就假设你已经安装过它们了，实在没有的话，请从[Node.js 官网](https://nodejs.org/en/)开始吧。

有两种方式安装 webpack （或着是其他 CLI 包）：全局安装（globally）或者本地安装（locally）。对于全局安装，虽然你可以在任意目录下使用它，但是它不会包括在项目的依赖模块列表（dependencies）中。此外，你也不能在两个不同的项目（有些项目可能需要投入更多工作量才能更新到最新版本，所以这些项目还需要维持老版本）中切换不同版本的 Webpack 。所以我更愿意本地安装 CLI 包，并且用相对路径抑或是 npm 脚本来运行它。如果你不习惯本地安装 CLI 包，可以看一下我之前写的关于[摆脱全局安装 npm 包](https://www.joezimjs.com/javascript/no-more-global-npm-packages/)的博文。

不管怎样，在示例项目中，我们就使用 npm 脚本。接下来，先本地安装示例项目。首先：创建一个用来实验和学习 Webpack 的目录。 我在 GitHub 上有一个[仓库](https://github.com/joezimjs/webpack-Introduction-Tutorial)，你可以将它 clone 到本地，然后在分支间切换来进行下面的学习，或者从零开始创建一个新项目，此后可以与我的仓库代码进行对照。

经过命令行选择，一进到项目目录，你将用 `npm init` 命令来初始化项目。接下来要填的信息一点都不重要（译者注：一路回车即可），除非你想把项目发布到 npm 上。

至此 package.json 文件准备就绪（它是通过 `npm init` 命令创建的），在此文件中，你可以保存依赖包信息。我们通过 `npm install webpack -D` （`-D` 是 `--save-dev` 命令的简写，它的作用是将 npm 包作为开发环境的依赖包安装，并将依赖信息保存到 `package.json` 文件中）命令将 Webpack 作为依赖包安装。

我们需要一个简单的应用来开启运用 Webpack 之旅。所谓的简单就是：首先执行 `npm install lodash -S`（`-S` == `--save`） 安装 [Lodash](http://www.lodash.com/)，如此一来我们的简单应用就有一个依赖包可以用来加载了。接着我们创建一个 `src` 目录，再于该目录中创建名为 `main.js` 的文件，其内容如下：

```js
var map = require('lodash/map');

function square(n) {
    return n*n;
}

console.log(map([1,2,3,4,5,6], square));
```
很简单对吧？我们仅仅创建了一个包含整数1至6的小数组，然后用 Loadash 库中的 `map` 函数创建了一个新数组，这个新数组中的数字是原数组中数字的平方。最后，我们在控制台中打印这个新数组。运行命令 `node src/main.js` 就能看到结果：`[1, 4, 9, 16, 25, 36]`。你瞧，其实 Node.js 都能运行这个文件。

但如果我们想打包这个小脚本，其中还包括我们能跑在浏览器的 Lodash 代码，使用 Webpack 应该从哪入手？如何做到？

## Webpack 命令行

若不想在配置文件上浪费时间，使用 Webpack 命令行是最容易的上手方式。如果不启用配置文件的话，最简洁的命令需要包含输入文件（input file）路径和输出文件（output file）路径。Webpack 会读取输入文件，追踪它的依赖关系树，并将所有依赖文件打包进一个文件，最终在你指定的输出路径下输出该文件。在本例中，输入路径是 `src/main.js` ，我们要将打包后的文件输出到 `dist/bundle.js` 下。为此，我们先添加 npm 脚本（我们并没有全局安装 Webpack ，所以不能直接在命令行中运行）。编辑 `package.json` 文件的 `"scripts"` 部分如下：

``` json
"scripts": {
    "build": "webpack src/main.js dist/bundle.js",
}
```

现在，执行 `npm run build` 命令，Webpack 就会运行了。很快，运行完毕的时候会生成 `dist/bundle.js` 文件。然后你便可以用 Node.js （通过 `node dist/bundle.js` 命令）运行该文件了。也可以借助简单的 HTML 将其跑在浏览器上，之后可在控制台中看到同样的运行结果。

在继续探索 Webpack 前，我们先把构建脚本调整得更专业一点：在重新构建（rebuilding）前删除 `dist` 目录及其内容，此外，我们再添加一些用于直接执行 bundle 文件的脚本。首先，安装 `del-cli` 工具，这样就不用在删除目录的时候顾虑操作系统的区别了（见谅，因为我用的是 Windows）。运行 `npm install del-cli -D` 命令即可。接着更新 npm 脚本如下：

```json
"scripts": {
    "prebuild": "del-cli dist -f",
    "build": "webpack src/main.js dist/bundle.js",
    "execute": "node dist/bundle.js",
    "start": "npm run build -s && npm run execute -s"
  }
```

我们保持 `"build"` 配置同之前一样，但增加了 `"prebuild"` 配置用以清除目录，这条配置所执行的命令会在每次 `"build"` 命令执行之前运行。同时增加的还有 `"execute"` 配置：使用 Node.js 执行已经打包好的脚本。此外，使用 `"start"` 配置可以通过一条命令执行以上所有命令（`-s` 的作用仅仅是不让 npm 脚本在控制台打印一些没用的东西）。执行 `npm start` 命令，就可以在控制台里看到 Webpack 的输出信息，紧接着打印的是平方后的数组。

恭喜！你刚刚完成了 `example1` 分支里所有的事情。这个分支就在我之前提到的仓库中。

## Webpack 配置文件

跟使用 Webpack 命令行上手一样有趣的是，一旦开始使用更多 Webpack 的功能， 你就会想要放弃通过命令行传递 Webpack 配置参数，转而投入配置文件的怀抱。使用配置文件虽然会更占位置，但与此同时增加了可读性，因为它是由 JS 写成的。

那我们就来创建配置文件吧。在根目录下创建一个新文件 `webpack.config.js`。Webpack 默认寻找该文件，但如果想给配置文件取别的名字或者将配置文件放在其他目录，你可以通过传递 `--config [filename]` 参数来做到。

在本教程中，我们使用默认文件名。现在，我们试着让配置文件起作用，达到与仅使用命令行同样的效果。为此，我们需要在配置文件中添置如下代码：

```js
module.exports = {
    entry: './src/main.js',
    output: {
        path: './dist',
        filename: 'bundle.js'
    }
};
```

如此前一样，我们规定输入和输出文件。因为这不是 JSON 文件而是 JS 文件，所以我们需要把配置对象（configuration object ）导出，故使用 `module.exports`。虽然现在还看不出写这些配置会比用命令好多少，但文章结尾你肯定会爱上这里的一切。

接下来，移除 `package.json` 文件中给 Webpack 传的配置，像这样：

```json
"scripts": {
    "prebuild": "del-cli dist -f",
    "build": "webpack",
    "execute": "node dist/bundle.js",
    "start": "npm run build -s && npm run execute -s"
}
```
像之前一样执行 `npm start` 命令，运行结果是不是似曾相识呢？以上就是分支 `example2` 中需要做的事情。

## Webpack 加载器（Loaders）

我们主要通过两种方式增强 Webpack: 加载器（loaders）和插件（plugins）。我们先讲加载器，插件稍后再议。**加载器用以转换或操作特定类型的文件**，你可以将多个加载器串联在一起来处理一种类型的文件。例如，规定 `.js` 后缀的文件要先通过 [ESLint](http://eslint.org/) 检查，再通过 [Babel](https://babeljs.io/) 把 ES2015 语法转换为 ES5 语法。ESLint 发出的警报将会在控制台打印出来，而遇到语法错误的时候则会阻止 Webpack 继续打包。

我们这里就不设置语法检查了，但要通过设置 Babel 来把代码转化成 ES5。当然我们得先有些 ES2015 代码吧？把 `main.js` 文件的代码改成下面的样子：

```js
import { map } from 'lodash';

console.log(map([1,2,3,4,5,6], n => n*n));

```

实际上这段代码和之前做的事情一样，但有两点：其一，使用箭头函数替代了之前定义的 `square` 函数。其二，使用了 ES2015 中的 `import` 语法加载 `lodash` 库中的 `map` 函数，但这将会把整个 Lodash 库的代码打包到我们的输出文件中，而不是引入仅仅包含 `map` 函数相关代码的 `'lodash/map'` 库。如果乐意的话，你也可以把第一行改成 `import map from 'lodash/map';` 但我写成这样有我的理由：

- 在更具规模的应用里，你可能要用到 Lodash 库的很多部分，所以你最好全加载进来。
- 如果你正在用 Backbone.js 框架，会发现仅打包你需要的函数是很困难的，因为根本就没有文档告诉你函数依赖哪些函数。
- 在 Webpack 的下一个大版本中，开发者打算加入一个叫 tree-shaking 的东西，tree-shaking 会排除掉引入模块中没有用到的部分。所以那也是一种办法。
- 这样写是为了举一个例子，好让你理解我之前提到的要点。

（注：Lodash 这两种加载方式都可以用，因为它的开发者明确规定可以这么做，而不是所有的库都可以通过这种加载方式工作。）

无论如何，ES2015 代码现已在手，我们要把它转化成 ES5 代码，这样它们就能在老式浏览器（事实上，在新版浏览器里 ES2015 的支持度还不错）里跑起来了。因此，我们需要 Babel 及在 Webpack 中运行 Babel 的配套设施。至少要有 [babel-core](https://www.npmjs.com/package/babel-core)（Babel 的核心功能库），[babel-loader](https://www.npmjs.com/package/babel-loader)（babel-core 的 Webpack 加载器接口），[babel-preset-es2015](https://www.npmjs.com/package/babel-preset-es2015)（里面有 ES2015 到 ES5 的转化规则，这是 Babel 需要得知的）。同时我们引进 [babel-plugin-transform-runtime](https://www.npmjs.com/package/babel-plugin-transform-runtime) 和 [babel-polyfill](https://www.npmjs.com/package/babel-polyfill) ，尽管它们实现方式有点不同，但都用于改变 Babel 添加语法填充（polyfills）和辅助函数（helper functions）的方式。正是因此，它们适应于不同种类的项目。你可能不想把它们俩都引入，二者择一即可，但我在这把它俩都引入，这样无论你选择哪个，都能知道引入的方式。想知道更多的话，请访问 polyfill 和 runtime transform 的官方文档吧。

不管怎样，先安装它们：`npm i -D babel-core babel-loader babel-preset-es2015 babel-plugin-transform-runtime babel-polyfill`。再为它们配置 Webpack。首先，添加一个部分用于增添加载器。更新 `webpack.config.js` 如下：

```js
module.exports = {
    entry: './src/main.js',
    output: {
        path: './dist',
        filename: 'bundle.js'
    },
    module: {
        rules: [
            …
        ]
    }
};
```

我们增加了一个 `module` 属性，其中包含了 `rules` 属性。`rules` 是一个数组，这个数组囊括每个加载器的配置。我们将把 babel-loader 相关配置加到这里。对于每一个加载器，我们都要配置至少两个参数：`test` 和 `loader`。`test` 通常是一个正则表达式，它用以验证（test）每个文件的绝对路径。我们一般只验证文件后缀，例如：`/\.js$/`验证所有以 `.js` 结尾的文件。在这里，我们把这个参数设为 `/\.jsx?$/` 这样可以匹配到 `.js` 文件和 `.jsx` 文件，以便使用 `React`。接下来配置 `loader` 参数，它描述了在相应的 `test` 参数下，应该使用哪一个加载器处理文件。

将加载器的名字所拼成的字符串传入该参数即可奏效，其中，名字用感叹号隔开，例如 `'babel-loader!eslint-loader'`。`eslint-loader` 会比 `babel-loader` 先运行，因为 Webpack 的读取顺序是从右到左。如果某个加载器有特殊参数配置，你可以使用 query string 语法。比如，要给 Babel 配置一个 `fakeoption` 参数为 `true`，我们得把前面的例子改为 `'babel-loader?fakeoption=true!eslint-loader'`。如果你觉得更易阅读和维护的话，也可以使用 `use` 替代 `loader` 配置，这样可以传入一个数组替代此前的字符串。把之前的例子改为：`use: ['babel-loader?fakeoption=true', 'eslint-loader']`，更有甚者，你可以把它们写成多行以提高可读性。

目前我们只用 Babel loader ，所以我们的配置文件看起来像下面这个样子：

```js
…
rules: [
    { test: /\.jsx?$/, loader: 'babel-loader' }
]
…
```

如果只用一个加载器，我们还可以这样配置来替代 query string 的写法：使用 `options` 配置对象，它就是一个键值对 map。因此，对于 `fakeoption` 的例子，我们的配置文件可以写成这样：

```js
…
rules: [
    {
        test: /\.jsx?$/,
        loader: 'babel-loader',
        options: {
            fakeoption: true
        }
    }
]
…
```

用上面这种方式来配置我们的 Babel 加载器：

```
…
rules: [
    {
        test: /\.jsx?$/,
        loader: 'babel-loader',
        options: {
            plugins: ['transform-runtime'],
            presets: ['es2015']
        }
    }
]
…
```

预设（presets）用于把 ES2015 特性转成 ES5，我们也给 Babel 设置了已经安装的 transform-runtime 插件。如此前所言，该插件并非必要，这里是为了演示。我们也可以另建 `.babelrc` 文件独立配置这些参数，但那样不利于演示 Webpack。一般我推荐使用 `.babelrc` 文件，但在这里我们还是保持不变。

万事俱备，只欠东风。我们需要告知 Babel 跳过处理 `node_modules` 中的文件，这样可以提高我们的构建速度。添置 `exclude` 属性以告知加载器忽略目标目录下的文件，它的值是一个正则表达式，因此我们这样写：`/node_modules/`。

```
…
rules: [
    {
        test: /\.jsx?$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
        options: {
            plugins: ['transform-runtime'],
            presets: ['es2015']
        }
    }
]
…
```

此外，我们本应使用 `include` 属性来描述我们仅读 `src` 目录，但我觉得应该保持原样。于是，你应该可以再次执行 `npm start` 命令，然后获取为浏览器准备的 ES5 代码了。若想使用 polyfill 替代 transform-runtime 插件，你需要做一两处改动。首先删除 `plugins: ['transform-runtime],` 这行（如果不打算再用了，你也可以直接用 npm 卸载该插件）。接下来，编辑 Webpack 配置文件的 `entry` 部分如下：

```js
entry: [
    'babel-polyfill',
    './src/main.js'
],
```

我们把描述单一入口的字符串替换成了描述多入口的数组，新添的入口乃 语法填充（polyfill）。我们将其置于首位，这样语法填充将会率先出现在打包后的文件里，因为我们在代码里使用语法填充前，要确保它们已经存在。

除了借助 Webpack 配置文件，我们本可以通过在 `src/main.js` 的首行加上 `import 'babel-polyfill;` 来达到相同的目的。而我们却使用了配置文件，除了用于服务本例，更是为了用作一个演示多入口打包至单一文件的范例。好吧，那便是仓库里的 `example3` 分支。容我再说一遍，你可以运行 `npm start` 命令来确认项目正常运行。

## 另一个例子：Handlebars 加载器

我们再为项目添置一个加载器：[Handlebars](http://handlebarsjs.com/)。Handlebars 加载器用以将 Handlebars 模版编译成函数，当你在 JS 中引入（import）一个 Handlebars 文件时，该文件编译成的函数就会被引入 JS 文件。这便是我喜欢 Webpack 加载器的地方：即便引入非 JS 文件，该文件也会在打包时被转化为 JS 里可用的东西。接下来的例子将会使用另一个加载器：允许引入图片文件并将图片文件转化成 base64 编码的 URL 字符串，该字符串可被用于在 JS 中为页面添加內联图片。这也意味着，如果你串联多个加载器，其中一个甚至能优化把图片的文件大小。

同样，我们首先安装这个加载器：执行 `npm install -D handlebars-loader` 命令。当你用的时候会发现 Handlebars 本身也是不可或缺的：执行 `npm install -D handlebars` 命令。这样你就可以在不更新加载器版本的情况下控制 Handlebars 的版本，它们可以分别独立迭代。

二者现已安装完毕，我们弄一个 Handlebars 模板来用。在 `src` 目录下创建一个 `numberlist.hbs` 文件，其内容如下：

```html
<ul>
  {{#each numbers as |number i|}}
    <li>{{number}}</li>
  {{/each}}
</ul>
```

该模板描绘了一个数组（变量名为 numbers ，也可以是别的变量名），创建了一个无序列表。

接下来，我们调整此前的 JS 文件来使用模板输出一个列表，不再止步于打印数组本身。`main.js` 看起来会像下面一样：

```js
import { map } from 'lodash';
import template from './numberlist.hbs';

let numbers = map([1,2,3,4,5,6], n => n*n);

console.log(template({numbers}));
```

可惜目前为止 Webpack 并不知道如何引入 `numberlist.hbs` ，因为它并非 JS 文件。我们可以在 `import` 的路径前加点东西通知 Webpack 要使用 Handlebars 加载器：

```js
import { map } from 'lodash';
import template from 'handlebars-loader!./numberlist.hbs';

let numbers = map([1,2,3,4,5,6], n => n*n);

console.log(template({numbers}));
``` 

通过给路径增添加载器名字，并将名字和路径以感叹号隔开的前缀，我们告知 Webpack 那个文件应该使用那个加载器。这样，我们不必在配置文件里添置任何东西。然而，在颇有规模的项目里，你极有可能加载不止一个模板，所以，在配置文件里告知 Webpack 我们使用 Handlebars ，以免去引入模板时在路径前添加前缀，这样做会更有意义。那我们就更新一下配置文件：

```js
…
rules: [
    {/* babel loader config… */},
    { test: /\.hbs$/, loader: 'handlebars-loader' }
]
…
```

这部分相当简单。我们所需要做的就是指定用 `handlebars-loader` 去处理以 `.hbs` 结尾的文件，仅此而已。我们搞定了 Handlebars 同时也搞定了 `example4` 分支。现在，一旦运行 `npm start` ，你会看到 Webpack 打包输出如下内容：

```js
<ul>
<li>1</li>
<li>4</li>
<li>9</li>
<li>16</li>
<li>25</li>
<li>36</li>
</ul>
```

## Webpack 插件

插件是另一种用来自定义 Webpack 功能的方式。你可以更自由地把它们添加到 Webpack 工作流（workflow）中，因为，除加载特殊文件类型之外，它们几乎不受限制。它们可被植入到任何地方，正因如此，他们更加强劲。我很难定义 Webpack 插件到底能做多少事情，因此我仅给出一个 npm 上的搜索结果列表 [npm packages that have “webpack-plugin”](https://www.npmjs.com/search?q=webpack-plugin)，那应该不失为一个好的答案。

本教程中我们只接触两个插件（其中一个马上揭晓）。行文已至此你也知道我的风格，过多的例子我们就不需要了。我们首先上 [HTML Webpack Plugin](https://github.com/ampedandwired/html-webpack-plugin) ，它的作用很纯粹：生成 HTML 文件 —— 终于可以开始进军浏览器了！

在使用该插件之前，我们首先更新 npm 脚本来运行一个能够测试示例应用的简单服务器。先安装一个服务器：运行 `npm i -D http-server` 命令。接着，仿照下面的代码将此前的 `execute` 脚本改成 `server` 脚本。

```js
…
"scripts": {
  "prebuild": "del-cli dist -f",
  "build": "webpack",
  "server": "http-server ./dist",
  "start": "npm run build -s && npm run server -s"
},
…
```

Webpack 完成构建后，`npm start` 会同时启动一个 web 服务器，将浏览器跳转到 `localhost:8080` 可以访问到你的页面。自然，我们仍然需要靠插件来创建该页面，所以接下来，我们需要安装插件：`npm i -D html-webpack-plugin` 。

安装完毕以后，我们移步 `webpack.config.js` 并作如下修改：

```js
var HtmlwebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: [
        'babel-polyfill',
        './src/main.js'
    ],
    output: {
        path: './dist',
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.jsx?$/, loader: 'babel-loader', exclude: /node_modules/,
                options: { plugins: ['transform-runtime'], presets: ['es2015'] }
            },
            { test: /\.hbs$/, loader: 'handlebars-loader' }
        ]
    },
    plugins: [
        new HtmlwebpackPlugin()
    ]
};
```

我们有作两处改动：其一在文件顶部引入新安装的插件，其二在配置对象尾部添置了一个 `plugins` 部分，并在此处传入了插件的实例对象。

目前我们并没有为该插件实例传入配置对象，默认使用它的基础模板，除了我们打包好的脚本文件以外，该基础模版并没有包含很多东西。在运行 `npm start` 后在浏览器访问相应 URL ，你会看到一空白页，但若在开发者工具中打开控制台，应该会看到里面打印出了 HTML。

我们可能要获得模板并将 HTML 吐（spit out）到页面上而不是控制台里，这样一个“正常人”就能真正从页面上得到信息了。我们先在 `src` 目录下创建 `index.html`  文件，这样就能定义自己的模板了。默认情况下，该插件用的是 EJS 模板语法，不过，你也可以配置该插件使其使用其它[受到支持的模板语言](https://github.com/jantimon/html-webpack-plugin/blob/master/docs/template-option.md)。在这里我们就用 EJS 因为用什么语法都没有实质区别，`index.html` 的内容如下：

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
    <h2>This is my Index.html Template</h2>
    <div id="app-container"></div>
</body>
</html>
```

请注意几点：

- 我们将为插件传入一个配置对象来定义标题（仅仅因为我们能做到）。
- 没有具体指定该在哪里插入我们的脚本文件，因为该插件默认会在 `body` 元素结尾前添加脚本。
- 这里 div 的 id 并非特定，我们在这里随便取了一个。

现在我们得到了想要的模板，最终不会只是一个空白页了。接下来更新 `main.js` ，把 HTML 结构加入那个 `div` 里以替代此前打印在控制台里。为此，我们仅需更新 `main.js` 的最后一行：`document.getElementById("app-container").innerHTML = template({numbers});`

同时，我们也需要更新 Webpack 配置文件，为插件传入两个参数。配置文件现在应改成这样：
```js
var HtmlwebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: [
        'babel-polyfill',
        './src/main.js'
    ],
    output: {
        path: './dist',
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.jsx?$/, loader: 'babel-loader', exclude: /node_modules/,
                options: { plugins: ['transform-runtime'], presets: ['es2015'] }
            },
            { test: /\.hbs$/, loader: 'handlebars-loader' }
        ]
    },
    plugins: [
        new HtmlwebpackPlugin({
            title: 'Intro to webpack',
            template: 'src/index.html'
        })
    ]
};

```

`template` 配置指定了模板文件的位置，`title` 配置被传入了模板。现在，运行 `npm start`，你将会在浏览器里看到下面的内容：

![template](./images/index-html-template-opt.png)

假如你一直跟着做的话，`example5` 分支便在此结束。不同插件传入的参数或者配置项也大异其趣，其原因在于插件种类繁多且涵盖范围广阔，但殊途同归的是，他们最终都会被添加到 `webpack.config.js` 的 `plugins` 数组中。同样，也有其他方式可以处理 HTML 页面的生成和文件名填充，一旦你开始为打包后的文件添加清缓存哈希值（cache-busting hashes）后缀，这些事情就会变得非常简单。

观察示例仓库，你会发现有一个 `example6` 分支，在该分支里我通过添加插件实现了 JS 代码压缩，但这不是必须的，除非你想改动 UglifyJS 配置。如果你不爽 UglifyJS 的默认配置，可将仓库切换 (check out)至该分支下（只需要查看 `webpack.config.js` ）去找到如何使用该插件并加以配置。但如果默认配置正合你意，你只需要在命令行运行 `webpack` 时传入 `-p` 参数。该参数是 `production` 的简写，与使用 `--optimize-minimize` 和 `--optimize-occurence-order` 参数的效果一样，前者用以压缩 JS 代码，后者用以优化已引入模块的顺序，着眼于稍小的文件尺寸和稍快的执行速度。在示例仓库完成一段时间后我才知道 `-p` 这个参数，所以我决定保存该插件示例，可以用来提醒你还有更简单的方法（除了添加插件之外）。另一可供使用的快捷命令参数是 `-d` ，`-d` 会展示更多 Webpack 打印出的信息，并且可不借助其他参数生成资料图（source map）。还有很多其他[命令行快捷参数](http://webpack.github.io/docs/cli.html)可供使用。

## 懒加载数据块

懒加载（lazy-loading）模块是我在 RequireJS 中用得舒适但在 Browserify 中难以工作的模块。一个颇具规模的 JS 文件固然可以从减少网络请求中受益，但也几乎坐实了在一次会话中，某些用户不必用到的代码会被下载下来。

Webpack 可以将打包文件拆分成可被懒加载的若干块（chunks），而且还不需要任何配置。你仅需要从两种书写方式中挑一种来书写代码，剩下的则交给 Webpack。这两种方式其一基于 CommonJS ，其二则基于 AMD。如果使用前者懒加载，需要这样写：

```js
require.ensure(["module-a", "module-b"], function(require) {
    var a = require("module-a");
    var b = require("module-b");
    // …
});
```

`require.ensure` 需要确保模块是可用的（但并非运行模块），然后传入一个由模块名构成的数组，接着传入一个回调函数（callback）。真正想要在回调函数里使用模块，你需要显式 `require` 数组里传入的相应模块。

私以为这种方式相麻烦，所以，我们来看 AMD 的写法。

```js
require(["module-a", "module-b"], function(a, b) {
    // …
});
```

AMD 模式下，使用 `require` 函数，传入包含依赖模块名的数组，接着再传入回调函数。该回调函数的参数就是依赖模块的引用，它们的排列顺序与依赖模块在数组中的排列顺序相同。

Webpack 2 同时也支持 `System.import`，其借助于 promises 而非回调函数。尽管将回调内容包裹在 promise 下并非难事，但我仍以为该提升非常有用。不过需要注意的是， `System.import` 现已过时，较新的规范推荐使用 `import()`。不过，这里告诫一下， Babel （以及 TypeScript）会在你使用`System.import`的时候抛出语法异常。你可以借助于 [babel-plugin-dynamic-import-webpack](https://www.npmjs.com/package/babel-plugin-dynamic-import-webpack) 插件，但该插件将会将其转化为 `require.ensure`，而不是让 Babel 合法处理新 `import` 或者任之由 Webpack 处置。我认为 AMD 或 `require.ensure` 在很久之后才会被弃置，且 Webpack 直到第三个版本才会支持 `System.import` ，那还远着呢，所以用你顺眼的那个就好了。

扩充我们的代码，令其停滞两秒，然后再将 Handlebars 模板懒加载进来并输出到屏幕上。为此，我们移除顶部 `import` 模板的语句，然后将最后一行包裹到 `setTimeout` 和 AMD 模式的 `require` 中引入模板。

运行 `npm start` ，你会发现生成了另外一个名为 `1.bundle.js` 的资源文件（asset）。在浏览器打开该页面，然后在开发者工具中监听网络流量，2秒之后你会发现新的资源文件最终被加载并且运行了。以上这些实现起来并不困难，但提升用户体验可不止一点。

注意，这些二级打包文件（sub-bundles）或曰数据块（chunks），内部囊括了他们的所有依赖模块（dependencies），但不包含其主数据块（parent chunks）已引入的依赖模块。（你可以有多个入口文件，每个都懒加载一个数据块，因此该数据块在其主数据块中加载的依赖模块也会不同。）

## 创建公共库数据块 （Vendor Chunk）

我们再说一个优化的点：公共库数据块。你可以定义一个单独用以打包的 bundle，该 bundle 中存放不常改动的 “common” 库或第三方代码。该策略可使用户独立缓存你的公共库文件，以区别于业务代码，以便在你迭代应用时让用户无需重新下载该库文件。

为此，我们使用 Webpack 官方插件：`CommonsChunkPlugin`。它已附带在 Webpack 中，所以我们无需安装。仅对 `webpack.config.js` 稍作修改即可：

```js
var HtmlwebpackPlugin = require('html-webpack-plugin');
var UglifyJsPlugin = require('webpack/lib/optimize/UglifyJsPlugin');
var CommonsChunkPlugin = require('webpack/lib/optimize/CommonsChunkPlugin');

module.exports = {
    entry: {
        vendor: ['babel-polyfill', 'lodash'],
        main: './src/main.js'
    },
    output: {
        path: './dist',
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.jsx?$/, loader: 'babel-loader', exclude: /node_modules/,
                options: { plugins: ['transform-runtime'], presets: ['es2015'] }
            },
            { test: /\.hbs$/, loader: 'handlebars-loader' }
        ]
    },
    plugins: [
        new HtmlwebpackPlugin({
            title: 'Intro to webpack',
            template: 'src/index.html'
        }),
        new UglifyJsPlugin({
            beautify: false,
            mangle: { screw_ie8 : true },
            compress: { screw_ie8: true, warnings: false },
            comments: false
        }),
        new CommonsChunkPlugin({
            name: "vendor",
            filename: "vendor.bundle.js"
        })
    ]
};
```

我们在第三行引入该插件。此后，在 `entry` 部分修改配置，将其换成了一个对象字面量（literal），用以指定多入口。`vendor` 入口记录了会在公共库数据块中——这里包含了 polyfill 和 Lodash ——被引入的库并将我们的主要入口放置在 `main` 入口里。接着，我们仅需将 `CommonsChunkPlugin` 添加到 `plugins` 部分，指定 “vendor” 数据块作为该插件生成数据块的索引，同时指定 `vendor.bundle.js` 文件用以存放公共库代码（译者注：这里插件配置中的 `name: "vendor"` 对应 `entry` 中的 `vendor` 入口，入口数组中指定的依赖模块即最终存放于 `vendor.bundle.js` 文件中的依赖模块）。

通过指定  “vendor” 数据块，该插件将拉取此数据块所有的依赖模块，并将其存放于公共库数据块內，这些依赖模块在一个单独入口文件里被指定。如果不在入口对象字面量中指定数据块名，插件会基于多入口文件之间公用的依赖模块来生成独立文件。

运行 Webpack ，你将看到3份 JS 文件：`bundle.js`, `1.bundle.js` 和 `vendor.bundle.js`。如果愿意的话也可以运行 `npm start` 命令来在浏览器中查看结果。看起来 Webpack 甚至会把自身加载不同模块的主要代码放进公共库数据块，此举极为实用。

至此我们结束了 `example8` 分支之旅，同时本篇教程也接近尾声。我所谈颇多，但仅让你对 Webpack 的能力浅尝辄止。Webpack 实现了更简便的 CSS module、清缓存、图片优化等等很多事情——多到即便书巨著一本，我也无法说穷道尽，且在我成书之前，大多数已写的内容也将被更新替代。So，尝试一下 Webpack 吧，且告诉我它有没有提升工作流。祝吾主保佑，编程愉快！

