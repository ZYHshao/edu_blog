---
title: ReactNative应用之汇率换算器开发全解析
date: 2016-12-26
categories: ReactNative
tags: []
---
## ReactNative应用之汇率换算器开发全解析

### 一、引言

    本篇博客将介绍如何开发一款简易的ReactNative小应用汇率换算器。本应用仅作为学习使用，其支持在人民币与美元间进行汇率计算。汇率计算器应用主要分为两部分：键盘与显示屏。键盘提供给与用户进行输入，在显示屏上进行汇率换算结果的显示。复杂的界面无非是简单组件的组合使用，因此，在进行开发之前，我们可以思考可能需要使用到的独立组件的开发，例如键盘按钮的开发，有键盘按钮组成的键盘的开发，显示屏开发等。首先创建一个初始的ReactNative工程，将index.ios.js与index.android.js文件中的内容全部删掉。在项目根目录中新建4个目录，分别为const、controller、image和view。这4个目录用于存放后面我们需要新建的静态文件，控制器文件，图片素材和视图文件。

### 二、用户键盘的封装

    在view文件夹下新建一个KeyButton.js文件，其用来创建键盘上的独立按钮，将其实现如下：

```javascript
import React, { Component,PropTypes } from 'react';
import { 
    TouchableHighlight,
    Text,
    StyleSheet,
    Image
} from 'react-native';

export default class KeyButton extends Component{
    render(){
        //根据number属性来做判断
        if (this.props.number == '删') {
            return(
            <TouchableHighlight 
            underlayColor='#f06d30' 
            style={[keyButtonStyles.buttonStyleNormal,{alignItems:'center'}]}
            onPress={this.props.buttonPress.bind(this,this.props.number)}>
                <Image source={require('../image/delete.png')}/>
            </TouchableHighlight>
            );
        };
        //特殊按钮需要状态来判断
        var buttonStyle;
        if(this.props.number == 'C' || this.props.number == '='){
            buttonStyle = keyButtonStyles.buttonStyleSpecial;
        }else{
            buttonStyle =  keyButtonStyles.buttonStyleNormal;
        }
        return(
            <TouchableHighlight 
            underlayColor='#f06d30' 
            style={buttonStyle}
            onPress={this.props.buttonPress.bind(this,this.props.number)}>
                <Text style={keyButtonStyles.textStyle}>{this.props.number}</Text>
            </TouchableHighlight>
        );
    }
}
//配置样式列表
const keyButtonStyles = StyleSheet.create({
    //正常按钮样式
    buttonStyleNormal:{
        flex:1,
        backgroundColor:'#323637',
        justifyContent: 'center',
    },
    //特殊按钮样式
    buttonStyleSpecial:{
        flex:1,
        backgroundColor:'#f06d30',
        justifyContent: 'center',
    },
    //文本样式
    textStyle:{
        color:'white',
        textAlign:'center',
        fontSize:30
    }
});
```

上面代码中预留number属性作为按钮的标题，不同的按钮可能带有不同的样式，同样通过这个属性来做区分。按钮的触发事件绑定给了buttonPress属性，并且在按钮触发执行时，将按钮的number属性传递出去。

    在const文件夹下创建一个Const,js文件，这个文件中用来定义全局的一些样式，实现如下：

```javascript
import React, { Component } from 'react';
import { 
    StyleSheet
     } from 'react-native';


export default mainViewStyles = StyleSheet.create({
    //主界面容器的样式
    container:{
        flex:1,
        flexDirection:'column',
        backgroundColor:'black'
    }, 
    //显示屏的样式
    screenView:{
        flex:3,
        backgroundColor:'#f06d30'
    },
    //键盘的样式
    keyboardView:{
        flex:7,
        backgroundColor:'#323637'
    }
});
```

    在View文件夹下新建一个KeyboardView.js文件，将其作为键盘视图类，将其实现如下：

```javascript
import React, { Component } from 'react';
import { 
    View, 
    Text,
    StyleSheet
     } from 'react-native';
import KeyButton from './KeyButton';
import mainViewStyles from '../const/Const';
export default class KeyboardView extends Component {
    render(){
        return(
            <View style={mainViewStyles.keyboardView}>
                <View style={keyboardViewStyles.keyboardRow}>
                    <KeyButton number='1' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='2' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='3' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='删' buttonPress={this.props.buttonPress}/>
                </View>
                <View style={keyboardViewStyles.keyboardRow}>
                    <KeyButton number='4' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='5' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='6' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='0' buttonPress={this.props.buttonPress}/>
                </View>
                <View style={keyboardViewStyles.keyboardRow}>
                    <KeyButton number='7' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='8' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='9' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='.' buttonPress={this.props.buttonPress}/>
                </View>
                <View style={keyboardViewStyles.keyboardRow}>
                    <KeyButton number='C' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='-' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='+' buttonPress={this.props.buttonPress}/>
                    <KeyButton number='=' buttonPress={this.props.buttonPress}/>
                </View>
            </View>
        );
    }
}
const keyboardViewStyles = StyleSheet.create({
    keyboardRow:{
        flex:1,
        backgroundColor:'black',
        flexDirection:'row'
    }
});

```

上面以九宫格的布局模式创建了16个功能按钮，并且将按钮的点击事件属性绑定给键盘的buttonPress属性，由上层视图来做处理，这里使用了flex权重的布局模式。

    至此，键盘部分先告一段落，我们需要对显示屏进行开发，在View文件夹下新建一个ScreenView.js文件，将其作为显示屏视图类，显示屏类和键盘比起来要复杂许多，因为其要实现各种屏幕操作功能，例如回退，删除，清空，计算等，实现如下：

```javascript
import React, { Component } from 'react';
import { 
    View, 
    Text,
    StyleSheet,
    TouchableHighlight,
    Image
     } from 'react-native';
import mainViewStyles from '../const/Const';
export default class ScreenView extends Component {
    constructor(props) {
      super(props);
      //这三个状态分别用来标记是美元转人民币或者人民币转美元，美元数，人民币数
      this.state = {
          transUS:true,
          USMoney:'0',
        CHMoney:'0'
      };
      let __this = this;
      //进行美元转人民币或者人民币转美元转换时调用
      this.change = function(){
        __this.setState({transUS:!__this.state.transUS});
      };
      //数字类按钮被点击触发的方法
      this.buttonClick = function(count){
          var us=__this.state.USMoney,ch=__this.state.CHMoney;
          if (__this.state.transUS) {
                  if (us=='0') {us=count;}else{us=__this.state.USMoney+count;};
              }else{
                  if (ch=='0') {ch=count;}else{ch=__this.state.CHMoney+count;};
              };
          __this.setState({
              USMoney:us,
              CHMoney:ch
          });
      };
      //清除按钮被点击触发的方法
      this.clear = function(){
          __this.setState({
              USMoney:'0',
              CHMoney:'0'
          });
      };
      //回退按钮被点击触发的方法
      this.delete = function(){
        if (__this.state.transUS) {
            if (__this.state.USMoney.length>1) {
                let newUS = __this.state.USMoney.substring(0,__this.state.USMoney.length-1);
                __this.setState({USMoney:newUS});
            }else if(__this.state.USMoney>'0'){
                __this.setState({USMoney:'0'});
            }
        }else{
            if (__this.state.CHMoney.length>1) {
                let newCH = __this.state.CHMoney.substring(0,__this.state.CHMoney.length-1);
                __this.setState({CHMoney:newCH});
            }else if(__this.state.CHMoney>'0'){
                __this.setState({CHMoney:'0'});
            }
        }
      };
      //减号触发的方法
      this.sub = function(){
          if (__this.state.transUS) {
           if(__this.state.USMoney>'0'){
                   let newUS = (parseInt(__this.state.USMoney)-1).toString();
                __this.setState({USMoney:newUS});
            }
        }else{
            if(__this.state.CHMoney>'0'){
                let newCH = (parseInt(__this.state.CHMoney)-1).toString();
                __this.setState({CHMoney:newCH});
            }
        }
      };
      //加号触发的方法
      this.add = function(){
          if (__this.state.transUS) {
           let newUS = (parseFloat(__this.state.USMoney)+1).toString();
            __this.setState({USMoney:newUS});
        }else{
            let newCH = (parseFloat(__this.state.CHMoney)+1).toString();
            __this.setState({CHMoney:newCH});
        }
      };
      //进行汇率转换
      this.trans = function(){
          if (__this.state.transUS) {
           let us = parseFloat(__this.state.USMoney);
           __this.setState({CHMoney:(us*6.7).toFixed(2).toString()});
        }else{
            let ch = parseFloat(__this.state.CHMoney);
           __this.setState({USMoney:(ch/6.7).toFixed(2).toString()});
        }
      }
    }

    render(){
        let transText = this.state.transUS?"从美元":"从人民币";
        let transToText = this.state.transUS?"到人民币":"到美元";
        let topText = this.state.transUS?this.state.USMoney+'$':this.state.CHMoney+'￥';
        let bottomText = this.state.transUS?this.state.CHMoney+'￥':this.state.USMoney+'$';
        return(
            <View style={mainViewStyles.screenView}>
                <Text style={{
                    color:'white',
                    textAlign:'right',
                    marginRight:20,
                    marginTop:20,
                    fontSize:17
                }}>{transText}</Text>
                <Text style={{
                    color:'white',
                    textAlign:'right',
                    fontSize:37,
                    marginRight:20,
                }}>{topText}</Text>
                <View style={{
                    flexDirection:'row',
                    zIndex:100,
                    marginTop:-7.5
                }}>
                <TouchableHighlight 
                 underlayColor='#f06d30'
                 style={{
                    left:50,
                    marginTop:0,
                }} onPress={this.change}>
                    <Image source={require('../image/exchange.png')}/>
                </TouchableHighlight>
                <View style={{
                    marginLeft:70,
                    height:1,
                    backgroundColor:'white',
                    marginTop:10,
                    flex:1
                }}></View>
                </View>
                <Text style={{
                    marginTop:10,
                    color:'white',
                    textAlign:'right',
                    fontSize:17,
                    marginRight:20,
                    marginTop:-3,
                    zIndex:99
                }}>{transToText}</Text>
                <Text style={{
                    color:'white',
                    textAlign:'right',
                    fontSize:37,
                    marginRight:20
                }}>{bottomText}</Text>
            </View>
        );
    }
}
```

    封装好了键盘与显示屏，我们需要将其联动结合起来，在View文件夹下再创建一个MainView.js文件，实现如下：

```javascript
import React, { Component } from 'react';
import { 
    View, 
    Text,
    StyleSheet
     } from 'react-native';
import KeyboardView from './KeyboardView';
import mainViewStyles from '../const/Const';
import ScreenView from './ScreenView';
// 汇率换算器主界面视图

export default class MainView extends Component {
  constructor(props) {
    super(props);
    this.state = {};
  }
  render() {
    let __this = this;
    return (
      <View style={mainViewStyles.container}>
        <ScreenView ref="screenView"/>
        <KeyboardView buttonPress={function(title){
          __this.buttonPress(title);
        }}/>
      </View>
    )
  }
  buttonPress(title){
    //根据按键类型进行相关功能调用
    if (title=='删') {
      this.refs.screenView.delete();
    }else if(title=='C'){
      this.refs.screenView.clear();
    }else if(title=='-'){
      this.refs.screenView.sub();
    }else if(title=='+'){
      this.refs.screenView.add();
    }else if(title=='='){
      this.refs.screenView.trans();
    }else{
      this.refs.screenView.buttonClick(title);
    }
    
  }

}


```

到此，ReactNative应用汇率转换器的核心功能已经全部完成了，此应用只有一个界面，其实我们已经可以直接将MainView组建注册为项目的根组建，但是如果是多界面的应用，我们最好再使用Controller将其进行封装，在Controller文件夹下创建一个MainViewController.js文件，为其提供一个view()方法，如下：

```javascript

import React, { Component } from 'react';
import { AppRegistry, Text } from 'react-native';
import MainView from '../View/MainView';
export class MainViewController{
    view(){
        return(
            <MainView />
        );
    }
}
```

修改index.ios.js与index.android.js文件如下：

```javascript
import React, { Component } from 'react';
import {
  AppRegistry,
  Text
} from 'react-native';
import {
  MainViewController
} from './Controller/MainViewController';

class News extends Component {
  render() {
    let controller = new MainViewController();
    return controller.view();
  }
}

AppRegistry.registerComponent('News', () => News);
```

运行项目，效果如下图：

iOS：

![](https://static.oschina.net/uploads/space/2016/1226/101258_UGjY_2340880.png)

android：

![](https://static.oschina.net/uploads/space/2016/1226/101317_INBP_2340880.png)

本项目的完整代码github地址如下：

[https://github.com/ZYHshao/ReactNative-ExchangeRate](https://github.com/ZYHshao/ReactNative-ExchangeRate)。

> ReactNative兴趣群：605794212，欢迎交流学习。
