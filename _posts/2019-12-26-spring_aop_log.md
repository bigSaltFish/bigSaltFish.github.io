---
layout: default
title: SpringBoot使用aop记录log
#excerpt: 

---

　　Spring的aop功能，除了处理事务，还可以记录日志(文章复用了 SpringBoot_Shiro 中的配置和代码)



# 1.加入maven依赖

　　这里只列举出 aop的依赖，其他的依赖，比如spring boot、jdbc的依赖，可以参考之前文章里的配置

```java
<!-- aop依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```



# 2.自定义log注解

　　自定义log注解，该注解使用在方法上，若方法被log注解标记，则会被记录日志

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value() default "";
}
```

　　然后，添加到方法上，比如，应用在login方法上

```java
@Log("请求登录")
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
```


# 3.切面和日志保存

　　定义一个aspect类，使用@Aspect注解标记，定义切入点，符合切入点的方法，再判断方法上是否有步骤2中定义的log注解，若包含，则日志入库。

    @Aspect
    @Component
    public class LogAspect {
        @Autowired
        private SysLogMapper sysLogMapper;
    
        // 扫描指定注解
    //        @Pointcut("@annotation(com.frank.annotation.Log)")
    
        // 扫描指定包下所有controller
        @Pointcut("execution(public * com.frank.controller..*.*(..))")
        public void pointcut() {
        }
    
        @Around("pointcut()")
        public Object around(ProceedingJoinPoint point) {
            Object result = null;
            long beginTime = System.currentTimeMillis();
            try {
                // 执行方法
                result = point.proceed();
            } catch (Throwable e) {
                e.printStackTrace();
            }
            // 执行时长(毫秒)
            long time = System.currentTimeMillis() - beginTime;
            // 保存日志
            saveLog(point, time);
            return result;
        }


```java
    /**
     * 保存日志
     *
     * @param joinPoint
     * @param time
     */
    private void saveLog(ProceedingJoinPoint joinPoint, long time) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        SysLog sysLog = new SysLog();
        Log logAnnotation = method.getAnnotation(Log.class);
        if (logAnnotation != null) {
            // 注解上的描述
            sysLog.setOperation(logAnnotation.value());
        }
        // 请求的方法名
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = signature.getName();
        sysLog.setMethod(className + "." + methodName + "()");
        // 请求的方法参数值
        Object[] args = joinPoint.getArgs();
        // 请求的方法参数名称
        LocalVariableTableParameterNameDiscoverer u = new LocalVariableTableParameterNameDiscoverer();
        String[] paramNames = u.getParameterNames(method);
        if (args != null && paramNames != null) {
            String params = "";
            for (int i = 0; i < args.length; i++) {
                params += "  " + paramNames[i] + ": " + args[i];
            }
            sysLog.setParams(params);
        }
        // 获取request
        HttpServletRequest request = HttpContextUtils.getHttpServletRequest();
        // 设置IP地址
        sysLog.setIp(IPUtils.getIpAddr(request));
        // 模拟一个用户名
        sysLog.setUsername("神秘人");
        sysLog.setTime((int) time);
        // 保存系统日志
        sysLogMapper.save(sysLog);
    }
}
```



# 4.日志保存的方法和建表sql

　　sysLogMapper的save()，是使用mybatis的方法，里面就一句 insert语句

```sql
<insert id="save" parameterType="com.frank.model.SysLog">
insert into sys_log(`USERNAME`,`OPERATION`,`TIME`,`METHOD`,`PARAMS`,`IP`) values (#{username},#{operation},#{time},#{method},#{params},#{ip})
</insert>
```

　　建表sql

```sql
CREATE TABLE `sys_log` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `USERNAME` varchar(255) NOT NULL COMMENT '用户名',
  `OPERATION` varchar(255) DEFAULT NULL COMMENT '用户操作',
  `TIME` int(11) NOT NULL COMMENT '响应时长',
  `METHOD` varchar(200) NOT NULL COMMENT '请求方法',
  `PARAMS` varchar(255) NOT NULL COMMENT '请求参数',
  `IP` varchar(255) NOT NULL DEFAULT '127.0.0.1' COMMENT 'IP地址',
  `CREATE_TIME` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='日志记录';
```

# 5.测试验证

启动springBoot项目，在浏览器上访问 http://127.0.0.1:8080/login ，输入用户名和密码，登陆成功后，可以查看数据库的记录，发现，有了

![登陆日志]({{site.url}}/assets/2019-12-26-spring_aop_log/aop_log.jpg)

