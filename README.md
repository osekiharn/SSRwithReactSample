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
