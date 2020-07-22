# NfcSample
NFC库，兼容4.3之前API以及4.4之后的API，读卡器模式，Sample读取羊城通卡号、余额、交易记录

[ ![NFClib](https://img.shields.io/badge/NFClib-1.0.4-blue) ](https://bintray.com/scauzhangpeng/NFCLib/nfclib/1.0.2/link)

本库将NFC的CardReader模式在API 4.3以及API 4.4之后不同的注册方式进行统一封装处理，使用本库不必考虑版本API差异，仅按照生命周期调用注册即可。

本库的Sample例子：利用手机NFC读出城市一卡通（羊城通、深圳通）的卡号、余额、交易记录

## 截图
|sony xm50t(4.3)|oppo A37m(5.1)|锤子 坚果pro(7.1.1)|
|---------------|--------------|------------------|
|<img src="https://s1.ax1x.com/2020/07/22/UHMZ6K.jpg" width="300"/>|<img src="https://s1.ax1x.com/2020/07/22/UHMFYR.png" width="300"/>|<img src="https://s1.ax1x.com/2020/07/22/UHMVl6.png" width="300"/>|
|nexus 6(5.0)|mate8 QQ-卡号|mate8 QQ-记录|
|<img src="https://s1.ax1x.com/2020/07/22/UHM9w4.png" width="300"/>|<img src="https://s1.ax1x.com/2020/07/22/UHMeOO.png" width="300"/>|<img src="https://s1.ax1x.com/2020/07/22/UHMik9.png" width="300"/>|
## 如何使用本库
1.使用Gradle依赖 

```
implementation 'com.ppy:nfclib:1.0.4'
```
```
//java版本
implementation 'com.ppy:nfclib:1.0.2'
```

2.初始化  

本库所有API统一从NfcCardReaderManager调用，NfcCardReaderManager采用建造者，初始化如下：
```java
private NfcCardReaderManager mReaderManager;

private void initNfcCardReader() {
    mReaderManager = new NfcCardReaderManager.Builder(this)
	.enableSound(false)//设置是否有声音
	.build();
}
```

3.生命周期调用进行注册NFC  
```java  

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
	...
	mReaderManager.onCreate(getIntent);
    }

    @Override
    protected void onResume() {
        super.onResume();
        mReaderManager.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        mReaderManager.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mReaderManager.onDestroy();
    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        mReaderManager.onNewIntent(intent);
    }
```
4.初始化的时候设置监听，一旦卡片贴紧NFC感应处，将回调onCardConnected
```java

  private CardOperatorListener mCardOperatorListener = new CardOperatorListener() {
    @Override
    public void onCardConnected(boolean isConnected) {
      if(isConnected) {
        //TODO 卡片已经贴在NFC感应处
      }
    }
  };
	
  @Override
  protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initNfcCardReader();//初始化
        mReaderManager.setOnCardOperatorListener(mCardOperatorListener);//设置监听
}
```

5.当回调NFC已读取卡片标签，则可以调用NfcCardReaderManager如下方法进行指令(APDU)操作。
```java
  public String sendData(byte[] data) throws IOException;

  public String sendData(String hexData) throws IOException;

  public byte[] tranceive(byte[] data) throws IOException;

  public byte[] tranceive(String hexData) throws IOException;
```

6.判断用户是否已经将卡贴在NFC感应处，可以调用NfcCardReaderManager的isCardConnected()方法

7.相关辅助工具类

|作用|方法|
|----|----|
|设备是否提供NFC|Util.isNfcExits|
|设备是否开启NFC|Util.isNfcEnable|
|跳转NFC设置界面|Util.intentToNfcSetting|
|设备是否提供AndroidBeam|Util.isAndroidBeamExits|
|设备是否开启AndroidBeam|Util.isAndroidBeamEnable|
|跳转AndroidBeam设置界面|Util.intentToNfcShare|
|金额分转元|Util.toAmountString|
|16进制转10进制|Util.hexToInt|
|16进制转10进制(小端模式)|Util.hexToIntLittleEndian|


## 特别说明几点
 - 有一些机型会贴卡后一直回调Tag，导致一直重读。曾经出现过的机型三星S3，Android 4.3
 
 - NFC跳转至设置界面，锤子坚果pro，需要跳转到Android Beam，不清楚是否锤子系统都这样。测试版本smartisan v3.5.2
 
 - OPPO A37m机型需要同时开启NFC以及Android Beam，论坛也有人反馈，但是目前本人测试后并不需要，但是QQ读卡需要同时开启，目前不清楚。
 (版本号：A37m_11_A.23_171025,内核版本：3.10.72-G201710251842,基带版本：MOLY.LR11.W1539.MD.TC16.JAD.SP.V1.P31.T47,2017/10/13 16:19)
 <img src = "https://s1.ax1x.com/2020/07/22/UHMkf1.png" width = "900"/>
 
 - 已测试机型如下表：
 
|机型|版本|ROM版本|
|----|---|-------|
|索尼XM50t|4.3|~|
|oppo A35|5.1|colorOs v3.0|
|锤子坚果pro|7.1.1|smartisan v3.5.2|
|谷歌Nexus 6|5.0/5.1.1/6.0.1/7.1.1|~|
|华为Mate 8|7.1.1|EMUI 5.0|
|华为p20 pro|8.1.0|EMUI 8.1.0|
|小米note2|8.0|MIUI 10.0|
|魅族16s|9.0|Flyme 7.3.0.0A|
|....希望大家一起测试完善|....|....|

## 感谢
1.潘工  
2.龙腿

## NFC参考链接
[谷歌官网NFC基础知识](https://developer.android.google.cn/guide/topics/connectivity/nfc/nfc.html)  
[谷歌官网NFC进阶知识](https://developer.android.google.cn/guide/topics/connectivity/nfc/advanced-nfc.html)  
[谷歌官网NFC卡模拟](https://developer.android.google.cn/guide/topics/connectivity/nfc/hce.html)

## 联系作者
本库关注的人并不多，目前也处于稳定使用，不定期佛性更新状态，如有bug或者需求不满足需要改的可以联系我  
写这个库由于之前工作中经常用NFC读写ETC卡(粤通卡、赣通卡、蒙通卡等)，也短暂搞过读银行卡卡号，读城市交通卡(羊城通、深圳通)都是我看一些资料后写的。  
博客：https://blog.csdn.net/scau_zhangpeng/article/details/70162775  
Email：scau_zhangpeng@163.com  

|微信|QQ|
|----|----|
|<img src = "https://s1.ax1x.com/2020/07/22/UHMCTJ.jpg" width = "200"/>|<img src = "https://s1.ax1x.com/2020/07/22/UHMESx.jpg" width = "200"/>|
