####Android NFC 基础

　　NFC通信中使用NDFF数据格式有两种主要的使用场景：  


- 从NFC标签中读取NDFF数据　　
- 传输NDFF信息同一个设备到另一个设备  

　　从NFC标签中读取NDFF数据是由标签分派系统（[tag dispatch system](http://developer.android.com/guide/topics/connectivity/nfc/nfc.html#tag-dispatch "tag dispatch system")）负责的.它会分析已经发现的NFC标签，适当地分类数据，启动对已分类的数据感兴趣的应用。某一个应用想要获取已经扫描到的NFC标签可以声明一个[intent filter](http://developer.android.com/guide/topics/connectivity/nfc/nfc.html#tag-dispatch)并请求获取数据。  
Android Beam功能允许一个设备通过物理接触传输一个NDFF格式数据到另一个设备中。不需要像蓝牙设备那样需要发现设备和配对。因为使用的是NFC API，因此任何程序都可以通过它传输数据。  

####标签分派系统（The Tag Dispatch System） 
　　Android设备通常都在寻找NFC标签，即使是屏幕锁屏状态，除非NFC是关闭的。当Android设备发现一个NFC标签,期望的行为是使得最合适的activity来获取这个intent而不是询问用户选择哪一个。因为设备在扫面NFC标签的时候是在非常短的距离上，如果要让用户在屏幕上选择的话，很可能会将设备移出通信范围。因此应该使得自己的activity只获取自己感兴趣的，阻止出现activity Chooser。  

　　为了帮助你完成这个目标，Android提供了一个特别的标签分派系统来分析已经扫描到的NFC标签，解析他们，并且试图定位对扫描到的数据感兴趣的应用。它通过如下方式来完成：　　

1.解析NFC标签并且弄清MIME类型或者在标签中标记数据载荷的URI
2.封装MIME类型或者URI和payload到intent里。这前两个步骤描述具体见[How NFC tags are mapped to MIME types and URIs](http://developer.android.com/guide/topics/connectivity/nfc/nfc.html#ndef)  
3.基于intent启动一个activity。详见[How NFC Tags are Dispatched to Applications](http://developer.android.com/guide/topics/connectivity/nfc/nfc.html#dispatching)

####NFC标签如何映射到MIME类型和URI  
　　在开始写你的NFC应用之前很重要的是理解标签之间的不同类型，标签分派系统如何解析NFC标签，还有一些在检测到一个NDFF消息时做的特别的工作。NFC标签在很多技术中都很流行，也可以有许多方式来将数据写入。Android支持大部分NDFF标准，NDFF标准定义在[NFC Forum](http://nfc-forum.org/)  
　　NDFF数据封装在一个消息([NdefMessage](http://developer.android.com/reference/android/nfc/NdefMessage.html))中，该消息包含一个或多个记录([NdefRecord](developer.android.com/reference/android/nfc/NdefRecord.html)）.每一个NDFF记录必须是完整格式的。Android也支持其他格式的不包含NDFF数据的标签，那样你可以使用android.nfc.tech中的类。使用其他类型的标签涉及到写自己的协议栈来和标签通信。所以我们推荐使用NDFF来简化开发。
> Note: To download complete NDEF specifications, go to the [NFC Forum Specification Download](http://nfc-forum.org/events/shoptalk/) site and see [Creating common types of NDEF records](http://developer.android.com/guide/topics/connectivity/nfc/nfc.html#creating-records) for examples of how to construct NDEF records. 

因为你有一些NFC标签的知识背景，所以下面章节将更详细地描述Android如何获取NDFF格式标签。当一个Android设备扫描一个NFC标签包含NDFF格式数据，它解析数据并且试图获取数据的MIME类型和URI标记。为了做到这些，系统读取NdefMessage中第一个NdefRecord来决定如何翻译整个NDFF信息。在一个完整格式的NDFF消息中，第一个个NdefRecord包含如下域：  
**3-bit TNF(Type Name Format)**  
　　指示如何翻译变长类型域，有效的值在表１中。

**Variable　length　type　变长类型**   
　　描述记录的类型。如果使用TNF_WELL_KNOWN，使用这个域来明确记录类型。   
　　Definition （RTD）有效的RTD值描述在表2中。
  
**Variable length ID 变长ID**  
　　一个记录的唯一标识符，这个域并不常被使用，但是如果你需要区别一个标签，你可以为它创建一个ID。  

**Variable length payload 变长载荷**  
　　这是你真正想读写的数据载荷。一个NDFF消息包含多个NDFF记录，所以不要假定全部的载荷都在NDFF消息的第一个NDFF记录中。  

标签分发系统使用TNF和类型域来试图映射一个MIME类型或者URI到NDFFmessage。如果成功，它将封装这些信息一个到[ACTION_NDEF_DISCOVERED](developer.android.com/reference/android/nfc/NfcAdapter.html#ACTION_NDEF_DISCOVERED)intent。然而也有一些情况下标签分发系统不能通过第一个NDFF记录就决定数据类型。这种情况发生在当NDFF数据不能被映射到MIME类型或者URI，或者当NFC标签不能包含NDFF数据为开始数据。在这些情况下，一个标签对象，他包含标签的技术信息和载荷，将被封装到[ACTION_TECH_DISCOVERED](developer.android.com/reference/android/nfc/NfcAdapter.html#ACTION_TECH_DISCOVERED)。