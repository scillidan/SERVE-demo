```sh
fnm install 16.20.0
fnm use 16.20.0
```

```sh
set NODE_ENV=production
npm i
npm install sequelize sequelize-cli --save
cd clinet
npm i
npm update vue
npm update fork-ts-checker-webpack-plugin
npm run generate
cd ..
```

Edit client/nuxt.config.js:

```
module.exports = {
...
server: {
  ...
  host: '127.0.0.1'
},
```

```sh
npm start
```

At same time:

```sh
cd client
npm start
```