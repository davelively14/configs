# Tutorial

## How to setup Phoenix 1.3+ with React / Redux and Webpack 2

#### The Stack

  * Elixir
  * Phoenix Framework
  * Ecto
  * PostgreSQL

#### Front End Packaging

  * Webpack 2
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
```elixir
dev.exs
...
config :new_project, NewProject.Repo,
  adapter: Ecto.Adapters.Postgres,
  database: "new_project_dev",
  hostname: "localhost",
  pool_size: 10
...
```
```elixir
test.exs
...
config :new_project, NewProject.Repo,
  adapter: Ecto.Adapters.Postgres,
  database: "new_project_test",
  hostname: "localhost",
  pool: Ecto.Adapters.SQL.Sandbox
...
```
* Navigate to the newly created directory.  Setup the git. Create the repo, `new_project` in this example, on the git account.
```
$ git init
$ git add .
$ git commit -m "Initial commit"
$ git remote add origin https://github.com/davelively14/new_project.git
$ git push -u origin master
```
* In the newly created directory `new_project`, create a new `package.json` file and select the defaults:
```
$ npm init
```
* Option A: Open `package.json` and add the base dependencies.
```json
"devDependencies": {
  "babel-core": "^6.24.0",
  "babel-loader": "^6.4.1",
  "babel-preset-es2015": "^6.24.0",
  "babel-preset-react": "^6.23.0",
  "webpack": "^2.3.2"
},
"dependencies": {
  "react": "^15.4.2",
  "react-dom": "^15.4.2",
  "react-redux": "^5.0.3",
  "react-router": "^4.0.0",
  "react-router-redux": "^4.0.8",
  "redux": "^3.6.0"
}
```
  Then, from the command line, type:
```
$ npm install
```
* Option B: Install most recent updates through the terminal. In the `new_project` directory:
```
$ npm install --save-dev babel-core babel-preset-es2015 babel-preset-react babel-loader webpack
  ...
  ...
$ npm install --save react react-router-redux react-router redux react-redux react-dom
```

* For either option, open `package.json`, add the following to the end of the `"dependencies"` object:
```json
"dependencies": {
  ...
  "phoenix": "file:deps/phoenix",
  "phoenix_html": "file:deps/phoenix_html"
}
```
* Add `/node_modules` to your `.gitignore` file.

* Create the `webpack.config.js` file in the main directory and configure like this (note: this is for Webpack 2 - it won't work if you're using an older version of Webpack). Also, replace [ADD APP NAME] with your own app.
```javascript
'use strict'

var path = require('path')
var webpack = require('webpack')

function root(dest) { return path.resolve(__dirname, dest) }
function web(dest) { return root('lib/[ADD APP NAME]/web/static/' + dest) }

var config = module.exports = {
  entry: {
    application: [
      web('js/application.js')
    ],
  },

  output: {
    path: root('priv/static/'),
    filename: 'js/application.js'
  },

  resolve: {
    extensions: ['.js'],
    modules: ['node_modules']
  },

  module: {
    noParse: /vendor\/phoenix/,
    loaders: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        query: {
          cacheDirectory: true,
          presets: ['react', 'es2015']
        }
      }
    ]
  },

  plugins: []
}

if (process.env.NODE_ENV === 'production') {
  config.plugins.push(
    new webpack.optimize.DedupePlugin(),
    new webpack.optimize.UglifyJsPlugin({ minimize: true })
  )
}
```
* Configure Phoenix to to start Webpack every time the dev server is started. Webpack will watch for changes and generate asset functions on the fly. Within `config/dev.exs`, replace the empty `array` value in the `watcher` key in the first config. We `--watch-stdin` so that we automatically kill the process when we close the server.
```elixir
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
* Create a `lib/web/static/js/application.js` file and add the following code:
```javascript
'use strict';

var elements = document.querySelectorAll('[data-submit^=parent]');
var len = elements.length;

for (var i = 0; i < len; ++i) {
  elements[i].addEventListener('click', function (event) {
    var message = this.getAttribute("data-confirm");
    if (message === null || confirm(message)) {
      this.parentNode.submit();
    };
    event.preventDefault();
    return false;
  }, false);
};
```
* In the `web/templates/layout/app.html.eex` file, change the static path pointing to the javascript from `app.js` to `application.js` as seen here:
```
<script src="<%= static_path(@conn, "/js/application.js") %>"></script>
```
* Run the server and everything should be up and running:
```
$ mix phoenix.server
```

## Heroku Deployment Tips

In order to get Phoenix working on Heroku using the instruction from the [Phoenix website](http://www.phoenixframework.org/docs/heroku), I've had to add a couple files.

* Add `elixir_buildpack.config` to the root directory with the following contents:
```
always_rebuild=true
```

* Add `Procfile` (note the capital 'P') to the root directory with the following contents:
```
web: MIX_ENV=prod mix phoenix.server
```

* Add `compile` (note no extension) to the root directory with the following contents:
```
./node_modules/.bin/webpack -p
mix phoenix.digest
```

On occasion, I've received the "App Not Started" page from Heroku when trying to load the page. In order to get past, I've had to restrt the dynos. At the command line from the root directory, enter these commands:
```
$ heroku ps:scale web=0
$ heroku ps:scale web=1
```

Or, you can simply use the restart command:
```
$ heroku restart
```