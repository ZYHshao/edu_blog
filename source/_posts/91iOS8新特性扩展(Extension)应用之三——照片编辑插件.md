---
title: iOS8新特性扩展(Extension)应用之三——照片编辑插件
date: 2015-07-30
categories: iOS逻辑初窥
tags: []
---
##  iOS8新特性扩展(Extension)应用之三——照片编辑插件

        通过前几篇博客的介绍，我们了解到扩展给app提供的更加强大的交互能力，这种强大的交互能力另一方面体现在照片编辑插件的应用。

       和通常一样，我们先创建一个工程，然后新建一个Target，选择photo editing：

![](http://static.oschina.net/uploads/space/2015/0730/194713_QXfj_2340880.png)

从模板中，我们可以看到系统为我们创建了一个controller，这个controller就是用于处理照片的controller，其中方法如下：

```
- (BOOL)canHandleAdjustmentData:(PHAdjustmentData *)adjustmentData {
    // Inspect the adjustmentData to determine whether your extension can work with past edits.
    // (Typically, you use its formatIdentifier and formatVersion properties to do this.)
    return NO;
}
//这个函数用于从系统相册获取到选中的照片，contentEditingInput对象中存有响应的数据类型和image对象
- (void)startContentEditingWithInput:(PHContentEditingInput *)contentEditingInput placeholderImage:(UIImage *)placeholderImage {
   //我们可以在这里将取到的数据进行展示等等
    self.input = contentEditingInput;
}
//结束编辑照片时的方法
- (void)finishContentEditingWithCompletionHandler:(void (^)(PHContentEditingOutput *))completionHandler {
    // Update UI to reflect that editing has finished and output is being rendered.
    
    // Render and provide output on a background queue.
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // Create editing output from the editing input.
        PHContentEditingOutput *output = [[PHContentEditingOutput alloc] initWithContentEditingInput:self.input];
        
        //我们可以在这里将新的图片数据写入到输出流中
        // output.adjustmentData = <#new adjustment data#>;
        // NSData *renderedJPEGData = <#output JPEG#>;
        // [renderedJPEGData writeToURL:output.renderedContentURL atomically:YES];
        
        // Call completion handler to commit edit to Photos.
        completionHandler(output);
        
        // Clean up temporary files, etc.
    });
}
```

在当前扩展执行结束编辑之前，我们可以自由渲染我们得到的图片，例如添加相框，文字等等，输出时将渲染后的图片进行输出即可。

这里还有一个地方需要我们注意，此类扩展有一个功能，如果我们中途退出编辑，系统会为我们保存我们扩展的处理状态，为了区分多个类似功能的扩展，在输出数据的对象中有一个PHAdjustmentData类型的对象，这个对象专门用于负责版本的记录，这个对象中有如下两个属性用于区分版本：

[@property](http://my.oschina.net/property) (readonly, copy) NSString *formatIdentifier;

[@property](http://my.oschina.net/property) (readonly, copy) NSString *formatVersion;

  
 

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592
