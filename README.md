# Phase 1

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

# Phase 2

## Adding Datepicker

  yarn add vuejs-datepicker moment

* Import datepicker
* Add datepicker fields for start and end period
* Add data models

New `period` composed of `periodStart` and `periodEnd`, `periodEnd` is set to current day. Have to remove time attribute of `Date()`. Suitable for `moment.js`

### Computing period

* Formatted `startDate`
* Formatted `endDate`
* Composed `period`

### Processing data

Getting labels (day) and data (downloads).

1. Format `day` key to year
2. Remove duplicates to have only unique years
3. Sum all downloads of the same year

Refactor as to extract month and week totals from process. Introduction of helper method to extract logic.

## Transforming time

Creating new data models `rawData` to hold downloads data gotten from the api call, `downloadsYear: []` and `labelsYear: []`. New method `formatYear()`.

Axios promise assigns `data.downloads` to `rawData` and calls `formatYear()`.

### Additional helper methods

For different data transformations of weekly and monthly.

1. removeDuplicate (a, b) {...}
2. getDownloadsPerYear(data) {...}

Import both modules in **Start** and use in `formatYear()` method.

### To get unique dates

Chain a `map()` to `reduce()` and return an object with the date and downloads of that date. `filter()` and `dateToYear()` helper, with a final `map()` to clean things up.

## Refactoring for more charts

Data transformer platform is setup for more additional graphs. `getDownloadsPerYear()` can be duplicated and tweaked along with `dateToYear()` helpers too. Yet this is not DRY.

`getDownloadsPerYear()` method needs to be generalised and changed.

1. Rename method to `groupData`
2. Add second argument to be helper function

