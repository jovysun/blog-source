---
title: keystonejs实战之页头页脚
date: 2018-04-09 17:37:06
tags:
  - keystonejs
  - cms
categories:
  - 后端
---
前两篇介绍了入门相关知识及对keystonejs整体可用性评估，这篇介绍下开始实际运用中的页头页脚部分，因为马上项目忙了，这个先匆匆的作个收尾。

不管是用WordPress还是其他CMS系统，应用最宽泛的也是最基础的就是企业宣传类网站，我们就讲下keystonejs实现的头尾改造，效果如下图：
PC页头
![图片描述](1.jpg)

PC页脚
![图片描述](2.jpg)

移动页头
![图片描述](3.jpg)

移动页脚
![图片描述](4.jpg)

具体改造也很简单，首先找到H:\workspace\keystonejs-project\routes\middleware.js文件，然后增加`navLinksCN`代码如下：

```
exports.initLocals = function(req, res, next) {
    res.locals.navLinks = [
        { label: 'Home', key: 'home', href: '/' },
        { label: 'Blog', key: 'blog', href: '/blog' },
        { label: 'Gallery', key: 'gallery', href: '/gallery' },
        { label: 'Contact', key: 'contact', href: '/contact' },
    ];
    res.locals.navLinksCN = [
        { label: '首页', key: 'home', href: '/' },
        { label: '新闻动态', key: 'blog', href: '/blog' },
        { label: '作品展示', key: 'gallery', href: '/gallery' },
        { label: '联系我们', key: 'contact', href: '/contact' },
    ];
    res.locals.user = req.user;
    next();
};
```
然后找到H:\workspace\keystonejs-project\templates\layouts\default.pug文件，复制一份，改名如main.pug，接下来就是具体的HTML+CSS部分了。

 - 在site.css下方再引入我们自定义的样式文件如：`link(href="/styles/style.css", rel="stylesheet")`；
 - 添加header代码，如：
```
		//- HEADER
		div(style='width:100%')

			//- Customise your site's navigation by changing the navLinks Array in ./routes/middleware.js
			//- ... or completely change this header to suit your design.
			
			.box1#head
				.navBox
					.mabox
						.weima
							img(src='../images/ma.jpg')
					a.nav-left(href='index')
						img(src='../images/logo.svg')
					.nav-right
						div
							a.weibo(href='',target='_blank')
							a.weixin
							a.gouwuche(href='',target='_blank')
					.nav-center
					
						ul.menu
							each link in navLinksCN
								li(class=(section == link.key ? 'active' : null)): a(href=link.href)= link.label

						nav(role='navigation').navbar.navbar-default
							.container-fluid
							.navbar-header.text-right
								button(type='button').navbar-toggle
									span.sr-only 切换导航
									span.icon-bar
									span.icon-bar
									span.icon-bar
```

 - 添加footer部分代码，如：

```
		//- FOOTER
		//- .container: #footer

		.box1.foot#foot
			.top
				.box2 返回顶部
			.box2
				.dianshang
					span xxx电商渠道:
					p 
						a(href="http://" target="_blank") 天猫
						a(href="http://" target="_blank") 京东
						a(href="http://" target="_blank") 苏宁
						a(href="http://" target="_blank") 微信商城
				.cont
					.d1
						img(src='../images/ma2.jpg')
					.d2
						p 正月初五科技有限公司
						p 联系电话：400-8888-888
						p 北京市朝阳区朝阳门大街88号
					.d3
						img(src='../images/ma2.jpg')
				p.bei ©2014-2017 正月初五   版权所有 | 京ICP备88888888号-1
```

 - 最后，把具体views中页面引用的default模板改成main，如：

```
extends ../layouts/main
```

好了，重启下应用看看效果吧。
备注：
pug模板引擎中文文档[pug文档][5]。


  [5]: https://pugjs.org/zh-cn/api/getting-started.html