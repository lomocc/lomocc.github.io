---
title: Karma 中使用 ChromeHeadless 和 Istanbul
tags: js
---
# `Karma` 中使用 `ChromeHeadless` 和 `Istanbul`
## 使用 `ChromeHeadless`
### 默认配置
```js
// karma.conf.js
process.env.CHROME_BIN = require('puppeteer').executablePath()
module.exports = function(config) {
  config.set({
    browsers: ['ChromeHeadless']
  });
}
```
### 自定义启动参数 `flag`
```js
// karma.conf.js
module.exports = function (config) {
  config.set({
    browsers: ['CustomChromeHeadless'],
    customLaunchers: {
      CustomChromeHeadless: {
        base: 'ChromeHeadless',
        flags: ['--no-sandbox', '--disable-gpu']
      }
    }
  });
}
```
## 测试覆盖率
### 完整依赖包
* `webpack`
* `babel-loader`
* `karma-webpack`
* `puppeteer`
* `jasmine-core`
* `karma-jasmine`
* `karma-coverage`
* `karma-chrome-launcher`
* `karma-sourcemap-loader`
* `karma-coverage-istanbul-reporter`
* `istanbul-instrumenter-loader`
```shell
$ yarn add -D karma webpack babel-loader karma-webpack puppeteer jasmine-core karma-jasmine karma-coverage karma-chrome-launcher karma-sourcemap-loader karma-coverage-istanbul-reporter istanbul-instrumenter-loader
```
### `karma.config.js`
```js
process.env.CHROME_BIN = require('puppeteer').executablePath();
module.exports = function (config) {
  config.set({
    browsers: ['ChromeHeadless'],
    frameworks: ['jasmine'],
    webpack: {
      devtool: 'source-map',
      module: {
        rules: [{
          test: /\.js$/,
          use: {
            loader: 'istanbul-instrumenter-loader',
            options: { esModules: true }
          },
          enforce: 'post',
          exclude: /node_modules|test/
        }, {
          test: /\.js$/,
          use: 'babel-loader'
        }]
      }
    },
    webpackMiddleware: {
      quiet: true
    },
    plugins: [
      'webpack',
      'karma-webpack',
      'karma-jasmine',
      'karma-chrome-launcher',
      'karma-coverage',
      'karma-sourcemap-loader',
      'karma-coverage-istanbul-reporter'
    ],
    exclude: [],
    files: [
      'test/**/*.js',
      'src/**/*.js'
    ],
    preprocessors: {
      'test/**/*': ['webpack', 'sourcemap'],
      'src/**/*.js': ['webpack', 'sourcemap']
    },
    reporters: [ 'progress', 'coverage-istanbul' ],
    coverageIstanbulReporter: {
      reports: ['html', 'text-summary'],
      fixWebpackSourcePaths: true
    },
    singleRun: true
  })
};
```
