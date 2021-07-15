---
title: iOS14新特性探索之二：App Widget小组件应用
date: 2020-07-10
categories: iOS14专题
tags: []
---
# iOS14新特性探索之二：App Widget小组件应用

        iOS 14除了引入了亮眼的App Clips功能外。还有一个也非常惹争议的功能就是App Widget。App Widget可以理解为小组件，在非常早的Android版本中就有了Widget的概念，应用开发者可以为系统开发自己应用相契合的Widget来让用户更加方便的使用应用提供的功能。例如Android早期系统中非常常见的钟表时间组件、快捷设置组件等。用户可以将这些小组件根据自己的喜好放在屏幕的指定位置。从这点看，iOS 14提供的App Widget功能的确不能算是一种创新，最多算是一种增强。

        其实，iOS Widget的概念并非是iOS 14突然引入的，在iOS 10发布时，iOS系统就引入了Extension相关功能，其中有一种Extension叫做Today Extension，这就是iOS 14中Widget的前身。Today Extension允许开发者为负一屏开发快捷功能入口。关于Today Extension的应用，如下博客有详细的介绍：

iOS8新特性扩展(Extension)应用之一——Today扩展：[https://my.oschina.net/u/2340880/blog/485533](https://my.oschina.net/u/2340880/blog/485533)

iOS中Today扩展插件与宿主APP的交互：[https://my.oschina.net/u/2340880/blog/711807](https://my.oschina.net/u/2340880/blog/711807)

需要注意，在iOS 14中，Today Extension相关的接口都已经被废弃，我们需要使用新的WidgetKit框架提供的小组件接口开发Widget。在iOS 14上，Today Extension依然可以使用，但是其功能受限，只能在负一屏展示它，用户不能随意的将其放在指定屏的指定位置。

## 1\. 关于App Widget

        Widget为应用程序提供了这样一种功能：其可以让用户在主屏幕上展示App中用户所关心的信息。例如一款天气软件，其可以附带一个Widget让用户在主屏幕就可查看今日的天气情况，例如股票相关的软件，用户将自己感兴趣的股票收藏，无需打开App，在主屏幕即可查到对应的股价信息。如下图所示，是系统提供的电池Widget展示在主屏幕上的示例：

![](https://oscimg.oschina.net/oscnet/up-52598d617b5bd87fde0fce8cf7e65898396.png)

一个App也可以提供多个Widget组件，用户可以选择将其最关心的放置在最重要的位置上，以便最方便的获取信息。对于同一种Widget组件，开发者也可以提供不同的尺寸或不同的布局，这可以提供给用户更多的选择以满足不同用户的偏好。

        为应用程序添加一个Widget组件并不复杂，但是有一点需要注意，小组件的UI部分只能够使用SwiftUI来开发，因此如果你要开发Widget组件，必须有一些Swift的基础并对SwiftUI有一定的了解。对于Swift与SwiftUI的相关内容，本篇博客就不再做过多赘述。

## 2\. 创建App Widget

        与其他的Extension扩展类似，App Widget本身也是一种扩展，因此其只能依赖一个宿主App而存在，首先向已有的App中添加App Widget非常简单，为项目创建一个新的Target，选择其中的Widget Extension模板进行创建，如下图：

![](https://oscimg.oschina.net/oscnet/up-495035e7c9d511ca945f00d450464be4fbf.png)        

创建完成后，Xcode会自动帮我们创建和配置的文件的工作都完成，默认的模板为我们创建了一个显示当前时间的组件，我们可以直接在真机上运行它(Bate版本的Xcode模拟器运行会有些异常)，之后，我们就可以将这个显示时间的小组件放置在主屏幕的任意位置，并且，默认提供了3种尺寸供用户选择，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-0380fbebb30effcd981bbe5082ec06f63f6.png)

Xcode为我们创建的这个模板虽然简单，但是五脏俱全。Widget加载的入口是@main标记的结构体，代码如下：

```swift
@main
struct WidgetExt: Widget {
    private let kind: String = "WidgetExt"

    public var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider(), placeholder: PlaceholderView()) { entry in
            WidgetExtEntryView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}
```

WidgetExt是我们为组件target项目设置的名字，模板自动使用这个名字帮我们生成了一个实现了Widget协议的结构体。结构体中实现了两个属性，其实Widget协议提供的核心只读属性只有一个body，将上面的代码改写如下也是一样的：

```swift
@main
struct WidgetExt: Widget {
    public var body: some WidgetConfiguration {
        StaticConfiguration(kind: "WidgetExt", provider: Provider(), placeholder: PlaceholderView()) { entry in
            WidgetExtEntryView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}
```

上面代码的核心在于body只读属性的实现，其需要返回一个实现了WidgetConfiguration协议的示例。这个协议描述了组件的配置信息，StaticConfiguration是系统提供的组件配置结构体，其用来对静态类型的组件提供配置。StaticConfiguration完整的构造方法如下：

```swift
public init<Provider, PlaceholderContent>(
kind: String, 
provider: Provider, 
placeholder: PlaceholderContent, 
content: @escaping (Provider.Entry) -> Content) 
where Provider : TimelineProvider, PlaceholderContent : View
```

可以看到，上面构造方法中的Provider和PlaceholderContent实际上是两个泛型，我们后面再介绍。目前，我们先关注下构造方法需要传的几个参数。

-   kind：这个参数是一个字符号，我们可以任意提供，用来标识这个Widget组件。
-   provider：简单理解，这是一个数据提供对象，用来为小组件提供渲染数据，其必须实现TimelineProvider协议，即是基于时间线来驱动小组件的渲染。
-   placeholder：提供一个占位的视图，当小组件没有数据或者在锁屏状态时，会显示这个占位视图。
-   content：为小组件提供内容，是一个闭包，其中会把Provider的entry属性传入，因此小组件的视图渲染实际是由Provider驱动的。

    明白了上面几个参数的意义，开发小组件就非常轻松了。首先，需要创建一个合适的Provider来为小组件提供数据支持，以模板中的代码为例，如下：

```swift
struct Provider: TimelineProvider {
    public typealias Entry = SimpleEntry

    public func snapshot(with context: Context, completion: @escaping (SimpleEntry) -> ()) {
        let entry = SimpleEntry(date: Date())
        completion(entry)
    }

    public func timeline(with context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [SimpleEntry] = []

        // Generate a timeline consisting of five entries an hour apart, starting from the current date.
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            let entry = SimpleEntry(date: entryDate)
            entries.append(entry)
        }

        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}

struct SimpleEntry: TimelineEntry {
    public let date: Date
}
```

如上代码所示，Provider结构体实现了TimelineProvider协议，这个协议中只定义了两个方法，分别是上面实现的snapshot方法和timeline方法。

        其中snapshop方法在小组件启动时会被调用一次，用来为小组件提供首屏渲染所需要的数据，其通常用来提供一些初始化的数据。调用完snapshot方法后，会调用timeline方法来定义要更新组件的时间线，这个方法的回调中需要传入一组Timeline对象，如上代码所示，其定义当前时刻开始，每隔一个小时进行一次刷新，将当前组件显示的时间刷新成最新的时刻，当最后一次刷新任务结束后，会再次调用timeline函数重新设置一组更新的时间线。关于时间线的详细介绍，后面会提及。

        有了Provider来对组件的更新提供驱动后，就是小组件页面的渲染了，在StaticConfiguration构造方法的闭包中，我们需要返回一个View作为小组件的内容，模板提供的示例代码如下：

```swift
struct WidgetExtEntryView : View {
    var entry: Provider.Entry

    var body: some View {
        Text(entry.date, style: .time)
    }
}
```

        在向主屏幕添加小组件时，用户可以选择不同尺寸的小组件进行添加，在小组件的渲染布局时，开发者也可以根据不同的环境尺寸配置不同的渲染策略，例如下面代码：

```swift
struct WidgetExtEntryView : View {
    @Environment(\.widgetFamily) var family: WidgetFamily
    
    var entry: Provider.Entry
    
    @ViewBuilder
    var body: some View {
        switch family {
        case .systemSmall: Text(entry.date, style: .time)
        case .systemMedium: Text(entry.date, style: .date)
        case .systemLarge: Text(entry.date, style: .relative)
        default: Text(entry.date, style: .time)
        }
    }
}
```

其中通过Enviroment用来判断当前组件的环境情况，即组件的尺寸信息，上面代码根据不同的尺寸渲染了不同格式的时间。

      现在，我们对小组件的创建流程已经有了初步的了解，需要注意，小组件只能用来展示静态的信息，并能支持可交互的组件，例如选择器或滚动视图，当用户点击小组件时，会唤起App本身，并传递一个特殊的URL用来给宿主App做逻辑处理。一个App只能创建一个App Widget，但这并不是说我们只能有一种功能类型的组件，可以通过定义组件包，来提供多个小组件供用户进行使用，示例如下：

```swift
struct WidgetExt: Widget {
    public var body: some WidgetConfiguration {
        StaticConfiguration(kind: "WidgetExt", provider: Provider(), placeholder: PlaceholderView()) { entry in
            WidgetExtEntryView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}

struct WidgetExt2: Widget {
    public var body: some WidgetConfiguration {
        StaticConfiguration(kind: "WidgetExt2", provider: Provider(), placeholder: PlaceholderView()) { entry in
            PlaceholderView()
        }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}

@main
struct WidgetsExt: WidgetBundle {
    @WidgetBundleBuilder
    var body: some Widget {
        WidgetExt()
        WidgetExt2()
    }
}

```

需要注意，不同的小组件定义的kind参数要有差异。

## 3\. App Widget 的更新机制

        通过前面的Widget初体验，我们知道App Widget可以通过定义时间线来实现视图的动态更新。App Widget使用SwiftUI来进行视图的渲染。Widget有单独的系统进程进行维护，因此即便小组件已经显示在屏幕上，其也并不是一直都是活跃的，开发者可以定义一些时机来对小组件的内容进行更新。

        首先，在开发小组件时，我们要清楚所需要的更新时机。例如对于天气类小组件，可能需要每3小时对组件进行一次更新。当我们定义小组件Widget时，需要指定一个TimelineProvider来对其更新进行驱动，TimelineProvider可以理解为定义了一条时间线，配合官方文档中的一张图片来理解时间线的作用会比较容易：

![](https://oscimg.oschina.net/oscnet/up-1478d7089b5ed99b1c76a16dfb7503cfd35.png)

如上图中所示，其定义时间线为之后每小时进行刷新，由于将时间线的Refresh机制设置为了atEnd，3小时后系统会重新请求新的Timeline策略，上图中将第2次请求Timeline策略是设置为了立即刷新一次，之后由于时间线的Refresh机制设置为了never，之后不会再尝试请求时间线进行组件更新。时间轴的Refresh选项实际上是设置了当已经定义的时间轴执行完成后，系统将采用怎样的策略(是重新请求还是从此结束更新)。例如下图：

![](https://oscimg.oschina.net/oscnet/up-731f97600bc70b728be9f3bcf17a764e13d.png)

上图描述了这样一种逻辑，首先请求的时间线定义在未来3个小时，每小时更新一次，并在2小时候重新请求时间线，2小时后新请求的时间线定义2小时后刷新Widget并指定了2小时候重新请求时间线，再2小时之后，重新请求的时间线定义立即刷新组件，并指定之后不再请求新的时间线，组件刷新从此结束。

        除了通过设置Timeline的Refresh机制让Widget请求时间线来进行刷新机制的定义外，宿主App也可以对Widget的刷新机制进行定义。宿主App可以使用WidgetCenter来触发指定Widget的刷新机制更新，如下：

```swift
WidgetCenter.shared.reloadTimelines(ofKind: "指定的widget的kind")
```

同样，WidgetCenter目前也只能使用Swift来调用。

        顺便提一下，关于WidgetCenter，其本身非常简单，提供的接口非常精简，如下：

```objectivec
// 获取单例对象
static let shared: WidgetCenter 
// 获取当前Widgets的用户自定义配置
/*
struct WidgetInfo {
    public let configuration: INIntent?
    public let family: WidgetFamily
    public let kind: String
}
*/
func getCurrentConfigurations((Result<[WidgetInfo], Error>) -> ())
// 重新刷新某个Widget的时间线
func reloadTimelines(ofKind: String)
// 刷新所有Widget的时间线
func reloadAllTimelines()

```

## 4\. 可配置的Widget组件

        前面我们所介绍的构建小组件的方式，虽然可以通过时间线做部分更新逻辑，但对用户来说，依然是静态的。用户不能够根据自己的偏好对组件进行配置，还以天气类组件为例，有些用户可能关心的是空气质量，湿度等信息，有些用户可能只关心阴天雨天的信息，由于小组件的显示空间有限，有时候你无法将所有的信息都展示在组件内，因此让用户选择他感兴趣的信息进行小组件的配置非常重要。

        首先，如果要让我们开发的Widget可以支持用户配置，需要在Widget的target工程中添加一个配置属性表文件，使用Xcode新建一个SiriKit Intent Definition File的文件，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-3433386bddd1810550f32ed78286502d957.png)

之后，需要创建一个新的Intent配置，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-9efc2344a7868b066bbd136025d9a18c35f.png)

之后，我们可以添加一系列的用户配置项，系统提供了各种类型的配置项，如让用户传入字符串信息的配置项，开关配置项，日期配置项等等，如下图：

![](https://oscimg.oschina.net/oscnet/up-57324c415cb93acd3504309761c7b29290b.png)

之后，重新运行Widget，我们的小组件就以支持用户配置功能，用户可以编辑小组件进行设置，如下图所示：

![](https://oscimg.oschina.net/oscnet/up-322a207eb6885d71a3d4628f7e10ecd74b9.png)

当用户修改了配置项后，组件会重新请求Timeline时间线，在timeline回调方法中，会传入configuration对象，用来存储用户的配置信息，如下：

```swift
    public func timeline(for configuration: ConfigurationIntent, with context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        // configuration中存放用户配置信息
        var entries: [SimpleEntry] = []
        let currentDate = Date()
        for hourOffset in 0 ..< 5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
            let entry = SimpleEntry(date: entryDate, configuration: configuration)
            entries.append(entry)
        }

        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
```

上面演示的这种配置方式，适用于当配置项固定的场景，更多时候，可能连配置项都是动态的，比如我们的应用会根据服务端的状态来提供不同的服务，这时可提供给用户开启的服务项目就是动态的。Widget的配置项也支持动态进行配置，这需要使用到Intents Extension的相关功能，本篇博客就不再过多介绍。

## 结语：

        App Widgets本身并没有什么新意，只是扩大了iOS系统中组件的能力，这从一定程度上可以带给用户更好的服务和更多元的交互体验。脱离App Widgets这个功能的产品意义本身，iOS 14推出这个功能还有一点非常令人惊讶，就是App Widgets只能使用SwiftUI进行开发，这或许从另一个角度暗示了Swift在未来的推广力度，与iOS开发所使用语言的最终方向。

> 专注技术，热爱生活，交流技术，也做朋友。
> 
> ——珲少 QQ群：805263726
