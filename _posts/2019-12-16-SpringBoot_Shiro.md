---
layout: default
title: SpringBoot整合Shiro
#excerpt: 

---

　　兜兜转转，转眼已经进入12月中旬了，马上就是2020年了，回顾这一年，技术上，真的提升很少很少，项目中使用的技术都是很老套的SpringMvc+MyBatis，有的老项目还使用的是Struts2+Hibernate，公司对技术也不重视，在稳定的基础上，不求有功，但求无过；也没有什么技术分享。我经历过上家公司的快速节奏，现在也放松下来了，去年我还自学python、学习数据结构和算法、碰到的技术问题都会去深究。今年，虽然还保持博客输出，但是，明显懈怠了，基本是工作中碰到的问题才会输出，问题的根本原因，除了那些比较明显的，会写一写；隐藏的比较深的那些问题，都是没有下文的。

　　上个月，老大说老板要搞聚合支付，然后不知道从哪弄来的一套支付代码，让我们几个开发熟悉代码，为将来的开发任务做准备。这项目使用了SpringCloud的很多组件，spring-cloud-bus、spring-cloud-consul、spring-cloud-feign、spring-cloud-hystrix等等，项目被拆分为9个相互关联的小项目，环境搭建还是使用的docker技术，别的不说，光是能在本地把项目跑起来，前前后后都花了近一天时间。

　　正好闲了快一年了，又燃起了对技术的热枕，准备把这支付项目使用的技术，都整理输出。第一篇输出是Shiro。为什么是Shiro呢？因为权限验证，是网站的基础，它是用户维度下，用户跟网站关联的第一步，所以，它是第一个输出。

# 理论部分

## Shiro是什么？

　　Shiro是一款简单易用的java安全框架，提供了认证、授权、会话管理等功能，由Apache组织开源，在业内被广泛使用。官网是：  http://shiro.apache.org 

## 核心组件

　　核心组件有三个，分别是Subject、Realm、SecurityManager。

　　Subject：当前操作的主体，可以是自然人，也可以是爬虫。比如，如果张三在网站通过登陆页面登陆，进入系统，那当前操作的主体就是张三；如果是爬虫S模拟登陆、进入系统，那当前操作的主体就是S。

　　Realm：域，对登陆时的身份验证、对访问的权限控制，这些都是要由我们自己来实现。

　　SecurityManager：Shiro框架中，调节者的功能，充当大主管，各种“琐事”都要交给它，比如，实现的realm要交给SM(SecurityManager下面统一简称SM)来管理；使用缓存管理器记录用户权限信息，也要交给SM；开启“记住我”功能，也要通知SM。

　　再附上Shiro的完整架构图 ![Shiro的完整架构图]({{site.url}}/assets/2019-12-16-SpringBoot_Shiro/shiro_architecture.jpg)



# 实践部分

　　上面都是说的理论，说的不是非常全面，想了解更多的，可以自行百度或者谷歌。
## 1.数据库支持

　　我这里使用的是MySQL，建表语句，这里就不贴出来了，太长了。  [可以戳我下载]( https://github.com/bigSaltFish/SpringBootStudy/blob/master/shiro_example/shiro_mysql.sql )

　　数据持久层使用的是MyBatis框架，对应表的XML文件，都是自动生成的。

## 2.引入Maven依赖

　　这里使用的是SpringBoot，对SpringBoot不了解的也没关系，就当它是一个普通Spring项目，只不过引入了特定的依赖。

　　这里是继承 spring-boot-starter-parent 的方式来构建SpringBoot应用。

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.1</version>
    </dependency>

    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.4.0</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.20</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
</dependencies>
```


## 3.ShiroConfig配置

　　顾名思义，ShiroConfig就是Shiro的配置相关，比如：定义SM、定义拦截和不拦截的url、添加注解支持、添加缓存支持等，具体代码如下：


```java
@Configuration
public class ShiroConfig {
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 设置securityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 登录的url
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 登录成功后跳转的url
        shiroFilterFactoryBean.setSuccessUrl("/index");
        // 未授权url
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");

        LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<>();

        // 定义filterChain，静态资源不拦截
        filterChainDefinitionMap.put("/css/**", "anon");
        filterChainDefinitionMap.put("/js/**", "anon");
        filterChainDefinitionMap.put("/fonts/**", "anon");
        filterChainDefinitionMap.put("/img/**", "anon");
        // druid数据源监控页面不拦截
        filterChainDefinitionMap.put("/druid/**", "anon");
        // 配置退出过滤器，其中具体的退出代码Shiro已经替我们实现了 
        filterChainDefinitionMap.put("/logout", "logout");
        filterChainDefinitionMap.put("/", "anon");
        // 除上以外所有url都必须认证通过才可以访问，未通过认证自动访问LoginUrl
//        filterChainDefinitionMap.put("/**", "authc"); // authc表示:需要通过认证
        filterChainDefinitionMap.put("/**", "user"); // user表示:通过认证，或者，remeber me记住了登录状态

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    /**
     * 安全管理器，Shiro的大主管，各种”琐事“处理都要委托给它<br/>
     * 比如设置realm域、设置“记住我”、设置缓存等
     *
     * @return
     */
    @Bean
    public SecurityManager securityManager() {
        // 配置SecurityManager，并注入shiroRealm
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(shiroRealm());
        securityManager.setRememberMeManager(rememberMeManager());
        securityManager.setCacheManager(getEhCacheManager());
        return securityManager;
    }

    /**
     * 自己实现的Realm
     *
     * @return
     */
    @Bean
    public ShiroRealm shiroRealm() {
        ShiroRealm shiroRealm = new ShiroRealm();
        return shiroRealm;
    }

    /**
     * Shiro注解支持
     * <li>表示当前Subject已经通过login进行了身份验证；即Subject.isAuthenticated()返回true。 @RequiresAuthentication   </li>
     * <li>表示当前Subject已经身份验证或者通过记住我登录的。 @RequiresUser  </li>
     * <li>表示当前Subject没有身份验证或通过记住我登录过，即是游客身份。 @RequiresGuest   </li>
     * <li>表示当前Subject需要角色admin和user。 @RequiresRoles(value={"admin", "user"}, logical= Logical.AND)</li>
     * <li>表示当前Subject需要权限user:a或user:b。 @RequiresPermissions (value={"user:a", "user:b"}, logical= Logical.OR)</li>
     *
     * @param securityManager
     * @return
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }

    /**
     * 缓存支持，这里是ehcache
     *
     * @return
     */
    @Bean
    public EhCacheManager getEhCacheManager() {
        EhCacheManager em = new EhCacheManager();
        em.setCacheManagerConfigFile("classpath:config/shiro-ehcache.xml");
        return em;
    }

    /**
     * 返回cookie管理对象
     * <li>1.默认cookie对象设置</li>
     * <li>2.放入cookie管理器</li>
     *
     * @return
     */
    private CookieRememberMeManager rememberMeManager() {
        // 设置cookie名称，对应login.html页面的<input type="checkbox" name="rememberMe"/>
        SimpleCookie cookie = new SimpleCookie("rememberMe");
        // 设置cookie的过期时间，单位为秒，这里为一天
        cookie.setMaxAge(86400);

        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(cookie);
        // rememberMe cookie加密的密钥
        cookieRememberMeManager.setCipherKey(Base64.decode("4AvVhmFLUs0KTA3Kprsdag=="));
        return cookieRememberMeManager;
    }
}
```


## 4.realm配置

　　该类需要继承 AuthorizingRealm ，需要实现两个方法，一个是 doGetAuthenticationInfo()，它是登录认证（登录时调用），验证用户名和密码；另外一个是 doGetAuthorizationInfo() ，它是获取用户角色和权限（访问控制），获取用户的权限，对访问的放行或者阻拦。

　　具体代码如下：

```java
public class ShiroRealm extends AuthorizingRealm {

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private UserRoleMapper userRoleMapper;
    @Autowired
    private UserPermissionMapper userPermissionMapper;

    /**
     * 获取用户角色和权限（验证权限时调用）
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {
        User user = (User) SecurityUtils.getSubject().getPrincipal();
        String userName = user.getUserName();

        System.out.println("用户" + userName + "获取权限-----ShiroRealm.doGetAuthorizationInfo");
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();

        // 获取用户角色集
        List<Role> roleList = userRoleMapper.findByUserName(userName);
        Set<String> roleSet = new HashSet<String>();
        for (Role r : roleList) {
            roleSet.add(r.getName());
        }
        simpleAuthorizationInfo.setRoles(roleSet);

        // 获取用户权限集
        List<Permission> permissionList = userPermissionMapper.findByUserName(userName);
        Set<String> permissionSet = new HashSet<String>();
        for (Permission p : permissionList) {
            permissionSet.add(p.getName());
        }
        simpleAuthorizationInfo.setStringPermissions(permissionSet);
        return simpleAuthorizationInfo;
    }

    /**
     * 登录认证（登录时调用）
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // 获取用户输入的用户名和密码
        String userName = (String) token.getPrincipal();
        String password = new String((char[]) token.getCredentials());

        // 通过用户名到数据库查询用户信息
        User user = userMapper.findByUserName(userName);

        if (user == null) {
            throw new UnknownAccountException("用户名或密码错误！");
        }
        if (!password.equals(user.getPassword())) {
            throw new IncorrectCredentialsException("用户名或密码错误！");
        }
        if (user.getStatus().equals("0")) {
            throw new LockedAccountException("账号已被锁定,请联系管理员！");
        }
        return new SimpleAuthenticationInfo(user, password, getName());
    }
}
```


## 5.前端页面和对应controller

### 1.login.html 登陆页面

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
    <link rel="stylesheet" th:href="@{/css/login.css}" type="text/css">
    <script th:src="@{/js/jquery-1.11.1.min.js}"></script>
</head>
<body>
<div class="login-page">
    <div class="form">
        <input type="text" placeholder="用户名" name="username" required="required"/>
        <input type="password" placeholder="密码" name="password" required="required"/>
        <p><input type="checkbox" name="rememberMe"/>记住我</p>
        <button onclick="login()">登录</button>
    </div>
</div>
</body>
<script th:inline="javascript">
    var ctx = '/';

    function login() {
        var username = $("input[name='username']").val();
        var password = $("input[name='password']").val();
        var rememberMe = $("input[name='rememberMe']").is(':checked');
        $.ajax({
            type: "post",
            url: ctx + "login",
            data: {"username": username, "password": password, "rememberMe": rememberMe},
            dataType: "json",
            success: function (r) {
                if (r.code == 0) {
                    location.href = ctx + 'index';
                } else {
                    alert(r.msg);
                }
            },
            error: function (r) {
                console.log("-------------" + r);
            }
        });
    }
</script>
</html>
```

### 2.LoginController 登陆控制器

```java
@Controller
public class LoginController {

    @GetMapping("/login")
    public String login() {
        return "login";
    }

    @PostMapping("/login")
    @ResponseBody
    public R login(String username, String password, Boolean rememberMe) {
        // 密码MD5加密
        password = MD5Utils.encrypt(username, password);
        UsernamePasswordToken token = new UsernamePasswordToken(username, password, rememberMe);
        // 获取Subject对象
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(token);
            return R.ok();
        } catch (UnknownAccountException e) {
            return R.error(e.getMessage());
        } catch (IncorrectCredentialsException e) {
            return R.error(e.getMessage());
        } catch (LockedAccountException e) {
            return R.error(e.getMessage());
        } catch (AuthenticationException e) {
            return R.error("认证失败！");
        }
    }

    @RequestMapping("/")
    public String redirectIndex() {
        return "redirect:/index";
    }

    @RequestMapping("/index")
    public String index(Model model) {
        // 登录成后，即可通过Subject获取登录的用户信息
        User user = (User) SecurityUtils.getSubject().getPrincipal();
        model.addAttribute("user", user);
        return "index";
    }

    @GetMapping("/403")
    public String forbid() {
        return "403";
    }
}
```



## 6.验证

　　启动项目，访问login页面，我本地是 http://localhost:8888/login ，可以看到，输入用户名和密码(用户名、密码都是test)后，跳进了首页

 ![Shiro登陆成功跳转]({{site.url}}/assets/2019-12-16-SpringBoot_Shiro/login.jpg)