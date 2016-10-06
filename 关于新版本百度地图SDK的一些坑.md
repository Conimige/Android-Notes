###### 在Manifest中的<application>标签下添加如下配置
```
<service
    android:name="com.baidu.location.f"
    android:enabled="true"
    android:process=":remote" />
```
###### 在初始化Application类（没有则在第一个Activity）的OnCreate方法添加
```
public LocationService locationService;
...
@Override
public void onCreate() {
    super.onCreate();
    locationService = new LocationService(getApplicationContext());
    ...
}
```
然后，编译运行，你会发现......咦？怎么还定位在非洲西海岸？
新版本系统请注意定位权限的申请，或者直接修改工程的targetSdkVersion小于等于22即可。

####二、线路规划的问题
新版本百度地图SDK不再集成路线规划相关的类，而将他们开源了，然而，例如我之前要是用到的根据步行规划路线的关键类WalkingRouteOverlay就不再存在了，然而，查询半天也没个解决办法，官方社区是有解释，但解决起来还是蛮费功夫的，具体来说，官方只给出了“哦，我们这个开源了你们自己去找源代码吧”大概这样子的答复，然而呢，事实是下载到源代码后你会眼花缭乱一脸蒙逼，原因是这一堆子的类互相都有关联，怎么集成啊(╯‵□′)╯︵┻━┻
好吧，我这里直接给出相关类，如有需要将这个包里头所有java文件拷到你的工程里就好了，不过注意改java类的包名哦。
```
下载地址：http://pan.baidu.com/s/1hsv3LaO
```

以上。
如果还有不明白的欢迎留言，如有建议也欢迎提出，就这样了，我继续搬砖去。
