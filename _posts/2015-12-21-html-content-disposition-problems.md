---
layout: post
title: 关于Content-Disposition遇到的一些问题
---

最近部门的产品接到客户的一个需求，具体就是在Notes Web端添加将邮件导出成[eml](https://en.wikipedia.org/wiki/Email#Filename_extensions)(e-mail markup language)格式的文件。这个功能在Notes Desktop端是有的，只要选中邮件，然后拖拽到桌面或者其它位置就能自动生成eml文件。经过一些调研之后，发现在服务器端实现过类似的生成邮件MIME content的功能，可以重用，这应该是一个简单的需求。使用现有的API生成符合MIME格式的文件，然后设置response中的`content-disposition: attachment; filename="mail.eml"`就OK了，的确是一个简单的需求。现在才发现根本不是那么回事。

桌面端生成的文件是以邮件标题来命名文件的，Web端为了尽量保持体验的一致也将采用这样的方式。可是邮件的标题能有多长，我试过给自己发送过一封标题有几千个字母的邮件，居然没有问题。不过显然不能生成这么长文件名的文件，首先Windows 7下的一个文件的Path(指文件的完整路径，eg，`c:\Users\Admin\Desktop\1.txt`)最长长度是260个字节(WinXP下是152，不完全准确，数量级是这样，具体可参考[Jeff Atwood的博客](http://blog.codinghorror.com/filesystem-paths-how-long-is-too-long/))，虽然NTFS最长支持32000个字符，但微软没能让这么做。其次，没有必要，用户也不会去关心完整的文件名，他们需要的是文件。

**1. 文件名截断**  
观察下面这样一段代码，截取邮件的标题，最长为`MAXSPRINTF`个字符，其中`LMBCS`是Notes使用支持多语言的编码格式，`GetItemValue`可以用来获取邮件的标题。  

``` cpp
#define MAXSPRINTF 256
 ...
char szSubject[MAXSPRINTF] = {0};
mail.GetItemValue(MAIL_SUBJECT, (LMBCS*)szSubject, sizeof(szSubject));
```

这段代码看起来没有什么问题。查了内部实现之后，其实里面是使用类似`char * strncpy ( char * destination, const char * source, size_t num )`的函数来拷贝邮件标题，`strncpy`中`n`指的是拷贝的字节(char)数。而在`LMBCS`编码中并不是一个char就表示一个字符，`LMBCS`可以是一个、两个、甚至是三个字节来表示一个字符。譬如说，邮件标题是_我欲乘风归去_，在`LMBCS`假如表示为`3A 3D 4F 13 20 01 8A 45 91 2A 0C 02 05 25 64`，如果`strncpy(source, dest, 14)`就会造成最后一个字符_`去`_在拷贝时被截断，`05 25 64`变成`05 25`，对于程序来说事无法用这么一段不完整的编码来解析的。这个问题不仅是Notes会遇到，任何使用多字节编码的程序都会遇到[链接](http://stackoverflow.com/questions/7344683/utf8-aware-strncpy)，解决方法拷贝时确保剩下的空间足够拷贝一个字符，不足则不拷贝。也有现成的方法可以使用，如[gutf8.c](http://web.mit.edu/ghudson/dev/nokrb/third/glib2/glib/gutf8.c)。

**2. 文件保存**  
如果希望浏览器弹出'另存为...'的文件保存窗口，可以在HTTP response中设置，如果文件名存在空格，则需要将文件名用双引号扩起来[链接](https://tools.ietf.org/html/rfc5987)

``` html
Content-Disposition: Attachment; filename=example.html
```

``` html
Content-Disposition: Attachment; filename="an example.html"
```

这里的文件名只能是US-ASCII字符，但是如果不是ASCII字符，有些浏览器是可以识别的，但这不是标准，不能保证兼容性。如果文件名中需要Non-ASCII(如€)字符，则有下面一种做法[链接](https://tools.ietf.org/html/rfc6266#section-4.2)。如果浏览器能够支持utf-8编码的文件名则会显示成`€ rates`，如果不支持则会显示成`EURO rates`，后一种相当于是前一种的备选方案

```
Content-Disposition: attachment;
                     filename="EURO rates";
                     filename*=utf-8''%e2%82%ac%20rates
```

目前主流的浏览器都已支持。

但是在浏览器测试这种做法的时候，我发现Chrome浏览器并不总是能弹出文件保存窗口，而Firefox和IE 11却没有任何问题。打开调试窗口发现，请求返回时浏览器会有`err_response_headers_multiple_content_disposition`错误，这表示浏览器接收到了重复的Cotent-Disposition header，而Chrome为了防止HTTP response split attack(不知所云)是不允许这样做的，结果就是报错。

>  Duplicate headers received from server  
>  The response from the server contained duplicate headers. This problem is generally the result of a misconfigured website or proxy. Only the website or proxy administrator can fix this issue.  
>  Error 349 (net::ERR_RESPONSE_HEADERS_MULTIPLE_CONTENT_DISPOSITION): Multiple distinct Content-Disposition headers received. This is disallowed to protect against HTTP response splitting attacks.  

我仔细检查了我的代码，真的是只添加了一次Content-Disposition header，而根据HTTP response看起来也没有任何问题，而且在Firefox和IE 11下也真的没有问题。

一番搜索发现，因为邮件标题包含逗号`,`，而对于utf-8而言，ASCII字符的值和在utf-8下是相同的，utf-8是对ASCII兼容的，[每个程序员需知道系列](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)，所以`,`经过utf-8编码后还是一样的。这就会导致文件名中包含`,`，而在[RFC2616](https://www.ietf.org/rfc/rfc2616.txt)中，`,`对于HTTP response header来说是一个分隔符，所以Chrome就会将文件名以`,`分割，造成有多个Content-Disposition的情况。根据[Issue 103618](https://code.google.com/p/chromium/issues/detail?id=103618)来看，Google不打算改变现有的做法。目前只能保证文件名中不出现这些分隔符，可以替换成其它字符。

PS: 期间试过了解下`LMBCS(Lotus Mulit-Btye Character Set)`编码，`LMBCS`最初是Lotus公司在1989年为[Lotus 1-2-3](https://en.wikipedia.org/wiki/Lotus_1-2-3)设计的编码格式，支持当时绝大多数语言。你要问为什么不用UTF-8？因为那时UTF-8还没有发明啊。[看了一会](http://web.archive.org/web/20050205225225/http://www.batutis.com/i18n/papers/lmbcs/LMBCS.html)之后，我放弃了，我爱UTF-8!
