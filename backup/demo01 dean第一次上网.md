# demo_01 
>一个简陋并且需要持续更新的java漏洞小网站test

# demo01  
    
    - 预期设想  
        
        - 理由：  
            
            - 理由一：有点久远了所以有点想不起来为什么要写这个网站了，好像是写完我那个简单的rss reader之后突发奇想。可能smarter baby听多了，  
                
            
            - 理由二：觉得如果自己写一个，就能更好的理解一个项目是什么样的结构，哪里会出现什么样的问题，以后看别人的源码也能稍微可以不那么苦恼一些  
                
            
            - 理由三：最开始是研究pdf的，发现找不到可以实现我预期效果的直接通过浏览器就能发动js的，就转向，如果诱导人下载pdf点击伪装好的连接触发点什么，毕竟现在的url都可以美化，直接是看不出来链接内容的，然后点击链接的人大多数不会在意显示的链接是什么，于是一个钓鱼网站的想法出现。  
                
        
        - 预期：  
            
            - 预期一：xss，这个好写也好防  
                
            
            - 预期二：toxic pdf or other docs，通过下载或者点击  
                
            
            - 预期三：sql注入语句  
                
            
            - 预期四：目录遍历  
                
    
    - 具体实践  
        
        - 前置条件  
            
            - 简单的学习了一些java相关  
                
                - spring  
                    
                
                - spring boot  
                    
                
                - maven archetype  
                    
                
                - tomcat  
                    
                
                - 安卓开发课程用的是java——想学kotlin但觉得还是深入java一些比较好  
                    
            
            - AI  
                
                - chatgpt  
                    
                
                - cody chat  
                    
                
                - 豆包  
                    
                
                - 听了两次关于prompt的例会  
                    
        
        - Spring Boot  
            
            - 理由：为了省去对tomcat的配置  
                - 之前最开始接触java projects的时候，用的就是maven archetype，需要手动配置tomcat作为本地服务器、打包war进行测试  
                    
            
            - 官方网址：[https://spring.io/projects/spring-boot](https://spring.io/projects/spring-boot)  
                
            
            - 版本：3.3.5  
                
            
            - 使用的经典的MVC架构  
                
                - 引用一下[https://pdai.tech/md/spring/spring-x-framework-springmvc.html#%E4%BB%80%E4%B9%88%E6%98%AFmvc](https://pdai.tech/md/spring/spring-x-framework-springmvc.html#%E4%BB%80%E4%B9%88%E6%98%AFmvc)这个教程里面的图片![](https://api2.mubu.com/v3/document_image/30816267_41fd4c56-ec42-4c31-c267-d71ee0de4fbd.png)  
                    
                
                - 也还涉及到一代IOC（但是不多，但这也就是我最开始部署失败的原因之一，没有了解清楚IOC和MVC）  
                    
                
                - 最后还有点关于数据库的交互，具体的在后面MYSQL的章节说。、  
                    
# 具体分层：  
                
                - 都是依照目前的完成品来写的，中间迭代了两次吧，但基本结构都是一样的  
                    
                
## Config  
                    
                    - 简介：写了一个用于解决在部署初始化的时候遇到的Cannot resolve reference to bean 'jpaSharedEM_entityManagerFactory' while setting bean property 'entityManager' 问题，一般除了这种功能还能写点其他的其他config设置  
                        
                        - entitymangerfactory没有被找到，所以进一步的entitymanager也无法被设置  
                            
                        
                        - 关于什么是entityManager：[https://jakarta.ee/specifications/persistence/2.2/apidocs/javax/persistence/entitymanager](https://jakarta.ee/specifications/persistence/2.2/apidocs/javax/persistence/entitymanager)  
                            
                            - 简单说一下就是，来自Jakarta的一个javax.persistence的一个接口。  
                                
                            
                            - Interface used to interact with the persistence context  
                                
                    
                    - 类：JpaConfig  
                        
                        - 其实看内容就是把application.properties的内容翻译成java代码  
                            
                        
                        - 包含了datasource()，entityManagerFactory()，transactionManager()  
                            
                            - datasource()：数据库的连接配置  
                                
                                - ![](https://api2.mubu.com/v3/document_image/30816267_31635b60-dc98-4b66-fb0a-87c9ef7f349d.png)  
                                    
                                
                                - 和图上所示一样。  
                                    
                            
                            - entityManagerFactory()：entity扫描目录和hibernate的设置，用来对数据与数据库的交互进行设置  
                                
                                - ![](https://api2.mubu.com/v3/document_image/30816267_f74cd15e-fe73-4cef-d3c0-0369070cb13e.png)  
                                    
                                
                                - 创建了EntityManagerFactory实例  
                                    
                                
                                - 配置数据源  
                                    
                                
                                - 指定实体类的目录  
                                    
                                
                                - 配置JPA适配器——这里选择的是Hibernate  
                                    
                            
                            - transactionManager()：设置了需要使用的entitymanager是谁，后续spring boot会自动将其作为entitymanager进行管理。  
                                
                                - ![](https://api2.mubu.com/v3/document_image/30816267_b044fb61-ad2a-4a2f-a153-1355b3bafd7b.png)  
                                    
                                
                                - 暂时没什么想说的。  
                                    
                        
                        - 反思：  
                            - 不知道是不是因为对spring了解甚少，所以对于这些跨API的交互不了解导致的混乱需要用这个类来处理，  
                                - 在看spring framework中关于transaction的部分[https://docs.spring.io/spring-framework/reference/data-access/transaction.html](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)，里面提到了第一句  
                                    
                                    - A consistent programming model access different transaction APIs, such as Java Transaction API(JTA), JDBC. Hibernate, and the Java Persistence API(JPA)  
                                        
                                    
                                    - 在某一次修改代码中我怀疑过是不是因为hibernate和JPA的问题（需要去具体了解这两个之间的关系，现在这个只是一个不靠谱的猜测，在后续中也可以看出来两者一般情况不仅不会冲突，还是协作者），因为之前非常混乱的把JDBC和JPA一起用来处理数据库。  
                                        
                
## Controller  
                    
                    - 简介：  
                        - 主要进行路径映射与功能匹配，基本的检测也可以在这里写，比如谁登录了，谁注册了  
                            
                    
                    - mapping  
                        
                        - 这个是一个让我很苦恼过的东西，不过到结束了也没有去认真阅读过文档，有点Get it like boom boom boom  
                            
                        
                        - 在本地测试，创建private post with attachments是可以的，但疑似远程服务器上不可以，也存在可能是登录超时。  
                            
                        
                        - 官方文档：[https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-requestmapping.html](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-requestmapping.html)  
                            
                            - 里面写得很清晰  
                                
                            
                            - 主要的一个疑惑是"/projects/{project}/versions"这样的mapping的意思，查询之后得到：  
                                
                                - 处理HTTP请求  
                                    
                                    - 用户GET该路径进行请求  
                                        
                                    
                                    - 设置了对应mapping的controller方法响应并进行处理  
                                        
                                
                                - 疑惑的来源是在return 'redirect:user/profile这样的重定向跳转中，需要跳转到存在的资源。不过估计也可以编写代码成重定向到其他请求mapping的。  
                                    
                            
                            - 在Suffix Match and RFD处还有个CVE-2015-5211  
                                
                    
                    - Model & String  
                        
                        - 简介：因为我用的是Model，在查看别人的项目的时候，看到用的是ResponseEntity  
                            
                        
                        - Model：  
                            
                            - 官方文档：[https://docs.spring.io/spring-framework/reference/web/webflux/controller/ann-modelattrib-methods.html](https://docs.spring.io/spring-framework/reference/web/webflux/controller/ann-modelattrib-methods.html)  
                                - 比较涉及到的是On a @RequestMapping method to mark its return value as a model attribute  
                                    
                            
                            - 举例一下：  
                                
                                - 这个是后端：![](https://api2.mubu.com/v3/document_image/30816267_4bb9bb8b-813d-4892-9bd7-479e105beaa4.png)  
                                    - 后端为model返回的属性有：  
                                        
                                        - user：model.addAttribute("user", user);  
                                            
                                        
                                        - posts：model.addAttribute("posts", postService.findByUser(user));  
                                            
                                
                                - 这个是使用thymeleaf编写的前端![](https://api2.mubu.com/v3/document_image/30816267_0e71216f-e5c7-447d-b85a-03b363d81ea2.png)  
                                    
                                
                                - 接收的  
                                    
                                    - user  
                                        
                                        - username  
                                            
                                        
                                        - email  
                                            
                                        
                                        - avatarPath  
                                            
                                    
                                    - posts  
                                        
                                        - id  
                                            
                                        
                                        - contents  
                                            
                                        
                                        - publicFlag  
                                            
                                
                                - 可以看到就是这样将数据进行传递  
                                    
                            
                            - 为什么用String，我的理解是，最后回到的return "admin/user-profile";是一串字符串。  
                                
                        
## ResponseEntity  
                            
                            - 官方文档：[https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/responseentity.html#page-title](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/responseentity.html#page-title)  
                                
                                - 是@ResponseBody的进阶版，feature是status和headers  
                                    
                                    - response status  
                                        
                                    
                                    - headers  
                                        
                                    
                                    - body  
                                        
                                
                                - 一般接收JSON这一类的数据进行处理，在返回结果的时候顺带返回httpstatus。  
                                    
                            
                            - 举例  
                                - ![](https://api2.mubu.com/v3/document_image/30816267_82d0cf64-d32e-49b1-9aab-17cd2f9028a2.png)  
                                    
                            
                            - RESTful API环境，不过我不需要，因为用这个我还要写转换数据的函数。  
                                
                    
                    - 4个controller  
                        
                        - 关于什么是controller的官方文档：[https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html)  
                            
                        
                        - AdminController  
                            
                        
                        - AuthController  
                            
                        
                        - PostController  
                            
                        
                        - UserController  
                            
                        
                        - 四个controller大同小异，没有值得说的。  
                            
                    
                    - 关于@Autiwired的正确配置：  
                        
                        - Dependency Injection：[https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)  
                            
                            - Constructot-based  
                                
                            
                            - Setter-based  
                                
                            
                            - Setter injection versus constructor injection and the use of required:[https://spring.io/blog/2007/07/11/setter-injection-versus-constructor-injection-and-the-use-of-required](https://spring.io/blog/2007/07/11/setter-injection-versus-constructor-injection-and-the-use-of-required)  
                                
                        
                        - 两者都可以，就是使用setter injection需要确保：The Spring team generally advocates constructor injection, as it lets you implement application components as immutable objects and ensures that required dependencies are not null.而这个也恰恰是我所遇到的问题。  
                            
                            - The Spring container validates the configuration of each bean as the container is created. However, the bean properties themselves are not set until the bean is actually created  
                                
                            
                            - 所以如果没有设置好，那么使用setter injection就会遇到我遇到的问题  
                                
                            
                            - 但是关于constructor injection，会有BeanCurrentlyInCreationExpection——循环的蛇。  
                                
                        
                        - 电脑可以连接两个蓝牙？  
                            
                    
                    - 有趣：  
                        
                        - 最开始每一个被引用的service都是用@Autowired进行配置，但是没有注意到IDE在说not recommend，然后在部署的时候就报错了  
                            
                        
                        - 不过为什么IDE本地运行可以，远程服务器就不可以？  
                            
                
## Db  
                    
                    - 这一层按理说是用来写点关于数据库规范的，但我只用来写了一个初始化创建管理员。  
                        
                    
                    - 写得很简单，通过sql或者目录遍历或者文件下载会得到的。不过说的都是预期，具体看我的实现逻辑是什么了。![](https://api2.mubu.com/v3/document_image/30816267_726ce5b3-331c-4d53-b7b3-195ab37b5102.png)  
                        
                
                - Entity  
                    
                    - 实体类  
                        
                    
                    - 比起之前比较值得说的是，关于与数据库的交互所用到几个annotation：  
                        
                        - 文档：[https://jakarta.ee/specifications/persistence/2.2/apidocs/javax/persistence/package-summary](https://jakarta.ee/specifications/persistence/2.2/apidocs/javax/persistence/package-summary)，基本都找得到  
                            
                        
                        - @Table(name = "xxx")  
                            
                        
                        - @Id  
                            
                        
                        - @GeneratedValue(strategy = xxxxx)  
                            
                        
                        - ManyToOne、OneToMany:[https://jakarta.ee/specifications/persistence/2.2/apidocs/javax/persistence/manytoone](https://jakarta.ee/specifications/persistence/2.2/apidocs/javax/persistence/manytoone)  
                            
                        
                        - JoinColumn(name = xxx)  
                            
                    
                    - 还有就是在设置关于post的可见性的private boolean publicFLag;，最开始用isPublic，在repository里面还搞出一些混乱，不过有点难以追溯究竟是为什么，可能是public和里面有什么冲突了。  
                        
                
                - Repository  
                    
                    - 这里我需要搞清楚DAO和Repository的区别与联系。  
                        
                        - Dao，data access object：更关注数据库层面的事务，通常是直接与数据库表的 CRUD 操作相对应，数据的返回形式可能更偏向于数据库记录的形式  
                            
                        
                        - repository：侧重于领域对象的持久化生命周期管理。它会确保领域对象在内存中的状态与数据库中的状态保持一致。返回的是领域对象本身，而不是数据库记录的原始形式。  
                            
                    
                    - 整体也是没有什么好说的，记得这是个与数据库进行交互的interface，除非是特殊的需要写的方法，其他的都可以自动完成。在里面进行创建方法的时候，就会发现会有很多类似sql的拼接名字，不过是基于实体类的定义而来的。  
                        
                    
                    - SQL Injection：  
                        
                        - 关于like语句的注入攻击![](https://api2.mubu.com/v3/document_image/30816267_ed32128f-9522-4890-adf2-779e41a5fa4d.png)  
                            - 注入语句example：%'; DROP TABLE products; --  
                                
                        
                        - JPA会将值作为非sql语句处理。so no。![](https://api2.mubu.com/v3/document_image/30816267_00e3e541-7356-4ca8-f0ed-cf93ecf61ef0.png)  
                            
                
## Service  
                    - 两个service提供具体的服务实现方法。  
                        
                        - PostService  
                            
                            - 上传附件![](https://api2.mubu.com/v3/document_image/30816267_c4f57163-c181-4a1b-98fa-5f62c5862148.png)  
                                
                            
                            - 按理说，下载附件也应该在service里面，但是我目前是写在了controller里面  
                                - 下载附件![](https://api2.mubu.com/v3/document_image/30816267_2169d4f7-4cdb-4448-ba34-7dd15409c1a1.png)  
                                    
                        
                        - UserService  
                            - 上传头像并保存其头像![](https://api2.mubu.com/v3/document_image/30816267_defc6d13-51f7-49dd-ccf1-c5e87a0cebf2.png)  
                                
                
                - Resources  
                    
                    - 存放资源的。  
                        
                    
                    - static  
                        - 就放了一个默认的avatar图片![](https://api2.mubu.com/v3/document_image/30816267_6b92e2bc-bed1-4134-a798-8579942dc4dd.jpeg)  
                            
                    
## templates  
                        
                        - 使用thymeleaf编写前端  
                            
                        
                        - 再写下去我要失去灵魂与生命了。  
                            
                        
                        - 分层  
                            
                            - admin  
                                
                            
                            - auth  
                                
                            
                            - post  
                                - public  
                                    
                            
                            - user  
                                
                        
                        - 也没什么好说的目前因为前端是找ai写的。  
                            
        
### Thymeleaf  
            - [https://docs.spring.io/spring-framework/reference/web/webmvc-view/mvc-thymeleaf.html](https://docs.spring.io/spring-framework/reference/web/webmvc-view/mvc-thymeleaf.html)