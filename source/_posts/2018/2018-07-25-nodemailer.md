---
title: html邮件模板测试及nodemailer的入门
tags:
  - node
categories:
  - 前端
date: 2018-07-25 09:35:34
updated: 2018-07-25 09:35:34
---

# 前言
项目中客户往来需要定制邮件模板，前端开发完后需要测试，简单的可以直接浏览html文件，但是各种邮件终端解析有些异同，因此需要用真实的各个终端进行测试，这就需要我们搭建一个简单邮件发送功能。业务需求是实现html模板的测试，对于nodemailer其他功能只作入门简介，具体可根据实际需要深究。

# 概述
## 示例效果图：
![html模板](emailTemplate.jpg)
![ejs模板](emailEjs.jpg)
<!-- more -->

# 详述
## nodemailer简介
+ 是什么，Node.js应用的邮件发送模块；
+ 环境要求Node.js v6+；
+ 安装`npm install nodemailer --save`
+ 官方入门示例代码：
```
'use strict';
const nodemailer = require('nodemailer');

// Generate test SMTP service account from ethereal.email
// Only needed if you don't have a real mail account for testing
nodemailer.createTestAccount((err, account) => {
    // create reusable transporter object using the default SMTP transport
    let transporter = nodemailer.createTransport({
        host: 'smtp.ethereal.email',
        port: 587,
        secure: false, // true for 465, false for other ports
        auth: {
            user: account.user, // generated ethereal user
            pass: account.pass // generated ethereal password
        }
    });

    // setup email data with unicode symbols
    let mailOptions = {
        from: '"Fred Foo 👻" <foo@example.com>', // sender address
        to: 'bar@example.com, baz@example.com', // list of receivers
        subject: 'Hello ✔', // Subject line
        text: 'Hello world?', // plain text body
        html: '<b>Hello world?</b>' // html body
    };

    // send mail with defined transport object
    transporter.sendMail(mailOptions, (error, info) => {
        if (error) {
            return console.log(error);
        }
        console.log('Message sent: %s', info.messageId);
        // Preview only available when sending through an Ethereal account
        console.log('Preview URL: %s', nodemailer.getTestMessageUrl(info));

        // Message sent: <b658f8ca-6296-ccf4-8306-87d57a0b4321@example.com>
        // Preview URL: https://ethereal.email/message/WaQKMgKddxQDoou...
    });
});

```
这个是ethereal.email邮件测试账户的示例，我们重点看下代码，主要是三个部分：transporter，mailOptions，sendMail。

首先，需要创建一个transporter实例对象，具体参数就像我们在邮件终端如foxmail中配置账户一样，需要注意的是pass，不是登录密码，而是授权码。（163邮箱的话，要开启POP3/SMTP服务，在设置 --> POP3/SMTP/IMAP页面，设置。开通后会有个授权码的，配置里的密码，就是用这个授权码；qq邮箱的话，同样也要开启这个服务，设置 --> 账户 --> POP3服务，点击开启，就会有个授权码，如果忘了记录，在开启服务下面有个“生成授权码”的，可以获取到的。）

其次，设置发送相关参数，这个与日常发邮件填写的参数一样，重点看下html参数，及示例没提到的attachments参数。html参数是邮件主题内容，值可以是html字符串，也可以是数据流，这个稍后使用实例再介绍；

最后，调用transporter实例的sendMail方法，可以在回调函数中处理成功失败相关业务逻辑。

## 实例简介：
### 上文提到mailOptions中的html参数值可以是数据流，具体代码如下：
```
// 内容来自html文件
const htmlData = fs.readFileSync('email/template.html','utf-8');

// 内容来自ejs模板
const template = ejs.compile(fs.readFileSync(path.resolve(__dirname, 'email/email.ejs'), 'utf8'));
const ejsData = template({
  title: 'Ejs',
  desc: '使用Ejs渲染模板',
});
```
```
	let mailOptions = {
		from: '"sun" <'+ config.user+'>', // sender address
		to: '"sun" <xxxx@qq.com>, xxxx@xxxx.com', // list of receivers
		subject: 'nodemailer测试demo', // Subject line
		// text: 'Hello', // plain text body
		// html: ejsData, // 内容来自ejs模板
		html: htmlData, // 内容来自html文件
		attachments: [
			{   // utf-8 string as an attachment
				filename: 'text1.txt',
				path: path.resolve(__dirname, 'attachments/text.txt')
			},
			{   // utf-8 string as an attachment
				filename: 'text2.txt',
				content: 'i am text2.txt'
			},
			{
				filename: '蚂蚁头像.jpg',
				path: path.resolve(__dirname, 'attachments/mayi.jpg'),
				cid: 'mayi.jpg'
			},
			{
				filename: '二维码',
				path: path.resolve(__dirname, 'attachments/canvas.png'),
				cid: 'qrcode'
			}
		]
	};
```
### 几个点强调下：
+ 对于常见的邮件服务商可以用service替换host配置，具体支持情况见[service列表](https://nodemailer.com/smtp/well-known/)。
```
let transporter = nodemailer.createTransport({
     service: 'QQ', // no need to set host or port etc.
     auth: {
         user: 'account.email@example.com',
         pass: 'smtp-password'
     }
});

transporter.sendMail(...)

```
+ attachments的使用，除了正常的添加附件外，还可以定义`cid`作为内容的引用源，附件配置见上文mailOptions示例代码，具体内容中使用如下：
```
<img src="cid:qrcode" alt="" title="">
```
+ [源代码](https://github.com/jovysun/sendMail)


# 备注
https://segmentfault.com/a/1190000012251328

https://nodemailer.com/about/
