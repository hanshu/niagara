---
layout: post
title:  "axvelocity"
categories: niagara
---

Niagara Framework从3.7版本就开始支持Apache Velocity作为模板引擎，这样我们就可以使用VTL (Velocity Template Language) 脚本语言为用户定制个性化的HTML页面。在实际项目里，一般都会先根据业务需求开发好HTML页面，然后通过下面一些做法向客户展现开发的页面：

- 直接访问Station下shared目录里的HTML页面；
- 通过自定义Servlet，通过读取模块里的HTML页面或shared目录下的HTML页面，再输出到ServletOutputStream；
- 也可以通过下面介绍的axvelocity模块，创建VelocityDoc以及相应的Velocity模板，从而生成页面；

# Velocity Station Components

首先需要axvelocity模块的授权才可以使用axvelocity模块

- VelocityServlet

负责处理页面请求

- VelocityDoc

提供Velocity模板和Velocity Context Elements的容器，这里可以定义页面的MIME Type (text/html, text/javascript, ...)

- VelocityContextOrdElement

建立Station里组件和VTL模板之间的对应关系

- VelocityDocWebProfile

与其他Web Profile类似，该Web Profile负责生成HTML页面的Layout

## Getting Started

1. 从Palette里打开axvelocity，将VelocityServlet拖入Station；
2. 双击该Servlet打开Velocity Document Manager视图：
    - 在这个视图里可以根据需要修改Servlet Name；
    - 添加Velocity Document Components；
3. 新建Velocity Document Component：
    - 提供该Document的Velocity模板；
    - 双击该新建的Velocity Document Component打开Velocity Context Element Manager视图；
4. 根据需要添加Velocity Context Ord Elements；

## Template File

Velocity模板包含了设计好的HTML页面，以及通过VTL动态生成的一些内容；同时也可以在该模板里引用之前添加好的Velocity Context Ord Elements。下面是一个Velocity模板的例子：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My first Velocity test!</title>
  </head>
  <body>
    <p>Boolean Point: $boolPoint</p>
    <p>Numeric Point: $numPoint</p>
  </body>
</html>
```

## View Parameters

可以通过URI(或ORD)将参数传给BVelocityDocView进行处理，看下面的例子：

- `http://localhost/velocity/test?param1=val1&param2=val2`
- `station:|slot:/VelocityServlet|view:test?param1=val1;param2=val2`

通过VTL可以获取到传入的参数：

```vtl
## Iterate through any query parameters used in the URL
#set( $parameters = $ax.op.getRequest().getParameterNames() )
#foreach ( $param in $parameters )
<p>
  Found parameter: $param - $ax.op.getRequest().getParameter($param)
</p>
#end
```

## BVelocityDocWebProfile

可以用来定义HTML页面的Layout(Header, Footer, ...)，VelocityDoc通过useProfile决定是否需要使用这个Web Profile来展现页面。

## Niagara VTL

Niagara Velocity封装了许多与Velocity相关的方法和属性，以便于在Velocity模板中使用这些API。

# Hx Velocity Views

BVelocityHxView, BVelocityHxFieldEditor, BVelocityHxPxWidget

# Velocity Px Views

在创建Px View时，可以选择Dynamic View: `axvelocity:VelocityPxView`，提供Px Velocity Template File。

# References

- module://docDeveloper/doc/velocity/velocity.html
- module://docDeveloper/doc/velocity/pxVelocity.html