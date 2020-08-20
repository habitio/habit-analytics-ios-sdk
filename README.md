# **HabitAnalytics**

HabitAnalytics is an SDK that easily helps you gather analytics information. 

To integrate with our SDK follow the steps described here and if you have any questions don't hesitate to contact us at support@habit.io.

## **[Cocoapods](https://cocoapods.org)**

To use the HabitAnalytics SDK make sure you have the latest version of Cocoapods installed by running:

```
gem install cocoapods
```

Update your local specs (if necessary):
```
pod update
```

Add the following line to your Podfile:

```
pod 'HabitAnalytics'
```

And then run the command on your project folder:
```
pod install
```


## Enable Background Modes

Enable *Background Modes* for the project in the Capabilities Tab as seen in the image. Make sure you check **Location updates** and **Background fetch**.

![Capabilities](capabilities.png "Enable Capabilities")


## Update .plist file

As required by Apple, it's necessary to add the following permissions' descriptions to the project .plist file. *Warning: App submission will fail if they are not provided.*

* NSLocationAlwaysUsageDescription
* NSLocationAlwaysAndWhenInUseUsageDescription
* NSLocationWhenInUseUsageDescription
* NSLocationUsageDescription
* NSBluetoothPeripheralUsageDescription
* NSBluetoothAlwaysUsageDescription
* NSMotionUsageDescription

The best practice is to notify the user beforehand, providing an incentive to approve the permissions. Once the SDK is *initialized* with the user authorization info, or the *setAuthorization* method is called, two alerts will be shown, asking the user to grant access to the location services and to the device's motion information.



### Import HabitAnalytics

To use HabitAnalytics in your source files make sure to include:
```
import HabitAnalytics
```


## Initialization

You need to initialize the HabitAnalytics SDK on your AppDelegate class.

The initialization must be done in **didFinishLaunchingWithOptions**:
```
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    HabitAnalytics.shared.initialize(namespace: String, analyticsInfo: NSDictionary?, authInfo: NSDictionary?, enabledCapabilities: [String]) { (statusCode) in
    }
}
```

Only the **namespace** of the application is mandatory in the initialization. The application namespace can be found in the [Selfcare Back Office](https://selfcare.habit.io)

### **Authorization info**

The **authInfo** parameter is the authorization object that was obtained when the user sign in was performed in our platform. Make sure to check the [Authorization Documentation](https://github.com/habitio/habit-analytics-ios-sdk/blob/master/Authorization.md) for more details.

****Important*:** If the initialization of the SDK is done before a user signs in, then the authInfo parameter must be nil and should be set as soon as the user is signed in using the **setAuthorization** method explained ahead.*

### **Analytics Info**

The parameter **analyticsInfo** is a parameter that allows some optional customizations but as default it should be *nil*. For more information please contact support@habit.io

### **Enabled Capabilities**


The parameter **enabledCapabilities** allows customizing which capabilities are to be enabled in the SDK. By default all capabilities are enabled.
The parameter is an array with the supported capabilities from the following list:

* HabitAnalyticsSupportedCapabilites.AnalyticsTracker
* HabitAnalyticsSupportedCapabilites.Bluetooth
* HabitAnalyticsSupportedCapabilites.Location
* HabitAnalyticsSupportedCapabilites.Motion



### **Status Codes**

The initialization returns a *status code* which can be used to identify if the initialization was successful or if an error occurred. 

Some status codes examples are: 
* 5220 - User initialization successful
* 5221 - HabitSDK initialized with success
* 5222 - HabitSDK initialized but Authentication must be set
* 5402 - A user has already been authenticated. You need to logout.
* 5403 - User initialization error
* 5404 - Error initializing HabitSDK
* 5420 - Invalid Auth Info
* 5406 - User not set because HabitSDK has not been initialized. Call Initialize method
* 5450 - User refresh token is invalid. Please set user with a valid authInfo object

To get a description of a status code you can call the method: 
```
HabitStatusCodes.getDescription(code: statusCode))
```


## Set Authorization

The *setAuthorization* method is called once a user has been successfully signed in our platform. 

```
HabitAnalytics.shared.setAuthorization(authInfo: NSDictionary, completion: { (statusCode) in
    
    if statusCode == HabitStatusCodes.USER_INITIALIZATION_SUCCESS {
    // Success
    }
    else { 
    // Handle according to status code 
    }

})
```

If **setAuthorization** is called before *HabitAnalytics* is initialized then it will return a *HabitStatusCodes.HABIT_SDK_NOT_INITIALIZED*. If this happens you must proceed to initialize the SDK.

Also, if the *authInfo* doesn't match the previous user, a status code *HabitStatusCodes.ANOTHER_USER_ALREADY_AUTHENTICATED_LOGOUT* is received. 


### **Background fetch**

Using the background app refresh mechanism, the OS allows some minimal work to be done from time to time. Ensure you have this code on **didFinishLaunchingWithOptions** to set the minimum background fetch interval:

```
UIApplication.shared.setMinimumBackgroundFetchInterval(TimeInterval(1800))
```

Also add the following method that is executed when background fetch is triggered:

```
func application(_ application: UIApplication, performFetchWithCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {

    HabitAnalytics.shared.handleBGFetch { (result) in
    completionHandler(result)
    }
}
```

### **Push Notifications**

After registering for push notifications ensure that you call the SDK push notifications handler every time you receive a new push notification.

```
func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any], fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {

    HabitAnalytics.shared.handlePushNotification(userInfo: userInfo) { (result) in
         completionHandler(result)
    }
}
```

## **Tracking events**

Using Habit Analytics SDK you can track events in your app. They can be used to gather information on UX experience, application flow or any other scenario you desire.
To track an event all you need to do is call the **track** method like this:

```
HabitAnalytics.shared.track(eventName: String, properties: [String: SupportedType]?);
```

*Event properties are **optional** and currently the supported types are the following:*


* String
```
HabitAnalytics.shared.track(eventName: 'User sign in', properties: {'type':'email'});
```
* Numeric
```
HabitAnalytics.shared.track(eventName: 'Application started', properties: {'Loading Time':0.9 });
```
* Boolean
```
HabitAnalytics.shared.track(eventName: 'User sign in', properties: {'First time': true });
```

* Date
```
HabitAnalytics.shared.track(eventName: 'User sign in', properties: {'Date': Date() });
```

* URL
```
HabitAnalytics.shared.track(eventName: 'Clicked URL', properties: {'url': URL });
```

* List
```
HabitAnalytics.shared.track(eventName: 'Favorite Cities', properties: {'Places selected': ["Lisbon", "New York", "London"] });
```


## **HabitAnalyticsDelegate**

For changes in the SDK status, make sure to implement the HabitAnalyticsDelegate.

The delegate will be triggered if a user access token has expired and a new one must be provided. You must then refresh the user token and *setAuthorization* with the new result. 

Check the [Authorization Documentation](https://github.com/habitio/habit-analytics-ios-sdk/blob/master/Authorization.md) for more details on how to refresh a token.

```
func HabitAnalyticsStatusChange(statusCode: HabitStatusCode) {
    switch statusCode {
    case HabitStatusCodes.SET_VALID_AUTHENTICATION_INFO:

    // Obtain a new authInfo by refreshing the token.
    HabitAnalytics.shared.setAuthorization(authInfo: newValidAuthInfo) { (statusCode) in
    }
    break

    default:
    debugPrint("HabitAnalytics: " + String(statusCode) + " -> " + HabitStatusCodes.getDescription(code: statusCode))
    }
}
```

## **Logout**

To logout a user, just call the corresponding method. *Make sure you call logout when switching users.*
```
HabitAnalytics.shared.logout(completion: { (code) in
    debugPrint(String(code) + " : " + HabitStatusCodes.getDescription(code: code))
    })
})
```

## Built With
Xcode 11.5 - IDE


## Minimum supported version
iOS 11


## Analysed Data 

| Property  | Received on event (Background) | Received on event (Foreground) | Query by snapshot | Permission requested to the user | Frequency |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| network/current_wifi | NO	| YES | YES	| Not necessary | application start, location change update, background fetch |
| network/carrier | NO | NO	| YES | Not necessary | application start, location change update, background fetch |
| network/internet_connection | NO | YES | YES | Not necessary | application start, location change update, background fetch |
| network/scan | NO | NO | NO | NA | Not possible due to OS limitations |
| network/roaming | NO | NO | NO | NA | Not possible due to OS limitations |
| network/flight_mode | NO | NO | NO | NA | Not possible due to OS limitations |
| network/mobile_data_enabled | NO | NO	| NO | NA | Not possible due to OS limitations |
| location/location	| YES | YES | YES | YES | application start, location change update, background fetch |
| location/visit | YES | YES | NO | YES | application start, location change update, background fetch |
| movement/activity	| NO | YES | NO | YES | application start, location change update, background fetch |
| movement/activity_history	| NO | NO | YES | YES | application start, location change update, background fetch |
| battery/charge | NO | NO | YES | Not necessary | application start, location change update, background fetch |
| battery/level | NO | NO | YES | Not necessary | application start, location change update, background fetch |
| devices/bluetooth | NO | NO | YES | Permission text is added but the OS is not requesting yet. May change in future | application start, location change update, background fetch |
| current_device/agent | NO | NO | YES | Not necessary | application start, location change update, background fetch
| current_device/installed_packages | NO | NO | NO | NA | Not possible due to OS limitations |
| current_device/added_package | NO | NO | NO | NA | Not possible due to OS limitations |
| current_device/removed_package | NO | NO | NO | NA | Not possible due to OS limitations |

## Future work

- Configuration of which data is supposed to be analysed by the SDK.


## Versioning
For the versions available, see the [tags on this repository](https://github.com/habitio/habit-analytics-ios-sdk/tags). 


## Contributors
See the list of [contributors](https://github.com/habitio/habit-analytics-ios-sdk/contributors) who participated in this project.


## Contact Us

For more information contact us at support@habit.io  

