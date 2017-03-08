
*此文来自微软web开发系列。*

JavaScript自从诞生之日起，经历了漫长的演变过程。今天，这门语言被我们广泛使用，主要得感谢[TC39][1]对于JavaScript（即**ECMAScript**）标准化所付出的努力。

**ECMAScript**在发展过程中得到的巨大提升之一就是关于**异步处理**。如果你是这方面的新手，可以从[asynchronous programming here][2]学到不少。幸运的是，在Windows 10的新一代Edge浏览器中，我们已经包含了这些特性，具体变化你可以查看[Microsoft Edge change log][3]。

##第一步: ECMAScript 5 —— Callback
**ECMAScript 5**以及之前的版本，都采用了callback的方式。为了更好地说明这一点，我们煮个简单的栗子，比如你基本上天天都会用到的：发起一个XHR请求。
```
var displayDiv = document.getElementById("displayDiv");

// Part 1 - Defining what do we want to do with the result
var processJSON = function (json) {
var result = JSON.parse(json);

    result.collection.forEach(function(card) {
var div = document.createElement("div");
        div.innerHTML = card.name + " cost is " + card.price;

        displayDiv.appendChild(div);
    });
}

// Part 2 - Providing a function to display errors
var displayError = function(error) {
    displayDiv.innerHTML = error;
}

// Part 3 - Creating and setting up the XHR object
var xhr = new XMLHttpRequest();

xhr.open('GET', "cards.json");

// Part 4 - Defining callbacks that XHR object will call for us
xhr.onload = function(){
if (xhr.status === 200) {
        processJSON(xhr.response);
    }
}

xhr.onerror = function() {
    displayError("Unable to load RSS");
}

// Part 5 - Starting the process
xhr.send();
```

有经验的JavaScript开发者看到这些算是再熟悉不过了：首先创建一个XHR请求对象，再给其相应的属性绑定callback函数。

相比之下，callback方式复杂的成因，主要来源于由于异步代码与生俱来的非线性执行顺序：
![图片url][4]
并且，最可怕的莫过于[“回调地狱”(callbacks hell)][5]，即在一个callback中发起另一个callback。

##第二步: ECMAScript 6 —— Promise
**ECMAScript 6**势头正旺，Edge目前已经覆盖到其特性的88%。[特性详情请戳我][6]

在众多提升中，**ECMAScript 6**对*Promise*的使用进行了标准化（之前被称作future）。
根据[MDN][7]所述，*promise*是一个用于延迟和异步进行计算的对象。一个*promise*代表了一个尚未完成，但是被期望在未来某个时间完成的操作。异步操作经过Promise的组织后，可以表现为同步操作的形式。Promise已经存在了一段时间，好消息是，你现在已经无需用到任何的库，因为浏览器已经开始支持它。

我们来使用Promise更新一下之前的例子，看看它是如何提高代码的可读性和可维护性。

```
var displayDiv = document.getElementById("displayDiv");

// Part 1 - Create a function that returns a promise
function getJsonAsync(url) {
// Promises require two functions: one for success, one for failure
return new Promise(function (resolve, reject) {
var xhr = new XMLHttpRequest();

        xhr.open('GET', url);

        xhr.onload = () => {
if (xhr.status === 200) {
// We can resolve the promise
resolve(xhr.response);
            } else {
// It's a failure, so let's reject the promise
reject("Unable to load RSS");
            }
        }

        xhr.onerror = () => {
// It's a failure, so let's reject the promise
reject("Unable to load RSS");
        };

        xhr.send();
    });
}

// Part 2 - The function returns a promise
// so we can chain with a .then and a .catch
getJsonAsync("cards.json").then(json => {
var result = JSON.parse(json);

    result.collection.forEach(card => {
var div = document.createElement("div");
        div.innerHTML = `${card.name} cost is ${card.price}`;

        displayDiv.appendChild(div);
    });
}).catch(error => {
    displayDiv.innerHTML = error;
});
```
你可以在这段代码中看到诸多提升，我们来细细分析下。

###建立Promise
首先，你要建立一个*promise*对象，去“包装”古老的XHR。
![此处输入图片的描述][8]

###使用promise
一旦建立之后，可以采用优雅地对异步操作进行链式调用：
![此处输入图片的描述][9]

现在，从我们的角度来看，做了如下事情。

 - 获得Promise（1）
 - 链接到处理成功时需要执行的代码（2和3）
 - 用try/catch链接到处理错误时需要执行的代码（4）

只需要傻傻地调用.then().then()就可以将多个promise“串连”起来，是不是很有趣？

***tips：***由于JavaScript是一门现代语言，你可能已经注意到我使用了***ECMAScript 6***中的一些[语法糖（syntax sugar）][10]，例如[模板字符串（template strings）][11]和[箭头函数（arrow functions）][12]。

##终点: ECMAScript 7 —— 无处不在的异步
最后，我们终于达到了和异步“融为一体”的目的！得益于Edge浏览器的快速迭代，开发团队已经在最新版本中引进了**ECMAScript 7**中的**异步函数（async function）**！

异步函数的实现基于ECMAScript 6中的特性，例如生成器（generator）。generator确实能和promise协作，从而产生同样的效果，可是却需要写更多的代码。有了异步函数，我们只需要将之前的例子改为：
```
// Let's create an async anonymous function
(async function() {
try {
// Just have to await the promise!
var json = await getJsonAsync("cards.json");
var result = JSON.parse(json);

        result.collection.forEach(card => {
var div = document.createElement("div");
            div.innerHTML = `${card.name} cost is ${card.price}`;

            displayDiv.appendChild(div);
        });
    } catch (e) {
        displayDiv.innerHTML = e;
    }
})();
```
这串代码的神奇之处就在于，它的执行逻辑看起来就是同步的，有着完美的线性执行顺序。
![此处输入图片的描述][13]

最后，读到这里，难道你还没有被深深地震撼到？不但如此，异步函数也可以用于箭头函数以及类方法。



  [1]: http://www.ecma-international.org/memento/TC39-M.htm
  [2]: https://msdn.microsoft.com/en-us/library/hh191443.aspx?WT.mc_id=16521-DEV-sitepoint-article57
  [3]: https://dev.modern.ie/platform/changelog/desktop/10547/?compareWith=10532/?utm_source=SitePoint&utm_medium=article57&utm_campaign=SitePoint
  [4]: http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/10/1445201763asynchronous-javascript01-es5-xhr-request-callback.png
  [5]: http://callbackhell.com/
  [6]: http://kangax.github.io/compat-table/es6/
  [7]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
  [8]: http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/10/1445201770asynchronous-javascript02-es6-creating-promise-object.png
  [9]: http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/10/1445201777asynchronous-javascript03-using-promise-to-chain-asynchronous-calls.png
  [10]: https://en.wikipedia.org/wiki/Syntactic_sugar
  [11]: http://tc39wiki.calculist.org/es6/template-strings/
  [12]: http://tc39wiki.calculist.org/es6/arrow-functions/
  [13]: http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/10/1445201783asynchronous-javascript04-creating-async-function.png



