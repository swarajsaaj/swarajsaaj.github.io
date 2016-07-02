---
layout: post
title: Using Twitter Digits for SMS based logins
description: Sms logins using Twitter Digits library eliminating need of SMS providers
date: 2016-06-23 20:49
author: swarajsaaj
comments: true
excerpt: Twitter Digits is an awesome library to use for SMS based logins and verification..
permalink: posts/use-twitter-digits-login-sms-based
categories: [android]
---

# Introduction

In most of android applications login is an essential feature, and SMS based phone sign-in is becoming a better alternative than username-password combo for phone logins. Developers had to face alot of headache to set up SMS providers to verify the user and several tasks like requestins a verification code, verifying the SMS code by reading it automatically. [Digits by Twitter](https://fabric.io/kits/android/digits) was an awesome release which gets you up and running with SMS based logins free of cost and easy to setup.

# Overview

We will need a Single Broadcast Receiver for our Purpose, a listener to connect our receiver and activity and an Activity (or Fragment etc) for responding to the callback when the message is received.

- **SmsReceiver.java** :- This is a broadcast receiver that is called when SMS is received.
- **SmsListener.java** :- This would be our listener interface that will be passed on to our receiver.
- **MainActivity.java** :- Our calling code where all the action happens.

![Architecture](/images/otp/architecture.png)

# Implementation

First of all add the receiver entry for **SMS_RECEIVED** action in our **AndroidManifest.xml**

```xml

    <receiver android:name=".SmsReceiver">
        <intent-filter>
                <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
         </intent-filter>
    </receiver>

```
This ensures that whenever a SMS is received, this broadcast listener is sent the broadcast with SMS data.

Create a simple interface **SmsListener.java** as follows:-

```java

    public interface SmsListener {
            public void messageReceived(String messageText);
    }

```


In our BroadcastReceiver, we will implement our core code.

```java

    public class SmsReceiver extends BroadcastReceiver {
        
        private static SmsListener mListener;

        @Override
        public void onReceive(Context context, Intent intent) {
            Bundle data  = intent.getExtras();

            Object[] pdus = (Object[]) data.get("pdus");

            for(int i=0;i<pdus.length;i++){
                SmsMessage smsMessage = SmsMessage.createFromPdu((byte[]) pdus[i]);

                String sender = smsMessage.getDisplayOriginatingAddress();
                //You must check here if the sender is your provider and not another one with same text.

                String messageBody = smsMessage.getMessageBody();

                //Pass on the text to our listener.
                mListener.messageReceived(messageBody);
            }

        }

        public static void bindListener(SmsListener listener) {
            mListener = listener;
        }
    }


```
Lets see what goes on behind the covers here,
The intent contains PDU objects (Protocol Data Unit), which is a protocol for transfer of SMS messages in telecoms. We obtain an array of these messages that were sent to to our receiver by system.

Iterating all these, we create a SmsMessage object using createFromPdu() method as shown.
**smsMessage.getDisplayOriginatingAddress()** gives us the sender number.
**smsMessage.getMessageBody()** gives us the message text.

**Note that we have called mListener.messageReceived()** here as a callback when message is received.

We are now ready with our core classes, lets implement this.

```MainActivity.java

    public class MainActivity extends AppCompatActivity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            SmsReceiver.bindListener(new SmsListener() {
                @Override
                public void messageReceived(String messageText) {
                    Log.d("Text",messageText);
                     Toast.makeText(MainActivity.this,"Message: "+messageText,Toast.LENGTH_LONG).show();
                }
            });
        }
    }

```

This is our calling code (or "Driver" as it is called), it just binds the listener to our receiver by calling **SmsReceiver.bindListener** method and a custom implmentation telling what should be done when a message is received, this is the same method that is called in our BroadcastReceiver.

Thats it, we can see the toast


![Sample App](/images/otp/screenshot.png){:width="250px"}

# Package information and links.

In case you want to skip this configuration and just want to use the functionality, I have written a package for the same, its easy to use , just include it in your project and follow instructions to integrate

Github Link:-
[Otp Reader Plugin](https://github.com/swarajsaaj/otpReader)

# Source code.

[Download Source Code](https://github.com/swarajsaaj/sms_reader_blog_demo/archive/master.zip)