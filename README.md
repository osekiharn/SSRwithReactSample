# SSRwithReactSample



client 側で起動した react app の場合
特筆すべきは　localhost:3000 を見ると
中には div id="root" と  bundle.js  だけの希薄なHTMLが帰ってくる

localhost と
bundle.js のリクエストが完了しないと何もコンテントが表示されない
 
SSRの肝

spaの場合
ブラウザがlocalhostでbundle.jsをリクエストして
リクエストが完了するまでcontent が表示されない

さらには

ブラウザが request page（localhost:3000）して
jsファイルをリクエストして
react app がbootsして jsonをバックエンドにリクエストして
ようやくcontentが表示される

明らかに効率的なソリューションではない。

ssrの場合は

ブラウザが　request page して
メモリ上でreact app がbootする
リクエストデータをフェッチして
HTMLがgenerate されてブラウザに返される

しかし、プロセスはこれで終わりではなく

インタラクティブな操作はまだできない
HTMLは　bundle.jsをリクエストして react app がbootsして
serverからデータが返されてに再び表示される




content　表示

　
Server side rendering is not perfect.
running react on our server is slow. Like really really slow.

react server side rendering is not very fase and it's really quite slow and very computationally intenseive. 計算機負荷が重い


reactDOM には　render とrenderToString と言うfunction がある


```
const express = require('express')
const React = require('react')
const renderToString = require('react-dom/server').renderToString
const Home = require('./client/components/Home').default
const app = express() 

app.get('/', (req, res) => {
  const content = renderToString(<Home />)

  res.send(content)
})

app.listen(3000, () => {
    console.log('Listening on port 3000');
    
})
```

nodejs server は JSX シンタックスがわからない

```
  const content = renderToString(<Home />)
                                 ^
```

解決策として webpack in babel を使う

こんなエラーがでた
```
ERROR in ./src/index.js
Module build failed: Error: Cannot find module '@babel/core'
 babel-loader@8 requires Babel 7.x (the package '@babel/core'). If you'd like to use Babel 6.x ('babel-core'), you should install 'babel-loader@7'.
```

`yarn add @babel/core`

```
Module build failed: Error: Plugin/Preset files are not allowed to export objects, only functions.
```

`yarn add @babel/preset-env`


https://github.com/babel/babel/issues/6808
```
Just like env is now @babel/env, react should be @babel/react and you'll need to install @babel/preset-react.
```

"dev:server": "nodemon --watch build --exec \"node build/bundle.js\"",

Server Side Rendering: generate HTML on the server and ship it down to the user's browser
we're doing server side rendering without react.
we could use say HTML templates on the server like with handlebars. That's really still server side rendering.

They also refer to that as server side rendering

Universal Javascript: The same code runs on the server and the browser
Isomorphic Javascript: The same code runs on the server and the browser

Both these terms essentially mean that some amount of code that we're writing on the server might also be executed on the browser.


section 3 16. Client side js

update Home component

```
const Home = () => {
    return (
        <div>
            <div>I'm VERY VERY BEST home component</div>
            <button
              onClick={() => console.log('Hi there!') }
            >Press me!</button>
        </div>
    )
}
```

but cosole.log does not appear
what's going on here


we make a request to the route the express server sends b ack the HTML that that home component and absolutely nothing else.
There's no javascript code that is being loaded into the browser that sets up that event handler for us.

response はHTMLだけでevent handler は含まれない。

we need to make sure that we somehaw ship down all the javascript code related to our application 
after we ship down all this HTML.

In server side world we are taking care of step number one.

step number one is getting HTML or getting content to show up to the screen.

step number two however is to make sure that we then load up our reactr application and have the react application set up the event handlers and action creators and data loading requests and all that kind stuff ...


17. client bundles

上の問題を解決するには bundle を２つに分ける

bundle1 Server Code + React App
server side code contain sensitive information or sensitive code. for example some secret API keys or special logic that could somehow be exploited.

bundle2 React App

So to implement this we are going to set up a second webpack pipline 

client side 用の webpack.config.js を作る

1. so the first thing we need to do is to remove the target from this webpack config file.
2. next we need to change the entry point of our file.

`src/client/client.js` を作る

we have one entry point for the browser side code base and then the other entry point is our index.js ifle for the server side code base.

`webpack.client.js` の entry を
`entry: './src/client/client.js'`に変える

client用 outputのフォルダを作る

```
output: {
  filename: 'bundle.js',
  path: path.resolve(__dirname, 'build')
}
```

anyone who asks for the javascript file this client side file.

3. client.js を書いていきましょう。

4. the last thing we have to do is make sure we have a npm script to run this new webpack pipline that we've set up.

`"dev:build:client": "webpack --config webpack.client.js --watch"
`

### 18. The Public Directory

However the client bundle is still not actually getting downloaded by the browser.

Well first open all the folders on the public directory to the outside world by telling express to treat this public directory as a freely available public directory.

```js
const app = express() 

app.use(express.static('public')) // 追加
```

So this tells express that it needs to treat that public directory as a static or public directory that is available to the outside world.

Also want to make sure that we tell the actual browser after we generate all this HTML 

So after we generate our content and this is the actual HTML of our application we've going to generate a tiny little HTML document that includes a script tagged inside of it.

```js
app.get('/', (req, res) => {
  const content = renderToString(<Home />)

  const html = `
    <html>
      <head></head>
      <body>
        <div>${content}</div>
        <script src="bundle.js" defer></script>
      </body>
    </html>
  `

  res.send(html)
})
```

### 19. Why Client.js

1. App rendered on the server into some div in the 'template'
2. Rendered app sent to the users browser
3. Browser renders HTML file on the screen, then loads client bundle.
4. Client bundle boots up
5. We manually render the React app a second time into the "same" div
6. React renders our app on the client side, and compares the new HTML to what already exists in the document.
7. React takes over the existing rendered app, binds event handlers, etc

### 20. Client Bootup

We then attempt to render this Home components into DOM. just as though we were building a normal react application.

we have already rendered our application once on the server and we render that into a div that we made in a template in our index.js file we render our up in the browser for the first time.

we want to make sure that we render the app into the same div as the one in the one on the server was rendered into.

```js
// startup point for the client side application
import React from 'react'
import ReactDOM from 'react-dom'
import Home from './components/Home'

ReactDOM.render(<Home />, document.querySelector('#root'))
```


２回目のrender の為に　template に `id="root"` を入れる


```js
<div id="root">${content}</div>
```

これでDOMに息が吹き込まれ、クリックイベントが効くようになる。が hydrationのワーニングがでる

So that process of kind of rendering over the once rendered HTML is refered to as hydration.

ReactDOM.render -> ReactDOM.hydrate

に変える
