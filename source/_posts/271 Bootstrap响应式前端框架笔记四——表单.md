---
title: Bootstrap响应式前端框架笔记四——表单
date: 2016-12-06
categories: 前后端
tags: []
---
## Bootstrap响应式前端框架笔记四——表单

### 一、基本表单样式

    在Bootstrap框架中，可以为表单标签添加form-control属性来为其设置默认样式，默认表单控件的宽度将充满父容器标签。需要注意，在布局表单时，可以为其设置一个label标签用于说明，将label标签的for属性与表单标签的id相对应，可以实现当用户点击label标签时使其对应的表单自动获取输入焦点。示例代码如下：

```html
        <p>Bootstrap为默认的表单便签添加了样式</p>
        <form>
            <div class="form-group">
                <label for="exampleInputEmail1">Email address</label>
                <input type="email" class="form-control" id="exampleInputEmail1" placeholder="Enter email">
            </div>
            <div class="form-group">
                <label for="exampleInputPassword1">Password</label>
                <input type="password" class="form-control" id="exampleInputPassword1" placeholder="Password">
            </div>
            <div class="checkbox">
                <label>
                <input type="checkbox">性别
                </label>
            </div>    
            <button type="submit">提交</button>
        </form>
```

需要注意，将label和表单标签包裹在form-group类内，会自动进行间距的布局设置。效果如下：

![](https://static.oschina.net/uploads/space/2016/1206/120805_I8oW_2340880.png)

    默认情况下，label与表单元素的排列是竖直布局的，可以使用form-horizontal类来将其设置为水平布局，示例如下：

```html
        <p>使用from-horizontal类可以将label与表单进行水平排列，并可以结合栅格系统使用</p>
        <form class="form-horizontal" role="form">
            <div class="form-group">
                <label for="inputEmail3" class="col-sm-2">Email</label>
                <div class="col-sm-10">
                    <input type="email" class="form-control" id="inputEmail3" placeholder="Email">
                </div>
            </div>
            <div class="form-group">
                <label for="inputPassword3" class="col-sm-2">Password</label>
                <div class="col-sm-10">
                    <input type="password" class="form-control" id="inputPassword3" placeholder="Password">
                </div>
            </div>
            <div class="form-group">
                <div class="col-sm-offset-2 col-sm-10">
                    <button type="submit" class="btn btn-default">Sign in</button>
                </div>
            </div>
        </form>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1206/132929_1Xak_2340880.png)

### 二、选择框与下拉列表

    HTML中有单选框和复选框两种选择框标签。示例代码如下：

```html
        <p>默认的单选框与复选框样式</p>
        <p>复选框</p>
        <div class="checkbox">
        <label>
            <input type="checkbox" name="" id="" value="" />
            足球
        </label>
        </div>
        <div class="checkbox">
        <label>
            <input type="checkbox" name="" id="" value="" />
            篮球
        </label>
        </div>
        <div class="checkbox">
        <label>
            <input type="checkbox" name="" id="" value="" />
            乒乓球
        </label>
        </div>
        <p>单选框</p>
        <div class="radio">
            <label>
                <input type="radio" name="sex"/>
                男
            </label>
        </div>
        <div class="radio">
            <label>
                <input type="radio" name="sex"/>
                女
            </label>
        </div>
```

![](https://static.oschina.net/uploads/space/2016/1206/140007_knLG_2340880.png)

可以看到，默认的单选框与复选框的排列也是垂直布局的，使用checkbox-inline类和radio-inline类可以实现水平排列布局，示例如下：

```html
        <p>水平排列的单选框与复选框样式</p>
        <p>复选框</p>
        <div class="checkbox-inline">
            <label>
            <input type="checkbox" name="" id="" value="" />
            足球
        </label>
        </div>
        <div class="checkbox-inline">
            <label>
            <input type="checkbox" name="" id="" value="" />
            篮球
        </label>
        </div>
        <div class="checkbox-inline">
            <label>
            <input type="checkbox" name="" id="" value="" />
            乒乓球
        </label>
        </div>
        <p>单选框</p>
        <div class="radio-inline">
            <label>
                <input type="radio" name="sex"/>
                男
            </label>
        </div>
        <div class="radio-inline">
            <label>
                <input type="radio" name="sex"/>
                女
            </label>
        </div>
```

效果如下图：

![](https://static.oschina.net/uploads/space/2016/1206/140609_6F1U_2340880.png)

    Bootstrap框架中默认的下拉列表样式示例如下：

```html
        <p>默认的下拉列表</p>
        <select class="form-control">
            <option>上海</option>
            <option>北京</option>
            <option>郑州</option>
            <option>深圳</option>
            <option>广州</option>
        </select>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1206/141957_cLyy_2340880.png)

开发者也可以通过添加multiple参数的方式来进行下拉选项的全部展示，示例如下：

```html
        <p>使用multiple参数来进行下拉选项的全部展示</p>
        <select multiple class="form-control">
            <option>上海</option>
            <option>北京</option>
            <option>郑州</option>
            <option>深圳</option>
            <option>广州</option>
        </select>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1206/142339_b6jl_2340880.png)

### 三、表单状态

    为表单元素添加disabled属性来将表单设置为禁用状态，示例如下：

```html
        <p>禁用表单</p>
        <input class="form-control" placeholder="被禁用的输入框" type="text" disabled/>
        <div  class="checkbox">
        <label>
            <input type="checkbox" disabled/>被禁用的复选框
        </label>
        </div>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1206/143451_V88P_2340880.png)

    如果在开发中需要使一组的表单元素全部处于禁用状态，可以使用fieldset标签进行包裹，并为fieldset标签添加disabled属性。示例如下：

```html
        <p>进行一组表单元素的禁用</p>
        <form>
            <fieldset disabled>
                <div class="form-group">
                    <label for="disabledTextInput">被禁用的输入框</label>
                    <input type="text" id="disabledTextInput" class="form-control" placeholder="被禁用的输入框">
                </div>
                <div class="form-group">
                    <label for="disabledSelect">被禁用的下拉菜单</label>
                    <select id="disabledSelect" class="form-control">
                        <option>被禁用的下拉菜单</option>
                    </select>
                </div>
                <div class="checkbox">
                    <label>
        <input type="checkbox"> 被禁用的选择框
      </label>
                </div>
                <button type="submit" class="btn btn-primary">被禁用的按钮</button>
            </fieldset>
        </form>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1206/144045_7e6q_2340880.png)

Bootstrap中也定义好了一些校验状态的样式，例如警告，成功，错误等状态，为表单元素的父标签添加这些状态类即可，示例如下：

```html
        <p>校验状态</p>
        <form>
            <div class="has-error form-group">
                <input class="form-control" placeholder="错误状态的表单" type="text" />
            </div>
            <div class="has-success form-group">
                <input class="form-control" placeholder="成功状态的表单" type="text" />
            </div>
            <div class="has-warning form-group">
                <input class="form-control" placeholder="警告状态的表单" type="text" />
            </div>
        </form>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1206/144706_tq0F_2340880.png)

开发者也可以为验证表单的右侧添加一个小图标，前提需要为表单元素的父元素设置has-feedback类，示例如下：

```html
        <p>为表单添加右侧icon</p>
        <form>
            <div class="form-group has-error has-feedback ">
                <input class="form-control" placeholder="错误状态的表单" type="text" />
                <span class="glyphicon glyphicon-eur form-control-feedback"></span>
            </div>
            <div class="has-success form-group has-feedback">
                <input class="form-control" placeholder="成功状态的表单" type="text" />
                <span class="glyphicon glyphicon-ok form-control-feedback"></span>
            </div>
            <div class="has-warning form-group has-feedback">
                <input class="form-control" placeholder="警告状态的表单" type="text" />
                <span class="glyphicon glyphicon-off form-control-feedback"></span>
            </div>
        </form>
```

效果如下：

![](https://static.oschina.net/uploads/space/2016/1206/150820_RoH1_2340880.png)

   另外，本篇博客中所有的实例代码及显示效果，在如下地址中，需要的可以自行对照学习。

[http://zyhshao.github.io/bootStrapDemo/form.html](http://zyhshao.github.io/bootStrapDemo/form.html)。

> 前端学习新人，有志同道合的朋友，欢迎交流与指导，QQ群:541458536
