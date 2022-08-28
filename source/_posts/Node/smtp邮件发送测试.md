---
title: C# smtp邮件发送测试
date: 2022年8月28日16:02:25
updated: 2022年8月28日16:02:32
tags: [笔记,smtp]
categories:
 - [笔记,smtp]
keywords: C# smtp邮件发送测试
description: C# smtp邮件发送测试
cover: false
---

 ~~~C#
 		/// <summary>
        /// 通过客户端发送邮件信息（如果发件人使用QQ邮箱，那么对应的smtpService需改成smtp.qq.com）
        /// 前提：发件人需要到邮箱中启用第三方客户端服务，位置：设置-->账户-->POP3/SMTP服务 -->开启，获取邮箱授权码
        /// </summary>
        /// <param name="senderEmail">发件人邮箱</param>
        /// <param name="authCode">发件人邮箱授权码</param>
        /// <param name="receiverEmail">收件人邮箱</param>
        /// <param name="emailTitle">邮件标题</param>
        /// <param name="emailContent">邮件内容</param>
        /// <param name="smtpService">邮件服务名：smtp.qq.com </param>
        public static void SendEmailBySMTP(string senderEmail, string authCode, string receiverEmail, string emailTitle, string emailContent, string smtpService)
        {
            //实例化一个发送邮件类
            MailMessage mailMsg = new MailMessage();
            //发件人邮箱地址
            mailMsg.From = new MailAddress(senderEmail);
            //收件人邮箱地址
            mailMsg.To.Add(new MailAddress(receiverEmail));
            //邮件标题的编码格式
            mailMsg.SubjectEncoding = Encoding.UTF8;
            //邮件标题
            mailMsg.Subject = emailTitle;
            //邮件内容的编码格式
            mailMsg.BodyEncoding = Encoding.UTF8;
            //邮件内容
            mailMsg.Body = emailContent;
            //是否是html邮件
            mailMsg.IsBodyHtml = true;
            //邮件优先级
            mailMsg.Priority = MailPriority.High;
            //实例化一个SmtpClient类
            SmtpClient client = new SmtpClient();
            //设置邮件服务名，这里使用的是QQ邮箱，所以是smtp.qq.com, 若使用163邮箱，则是 smtp.163.com
            client.Host = smtpService;
            //设置邮件端口
            client.Port = 25;
            //使用安全加密连接
            client.EnableSsl = true;
            //不和请求一块发送
            client.UseDefaultCredentials = false;
            //验证发件人身份（发件人邮箱，邮箱生成的授权码）
            client.Credentials = new NetworkCredential(senderEmail, authCode);
            //发送
            client.Send(mailMsg);
        }
 ~~~

