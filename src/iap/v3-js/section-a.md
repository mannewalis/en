### 2.5 `Cocos2dxActivity.java`
*   `Cocos2dxActivity.java` is located:

    ```
    /frameworks/js-bindings/cocos2d-x/cocos/platform/android/java/src/org/
    cocos2dx/lib/Cocos2dxActivity.java
    ```

* Modify `Cocos2dxActivity.java` to add the following imports:
```java
import android.content.Intent;
import com.sdkbox.plugin.SDKBox;
```

* Second, modify `Cocos2dxActivity` to edit the `onCreate(final Bundle savedInstanceState)` function to add a call to `SDKBox.init(this);`. The placement of this call is important. It must be done after the call to `onLoadNativeLibraries();`. Example:
```java
onLoadNativeLibraries();
SDKBox.init(this);
```

* Last, we need to insert the proper __overrides__ code. There are a few rules here.
    * If the method listed has not been defined, __add it__.

    * If the method listed has been defined, add the calls to `SDKBox` in the __same__ existing function.
```java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
          if(!SDKBox.onActivityResult(requestCode, resultCode, data)) {
            super.onActivityResult(requestCode, resultCode, data);
          }
    }
    @Override
    protected void onStart() {
          super.onStart();
          SDKBox.onStart();
    }
    @Override
    protected void onStop() {
          super.onStop();
          SDKBox.onStop();
    }
    @Override
    protected void onResume() {
          super.onResume();
          SDKBox.onResume();
    }
    @Override
    protected void onPause() {
          super.onPause();
          SDKBox.onPause();
    }
    @Override
    public void onBackPressed() {
          if(!SDKBox.onBackPressed()) {
            super.onBackPressed();
          }
    }
```

## Step 3: Modify your game code to invoke in-app purchase

### 3.1 Config `sdkbox_config.json`
* First time `SDKBOX` user:
      * Rename the included `sdkbox_config.json.sample` to `sdkbox_config.json`
      * Copy `sdkbox_config.json` to the __Resources__ directory of your project  
      * Then Drag & drop `sdkbox_config.json` from your __Resources__ folder to Xcode’s Resources folder this will make sure `sdkbox_config.json` is in your bundle

* If you already have an `sdkbox_config.json` file in your project:
      * Insert the __iap__ section of `sdkbox_config.json.sample` into your
      existing `sdkbox_config.json` and change the config as you see fit

### 3.2 Initialize IAP
* Call `sdkbox.IAP.init();` where appropriate in your code. We
recommend to do this in the `app.js`

* Modify `AppDelegate.cpp` to include the following headers:
```cpp
#include "PluginIAPJS.hpp"
#include "PluginIAPJSHelper.hpp"
```

* Modify `AppDelegate.cpp` make sure to call:
```cpp
sc->addRegisterCallback(register_all_PluginIAPJS);
sc->addRegisterCallback(register_all_PluginIAPJS_helper);
```
This registers the Javascript callbacks.

### 3.3 Retrieve latest Product data
It's always a good idea to retrieve the latest product data from store when your game starts.

To retrieve latest IAP data, simply call `sdkbox.IAP.refresh()`.

> `onProductRequestSuccess` if retrieved successfully.

> `onProductRequestFailure` if exception occurs.

### 3.4 Make a purchase
To make a purchase call `sdkbox.IAP.purchase(name)`

__Note:__ __name__ is the name of the IAP item in your config file under `items` tag, not the product id you set in iTunes or GooglePlay Store

> `onSuccess` will be triggered if purchase is successful.

> `onFailure` will be triggered if purchase fails.

> `onCanceled` will be triggered if purchase is canceled by user.

### 3.5 Restore purchase
To restore purchase call `sdkbox.IAP.restore()`.

> `onRestored` will be triggered if restore is successful.

__Note:__ `onRestored` could be triggered multiple times

### 3.6 Handling Purchase Events
This allows you to catch the `IAP` events so that you can perform operations based upon the response from your players and IAP servers.
```Javascript
sdkbox.IAP.setListener({
    onSuccess : function (product) {
        //Purchase success
    },
    onFailure : function (product, msg) {
        //Purchase failed
        //msg is the error message
    },
    onCanceled : function (product) {
        //Purchase was canceled by user
    },
    onRestored : function (product) {
        //Purchase restored
    },
    onProductRequestSuccess : function (products) {
        //Returns you the data for all the iap products
        //You can get each item using following method
        for (var i = 0; i < products.length; i++) {
            // loop
        }
    },
    onProductRequestFailure : function (msg) {
        //When product refresh request fails.
    }
});
```