---
layout:     post
title:      Spring Boot集成Shiro + redis
subtitle:   介绍一下通过基于角色的权限验证的Shiro配置
date:       2019-03-01
author:     Reed
header-img: images/2019-3/IMG_20190220_090030.jpg
catalog: true
tags:
    - Spring Boot
---
# 前言

在Spring中主流的安全框架有Spring Security和Apache Shiro，两者各有特点，这里介绍一下Apache Shiro的简单配置（使用redis存储缓存）。

# 正文
#### 配置依赖
在`build.gradle`中引入所需依赖(当然提前安装好redis是必需的)
``` gradle
    // https://mvnrepository.com/artifact/org.crazycake/shiro-redis
    compile group: 'org.crazycake', name: 'shiro-redis', version: '3.2.2'
    // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-data-redis', version: '2.1.2.RELEASE'
    // shiro
    compile group: 'org.apache.shiro', name: 'shiro-spring', version: '1.4.0'
```


#### 创建`ShiroConfigure.java`
``` java
@Configuration
public class ShiroConfigure {
    // 添加自己的验证方式
    @Bean
    public ShiroRealm shiroRealm() {
        return new ShiroRealm();
    }

    // 权限管理，配置主要是Realm的管理认证
    @Bean
    public SecurityManager securityManager() {

        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(shiroRealm());
        securityManager.setCacheManager(cacheManager());
        securityManager.setSessionManager(sessionManger());
        return securityManager;
    }

    /**
     * 功能描述: 配置shiro redisManager
     * @Param: []
     * @Return: org.crazycake.shiro.RedisManager
     */
    public RedisManager redisManager(){
        RedisManager redisManager = new RedisManager();
        return redisManager;
    }

    /**
     * 功能描述: cacheManager 缓存 redis 实现
     * @Param: []
     * @Return: org.crazycake.shiro.RedisCacheManager
     */
    public RedisCacheManager cacheManager(){
        RedisCacheManager redisCacheManager = new RedisCacheManager();
        redisCacheManager.setRedisManager(redisManager());
        return redisCacheManager;
    }

    /**
     * 功能描述: seesion DAO 层的实现 通过redis
     * @Param: []
     * @Return: org.crazycake.shiro.RedisSessionDAO
     */
    @Bean
    public RedisSessionDAO redisSessionDAO(){
        RedisSessionDAO redisSessionDAO = new RedisSessionDAO();
        redisSessionDAO.setRedisManager(redisManager());
        redisSessionDAO.setKeyPrefix("zhuimeng:edu");
        return redisSessionDAO;
    }

    /**
     * 功能描述: shiro session 的管理
     * @Param: []
     * @Return: org.apache.shiro.web.session.mgt.DefaultWebSessionManager
     */
    @Bean
    public DefaultWebSessionManager sessionManger(){
        DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
        sessionManager.setSessionDAO(redisSessionDAO());
        return sessionManager;
    }

    //Filter工厂，设置对应的过滤条件和跳转条件
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);

        //登录 页面 这个先访问 Controller 找对应的url，默认是get方法，再通过方法中的去返回页面
        shiroFilterFactoryBean.setLoginUrl("/login");
        //首页 登陆成功后要跳转的 失败
        shiroFilterFactoryBean.setSuccessUrl("/index");

        // 过滤链执行，从上往下执行的
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        // 以后可能要配置 资源的路径 默认从static下开始， 如果需要验证码 记得开放 不需要权限

        filterChainDefinitionMap.put("/assets/**", "anon");

        //登出
        filterChainDefinitionMap.put("/logout", "logout");
        filterChainDefinitionMap.put("/register", "anon");


        //用户，需要角色权限 “user”
        filterChainDefinitionMap.put("/user/**", "roles[user]");
        //管理员，需要角色权限 “admin”
        filterChainDefinitionMap.put("/admin/**", "roles[admin]");


        //对所有用户认证
        filterChainDefinitionMap.put("/**", "authc");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);


        return shiroFilterFactoryBean;
    }

    //加入注解的使用，不加入这个注解不生效
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }
    /**
     * Shiro生命周期处理器
     */
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor(){
        return new LifecycleBeanPostProcessor();
    }
    /**
     * 自动创建代理
     */
    @Bean
    @DependsOn({"lifecycleBeanPostProcessor"})
    public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator(){
        DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        advisorAutoProxyCreator.setProxyTargetClass(true);
        return advisorAutoProxyCreator;
    }
```
需要注意的是，其中的`filterChainDefinitionMap.put("/user/**", "roles[user]");`，这是基于url的拦截，需要放在`authc`配置的上边，否则不起作用。

如果需要使用更灵活的`@ReqiureRoles`的话，这里可以不配置roles，然后在需要的类或者方法上配置即可。

#### 配置`ShiroRealm.java`
``` java

public class ShiroRealm extends AuthorizingRealm {

    @Autowired
    private UserService userService;

    // 角色权限添加
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        // 获取登录用户名

        //Membership是用户类，这里换用自己的即可
        Long username = ((Membership) principals.getPrimaryPrincipal()).getUsername();
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();

        // 数据库寻找用户 通过 userCode 查找
        Membership user = userService.findByUsername(username);

        if (user == null) {
            
            return null;
        } else {
                simpleAuthorizationInfo.addRole(user.getRole());
            }
            return simpleAuthorizationInfo;

        }
    }

    // 用户认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        if (token.getPrincipal() == null) {
            return null;
        }
        
        long name = Long.parseLong(token.getPrincipal().toString());

        Membership user = userService.findByUsername(name);
        SimpleAuthenticationInfo simpleAuthenticationInfo = null;
        if (user == null) {
            throw new AuthenticationException("用户[" + name + "]不存在");
        } else {
                simpleAuthenticationInfo = new SimpleAuthenticationInfo(user, user.getPassword(), getName());
        }
        return simpleAuthenticationInfo;
    }
}
```
这里需要的注意的是，`SimpleAuthenticationInfo`传递过去的应该是一个用户对象。
#### 配置登录接口
``` java
    @RequestMapping(value = "/login", method = RequestMethod.POST)
    @ResponseBody
    public Result login(@RequestBody Map<String, String> userMap, HttpServletRequest servletRequest) throws Exception {
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken usernamePasswordToken = null;
        long username = Long.parseLong(userMap.get("username").toString());
        String password = userMap.get("password");

        // token
        if (password != null) {
            Membership user = userService.findByUsername(username);
            if (user != null) {
                if (CodeUtil.passwordEncoder(password).equals(user.getPassword())) {
                    usernamePasswordToken = new UsernamePasswordToken(
                            user.getUsername(), user.getPassword()
                    );
                    subject.login(usernamePasswordToken);
                    log.info("用户 " + username + " 登陆成功！");
                    return Result.success(user);
                }
            }
        }
        return Result.failure(2004);
    }
```

这样就配置好了shiro和redis，有一个好处是调测试接口和页面的时候不需要每次都登陆了，因为信息是存放在redis里面的了，只要cookie正确就没问题。