# Android M下新的权限管理问题

前几天开了一个新项目，开发过程中遇到了一个极其奇葩的问题，

我做的项目采用了最新的Android Studio测试版本开发，程序需要访问网络、获取手机状态等权限，这很正常对吧？

可是做好的程序测试在Android L以下（包含L）是正常的，在Android M+安装时提示：

“该程序不需要任何权限”



纳尼？不需要任何权限？你™逗我？

然后呢，程序运行过程中不出意外的崩溃了

日志显示崩溃的原因是没有声明权限..........WTF，我还不至于傻到没在Android Manifests声明权限呀(╯‵□′)╯︵┻━┻

然后呢我尝试呢在低版本（Android L）测试模拟器上运行了下，一切正常，在Android4.4.4-更是运行的飞起......

Android M的设置-应用管理中找到我的程序后，权限中的选项竟然全是关掉的，手动开启后运行一切正常

怎么回事......

别急，让我想想，我建立工程使用的是最新测试版本的Android Studio，是不是这原因？换了个稳定版的，结果还是不出意外的崩溃了...

好吧，不是兼容问题啊，那我再想想，Android M是不是高了点啥特别的东西？

网上搜了下Android M的更新日志.......嗯，新的权限管理？什么鬼？

于是乎，原因其实是Android M开始将权限做成了动态调用的......好吧谷歌你够潮

## 真正原因

以下来自谷歌官方的解释：

Beginning in Android 6.0 (API level 23), users grant permissions to apps while the app is running, not when they install the app. This approach streamlines the app install process, since the user does not need to grant permissions when they install or update the app. It also gives the user more control over the app's functionality; for example, a user could choose to give a camera app access to the camera but not to the device location. The user can revoke the permissions at any time, by going to the app's Settings screen.

不废话了，上解决办法......
## 解决办法

exp.我们常用的打电话：
```java
 final public static int REQUEST_CODE_ASK_CALL_PHONE = 123;

  public void onCall(String mobile){
      this.mMobile = mobile;
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
          int checkCallPhonePermission = ContextCompat.checkSelfPermission(mContext,Manifest.permission.CALL_PHONE);
          if(checkCallPhonePermission != PackageManager.PERMISSION_GRANTED){
              ActivityCompat.requestPermissions(mContext,new String[]{Manifest.permission.CALL_PHONE},REQUEST_CODE_ASK_CALL_PHONE);
              return;
          }else{
              //调用拨号
              callDirectly(mobile);
          }
      } else {
          //调用拨号
          callDirectly(mobile);
      }
  }
  
  //回调方法
  @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        switch (requestCode) {
            case REQUEST_CODE_ASK_CALL_PHONE:
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    // 用户点击了允许获取权限
                    callDirectly(mobile);
                } else {
                    // 用户点击了不允许获取权限
                    Toast.makeText(MainActivity.this, "CALL_PHONE Denied", Toast.LENGTH_SHORT)
                            .show();
                }
                break;
            default:
                super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        }
    }
```
然后，当我们的程序需要打电话的时候会弹出确认对话框：“是否允许XXX拨打电话”...好了完美解决........么？

这不蛋疼么每次调用个权限还得写这么一长串代码，Android Manifests声明也不能去掉否则低版本手机上程序有得崩掉了......

好吧，有开源的库可以帮你解决这问题：传送门：https://github.com/k0shk0sh/PermissionHelper


不还得麻烦？好吧我就说说我是怎么解决的吧，干一件事就行，简单快捷方便！

虽然可能是暂时可以这么玩的，但目前在谷歌没有统一权限问题的前提下还是没问题的。

##完美解决方案（暂时）：
修改Build.gradle中defaultConfig{}下的targetSdkVersion为22

就好了......

真的别打我，这就是实实在在的碎片化，实实在在的坑，我相信不少同学应该会掉进去吧？（笑）


//End
