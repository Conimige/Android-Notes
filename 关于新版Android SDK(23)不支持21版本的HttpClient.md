# 关于新版Android SDK(23)不支持21版本的HttpClient

有时候真的能折腾死人...前几天建立了个新工程，想做个post连网获取数据，结果发现HttpClient这个类怎么都找不到了

什么情况...import也找不到...仔细看了下，这个类应该是org.apache.http.client.HttpClient包中，但是我之前的项目也没额外引入这个包呀，那么这个包应该在SDK中存在的

然而并不能找到...

仔细对比了之前的工程和新工程，结果发现也就sdk版本使用的不一样，之前那个工程是android-21，因为更新了Android Studio和Android SDK所以新创建的工程是Android-23的

果然是这货的锅吧？(╯‵□′)╯︵┻━┻

之后去查了下，org.apache.http.client.HttpClient这个包果然集成在了android-21的SDK中，但是23版本的没有......

次奥...然而我并不能去改sdk版本，会出很大耦合性的口牙

## 解决方案

折腾了好久，包括去手动导入阿帕奇的HttpClient的jar包，但是依然存在我们代码中使用的其他东西，诸如BasicNameValuePair之类的不存在，一个一个导入太麻烦了啊...

所以说其实还是有简单的办法了，这个类不能使用的原因是————过期了...不推荐使用了

我们还要使用怎么办？

解，进入build.gradle(Module:app)，在android {...}的大括号内加入

useLibrary 'org.apache.http.legacy'

一行即可。

再次掀桌(╯‵□′)╯︵┻━┻
