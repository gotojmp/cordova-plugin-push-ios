#cordova-plugin-push-ios

> Register and receive push notifications for ios
> 由于天朝网络原因，删除了Android模块，只保留IOS部分。

## Installation

This requires phonegap/cordova CLI 5.0+ ( current stable v1.4.2 )

```
phonegap plugin add https://github.com/gotojmp/cordova-plugin-push-ios.git
```

or 

```
cordova plugin add https://github.com/gotojmp/cordova-plugin-push-ios.git
```

## Supported Platforms

- iOS

## Quick Example

```javascript
    var push = PushNotification.init({ "ios": {"alert": "true", "badge": "true", "sound": "true"} } );

    push.on('registration', function(data) {
        // data.registrationId
    });

    push.on('notification', function(data) {
        // data.message,
        // data.title,
        // data.count,
        // data.sound,
        // data.image,
        // data.additionalData
    });

    push.on('error', function(e) {
        // e.message
    });
```

## API

### PushNotification.init(options)

Parameter | Description
--------- | ------------
`options` | `JSON Object` platform specific initialization options.
`options.ios` | `JSON Object` iOS specific initialization options.
`options.ios.alert` | `Boolean`\|`String` Optional. If `true`\|`"true"` the device shows an alert on receipt of notification. Default is `false`\|`"false"`. **Note:** the value you set this option to the first time you call the init method will be how the application always acts. Once this is set programmatically in the init method it can only be changed manually by the user in Settings>Notifications>`App Name`. This is normal iOS behaviour.
`options.ios.badge` | `Boolean`\|`String` Optional. If `true`\|`"true"` the device sets the badge number on receipt of notification. Default is `false`\|`"false"`. **Note:** the value you set this option to the first time you call the init method will be how the application always acts. Once this is set programmatically in the init method it can only be changed manually by the user in Settings>Notifications>`App Name`. This is normal iOS behaviour.
`options.ios.sound` | `Boolean`\|`String` Optional. If `true`\|`"true"` the device plays a sound on receipt of notification. Default is `false`\|`"false"`. **Note:** the value you set this option to the first time you call the init method will be how the application always acts. Once this is set programmatically in the init method it can only be changed manually by the user in Settings>Notifications>`App Name`. This is normal iOS behaviour.
`options.ios.clearBadge` | `Boolean`\|`String` Optional. If `true`\|`"true"` the badge will be cleared on app startup. Default is `false`\|`"false"`.

#### Returns

- Instance of `PushNotification`.

#### Example

```javascript
    var push = PushNotification.init({ "ios": {"alert": "true", "badge": "true", "sound": "true"} } );
```

### push.on(event, callback)

Parameter | Description
--------- | ------------
`event` | `String` Name of the event to listen to. See below for all the event names.
`callback` | `Function` is called when the event is triggered.

### push.on('registration', callback)

The event `registration` will be triggered on each successful registration with the 3rd party push service.

Callback Parameter | Description
------------------ | -----------
`data.registrationId` | `String` The registration ID provided by the 3rd party remote push service.

#### Example

```javascript
push.on('registration', function(data) {
    // data.registrationId
});
```

### push.on('notification', callback)

The event `notification` will be triggered each time a push notification is received by a 3rd party push service on the device.

Callback Parameter | Description
------------------ | -----------
`data.message` | `String` The text of the push message sent from the 3rd party service.
`data.title` | `String` The optional title of the push message sent from the 3rd party service.
`data.count` | `String` The number of messages to be displayed in the badge iOS or message count in the notification shade in Android. For windows, it represents the value in the badge notification which could be a number or a status glyph.
`data.sound` | `String` The name of the sound file to be played upon receipt of the notification.
`data.image` | `String` The path of the image file to be displayed in the notification.
`data.additionalData` | `JSON Object` An optional collection of data sent by the 3rd party push service that does not fit in the above properties.
`data.additionalData.foreground` | `Boolean` Whether the notification was received while the app was in the foreground

#### Example

```javascript
    push.on('notification', function(data) {
        // data.message,
        // data.title,
        // data.count,
        // data.sound,
        // data.image,
        // data.additionalData
    });
```

### push.on('error', callback)

The event `error` will trigger when an internal error occurs and the cache is aborted.

Callback Parameter | Description
------------------ | -----------
`e` | `Error` Standard JavaScript error object that describes the error.

#### Example

```javascript
push.on('error', function(e) {
    // e.message
});
```

### push.unregister(successHandler, errorHandler)

The unregister method is used when the application no longer wants to receive push notifications.

#### Example

```javascript
push.unregister(successHandler, errorHandler);
```

### push.setApplicationIconBadgeNumber(successHandler, errorHandler, count) - iOS only

Set the badge count visible when the app is not running

The `count` is an integer indicating what number should show up in the badge. Passing 0 will clear the badge. Each `notification` event contains a `data.count` value which can be used to set the badge to correct number.

#### Example

```javascript
push.setApplicationIconBadgeNumber(successHandler, errorHandler, count);
```

### push.getApplicationIconBadgeNumber(successHandler, errorHandler) - iOS only

Get the current badge count visible when the app is not running

successHandler gets called with an integer which is the current badge count

#### Example

```javascript
push.getApplicationIconBadgeNumber(successHandler, errorHandler);
```

### push.finish(successHandler, errorHandler) - iOS only

Tells the OS that you are done processing a background push notification.

successHandler gets called when background push processing is successfully completed.

#### Example

```javascript
push.finish(successHandler, errorHandler);
```


## iOS Behaviour

### Sound

In order for your your notification to play a custom sound you will need to add the files to root of your iOS project. The files must be in the proper format. See the [Local and Remote Notification Programming Guide](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/IPhoneOSClientImp.html#//apple_ref/doc/uid/TP40008194-CH103-SW6) for more info on proper file formats and how to convert existing sound files.

Then send the follow JSON from APNS:

```javascript
{
    "aps": {
        "alert": "Test sound",
        "sound": "sub.caf"
    }
}
```

### Background Notifications

On iOS if you want your `on('notification')` event handler to be called when your app is in the background you will need to do a few things.

First the JSON you send from APNS will need to include `content-available: 1` to the `aps` object. The `content-available: 1` property in your push message is a signal to iOS to wake up your app and give it up to 30 seconds of background processing. If do not want this type of behaviour just omit `content-available: 1` from your push data.


For instance the following JSON:

```javascript
{
    "aps": {
        "alert": "Test background push",
        "content-available": "1"
    }
}
```

will produce a notification in the notification shade and call your `on('notification')` event handler.

However if you want your `on('notification')` event handler called but no notification to be shown in the shader you would omit the `alert` property and send the following JSON to APNS:

```javascript
{
    "aps": {
        "data": "Test silent background push",
        "moredata": "Do more stuff",
        "content-available": "1"
    }
}
```

That covers what you need to do on the server side to accept background pushes on iOS. However, it is critically important that you continue reading as there will be a change in your `on('notification')`. When you receive a background push on iOS you will be given 30 seconds of time in which to complete a task. If you spend longer than 30 seconds on the task the OS may decide that your app is misbehaving and kill it. In order to signal iOS that your `on('notification')` handler is done you will need to call the new `push.finish()` method. 

For example:

```javascript
        var push = PushNotification.init({
            "ios": {
              "sound": true,
              "vibration": true,
              "badge": true,
              "clearBadge": true
            }
        });
        
        push.on('registration', function(data) {
        	// send data.registrationId to push service
        });
        

        push.on('notification', function(data) {
        	// do something with the push data
        	// then call finish to let the OS know we are done
            push.finish(function() {
                console.log("processing of push data is finished");
            });
        });
```

It is absolutely critical that you call `push.finish()` when you have successfully processed your background push data.
