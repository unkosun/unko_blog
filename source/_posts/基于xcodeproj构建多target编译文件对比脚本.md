---
title: 基于xcodeproj构建多target编译文件对比脚本
date: 2019-01-11 14:09:46
tags:
	- iOS
	- XcodeProj
---

> 人力有时尽，物力有时穷

### 一、起

我们的项目工程主要用的target有两个，一个用于App Store版本的开发，一个用于企业包版本的开发。

![image-20190102112048386](https://ws4.sinaimg.cn/large/006tNbRwly1fys2oydw4fj30940a6wfo.jpg)

因为金唐支付业务需求，很久没打得企业包又重新上场了。线上打包很顺利，蓝色小灯亮起

![image-20190102135118700](https://ws4.sinaimg.cn/large/006tNbRwgy1fys71hkpzmj306h06t74k.jpg)

说明打包没问题，编译通过，心里喜滋滋。。。

但是运行的时候，缺失各种崩溃，我的内心也是崩溃的。。。



### 二、因

![image-20190102141406774](https://ws3.sinaimg.cn/large/006tNbRwgy1fys7pc23mwj31970u0b2a.jpg)



我们可以看到问题的原因是类的.m文件未勾选相应target导致的问题。当然这里也分两种情况：

1. **普通类，编译不会通过，打包时直接报错，易发现。**
2. **分类，编译通过，打包不会报错，运行时崩溃，难发现。**



**在我们新建分类文件的时候，默认是不勾选其他target。**如下图显示

![image-20190102150940858](https://ws2.sinaimg.cn/large/006tNbRwgy1fys9b1416bj30y50u04m6.jpg)

------

如果未勾选Enterprise Target,将不参加编译。

**第一种情况还好，最迟在打包的时候可以发现，细心一点自己提前build一下也可以提前发现。第二种情况如果在创建文件的时候忽略了勾选target，到后期绝对是灾难。**





### 三、思

那么怎么来避免这种情况的发生？虽然我一直强调的"创建文件一定要勾选全target",感觉没什么卵用。。。

后面想着对比下两个target的**Compile Sources**文件来着，一看到我们工程976个文件我就放弃了。

![image-20190102172210888](https://ws4.sinaimg.cn/large/006tNbRwgy1fysd4waz8fj31ba0l0n5a.jpg)





**那我就想有没有办法写个脚本将两个Target的Compile Sources文件分别读取出来，然后对比,输出差集。**这样就简单多了



### 四、解

我们在使用cocoapods的时候，有没有注意到，我们的工程项目被修改了。

![image-20190107084922219](https://ws3.sinaimg.cn/large/006tNc79gy1fyxrc95bu8j30ox09qwfe.jpg)

这说明cocoapods有操作（读写）工程文件的能力，在github上一搜还真找到了一个项目`CocoaPods/Xcodeproj`

#### cocoapods的Xcodeproj

在readme中可以看到，如何访问到target中的文件files，如下图

![image-20190107092120468](https://ws4.sinaimg.cn/large/006tNc79gy1fyxrc6kxioj30hl0i8gnm.jpg)





#### 求解思路

```ruby
#获取第一个target
targetA = project.targets[0]
filesA = targetA.source_build_phase.files.to_a.map do |pbx_build_file|
    pbx_build_file.file_ref.real_path.to_s
end

#获取第二个target
targetB = project.targets[1]
filesB = targetB.source_build_phase.files.to_a.map do |pbx_build_file|
    pbx_build_file.file_ref.real_path.to_s
end

#求差集
files = (filesA | filesB) - filesB

```







#### 最终实现的脚本：

```ruby
#!/usr/bin/ruby
#此脚本需要用到xcodeproj，如未安装，请先执行 gem install xcodeproj 安装xcodeproj。
require 'xcodeproj'
#比较资源文件开关 默认false不比较， true比较
resourcesCheck = false
#打开项目工程文件
project_path = '../Documents/code/workspace/TownCheer/GameCenter.xcodeproj'
project = Xcodeproj::Project.open(project_path)
#过滤出NativeTarget
native_targets = project.native_targets
#将第一个target做为默认对比的target
baseTarget = native_targets.first
baseTargetFiles = baseTarget.source_build_phase.files.to_a.map do |pbx_build_file|
	pbx_build_file.file_ref.real_path.to_s
end
baseTargetResource = baseTarget.resources_build_phase.files.to_a.map do |pbx_build_file|
    pbx_build_file.file_ref.real_path.to_s
end
#打印targets
puts native_targets
#计算target差集
native_targets.each do |target| 
    if target != baseTarget 
        #解析代码文件
        files = target.source_build_phase.files.to_a.map do |pbx_build_file|
            pbx_build_file.file_ref.real_path.to_s
        end
        fileset = (baseTargetFiles | files) - files
        puts '==============source===================='
        if (fileset.length > 0)
            puts "target #{target.name} 相对于 target #{baseTarget.name} 中未勾选以下文件:"
            puts fileset 
        else 
            puts "未找到"
        end
        #判断是否开启资源文件检查
        if resourcesCheck
            #解析资源文件
            resources = target.resources_build_phase.files.to_a.map do |pbx_build_file|
                pbx_build_file.file_ref.real_path.to_s
            end
            resourceset = (baseTargetResource | resources) - resources
            puts '==============resource===================='

            if (resourceset.length > 0)
                puts "target #{target.name} 相对于 target #{baseTarget.name} 中未勾选以下文件:"
                puts resourceset
            else 
                puts "未找到"
            end
        end
    end
end
```



#### 如何查询接口

下图是XcodeProj官方文档中提供的class list

![XcodeProj Doc](https://ws1.sinaimg.cn/large/006tNc79ly1fyy3lao888j30d40ok0v7.jpg)



查询第三方文档，推荐使用Dash.

演示Dash查询文档





#### ruby的一些简单语法

集合,交集和并集

```ruby
# & 为求交集，|为求并集

> arr_a = ["a", "b", "c"]
> arr_b = ["c", "d"]
> (arr_a|arr_b) - arr_a
> ["d"]
```



条件判断

```ruby
if conditional
      code
elsif conditional 
      code
else
      code
end
```

占位符

```
#{变量名称}
```

例

```ruby
 puts "局部变量的值为 #{i}"
```



### 五、xcode工程文件

#### .xcodeproj文件

Xcode的工程文件是 工程名.xcodeproj，而它其实是个package目录，通过显示包内容，可以查看到它内部主要有project.pbxproj 和 xcuserdata。其中，xcuserdata 一般是跟用户相关的一些设置，如断点 记录等，一般不用放到版本管理中。而project.pbxproj 是工程描述文件，描述了工程里的源码文件、schema设置等。



#### project.pbxproj

project.pbxproj文件存储着 Xcode 工程的各项配置参数,它本质上是一种旧风格的 Property List 文件，历史可追溯到 NeXT 的 OpenStep 。

对于NeXTSTEP格式的Property List 文件来说，整个的project.pbxproj文件就是一个字典，里面最外层有5个键值对，key分别为：

```
{
	archiveVersion = 1;  
	classes = {};
	objectVersion = 48;
	objects = {
        /* Begin PBXBuildFile section */
        ...
        /* End PBXBuildFile section */
        
        /* Begin PBXFileReference section */
        ...
        /* End PBXFileReference section */
        
        /* Begin PBXGroup section */
        ...
        /* End PBXGroup section */
        
        /* Begin PBXNativeTarget section */
        ...
        /* End PBXNativeTarget section */
        
        /* Begin PBXProject section */
        ...
        /* End PBXProject section */
	};  
	rootObject = 9A1EC9081FC2A4AD009772FC /* Project object */;
} 
```

其中重要的 Key 是 objects 和 rootObject。



**rootObject**像是一个入口函数，存放了一个PBXProject对象的UUID。

```
rootObject = 9A1EC9081FC2A4AD009772FC /* Project object */;
```



**PBXProject**

```
/* Begin PBXProject section */
		9A1EC9081FC2A4AD009772FC /* Project object */ = {
			isa = PBXProject;
			attributes = {
				LastUpgradeCheck = 0910;
				ORGANIZATIONNAME = unko;
				TargetAttributes = {
					9A1EC90F1FC2A4AD009772FC = {
						CreatedOnToolsVersion = 9.1;
						ProvisioningStyle = Manual;
					};
					9A1EC9271FC2A4AE009772FC = {
						CreatedOnToolsVersion = 9.1;
						ProvisioningStyle = Automatic;
						TestTargetID = 9A1EC90F1FC2A4AD009772FC;
					};
					9A1EC9321FC2A4AE009772FC = {
						CreatedOnToolsVersion = 9.1;
						ProvisioningStyle = Automatic;
						TestTargetID = 9A1EC90F1FC2A4AD009772FC;
					};
				};
			};
			buildConfigurationList = 9A1EC90B1FC2A4AD009772FC /* Build configuration list for PBXProject "TestDemo" */;
			compatibilityVersion = "Xcode 8.0";
			developmentRegion = en;
			hasScannedForEncodings = 0;
			knownRegions = (
				en,
				Base,
			);
			mainGroup = 9A1EC9071FC2A4AD009772FC;
			productRefGroup = 9A1EC9111FC2A4AD009772FC /* Products */;
			projectDirPath = "";
			projectRoot = "";
			targets = (
				9A1EC90F1FC2A4AD009772FC /* TestDemo */,
				9A1EC9271FC2A4AE009772FC /* TestDemoTests */,
				9A1EC9321FC2A4AE009772FC /* TestDemoUITests */,
			);
		};
/* End PBXProject section */
```



PBXProject顾名思义就是工程，对应构建可执行的二进制目标程序或库，里面包含了编译工程所需的全部信息。
1、isa 可以把每个value看做一个对象，可以发现每一个value都有一个isa字段，代表value的类型。
2、attributes 属性，包含一些编译器的基本信息，版本，以及项目中的target，每一个target一个UUID其中，Xcode自动创建的项目里面有三个target一个就是所要编译的APP主target，其余两个为test Target，可以看到其余两个target中有一个字段TestTargetID指向主target，可以理解为依赖相关吧。
3、buildConfigurationList 配置列表 指向一个配置字典 XCConfigurationList 类型类型（稍后讲）
4、compatibilityVersion 应该是兼容版本 目前看来是 Xcode 3.2
5、developmentRegion 语言版本，en英语
6、hasScannedForEncodings 是否已经扫描了文件编码信息
7、knownRegions 不同区域的本地资源文件列表
8、mainGroup Xcode的文件组织形式，可以理解为文件层次 PBXGroup 类型
9、productRefGroup 编译后的输出文件 PBXGroup 类型
10、projectDirPath，projectRoot 项目路径和项目的根目录 目前为空
11、targets 项目下的三个target对象 PBXNativeTarget类型



**XCConfigurationList**

```
/* Begin XCConfigurationList section */
		9A1EC90B1FC2A4AD009772FC /* Build configuration list for PBXProject "TestDemo" */ = {
			isa = XCConfigurationList;
			buildConfigurations = (
				9A1EC93A1FC2A4AE009772FC /* Debug */,
				9A1EC93B1FC2A4AE009772FC /* Release */,
			);
			defaultConfigurationIsVisible = 0;
			defaultConfigurationName = Release;
		};
		9A1EC93C1FC2A4AE009772FC /* Build configuration list for PBXNativeTarget "TestDemo" */ = {
			isa = XCConfigurationList;
			buildConfigurations = (
				9A1EC93D1FC2A4AE009772FC /* Debug */,
				9A1EC93E1FC2A4AE009772FC /* Release */,
			);
			defaultConfigurationIsVisible = 0;
			defaultConfigurationName = Release;
		};
		9A1EC93F1FC2A4AE009772FC /* Build configuration list for PBXNativeTarget "TestDemoTests" */ = {
			isa = XCConfigurationList;
			buildConfigurations = (
				9A1EC9401FC2A4AE009772FC /* Debug */,
				9A1EC9411FC2A4AE009772FC /* Release */,
			);
			defaultConfigurationIsVisible = 0;
			defaultConfigurationName = Release;
		};
		9A1EC9421FC2A4AE009772FC /* Build configuration list for PBXNativeTarget "TestDemoUITests" */ = {
			isa = XCConfigurationList;
			buildConfigurations = (
				9A1EC9431FC2A4AE009772FC /* Debug */,
				9A1EC9441FC2A4AE009772FC /* Release */,
			);
			defaultConfigurationIsVisible = 0;
			defaultConfigurationName = Release;
		};
/* End XCConfigurationList section */
```

XCConfigurationList是一个构建配置相关元素的列表，里面有一个项目文件（TestDemo），三个target（TestDemo、TestDemoTests和TestDemoUITests）对应于你在Xcode界面中看到的这样，如图，![image-20190108155018299](https://ws3.sinaimg.cn/large/006tNc79gy1fyz877i9gmj30ia0b1gqd.jpg)

每一个都有相对应的配置属性buildConfigurations，而每个配置属性都有两个的版本（Debug和Release）
那Debug和Release又对应什么呢？是XCBuildConfiguration：

**XCBuildConfiguration**

XCBuildConfiguration构建具体的配置元素，也就是Xcode的Build Setting面板中所涉及的选项。至于每个字段所代表的含义可再深入研究。



**PBXNativeTarget**

```
9A1EC90F1FC2A4AD009772FC /* TestDemo */ = {
			isa = PBXNativeTarget;
			buildConfigurationList = 9A1EC93C1FC2A4AE009772FC /* Build configuration list for PBXNativeTarget "TestDemo" */;
			buildPhases = (
				5D85ED4278F395AE0A1EC0E7 /* [CP] Check Pods Manifest.lock */,
				9A1EC90C1FC2A4AD009772FC /* Sources */,
				9A1EC90D1FC2A4AD009772FC /* Frameworks */,
				9A1EC90E1FC2A4AD009772FC /* Resources */,
				C91EA1773EF316C0A4208A7B /* [CP] Embed Pods Frameworks */,
				D96710C682564835D2F9EEC7 /* [CP] Copy Pods Resources */,
			);
			buildRules = (
			);
			dependencies = (
			);
			name = TestDemo;
			productName = TestDemo;
			productReference = 9A1EC9101FC2A4AD009772FC /* TestDemo.app */;
			productType = "com.apple.product-type.application";
		};
```



PBXNativeTarget，对应于生成的可执行二进制程序或库文件的本地构建目标对象。里面的主要内容是：
1、buildConfigurationList 这个我们在讲述PBXProject时已经介绍过，是一个配置相关元素相关的列表。
2、buildPhases 构建阶段所涉及的源文件（PBXSourcesBuildPhase类型）、框架（PBXFrameworksBuildPhase类型）和资源（PBXResourcesBuildPhase）以数组形式组织。
3、buildRules 指定了不同文件类型该如何编译
4、dependencies 列出了在Xcode build phase tab中列出的target依赖项
5、productReference 目标文件（PBXFileReference）

**PBXSourcesBuildPhase**

```
/* Begin PBXSourcesBuildPhase section */
		9A1EC90C1FC2A4AD009772FC /* Sources */ = {
			isa = PBXSourcesBuildPhase;
			buildActionMask = 2147483647;
			files = (
				...
				9A1EC9151FC2A4AD009772FC /* AppDelegate.m in Sources */,
			);
			runOnlyForDeploymentPostprocessing = 0;
		};
/* End PBXSourcesBuildPhase section */
```

PBXSourcesBuildPhase 代表构建阶段需要复制的资源文件 文件组织在key为files的value中的数组列表里，每个元素为PBXBuildFile类型。

**PBXBuildFile**

```
/* Begin PBXBuildFile section */
		9A1EC9151FC2A4AD009772FC /* AppDelegate.m in Sources */ = {isa = PBXBuildFile; fileRef = 9A1EC9141FC2A4AD009772FC /* AppDelegate.m */; };
		...
/* End PBXBuildFile section */
```

PBXBuildFile 代表文件元素，被PBXBuildPhase等作为文件包含或被引用的资源，其实里面fileRef指向最终的文件PBXFileReference。

**PBXFileReference**

```
/* Begin PBXFileReference section */
9A1EC9141FC2A4AD009772FC /* AppDelegate.m */ = {isa = PBXFileReference; lastKnownFileType = sourcecode.c.objc; path = AppDelegate.m; sourceTree = "<group>"; };
/* End PBXFileReference section */
```

项目所引用的每一个外部文件，比如源代码文件、资源文件、库文件、生成目标文件等，每一个外部文件都可以在这里找到，也就是在Xcode的文件导航栏所看到的文件组织形式。



**文件结构流程图**

![](https://upload-images.jianshu.io/upload_images/1933920-583c47f85ff24c6e.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

### 结尾

虽然事情的起因是因为代码文件没有勾选target，但是本次分享的目的是让大家了解project.pbxproj文件结构及实现脚本一些思路。

了解 project.pbxproj文件的结构，有利于我们在解决 project.pbxproj文件冲突的时候，不破坏project.pbxproj文件。

