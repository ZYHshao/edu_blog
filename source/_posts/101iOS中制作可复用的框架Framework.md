---
title: iOS中制作可复用的框架Framework
date: 2015-08-12
categories: 代码优化
tags: []
---
## iOS中制作可复用的框架Framework

        在iOS开发中，我们时常会使用一些我们封装好的管理类，框架类，方法类等，我们在实现这些文件时，可能还会依赖一些第三方库或者系统库。如果每次我们复用这些代码时，都要将关联的这些东西进行导入，甚至还要进行arc和mrc的编译设置，会浪费我们很大的精力。除此之外，如果项目需要多人合作，你可能也并不希望你的源代码暴漏在所有人的面前，这个时候，我们就可以使用静态库或者动态库的方式来对我们的代码进行包装，便于复用。静态库的制作方法在一篇旧的博客中有描述：[http://my.oschina.net/u/2340880/blog/398887](http://my.oschina.net/u/2340880/blog/398887)。相比静态库文件，动态库的效率会更高且封装性更好，这里主要讨论动态库的制作。

        xcode6后支持在xcode中制作动态库，并且过程也十分简单。

        新建一个项目，选择framework：

![](http://static.oschina.net/uploads/space/2015/0812/150329_Oflx_2340880.png)

        之后我们在里面编写我们的代码，比如我们创建一个MyObject类：

```
@interface MyObject : NSObject
-(void)myLog;
@end

@implementation MyObject
-(void)myLog{
    NSLog(@"framework");
}
@end
```

        和静态库类似，如果我们不做任何处理，打包出来的库文件只能在模拟器或者只能在真机上使用，为了方便我们调试，我们可以添加一个脚本命令，是的生成一个同时支持模拟器和真机的framework：

        新建target：

![](http://static.oschina.net/uploads/space/2015/0812/151005_aEul_2340880.png)

        选择Aggregate：

![](http://static.oschina.net/uploads/space/2015/0812/151058_h0Zy_2340880.png)

        之后，我们在target的Build Phases中点击加号：

![](http://static.oschina.net/uploads/space/2015/0812/151246_pU2n_2340880.png)

        添加一个Run Script：

![](http://static.oschina.net/uploads/space/2015/0812/151553_6MXT_2340880.png)

        在里面添加如下的脚本：

![](http://static.oschina.net/uploads/space/2015/0812/151747_D1sw_2340880.png)

```
set -e
set +u
# Avoid recursively calling this script.
if [[ $SF_MASTER_SCRIPT_RUNNING ]]
then
exit 0
fi
set -u
export SF_MASTER_SCRIPT_RUNNING=1

SF_TARGET_NAME=${PROJECT_NAME}
SF_EXECUTABLE_PATH="${SF_TARGET_NAME}.framework/${SF_TARGET_NAME}"
SF_WRAPPER_NAME="${SF_TARGET_NAME}.framework"

if [[ "$SDK_NAME" =~ ([A-Za-z]+) ]]
then
SF_SDK_PLATFORM=${BASH_REMATCH[1]}
else
echo "Could not find platform name from SDK_NAME: $SDK_NAME"
exit 1
fi

if [[ "$SDK_NAME" =~ ([0-9]+.*$) ]]
then
SF_SDK_VERSION=${BASH_REMATCH[1]}
else
echo "Could not find sdk version from SDK_NAME: $SDK_NAME"
exit 1
fi

if [[ "$SF_SDK_PLATFORM" = "iphoneos" ]]
then
SF_OTHER_PLATFORM=iphonesimulator
else
SF_OTHER_PLATFORM=iphoneos
fi

if [[ "$BUILT_PRODUCTS_DIR" =~ (.*)$SF_SDK_PLATFORM$ ]]
then
SF_OTHER_BUILT_PRODUCTS_DIR="${BASH_REMATCH[1]}${SF_OTHER_PLATFORM}"
else
echo "Could not find platform name from build products directory: $BUILT_PRODUCTS_DIR"
exit 1
fi

rm -rf buildProducts
mkdir buildProducts

# Build the other platform.
xcrun xcodebuild -project "${PROJECT_FILE_PATH}" -target "${TARGET_NAME}" -configuration "${CONFIGURATION}" -sdk ${SF_OTHER_PLATFORM}${SF_SDK_VERSION} BUILD_DIR="${BUILD_DIR}" OBJROOT="${OBJROOT}" BUILD_ROOT="${BUILD_ROOT}" SYMROOT="${SYMROOT}" $ACTION

# Smash the two static libraries into one fat binary and store it in the .framework
xcrun lipo -create "${BUILT_PRODUCTS_DIR}/$PRODUCT_NAME.framework/$PRODUCT_NAME" "${SF_OTHER_BUILT_PRODUCTS_DIR}/$PRODUCT_NAME.framework/$PRODUCT_NAME" -output "${PROJECT_DIR}/buildProducts/$PRODUCT_NAME"

cp -rf ${BUILT_PRODUCTS_DIR}/$PRODUCT_NAME.framework ${PROJECT_DIR}/buildProducts
mv ${PROJECT_DIR}/buildProducts/$PRODUCT_NAME ${PROJECT_DIR}/buildProducts/$PRODUCT_NAME.framework
```

接着，我们需要将给外界的接口文件暴露出来，将其移动到public下即可：

![](http://static.oschina.net/uploads/space/2015/0812/153022_kn2X_2340880.png)

之后我们运行程序，需要注意的一点事，如果要支持64位，需要在编译选项中设置，如下：

![](http://static.oschina.net/uploads/space/2015/0812/152120_msOv_2340880.png)

到此时，我们的framework库文件就制作完成，在xcode的window->projects中选中我们的这个项目，点击进入文件夹的小箭头：

![](http://static.oschina.net/uploads/space/2015/0812/152512_GgAO_2340880.png)

![](http://static.oschina.net/uploads/space/2015/0812/152512_29At_2340880.png)

在build->product中便可以找到我们的framework文件，我们将其赋值出来即可以使用。

![](http://static.oschina.net/uploads/space/2015/0812/152719_n80M_2340880.png)

 我们测试一下，新建一个工程，将刚才制作的静态库导入，如下加入头文件，调用方法，可以使用。

```
#import <MyFramework/MyObject.h>
 MyObject * obj = [[MyObject alloc]init];
    [obj myLog];
```

两个技巧：

一、如果你运行程序出现类似Reason: image not found!的崩溃信息，可能的原因是动态库文件中的某些文件你的项目中已经包含了，在Build Phases中将required改成optional即可。

二、一个优秀且完整的框架可能会包含相当多的文件，包括框架自己的和其他第三方的，为了使用的方便，我们可以将头文件都导入一个的头文件中，这里有一个地方我们需要注意，我们直接在framework工程中添加的头文件是不会编译的，我的解决方案是通过建一个OC的类，在这个类中导入这个总的头文件，将这个类隐藏成私有的，就可以解决问题了。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592
