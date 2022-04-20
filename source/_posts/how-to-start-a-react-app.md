---
title: 从零开始搭建一个 React APP
date: 2022-04-21 03:26:15
tags:
---

学习不依赖任何脚手架搭建一个 react 项目

<!-- more -->

> Notice: Use yarn as package manage, node version v12.22.0.

## Steps

### Step One: Create project-folder

1. Create Folder

```bash
mkdir project-name
cd project-name
```

2. Init npm

```bash
yarn init
```

3. Init git

```bash
git init
```

4. Create git ignore file 

```bash
touch .gitignore
```

And enter those line

```bash
## ide relative
/.idea

## dependencies
/node_modules

## log file
yarn-error.log

## dist
/dist
```

### Step Two: Initial project construction

1. Create config folder
2. Create src folder
3. Create public folder

### Step Three: Install dependencies

You will need `webpack`, `webpack-cli`, `webpack-dev-server` as your devDependencies.

And you also need `react`, `react-dom` as you dependencies.

Use command below to add dependencies.

```bash
yarn add webpack webpack-cli webpack-dev-server --dev 
yarn add react react-dom
```

#### Step Four: Write entry file

Create an index.jsx in src folder

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

const App = () => {
  return (<div>Hello world</div>)
}

ReactDOM.render(<App/>, document.getElementById('root'));
```

Create an index.css in src folder

```css
div {
  color: blue
}
```

### Step Five: Write entry html

Create an index.html in public folder

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

### Break: Why is the project so complex

Actually, in this position, you may think I can run this app, but there are too many questions for browsers to run the project.

- HTML doesn't involve javascript file.
- Browser doesn't recognize jsx extension.
- JSX syntax isn't supported in any browser.
- ES6 syntax isn't supported in all browsers.

So, we need to use webpack to build the project.

### Step Six: Write your config

In config folder, you may create a file named webpack.config.js

Before that, you need to install so many dependencies
`html-webpack-plugin`, `babel-loader`, `css-loader`, `style-loader`, `url-loader`

```bash
yarn add html-webpack-plugin babel-loader css-loader style-loader url-loader --dev
```

And then, copy those in, I already wrote comments in this file.

```javascript
const path = require('path');
const HTMLWebpackPlugin = require('html-webpack-plugin')

const isDev = process.env.NODE_ENV === 'development'

module.exports = {
  // Switch mode depend on process env
  mode: isDev ? 'development' : 'production',
  // enable source-map generation
  devtool: 'source-map',
  // specific entry to src/index.jsx
  entry: {
    index: path.resolve(__dirname, '..', 'src', 'index.jsx'),
  },
  // specific build result to dist folder, as [entry-key].js 
  output: {
    path: path.resolve(__dirname, '..', 'dist'),
    libraryTarget: "umd",
    globalObject: 'typeof self !== \'undefined\' ? self : this',
    filename: '[name].js',
    publicPath: "/"
  },
  // declare which kind of file will load by which loader
  module: {
    rules: [
      {
        // jsx and js file will be loaded by babel-loader
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: [{
          loader: 'babel-loader',
        }]
      },
      {
        // css file will be loaded by css-loader & style-loader
        test: /\.css$/,
        use: [{
          loader: "style-loader"
        }, {
          loader: 'css-loader',
        }]
      },
      {
        // jpeg, png, gif, svg will be loaded by url loader
        test: /\.(jpeg|jpg|png|gif|svg)$/,
        use: [{
          loader: 'url-loader'
        }]
      }
    ]
  },
  plugins: [
    // use public/index.html as template, generate dist/index.html
    new HTMLWebpackPlugin({
      template: path.resolve(__dirname, '..', 'public', 'index.html')
    })
  ],
  resolve: {
    // auto resolve file extension
    extensions: [".js", ".jsx"]
  },
  // specific dev server will open on both localhost and ip:port
  devServer: {
    host: '0.0.0.0',
    hot: true,
    historyApiFallback: true,
    client: {
      overlay: {errors: true, warnings: false},
    },
  },
};
```

Is that easy to understand this config file? You need one more step to start local dev.

Install those babel dependencies `@babel/core`, `@babel/preset-env`, `@babel/preset-react`

```bash
yarn add @babel/core @babel/preset-env @babel/preset-react --dev
```

Create `babel.config.js` in root folder, and then copy those in

```javascript
module.exports = {
  "presets": [
    // parse es6 syntax
    [
      "@babel/preset-env",
      {
        "targets": {
          "chrome": "58",
          "ie": "9",
          "android": "4.4",
          "ios": "9"
        },
      }
    ],
    // parse jsx syntax
    "@babel/preset-react"
  ],
};
```

This file will config how babel-loader handle .js and .jsx file

### Finally development: Start your server
Actually, you can run 

```
npx webpack-dev-server --config ./config/webpack.config.js
```

Then you will see command line show IP:port, click it or directly open http://localhost:8080. You will see a blue hello world

You can also create a dev script for your project, is not required, but it's usefull.

Create a script key in package.json, like this

```json
{
  ...others,
  "scripts": {
    "dev": "webpack-dev-server --config ./config/webpack.config.js"
  },
  ...others
}
```

Then you can run yarn dev to start local develop server.
Enjoy :)

### Finally deployment: Build your project

Actually, you can run 

```
npx webpack --config ./config/webpack.config.js
```

Same, you can create a build script for your project too.
Create a script key in package.json, like this

```json
{
  ...others,
  "scripts": {
    "build": "webpack --config ./config/webpack.config.js"
  },
  ...others
}
```

Then you can run yarn build to build a production result, the output will show in dist folder.

If you want to test build result, you can use serve command, need install serve in you computer

Then use serve dist, it will start a static server on your computer, you can open http://localhost:3000 to see build result.

Enjoy

## Optimize

### Optimize 1: Enable Typescript
1. Install typescript

```bash
yarn add typescript ts-loader --dev
## this command is require too, react and react-dom doesn't default include type declartion
yarn add @types/react @types/react-dom --dev
```

2. Create tsconfig file

tsconfig.json is used to declare how typescript handle ts(x?) file.

Create tsconfig.json in root folder

```json
{
    "compilerOptions": {
        "lib": ["es2015", "dom"],
        "target": "es5",
        "module": "esnext",
        "moduleResolution": "node",
        "allowJs": true,
        "checkJs": true,
        "esModuleInterop": true,
        "jsx": "react",
        "allowSyntheticDefaultImports": true,
    },
    "include": ["./src/*"],
    "exclude": ["./node_modules/*"]
}
```

3. Create a typescript entry

Create or change the previous file to src/index.tsx

```typescript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

interface Props {
  text: string;
}

const App: React.FC<Props> = ({ text }) => {
  return (<div>{text}</div>)
}

ReactDOM.render(<App text="hello world"/>, document.getElementById('root'));
```

4. Change webpack config

Previous config file didn't declare how to handle typescript file(.tsx, .ts), we need add one more loader to handle those.

```javascript
module.exports = {
  // specific entry to src/index.tsx
  entry: {
    index: path.resolve(__dirname, '..', 'src', 'index.tsx'),
  },
  // declare which kind of file will load by which loader
  module: {
    rules: [
      {
        // tsx and ts file will be loaded by ts-loader then babel-loader
        test: /\.tsx?$/,
        exclude: /node_modules/,
        use: [{
          loader: 'babel-loader',
        }, {
          loader:"ts-loader"
        }]
      },
    ]
  },
  resolve: {
    // auto resolve file extension
    extensions: [".js", ".jsx", '.ts', '.tsx']
  },
  // ...don\'t change
};
```

### Optimize 2: Enable Eslint

We will add eslint to our project, we will start with typescript.

1. Install dependencies

```bash
yarn add eslint eslint-webpack-plugin --dev
# load parser and plugins for typescript
yarn add @typescript-eslint/eslint-plugin eslint-plugin-react @typescript-eslint/parser --dev
```

2. Create config file

Create .eslintrc.js file in the root folder

```javascript
module.exports = {
  "extends": [
    "plugin:react/recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  plugins: ['@typescript-eslint'],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    // 支持最新 JavaScript
    ecmaVersion: 2018,
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true // Allows for the parsing of JSX
    }
  },
  env: {
    es6: true,
    node: true,
  },
  rules: {
    "semi": "error", // test feature
  },
  ignorePatterns: [
    "config",
    "public",
  ],
};
```

3. Try out

You can open src/index.tsx, remove semi after each line, you will see your ide give wrong message.

4. Add it to build flow

You may notice that we install eslint-loader, so let use put eslint stage in the build flow.

In config/webpack.config.js

```javascript
const ESLintPlugin = require('eslint-webpack-plugin');

module.exports = {
  plugins: [
    // use public/index.html as template, generate dist/index.html
    new HTMLWebpackPlugin({
      template: path.resolve(__dirname, '..', 'public', 'index.html')
    }),
    new ESLintPlugin({
      extensions: ['.js','.ts','.tsx','.jsx'],
    })
  ],
  // ...don\'t change
};
```

Then, run yarn dev, you will see build failed message due to missing semicolon

You can add `fix: true` to ESLintPlugin options like this, then restart you dev server(ctrl + c to stop, yarn dev to run), then you will see the file has been auto fixed, and server auto start.

```javascript
new ESLintPlugin({
  extensions: ['.js','.ts','.tsx','.jsx'],
  fix: true
})
```

> Why we need add eslint to build flow?
> 
> Normally, we do not develop a project by ourselves, other people may change some code in your project, they can choose ignore editor warning. So add it to build flow is good for project clean.

### Optimize 3: Enable Less

Sometimes you may want to use less/scss to write your stylesheet. We need do something to enable that.

1. Install dependencies

```
yarn add less less-loader --dev
```

2. Create a less file

Create or change the previous file to src/index.less

```less
@color: red;

div {
  color: @color;
}
```

Remember change reference in src/index.tsx.

```javascript
import './index.less'
```

3. Change webpack config

Previous config file didn't declare how to handle less file(.less), we need add one more loader to handle those. Notice, we only need to change the previous CSS pattern.

```javascript
module.exports = {
  module: {
    rules: [
      {
        // less, css file will be loaded by css-loader & style-loader
        test: /\.(le|c)ss$/,
        use: [{
          loader: "style-loader"
        }, {
          loader: 'css-loader',
        }, {
          loader: 'less-loader',
        }]
      },
    ]
  },
};
```

4. Try out

Run yarn dev to try less file result

### Optimize 4: Use Alias

Once your project gets bigger, there will be so many folders, reference file will show so many '../../../'

```javascript
import xxx from '../../../utils/test'
```

It is not good for understanding.

So, alias can help you simplize the reference code.
Before all, we need to create another file for test.

Create src/utils/add.ts

```typescript
export const add = (a: number, b: number) => {
  return a + b;
};
```

If we want to use this add function, we will use like this in src/index.tsx

```typescript
import React from 'react';
import ReactDOM from 'react-dom';
import {add} from './utils/add';
import './index.less';

interface Props {
  text: string,
}

const App: React.FC<Props> = ({ text }) => {
  return (<div>{text} {add(1, 2)}</div>);
};

ReactDOM.render(<App text='hello typescript' />, document.getElementById('root'));
```

1. Add Webpack config

```javascript
const path = require('path');

module.exports = {
  resolve: {
    // src folder can be use with @/
    alias: {
      '@': path.resolve(__dirname, '..', 'src'),
    },
    // auto resolve file extension
    extensions: [".js", ".jsx", '.ts', '.tsx']
  },
  
};
```

2. Add Typescript config

If you are using typescript, you need one more step.
In tsconfig.json, add 

```json
{
  "compilerOptions": {
      "baseUrl": "./src/",
      "paths": {
        "@/*": ["./*"]
      }
  },
}
```

3. Try out.

Change src/index.tsx to

```javascript
import {add} from '@/utils/add';
```

Run yarn dev to see result.


## Project

### package.json

```json
{
  "name": "min-react",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "dev": "webpack-dev-server --config ./config/webpack.config.js",
    "build": "webpack --config ./config/webpack.config.js"
  },
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@babel/core": "^7.17.9",
    "@babel/preset-env": "^7.16.11",
    "@babel/preset-react": "^7.16.7",
    "@types/react": "^18.0.5",
    "@types/react-dom": "^18.0.1",
    "@typescript-eslint/eslint-plugin": "^5.19.0",
    "@typescript-eslint/parser": "^5.19.0",
    "babel-loader": "^8.2.4",
    "css-loader": "^6.7.1",
    "eslint": "^8.13.0",
    "eslint-plugin-react": "^7.29.4",
    "eslint-webpack-plugin": "^3.1.1",
    "html-webpack-plugin": "^5.5.0",
    "less": "^4.1.2",
    "less-loader": "^10.2.0",
    "style-loader": "^3.3.1",
    "ts-loader": "^9.2.8",
    "typescript": "^4.6.3",
    "url-loader": "^4.1.1",
    "webpack": "^5.72.0",
    "webpack-cli": "^4.9.2",
    "webpack-dev-server": "^4.8.1"
  }
}
```

### Codebase
https://code.byted.org/liyuepeng.lester/template-mini-react

