# npm-stats

> A Vue.js project

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

## API

Endpoints:

  GET https://api.npmjs.org/downloads/point/{period}/{package}

Gets only one point value of total downloads. For more data:

  GET https://api.npmjs.org/downloads/range/{period}/{package}

Period can be defined as `last-day` or `last-month` or specific date range `2017-01-01:2017-10-02`. By default we're keeping it at `last-month`.

Data model:

```javascript
  data () {
    return {
      package: null,
      packageName: '',
      period: 'last-month',
      loaded: false,
      downloads: [],
      labels: [],
      showError: false,
      errorMessage: 'Please enter a package name'
    }
  },
  ```

  ## Template

  * Input field
  * Trigger button for search
  * Error message output
  * Headline with package name
  * Chart

  ### Additions

  * **Input field** has a keyup event on enter
  * `v-model` is bound to `package`

  ### Errors

  1. Search for a package without entering a name: Default error message
  2. Search for a package that does not exist: Grab error message from the request

## Parsing the Data

Response data has to be processed to get one array only with data (downloads) and one array with labels (days). Use **map**.

