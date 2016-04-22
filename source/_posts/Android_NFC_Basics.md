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

Table 1 描述标签分发系统是如何将TNF和类型域与MIME类型或者URI映射起来的。他也描述了哪种TNF不能被映射到MIME类型或者URI，在这种情况下，标签分发系统退回到ACTION_TECH_DISCOVERED。  

[Table 1. Supported TNFs and their mappings
](http://developer.android.com/guide/topics/connectivity/nfc/nfc.html#table1)


[Table 2. Supported RTDs for TNF_WELL_KNOWN and their mappings](http://developer.android.com/guide/topics/connectivity/nfc/nfc.html#table2)


####NFC标签如何分派到应用程序  
当标签分派系统创建完成一个封装了NFC标签和其标记信息的intent，它将这个intent送到对该intent感兴趣的应用的过滤器。如果有多个应用可以获取这个intent，Activity Chooser就会跳出，让用户选择一个Activity。标签分派系统定义了三种intent：  
1. ACTION_NDFF_DISCOVERED:这个intent是用来当一个标签包含NDFF载荷并且被被扫描到是可以识别的类型时启动一个Activity。这是最高优先级的intent，标签分派系统尽可能试图在其他intent之前为这个intent启动Activity。  
2. ACTION_TECH_DISCOVERED:如果没有activity来获取ACTION_NDFF_DISCOVERED intent，标签分派系统会试图用ACTION_TECH_DISCOVERED intent启动一个应用程序。这个intent也直接启动，当标签被扫描到包含NDEF数据但是不能被映射到MIME类型或者URI，或者如果标签不包含NDEF数据但是是一个已知的标签技术。  
3. ACTION_TAG_DISCOVERED:这个intent在没有activity响应ACTION_NDEF_DISCOVERED 和ACTION_TECH_DISCOVERED intent的时候启动。 

标签分派系统的基本工作方式如下：  
1. 当解析NFC标签的时候标签分派系统创建一个intent，并且试着使用该intent启动一个Activity。  
2. 如果没有activity 过滤器对应该intent。试着用更低一级的intent启动一个activity直到一个应用过滤器对应该intent或者试过所有的可能的intent。  
3. 如果没有应用 过滤器对应任何级别的intent，就什么也不做。  
![](http://developer.android.com/images/nfc_tag_dispatch.png)
如果可能尽量使用NDEF信息和ACTION_NDEF_DISCOVERED intent 因为这样更加地精确。这个intent使得应用启动更加快速直接，给用户更好的体验。  

####在Android Manifest中请求NFC接入  
在你访问设备的NFC硬件和更好得获取NFC intent 之前，把他们声明在AndroidManifest.xml文件中:  

- NFC<uses-permission>元素来接入NFC硬件：  
`<uses-permission android:name="android.permission.NFC" />`  
- 你的应用支持的最低SDK版本。API 9只通过ACTION_TAG_DISCOVERED支持有限的标签,并且通过EXTRA_NDEF_MESSAGES只给了访问NDEF的权限。没有其他标签支持，或者I/O操作访问权。API10包含广泛的读写支持，API 14提供更简单的方式来写入NDEF信息到其他设备。  
`<uses-sdk android:minSdkVersion="10" />`  
- uses-feature元素可以使得你的程序在google play中只为有NFC设备的硬件显示。  
`<uses-feature android:name="android.hardware.nfc" android:required="true" />`  
如果你的应用使用NFC功能，对应用来说但是这个功能并不重要，你可以省略uses-feature元素，在运行时通过getDefaultAdapter()方法检查NFC是否可用。  

####筛选NFC Intent  
为了在扫描到你想要的NFC标签时启动应用，你的应用可以在Manifest中筛选一个，两个或者所有三个NFC intent。然而你通常想要筛选ACTION_NDEF_DISCOVERED intent为了在启动应用的过程中达到最大程度的控制。ACTION_TECH_DISCOVERED intent是对于ACTION_NDEF_DISCOVERED 没有程序筛选的情况和载荷不是NDEF的情况的反馈。筛选ACTION_TAG_DISCOVERED通常太过于宽泛。许多程序会在ACTION_TAG_DISCOVERED之前筛选ACTION_NDEF_DISCOVERED 和ACTION_TECH_DISCOVERED，这样你的程序有一个低概率的启动机会。ACTION_TAG_DISCOVERED是仅仅可用于最后的手段来为程序筛选为了防止没有其他程序来获取ACTION_NDEF_DISCOVERED 或者 ACTION_TECH_DISCOVEREDintent。  

因为NFC标签部署的多样化，并且许多时候并不在自己的控制之下，这就是为什么你可以反馈其他两个intent。当你完全掌握标签的类型和数据写入，推荐的方式是使用NDEF格式。  

**ACTION_ NDEF_ DISCOVERED**  
为了筛选出ACTION_NDEF_DISCOVERED intent，声明intent过滤器连同你想要的数据。下面例子是为了筛选有MIME类型为text/plain 的ACTION_NDEF_DISCOVERED intent：  

    <intent-filter>
        <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain" />
    </intent-filter>

下面的例子是为了筛选一个URI格式为http://developer.android.com/index.html。  
     
    <intent-filter>
        <action android:name="android.nfc.action.NDEF_DISCOVERED"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:scheme="http"
              android:host="developer.android.com"
              android:pathPrefix="/index.html" />
    </intent-filter>  

**ACTION_ TECH_DISCOVERED**  
如果你的activity过滤器为了筛选ACTION_TECH_DISCOVERED intent,你必须创建一个XML资源文件使用一个 tech-list集合来指明你的应用支持的技术。你的activity需要考虑一个匹配问题：是否一个 tech-list集合是标签支持的技术的子集，你可以通过调用getTechList()来获取。  
举个例子，如果被扫描到的标签支持MifareClassic,NdefFormatble,和NfcA ,你的tech-list集合必须明确指定所有三个，两个或者一个这些技术并且没有其他技术。  
下面的例子定义了所有的技术，你可以移除你不需要的把这个文件保存到<project-root>/res/xml文件夹下。  

    <resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
        <tech-list>
            <tech>android.nfc.tech.IsoDep</tech>
            <tech>android.nfc.tech.NfcA</tech>
            <tech>android.nfc.tech.NfcB</tech>
            <tech>android.nfc.tech.NfcF</tech>
            <tech>android.nfc.tech.NfcV</tech>
            <tech>android.nfc.tech.Ndef</tech>
            <tech>android.nfc.tech.NdefFormatable</tech>
            <tech>android.nfc.tech.MifareClassic</tech>
            <tech>android.nfc.tech.MifareUltralight</tech>
        </tech-list>
    </resources>  
你也可以指定多个tech-list集合。每一个集合被认为是独立的，你的activity被认为是一个匹配，如果任何一个tech-list集合是被getTechList的一个子集。这为匹配的数据提供提供 且 和 或语义。下面的例子表示匹配支持NfcA 和Ndef技术或者支持NfcB和Ndef技术的标签：  
    

    <resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
    <tech-list>
        <tech>android.nfc.tech.NfcA</tech>
        <tech>android.nfc.tech.Ndef</tech>
    </tech-list>
    </resources>

    <resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
    <tech-list>
        <tech>android.nfc.tech.NfcB</tech>
        <tech>android.nfc.tech.Ndef</tech>
    </tech-list>
    </resources>  

在你的AndroidManifest.xml文件中，在<meta-data>元素中指定刚才创建的源文件。  


    <activity>
    ...
    <intent-filter>
        <action android:name="android.nfc.action.TECH_DISCOVERED"/>
    </intent-filter>

    <meta-data android:name="android.nfc.action.TECH_DISCOVERED"
        android:resource="@xml/nfc_tech_filter" />
    ...
    </activity>  

**ACTION_TAG_DISCOVERED**  
为了筛选出ACTION_TAG_DISCOVERED intent ，使用如下intent 过滤器：  


    <intent-filter>
         <action android:name="android.nfc.action.TAG_DISCOVERED"/>
    </intent-filter>  


####从intent中获取信息  
如果一个activity因为NFC intent启动，你可以从这哥intent中获取有关扫描到的标签的信息。intent可以包含下列额外部分，具体的取决于扫描到的是哪种标签：  


- EXTRA_TAG(必须的)：一个代表着已扫描到的标签对象。  
- EXTRA_NDEF_MESSAGES（可选的）：一组从标签中解析出来的信息。这部分信息强制出现在ACTION_NDEF_DISCOVERED intents中。  
- EXTRA_ID(可选的)：标签的低位ID。  


为了获取这些额外的信息，检查你的activity是否通过NFC intent 启动，确保标签被扫描到，然后从标签中获得这些信息。  


     public void onResume() {
        super.onResume();
        ...
        if (NfcAdapter.ACTION_NDEF_DISCOVERED.equals(getIntent().getAction())) {
            Parcelable[] rawMsgs = intent.getParcelableArrayExtra(NfcAdapter.EXTRA_NDEF_MESSAGES);
            if (rawMsgs != null) {
                msgs = new NdefMessage[rawMsgs.length];
                for (int i = 0; i < rawMsgs.length; i++) {
                    msgs[i] = (NdefMessage) rawMsgs[i];
                }
            }
        }
        //process the msgs array
    }


另一种选择是你可以从intent中获取TAG对象，它包含信息载荷并且允许你枚举标签的技术。  
`Tag tag = intent.getParcelableExtra(NfcAdapter.EXTRA_TAG);`


####创建普通类型的NDEF记录  

----------
这部分描述如何创建一个普通类型的NDEF记录来帮助你写入数据到NFC标签中或者通过Android Beam 传输数据。  从Android 4.0(API 14)开始，createUri()方法便可用了，可以帮助你自动地创建URI记录 。从Android 4.1开始(API 16)createExternal() 和 createMime()方法就可用了，可以帮助创建MIME或者其他类型的NDEF记录。使用这些帮助方法可以避免手动创建NDEF信息时的错误。  

**TNF_ ABSOLUTE_URI**  

> 推荐使用RTD_URI 类型，而不是TNF_ABSOLUTE_URI，因为这样更加高效。  


通过如下方式创建TNF_ABSOLUTE_URI NDEF记录：  


    NdefRecord uriRecord = new NdefRecord(
         NdefRecord.TNF_ABSOLUTE_URI ,
         "http://developer.android.com/index.html".getBytes(Charset.forName("US-ASCII")),
         new byte[0], new byte[0]);

intent 过滤器是如下的：  


    <intent-filter>
        <action android:name="android.nfc.action.NDEF_DISCOVERED" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="http"
            android:host="developer.android.com"
            android:pathPrefix="/index.html" />
    </intent-filter>  

**TNF_ MIME_MEDIA**  
通过如下的方式创建TNF_MIME_MEDIA NDEF 记录：  
使用createMime（）方法：  


    NdefRecord mimeRecord = NdefRecord.createMime("application/vnd.com.example.android.beam","Beam me up, Android".getBytes(Charset.forName("US-ASCII"))); 

手动创建：  

        NdefRecord mimeRecord = new NdefRecord( NdefRecord.TNF_MIME_MEDIA ,
            "application/vnd.com.example.android.beam".getBytes(Charset.forName("US-ASCII")),
            new byte[0], "Beam me up, Android!".getBytes(Charset.forName("US-ASCII")));

过滤器如下：  


    <intent-filter>
        <action android:name="android.nfc.action.NDEF_DISCOVERED" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="application/vnd.com.example.android.beam" />
    </intent-filter>  

**TNF_WELL_KNOWN with RTD_URI**  
可以通过如下方式创建TNF_WELL_KNOWN NDEF记录：  
使用 createUri(String) 方法:

    NdefRecord rtdUriRecord1 = NdefRecord.createUri("http://example.com");  

使用createUri(Uri)方法：  

    Uri uri = new Uri("http://example.com");
    NdefRecord rtdUriRecord2 = NdefRecord.createUri(uri);  

手动创建NdefRecord：  

    byte[] uriField = "example.com".getBytes(Charset.forName("US-ASCII")); 
    byte[] payload = new byte[uriField.length + 1];              //add 1 for the URI Prefix
    byte payload[0] = 0x01;                                      //prefixes http://www. to the URI
    System.arraycopy(uriField, 0, payload, 1, uriField.length);  //appends URI to payload
    NdefRecord rtdUriRecord = new NdefRecord(
        NdefRecord.TNF_WELL_KNOWN, NdefRecord.RTD_URI, new byte[0], payload);


intent 过滤器：  


    <intent-filter>
        <action android:name="android.nfc.action.NDEF_DISCOVERED" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="http"
            android:host="example.com"
            android:pathPrefix="" />
    </intent-filter>

**TNF_ EXTERNAL_TYPE**  
通过如下方式创建TNF_EXTERNAL_TYPE NDEF记录：  
使用createExternal()方法：  

    byte[] payload; //assign to your data
    String domain = "com.example"; //usually your app's package name
    String type = "externalType";
    NdefRecord extRecord = NdefRecord.createExternal(domain, type, payload);  

手动创建NdefRecord：  


    byte[] payload;
    ...
    NdefRecord extRecord = new NdefRecord(
        NdefRecord.TNF_EXTERNAL_TYPE, "com.example:externalType", new byte[0], payload); 

intent 过滤器：  


    <intent-filter>
        <action android:name="android.nfc.action.NDEF_DISCOVERED" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="vnd.android.nfc"
            android:host="ext"
            android:pathPrefix="/com.example:externalType"/>
    </intent-filter>  

使用TNF_EXTERNAL_TYPE可以兼容更通用的NFC标签，更好得支持android和非android设备。