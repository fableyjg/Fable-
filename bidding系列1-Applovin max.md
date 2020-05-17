
# 1.bidding支持的渠道

目前Applovin max支持bidding渠道的有：AdColony，Applovin，Facebook，InMobi，Mintegral，Tapjoy，Verizon（按字母排序，上一次看文档只有5家，变化很快的）

# 2.接入文档：

Android：[https://dash.applovin.com/documentation/mediation/android/getting-started](https://dash.applovin.com/documentation/mediation/android/getting-started)


iOS：[https://dash.applovin.com/documentation/mediation/ios/getting-started](https://dash.applovin.com/documentation/mediation/ios/getting-started)

Unity：[https://dash.applovin.com/documentation/mediation/unity/getting-started](https://dash.applovin.com/documentation/mediation/unity/getting-started)

文件接入非常方便，按照步骤一步步来，基本没问题。

# 3.接入过程需要注意的点
## 3.1版本号

iOS是通过cocos pod一键导入，没有带版本号
```
pod ‘AppLovinSDK’
```
Android是通过
```gradle
dependencies {
    implementation 'com.applovin:applovin-sdk:+'
    ...
}
```
都有一个共同点，没带版本号，万一哪天程序员更新版本，没有兼容之前版本的API，可能会出问题。

建议：
这里直接带版本号接入
```gradle
dependencies {
    implementation 'com.applovin:applovin-sdk:9.11.6'
    ...
}
```

## 3.2 其他家的接入

不得不再次点赞max的文档，哪里不会点哪里，文档写的非常为开发者考虑。
假设，在Android中需要接入facebook和admob。

![](https://github.com/fableyjg/FableSay/blob/master/pic/applovin%20max%20gradle%20fb%20admob.png)

勾选了facebook和admob，如何配置，文档已经动态给出结果。
包括Android Manifest中需要配置的admob app id已经给出示例代码。

![](https://github.com/fableyjg/FableSay/blob/master/pic/applovin%20max%20admob%20appid.png)

``` gradle
dependencies {
    implementation 'com.applovin:applovin-sdk:9.12.6'
    implementation 'com.applovin.mediation:facebook-adapter:+'
    implementation 'com.android.support:recyclerview-v7:28.+'
    implementation 'com.android.support:appcompat-v7:28.+'
    implementation 'com.applovin.mediation:google-adapter:+'
}
```
如果applovin指定了版本，facebook，admob最好也指定一下版本。虽然大厂在发布adapter肯定做了各种测试，具体业务逻辑实现是否和大厂的测试用例一样，需要思考一下。指定版本就是接入的时候麻烦点，测试通过后，在线上跑的时候，不用担心版本的问题。
版本检测，推荐一个网站：[https://mvnrepository.com/](https://mvnrepository.com/)

## 3.3当广告加载失败时，学一下官方的处理方式

广告加载失败时，如何保证既占用用户的手机资源，又当用户需要广告时，能及时加载到广告。
豪放式：加载失败就立即请求广告，若没网，这里很容易导致死循环。
改进豪放式：加载失败等5秒或10秒，用户手机本来不多的线程资源被占用导致频繁的请求，加重应用的发热。
优雅式：看看max demo给的答案
```java
    @Override
    public void onAdLoadFailed(final String adUnitId, final int errorCode)
    {
        // Rewarded ad failed to load. We recommend retrying with exponentially higher delays.
        retryAttempt++;
        long delayMillis = TimeUnit.SECONDS.toMillis( (long) Math.pow( 2, retryAttempt ) );

        new Handler().postDelayed( new Runnable()
        {
            @Override
            public void run()
            {
                rewardedAd.loadAd();
            }
        }, delayMillis );
    }
```

留一个小问题，大家思考一下？
假设用户进入游戏，觉得插屏太打扰体验了；关闭网络，接着玩了10分钟游戏，到boss这里过不去，想通过观看激励视频复活，请问，用官方示例代码，需要多久后才能加载到广告？

如果知道答案，如何进一步优化，相信你已经有答案。

ps:按照加载失败，下一次加载的时长是上一次加载时间间隔的2倍；那么玩了10分钟，此时加载失败的时间为256秒，一共加载失败了8次；下一次加载需要512秒。
