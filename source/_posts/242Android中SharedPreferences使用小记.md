---
title: Android中SharedPreferences使用小记
date: 2016-08-03
categories: Android小记
tags: []
---
## Android中SharedPreferences使用小记

### 一、引言

        Android中使用SharedPreferences来进行简单数据的持久化处理，从名字就可以了解，其设计目的是为了保存应用程序的一些偏好设置，如音量，主题等信息。其与iOS开发中的NSUserDefault十分类似，并且，他们的实质都是采用XML格式的文件来存储数据。

### 二、SharedPreferences的简单应用

        对数据的持久化操作都会分为两个部分，一部分为存，另一部分为取。首先，开发者在Activity中使用如下方法可以获取获取创建一个SharedPreferences实例：

```java
/*
这个方法需要传入两个参数，第一个参数为文件名，第二个参数为文件模式
*/
SharedPreferences sharedPreferences = getSharedPreferences("MyPreference",MODE_PRIVATE);
```

在getSharedPreference()方法中第一个参数决定这个存储文件的名字，在获取SharedPreferences实例时，如果系统创建过这个文件，则会返回本地的原文件，如果没有这个文件，则会进行创建。第二个参数决定这个文件的访问权限，可选参数如下：

```
Activity.MODE_PRIVATE,//默认操作模式，代表该文件是私有数据，只能被应用本身访问
Activity.MODE_WORLD_READABLE,//表示当前文件可以被其他应用读取，  
Activity.MODE_WORLD_WRITEABLE,//表示当前文件可以被其他应用写入；
```

有了SharedPreferences实例，在需要进行数据存储时，需要获取到SharedPreferences实例中的Editor对象，SharedPreferences类中有一个Editor的内部接口，其中提供了存储数据的相关方法，示例代码如下:

```java
//获取Editor对象
SharedPreferences.Editor editor = sharedPreferences.edit();
//进行字符串存储
editor.putString("password","123456");
//提交存储内容
editor.commit();
```

Editor采用键值对的存储方式，可以存储的数据即常用方法如下：

```java
public interface Editor {
        //进行字符串数据存储
        SharedPreferences.Editor putString(String var1, String var2);
        //进行字符Set存储
        SharedPreferences.Editor putStringSet(String var1, Set<String> var2);
        //进行Int值存储
        SharedPreferences.Editor putInt(String var1, int var2);
        //进行Long值存储
        SharedPreferences.Editor putLong(String var1, long var2);
        //进行Float值存储
        SharedPreferences.Editor putFloat(String var1, float var2);
        //进行布尔值存储
        SharedPreferences.Editor putBoolean(String var1, boolean var2);
        //删除一个键 与其对应的值
        SharedPreferences.Editor remove(String var1);
        //清空所有数据
        SharedPreferences.Editor clear();
        //提交存储
        boolean commit();
        //提交存储请求
        void apply();
    }
```

上面的方法中，有两点需要注意，首先clear()方法是将所有的键的值清空，并没有删除键，而remove是删除键和值。第二点，commit()方法和apply()方法都用于提交数据，不同的是，commit()方法会直接将数据同步到磁盘，返回值会告知开发者是否同步成功，而apply()方法只是将数据存储在内存，之后异步进行存盘操作，没有返回值，在开发中，如果要保证数据立马存入磁盘，要使用commit()方法。

        对存储的数据进行读取，可以直接调用SharedPreferences实例的如下方法：

```java
public interface SharedPreferences {
    //获取所有键值映射表
    Map<String, ?> getAll();
    //通过键获取字符串值 第一个参数为键 第二个参数为此键不存在时使用的默认值
    String getString(String var1, String var2);
    //通过键获取字符串值集合 第一个参数为键 第二个参数为此键不存在时使用的默认值
    Set<String> getStringSet(String var1, Set<String> var2);
    //通过键获取整形值 第一个参数为键 第二个参数为此键不存在时使用的默认值
    int getInt(String var1, int var2);
    //通过键获取长整形值 第一个参数为键 第二个参数为此键不存在时使用的默认值
    long getLong(String var1, long var2);
    //通过键获取浮点值 第一个参数为键 第二个参数为此键不存在时使用的默认值
    float getFloat(String var1, float var2);
    //通过键获取布尔值 第一个参数为键 第二个参数为此键不存在时使用的默认值
    boolean getBoolean(String var1, boolean var2);
    //检查文件中是否包含某个键
    boolean contains(String var1);
    //注册监听
    void registerOnSharedPreferenceChangeListener(SharedPreferences.OnSharedPreferenceChangeListener var1);
    //取消注册监听
    void unregisterOnSharedPreferenceChangeListener(SharedPreferences.OnSharedPreferenceChangeListener var1);
    public interface OnSharedPreferenceChangeListener {
        void onSharedPreferenceChanged(SharedPreferences var1, String var2);
    }
}

```

注册监听方法可以提供给开发者一个回调接口，当SharedPreferences中数据改变时，会通知给开发者进行逻辑处理，示例代码如下：

```java
//创建监听者
 final SharedPreferences.OnSharedPreferenceChangeListener listener = new SharedPreferences.OnSharedPreferenceChangeListener() {
            //需要重写这个方法 这个方法中会传入发生变化的键s
            @Override
            public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String s) {
                Log.d("**********",s);
            }
        };
//进行注册
sharedPreferences.registerOnSharedPreferenceChangeListener(listener);
```

温馨提示：可以在Android Device Monitor中查看创建的SharedPreferences文件，路径为data/data/APP包名/shared_prefs目录下，可以看到其为XML文件，如下图：

![](http://static.oschina.net/uploads/space/2016/0803/233659_bHSn_2340880.png)

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：435043639
