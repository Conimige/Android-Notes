# 关于遇到Android Studio开发中遇到修改compileSdkVersion或者导入其他人工程时出现奇怪的异常

报错位置：
诸如以下的错误
...\build\intermediates\res\merged\debug\values-v23\values-v23.xml 
Error:(2) Error retrieving parent for item: No resource found that matches the given name 'android:TextAppearance.Material.Widget.Button.Inverse'.  
Error:(2) Error retrieving parent for item: No resource found that matches the given name 'android:Widget.Material.Button.Colored'.  

问题是该错误均存在于Android SDK中，查看该错误应该是布局文件引起的，推测原因是将compileSdkVersion修改导致一些之前版本的样式不可用导致的。

那么，修复此问题理论上可以通过将SDK中values-v23部分删除并ReBuild工程解决，即以下办法：
1.修改build.gradle中compileSdkVersion版本，从23+修改为22+，后报错
2.WARNING：以下操作请预先备份；
  进入Android SDK\extras\android\m2repository\com\android\support\appcompat-v7目录下删除23+所有目录；
  删除maven-metadata.xml中<version>23.0.0</version>以上所有部分；
  回到Android Studio，Clean工程，ReBuild工程，解决。
  
不过此方法并不好，原因是此修改方法影响了SDK，会导致其他程序出现问题，缺少values-v23样式
那么更好的解决办法是在build.gradle的dependencies{}中修改compile 'com.android.support:appcompat-v7:23.2.1'为compile 'com.android.support:appcompat-v7:22.0.0+'即可。     

坑已解决。
//End
