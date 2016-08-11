# Tutorial

## How to setup Phoenix with React, Redux and Sass

#### The Stack

  * Elixir
  * Phoenix Framework
  * Ecto
  * PostgreSQL

#### Front End Packaging

  * Webpack
  * Sass
  * React
  * React Router
  * Redux
  * ES6/7

#### Step by Step

* Create new Phoenix project without Brunch. Do NOT install dependencies.
```
$ mix phoenix.new --no-brunch new_project
```
* Ensure that the `new_project/config/dev.exs` and `new_project/config/test.exs` files are configured for Postgres (remove `username` and `password` data).
```
dev.exs
config :new_project, NewProject.Repo,
  adapter: Ecto.Adapters.Postgres,
  database: "new_project_dev",
  hostname: "localhost",
  pool_size: 10
```
```
test.exs
config :new_project, NewProject.Repo,
  adapter: Ecto.Adapters.Postgres,
  database: "new_project_test",
  hostname: "localhost",
  pool: Ecto.Adapters.SQL.Sandbox
```
* Initialize the database:
```
$ mix ecto.create
```
* Navigate to the newly created directory.  Setup the git. Create the repo, `new_project` in this example, on the git account.
```
$ git init
$ git add .
$ git commit -m "Initial commit"
$ git remote add origin https://github.com/davelively14/new_project.git
$ git push -u origin master
```
* In the newly created directory `new_project`, create a new `package.json` file:
```
$ npm init
```
* Option A: Open `package.json` and add the base dependencies.
```
"devDependencies": {
  "babel-core": "^6.13.2",
  "babel-preset-es2015": "^6.13.2",
  "babel-preset-react": "^6.11.1",
  "babel-loader": "^6.2.4",
  "extract-text-webpack-plugin": "^1.0.1",
  "style-loader": "^0.13.1",
  "webpack": "^1.13.1"
},
"dependencies": {
  "react": "^15.3.0",
  "react-dom": "^15.3.0",
  "react-redux": "^4.4.5",
  "react-router": "^2.6.1",
  "react-router-redux": "^4.0.5",
  "redux": "^3.5.2"
}
```
  Then, from the command line, type:
```
npm install
```
* Option B: Install most recent updates through the terminal. In the `new_project` directory:
```
$ npm install --save-dev babel-core babel-preset-es2015 babel-preset-react babel-loader extract-text-webpack-plugin node-sass style-loader css-loader sass-loader webpack
  ...
  ...
$ npm install --save react react-router-redux react-router redux react-redux react-dom
```
  Then, inside `package.json`, add the following to the end of the `"dependencies"` object:
```
"dependencies": {
  ...
  "phoenix": "file:deps/phoenix",
  "phoenix_html": "file:deps/phoenix_html"
}
```
* Create the `webpack.config.js` file in the main directory and configure like this:
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
      web('css/application.sass'),
      web('js/application.js')
    ],
  },

  output: {
    path: root('priv/static/'),
    filename: 'js/application.js'
  },

  resolve: {
    extension: ['', '.js', '.sass'],
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
        test: /\.sass$/,
        loader: ExtractTextPlugin.extract('style', 'css!sass?indentedSyntax&includePaths[]=' + __dirname +  '/node_modules')
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
* Configure Phoenix to to start Webpack every time the dev server is started. Webpack will watch for changes and generate asset functions on the fly. Within `config/dev.exs`, replace the empty `array` value in the `watcher` key in the first config:
```
config :new_project, NewProject.Endpoint,
  http: [port: 4000],
  debug_errors: true,
  code_reloader: true,
  check_origin: false,
  watchers: [
    node: ["node_modules/webpack/bin/webpack.js", "--watch", "--color", cd: Path.expand("../", __DIR__)]
  ]
```
* Get the Phoenix dependencies and initialize the database
```
mix deps.get
mix ecto.create
```
* Create a blank `web\static\js\application.js` file.
* Move the `private/static/js/phoenix.js` file to the `web/static/js` folder.
* Run the server and everything should be up and running:
```
$ mix phoenix.server
```
