# 30-seconds-of-css

「 一个有用的CSS片段的策划集合 」

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)
    


Explanation

> "version": "1.0.0"

[github source](https://github.com/atomiks/30-seconds-of-css)

~~[english](./README.en.md)~~

---

虽然说-本次的项目主要是供给一些`css有用的片段`

但主要编写的还是一个-`md`-`网页-生成`

---

本目录

- [1. 导入](#1-导入)

- [2-全局变量](#2-全局变量)

- [3-marked-设置](#3-marked-设置)

- [4-模拟浏览器环境](#4-模拟浏览器环境)

- [5-依据-tags-添加侧边-便签名单](#5-依据-tags-添加侧边-便签名单)

- [6-将md文件-html化](#6-将md文件-html化)

- [7-组建-便签-超链接](#7-组建-便签-超链接)

- [8- 加-html-文档-解析](#8-加-html-文档-解析)

- [toKebabCase](toKebabCase)

- [dom](dom)

- [createElement](createElement)


---

## package.json

``` json
  "main": "index.js",
  "scripts": {
    "dev": "nodemon -e md,js ./scripts/build.js & parcel",
    "build": "node ./scripts/build.js && npm run prettier && npm run parcel",
    "parcel": "parcel build index.html -d docs/ --public-url ./",
    "prettier": "prettier --single-quote --no-semi --print-width=100 ./js/index.js --write \"./src/**/*.js\" \"./src/**/*.scss\""
  },
```

主要文件-[`index.js`](#index)

- `npm run dev`

[nodemon](https://github.com/remy/nodemon) 作为一个监控node.js应用程序中的任何更改并自动重启服务器

> `nodemon -e md,js ./scripts/build.js & parce` 启动 build.js 文件 并 观察后缀md，js 文件自动重启

- `parcel`

[`parcel`](https://github.com/parcel-bundler/parcel) 快速发展，零配置的Web应用程序捆绑器

> `parcel build index.html -d docs/ --public-url ./` 构建以`index.html`为头, 输出目录-`docs/` 关键网址路径为 `./` 本目录

- `prettier`

[`prettier`](https://github.com/prettier/prettier) `Prettier`是一个有见识的代码格式化工具。

> `prettier --single-quote --no-semi --print-width=100 ./js/index.js --write \"./src/**/*.js\" \"./src/**/*.scss\"` 就是变漂亮, 有兴趣的看官网😊

- `build`

集合前面所有

> 其实到这里我们可以发现有个未知功能的构建文件`./scripts/build.js`, 同时也是一切的开头-功能的文件

---

让我们看看它`./scripts/build.js`做了什么

---

## build-js

### 1. 导入

代码 1-6

``` js
const fs = require('fs')
const path = require('path')
const marked = require('marked')
const pretty = require('pretty')
const caniuseDb = require('caniuse-db/data.json')
const { toKebabCase, createElement, template, dom } = require('../utils/utils.js')

```

### 2. 全局变量

代码 8-30

``` js
const SNIPPETS_PATH = './snippets'
const TAGS = [
  {
    name: 'all',
    icon: 'check'
  },
  {
    name: 'layout',
    icon: 'layout'
  },
  {
    name: 'visual',
    icon: 'eye'
  },
  {
    name: 'animation',
    icon: 'loader'
  },
  {
    name: 'interactivity',
    icon: 'edit-2'
  }
]
```

### 3. marked-设置

> markdown-解析器和编译器。为速度而建造 [github-source>>](https://github.com/markedjs/marked)

代码 32-42

``` js
const renderer = new marked.Renderer()
renderer.heading = (text, level) => {
  if (level === 3) {
    //   markdown  -  ###
    // 第一种情况
    return `<h${level} id="${toKebabCase(text)}"><span>${text}</span></h${level}>`
  } else {
      // 第二
    return ['HTML', 'CSS', 'JavaScript'].includes(text)
      ? `<h${level} data-type="${text}">${text}</h${level}>`
    //   第三
      : `<h${level}>${text}</h${level}>`
  }
}
renderer.link = (url, _, text) => `<a href="${url}" target="_blank">${text || url}</a>`

```

- 3.1 `renderer` 

> 从 `marked.Renderer()` 获得一个新对象

- 3.2 [`toKebabCase`](#tokebabcase)

> 剔除不正确字符-用`-`串联

- 3.3 `renderer.heading`

> 设置 `###` 转换命令， 有三种情况

<details>

<summary>3⃣️种情况-详解</summary>

---

第一种情况1⃣️

`### Box-sizing reset`<kbd>md</kbd> -> 

`<h3 id="box-sizing-reset"><span>Box-sizing reset</span></h3>`<kbd>html</kbd>

第二2⃣️

`#### Css` ->

`<h4 data-type="CSS">CSS</h4>`

第三3⃣️

`#### Demo` ->

`<h4>Demo</h4>`

---

</details>

<sapn>-</sapn>

- 3.4 `renderer.link`

> 设置-url

`* https://caniuse.com/#feat=text-overflow`<kbd>md</kbd> ->

`<a href="https://caniuse.com/#feat=text-overflow" target="_blank">https://caniuse.com/#feat=text-overflow</a>`<kbd>html</kbd>

### 4. 模拟浏览器环境

``` js
const document = dom('./src/html/index.html')
const components = {
  backToTopButton: dom('./src/html/components/back-to-top-button.html'),
  sidebar: dom('./src/html/components/sidebar.html'),
  header: dom('./src/html/components/header.html'),
  main: dom('./src/html/components/main.html'),
  tags: dom('./src/html/components/tags.html')
}
```

- [dom](#dom)

> 用 JSDOM 给出 浏览器api

---

### 5. 依据-tags-添加侧边-便签名单

``` js
const snippetContainer = components.main.querySelector('.container')
const sidebarLinkContainer = components.sidebar.querySelector('.sidebar__links')
TAGS.slice(1).forEach(tag => {
  sidebarLinkContainer.append(
    createElement(`
      <section data-type="${tag.name}" class="sidebar__section">
        <h4 class="sidebar__section-heading">${tag.name}</h4>
      </section>
    `)
  )
})
```

---

### 6. 将md文件-html化

``` js
for (const snippetFile of fs.readdirSync(SNIPPETS_PATH)) { // 拿到-讲解-css-md 文件
  const snippetPath = path.join(SNIPPETS_PATH, snippetFile) // 路径
  const snippetData = fs.readFileSync(snippetPath, 'utf8') // 数据
  const markdown = marked(snippetData, { renderer }) // 解析 md -> html
  const snippetEl = createElement(`<div class="snippet">${markdown}</div>`) // 放入 html 元素
  snippetContainer.append(snippetEl) // 再 放入 html 元素

  // browser support usage 浏览器支持-情况
  const featUsageShares = (snippetData.match(/https?:\/\/caniuse\.com\/#feat=.*/g) || []).map(
    feat => {
      const featData = caniuseDb.data[feat.match(/#feat=(.*)/)[1]]
      return featData ? Number(featData.usage_perc_y + featData.usage_perc_a) : 100
    }
  )
  const browserSupportHeading = snippetEl.querySelector('h4:last-of-type')
  browserSupportHeading.after(
    createElement(`
      <div>
        <div class="snippet__browser-support">
          ${featUsageShares.length ? Math.min(...featUsageShares).toPrecision(3) : '99+'}%
        </div>
      </div>
    `)
  )

  // sidebar link 侧边-css-超链接
  const link = createElement(
    `<a class="sidebar__link" href="#${snippetFile.replace('.md', '')}">${
      snippetEl.querySelector('h3').innerHTML
    }</a>`
  )

  // tags css-讲解-html的小标签
  const tags = (snippetData.match(/<!--\s*tags:\s*(.+?)-->/) || [, ''])[1]
    .split(/,\s*/)
    .forEach(tag => {
      tag = tag.trim().toLowerCase()
      snippetEl
        .querySelector('h3')
        .append(
          createElement(
            `<span class="tags__tag snippet__tag" data-type="${tag}"><i data-feather="${
              TAGS.find(t => t.name === tag).icon
            }"></i>${tag}</span>`
          )
        )

      sidebarLinkContainer.querySelector(`section[data-type="${tag}"]`).append(link)
    })
}
```


### 7. 组建-便签-超链接

``` js
// build dom
TAGS.forEach(tag =>
  components.tags.append(
    createElement(
      `<button class="tags__tag is-large ${tag.name === 'all' ? 'is-active' : ''}" data-type="${
        tag.name
      }"><i data-feather="${tag.icon}"></i>${tag.name}</button>`
    )
  )
)
const content = document.querySelector('.content-wrapper')
content.before(components.backToTopButton) 
content.before(components.sidebar)
content.append(components.header)
content.append(components.main)
components.main.querySelector('.container').prepend(components.tags)
```

### 8.  加-html-文档-解析

``` js
// doctype declaration gets stripped, add it back in
const html = `<!DOCTYPE html>
${pretty(document.documentElement.outerHTML, { ocd: true })}
`

fs.writeFileSync('./index.html', html)
```



---

## utils


### toKebabCase

``` js
const fs = require('fs')
const { JSDOM } = require('jsdom')

exports.toKebabCase = str =>
  str &&
  str
    .match(/[A-Z]{2,}(?=[A-Z][a-z]+[0-9]*|\b)|[A-Z]?[a-z]+[0-9]*|[A-Z]|[0-9]+/g)
    .map(x => x.toLowerCase())
    .join('-')
```

> 变小写 同时 将 类似 字段含有空格「HELLO World」 => 变 => 「hello-world」

### dom

``` js
exports.dom = path => {
  const doc = new JSDOM(fs.readFileSync(path, 'utf8')).window.document
  return path.includes('component') ? doc.body.firstElementChild : doc
}

```

- `JSDOM` 可以提供 浏览器的api, 用户像在浏览器体验

> 这里是 返回 主document

### createElement


``` js
exports.createElement = str => {
  const el = new JSDOM().window.document.createElement('div')
  el.innerHTML = str
  return el.firstElementChild
}
```

- `JSDOM` 可以提供 浏览器的api, 用户像在浏览器体验

> 这里是返回 html element