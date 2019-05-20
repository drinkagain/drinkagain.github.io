---
title: (01)SpringBoot EL获取配置文件中的值的方式
date: 2019-05-20 09:43:28
tags: SpringBoot
---

### 一、准备
- 为了方便IO操作，引入了commons-io
```
<dependency>
     <groupId>commons-io</groupId>
     <artifactId>commons-io</artifactId>
     <version>2.6</version>
</dependency>
```
<!--more-->
- application.yml
``` yml
book:
  name: 熊猫酒仙_
```
- resouces 下新建一个test文件夹，文件夹下一个test.txt文件,文件内容:
``` string
  Test
```
### 二、代码
- OtherService
``` java

@Service
public class OtherService {

    @Value("测试其它类的属性")
    private String another;

    public String getAnother() {
        return another;
    }

    public void setAnother(String another) {
        this.another = another;
    }
}
```

- ElConfig
``` java


@Configuration
public class ElConfig {
    /**
     * 注入普通字符串
     */
    @Value("I Love Java")
    private String normal;
    /**
     * 注入操作系统属性
     */
    @Value("#{systemProperties['os.name']}")
    private String osName;
    /**
     * 注入表达式结果
     */
    @Value("#{T(java.lang.Math).random()*100 }")
    private double random;
    /**
     * 注入其它Bean的属性(其它Bean的属性需要get set 方法，或者 public)
     */
    @Value("#{otherService.another}")
    private String fromAnother;
    /**
     * 注入文件
     */
    @Value("classpath:/test/test.txt")
    private Resource testFile;
    /**
     * 注入网址
     */
    @Value("http://www.baidu.com")
    private Resource testUrl;
    /**
     * 注入配置文件
     */
    @Value("${book.name}")
    private String userName;
    /**
     * 注入配置文件
     */
    @Autowired
    private Environment environment;

    public void getSecret() {
        try {
            System.out.println(normal);
            System.out.println(osName);
            System.out.println(random);
            System.out.println(fromAnother);
            System.out.println(IOUtils.toString(testFile.getInputStream(), "utf-8"));
            System.out.println(IOUtils.toString(testUrl.getInputStream(), "utf-8"));
            System.out.println(userName);
            //注入配置文件
            System.out.println(environment.getProperty("book.name"));

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 三、测试
``` java  

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootAopApplicationTests {

    @Resource
    private ElConfig elConfig;

    @Test
    public void elConfigTest() {
        elConfig.getSecret();
    }

}
```
### 四、输出
``` string
I Love You
Windows 10
8.226641172488735
测试其它类的属性
Test
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=http://s1.bdstatic.com/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn"></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=http://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');</script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>

熊猫酒仙_
熊猫酒仙_
```
### 五、GitHub源码地址
[GitHub源码: https://github.com/drinkagain/SpringBoot2.0/tree/master/springboot-el](https://github.com/drinkagain/SpringBoot2.0/tree/master/springboot-el)
