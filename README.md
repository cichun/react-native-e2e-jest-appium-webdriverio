# E2E testing for React Native with Jest, Appium and WebDriverIO (iOS and Android)

[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen)](./CONTRIBUTING.md)



## Info
It's a fork of https://github.com/kelset/react-native-e2e-jest-appium-webdriverio just to showcase problems with getting a component in Appium tests for iOS when there are Deep Components Tree.

A solution for some people might be running the app in _**React Native New Architecure**_, which uses **Fabric** as a renderer. The Fabric uses a mechanism called View Flattening, which might solve this problem. 
Before reading further read 2 articles to get basic knowledge:
 - https://reactnative.dev/architecture/view-flattening
 - https://github.com/reactwg/react-native-new-architecture/discussions/110





Original README can be found here(please read it you have problems running the project):
https://github.com/kelset/react-native-e2e-jest-appium-webdriverio/blob/main/README.md

## The App
The test app has 2 inputs and a deeply nested _Login_ button. 
Appium will fail getting the _Login_ button, because it is more than 50 levels nested in The Components Tree.


<img src="assets/theTestApp.png" width="400">

## Run the tests
To run the tests run command:
`yarn test:e2e:ios`

The tests will fail because the 'Login' button is nested deeply in the Components Tree (109 levels deep)
![appiumTestsFails.png](assets%2FappiumTestsFails.png)

### Appium Inspector
If you'll try to see the app structure in _Appium Inspector_ you'll notice that only first 50 levels of Compoonents Tree are being recognized.
![appiumInspector1.png](assets%2FappiumInspector1.png)

> The magic number 50 is called **snapshotMaxDepth** and its the default value https://appium.github.io/appium-xcuitest-driver/4.16/settings/
you can increase it using 'appium:settings[snapshotMaxDepth]': 62, look at file _e2e-config.ts_
Bad things will happen If you try to push it further - 62 is total max!! 

### Apple Accessibility Inspector
The Components Tree looks similar in Apple Accessibility Inspector:
![accessibilityInspector1.png](assets%2FaccessibilityInspector1.png)

# React Native New Architecture & View Flattening to the rescue 🛟
1) Reinstal pods to use the RN New Architecture
```
cd ios
RCT_NEW_ARCH_ENABLED=1 bundle exec pod install
```

2. rebuild the app
```
yarn run ios
```

3. start Metro
```
yarn start
```

In the Metro console you should notice a flag indicating it uses RN New Architecture and Fabric Renderer
![MetroFabric.png](assets%2FMetroFabric.png )

This means that now the app uses The View Flattening Mechanism https://reactnative.dev/architecture/view-flattening which reduces unnecessary component levels.

## Run the tests again
`yarn test:e2e:ios`
Now it works!
![AppiumTestsSucceeds.png](assets%2FAppiumTestsSucceeds.png)

### Appium Inspector again
![appiumInspector2.png](assets%2FappiumInspector2.png)

The Components Tree has been flattened - now the `Login` button is in the 10th level - so it is accessible to appium webdriver.

also in Apple Accessibility Inspector, the tree looks much better:
![accessibilityInspector2.png](assets%2FaccessibilityInspector2.png)



# Known Issues
There's a known issue with getting element value in Fabric, more on this:
https://github.com/facebook/react-native/issues/38709



# The original README
<details>
<summary>
show original README
</summary>

In this repo you will find a sample project to showcase how to do E2E testing with [Jest](https://jestjs.io/) + [Appium](https://appium.io/) + [WebDriverIO](https://webdriver.io/) for Android and iOS on react-native.
_It's a bit janky but it serves the purpose of showcasing how to a basic setup needs to be correctly wired._

## How to use

First off, install the needed tooling:

```bash
npm install appium@2.0.0-beta.53 -g
appium driver install uiautomator2
appium driver install xcuitest
```

> More details about drivers in Appium [here](https://appium.github.io/appium/docs/en/2.0/guides/managing-exts/) and [here](https://appium.github.io/appium/docs/en/2.0/quickstart/uiauto2-driver/)

Once you have done that, you can get the repo locally via the classic `git clone git@github.com:kelset/react-native-e2e-jest-appium-webdriverio.git` command (I prefer SSH over HTTP for cloning, but you do you).

Then you can navigate right into the codebase via `cd react-native-e2e-jest-appium-webdriverio`, followed by a `yarn install` command to install all the necessary dependencies.

After this, run the app on the Android emulator/ iOS simulator via `yarn android`/`yarn ios` - **you need to do this at least once** (for simplicity sake, we want the app to be already installed on the simulator/emulator before testing).

Once the app is on the emulator/simulator and Metro is running, you can open a new terminal window and start the Appium server via `yarn start:appium`.

With the server is running, you can use the commands `test:e2e:android` and `test:e2e:ios` to try out the E2E loop, or use `test:e2e:all` to run both one after the other.

### A note on setup

Please make sure that your local emulator/simulator config matches the `e2e-config.js` setup or it will fail 'cause it won't be able to connect to the platform.

## Notes on E2E: how does it work?

I'll try to keep it simple and to walk through all the main concepts and hacks that make it all work.

The basic premise is that this is, from Appium's perspective, just a project like any other: the app it needs to test is a black box, and it gets to communicate with it via webdriverIO's client.

Via the command `test:e2e:android` we start the testing, that starts up the `basicE2E.test.js` script - this file gets via an helper script `e2e-config.js` which platform to test (passed as an ENV variable, `E2E_DEVICE` during the yarn command, check `test:e2e:android` in `package.json`) and goes into the `package.json`, section `e2e`, and uses those info the `beforeAll` to stand up the webdriverIO client.

Then the actual testing is done by using as "communication point" to invoke the native components this following pattern `client.$('~<string>')` (the ~ is intentional, and important!). The `<string>` here is what we setup in `App.js`, and it should be just the `accessibilityID` option (that RN passes back to the native component) but actually we need to use a bit of a workaround script called `testProps` (at the top of `App.js`) to tailor this use for iOS/Android and for the `Text` component. (_huge props to Slav Kurochkin for finding this out and explaining it [in this article](http://93days.me/testing-react-native-application/)_)

This way we can interact with all the elements on screen that have their string setup as props via `{...testProps(<string>)}`.

If this isn't clear enough or you'd like a blogpost on this subject, feel free to [open an issue](https://github.com/kelset/react-native-e2e-jest-appium-webdriverio/issues/new) or talk to me [over on Twitter](https://twitter.com/kelset).

## Inspiration and resources

Getting this together was quite a bit of work because there aren't many resources around that walk you through the entire setup for React Native Android/iOS - I pieced this sample app together by following and taking bit and pieces from multiple places. In no particular order:

- https://appium.io/docs/en/about-appium/intro/?lang=en
- https://webdriver.io/docs/gettingstarted
- https://appium.io/docs/en/drivers/android-uiautomator2/
- https://appium.io/docs/en/drivers/ios-xcuitest/index.html
- http://93days.me/appium-test-automation-with-jest-and-webdriver-io/
- http://93days.me/testing-react-native-application/
- https://blog.codemagic.io/mobile-testing-appium-react-native-apps/
- https://blog.logrocket.com/testing-your-react-native-app-with-appium/

I was also inspired by a few repos and how they dealt with similar configs:

- https://github.com/hadnazzar/react-native-appium/blob/master/e2e/__tests__/appium-test-wdio.js.old
- https://github.com/garthenweb/react-native-e2etest/blob/master/package.json
- https://github.com/microsoft/react-native-windows/blob/main/docs/e2e-testing.md
- https://github.com/microsoft/fluentui-react-native/tree/main/apps/fluent-tester/src/E2E#e2e-testing-overview

## Contributing

Contributions are more than welcome! _(as already mentioned, code is janky and could use a bit more polish)_

Check [CONTRIBUTING](./CONTRIBUTING.md) for more.

Thanks to [@MadeInFrance](https://github.com/MadeinFrance) for his help.

</details>


## License & CoC

This repo is [MIT licensed](./LICENSE) and follows the [Contributor Covenant Code of Conduct](./CODE_OF_CONDUCT.md).
