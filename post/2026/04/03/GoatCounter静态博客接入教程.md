tags: 博客 教程 统计

[TOC]

## 为什么推荐 GoatCounter

如果你想给静态博客加一个“文章阅读数”，`GoatCounter` 是很适合的一类服务：

- 它本身就支持静态网站
- 接入只需要一段脚本
- 可以按页面 path 统计访问
- 可以直接通过 `counter/*.json` 这样的接口把数字读出来，显示在文章页或者文章列表里

对个人博客来说，它的优点是足够轻，而且不需要自己先搭后端。

## 第一步：注册并创建站点

先到 GoatCounter 官网创建一个站点。

创建完成之后，你会拿到一段类似下面的脚本：

```html
<script
  data-goatcounter="https://drunkpig.goatcounter.com/count"
  async
  src="//gc.zgo.at/count.js"></script>
```

这里最重要的是：

- `data-goatcounter` 里是你的站点地址
- `count.js` 是 GoatCounter 提供的统计脚本

只要这个脚本被插入到页面里，页面访问就会开始上报。

## 第二步：把统计脚本放进博客配置

如果你是像我这样用 `mdwiki` 生成静态博客，可以把这段脚本放到站点根目录的 `config.json` 里：

```json
{
  "goatcounter_script": "<script data-goatcounter=\"https://drunkpig.goatcounter.com/count\" async src=\"//gc.zgo.at/count.js\"></script>"
}
```

这样做的好处是：

- 统计脚本不写死在模板里
- 换站点时只需要改配置
- 不启用 GoatCounter 时删掉这个字段即可

如果你不是用 `mdwiki`，思路也是一样的：把这段脚本统一插入到每一个页面。

## 第三步：确保脚本真的插入到了每个页面

这一点非常关键。

正确做法是：

- 把 GoatCounter 脚本插入到所有页面
- 最好放在 `<head>` 里，至少要保证在你自己读取阅读数的脚本之前加载

例如：

```html
<head>
  ...
  <script
    data-goatcounter="https://drunkpig.goatcounter.com/count"
    async
    src="//gc.zgo.at/count.js"></script>
</head>
```

如果你只是把它插在某几个页面里，或者插在页面很后面，而你自己的计数脚本又先执行了，就会出现页面上一直显示“读取中”或者“未知”。

## 第四步：在 GoatCounter 后台打开 visitor counter

只插入 `count.js` 还不够。

如果你还想把“阅读数”显示在网页上，而不只是去 GoatCounter 后台看统计，还要在 GoatCounter 后台打开 visitor counter 功能。

关键设置是：

```text
Allow adding visitor counts on your website
```

这项打开之后，前端页面才能去请求类似下面这样的计数接口：

```text
https://drunkpig.goatcounter.com/counter/xxx.json
```

如果这项没打开，页面读取计数时通常会失败。

## 第五步：先让文章详情页显示阅读数

最容易先做通的是文章详情页。

因为详情页当前浏览器地址就是真实页面地址，可以直接按 GoatCounter 默认的 path 规则取数。

GoatCounter 的默认 path 规则是：

- 优先用 canonical
- 否则用 `location.pathname + location.search`

所以详情页前端脚本可以按这个思路写：

```html
<script>
  (function () {
    var endpointBase = "https://drunkpig.goatcounter.com";
    var path = window.location.pathname + window.location.search;
    var normalizedPath = path === "/" ? "" : path.replace(/^\//, "");
    var endpoint = endpointBase + "/counter/" + encodeURI(normalizedPath) + ".json";

    fetch(endpoint).then(function (response) {
      if (response.status === 404) {
        return { count: 0 };
      }
      if (!response.ok) {
        throw new Error("request failed: " + response.status);
      }
      return response.json();
    }).then(function (data) {
      document.querySelector(".read-count").textContent = data.count || 0;
    }).catch(function () {
      document.querySelector(".read-count").textContent = "未知";
    });
  })();
</script>
```

这里有两个小点很重要：

- `404` 不一定是程序错了，很多时候只是这个页面还没有统计记录，这时显示 `0` 比显示错误更合理
- path 最终要和 GoatCounter 实际记录的 path 保持一致，所以尽量按它的默认规则来

## 第六步：再让首页文章列表显示阅读数

文章列表页的做法和详情页差不多，只是 path 不再来自 `window.location`，而是来自每篇文章自己的链接。

思路是：

1. 页面里先渲染出文章列表链接
2. 遍历每个文章链接
3. 从链接里解析出 `pathname + search`
4. 去 GoatCounter 读对应 path 的计数
5. 把数字填回列表里的占位元素

像这样：

```html
<tr>
  <td><a href="/2026-04-03/GoatCounter静态博客接入教程.html">GoatCounter 静态博客接入教程</a></td>
  <td><span class="read-count">读取中</span></td>
</tr>
```

```html
<script>
  (function () {
    var base = "https://drunkpig.goatcounter.com";
    document.querySelectorAll("tr").forEach(function (row) {
      var link = row.querySelector("a[href]");
      var countNode = row.querySelector(".read-count");
      if (!link || !countNode) return;

      var parsed = new URL(link.getAttribute("href"), window.location.href);
      var path = parsed.pathname + parsed.search;
      var normalizedPath = path === "/" ? "" : path.replace(/^\//, "");
      var endpoint = base + "/counter/" + encodeURI(normalizedPath) + ".json";

      fetch(endpoint).then(function (response) {
        if (response.status === 404) {
          return { count: 0 };
        }
        if (!response.ok) {
          throw new Error("request failed");
        }
        return response.json();
      }).then(function (data) {
        countNode.textContent = data.count || 0;
      }).catch(function () {
        countNode.textContent = "未知";
      });
    });
  })();
</script>
```

这样首页、归档页、标签页都可以用同一套思路扩展。

## 第七步：给页面补 canonical

虽然 GoatCounter 默认可以直接用 `location.pathname + location.search`，但给文章页补一个 canonical 还是值得的。

原因是：

- 统计 path 更稳定
- SEO 更友好
- 以后如果你有不同入口或参数，也更容易统一统计口径

最简单的 canonical 例子：

```html
<link rel="canonical" href="https://mkmerich.com/2026-04-03/GoatCounter静态博客接入教程.html">
```

如果你的生成器能自动根据文章输出路径生成 canonical，最好直接做进模板。

## 第八步：用浏览器开发者工具确认是否生效

接入完成之后，不要只看页面上有没有数字，最好顺手检查这几件事：

1. 页面源码里是否真的包含 GoatCounter 的 `count.js`
2. 这段脚本是不是在每个页面里都有
3. 文章详情页和列表页是否真的发起了 `counter/*.json` 请求
4. 请求返回的是 `200`、`404`，还是别的状态码

一个常见判断方式是：

- `200`：正常，说明拿到计数了
- `404`：通常表示这个 path 还没有统计记录，可以显示 `0`
- `403`：通常说明 visitor counter 没放开，或者请求条件还不对

## 第九步：在静态博客里推荐的接入顺序

如果你想少走弯路，我建议按这个顺序来：

1. 先在 GoatCounter 后台建站
2. 把 `count.js` 脚本插入每个页面
3. 在后台打开 `Allow adding visitor counts on your website`
4. 先做详情页阅读数
5. 确认详情页已经能显示数字或 `0`
6. 再做首页和列表页
7. 最后补 canonical 和样式细节

这样每一步都能单独验证，排错最省时间。

## 常见坑

放到最后简单总结几条最常见的：

- 只插了脚本，但没打开 visitor counter
- 统计脚本插得太晚，自己的读取逻辑先执行了
- 读取计数时用的 path 和 GoatCounter 实际记录的 path 不一致
- 把 `404` 当成错误，导致页面一片“未知”
- 列表页不是从真实文章链接解析 path，而是手写了一套和实际 URL 不一致的路径规则

## 总结

如果你只想把 GoatCounter 顺利接到一个静态博客上，最核心的其实就三件事：

- 每个页面都插入 `count.js`
- 后台打开 visitor counter
- 读取计数时严格按 GoatCounter 的默认 path 规则来

把这三件事做对之后：

- 文章详情页可以显示阅读数
- 首页文章列表也可以显示阅读数
- 新文章没有记录时显示 `0`，而不是一片报错

对个人静态博客来说，这套方案已经够轻、够稳，也足够实用。
