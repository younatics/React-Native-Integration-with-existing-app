# React Native [0.45.1] Intergrating with existing apps

## Purpose
This repository is made for React Natvie users who want to integrate RN[0.45.1] to exisiting application. [Integration with existing apps](http://facebook.github.io/react-native/releases/next/docs/integration-with-existing-apps.html#integration-with-existing-apps) is well documented, but I want to make it more easily. This document is tested recently. Feel free to make Pull Request.
Thanks.

## Develop Enviroment
This document is based on below version. React Native currenly changes a lot, so be careful.

```javascript
"dependencies": {
		"react": "16.0.0-alpha.12",
		"react-native": "0.45.1"
	}
```
If you have `iOS` and `Android` application independently, this will be perfect guide for you. Please follow these steps.

## Getting started

### Step 1. Install tools and init project 
#### Install tools
you can also see this in [Getting Started](http://facebook.github.io/react-native/releases/next/docs/getting-started.html)
```
brew install node
brew install watchman
npm install -g react-native-cli
```

If you all done this process, init your project.
```
react-native init ReactProject
```
In your `ReactProject` directory, you can find `package.json`. this should be look like this.
```javascript
{
	"name": "ReactProject",
	"version": "0.0.1",
	"private": true,
	"scripts": {
		"start": "node node_modules/react-native/local-cli/cli.js start",
		"test": "jest"
	},
	"dependencies": {
		"react": "16.0.0-alpha.6",
		"react-native": "0.44.0"
	},
	"devDependencies": {
		"babel-jest": "20.0.3",
		"babel-preset-react-native": "1.9.2",
		"jest": "20.0.4",
		"react-test-renderer": "16.0.0-alpha.6"
	},
	"jest": {
		"preset": "react-native"
	}
}
```

> you can not use `expo` when you integrate with exisiting apps.

### Step 2. Add `postinstall` in `package.json`
```javascript
{
	"name": "ReactProject",
	"version": "0.0.1",
	"private": true,
	"scripts": {
		"postinstall": "sed -i '' 's/#import <RCTAnimation\\/RCTValueAnimatedNode.h>/#import \"RCTValueAnimatedNode.h\"/' ./node_modules/react-native/Libraries/NativeAnimation/RCTNativeAnimatedNodesManager.h",
		"start": "node node_modules/react-native/local-cli/cli.js start",
		"test": "jest"
	},
	"dependencies": {
		"react": "16.0.0-alpha.12",
		"react-native": "0.45.1"
	},
	"devDependencies": {
		"babel-jest": "20.0.3",
		"babel-preset-react-native": "2.0.0",
		"jest": "20.0.4",
		"react-test-renderer": "16.0.0-alpha.12"
	},
	"jest": {
		"preset": "react-native"
	}
}
```
Currently, RN 0.45.1 have a problem in `iOS - #import <RCTAnimation\\/RCTValueAnimatedNode.h>`. You need to change it to `#import "RCTValueAnimatedNode.h`. So use `postinstall`

### Step 3. `Git clone` your `iOS` and `Android` Project
Main purpose of this step is to make structure for cofiguration management. You can just copy and paste to `/android` directory and `/iOS` directory. or `git clone` to make below srtucture.
```
[React native repository] 
|
|- /android [Android repository]
|- /ios [iOS repository]
```


