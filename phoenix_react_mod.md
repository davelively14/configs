# Tutorial

## How to modify an existing Phoenix act to use React and Redux

#### The Stack

  * Elixir
  * Phoenix Framework
  * Ecto
  * PostgreSQL

#### Front End Packaging

  * Webpack
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
```

* Remove the `brunch-config.js` file from the main directory.

* 
