---
title: Extracting pre-built native library of another Android App for embedding and method invoking inside our custom Android App
date: 2024-11-07 02:00:00 +0545
categories: [Android App Security]
tags: [Android, App Security, Reverse Engineering, Android App Security]
---

During reverse engineering android app we might find native libraries. Sometimes we might need to invoke the methods of those library to analyse its behavior. But we cannot call those methods just with library file. We need to build our own custom android app, embed that library to our app, then call the function with our custom input. 

# Extracting native libraries from APK

1. Decompile app with apktool
    
    ```bash
    apktool d <nameofapk.apk>
    ```
    
2. `<nameofapk>` folder will be created and there will be `lib` directory inside it. There it consist native libraries inside the folder named with respective architecture.
    
    ![image.png](../images/Extracting%20pre-built%20native%20library%20of%20another%20And%201374d591802a803aaed9e98960dfbb6e/image.png)
    

# Embedding native libraries to custom apk

If we have native library in the following directory structure, we can embed them to our custom app. Code base for all these library are same but compiled for different architecture. 

![native libraries](../images/Extracting%20pre-built%20native%20library%20of%20another%20And%201374d591802a803aaed9e98960dfbb6e/image%201.png)

native libraries

Open the project in which you want to embed the native code library. Initially, your project directory might look similar like below. 

![image.png](../images/Extracting%20pre-built%20native%20library%20of%20another%20And%201374d591802a803aaed9e98960dfbb6e/image%202.png)

Taking references from [https://developer.android.com/studio/projects/gradle-external-native-builds#jniLibs](https://developer.android.com/studio/projects/gradle-external-native-builds#jniLibs)

Place your native library in following format by creating `jniLibs`  directory inside `app/**src/main/**`

![image.png](../images/Extracting%20pre-built%20native%20library%20of%20another%20And%201374d591802a803aaed9e98960dfbb6e/image%203.png)

That’s all for embedding. 

# Invoking Methods in custom app

Methods can be invoked if and only if we know the package, class and methods name. 

![image.png](../images/Extracting%20pre-built%20native%20library%20of%20another%20And%201374d591802a803aaed9e98960dfbb6e/image%204.png)

In above example, we get  `Java_io_hextree_weatherusa_InternetUtil_getKey`  as function name. The naming **convention** for native library function name is  

```bash
Java_<package_name>*<class_name>*<method_name>
```

 We get `InternetUtil` as  class name, `getKey` is method name and  `io_hextree_weatherusa` is package name.

Now, In our custom app we need to create the java file with the name same as class name and declare package and declare the native method.

![image.png](../images/Extracting%20pre-built%20native%20library%20of%20another%20And%201374d591802a803aaed9e98960dfbb6e/image%205.png)

Now we can add another method that loads the library at runtime and calls the native function.

```bash
package io.hextree.weatherusa;

public class InternetUtil {
    private  static native String getKey(String str);

    public static  String solve(){
        System.loadLibrary("native-lib");
        return  getKey("moiba1cybar8smart4sheriff4securi");
    }

}

```

Now if we call the solve method anywhere within our app this will executes the `getKey` function from native library. 

Example: we call it at our main activily like below 

```bash
package com.example.simplebutton;

import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;

import io.hextree.weatherusa.InternetUtil;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView textView = findViewById(R.id.textView);

        Button homeButton = findViewById(R.id.mainButton);

        homeButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                textView.setText("Number of clicks are: "+ InternetUtil.solve());
            }
        });
    }
}
```

This way we successfully e**xtracted pre-built native library of another Android App and embedded and  invoked its method inside our custom Android App.**