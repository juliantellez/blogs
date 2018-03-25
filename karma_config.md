## Karma config

### Setting the browsers

```javascript
const browersConfig = {
    browserNoActivityTimeout: 60000,
    browsers: ['ChromeNoSandbox', 'FirefoxHeadless', 'Opera', 'Safari'],
    customLaunchers: {
        ChromeNoSandbox: {
            base: 'ChromeHeadless',
            flags: ['--no-sandbox'],
        },
        FirefoxHeadless: {
            base: 'Firefox',
            flags: ['-headless'],
        },
    },
}
```

### Setting the reporters
```javascript
const reportersConfig = {
    frameworks: ['mocha', 'sinon-chai'],
    reporters: ['mocha', 'coverage', 'coverage-istanbul'],

    coverageReporter: {
        dir: path.join(__dirname, '../coverage'),
        reporters: [
            { type: 'text' },
            { type: 'text-summary' },
        ],
        check: {
            global: {
                statements: 99,
                branches: 90,
                functions: 90,
                lines: 90,
            },
        },
    },

    client: {
        mocha: {
            timeout: '10000',
        },
    },
}
```


```javascript
const path = require('path');

const webpackConfig = require('./webpack.config');

const karmaConfig = Obejct.assign(
    browersConfig,
    reportersConfig,
{

    singleRun: true,
    colors: true,
    port: process.env.PORT || 4000,


    files: [
        path.join(__dirname, 'main.js'),
    ],
    preprocessors: {
        'main.js': ['webpack', 'sourcemap'],
    },
    webpack: webpackConfig,
};

module.exports = config => config.set(karmaConfig);

```

```
const config = {
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: [
                    /\.spec\.js/,
                    /node_modules/,
                ],
                use: [
                    {
                        loader: 'istanbul-instrumenter-loader',
                    },
                    {
                        loader: 'babel-loader',
                    },
                ],

            },
        ],
    },
};

module.exports = config;

```