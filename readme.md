# Autojs Pro 8.0 新增方法 #

编写时间:2019/11/13

编写人: 稻草人

----------
#### &#8195;&#8195;从Pro 7.0 版本以后Autojs接入了Shizuku的授权功能,该功能需要手机连接电脑后,在电脑上使用adb命令来激活Shizuku,再在Shizuku上授权AutojsPro.授权的好处是:可以通过adb权限调用原本需要root权限才能使用的系统api.例如Tap Swipe等函数可以免root执行.可以调用shell命令切换飞行模式,杀死app的后台进程,清理app的应用数据,可以使用shell直接启动intent,实现类似冰箱/黑域app那样的功能等等...当然弊端就是每次重启后授权消失,需要重新激活! ####

## Shizuku下载地址: ##

	[https://www.coolapk.com/apk/moe.shizuku.privileged.api](https://www.coolapk.com/apk/moe.shizuku.privileged.api)
`关于Shizuku的激活方法,在该app内有使用教程,需要会使用adb命令.`

----------

## $shell ##
### $shell.checkAccess(permission); ###

&#8195;&#8195;检查权限

&#8195;&#8195;&#8195;&#8195;`permission` `{string}`目前该参数只有两个值 "root"或者"adb"

&#8195;&#8195;&#8195;&#8195;该方法返回一个Bool值

例:

    var isRoot = $shell.checkAccess("root");
    if (isRoot) {
    	toastLog("autojs已经获取到root权限");
    }
### $shell.setDefaultOptions(option); ###
&#8195;&#8195;设置shell的运行模式

&#8195;&#8195;&#8195;&#8195;`option` `{object}` 设置shell以什么权限执行命令 提供adb方式和root方式执行

&#8195;&#8195;&#8195;&#8195;`{adb : true}` 或者 `{root : true}`

#### 例1: 关闭QQ进程 ####

    var isAdb = $shell.checkAccess("adb");
    if (!isAdb) {
    	alert("请先使用Shizuku授权,再运行");
    	exit();
    }
    $shell.setDefaultOptions({adb : true}); //将shell切换到adb模式
    $shell("am force-stop com.tencent.mobileqq"); //使用adb权限杀掉QQ的进程,直接关闭QQ
#### 例2:打开给指定好友发红包界面 ####

先看一下root权限的代码:

    var friendQQ = "289986635"; //你好友的QQ号码
    app.startActivity({
    	packageName : "com.tencent.mobileqq",
    	className : "com.tencent.mobileqq.activity.qwallet.SendHbActivity",
    	extras : {
    		extra_data : '{"recv_type":1,"recv_uin":'+ friendQQ +',"channel":1,"bus_type":"1"}',
    	},
    	root : true  //需要root权限才能执行
    });
然后再来看一下使用adb权限如何使用

    var isAdb = $shell.checkAccess("adb");
    if (!isAdb) {
     	alert("请先使用Shizuku授权,再运行");
       	exit();
    }
    $shell.setDefaultOptions({adb : true}); //将shell切换到adb模式
    
    var option = {
    	packageName : "com.tencent.mobileqq",
    	className : "com.tencent.mobileqq.activity.qwallet.SendHbActivity",
    	extras : {
    		extra_data : '{"recv_type":1,"recv_uin":'+ friendQQ +',"channel":1,"bus_type":"1"}',
    	}
    }
    $shell("am start " + app.intentToShell(option));
	
示例2中root权限下的代码和adb权限下的代码效果相同.

在adb模式下内置的Tap、Swipe、RootAutomator将使用adb权限

对应方法的使用请参照官方文档RootAutomator部分

&#8195;&#8195;[https://hyb1996.github.io/AutoJs-Docs/#/coordinatesBasedAutomation?id=rootautomator](https://hyb1996.github.io/AutoJs-Docs/#/coordinatesBasedAutomation?id=rootautomator)

原Shell部分的命令使用请参照Shell部分的文档

&#8195;&#8195;[https://hyb1996.github.io/AutoJs-Docs/#/shell?id=附录-shell命令简介](https://hyb1996.github.io/AutoJs-Docs/#/shell?id=附录-shell命令简介)

#### &#8195;&#8195;adb权限模式下与root权限模式是有差别的,两者并非等同,root权限为最高权限,adb只是可以调用root用户的大部分权限 ####
切换回root模式

    $shell.setDefaultOptions({root : true});

### $shell(cmd, option); ###

&#8195;&#8195;以指定的权限执行shell命令

&#8195;&#8195;`cmd` `{string}` 要执行的shell命令

&#8195;&#8195;`option` `{object}` 使用何种权限执行 例如: `{adb : true}` 或者 `{root : true}`

例:

    $shell("pm clear com.tencent.mobileqq", {adb : true}); //以adb权限清空QQ的数据文件

### $shell.isRootAvailable() ###

&#8195;&#8195;判断当前设备是否root

&#8195;&#8195;返回一个Bool值

例:

    var isRoot = $shell.isRootAvailable();
    if (isRoot) {
    	toastLog("当前设备已经root");
    }

----------

## $base64 ##
字符串的base64加密与解密
### $base64.encode(str); ###

&#8195;&#8195;将字符串进行base64加密

&#8195;&#8195;`str`  `{string}` 要加密的字符串

例:

    var str = "Hello Autojs";
    var encode = $base64.encode(str);
    console.log(encode);

### $base64.decode(str); ###

&#8195;&#8195;将base64数据解密成字符串

&#8195;&#8195;`str` `{string}` base64加密后的字符串

例:

    var str = "Hello Autojs";
    var encode = $base64.encode(str);
    var decode = $base64.decode(encode);
    console.log(decode);

----------

## $crypto ##

新增的一些加密算法, 支持MD5, SHA, AES, RSA等...

### $crypto.digest(message, algorithm[, options]); ###

&#8195;&#8195;MD5和SHA的不可逆加密

&#8195;&#8195;`message` `{string}` 要加密的字符串

&#8195;&#8195;`algorithm` `{string}` 加密方式

&#8195;&#8195;`options` `{object}` 确定输入或者输入的数据类型

&#8195;&#8195;&#8195;&#8195;`input` `{string}` 输入的数据类型,`"string"`[默认]或`"file"`,  **若输入的类型是文件时,此选项必填**

&#8195;&#8195;&#8195;&#8195;`output` `{string}` 输出的数据类型,`"string"`[默认]或`"base64"`

例1:

    // 字符串消息摘要
    let message = "Hello, Auto.js Pro 7.0.4";
    // 输出各种消息摘要算法结果的hex值
    log("字符串: ", message);
    log("MD5: ", $crypto.digest(message, "MD5"));
    log("SHA1: ", $crypto.digest(message, "SHA-1"));
    log("SHA256: ", $crypto.digest(message, "SHA-256"));
    // 输出各种消息摘要算法结果的base64值
    log("MD5 [base64]: ", $crypto.digest(message, "MD5", {output: 'base64'}));
    log("SHA1 [base64]: ", $crypto.digest(message, "SHA-1", {output: 'base64'}));
    log("SHA256 [base64]: ", $crypto.digest(message, "SHA-256", {output: 'base64'}));

例2:

    // 文件消息摘要
    let file = "/sdcard/脚本/_test_for_message_digest.js"
    // 写入文件内容，提供为后续计算MD5等
    $files.write(file, "Test!");
    log("文件: ", file);
    log("MD5: ", $crypto.digest(file, "MD5", {input: 'file'}));
    log("SHA1: ", $crypto.digest(file, "SHA-1", {input: 'file'}));
    log("SHA256: ", $crypto.digest(file, "SHA-256", {input: 'file'}));

### $crypto.Key(key); ###

&#8195;&#8195;获取有一个AES秘钥, 由于AES等算法要求是16位的倍数,所以key值的字符串长度应为 16或32或48或64...等等

&#8195;&#8195;`key` `{string}` 字符串秘钥

例:

    let key = new $crypto.Key("password12345678"); //秘钥长度16位
    log("密钥: ", key);

### $crypto.generateKeyPair(algorithm[, length]); ###

&#8195;&#8195;生成RSA密钥对

&#8195;&#8195;`algorithm` `{string}` 加密方式

&#8195;&#8195;`length` `{int}` 密钥长度 默认为256位,但目前长度小于1024的已被证明不安全

例:

    let keyPair = $crypto.generateKeyPair("RSA");
    log("密钥对: ", keyPair);

### $crypto.encrypt(data, key, algorithm[, options]); ###

`data` `{string}` 要加密的数据

`key` `{object}` 秘钥

`algorithm` `{string}` 加密方式 

AES加密:`"AES/ECB/PKCS5padding"`

RSA加密:`"RSA/ECB/PKCS1padding"`

`options` `{object}` 确定数据的输入/输出类型

&#8195;&#8195;&#8195;&#8195;`input` `{string}` 输入的数据类型,`"string"`[默认]或`"file"`,  **若输入的类型是文件时,此选项必填**

&#8195;&#8195;&#8195;&#8195;`output` `{string}` 输出的数据类型,`"string"`[默认]或`"base64"`

例1: AES加密 对字符串的加密

    let message = "未加密字符串";
    log("明文: ", message);
    // 密钥，由于AES等算法要求是16位的倍数，我们这里用一个16位的密钥
    let key = new $crypto.Key("password12345678");
    log("密钥: ", key);
    // AES加密
    let aes = $crypto.encrypt(file, key, "AES/ECB/PKCS5padding");
    log("AES加密后二进制数据: ", aes);
    log("AES解密: ", $crypto.decrypt(aes, key, "AES/ECB/PKCS5padding", {output: 'string'}));

例2: AES加密 对一个文件加密

    // 文件消息摘要
    let file = "/sdcard/脚本/_test_for_message_digest.js"
    // 写入文件内容，提供为后续计算MD5等
    $files.write(file, "Test!");
    // 密钥，由于AES等算法要求是16位的倍数，我们这里用一个16位的密钥
    let key = new $crypto.Key("password12345678");
    log("密钥: ", key);
    // AES加密
    let aes = $crypto.encrypt(file, key, "AES/ECB/PKCS5padding",{input: 'file'});
    log("AES加密后二进制数据: ", aes);
    log("AES解密: ", $crypto.decrypt(aes, key, "AES/ECB/PKCS5padding", {output: 'string'}));

例3: RSA加密

    let message = "未加密字符串";
    log("明文: ", message);
    // 生成RSA密钥
    let keyPair = $crypto.generateKeyPair("RSA");
    log("密钥对: ", keyPair);
    // 使用私钥加密
    let rsa = $crypto.encrypt(message, keyPair.privateKey, "RSA/ECB/PKCS1padding");
    log("RSA私钥加密后二进制数据: ", rsa);
    // 使用公钥解密
    log("RSA公钥解密: ", $crypto.decrypt(rsa, keyPair.publicKey, "RSA/ECB/PKCS1padding", {output: 'string'}));

----------

## $notifications ##

发布一个通知,用法未知



----------

## $zip ##

### $zip.zipDir(dir, zipFile[, options]); ###

&#8195;&#8195;压缩一个文件夹

&#8195;&#8195;`dir` `{string}` 文件夹路径

&#8195;&#8195;`zipFile` `{string}` 压缩后zip文件的路径

&#8195;&#8195;`options` `{boject}` 压缩选项, 已知参数 `password`

例1:

    // 要压缩的文件夹路径
    let dir = '/sdcard/脚本/zip_test/';
    // 压缩后的文件路径
    let zipFile = '/sdcard/脚本/zip_out/未加密.zip';
    $files.remove(zipFile);
    $zip.zipDir(dir, zipFile);

例2:

    // 加密压缩文件夹
    let encryptedZipFile = '/sdcard/脚本/zip_out/加密.zip';
    $files.remove(encryptedZipFile);
    $zip.zipDir(dir, encryptedZipFile, {
    	password: 'Auto.js Pro'
    });

### $zip.zipFile(file, zipFile[, options]); ###

&#8195;&#8195;压缩一个文件

&#8195;&#8195;`dir` `{string}` 文件路径

&#8195;&#8195;`zipFile` `{string}` 压缩后zip文件的路径

&#8195;&#8195;`options` `{boject}` 压缩选项, 已知参数 `password`

例: 

    let zipSingleFie = '/sdcard/脚本/zip_out/单文件.zip'
    $files.remove(zipSingleFie);
    $zip.zipFile('/sdcard/脚本/zip_test/1.txt', zipSingleFie);

### $zip.zipFiles(fileList, zipMultiFile[, options]); ###

&#8195;&#8195;压缩多个文件

&#8195;&#8195;`fileList` `{Array}` 文件列表数组

&#8195;&#8195;`zipMultiFile` `{string}` 压缩后zip文件的路径

&#8195;&#8195;`options` `{boject}` 压缩选项, 已知参数 `password`

例:

    // 压缩多个文件
    let zipMultiFile = '/sdcard/脚本/zip_out/多文件.zip';
    $files.remove(zipMultiFile);
    let fileList = ['/sdcard/脚本/zip_test/1.txt', '/sdcard/脚本/zip_test/2.txt']
    $zip.zipFiles(fileList, zipMultiFile);

### $zip.unzip(zipFile, dir[, options]); ###

&#8195;&#8195;解压一个zip文件

&#8195;&#8195;`zipFile` `{string}` zip文件的路径

&#8195;&#8195;`dir` `{string}` 解压后存放的位置

&#8195;&#8195;`options` `{boject}` 压缩选项, 已知参数 `password`

例1: 

    $zip.unzip('/sdcard/脚本/zip_out/未加密.zip', '/sdcard/脚本/zip_out/未加密/');

例2:

    $zip.unzip('/sdcard/脚本/zip_out/加密.zip', '/sdcard/脚本/zip_out/加密/', {
    	password: 'Auto.js Pro'
    });

### $zip.open(zipFile); ###

&#8195;&#8195;打开一个zip文件.返回一个ZipFile对象

### ZipFile.removeFile(file); ###

&#8195;&#8195;在zip文件中删除一个文件

&#8195;&#8195;`file` `{string}` zip文件内部的文件

例:

    $files.create("/sdcard/脚本/zip_test/");
    $files.create("/sdcard/脚本/zip_out/");
    $files.write("/sdcard/脚本/zip_test/1.txt", "Hello, World");
    $files.write("/sdcard/脚本/zip_test/2.txt", "GoodBye, World");
    $files.write("/sdcard/脚本/zip_test/3.txt", "Auto.js Pro");
    
    let zipMultiFile = '/sdcard/脚本/zip_out/多文件.zip';
    $files.remove(zipMultiFile);
    let fileList = ['/sdcard/脚本/zip_test/1.txt', '/sdcard/脚本/zip_test/2.txt']
    $zip.zipFiles(fileList, zipMultiFile, {
    password : "123"
    });

    let z = $zip.open('/sdcard/脚本/zip_out/多文件.zip',{
    	password : "123"
    });
    z.removeFile('1.txt');

### ZipFile.addFile(file); ###

&#8195;&#8195;向zip压缩包中添加一个文件

&#8195;&#8195;`file` `{string}` 要增加的文件路径

例:

    let z = $zip.open('/sdcard/脚本/zip_out/多文件.zip',{
    	password : "123"
    });
    z.addFile('/sdcard/脚本/zip_test/3.txt');


----------
## app模块新增方法 ##

### app.getInstalledApps(); ###

&#8195;&#8195;获取当前已安装的所有app, 返回一个数组

单个元素内容如下:

    AppInfo{label='支付宝', className='com.alipay.mobile.quinox.LauncherApplication', descriptionRes=0, enabled=true, flags=951598660, manageSpaceActivityName='null', nativeLibraryDir='/data/app/com.eg.android.AlipayGphone-2/lib/arm', permission='null', processName='com.eg.android.AlipayGphone', sharedLibraryFiles=null, sourceDir='/data/app/com.eg.android.AlipayGphone-2/base.apk', targetSdkVersion=28, taskAffinity='com.eg.android.AlipayGphone', theme=2131034112, uiOptions=0, uid=10142, icon=2130837513, labelRes=2131230737, logo=0, metaData=Bundle[mParcelledData.dataSize=7188], name='null', nonLocalizedLabel=null, packageName='com.eg.android.AlipayGphone'}

### app.getApkInfo(apkPackageName); ###

&#8195;&#8195;获取一个apk包的信息

&#8195;&#8195;`apkPackageName` `{string}` apk包的路径

例:

    log(app.getApkInfo("/sdcard/QQ.apk"));

### app.getInstalledPackages() ###

&#8195;&#8195;获取当前已安装的所有app包的信息

----------

## device模块新增属性 ##

### device.screenRotation ###

&#8195;&#8195;返回 0 | 90 | 270

&#8195;&#8195;&#8195;&#8195;  0 : 手机竖直向上

&#8195;&#8195;&#8195;&#8195; 90 : 手机向左旋转屏幕

&#8195;&#8195;&#8195;&#8195;270 : 手机向右旋转屏幕


### device.screenOrientation ###

&#8195;&#8195;返回 portrait | landscape

&#8195;&#8195;&#8195;&#8195;portrait:竖向

&#8195;&#8195;&#8195;&#8195;landscape:横向


----------
## files模块新增方法 ##

### $files.observe() ###

&#8195;&#8195;此方法通过一个示例来演示使用方法

&#8195;&#8195;脚本说明:此脚本是一个监听QQ闪照的脚本,运行后会监听闪照文件的创建事件,当监听到以"_fp"为后缀的文件创建时,弹出alert弹窗

&#8195;&#8195;一个文件的创建可能会有多次create事件,所以弹窗会多次提醒...

示例:

    var sdcardPath = files.getSdcardPath();
    
    var CACHE_PATH = sdcardPath+"/tencent/MobileQQ/diskcache/";
    var RESULT_PATH = sdcardPath+"/QQ闪照/";
    
    if (!files.exists(RESULT_PATH)) {
    	files.ensureDir(RESULT_PATH);
    }
    
    let watcher = $files.observe(CACHE_PATH);
    
    watcher.on("create", (path) =>{
        if (path.substring(path.length-3) == "_fp") {
	        dialogs.confirm("Bingo!!!", "检测到一张QQ闪照,是否打开查看?", (value)=>{
	            if (!$files.exists(RESULT_PATH+path+".jpg")) {
	                $files.copy(CACHE_PATH+path, RESULT_PATH+path+".jpg");
	            }
	            if (value) {
	                app.viewFile(RESULT_PATH+path+".jpg");
	            } else {
	                toast("闪照已保存到: "+RESULT_PATH);
	            }
	        })
    	}
    });

