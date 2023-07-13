# burpsuit_pro 破解

### 环境准备

下载Burpsuite Pro 2022.2.2 Jar文件、注册机、jdk

```text
 Burpsuite：https://portswigger.net/Burp/Releases
 注册机：https://github.com/h3110w0r1d-y/BurpLoaderKeygen/releases
 jdk：https://www.oracle.com/java/technologies/downloads/#jdk17-windows
```

![img](v2-b654879cb104e6801bc69c9a441e45ae_720w.webp)

![img](v2-3ab072fca08e56605646ae5555424330_720w.webp)

![img](v2-26bb80f2eb98208641e53c9c0b68b100_720w.webp)

### 破解步骤

### 1、安装好jdk并配置好环境变量，将下载好的两个jar文件放在同一个文件夹下。 

![img](v2-465f6e4a15654a6d715b4b00350da329_720w.webp)

### 2、直接运行注册机：BurpLoaderKeygen.jar文件，双击或者使用 java -jar命令打开也可以。注册机会自动识别目录下的burpsuite_pro_v2022.2.2.jar文件，并且自动添加参数。 

![img](v2-dea0fc6fcd3f16109f7a33ab0c92c826_720w.webp)


**点击run，启动burpsuite。**

![img](v2-e1ebae58c4ab91b14ec32bbd3b36931a_720w.webp)


**当出现这个界面，说明已经破解完成，点击 Next 可以直接使用了。**

### 3、Diy自己的license信息： **点击Help——>Clean Burp from computer** 

![img](v2-4e9e5dbc79c941617676b71c55c43dc9_720w.webp)


**勾选 Remove Burp license key，点击Next**

![img](v2-31b9548f20fa5879c5ce4b7c72fadc21_720w.webp)


**再次点击run的时候，就会跳出重新注册的界面**

![img](v2-dec5afe7430834bf69897dc5f2762f91_720w.webp)


**Diy自己的license信息：**

![img](v2-733d77b61e7b623234944fdb8c48cc09_720w.webp)


**将BurpLoaderKeygen.jar界面上的License信息复制到burpsuite上面：**

![img](v2-4f41b8b0303edaeda5e10d3f94abd8bd_720w.webp)


**点击Next**

![img](v2-55503ed30a639b74ecced4b1bbd83514_720w.webp)

![img](v2-90e57199faa7f7a96daf93a47f5da957_720w.webp)

![img](v2-f87c2e0697c6c45a9a1cc81d6326360d_720w.webp)


**点击Next，完成激活**

![img](v2-45e74820f37242257e758de3e7b71a12_720w.webp)

![img](v2-7e2118df365f1cf9ffc1625d94ec8cb9_720w.webp)

### 4、创建bat文件启动Burpsuite

### **由于每次启动burpsuite都要先运行BurpLoaderKeygen.jar，然后再点击run，这样很不方便，所以我们写一个启动程序来直接启动burpsuite** **在同目录下创建burpsuite.bat文件，内容如下：**

```bat
java -javaagent:BurpLoaderKeygen.jar -noverify -jar burpsuite_pro_v2022.2.2.jar
```

![img](v2-cc72aa0afd94c0408464de5c4a01f9cf_720w.webp)


**双击burpsuite.bat文件，直接启动burpsuite**

![img](v2-8d5b33fc0d52d7e18787fa158e62a04f_720w.webp)

### 5、解决掉CMD窗口问题


**我们运行完bat文件打开burpsuite后，会出现一个CMD窗口，并且这个窗口不可以关掉，关掉的话，burpsuite也会直接关掉，我们更改一下burpsuite.bat文件里的内容来解决这个问题**

![img](v2-8e8d912d2dd35b3d5ef99c0d2ef7733d_720w.webp)

```text
 start javaw -javaagent:BurpLoaderKeygen.jar -noverify -jar burpsuite_pro_v2022.2.2.jar
```

![img](v2-8d92f8d5f73d94396fe288e23b08d0f4_720w.webp)


**这样就不会有CMD窗口啦！**

### 6、创建桌面快捷方式

**添加自己的burpsuite.bat文件路径**

![img](v2-3eb3e7b667c378ab880d45c1d3e92a2f_720w.webp)


**名字随便取**

![img](v2-319e89682f84d5b84ad5f60e04a8e6f8_720w.webp)


**添加上图标就可以啦！**

![img](v2-24ff0c64088babe9990638db71f1f6de_720w.webp)