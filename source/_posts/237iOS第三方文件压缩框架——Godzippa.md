---
title: iOS第三方文件压缩框架——Godzippa
date: 2016-07-11
categories: iOS第三方库
tags: []
---
## iOS第三方文件压缩框架——Godzippa

    Godzippa是iOS开发中常用的一个第三方数据压缩框架，其采用类别的方式，为NSData类与NSFileManager类提供了压缩和解压缩数据的方法。

    Godzippa的github地址如下：[https://github.com/mattt/Godzippa](https://github.com/mattt/Godzippa)。

    NSData类别中提供的方法如下：

```objectivec
//进行数据压缩操作
- (NSData *)dataByGZipCompressingWithError:(NSError * __autoreleasing *)error;
//进行数据压缩操作，支持配置缓存区大小，压缩比等参数
- (NSData *)dataByGZipCompressingAtLevel:(int)level
                              windowSize:(int)windowBits
                             memoryLevel:(int)memLevel
                                strategy:(int)strategy
                                   error:(NSError * __autoreleasing *)error;
//进行数据解压缩操作
- (NSData *)dataByGZipDecompressingDataWithError:(NSError * __autoreleasing *)error;
- (NSData *)dataByGZipDecompressingDataWithWindowSize:(int)windowBits
                                                error:(NSError * __autoreleasing *)error;
```

    NSFileManager类别中提供的方法如下：

```objectivec
//压缩文件并写入磁盘 返回值确定压缩操作是否成功
- (BOOL)GZipCompressFile:(NSURL *)sourceFile
   writingContentsToFile:(NSURL *)destinationFile
                   error:(NSError * __autoreleasing *)error;
//进行文件压缩，支持配置压缩级别
- (BOOL)GZipCompressFile:(NSURL *)sourceFile
   writingContentsToFile:(NSURL *)destinationFile
                 atLevel:(int)level
                   error:(NSError *__autoreleasing *)error;
//进行文件的解压缩
- (BOOL)GZipDecompressFile:(NSURL *)sourceFile
     writingContentsToFile:(NSURL *)destinationFile
                     error:(NSError * __autoreleasing *)error;
```

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：203317592
