# YKitIOS for Native

## Get Started

YKit SDK for iOS is the most simple way to intergrate user and payment to YGame system.YKit SDK provide solution for payment such as: SMS, card, internet banking và Apple Payment.

## Steps to integrate SDK

    1. Setup YKit SDK
    2. Config SDK - Payment function
    3. YKit SDK flow

## Note
   - Make sure our project's deployment target is 8.0 at least.

### 1. Setup YKit SDK 
#### 1.1. Import YKit.framework into project

   - Drag and drop YKit.framework into your project.
   - Tick on checkbox: “Copy items into destination group's folder (if needed)”.
   - Embedded Binaries with SDK

![alt tag](https://github.com/shintegu/wiki-ios/blob/master/Images/addEmbed.png)

#### 1.2. Add url schemes

   - Add the following url schemes for Facebook(“fb” + facebook app id) and Google sign in (Reverse client id) from YKitConfig.plist file
    
![alt tag](https://github.com/shintegu/wiki-ios/blob/master/Images/AddUrlScheme.png)
    
   - Go to ios info.plist
   - Add  facebook app id, facebook display name and application queries scheme as below. Please replace app id and display name with the value in the YKitConfig.plist file 
   
![alt tag](https://github.com/shintegu/wiki-ios/blob/master/Images/addFacebookID.png)

   - Add file YKitConfig.plist to your root project

### 1.3. Config exception domain

   - Go to ios info.plist
   - Add setting like image below (There is code of the setting below. It can be copied and pasted quickly into info.plist)
   
![alt tag](https://github.com/shintegu/wiki-ios/blob/master/Images/settingDomain.png)


```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>NSExceptionDomains</key>
	<dict>
		<key>api.ygame.vn</key>
		<dict>
			<key>NSIncludesSubdomains</key>
			<true/>
			<key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
			<true/>
			<key>NSTemporaryExceptionMinimumTLSVersion</key>
			<string>TLSv1.1</string>
		</dict>
	</dict>
	<key>NSAllowsArbitraryLoads</key>
	<true/>
</dict>
</plist>
```

#### 1.4. Setup Code

- Import SDK : #import <YKit/YK.h> in Cocos AppController.m

- Add these lines of code in Application didFinishLaunchingWithOptions function in AppController class, after window setup. You can get Google Signin client ID in the YKitConfig.plist.

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Project configure
    
    
    // Start YKIT CONFIGURING
    YKit *launcher = [YKit getInstance];
        
    launcher.isPotrait = NO;
    
    //
    // if using facebook API, you need to implement 
    // [launcher setPermissionFacebook:@"public_profile"]; // string is the permission you want to 
    //
    launcher setupWithWindow:window usingFacebookSDK:YES];
    [launcher setPermissionFacebook:@"public_profile"]
    
    // Handle login callback
    [launcher handleLoginWithCompletion:^(NSDictionary *data) {
         [launcher getFacebookInfo];
         NSString *userID = data[kParamUserID];
         NSString *userName = data[kParamUserName];
         NSString *accessToken = data[kParamAccessToken];
         //  NSLog(@"sample %@" ,[launcher getFacebookInfo]);
         [launcher showButtonLauncherWithAnimation:YES];
    }];
        
    // Handle logout callback
    [launcher handleLogoutWithCompletion:^{
            //do something
    }];
    
    // handle payment callback
    [launcher handlePaymentWithCompletion:^(NSString *data){
         NSLog(@"Payment success! %@", data);
    }];
        
    //    NSLog(@"sample %@" ,[launcher getFacebookInfo]);
    [launcher setDomainDebug:YES];
    if ([launcher silentLogin]) {
            
    }
        
    [launcher handleShowSDKCompletion:^{
        NSLog(@"kign");
    }];
    NSString* appID = [[[NSBundle mainBundle] infoDictionary] objectForKey:@"CFBundleIdentifier"];
    [launcher handleCloseSDKCompletion:^{
        NSLog(@"queee");
    }];
        
    NSDictionary *dict = @{kParamApplication: ATNonNilObject(application),
                           kParamOptions: ATNonNilObject(launchOptions)};
    ATDispatchEvent(Event_AppDidFinishLaunching, dict);    
    // END YKIT CONFIGURING
    
    return YES;
}

```
- In the previous code, we provide two callback functions. There are handleLoginWithCompletion and handleLogoutWithCompletion. You may use these functions to call login or logout with your server
            
- Add function handle Facebook schemes 

```
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)
            sourceApplication annotation:(id)annotation { 

            NSDictionary *dict = @{kParamApplication: ATNonNilObject(application), kParamUrl: ATNonNilObject(url), 
            kParamSourceApplication: ATNonNilObject(sourceApplication), kParamAnnotation: ATNonNilObject(annotation)}; 

            ATDispatchEvent(Event_AppOpenUrl, dict); 

            return YES; 
}
```

- This example code is apply for landscape mode.You can set it through "didFinishLaunchingWithOptions"

		[YKit getInstance].isPotrait = YES (Potrait only);
		// else landscape is NO;			
- Base on your game orientation, if your game support both portrait and landscape then you must replace UIInterfaceOrientationMaskLandscape with UIInterfaceOrientationMaskAll, if you game is only support portrait mode, then you don’t need to add this function.
	
```
- (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window
            { 

            if ([[YKit getInstance] isScreenRotateToPortrait]) { 

            return UIInterfaceOrientationMaskPortrait; 

            } else 	
            
            return UIInterfaceOrientationMaskLandscape; 
            
            } 
```

#### 1.5. Public functions
- Here is the list of public functions you can call to customize the YKit in your game: 

* setLauncherStickySide: You can specific the side that launcher can stick to via the or bitwise. 
Ex: ATButtonStickySideTop | ATButtonStickySideBottom 

* silentLogin: When open the app, maybe user is already logged in. Call this function to check if user is logged in or not, if not, you must call showLoginScreen function to show the login screen. 

```
if([[YKit getInstance] silentLogin])
	// Move direct to game
else {
	// Show login screen
	[[YKit getInstance] showLoginScreen];
}
```
        
* showButtonLauncherWithAnimation 
* hideButtonLauncherWithAnimation
* showLoginScreen: Show the login screen, if user not logged in yet
* showPaymentScreen: You may want to show payment screen from your game
* handleShowSDKCompletion: You can get the event show SDK here
* handleCloseSDKCompletion: You can get the event show SDK here
### 2. Implement payment extra data

Payment extra data (PED) is the data you send to game server when user make payment success. 
For example: if your game have multiple servers or multiple characters, you may want to send this data to game server, so its will know which character get the gold. The format is defined on your demand. 
    
Note*: 
* It must be unique string
* Maximum is 50 characters
* There's no special character in string

    
To implement PED, you create a class that implement PaymentExtraDataProtocol with function
```
- (NSString *)getPaymentExtraData { 

	return @“server12-id1001”; // ex: server 12, userid = 1001
}
```
Then you create an PaymentExtraDataImp object and set it to YKit

```
[launcher setPaymentExtraDataObject:[PaymentExtraDataImp new]];
```
    
### 3. Flow

#### 3.1. Login flow: 
![alt tag](https://github.com/shintegu/wiki-ios/blob/master/Images/loginFlow.png)

#### 3.2. Payment flow:
![alt tag](https://github.com/shintegu/wiki-ios/blob/master/Images/PaymentFlow.png)

### 4. Build note
Please input full information in Xcode before build the product
- Display Name: name appear on the device
- Bundle identifier: bundle id of your game which provided by XCT
- Version: string, for example: 1.0.0
- Build: number, for example: 100

![alt tag](https://github.com/shintegu/wiki-ios/blob/master/Images/identity.png)
