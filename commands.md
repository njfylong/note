## 常用命令

#### 搜索软件
```
apt-cache search
```

#### 打点
```
adb shell input tap
```

#### MTK image解压、打包
```
./out/host/linux-x86/bin/simg2img system.img system.img.ext4
sudo mount -t ext4 -o loop system.img.ext4 system_tmp

./out/host/linux-x86/bin/make_ext4fs -s -l BOARD_PROTECT_SIMAGE_PARTITION_SIZE -a system system_new.img system_tmp
```

#### python加密
```
python -m py_compile xxx.py
```

#### 抓取memory和CPU信息
```
adb shell top -t -m 10 -d 1
adb shell vmstat
```

#### 常用adb命令
```
adb shell pm list packages -f //查看所有安装的应用
adb shell reboot -p //用命令关手机
adb shell input keyevent 82 && adb shell stop && sleep 2 && adb shell start //快速重启
adb shell pm clear package  //清空某一应用数据
adb shell dumpsys activity | sed -n -e '/Stack #/p' -e '/Running activities/,/Run #0/p' //查看Activity栈
adb shell am broadcast -a xxx //发送xxx广播
```

#### 测量应用启动的时间
```
adb shell am start -W pacakgename/packagename.class
例：adb shell am start -W com.android.settings/com.android.settings.Settings
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.android.settings/.Settings }
Warning: Activity not started, its current task has been brought to the front
Status: ok
Activity: com.android.settings/.Settings
ThisTime: 447 	//一般和TotalTime时间一样，除非在应用启动时开了一个透明的Activity预先处理一些事再显示出主Activity，这样将比TotalTime小
TotalTime: 447	//应用的启动时间，包括创建进程+Application初始化+Activity初始化到界面显示
WaitTime: 535	//一般比TotalTime大点，包括系统影响的耗时
Complete
```

#### 打印堆栈
```
android.util.Log.d("ylfu", Log.getStackTraceString(new Throwable()));
```

#### 在所有xml内容里查找XXX
```
grep -rnIs XXX --exclude-dir=out --exclude-dir=.repo --exclude-dir=.git --include=*.xml *
```

#### 在整个项目搜索XXX.java
```
find . ! -path "./.repo*" ! -path "./out*" ! -path *\.git/* -name XXX.java
```

#### Terminal颜色设置
背景颜色　#07242E    
文字颜色　#708284
