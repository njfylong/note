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

#### Terminal颜色设置
背景颜色　#07242E    
文字颜色　#708284
