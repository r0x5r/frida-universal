markdown
Copy code
# Universal Frida Scripts for Security Testing

This repository contains a collection of universal Frida scripts that can be used for security testing of Android applications. These scripts target common security mechanisms and functionalities in Android applications.

## Disclaimer

Bypassing security mechanisms and intercepting traffic should only be done in a lawful manner, with explicit permission from the application owner or within a controlled environment for security testing purposes.

## Scripts

### 1. Bypass Root Detection

This script aims to bypass common root detection checks implemented in Android applications.

```bash
// Bypass Root Detection

Java.perform(function () {
    var RootPackages = ["com.noshufou.android.su", "com.thirdparty.superuser", "eu.chainfire.supersu", "com.koushikdutta.superuser", "com.zachspong.temprootremovejb", "com.ramdroid.appquarantine"];
    var RootBinaries = ["su", "busybox", "supersu"];
    var RootProperties = ["ro.build.selinux", "ro.build.tags"];

    // Bypass package checks
    var PackageManager = Java.use("android.app.ApplicationPackageManager");
    PackageManager.getPackageInfo.overload('java.lang.String', 'int').implementation = function (pkg, flags) {
        if (RootPackages.indexOf(pkg) >= 0) {
            console.log("Bypassing root package check: " + pkg);
            pkg = "com.nonexistent.package";
        }
        return this.getPackageInfo(pkg, flags);
    };

    // Bypass binary checks
    var Runtime = Java.use('java.lang.Runtime');
    Runtime.exec.overload('[Ljava.lang.String;').implementation = function (cmd) {
        if (RootBinaries.indexOf(cmd[0]) >= 0) {
            console.log("Bypassing root binary check: " + cmd[0]);
            cmd[0] = "nonexistentbinary";
        }
        return this.exec(cmd);
    };

    // Bypass property checks
    var SystemProperties = Java.use('android.os.SystemProperties');
    SystemProperties.get.overload('java.lang.String').implementation = function (key) {
        if (RootProperties.indexOf(key) >= 0) {
            console.log("Bypassing root property check: " + key);
            return "false";
        }
        return this.get(key);
    };

    console.log('Root detection bypass script loaded');
});
```


 2. Intercept and Log Function Calls
This script intercepts method calls, logs the arguments passed to the methods, and prints the return values.


```bash
// Intercept and Log Function Calls

Java.perform(function () {
    var targetClass = "com.example.app.TargetClass";  // Replace with the target class name
    var targetMethod = "targetMethod";  // Replace with the target method name

var TargetClass = Java.use(targetClass);
    TargetClass[targetMethod].overload().implementation = function () {
        console.log('Intercepted ' + targetMethod + ' called with arguments: ' + JSON.stringify(arguments));
        var returnValue = this[targetMethod].apply(this, arguments);
        console.log('Return value: ' + returnValue);
        return returnValue;
    };

console.log('Function call interception script loaded');
});
```
# 3. Hook and Modify Method Returns
This script hooks methods and modifies their return values.


```bash
// Hook and Modify Method Returns

Java.perform(function () {
    var targetClass = "com.example.app.TargetClass";  // Replace with the target class name
    var targetMethod = "targetMethod";  // Replace with the target method name

var TargetClass = Java.use(targetClass);
    TargetClass[targetMethod].overload().implementation = function () {
        var returnValue = this[targetMethod].apply(this, arguments);
        console.log('Original return value: ' + returnValue);

// Modify the return value
        returnValue = "ModifiedReturnValue";  // Replace with the desired return value
        console.log('Modified return value: ' + returnValue);
        return returnValue;
    };
    console.log('Method return modification script loaded');
});

```
4. Dump All Loaded Classes and Methods
This script lists all loaded classes and their methods, which can be useful for reconnaissance.

```javascript

// Dump All Loaded Classes and Methods

Java.perform(function () {
    var classes = Java.enumerateLoadedClassesSync();
    classes.forEach(function (className) {
        try {
            var classMethods = Java.use(className).class.getDeclaredMethods();
            console.log('Class: ' + className);
            classMethods.forEach(function (method) {
                console.log(' - Method: ' + method);
            });
        } catch (e) {
            console.error(e);
        }
    });

    console.log('Loaded classes and methods dump script loaded');
});
```

5. Bypass SSL Pinning (Alternate Method)
This script bypasses SSL pinning by replacing the TrustManager.

```bash
// Alternate SSL Pinning Bypass

Java.perform(function () {
    var TrustManagerImpl = Java.use('com.android.org.conscrypt.TrustManagerImpl');
    TrustManagerImpl.checkServerTrusted.implementation = function (chain, authType) {
        console.log('Bypassing TrustManagerImpl.checkServerTrusted');
    };

    var TrustManagerFactory = Java.use('javax.net.ssl.TrustManagerFactory');
    TrustManagerFactory.getTrustManagers.implementation = function () {
        console.log('Bypassing TrustManagerFactory.getTrustManagers');
        return [TrustManagerImpl.$new()];
    };

    console.log('Alternate SSL pinning bypass script loaded');
});
```

6. Bypass Secure Flag (Prevent Screenshots)
This script disables the FLAG_SECURE flag to allow screenshots and screen recording.

```bash

// Bypass Secure Flag

Java.perform(function () {
    var Window = Java.use('android.view.Window');
    Window.setFlags.implementation = function (flags, mask) {
        if ((flags & 8192) == 8192) {  // 8192 is the value of FLAG_SECURE
            console.log('Bypassing FLAG_SECURE');
            flags &= ~8192;  // Remove FLAG_SECURE
        }
        this.setFlags(flags, mask);
    };

    console.log('Secure flag bypass script loaded');
});
```

7. Inspect HTTP Requests and Responses
This script hooks into the HTTP request and response methods to log the data being sent and received.

```bash
// Inspect HTTP Requests and Responses

Java.perform(function () {
    var HttpURLConnection = Java.use('java.net.HttpURLConnection');
    
    HttpURLConnection.getInputStream.overload().implementation = function () {
        console.log('HTTP Request URL: ' + this.getURL());
        var inputStream = this.getInputStream();
        var buffer = Java.array('byte', []);
        var data = inputStream.read(buffer);
        console.log('HTTP Response Data: ' + Java.use('java.lang.String').$new(buffer));
        return inputStream;
    };

    HttpURLConnection.getOutputStream.overload().implementation = function () {
        console.log('HTTP Request URL: ' + this.getURL());
        var outputStream = this.getOutputStream();
        console.log('HTTP Request Method: ' + this.getRequestMethod());
        return outputStream;
    };

    console.log('HTTP request and response inspection script loaded');
});
```

usage
Install Frida:
Ensure you have Frida installed on your system. You can install it using pip:


pip install frida-tools
Run the Script:
Execute the script on the target application using Frida:


frida -U -f <package_name> -l <script_name>.js --no-pause
Replace <package_name> with the package name of the target Android application and <script_name> with the name of the script file.

Contributing
Contributions are welcome! Please submit a pull request or open an issue to discuss improvements or new features.

License
This project is licensed under the MIT License.


This `README.md` file includes all the scripts and instructions for using them, making it easy for others to understand and utilize the provided Frida scripts.

