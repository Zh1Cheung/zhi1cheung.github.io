---
title: Spring Security源码分析
categories:
- Spring
tags:
- Spring Security 

---




## Spring Security认证过程

Spring Security是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring IoC，DI（控制反转Inversion of Control ,DI:Dependency Injection 依赖注入）和AOP（面向切面编程）功能，为应用系统提供声明式的安全访问控制功能，减少了为企业系统安全控制编写大量重复代码的工作。 


![image](http://dandandeshangni.oss-cn-beijing.aliyuncs.com/github/Spring%20Security/core-classdiagram.png)

### 核心验证器
#### AuthenticationManager
该对象提供了认证方法的入口，接收一个Authentiaton对象作为参数;

```
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication)
            throws AuthenticationException;
}
```

#### ProviderManager
- 它是 AuthenticationManager 的一个**实现类**，提供了基本的**认证逻辑和方法**；
- 它包含了一个 List<AuthenticationProvider> 对象，通过 **AuthenticationProvider接口来扩展**出不同的认证提供者(当Spring Security默认提供的实现类不能满足需求的时候可以扩展AuthenticationProvider 覆盖supports(Class<?> authentication)方法)；

### 验证逻辑
**AuthenticationManager** 接收 **Authentication 对象**作为参数，并通过 **authenticate(Authentication)** 方法对其进行验证；**AuthenticationProvider实现类**用来支撑对 Authentication 对象的**验证动作**；**UsernamePasswordAuthenticationToken****实现了 Authentication**主要是将用户输入的用户名和密码进行**封装**，并供给 **AuthenticationManager** 进行验证；验证完成以后将返回一个认证成功的 Authentication 对象；

#### Authentication

```
public interface Authentication extends Principal, Serializable {
    //#1.权限结合，可使用AuthorityUtils.commaSeparatedStringToAuthorityList("admin,ROLE_ADMIN")返回字符串权限集合
    Collection<? extends GrantedAuthority> getAuthorities();
    //#2.用户名密码认证时可以理解为密码
    Object getCredentials();
    //#3.认证时包含的一些信息。
    Object getDetails();
    //#4.用户名密码认证时可理解时用户名
    Object getPrincipal();
    #5.是否被认证，认证为true    
    boolean isAuthenticated();
    #6.设置是否能被认证
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
```

#### ProviderManager
ProviderManager是AuthenticationManager的实现类，提供了基本认证实现逻辑和流程；

```
public Authentication authenticate(Authentication authentication)
            throws AuthenticationException {
        //#1.获取当前的Authentication的认证类型
        Class<? extends Authentication> toTest = authentication.getClass();
        AuthenticationException lastException = null;
        Authentication result = null;
        boolean debug = logger.isDebugEnabled();
        //#2.遍历所有的providers使用supports方法判断该provider是否支持当前的认证类型，不支持的话继续遍历
        for (AuthenticationProvider provider : getProviders()) {
            if (!provider.supports(toTest)) {
                continue;
            }

            if (debug) {
                logger.debug("Authentication attempt using "
                        + provider.getClass().getName());
            }

            try {
                #3.支持的话调用provider的authenticat方法认证
                result = provider.authenticate(authentication);

                if (result != null) {
                    #4.认证通过的话重新生成Authentication对应的Token
                    copyDetails(authentication, result);
                    break;
                }
            }
            catch (AccountStatusException e) {
                prepareException(e, authentication);
                // SEC-546: Avoid polling additional providers if auth failure is due to
                // invalid account status
                throw e;
            }
            catch (InternalAuthenticationServiceException e) {
                prepareException(e, authentication);
                throw e;
            }
            catch (AuthenticationException e) {
                lastException = e;
            }
        }

        if (result == null && parent != null) {
            // Allow the parent to try.
            try {
                #5.如果#1 没有验证通过，则使用父类型AuthenticationManager进行验证
                result = parent.authenticate(authentication);
            }
            catch (ProviderNotFoundException e) {
                // ignore as we will throw below if no other exception occurred prior to
                // calling parent and the parent
                // may throw ProviderNotFound even though a provider in the child already
                // handled the request
            }
            catch (AuthenticationException e) {
                lastException = e;
            }
        }
        #6. 是否擦除遍历所有的 Providers，然后依次执行该 Provider 的验证方法 
如果某一个 Provider 验证成功，则跳出循环不再执行后续的验证；
如果验证成功，会将返回的 result 既 Authentication 对象进一步封装为 Authentication Token； 
比如 UsernamePasswordAuthenticationToken、RememberMeAuthenticationToken 等；这些 Authentication Token 也都继承自 Authentication 对象；
如果 #1 没有任何一个 Provider 验证成功，则试图使用其 parent Authentication Manager 进行验证；
是否需要擦除密码等敏感信息；敏感信息
        if (result != null) {
            if (eraseCredentialsAfterAuthentication
                    && (result instanceof CredentialsContainer)) {
                // Authentication is complete. Remove credentials and other secret data
                // from authentication
                ((CredentialsContainer) result).eraseCredentials();
            }

            eventPublisher.publishAuthenticationSuccess(result);
            return result;
        }

        // Parent was null, or didn't authenticate (or throw an exception).

        if (lastException == null) {
            lastException = new ProviderNotFoundException(messages.getMessage(
                    "ProviderManager.providerNotFound",
                    new Object[] { toTest.getName() },
                    "No AuthenticationProvider found for {0}"));
        }

        prepareException(lastException, authentication);

        throw lastException;
    }
```



1. 遍历所有的 **Providers**，然后依次执行该 Provider 的验证方法 
   1. 如果某一个 Provider 验证成功，则跳出循环不再执行后续的验证；
   2. 如果验证成功，会将返回的 result 既 Authentication 对象**进一步封装为 Authentication Token**； 
比如 UsernamePasswordAuthenticationToken、RememberMeAuthenticationToken 等；这些 Authentication Token 也都继承自 Authentication 对象
2. 如果 #1 没有任何一个 Provider 验证成功，则试图使用其**parent Authentication Manager** 进行验证；
3. 是否需要擦除密码等敏感信息；



#### AuthenticationProvider

**ProviderManager** 通过 **AuthenticationProvider** 扩展出更多的验证提供的方式；而 AuthenticationProvider 本身也就是一个接口，从类图中我们可以看出它的实现类AbstractUserDetailsAuthenticationProvider和AbstractUserDetailsAuthenticationProvider的子类DaoAuthenticationProvider**。DaoAuthenticationProvider**是Spring Security中一个核心的Provider,对所有的数据库提供了基本方法和入口。


#### DaoAuthenticationProvider

1. 对用户身份尽心加密操作,通过注**入UserDetailsService接口对象**，并调用其**接口方法 loadUserByUsername(String username)** 获取得到相关的用户信息。


```
//获取用户信息的扩展点
protected final UserDetails retrieveUser(String username,
            UsernamePasswordAuthenticationToken authentication)
            throws AuthenticationException {
        UserDetails loadedUser;

        try {
            loadedUser = this.getUserDetailsService().loadUserByUsername(username);
        }
```

2. 实现 additionalAuthenticationChecks 的验证方法(主要验证密码)




#### AbstractUserDetailsAuthenticationProvider
AbstractUserDetailsAuthenticationProvider为DaoAuthenticationProvider提供了基本的认证方法；

```
public Authentication authenticate(Authentication authentication)
            throws AuthenticationException {
        Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
                messages.getMessage(
                        "AbstractUserDetailsAuthenticationProvider.onlySupports",
                        "Only UsernamePasswordAuthenticationToken is supported"));

        // Determine username
        String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED"
                : authentication.getName();

        boolean cacheWasUsed = true;
        UserDetails user = this.userCache.getUserFromCache(username);

        if (user == null) {
            cacheWasUsed = false;

            try {
                #1.获取用户信息由子类实现即DaoAuthenticationProvider
                user = retrieveUser(username,
                        (UsernamePasswordAuthenticationToken) authentication);
            }
            catch (UsernameNotFoundException notFound) {
                logger.debug("User '" + username + "' not found");

                if (hideUserNotFoundExceptions) {
                    throw new BadCredentialsException(messages.getMessage(
                            "AbstractUserDetailsAuthenticationProvider.badCredentials",
                            "Bad credentials"));
                }
                else {
                    throw notFound;
                }
            }

            Assert.notNull(user,
                    "retrieveUser returned null - a violation of the interface contract");
        }

        try {
            #2.前检查由DefaultPreAuthenticationChecks类实现（主要判断当前用户是否锁定，过期，冻结User接口）
            preAuthenticationChecks.check(user);
            #3.子类实现
            additionalAuthenticationChecks(user,
                    (UsernamePasswordAuthenticationToken) authentication);
        }
        catch (AuthenticationException exception) {
            if (cacheWasUsed) {
                // There was a problem, so try again after checking
                // we're using latest data (i.e. not from the cache)
                cacheWasUsed = false;
                user = retrieveUser(username,
                        (UsernamePasswordAuthenticationToken) authentication);
                preAuthenticationChecks.check(user);
                additionalAuthenticationChecks(user,
                        (UsernamePasswordAuthenticationToken) authentication);
            }
            else {
                throw exception;
            }
        }
        #4.检测用户密码是否过期对应#2 的User接口
        postAuthenticationChecks.check(user);

        if (!cacheWasUsed) {
            this.userCache.putUserInCache(user);
        }

        Object principalToReturn = user;

        if (forcePrincipalAsString) {
            principalToReturn = user.getUsername();
        }

        return createSuccessAuthentication(principalToReturn, authentication, user);
    }
```



1. 获取用户 **返回UserDetails** 
AbstractUserDetailsAuthenticationProvider定义了一个抽象的方法 
2. 三步验证工作
    1. preAuthenticationChecks
    2. additionalAuthenticationChecks（抽象方法，子类实现）
    3. postAuthenticationChecks
3. 将已通过验证的用户信息封装成 **UsernamePasswordAuthenticationToken** 对象并返回；该对象封装了用户的身份信息
```
protected Authentication createSuccessAuthentication(Object principal,
        UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken(
                principal, authentication.getCredentials(),
                authoritiesMapper.mapAuthorities(user.getAuthorities()));
        result.setDetails(authentication.getDetails());

        return result;
    }
```
#### UserDetailsService


```
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

通过用户名 username 调用方法 loadUserByUsername **返回了一个UserDetails接口对象**；

```
public interface UserDetails extends Serializable {
    #1.权限集合
    Collection<? extends GrantedAuthority> getAuthorities();
    #2.密码   
    String getPassword();
    #3.用户民
    String getUsername();
    #4.用户是否过期
    boolean isAccountNonExpired();
    #5.是否锁定 
    boolean isAccountNonLocked();
    #6.用户密码是否过期 
    boolean isCredentialsNonExpired();
    #7.账号是否可用（可理解为是否删除）
    boolean isEnabled();
}
```

#### JdbcUserDetailsManager
Spring 为UserDetailsService默认提供了一个实现类 org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl，该实现类主要是提供基于JDBC对 User 进行增、删、查、改的方法。


```
public class JdbcUserDetailsManager extends JdbcDaoImpl implements UserDetailsManager,
        GroupManager {
    // ~ Static fields/initializers
    // =====================================================================================

    // UserDetailsManager SQL
    #1.定义了一些列对数据库操作的语句
    public static final String DEF_CREATE_USER_SQL = "insert into users (username, password, enabled) values (?,?,?)";
    public static final String DEF_DELETE_USER_SQL = "delete from users where username = ?";
    public static final String DEF_UPDATE_USER_SQL = "update users set password = ?, enabled = ? where username = ?";
    public static final String DEF_INSERT_AUTHORITY_SQL = "insert into authorities (username, authority) values (?,?)";
    public static final String DEF_DELETE_USER_AUTHORITIES_SQL = "delete from authorities where username = ?";
    public static final String DEF_USER_EXISTS_SQL = "select username from users where username = ?";
    public static final String DEF_CHANGE_PASSWORD_SQL = "update users set password = ? where username = ?";

}
```

#### InMemoryUserDetailsManager
该实现类主要是提供基于内存对 User 进行增、删、查、改的方法 。

```
public class InMemoryUserDetailsManager implements UserDetailsManager { 
protected final Log logger = LogFactory.getLog(getClass());}
```


### 总结


- **UserDetailsService接口**作为桥梁，是DaoAuthenticationProvier与特定用户信息来源**进行解耦的地方**
- UserDetailsService由UserDetails（对基本用户信息进行封装）和UserDetailsManager（对基本用户信息进行管理）所**构成**
- UserDetailsService、UserDetails以及UserDetailsManager都是**可被用户自定义的扩展点**，我们可以继承这些接口提供自己的读取用户来源和管理用户的方法。

![image](http://dandandeshangni.oss-cn-beijing.aliyuncs.com/github/Spring%20Security/core-service-Sequence.png)


## Spring Security授权过程
### 调试过程
比较重要的 Filter 的处理逻辑，UsernamePasswordAuthenticationFilter，AnonymousAuthenticationFilter，ExceptionTranslationFilter，FilterSecurityInterceptor 
#### UsernamePasswordAuthenticationFilter
整个调用流程是，先调用其父类 **AbstractAuthenticationProcessingFilter.doFilter() 方法**，然后再执行 UsernamePasswordAuthenticationFilter.attemptAuthentication() 方法进行验证；

```
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;
        #1.判断当前的filter是否可以处理当前请求，不可以的话则交给下一个filter处理
        if (!requiresAuthentication(request, response)) {
            chain.doFilter(request, response);

            return;
        }

        if (logger.isDebugEnabled()) {
            logger.debug("Request is to process authentication");
        }

        Authentication authResult;

        try {
            #2.抽象方法由子类UsernamePasswordAuthenticationFilter实现
            authResult = attemptAuthentication(request, response);
            if (authResult == null) {
                // return immediately as subclass has indicated that it hasn't completed
                // authentication
                return;
            }
            #2.认证成功后，处理一些与session相关的方法 
            sessionStrategy.onAuthentication(authResult, request, response);
        }
        catch (InternalAuthenticationServiceException failed) {
            logger.error(
                    "An internal error occurred while trying to authenticate the user.",
                    failed);
            #3.认证失败后的的一些操作
            unsuccessfulAuthentication(request, response, failed);

            return;
        }
        catch (AuthenticationException failed) {
            // Authentication failed
            unsuccessfulAuthentication(request, response, failed);

            return;
        }

        // Authentication success
        if (continueChainBeforeSuccessfulAuthentication) {
            chain.doFilter(request, response);
        }
        #3. 认证成功后的相关回调方法 主要将当前的认证放到SecurityContextHolder中
        successfulAuthentication(request, response, chain, authResult);
    }
```
整个程序的执行流程如下: 
1. 判断filter是否可以处理当前的请求，如果不可以则放行交给下一个filter 
2. 调用抽象方法**attemptAuthentication**进行验证，该方法由子类**UsernamePasswordAuthenticationFilter实现** 
3. 认证成功以后，回调一些与 session 相关的方法； 
4. 认证成功以后，认证成功后的相关回调方法；认证成功以后，认证成功后的相关回调方法；

```
protected void successfulAuthentication(HttpServletRequest request,
            HttpServletResponse response, FilterChain chain, Authentication authResult)
            throws IOException, ServletException {

        if (logger.isDebugEnabled()) {
            logger.debug("Authentication success. Updating SecurityContextHolder to contain: "
                    + authResult);
        }

        SecurityContextHolder.getContext().setAuthentication(authResult);

        rememberMeServices.loginSuccess(request, response, authResult);

        // Fire event
        if (this.eventPublisher != null) {
            eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(
                    authResult, this.getClass()));
        }

        successHandler.onAuthenticationSuccess(request, response, authResult);
    }
```
1. 将当前认证成功的 **Authentication** 放置到 SecurityContextHolder 中；
2. 调用其它可扩展的 handlers 继续处理该认证成功以后的回调事件；（实现**uthenticationSuccessHandler接口**即可）


#### UsernamePasswordAuthenticationFilter


```
public Authentication attemptAuthentication(HttpServletRequest request,
            HttpServletResponse response) throws AuthenticationException {
        #1.判断请求的方法必须为POST请求
        if (postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException(
                    "Authentication method not supported: " + request.getMethod());
        }
        #2.从request中获取username和password
        String username = obtainUsername(request);
        String password = obtainPassword(request);

        if (username == null) {
            username = "";
        }

        if (password == null) {
            password = "";
        }

        username = username.trim();
        #3.构建UsernamePasswordAuthenticationToken（两个参数的构造方法setAuthenticated(false)）
        UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
                username, password);

        // Allow subclasses to set the "details" property
        setDetails(request, authRequest);
        #4. 调用 AuthenticationManager 进行验证（子类ProviderManager遍历所有的AuthenticationProvider认证）
        return this.getAuthenticationManager().authenticate(authRequest);
    }
```
1. 认证请求的方法必须为POST
2. 从request中获取 username 和 password
3. 封装Authenticaiton的实现类**UsernamePasswordAuthenticationToken**，（UsernamePasswordAuthenticationToken调用两个参数的构造方法setAuthenticated(false)）
4. 调用 **AuthenticationManager** 的 **authenticate 方法**进行验证；

#### AnonymousAuthenticationFilter

AnonymousAuthenticationFilter过滤器是在UsernamePasswordAuthenticationFilter等过滤器**之后**，如果它前面的过滤器都**没有认证成功**，Spring Security则为当前的SecurityContextHolder中添加一个Authenticaiton 的**匿名实现类AnonymousAuthenticationToken**;

```
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        #1.如果前面的过滤器都没认证通过，则SecurityContextHolder中Authentication为空
        if (SecurityContextHolder.getContext().getAuthentication() == null) {
            #2.为当前的SecurityContextHolder中添加一个匿名的AnonymousAuthenticationToken
            SecurityContextHolder.getContext().setAuthentication(
                    createAuthentication((HttpServletRequest) req));

            if (logger.isDebugEnabled()) {
                logger.debug("Populated SecurityContextHolder with anonymous token: '"
                        + SecurityContextHolder.getContext().getAuthentication() + "'");
            }
        }
        else {
            if (logger.isDebugEnabled()) {
                logger.debug("SecurityContextHolder not populated with anonymous token, as it already contained: '"
                        + SecurityContextHolder.getContext().getAuthentication() + "'");
            }
        }

        chain.doFilter(req, res);
    }

    #3.创建匿名的AnonymousAuthenticationToken
    protected Authentication createAuthentication(HttpServletRequest request) {
        AnonymousAuthenticationToken auth = new AnonymousAuthenticationToken(key,
                principal, authorities);
        auth.setDetails(authenticationDetailsSource.buildDetails(request));

        return auth;
    }

        /**
     * Creates a filter with a principal named "anonymousUser" and the single authority
     * "ROLE_ANONYMOUS".
     *
     * @param key the key to identify tokens created by this filter
     */
     ##.创建一个用户名为anonymousUser 授权为ROLE_ANONYMOUS
    public AnonymousAuthenticationFilter(String key) {
        this(key, "anonymousUser", AuthorityUtils.createAuthorityList("ROLE_ANONYMOUS"));
    }
```


1. 判断SecurityContextHolder中Authentication**是否为空**；
2. 如果空则为当前的SecurityContextHolder中添加一个匿名的AnonymousAuthenticationToken（用户名为 anonymousUser 的AnonymousAuthenticationToken）


#### ExceptionTranslationFilter
ExceptionTranslationFilter 异常处理过滤器,该过滤器用来处理在系统认证授权过程中抛出的异常（也就是下一个过滤器FilterSecurityInterceptor）,主要是 处理 AuthenticationException 和 AccessDeniedException 。

```
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;

        try {
            chain.doFilter(request, response);

            logger.debug("Chain processed normally");
        }
        catch (IOException ex) {
            throw ex;
        }
        catch (Exception ex) {
            // Try to extract a SpringSecurityException from the stacktrace
            #.判断是不是AuthenticationException
            Throwable[] causeChain = throwableAnalyzer.determineCauseChain(ex);
            RuntimeException ase = (AuthenticationException) throwableAnalyzer
                    .getFirstThrowableOfType(AuthenticationException.class, causeChain);

            if (ase == null) {
                #. 判断是不是AccessDeniedException
                ase = (AccessDeniedException) throwableAnalyzer.getFirstThrowableOfType(
                        AccessDeniedException.class, causeChain);
            }

            if (ase != null) {
                handleSpringSecurityException(request, response, chain, ase);
            }
            else {
                // Rethrow ServletExceptions and RuntimeExceptions as-is
                if (ex instanceof ServletException) {
                    throw (ServletException) ex;
                }
                else if (ex instanceof RuntimeException) {
                    throw (RuntimeException) ex;
                }

                // Wrap other Exceptions. This shouldn't actually happen
                // as we've already covered all the possibilities for doFilter
                throw new RuntimeException(ex);
            }
        }
    }
```

#### FilterSecurityInterceptor
此过滤器为认证授权过滤器链中最后一个过滤器，该过滤器之后就是请求真正的/persons 服务

```
public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException {
        FilterInvocation fi = new FilterInvocation(request, response, chain);
        invoke(fi);
    }

public void invoke(FilterInvocation fi) throws IOException, ServletException {
        if ((fi.getRequest() != null)
                && (fi.getRequest().getAttribute(FILTER_APPLIED) != null)
                && observeOncePerRequest) {
            // filter already applied to this request and user wants us to observe
            // once-per-request handling, so don't re-do security checking
            fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
        }
        else {
            // first time this request being called, so perform security checking
            if (fi.getRequest() != null) {
                fi.getRequest().setAttribute(FILTER_APPLIED, Boolean.TRUE);
            }
            #1. before invocation重要
            InterceptorStatusToken token = super.beforeInvocation(fi);

            try {
                #2. 可以理解开始请求真正的 /persons 服务
                fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
            }
            finally {
                super.finallyInvocation(token);
            }
            #3. after Invocation
            super.afterInvocation(token, null);
        }
    }
```


#### before invocation: AccessDecisionManager

```
protected InterceptorStatusToken beforeInvocation(Object object) {
        ...

        Collection<ConfigAttribute> attributes = this.obtainSecurityMetadataSource()
                .getAttributes(object);

        ...
        Authentication authenticated = authenticateIfRequired();

        // Attempt authorization
        try {
    
            //调用 AccessDecisionManager 来验证当前已认证成功的用户是否有权限访问该资源；
            this.accessDecisionManager.decide(authenticated, object, attributes);
        }
        catch (AccessDeniedException accessDeniedException) {
            publishEvent(new AuthorizationFailureEvent(object, attributes, authenticated,accessDeniedException));

            throw accessDeniedException;
        }

        ...
    }
```

#### attributes和object 
```
Collection<ConfigAttribute> attributes = this.obtainSecurityMetadataSource()
                .getAttributes(object);
```

调试 ：object为当前请求的 url:/persons, 那么getAttributes方法就是使用当前的访问资源路径去匹配我们自己定义的匹配规则。

#### AccessDecisionManager如何授权

Spring Security默认使用AffirmativeBased实现AccessDecisionManager 的 decide 方法来实现授权

```
public void decide(Authentication authentication, Object object,
            Collection<ConfigAttribute> configAttributes) throws AccessDeniedException {
        int deny = 0;
        #1.调用AccessDecisionVoter 进行vote(投票)
        for (AccessDecisionVoter voter : getDecisionVoters()) {
            int result = voter.vote(authentication, object, configAttributes);

            if (logger.isDebugEnabled()) {
                logger.debug("Voter: " + voter + ", returned: " + result);
            }

            switch (result) {
            #1.1只要有voter投票为ACCESS_GRANTED，则通过 直接返回
            case AccessDecisionVoter.ACCESS_GRANTED://1
                return;
            @#1.2只要有voter投票为ACCESS_DENIED，则记录一下
            case AccessDecisionVoter.ACCESS_DENIED://-1
                deny++;

                break;

            default:
                break;
            }
        }

        if (deny > 0) {
        #2.如果有两个及以上AccessDecisionVoter(姑且称之为投票者吧)都投ACCESS_DENIED，则直接就不通过了
            throw new AccessDeniedException(messages.getMessage(
                    "AbstractAccessDecisionManager.accessDenied", "Access is denied"));
        }

        // To get this far, every AccessDecisionVoter abstained
        checkAllowIfAllAbstainDecisions();
    }
```


#### WebExpressionVoter.vote()


```
public int vote(Authentication authentication, FilterInvocation fi,
            Collection<ConfigAttribute> attributes) {
        assert authentication != null;
        assert fi != null;
        assert attributes != null;

        WebExpressionConfigAttribute weca = findConfigAttribute(attributes);

        if (weca == null) {
            return ACCESS_ABSTAIN;
        }

        EvaluationContext ctx = expressionHandler.createEvaluationContext(authentication,
                fi);
        ctx = weca.postProcess(ctx, fi);

        return ExpressionUtils.evaluateAsBoolean(weca.getAuthorizeExpression(), ctx) ? ACCESS_GRANTED
                : ACCESS_DENIED;
    }
```

到此位置authentication当前用户信息，fl当前访问的资源路径及attributes当前资源路径的决策（即是否需要认证）。剩下就是判断当前用户的角色Authentication.authorites是否权限访问决策访问当前资源fi。
![image](http://dandandeshangni.oss-cn-beijing.aliyuncs.com/github/Spring%20Security/authenorization-Sequence%20Diagram0.png)





