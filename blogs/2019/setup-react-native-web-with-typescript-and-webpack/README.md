# Setup React Native Web App with TypeScript and WebPack

In this tutorial we will setup react native web app locally and deploy on [Render](https://render.com/).

Before we start with the [react-native-web](https://github.com/necolas/react-native-web) setup, I assume that you have installed [Node.js](https://nodejs.org/en/), [Yarn](https://yarnpkg.com/lang/en/) and [react-native-cli](https://www.npmjs.com/package/react-native-cli) on your machine.```

### 1. Setup React Native App with Typescript

To create react native app you need to run following command

```
react-native init ReactNativeWebApp
```

This command will create react native app and your app directory will look like as shown in the following image (image 1.1)

![image 1.1: project root](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_1.png)

> image 1.1: project root

To add typescript to our app first we need to install [react-native-typescript-transformer](To add typescript to our app first we need to install react-native-typescript-transformer)

```
yarn add --dev react-native-typescript-transformer typescript
```

Then create a new file named `tsconfig.json` in your project’s root directory and copy-paste the JSON as shown in following code snippet (snippet 1.1)

```js
{
  "compilerOptions": {
    /* Basic Options */
    "target": "esnext" /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */,
    "module": "commonjs" /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */,
    "lib": [
      "esnext",
      "dom"
    ] /* Specify library files to be included in the compilation. */,
    // "allowJs": true,                       /* Allow javascript files to be compiled. */
    // "checkJs": true,                       /* Report errors in .js files. */
    "jsx": "react" /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */,
    // "declaration": true,                   /* Generates corresponding '.d.ts' file. */
    // "declarationMap": true,                /* Generates a sourcemap for each corresponding '.d.ts' file. */
    // "sourceMap": true,                     /* Generates corresponding '.map' file. */
    // "outFile": "./",                       /* Concatenate and emit output to single file. */
    // "outDir": "./dist/",                   /* Redirect output structure to the directory. */
    // "rootDir": "./",                       /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
    // "composite": true,                     /* Enable project compilation */
    // "removeComments": true,                /* Do not emit comments to output. */
    "noEmit": false /* Do not emit outputs. */,
    // "importHelpers": true                  /* Import emit helpers from 'tslib'. */,
    // "downlevelIteration": true,            /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
    // "isolatedModules": true,               /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */
    /* Strict Type-Checking Options */
    "strict": true /* Enable all strict type-checking options. */,
    // "noImplicitAny": true,                 /* Raise error on expressions and declarations with an implied 'any' type. */
    // "strictNullChecks": true,              /* Enable strict null checks. */
    // "strictFunctionTypes": true,           /* Enable strict checking of function types. */
    // "strictBindCallApply": true,           /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
    // "strictPropertyInitialization": true,  /* Enable strict checking of property initialization in classes. */
    // "noImplicitThis": true,                /* Raise error on 'this' expressions with an implied 'any' type. */
    // "alwaysStrict": true,                  /* Parse in strict mode and emit "use strict" for each source file. */
    /* Additional Checks */
    "noUnusedLocals": true /* Report errors on unused locals. */,
    "noUnusedParameters": true /* Report errors on unused parameters. */,
    // "noImplicitReturns": true,             /* Report error when not all code paths in function return a value. */
    // "noFallthroughCasesInSwitch": true,    /* Report errors for fallthrough cases in switch statement. */
    /* Module Resolution Options */
    "moduleResolution": "node" /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */,
    // "baseUrl": "./",                       /* Base directory to resolve non-absolute module names. */
    // "paths": {},                           /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
    // "rootDirs": [],                        /* List of root folders whose combined content represents the structure of the project at runtime. */
    // "typeRoots": [],                       /* List of folders to include type definitions from. */
    // "types": [],                           /* Type declaration files to be included in compilation. */
    // "allowSyntheticDefaultImports": true   /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */,
    "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */,
    // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */
    /* Source Map Options */
    // "sourceRoot": "",                      /* Specify the location where debugger should locate TypeScript files instead of source locations. */
    // "mapRoot": "",                         /* Specify the location where debugger should locate map files instead of generated locations. */
    // "inlineSourceMap": true,               /* Emit a single file with source maps instead of having a separate file. */
    // "inlineSources": true,                 /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */
    /* Experimental Options */
    // "experimentalDecorators": true,        /* Enables experimental support for ES7 decorators. */
    // "emitDecoratorMetadata": true,         /* Enables experimental support for emitting type metadata for decorators. */
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "exclude": ["node_modules"]
}
```

> snippet 1.1: tsconfig.json

Add `tslint.json` in the project’s root directory and copy-paste the JSON content from the following code snippet (snippet 1.2)

```json
{
  "extends": ["tslint-react-recommended"],
  "rules": {
    "ordered-imports": [true],
    "quotemark": [true, "single", "jsx-single", "avoid-escape"],
    "semicolon": [true, "never"],
    "member-access": [false],
    "member-ordering": [false],
    "trailing-comma": [
      true,
      {
        "singleline": "never",
        "multiline": "always"
      }
    ],
    "no-empty": false,
    "no-submodule-imports": false,
    "no-implicit-dependencies": false,
    "no-constant-condition": false,
    "triple-equals": [true, "allow-undefined-check"],
    "ter-indent": [
      true,
      2,
      {
        "SwitchCase": 1
      }
    ],
    "no-duplicate-imports": false,
    "jsx-alignment": true,
    "jsx-no-bind": true,
    "jsx-no-lambda": true,
    "max-classes-per-file": [false]
  }
}
```

> snippet 1.2: tslint.json

Next rename `App.js` to `App.tsx` and add types for react and react native

```
yarn add --dev @types/react @types/react-native
```

Final changes of `App.tsx` shown in the following code snippet (snippet 1.3)

```jsx
import React from 'react';
import { Dimensions, StyleSheet, Text, View } from 'react-native';

const { height } = Dimensions.get('screen');

const App = () => {
  return (
    <View style={styles.container}>
      <View style={styles.center}>
        <Text>Hello React Native Web!!!</Text>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    height
  },
  center: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
});

export default App;
```

Now run `yarn ios` or `yarn android` and you will see screen as shown in following image (image 1.4)

![image 1.3: Final snapshot for mobile
](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_4.png)

> image 1.3: Final snapshot for mobile

### 2. Setup React Native Web & Webpack

1. Add [react-dom](https://www.npmjs.com/package/react-dom) and [react-native-web](https://www.npmjs.com/package/react-native-web) to the project

```
yarn add react-dom react-native-web
```

2. Create a directory named `web` in the project’s root directory and add `index.html` (snippet 1.4) inside the `web` directory

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>
      RNWP
    </title>
    <link href="main.css" rel="stylesheet" />
  </head>
  <body>
    <div id="app-root"></div>
  </body>
</html>
```

> snippet 1.4: index.html

![image 1.2: After adding index.html](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_2.png)

> image 1.2: After adding index.html

3. Add webpack and its configuration inside web directory

To know more about webpack visit https://webpack.js.org/ and to know about webpack-dev-server visit https://webpack.js.org/configuration/dev-server/

```
yarn add --dev webpack webpack-cli webpack-dev-server
```

To add webpack configuration first we need to create a file with name `webpack.config.js` inside `web` directory, then copy-paste content from following snippet (snippet 1.5) into `webpack.config.js`

```js
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const rootDir = path.join(__dirname, '..');
const webpackEnv = process.env.NODE_ENV || 'development';

module.exports = {
  mode: webpackEnv,
  entry: {
    app: path.join(rootDir, './index.web.ts')
  },
  output: {
    path: path.resolve(rootDir, 'dist'),
    filename: 'app-[hash].bundle.js'
  },
  devtool: 'source-map',
  module: {
    rules: [
      {
        test: /\.(tsx|ts|jsx|js|mjs)$/,
        exclude: /node_modules/,
        loader: 'ts-loader'
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, './index.html')
    }),
    new webpack.HotModuleReplacementPlugin()
  ],
  resolve: {
    extensions: [
      '.web.tsx',
      '.web.ts',
      '.tsx',
      '.ts',
      '.web.jsx',
      '.web.js',
      '.jsx',
      '.js'
    ], // read files in following order
    alias: Object.assign({
      'react-native$': 'react-native-web'
    })
  }
};
```

> snippet 1.5 webpack.config.js

We have used [html-webpack-plugin](https://webpack.js.org/plugins/html-webpack-plugin/), [ts-loader](https://github.com/TypeStrong/ts-loader) and we need to add them to our project before we start development

```
yarn add --dev html-webpack-plugin ts-loader
```

Now add `index.web.ts` to the project’s root directory and copy-paste content from following snippet (snippet 1.6) to `index.web.ts`

```js
import { AppRegistry } from 'react-native';
import { name as appName } from './app.json';
import App from './App';

AppRegistry.registerComponent(appName, () => App);

AppRegistry.runApplication(appName, {
  initialProps: {},
  rootTag: document.getElementById('app-root')
});
```

> snippet 1.6: index.web.ts

Now add `web` script command inside `package.json` as shown in image 1.4

![image 1.4: yarn web command
](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_3.png)

> image 1.4: yarn web command

Run `yarn web` command and visit `localhost:8080`. You will see a screen as shown in following image (image 1.5)

![image 1.5: Final snapshot for web
](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_5.png)

> image 1.5: Final snapshot for web

### **Step 3 -> Deploy web app**

Add the following script command to `package.json` and to create a deployable `dist` source directory.

```
"build:web": "cd web && webpack"
```

![image 1.6: diff for the script command](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_6.png)

> image 1.6: diff for the script command

Then run `yarn build:web` which will create a new folder named `dist` with the required files needed to run the web app.

Open `.gitignore` and search for production then add `/dist` below `production` make sure it is uncommented as shown below.

```
# production
/dist
/build
```

We will deploy the web app on [Render](https://render.com/) cloud platform. To setup continuous deployment follow the below steps:

1. Create an account (Render supports only Github / GitLab).

2. If you are successfully created an account then you will see a dashboard as shown in following image (image 1.7)

![image 1.7: Render dashboard](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_7.png)

> image 1.7: Render dashboard

3. Now click `New Web Service` button and connect with Github and install render on your react native web app repository

4. Then you will see your app/repository here https://dashboard.render.com/select-repo?type=web

![image 1.8: Your repository
](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_8.png)

> image 1.8: Your repository

5. Select your repo and add following settings shown in image 1.9, except Name, choose different and unique name for your app and then click on create web service

![image 1.9: Render settings](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_9.png)

> image 1.9: Render settings

6. If everything goes as expected then you can visit your site with the link provided by render, link highlighted in image 1.10

![image 1.10: you will see your link in the highlighted area](/blogs/2019/setup-react-native-web-with-typescript-and-webpack/assets/1_10.png)

> image 1.10: you will see your link in the highlighted area

All the source code is available at https://github.com/vemarav/ReactNativeWebPlayground

If you face any issue, drop a comment📝 👇 in the comments section mentioning the issue and error-details if any.

<div style='text-align:center;font-size:2rem;font-style:italic;font-weight:200;'>We would ❤️ to hear from you</div>
