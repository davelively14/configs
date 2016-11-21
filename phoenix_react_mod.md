# Tutorial

## How to modify an existing Phoenix app to use React and Redux

#### The Stack

  * Elixir
  * Phoenix Framework
  * Ecto
  * PostgreSQL

#### Front End Packaging

  * Webpack
  * Saas
  * Bootstrap (from Phoenix)
  * React
  * React Router
  * Redux
  * ES6/7

#### Step by Step

* From the command line, amend the default `package.json` by using the `npm init` command. Change the entry point from the default `(brunch-config.js)` to `index.js`. Otherwise, use the defaults or change according to your preference.
```
$ npm init
...
name: (my_app)
version: (1.0.0)
entry point: (brunch-config.js) index.js
...
```

* Remove the `brunch-config.js` file from the main directory.

* Open the `package.json` file in the main directory and remove all dependencies except for `phoenix` and `phoenix_html`. It should look like this:
```
...
"dependencies": {
  "phoenix": "file:deps/phoenix",
  "phoenix_html": "file:deps/phoenix_html"
},
...
```

* From the command line in the main directory, install the required NPM packages:
```
$ npm install --save-dev babel-core babel-preset-es2015 babel-preset-react babel-loader extract-text-webpack-plugin node-sass style-loader css-loader sass-loader webpack
...
...
$ npm install --save react react-router-redux react-router redux react-redux react-dom
```

* Add `/node_modules` to your `.gitignore` file.

* Check the `dependencies` and `devDependencies` sections of the `package.json` file in the main directory. It should resemble this (potentially with newer versions):
```
"dependencies": {
  "phoenix": "file:deps/phoenix",
  "phoenix_html": "file:deps/phoenix_html",
  "react": "^15.3.1",
  "react-dom": "^15.3.1",
  "react-redux": "^4.4.5",
  "react-router": "^2.8.1",
  "react-router-redux": "^4.0.5",
  "redux": "^3.6.0"
},
...
"devDependencies": {
  "babel-core": "^6.14.0",
  "babel-loader": "^6.2.5",
  "babel-preset-es2015": "^6.14.0",
  "babel-preset-react": "^6.11.1",
  "css-loader": "^0.25.0",
  "extract-text-webpack-plugin": "^1.0.1",
  "node-sass": "^3.10.0",
  "sass-loader": "^4.0.2",
  "style-loader": "^0.13.1",
  "webpack": "^1.13.2"
},
```

* Create the `webpack.config.js` file in the main directory to look like this:
```
'use strict'

var path = require('path')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var webpack = require('webpack')

function root(dest) { return path.resolve(__dirname, dest) }
function web(dest) { return root('web/static/' + dest) }

var config = module.exports = {
  entry: {
    application: [
      web('stylesheets/application.scss'),
      web('js/app.js')
    ],
  },

  output: {
    path: root('priv/static/'),
    filename: 'js/app.js'
  },

  resolve: {
    extension: ['', '.js', '.scss'],
    modulesDirectories: ['node_modules']
  },

  module: {
    noParse: /vendor\/phoenix/,
    loaders: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel',
        query: {
          cacheDirectory: true,
          presets: ['react', 'es2015']
        }
      },
      {
        test: /\.scss$/,
        loader: ExtractTextPlugin.extract('style', 'css!sass?includePaths[]=' + __dirname +  '/node_modules')
      }
    ]
  },

  plugins: [
    new ExtractTextPlugin('css/application.css')
  ]
}

if (process.env.NODE_ENV === 'production') {
  config.plugins.push(
    new webpack.optimize.DedupePlugin(),
    new webpack.optimize.UglifyJsPlugin({ minimize: true })
  )
}
```

* Configure Phoenix to to start Webpack every time the dev server is started. Webpack will watch for changes and generate asset functions on the fly. Within `config/dev.exs`, remove the `brunch` configuration inside the `watcher` array and add the `webpack` config. It should look like the following:
```
config :my_app, MyApp.Endpoint,
  http: [port: 4000],
  debug_errors: true,
  code_reloader: true,
  check_origin: false,
  watchers: [
    node: ["node_modules/webpack/bin/webpack.js", "--watch-stdin", "--color"]
  ]
```

* Rename the `web/static/css` directory to `web/static/stylesheets`. Within that directory, create a file named `application.scss`. It can remain blank for now.

* In the same `web/static/stylesheets` directory, rename the `app.css` file to `bootstrap.css` and move that file to the `priv/static/css` directory.

* Open `web/templates/layout/app.html.eex`. In order to reference our new stylesheets, delete the link to `app.css` and add the snippet of code below to the `head` of the file. The link to `application.css` is the Saas output file. If you wish to keep Bootstrap, add the link to to `bootstrap.css` as well.
```
<head>
  ...
  <link rel="stylesheet" href="<%= static_path(@conn, "/css/application.css") %>">
  <link rel="stylesheet" href="<%= static_path(@conn, "/css/bootstrap.css") %>">
</head>
```

* Run the server, and everything should be up and running:
```
$ mix phoenix.server
[info] Running MyApp.Endpoint with Cowboy using http on port 4000
Hash: 110d2532c598add5eaf3
Version: webpack 1.13.2
Time: 1024ms
              Asset     Size  Chunks             Chunk Names
          js/app.js  51.7 kB       0  [emitted]  application
css/application.css  0 bytes       0  [emitted]  application
   [0] multi application 40 bytes {0} [built]
    + 9 hidden modules
Child extract-text-webpack-plugin:
        + 2 hidden modules
```
