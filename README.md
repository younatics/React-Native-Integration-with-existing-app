# React Native [0.45.1] Intergrating with existing apps

## Purpose
This repository is made for React Natvie users who want to integrate RN[0.45.1] to exisiting application using Swift3, Obbjective-C, JAVA, Kotlin. [Integration with existing apps](http://facebook.github.io/react-native/releases/next/docs/integration-with-existing-apps.html#integration-with-existing-apps) is well documented, but I want to make it more easily. This document is tested recently. Feel free to make Pull Request.
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

> you can't use `expo` when you integrate with exisiting apps.

---
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

---
### Step 3. `Git clone` your `iOS` and `Android` Project
Main purpose of this step is to make structure for cofiguration management. You can just copy and paste to `/android` directory and `/iOS` directory. or `git clone` to make below srtucture.
```
[React native repository] 
|
|- /android [Android repository]
|- /ios [iOS repository]
```

---
### Step4. Modify iOS project
You can do easily beacuse iOS depedency tool is `cocoapods`. Add libraries in `Podfile` and `pod install`
```ruby
#React Native
pod 'React', :path => '../node_modules/react-native', :subspecs => [
	'Core',
	'RCTText',
	'RCTNetwork',
	'RCTImage',
	'RCTWebSocket',
	'DevSupport',
	'BatchedBridge',
	'RCTAnimation'
]
pod 'Yoga', :path => '../node_modules/react-native/ReactCommon/yoga'
```
Make `UIViewController` to present `React Native view`
```Swift
import UIKit
import React
import SnapKit

class ReactViewController: UIViewController, ZBAnSimPopupDelegate {
    var reactView: RCTRootView!
	var item_id: Int?

    override func loadView() {
        super.loadView()
        
        guard let id = item_id else { return }
        let mockData = ["item_id": "\(id)"]
        
        self.reactView = ReactBridge.shared.viewForModule("ReactProject", initialProperties: mockData)
        self.view.addSubview(self.reactView)
        
        self.reactView.snp.makeConstraints { (m) in
            m.edges.equalTo(self.view)
        }
    }
}
```
Add `React Bridge` to communicate with `index.ios.js` <-> `UIViewController`. `forHotReloading` flag is for compile bundle or localhost. 
```Swift
let localUrl = "http://localhost:8081/index.ios.bundle?platform=ios&dev=true"
let forHotReloading = false

class ReactBridge: NSObject {
    static let shared = ReactBridge()
}
extension ReactBridge: RCTBridgeDelegate {
    func sourceURL(for bridge: RCTBridge!) -> URL! {
        return URL(string: localUrl)
    }
    
    func createBridgeIfNeeded() -> RCTBridge {
        let bridge = RCTBridge.init(delegate: self, launchOptions: nil)
        return bridge ?? RCTBridge()
    }
    
    func viewForModule(_ moduleName: String, initialProperties: [String : Any]?) -> RCTRootView {
        if forHotReloading {
            let viewBridge = createBridgeIfNeeded()
            let rootView: RCTRootView = RCTRootView(
                bridge: viewBridge,
                moduleName: moduleName,
                initialProperties: initialProperties)
            return rootView
        } else {
            if let iosBundle = Bundle.main.url(forResource: "main", withExtension: "jsbundle") {
                guard let bundleRootView = RCTRootView(bundleURL: iosBundle, moduleName: moduleName, initialProperties: initialProperties, launchOptions: nil) else { return RCTRootView() }
                return bundleRootView
            }
        }
        return RCTRootView()
    }
}
```
Add `ReactManager.swift` to call method with `Native` <-> `React-Native`
```Swift
@objc(ReactManager) class ReactManager: NSObject {
    var bridge: RCTBridge!
    
    @objc func back(_ reactTag: NSNumber) {
        DispatchQueue.main.async {
            if let view = self.bridge.uiManager.view(forReactTag: reactTag) {
                let presentedViewController = view.reactViewController()
                presentedViewController?.navigationController?.pop(animated: true)
            }
        }
    }
}    
```
We need addtional `Objective-C file` beacuse `React` use macro. Add `ReactManagerBridge.m` file like this
```Swift
#import "ReactManagerBridge.h"
#import "zigbang-Swift.h"

@implementation ReactManagerBridge

RCT_EXPORT_MODULE(ReactManager);

RCT_EXPORT_METHOD(back:(nonnull NSNumber *)reactTag) {
    ReactManager* reactManager = [[ReactManager alloc] init];
    reactManager.bridge = _bridge;
    [reactManager back:reactTag];

}
@end
```

Done!

