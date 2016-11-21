# Tutorial

## How to setup Phoenix with React, Redux and Sass 3

#### The Stack

  * Elixir
  * Phoenix Framework
  * Ecto
  * PostgreSQL

#### Front End Packaging

  * Webpack
  * Sass (organized with CSS Burrito)
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
  - dev.exs:
```
config :new_project, NewProject.Repo,
  adapter: Ecto.Adapters.Postgres,
  database: "new_project_dev",
  hostname: "localhost",
  pool_size: 10
```
  - test.exs:
```
config :new_project, NewProject.Repo,
  adapter: Ecto.Adapters.Postgres,
  database: "new_project_test",
  hostname: "localhost",
  pool: Ecto.Adapters.SQL.Sandbox
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
* Install dependencies
  **Option A:** Open `package.json` and add the base dependencies. Doing it this way is easier, but some of the dependencies may not be up to date. This will work with the configuration.
```
"devDependencies": {
  "babel-core": "^6.13.2",
  "babel-loader": "^6.2.4",
  "babel-preset-es2015": "^6.13.2",
  "babel-preset-react": "^6.11.1",
  "css-loader": "^0.23.1",
  "extract-text-webpack-plugin": "^1.0.1",
  "node-sass": "^3.8.0",
  "sass-loader": "^4.0.0",
  "style-loader": "^0.13.1",
  "webpack": "^1.13.1"
},
"dependencies": {
  "react": "^15.3.0",
  "react-dom": "^15.3.0",
  "react-redux": "^4.4.5",
  "react-router": "^2.6.1",
  "react-router-redux": "^4.0.5",
  "redux": "^3.5.2",
  "phoenix": "file:deps/phoenix",
  "phoenix_html": "file:deps/phoenix_html"
}
```
  Then, from the command line, type:
```
$ npm install
```
  **Option B:** Install most recent updates through the terminal. In the `new_project` directory:
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
* Organize the Sass file strucutre by using `css-burrito`. If you haven't already installed the [package from npm](), do so from the command line:
```
$ npm install -g css-burrito
```
Then, from the `web/static` directory of `new_project`, create the `css-burrito` project:
```
$ burrito -n
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
      web('stylesheets/application.scss'),
      web('js/application.js')
    ],
  },

  output: {
    path: root('priv/static/'),
    filename: 'js/application.js'
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
* Configure Phoenix to to start Webpack every time the dev server is started. Webpack will watch for changes and generate asset functions on the fly. Within `config/dev.exs`, replace the empty `array` value in the `watcher` key in the first config:
```
config :new_project, NewProject.Endpoint,
  http: [port: 4000],
  debug_errors: true,
  code_reloader: true,
  check_origin: false,
  watchers: [
    node: ["node_modules/webpack/bin/webpack.js", "--watch-stdin", "--color", cd: Path.expand("../", __DIR__)]
  ]
```
* Get the Phoenix dependencies and initialize the database
```
mix deps.get
mix ecto.create
```
* Create a blank `web/static/js/application.js` file.
* Move the `private/static/js/phoenix.js` file to the `web/static/js` folder.
* Open `web/templates/layout/app.html.eex`. In order to reference our new stylesheets, and the following to the `head` of the file:
```
<head>
  ...
  <link rel="stylesheet" href="<%= static_path(@conn, "/css/application.css") %>">
</head>
```
* Keep the old with the new?
  **Option A: Keep it.** In order to maintain the custom Phoenix styles and out of the box Bootstrap functionality, move the `web/static/css/app.css` file to the `priv/static/css` and rename it to `phoenix.css`. Delete the `web/static/css` directory. Open `web/templates/layout/app.html.eex` and change the `stylesheet` link from:
```
<link rel="stylesheet" href="<%= static_path(@conn, "/css/app.css") %>">
```
  to:
```
<link rel="stylesheet" href="<%= static_path(@conn, "/css/phoenix.css") %>">
```
  **Option B: Get rid of it.** Simply delete the `web/static/css` directory. Open `web/templates/layout/app.html.eex` and remove the following line:
```
<link rel="stylesheet" href="<%= static_path(@conn, "/css/app.css") %>">
```
* In `web/templates/layout/app.html.eex` file, change the static path pointing to the javascript from `app.js` to `application.js` as seen here:
```
<script src="<%= static_path(@conn, "/js/application.js") %>"></script>
```
* Run the server and everything should be up and running:
```
$ mix phoenix.server
...
[info] Running NewProject.Endpoint with Cowboy using http://localhost:4000
Hash: 01d3f6e25f54222b539d
Version: webpack 1.13.1
Time: 634ms
              Asset      Size  Chunks             Chunk Names
  js/application.js   1.67 kB       0  [emitted]  application
css/application.css  31 bytes       0  [emitted]  application
   [0] multi application 40 bytes {0} [built]
    + 5 hidden modules
Child extract-text-webpack-plugin:
        + 2 hidden modules
```
