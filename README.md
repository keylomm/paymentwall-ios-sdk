INTRODUCTION
------------
Do you want to accept payments from mobile users in different countries, different payment system by writing just few of code lines? 
Paymentwall SDK is a global mobile payment gateway that accepts payments from more than 200 countries with 100+ alternative payment options. We now provide SDK for iOS which will become a native part of your application, it eliminates the necessity to open a web browser for payments. Less steps, faster process, there’s no doubt your conversion rate will get boost! All you have to do is import the library into your project and config it to start accepting in-app payment. It is quick and easy! We'll guide you through the process here.

HOW DOES IT WORK?
-----------------
1. Add the framework to your project. 
With different areas, we provide corresponding external payment system framework files. You can add as many as you want. You can also enable/disable default payment options too. You can add any payment option as they want by importing the payment system framework and framework plugin provided by Paymentwall to your project
2. User requests a purchase inside your application.
3. Paymentwall SDK initializes payment screen with 3 core payment options (Brick, MINT, Mobiamo) and the other is “Local Payments” option.
4. User initiates payment in-app 
With Brick, Mint, Mobiamo the payment process will totally be native.
With local payments, local payment screen will be shown with payment methods corresponding to user’s current location. Here users can then select a payment option they prefer.

REQUIREMENTS
------------
XCode 8.0+, iOS 8.0+

CREDENTIALS
-----------
SDK integration requires a project key. Obtain these Paymentwall API credentials in the application settings of your Merchant Account at [Paymentwall.com](http://paymentwall.com/)

Available native payment option plugins
------------------------------
- [Alipay](https://github.com/paymentwall/paymentwall-ios-sdk/tree/master/Plugins/PWAlipayPlugin)
- [Wechatpay](https://github.com/paymentwall/paymentwall-ios-sdk/tree/master/Plugins/PWWechatpayPlugin)
- [MyCard](https://github.com/paymentwall/paymentwall-ios-sdk/tree/master/Plugins/PWMycardPlugin)

Available extra plugins
------------------------------
- [Card scanner](https://github.com/paymentwall/paymentwall-ios-sdk/tree/master/Plugins/PWCardScannerPlugin)
- [Customization](https://github.com/paymentwall/paymentwall-ios-sdk/tree/master/Plugins/PWCustomizationPlugin)

Setup (Paymentwall Core SDK)
---------------------
Our SDK is delivered as static library or public repository.

### Install with Cocoapods:

1. Install CocoaPods 0.39.0 or later
2. In your Podfile, add `pod 'PWCoreSDK'` and `pod 'PW[Local payment method]Plugin'`(optional) to your main and test targets
3. From the command line, run `pod install`
4. Use the .xcworkspace file generated by CocoaPods
5. Add `-ObjC` to `Other Linker Flags` in your project's Build settings
6. If you are using Swift, create a new Objective-C file in order to create new `bridging-header.h` files, then import the following files into the header file like:
```objective-c
#import <PWCoreSDK/PWCoreSDK.h> 
```

### Install as Static library:

1. Download the latest release of the SDK and extract the zip.
2. Go to your Xcode project’s “General” settings. Drag libPWCoreSDK.a and other plugin library to your project directory to the “Linked frameworks and libraries” section. Make sure Copy items if needed is selected and click Finish.
3. In your unit test target’s “Build Settings”, add the parent path to libPWCoreSDK.a in the “Library Search Paths” section.
4. Drag the bundle into your "Copy bundle resources" in project's "Build phase"
5. Add `-ObjC` to `Other Linker Flags` in your project's Build settings
6. If you are using Swift, create a new Objective-C file in order to create new `bridging-header.h` files, then import the following files into the header file like:
```objective-c
#import <PWCoreSDK/PWCoreSDK.h> 
```

>Update: If your CoreSDK got update during install any plugin, you should also check to update the rest of plugins.

Create your first payment
-------------------------
1. Setup the SDK with the following params, only need to setup (step 1-7) once:
```swift
PWCoreSDK.sharedInstance().setupPaymentwall(withProjectKey: "YOUR PUBLIC KEY", secretKey: "YOUR SECRET KEY", requestTimeout: 30, clearPaymentMethodsAfterFinish: false)
```
>Optional: Extra settings can be found in the Extra settings section below

>Project key: All payment option will use this Project key if their Project key set to nil, you also can specify their own Project key

>Secret key: PWLocal and local payment options plugins will use this Secret key as default if you specify it here

2. Implement `PWCoreSDKDelegate` protocol to handle payment response:
```swift
func paymentResponse(_ response: PWCoreSDKResponse?) {
    guard let response = response else { return }
    switch response.responseCode {
        case .SUCCESSFUL:
        case .FAILED:
        case .CANCEL:
        case .MERCHANT_PROCESSING:
        //Example when you use Brick and `useNativeFinishDialog == true` can be found below
    }

    switch response.paymentType {
        case .NONE:
        case .MINT:
        case .PWLOCAL:
        case .BRICK:
        case .MOBIAMO:
        case .OTHERS:
    }
}
```

3. Add Brick payment type, Brick is Paymentwall's credit card solution available for businesses in 200+ countries, please refer below on how to handle Brick payment flow, cardScannerPlugin is distributed as a plugin and is optional:

```swift
PWCoreSDK.sharedInstance().addBrickPayment(withPublicKey: nil, useNativeFinishDialog: true, cardScannerPlugin: PWCardScannerPlugin.sharedInstance())
```

4. Add Mobiamo payment type, Mobiamo is a mobile carrier billing platform with global coverage via SMSs, `noPrice` specify if you want to use the default Mobiamo price for each country:

```swift
PWCoreSDK.sharedInstance().addMobiamoPayment(withAppID: nil, noPrice: true)
```

5. Add Mint payment type, MINT is Paymentwall's cash payment solution designed to help monetize your digital goods and services all over the world via MINT prepaid card:
```swift
PWCoreSDK.sharedInstance().addMintPayment(withAppID: nil)
```
6. Add PWLocal payment type, `type` can be VIRTUAL_CURRENCY / DIGITAL_GOODS_FLEXIBLE / DIGITAL_GOODS_DEFAULT / CART:
```swift
PWCoreSDK.sharedInstance().addPWLocalPayment(with: .DIGITAL_GOODS_FLEXIBLE, secretKey: nil)
```

7. Add any other local payment option plugin if you want, please refer Implement local payment options section below on how to create them:

```swift
PWCoreSDK.sharedInstance().addCustomPaymentOptions([alipay, unionpay, ...])
```
>Note: These payment options will be place in `Local payments` together with PWLocal.

8. Create new payment with `PaymentObject` class and assign to the CoreSDK:
```swift
let payment = PaymentObject()
payment.name = choosenItem.name
payment.price = Double(choosenItem.price)!
payment.currency = "USD"
payment.image = choosenItem.image
payment.userID = "test_user"
payment.itemID = choosenItem.name+"id"
payment.signVersion = 3

let customSetting = ["widget":"pw",
                    "ag_type":"fixed"]
payment.pwLocalParams = customSetting

PWCoreSDK.sharedInstance().setPaymentObject(payment)
```

>Note: pwLocalParams can be Dictionary or any of the defined class: `CartDefaultWidget`, `DigitalGoodsDefaultWidget`, `DigitalGoodsFlexibleWidget`, `VirtualCurrency`, refer their headers for required property or [PWLocal docs](https://github.com/paymentwall/paymentwall-pwlocal-ios). Key and value like prices, amount, currencyCode, currencies, ag_name, ag_external_id, uid will be ignored and use the one you described in `PaymentObject`

9. Present Payment options view controller:

```swift
PWCoreSDK.showPaymentOptionsViewController(withParentViewcontroller: self, delegate: self, showCompletion: nil)
```

Extra settings
-------------------
- `clearPaymentMethodsAfterFinish`: set to true if you wish to remove already added payment system after the SDK view controller is closed, you will have to add the payment system again, but the UI setting will be kept.
- Default UI of the SDK is flat style, to use game UI, add this to your code: `PWCoreSDK.sharedInstance().setUseGameUI(true)`
- If you wish to customize the flat UI to suit your app, you can get the [PWCustomizationPlugin](https://github.com/paymentwall/paymentwall-ios-sdk/tree/master/Plugins/PWCustomizationPlugin) and set it while setup the SDK.
- By default (except Brick), all payments method will show success once the payment is finish, if you don't want to use the SDK's success dialog but return delegate for you to work with, add this to your code: `PWCoreSDK.sharedInstance().setUseNativeFinishDialogForAllMethods(false)`
- To show bank processor adress for Brick, use `PWCoreSDK.sharedInstance().setShowBrickFooter(true)`
- To use custom icon for local payments, use `PWCoreSDK.sharedInstance().setCustomLocalPaymentImage(image)`

Brick payment flow
-------------------
1. After the token is successfully fetched, the `response: PWCoreSDKResponse` will contain the `token: BrickToken` along with it
2. Send request to your server to handle the token: 

- If `useNativeFinishDialog` is set to `false`, the SDK will dismiss after token is successfully fetched, the `response.responseCode` will also be `.MERCHANT_PROCESSING`, you will have to handle success/failed/3D secure yourself and store card feature wont be available

- If `useNativeFinishDialog` is set to `true`, the loading popup will keep showing, after you process the token in your sever, post a `Notification` to use the SDK success or failed dialog, also the SDK will call delegate again to return `.SUCCESS` or `.FAILED`:

```swift
if response.paymentType == .BRICK {
    if let returnedToken = response.token {
        //Process the token with your server here asynchronous
        //When done:
        NotificationCenter.default.post(name: Notification.Name(BRICK_TOKEN_PROCESSED_FINISH), object: nil, userInfo: nil)
    }
}
```
- Pass the error in Dictionary as `["error": errorMessage]` via `userInfo`, the SDK will automatically show failed dialog instead of successful dialog

- If 3D secure happen, we also support 3D secure, pass the URL in Dictionary as `["secure": urlString]` via `userInfo`, the SDK will handle showing the 3D secure and pass `SUCCESS/FAILED` back to app after user finish enter secure info

- If you want to enable store card feature for user, pass the Charge object to our SDK via `userInfo`, Charge object format can be found in our [Brick docs](https://www.paymentwall.com/en/documentation/Brick/2968)

Custom PWLocal signature
------------------------
If you wish to provide your PWLocal signature manually:

```swift
//Define all params in step 8., and extra custom settings, then get `stringToSign`:

let payment = PaymentObject()
payment.name = choosenItem.name
payment.price = Double(choosenItem.price)!
payment.currency = "USD"
payment.image = choosenItem.image
payment.userID = "test_user"
payment.itemID = choosenItem.name+"id"
payment.signVersion = 3
let customSetting = ["widget":"pw",
                    "ag_type":"fixed"]

let strToSign = PWCoreSDK.sharedInstance().getStringToSign(customSetting, paymentObject: payment)

//Calculate your sign and set it
customSetting["sign"] = sha256(text: "\(strToSign!)YOUR SECRET KEY")
payment.pwLocalParams = customSetting
```

Implement local payment option
------------------------------
Paymentwall SDK supports external payment system injection (which are in our defined payment system (PS) list). Each time you want to add a new payment system, you have to include it's native SDK into your project along with our plugin framework, our framework will handle creating all the necessary parameters then you can use them to show the native local payment SDK:

1. Add the plugin with Cocoapods with `pod 'PW[Local payment method]Plugin'` or manually dragging the `PW[Local payment method]Plugin.a` and it's headers file to your project
2. Import the library header into your project or via `bridging-headers.h` if you use Swift 
3. Setup the plugin, each plugin have different requirements, details can be found in their headers or detailed docs below.

>Note: All plugins support your own signature string if you don't specify Secret key in both CoreSDK and PluginSDK, use `plugin.getStringToSign()` to get the string to sign, then add your signed string to `plugin.signString`
