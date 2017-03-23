## Android的启动过程

Android是基于Linux基础上开发的OS，其内核还是Linux，所以Android系统的启动可以看成：
Linux -> Android，本文仅介绍Android的启动过程。

### init启动过程
Linux Kernel启动完成后，转而进入Android的启动，即由内核层转向用户层。init是
Linux Kernel完成后进入用户空间的第一个进程，后面的进程在该进程上fork出来的。


#### AIL(Android Init Language)
在init中，会涉及到很多的AIL（Android Init Language）也就是.rc文件，如init.rc、
init.environ.rc、init.usb.rc、init.${ro.hardware}.rc、init.${ro.zygote}.rc等。AIL
中主要做一些启动和属性相关的配置。

AIL包括四中类型：Actions、Command、Services、Options
* Actions  
Actions格式：
```
on <trigger> [&& <trigger>]*
   <command>
   <command>
   <command>
```
trigger是一个触发器，当系统匹配该触发器的事件发生时，将会执行command中命令。常见的command有：
bootchart_init、chmod、chown等。(更多command可参考./system/core/init/readme.txt)

* Services  
Services是系统启动具体是init开始是需要启动的service，这里的service跟后面我们要说的各种
service不是一个概念。  
Service格式如下：
```
service <name> <pathname> [ <argument> ]*
   <option>
   <option>
   ...
```
service中的option有：  
critical：在四分中内重启超过四次，系统重启进入recovery模式  
disabled：不能自动启动  
setenv：设置service环境变量  
socket：创建socket  
user：执行服务前更改用户名  
group：执行服务前更改用户组  
seclabel：执行服务前更改安全级别  
oneshot：退出时不重启服务  
class：指定服务类，相同类名的服务可以同时启动或停止，模式为default  
onrestart：当服务重新启动时执行一个命令  
writepid：创建进程时将子进程号写入文件  

* Imports  
这里的import不是一个command

#### init基本流程
init的入口main函数在system/core/init/init.cpp中，
```
if (!strcmp(basename(argv[0]), "ueventd")) {
    return ueventd_main(argc, argv);
}

if (!strcmp(basename(argv[0]), "watchdogd")) {
    return watchdogd_main(argc, argv);
}

```
根据传入参数，启动ueventd、watchdogd。这里的ueventd_main、watchdogd_main两个函数可以看成
是ueventd、watchdogd的入口函数，它们完全可以独立写成和init的模块，之所以没有这么做是因为跟init
共享的东西太多。这里把ueventd、watchdogd做成init的软链接,system/core/init/Android.mk:
```
# Create symlinks
LOCAL_POST_INSTALL_CMD := $(hide) mkdir -p $(TARGET_ROOT_OUT)/sbin; \
    ln -sf ../init $(TARGET_ROOT_OUT)/sbin/ueventd; \
    ln -sf ../init $(TARGET_ROOT_OUT)/sbin/watchdogd
```

```
// Clear the umask.
umask(0);
```
这里主要mask文件权限。  


```
// 添加环境
add_environment("PATH", _PATH_DEFPATH);

bool is_first_stage = (argc == 1) || (strcmp(argv[1], "--second-stage") != 0);

// Get the basic filesystem setup we need put together in the initramdisk
// on / and then we'll let the rc file figure out the rest.
if (is_first_stage) {
    mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
    mkdir("/dev/pts", 0755);
    mkdir("/dev/socket", 0755);
    mount("devpts", "/dev/pts", "devpts", 0, NULL);
    #define MAKE_STR(x) __STRING(x)
    mount("proc", "/proc", "proc", 0, "hidepid=2,gid=" MAKE_STR(AID_READPROC));
    mount("sysfs", "/sys", "sysfs", 0, NULL);
}
```
创建tmpfs、/dev/pts、/dev/socket、devpts等节点并挂载。  

```
// We must have some place other than / to create the device nodes for
// kmsg and null, otherwise we won't be able to remount / read-only
// later on. Now that tmpfs is mounted on /dev, we can actually talk
// to the outside world.
open_devnull_stdio();
klog_init();
klog_set_level(KLOG_NOTICE_LEVEL);
```
open_devnull_stdio：实现在system/core/init/Util.cpp中。  
klog_init：初始化kernel log，可以用于init过程的调试  
klog_set_level：设置kernel log的级别，默认的是KLOG_NOTICE_LEVEL，定义的级别有：
```
#define KLOG_ERROR_LEVEL   3
#define KLOG_WARNING_LEVEL 4
#define KLOG_NOTICE_LEVEL  5
#define KLOG_INFO_LEVEL    6
#define KLOG_DEBUG_LEVEL   7
```

```
// Indicate that booting is in progress to background fw loaders, etc.
close(open("/dev/.booting", O_WRONLY | O_CREAT | O_CLOEXEC, 0000));
```
创建/dev/.booting，加载fimware的时候用。  

```
property_init();

// If arguments are passed both on the command line and in DT,
// properties set in DT always have priority over the command-line ones.
process_kernel_dt();
process_kernel_cmdline();

// Propagate the kernel variables to internal variables
// used by init as well as the current required properties.
export_kernel_boot_props();                                             
```
初始化属性和导入kernel启动属性，主要有：ro.boot.serialno、ro.boot.mode、ro.boot.baseband、ro.boot.bootloader、ro.boot.hardware、ro.boot.revision。  

```
// Set up SELinux, including loading the SELinux policy if we're in the kernel domain.
selinux_initialize(is_first_stage);                                               ```
初始化selinux。  

```
property_load_boot_defaults();
```
加载property，这里主要是build.prop、system.prop、default.prop。  

```
Parser& parser = Parser::GetInstance();
parser.AddSectionParser("service",std::make_unique<ServiceParser>());
parser.AddSectionParser("on", std::make_unique<ActionParser>());
parser.AddSectionParser("import", std::make_unique<ImportParser>());
parser.ParseConfig("/init.rc");
```
载入init.rc并将service、on、*.rc参考AIL。跟Android N之前的版本相比，这块的实现有变化。Android N之前，action、command、service都是通过结构体实现，Android N中把它们提出来分别定义成类。


### system_server启动



### services启动
