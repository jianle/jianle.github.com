---
title: Java 捕获异常邮箱
date: 2016-12-17 14:12:47
tags:
  - java
  - mail
categories:
  - Java
lede: "Java在使用腾讯企业邮箱发送邮件时，捕获异常邮箱地址"
thumbnail: /img/tencent-mail-2.png
---

在使用`javax.mail` 群发邮件过程中，遇到邮箱失效导致邮件发送失败。  
针对这个问题，一开始我们考虑从Exception里面获取到异常邮箱地址，然后剔除后再次发送， 但是当使用腾讯企业邮箱后，异常捕获不到具体的无效地址，针对这个问题，我们排查结果是：  

发送邮件异常其实是保存了invalid的address，但是因为经过很多次的try catch 后异常信息没有抛出到最外层

<!-- more -->

我们的解决方案：

A. 自定义`SMTPTransport`继承`Transport`   
B. 使用自定义`SMTPTransport`发送邮件或附件   


实现方案
-------

### 环境  

* jdk7
* commons-email-1.4/javax.mail-1.5.2


### 自定义Class

* SMTPMessage  

之所以要定义`SMTPMessage`是因为`SMTPTransport`用到了其中的方法，而`SMTPMessage`使用了protect定义，所以必须在同一个包下面才能引用， `SMTPMessage`照抄javax.mail中同名类即可

* SMTPTransport  

```java
//省略...
public class SMTPTransport extends Transport {
    //...省略部分
    public synchronized void sendMessage(Message message, Address[] addresses)
           throws MessagingException, SendFailedException {
        // 省略部分
        try {
            mailFrom();
            rcptTo();
            //...
        } catch (SendFailedException se) {
            // 抛出异常邮箱
            throw se;
        } catch (MessagingException mex) {
            // 省略部分
        }
        // 省略部分
    }
    // 省略部分
}
```

### 使用

```java  
public void sendAccurate(String subject, List<String> receivers, String html, String filename)
        throws SendFailedException, Exception {

    String hostname = commonProperties.getProperty("email.host");

    Session session = createSession();
    Message message = createMessage(session, subject, receivers
            , html, filename, hostname);
    //获取发送对象，连接发送，断开连接设置  
    URLName smtpURL = new URLName(hostname);
    SMTPTransport sender = new SMTPTransport(session, smtpURL);
    sender.connect(hostname
            , commonProperties.getProperty("email.username"),
            commonProperties.getProperty("email.password"));
    try {
        sender.sendMessage(message, message.getRecipients(RecipientType.TO));
    } catch (SendFailedException e) {
        throw e;
    }

    sender.close();
}

public Session createSession() {

    /**
     * 必须要设置mail.smtp.auth为true这样SMTPTranport对象才会向SMTP服务器提交用户认证信息
     */
    Properties props = new Properties();
    props.setProperty("mail.transport.protocol", "smtp");
    props.setProperty("mail.smtp.auth", "true");

    Session session = Session.getDefaultInstance(props);
    //打印详细信息
    //session.setDebug(true);
    return session;
}

public Message createMessage(Session session, String subject, List<String> receivers
        , String html, String filename, String hostName) throws Exception {

    StringBuffer tos = new StringBuffer();

    for (String to : receivers) {
        tos.append(to).append(",");
    }

    Message msg = new MimeMessage(session);
    msg.setFrom(new InternetAddress(commonProperties.getProperty("email.username"), "Mini-Report"));
    msg.setRecipients(RecipientType.TO, InternetAddress.parse(tos.deleteCharAt(tos.length() - 1).toString()));
    msg.setSentDate(new Date());
    msg.setSubject(subject);
    // 创建组合类型为related的MIME消息
    MimeMultipart multipart = new MimeMultipart("related");
    MimeBodyPart contentPart = new MimeBodyPart();
    StringBuilder sb = new StringBuilder();
    sb.append("<html><head>");
    sb.append("<meta http-equiv=\"content-type\" content=\"text/html;charset=utf-8\">");
    sb.append("</head><body>");
    sb.append(html);
    sb.append("</body></html>");
    contentPart.setContent(html, "text/html; charset=utf-8");

    // 加入MimeMultipart对象
    multipart.addBodyPart(contentPart);

    if (filename != null) {
        MimeBodyPart bodyPart = new MimeBodyPart();
        FileDataSource fds = new FileDataSource(filename);
        bodyPart.setDataHandler(new DataHandler(fds));
        bodyPart.setFileName(MimeUtility.encodeText(fds.getName()));
        multipart.addBodyPart(bodyPart);
    }

    // 返回MimeBodyPart对象将作为MimeMessage对象的一部分
    msg.setContent(multipart);
    msg.saveChanges(); // 容易忘记这句话，否则结果会出现问题
    return msg;
}
```

腾讯邮箱就是坑，具体也没去多查到底为什么别的邮箱发邮件的时候能捕获到，而腾讯就不可以

备注： 如果javax.mail的版本不一致，自定义的`SMTPTransport` 需要有所改变
