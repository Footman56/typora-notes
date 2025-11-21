# 重要类

Security 的主要实现就是靠Filter处理的，对各种情况进行**认证**（帐号密码认证、token认证），角色**鉴权**

认证：处理是这个用户是否存在

鉴权：这个用户是否有对应的权限

## SecurityFilterChain

拦截器链进行对请求、响应进行拦截，在拦截的时候实现了一些功能，认证、鉴权

## HttpSecurity

##  WebSecurity

##  AuthenticationManager



##  HTTP Basic



# RBAC 权限模型

用户-角色-权限

一个用户可以有一个或者多个角色。 用户 ：角色 = 1:N

一个角色可以对应一个或者多个权限。 角色：权限 =1 : N



权限可以分为：数据权限、菜单权限、接口权限

其中菜单权限包含接口权限。也就是有个菜单权限的话，就会拥有这个菜单下接口的权限。

数据权限控制的是管理的数据



# 配置方式一

```java
/**
 * 废弃了WebSecurityConfigurerAdapter ,推荐注册bean来配置
 *@author peilizhi
 *@date 2024/1/4 10:53
 **/
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    /**
     * 将用户设置在内存中,不安全,仅测试使用
     * 是全局的
     */
    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails admin = User.withDefaultPasswordEncoder()
                .username("admin")
                .password("admin")
                .roles("ADMIN")
                .build();
        UserDetails customer = User.withDefaultPasswordEncoder()
                .username("customer")
                .password("customer")
                .roles("CUSTOM_ROLE_NORMAL")
                .build();
        // 可以指定多个固定的角色
        return new InMemoryUserDetailsManager(admin, customer);
    }
    /**
     * 用于配置url权限控制
     * @param http
     * @return
     * @throws Exception
     */
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                // 配置请求认证方式
                .authorizeRequests(auth ->
                        auth
                                // /test/echo 任何用户都可以访问
                                .antMatchers("/test/echo").permitAll()
                                // /test/admin 只有ADMIN角色可以访问
                                .antMatchers("/test/admin").hasRole("ADMIN")
                                // /test/normal 只有ROLE_NORMAL角色可以访问
                                .antMatchers("test/normal").hasRole("CUSTOM_ROLE_NORMAL")
                                // 任何请求都需要认证
                                .anyRequest().authenticated())
                // 设置form表单 登录页面,使用默认页面
                .formLogin(Customizer.withDefaults())
                // 设置退出页面
                .logout(Customizer.withDefaults())
                // 设置http base认证 ,使用默认页面
                .httpBasic(Customizer.withDefaults())
                // 关闭csrf
                .csrf().disable();
        return http.build();
    }

    /**
     * 配置要忽略的路径
     * @return
     */
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        // 忽略 error 开头的请求
        return (web) -> web.ignoring().antMatchers("/error")
                // 忽略静态资源路径
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations())
                ;
    }
}
```

这样配置不好，因为把用户角色放在内存中，不方便拓展。并且也不安全

好的方式就是将用户、角色等存放在数据库中。这样安全，也可以自己添加角色



# 配置方式二

```java
/**
 * 废弃了WebSecurityConfigurerAdapter ,推荐注册bean来配置
 *@author peilizhi
 *@date 2024/1/4 10:53
 **/
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Resource
    private SysUserService sysUserService;
    /**
     * token 认证过滤器
     */
    @Resource
    private JwtAuthenticationTokenFilter authenticationTokenFilter;
    /**
     * 登出处理
     */
    @Resource
    private JwtLogoutSuccessHandler jwtLogoutSuccessHandler;
    /**
     * 认证失败处理
     */
    @Resource
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    /**
     * 无权限处理
     */
    @Resource
    private JwtAccessDeniedHandler jwtAccessDeniedHandler;

    /**
     * 权限管理器-用于认证用户，并且含有用户的角色
     * @return
     */
    @Bean
    public AuthenticationManager authenticationManager() {
        DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();
        daoAuthenticationProvider.setUserDetailsService(sysUserService);
        // 强散列哈希加密实现
        daoAuthenticationProvider.setPasswordEncoder(new BCryptPasswordEncoder());
        return new ProviderManager(daoAuthenticationProvider);
    }

    /**
     * 用于配置url权限控制
     * @param http
     * @return
     * @throws Exception
     */
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                // 配置请求认证方式
                .authorizeRequests(auth ->
                        auth
                                // /test/echo 任何用户都可以访问
                                .antMatchers("/test/echo").permitAll()
                                // /test/admin 只有ADMIN角色可以访问
                                .antMatchers("/admin/**").hasRole("ADMIN")
                                // /test/normal 只有ROLE_NORMAL角色可以访问
                                .antMatchers("/normal/**").hasRole("CUSTOM_ROLE_NORMAL")
                                // 对于登录login 验证码captchaImage  logout 允许匿名访问
                                .antMatchers("/login", "/captchaImage").anonymous()
                                // 测试接口任意访问
                                .antMatchers("/test/**").anonymous()
                                // 任何请求都需要认证
                                .anyRequest().authenticated()
                )
                // 设置form表单 登录页面,使用默认页面
                .formLogin(Customizer.withDefaults())
                // 设置退出页面
                .logout()
                // 退出成功处理接口
                .logoutSuccessHandler(jwtLogoutSuccessHandler)
                .and()
                // 设置http base认证 ,使用默认页面
                .httpBasic(Customizer.withDefaults())
                // 认证失败
                .exceptionHandling()
          			// 屏蔽默认的登录页面(前后端分离项目,不能返回页面)
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
                .accessDeniedHandler(jwtAccessDeniedHandler)
                .and()
                // 基于token，所以不需要session
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                // 关闭csrf
                .csrf().disable()
                // 在UsernamePasswordAuthenticationFilter 之前执行token filter
                .addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    /**
     * 配置要忽略的路径
     */
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        // 忽略 error 开头的请求
        return (web) -> web.ignoring().antMatchers("/error", "/login", "logout")
                // 忽略静态资源路径
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations())
                ;
    }
}

```

Authentication

记录用户认证信息、用户角色、是否认证过

```java
public interface Authentication extends Principal, Serializable {

	Collection<? extends GrantedAuthority> getAuthorities();
	Object getCredentials();
	Object getDetails();
	Object getPrincipal();
	boolean isAuthenticated();
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```



# UsernamePasswordAuthenticationFilter

用户密码认证：根据前端传过来的帐号、密码来获取对应的用户。这个帐号密码可以存在内存中InMemoryUserDetailsManager ，也可以从数据库中获取UserDetailsService

AbstractAuthenticationProcessingFilter(父类):

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		if (!requiresAuthentication(request, response)) {
			chain.doFilter(request, response);
			return;
		}
		try {
      // 
			Authentication authenticationResult = attemptAuthentication(request, response);
			if (authenticationResult == null) {
				// return immediately as subclass has indicated that it hasn't completed
				return;
			}
			this.sessionStrategy.onAuthentication(authenticationResult, request, response);
			// Authentication success
			if (this.continueChainBeforeSuccessfulAuthentication) {
				chain.doFilter(request, response);
			}
      // 处理登录成功
			successfulAuthentication(request, response, chain, authenticationResult);
		}
		catch (InternalAuthenticationServiceException failed) {
			this.logger.error("An internal error occurred while trying to authenticate the user.", failed);
      // 处理登录失败
			unsuccessfulAuthentication(request, response, failed);
		}
		catch (AuthenticationException ex) {
			// Authentication failed
			unsuccessfulAuthentication(request, response, ex);
		}
	}
```



UsernamePasswordAuthenticationFilter

```java
@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException {
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
    // 获取帐号密码
		String username = obtainUsername(request);
		username = (username != null) ? username.trim() : "";
		String password = obtainPassword(request);
		password = (password != null) ? password : "";
    // 
		UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,
				password);
		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);
    
    // 调用各种认证方式：帐号密码、token 只有一个验证通过就可以
		return this.getAuthenticationManager().authenticate(authRequest);
	}
```

this.getAuthenticationManager().authenticate(authRequest)

遍历所有认证器去认证，其中AbstractUserDetailsAuthenticationProvider 就是用户密码验证的

```java
for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}
			if (logger.isTraceEnabled()) {
				logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",
						provider.getClass().getSimpleName(), ++currentPosition, size));
			}
			try {
				result = provider.authenticate(authentication);
				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException | InternalAuthenticationServiceException ex) {
				prepareException(ex, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw ex;
			}
			catch (AuthenticationException ex) {
				lastException = ex;
			}
		}
```

```java
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		String username = determineUsername(authentication);
		boolean cacheWasUsed = true;
		UserDetails user = this.userCache.getUserFromCache(username);
		if (user == null) {
			cacheWasUsed = false;
			try {
        // 调用UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
				user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
			}
			catch (UsernameNotFoundException ex) {
			}
		}
	}
```

我们只要实现userDetailsService#loadUserByUsername 就行

```java
@Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 查询数据库的数据
        if (StringUtils.isBlank(username)) {
            return null;
        }
        // 从数据库中，查询用户信息
        SysUserPO user = sysUserManger.getUserByName(username);
        if (null == user) {
            throw new UsernameNotFoundException("用户名不存在");
        }
        Integer status = user.getStatus();
        if (EUserStatusEnum.FORBIDDEN.getType().equals(status)) {
            throw new BaseException(20, "用户已被禁用");
        } else if (EUserStatusEnum.DELETE.getType().equals(status)) {
            throw new BaseException(20, "用户已被删除");
        }
        // 组装用户
        return createDetailUser(user);
    }

    private UserDetails createDetailUser(SysUserPO user) {
        LoginUser loginUser = new LoginUser();

        SysUserDO sysUserDO = new SysUserDO(user);
        loginUser.setUser(sysUserDO);
        Long userId = sysUserDO.getUserId();
        // 查询用户对应的角色
        List<SysRoleDO> roleDOList = sysUserRoleService.getRoleIdsByUserId(userId);
        loginUser.setRoles(roleDOList);
        List<Long> roleIds = StreamUtil.mapToList(roleDOList, SysRoleDO::getRoleId);
        loginUser.setRoleIds(roleIds);
        
        // 查询角色对应的菜单
        List<SysMenuDO> menuDOS = menuService.getMenuIdsByRoleIds(roleIds);
        loginUser.setMenus(menuDOS);
        Map<Long, List<Long>> roleMenuMap = StreamUtil.groupByThenMap(menuDOS, SysMenuDO::getRoleId, SysMenuDO::getMenuId);
        loginUser.setRoleMenuMap(roleMenuMap);
        // 设置权限
        getPermissions(loginUser);
        return loginUser;
    }

    private Set<String> getPermissions(LoginUser loginUser) {
        Set<String> permissions = new HashSet<>();
        SysUserDO user = loginUser.getUser();
        boolean admin = user.isAdmin();
        if (admin) {
            permissions.add("*:*:*");
        } else {
            // 非管理员，获取用户的权限
            List<SysMenuDO> menus = loginUser.getMenus();
            for (SysMenuDO menu : menus) {
                String perms = menu.getPerms();
                if (StringUtils.isNotEmpty(perms)) {
                    permissions.addAll(Arrays.asList(perms.trim().split(",")));
                }
            }
        }
        loginUser.setPermissions(permissions);
        return permissions;
    }
```



# token

Token 验证的逻辑是第一次登录成功之后，生成一个token ，并且返回给前端，前端后续请求的时候在request中携带这个token。服务器获取token 并且通过token来获取用户、认证时拿到用户的角色，使用这个角色鉴权后续操作（就不用登录也能拿到用户信息）

配置的时候将token 拦截放在用户密码登录之前

```java
.addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
```

步骤一：帐号密码登录，生成token

```java
public String login(String username, String password, String code, String uuid) {
        // <1> 验证图片验证码的正确性
        // uuid 的作用，是获得对应的图片验证码
        String verifyKey = BaseConstant.CAPTCHA_CODE_KEY + uuid;
        // 从 Redis 中，获得图片验证码
        String captcha = commonRedisDAO.get(verifyKey);
        // 从 Redis 中，删除图片验证码
        commonRedisDAO.delete(verifyKey);
        // 图片验证码不存在
        if (StringUtils.isEmpty(captcha)) {
            // TODO: 2024/1/7  lizhi 维护异常
            throw new BaseException(20, "图片验证码不存在");
        }
        if (!captcha.equalsIgnoreCase(code)) {
            throw new BaseException(20, "图片验证码错误");
        }
        // 用户验证
        Authentication authentication = null;
        try {
            // 该方法会去调用 UserDetailsServiceImpl.loadUserByUsername
           authentication= authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(username, password));
        } catch (Exception e) {
            if (e instanceof BadCredentialsException) {
                throw new BaseException(20, "用户名或密码错误");
            } else {
                throw new BaseException(30, e.getMessage());
            }
        }
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        return tokenService.createToken(loginUser);
    }
```



```java
/**
     * 创建令牌
     * @param loginUser 用户信息
     * @return 令牌
     */
    public String createToken(LoginUser loginUser) {
        // <1> 设置 LoginUser 的用户唯一标识。注意，这里虽然变量名叫 token ，其实不是身份认证的 Token
        String uuid = IdUtil.fastUUID();
        loginUser.setToken(uuid);
        // <2> 设置用户终端相关的信息，包括 IP、城市、浏览器、操作系统
        setUserAgent(loginUser);
        // <3> 记录缓存
        refreshToken(loginUser);
        // <4> 生成 JWT 的 Token
        Map<String, Object> claims = new HashMap<>();
        claims.put(BaseConstant.LOGIN_USER_TOKEN, uuid);
        return JwtUtil.createJwt(claims);
    }

public void refreshToken(LoginUser loginUser) {
        // 登录时间
        loginUser.setLoginTime(System.currentTimeMillis());
        loginUser.setExpireTime(loginUser.getLoginTime() + REDIS_USER_KEY_EXPIRE_TIME * BaseConstant.MILLIS_MINUTE);
        // 根据 uuid 将 loginUser 缓存
        String redisKey = getRedisUserKey(loginUser.getToken());
        commonRedisDAO.set(redisKey, JsonUtil.toJson(loginUser), REDIS_USER_KEY_EXPIRE_TIME, TimeUnit.MINUTES);
    }
```

```java
// 根据jwt 生成一个token
public static String createJwt(Map<String, Object> claims) {
        Map<String, Object> header = new HashMap<>();
        header.put("typ", jwt);
        header.put("alg", alg);
        Date expiraDate = Date.from(Instant.now().plusSeconds(expireTime * BaseConstant.MILLIS_MINUTE));
        return Jwts.builder().header().add("typ", jwt).add("alg", alg).and()
                // 过期时间
                .expiration(expiraDate)
                // 签发时间
                .issuedAt(new Date())
                // 签发者
                .issuer(jwtIssuer)
                // 主题
                .subject(subject)
                // 设置自定义负载信息payload
                .claims(claims).signWith(KEY, ALGORITHM).compact();
    }
```

# jwt

JWT实际上就是一个字符串，它由三部分组成，**头部**、**载荷**与**签名**。

## 头部

头部用于描述关于该JWT的最基本的信息

```json
{
  "typ": "JWT",
  # 加密算法
  "alg": "HS256"
}
```

对头部进行Base64 编码，得到一个StringA

## 载荷

```json
{
   # 该JWT的签发者
    "iss": "John Wu JWT",
  # 在什么时候签发的
    "iat": 1441593502,
  # 什么时候过期，这里是一个Unix时间戳
    "exp": 1441594722,
  # 接收该JWT的一方
    "aud": "www.example.com",
  # 该JWT所面向的用户
    "sub": "jrocket@example.com",
  # 自定义字段
    "from_user": "B",
    "target_user": "A"
}
```

对载荷进行Base64 编码，得到一个StringB

## 签名

将 StringA通过StringB  . 拼接成StringC ,将StringC 按照头部配置的算法加密，得到一个StringD.加密的时候需要提供一个私钥【自己指定】

最终结果是StringA.StringB.String

```
StringA.StringB = StringC
StringC + (secret) ->加密  = StringD
result = StringA.StringB.StringD
```

在JWT中，不应该在载荷里面加入任何敏感的数据，可以存储用户id



步骤二，token拦截根据之前登录获取员工

```java
/**
 *@author peilizhi
 *@date 2024/1/7 20:21
 **/
@Component
@Slf4j
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Resource
    private TokenService tokenService;
    @Resource
    private CommonRedisDAO commonRedisDAO;
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 从请求中获取token
        String authorization = request.getHeader(BaseConstant.TOKEN);
        // 如果没有token 就放行
        if (StringUtil.isEmpty(authorization)) {
            filterChain.doFilter(request, response);
        }
        // 解析jwt token
        Claims claims = JwtUtil.parsePayload(authorization);
        if (null == claims) {
            throw new JwtException("token 异常");
        }
        boolean tokenExpired = JwtUtil.isTokenExpired(claims);
        if (tokenExpired) {
            throw new JwtException("token 已过期");
        }
        LoginUser loginUser = null;
        try {
            // 获取jwt 中存储的员工信息
            String token = (String) claims.get(BaseConstant.LOGIN_USER_TOKEN);
            if (StringUtil.isNotBlank(token)) {
                // 从缓存中获取对象
                String redisKey = BaseConstant.REDIS_LOGIN_USER + token;

                String value = commonRedisDAO.get(redisKey);
                if (StringUtil.isNotBlank(value)) {
                    loginUser = JsonUtil.fromJson(value, LoginUser.class);
                }
            }
        } catch (Exception e) {
            log.error("token 解析失败", e);
            // 继续放行
            filterChain.doFilter(request, response);
        }

        // 缓存中有员工信息 并且之前没有认证过
        if (null != loginUser && null == SecurityUtil.getAuthentication()) {
            // 校验令牌是否失效
            long expireTime = loginUser.getExpireTime();
            long currentTime = System.currentTimeMillis();
            // 相差不足 30 分钟，自动刷新缓存
            boolean expired = expireTime - currentTime <= expireTime * BaseConstant.MILLIS_MINUTE;
            if (expired) {
                String token = loginUser.getToken();
                loginUser.setToken(token);
                tokenService.refreshToken(loginUser);
            }
            // 将Authentication 放入容器中,后面的请求就可以直接使用
            UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(loginUser, null, loginUser.getAuthorities());
            authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        }
        // 放行
        filterChain.doFilter(request, response);
    }
}
```



# 自定义邮箱配置

实现逻辑完成参考UsernamePasswordAuthenticationFilter 配置原理



实现EmailAuthenticationFilter,继承 AbstractAuthenticationProcessingFilter ，重新doFilter方法里面的认证逻辑

```java
/**
 *
 *@author peilizhi
 *@date 2024/1/10 15:28
 **/
public class EmailAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    public static final String SPRING_SECURITY_FORM_EMAIL_KEY = "email";
    public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";
    private static final AntPathRequestMatcher DEFAULT_ANT_PATH_REQUEST_MATCHER = new AntPathRequestMatcher("/login-email", "POST");

    private String emailParameter = SPRING_SECURITY_FORM_EMAIL_KEY;
    private String passwordParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;
    private boolean postOnly = true;

    public EmailAuthenticationFilter() {
        super(DEFAULT_ANT_PATH_REQUEST_MATCHER);
    }

    public EmailAuthenticationFilter(AuthenticationManager authenticationManager) {
        super(DEFAULT_ANT_PATH_REQUEST_MATCHER, authenticationManager);
    }

    /**
     * 通过email进行认证
     * @param request
     * @param response
     * @return
     * @throws AuthenticationException
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
        if (this.postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        String email = obtainEmail(request);
        email = (email != null) ? email.trim() : "";
        String password = obtainPassword(request);
        password = (password != null) ? password : "";
        EmailPasswordAuthenticationToken authRequest = EmailPasswordAuthenticationToken.unauthenticated(email, password);
        // Allow subclasses to set the "details" property
        setDetails(request, authRequest);
        // 获取安全管理器，从安全管理器里面获取AuthenticationProvider，来交给AuthenticationProvider 去实现具体认证功能 
        return this.getAuthenticationManager().authenticate(authRequest);
    }

    /**
     * 获取登录密码-验证的条件之一
     * @param request
     * @return
     */
    @Nullable
    protected String obtainPassword(HttpServletRequest request) {
        return request.getParameter(this.passwordParameter);
    }

    /**
     * 获取登录邮箱-验证的条件之一
     * @param request
     * @return
     */
    @Nullable
    protected String obtainEmail(HttpServletRequest request) {
        return request.getParameter(this.emailParameter);
    }

    /**
     * 补充信息
     * @param request
     * @param authRequest
     */
    protected void setDetails(HttpServletRequest request, EmailPasswordAuthenticationToken authRequest) {
        authRequest.setDetails(this.authenticationDetailsSource.buildDetails(request));
    }

    /**
     * Sets the parameter name which will be used to obtain the username from the loginByUserName
     * request.
     * @param emailParameter the parameter name. Defaults to "username".
     */
    public void setEmailParameter(String emailParameter) {
        Assert.hasText(emailParameter, "Username parameter must not be empty or null");
        this.emailParameter = emailParameter;
    }

    /**
     * Sets the parameter name which will be used to obtain the password from the loginByUserName
     * request..
     * @param passwordParameter the parameter name. Defaults to "password".
     */
    public void setPasswordParameter(String passwordParameter) {
        Assert.hasText(passwordParameter, "Password parameter must not be empty or null");
        this.passwordParameter = passwordParameter;
    }

    /**
     * 是否只支持post请求
     * @param postOnly
     */
    public void setPostOnly(boolean postOnly) {
        this.postOnly = postOnly;
    }

    public final String getEmailParameter() {
        return this.emailParameter;
    }

    public final String getPasswordParameter() {
        return this.passwordParameter;
    }
}
```

需要创建EmailPasswordAuthenticationToken ，用于携带登录信息，用于认证,继承EmailPasswordAuthenticationToken

```java
public class EmailPasswordAuthenticationToken extends AbstractAuthenticationToken {

    private final Object principal;

    private Object credentials;

    public EmailPasswordAuthenticationToken(Object principal, Object credentials) {
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false);
    }

    public EmailPasswordAuthenticationToken(Object principal, Object credentials,
                                            Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;

        // 认证通过
        super.setAuthenticated(true);
    }

    public static EmailPasswordAuthenticationToken unauthenticated(Object principal, Object credentials) {
        return new EmailPasswordAuthenticationToken(principal, credentials);
    }
    
    public static EmailPasswordAuthenticationToken authenticated(Object principal, Object credentials,
                                                                 Collection<? extends GrantedAuthority> authorities) {
        return new EmailPasswordAuthenticationToken(principal, credentials, authorities);
    }

    @Override
    public Object getCredentials() {
        return this.credentials;
    }

    @Override
    public Object getPrincipal() {
        return this.principal;
    }

    @Override
    public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
        Assert.isTrue(!isAuthenticated,
                "Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
        super.setAuthenticated(false);
    }

    @Override
    public void eraseCredentials() {
        super.eraseCredentials();
        this.credentials = null;
    }
}

```

EmailAuthenticationProvider 继承 AuthenticationProvider， 在EmailAuthenticationFilter 认证的时候将任务交给EmailAuthenticationProvider

```java
@Data
@Slf4j
public class EmailAuthenticationProvider implements AuthenticationProvider {
    /**
     * 自定义查询员工信息
     */
    private SysUserService sysUserService;

    /**
     * 密码解析
     */
    private PasswordEncoder passwordEncoder;

    /**
     * 具体认证实现
     * @param authentication
     * @return
     * @throws AuthenticationException
     */
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String email = determineEmail(authentication);
        // 根据email 获取员工
        UserDetails user = null;
        try {
            user = this.sysUserService.loadUserByEmail(email);
        } catch (Exception e) {
            log.error("获取用户信息失败", e);
            throw new EmailAuthenticationException(e.getMessage());
        }
        if (null == user) {
            throw new EmailAuthenticationException(ExceptionEnum.MAILBOX_DOES_NOT_EXIST);
        }
        // 校验员工邮箱与密码是否匹配
        try {
            additionalAuthenticationChecks(user, (EmailPasswordAuthenticationToken) authentication);
        } catch (AuthenticationException e) {
            e.printStackTrace();
        }
        Object principalToReturn = user;
        Authentication successAuthentication = createSuccessAuthentication(principalToReturn, authentication, user);

        // 放入容器中
        SecurityContextHolder.getContext().setAuthentication(successAuthentication);
        return successAuthentication;
    }

    private void additionalAuthenticationChecks(UserDetails userDetails, EmailPasswordAuthenticationToken authentication) throws AuthenticationException {
        if (authentication.getCredentials() == null) {
            log.debug("邮箱认证时密码为空");
            throw new BadCredentialsException("email is empty");
        }
        String presentedPassword = authentication.getCredentials().toString();
        if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
            log.debug("邮箱认证时输入密码与用户密码不配置");
            throw new BadCredentialsException("email password  is not match");
        }
    }

    private String determineEmail(Authentication authentication) {
        return (authentication.getPrincipal() == null) ? "NONE_PROVIDED" : (String) authentication.getPrincipal();
    }

    /**
     * 创建认证成功的标志
     */
    protected Authentication createSuccessAuthentication(Object principal, Authentication authentication, UserDetails user) {
        EmailPasswordAuthenticationToken result = EmailPasswordAuthenticationToken.authenticated(principal, authentication.getCredentials(), user.getAuthorities());
        result.setDetails(authentication.getDetails());
        log.debug("Email Authenticated user success");
        return result;
    }

    /**
     * 只处理对应的token的验证
     */
    @Override
    public boolean supports(Class<?> authentication) {
        return (EmailPasswordAuthenticationToken.class.isAssignableFrom(authentication));
    }
}

```

// 调用方式

```java
 @Resource
    @Qualifier("emailAuthenticationManager")
    private AuthenticationManager emailAuthenticationManager;

@Override
    public String loginByEmail(String email, String password, String code, String uuid) {
        Authentication authentication = null;
        try {
            // 该方法会去调用 UserDetailsServiceImpl.loadUserByUsername
            authentication = emailAuthenticationManager.authenticate(new EmailPasswordAuthenticationToken(email, password));
        } catch (Exception e) {
            log.error("登录失败", e);
            if (e instanceof BadCredentialsException) {
                throw new BaseException(20, "用户名或密码错误");
            } else {
                throw new BaseException(30, e.getMessage());
            }
        }
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        return tokenService.createToken(loginUser);
    }
```

// 完整配置

// 注：登录接口是不需要任何认证的(任何过滤器都不应拦截)，也不需要任何鉴权的。直接放行

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Resource
    @Lazy
    private SysUserService sysUserService;
    /**
     * token 认证过滤器
     */
    @Resource
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;
    /**
     * 登出处理
     */
    @Resource
    private JwtLogoutSuccessHandler jwtLogoutSuccessHandler;
    /**
     * 认证失败处理
     */
    @Resource
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    /**
     * 无权限处理
     */
    @Resource
    private JwtAccessDeniedHandler jwtAccessDeniedHandler;

    /**
     * 权限管理器-用于认证用户，并且含有用户的角色
     * @return
     */
    @Bean("authenticationManager")
    public AuthenticationManager authenticationManager() {
        DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();
        daoAuthenticationProvider.setUserDetailsService(sysUserService);
        // 强散列哈希加密实现
        daoAuthenticationProvider.setPasswordEncoder(new BCryptPasswordEncoder());
        return new ProviderManager(daoAuthenticationProvider);
    }

    @Bean("emailAuthenticationManager")
    public AuthenticationManager emailAuthenticationManager() {
        EmailAuthenticationProvider emailAuthenticationProvider = new EmailAuthenticationProvider();
        emailAuthenticationProvider.setSysUserService(sysUserService);
        emailAuthenticationProvider.setPasswordEncoder(new BCryptPasswordEncoder());
        return new ProviderManager(emailAuthenticationProvider);
    }

    /**
     * 用于配置url权限控制
     * @param http
     * @return
     * @throws Exception
     */
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        EmailAuthenticationFilter emailAuthenticationFilter = new EmailAuthenticationFilter();
        http
                // 配置请求认证方式
                .authorizeRequests(auth -> auth
                        // /test/echo 任何用户都可以访问
                        .antMatchers("/test/echo", "/login-username", "/captchaImage", "/login-email").permitAll()
                        // /test/admin 只有ADMIN角色可以访问
                        .antMatchers("/admin/**").hasRole("ADMIN")
                        // /test/normal 只有ROLE_NORMAL角色可以访问
                        .antMatchers("/normal/**").hasRole("CUSTOM_ROLE_NORMAL")
                        // 对于登录login 验证码captchaImage  logout 允许匿名访问
                        .antMatchers("/login-username", "/captchaImage", "/login-email").anonymous()
                        // 测试接口任意访问
                        .antMatchers("/test/**").anonymous()
                        // 任何请求都需要认证
                        .anyRequest().authenticated())
                // 设置form表单 用户密码登录接口,使用默认页面
                .formLogin().loginProcessingUrl("login-username").and()
                // 设置退出页面
                .logout()
                // 退出成功处理接口
                .logoutSuccessHandler(jwtLogoutSuccessHandler).and()
                // 设置http base认证 ,使用默认页面
                .httpBasic(Customizer.withDefaults())
                // 认证失败
                .exceptionHandling()
                // 屏蔽默认的登录页面(前后端分离项目,不能返回页面)
                .authenticationEntryPoint(jwtAuthenticationEntryPoint).accessDeniedHandler(jwtAccessDeniedHandler).and()
                // 基于token，所以不需要session
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
                // 关闭csrf
                .csrf().disable()
                // 在UsernamePasswordAuthenticationFilter, EmailAuthenticationFilter 之前执行token filter
                .addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class)
                .addFilterAfter(emailAuthenticationFilter, JwtAuthenticationTokenFilter.class)
        ;
        return http.build();
    }
}
```



# 鉴权流程

后端鉴权主要以接口权限为准，就是坚持用户的角色是否支持访问这个接口

具体实现是由**FilterSecurityInterceptor** 实现的，也是一个拦截器,具体由invoke 方法实现

```java
// FilterInvocation封装了request,repose
public void invoke(FilterInvocation filterInvocation) throws IOException, ServletException {
		if (isApplied(filterInvocation) && this.observeOncePerRequest) {
			filterInvocation.getChain().doFilter(filterInvocation.getRequest(), filterInvocation.getResponse());
			return;
		}
		if (filterInvocation.getRequest() != null && this.observeOncePerRequest) {
			filterInvocation.getRequest().setAttribute(FILTER_APPLIED, Boolean.TRUE);
		}
    // 前置鉴权
		InterceptorStatusToken token = super.beforeInvocation(filterInvocation);
		try {
			filterInvocation.getChain().doFilter(filterInvocation.getRequest(), filterInvocation.getResponse());
		}
		finally {
      // 最终鉴权
			super.finallyInvocation(token);
		}
   // 后置鉴权
		super.afterInvocation(token, null);
	}
```

调用AbstractSecurityInterceptor#beforeInvocation

```java
protected InterceptorStatusToken beforeInvocation(Object object) {
		
   // this.obtainSecurityMetadataSource()获取配置文件中的定义的接口以及用户
   // getAttributes() 获取当前url对应的权限设置
		Collection<ConfigAttribute> attributes = this.obtainSecurityMetadataSource().getAttributes(object);
		if (CollectionUtils.isEmpty(attributes)) {
			publishEvent(new PublicInvocationEvent(object));
			return null; // no further work post-invocation
		}
    // 如果没有认证的话
		if (SecurityContextHolder.getContext().getAuthentication() == null) {
			credentialsNotFound(this.messages.getMessage("AbstractSecurityInterceptor.authenticationNotFound",
					"An Authentication object was not found in the SecurityContext"), object, attributes);
		}
    // 如果没有认证的话就重新认证
		Authentication authenticated = authenticateIfRequired();
		
		// 尝试鉴权
		attemptAuthorization(object, attributes, authenticated);
		if (this.publishAuthorizationSuccess) {
			publishEvent(new AuthorizedEvent(object, attributes, authenticated));
		}
		// Attempt to run as a different user
		Authentication runAs = this.runAsManager.buildRunAs(authenticated, object, attributes);
		if (runAs != null) {
			SecurityContext origCtx = SecurityContextHolder.getContext();
			SecurityContext newCtx = SecurityContextHolder.createEmptyContext();
			newCtx.setAuthentication(runAs);
			SecurityContextHolder.setContext(newCtx);

			if (this.logger.isDebugEnabled()) {
				this.logger.debug(LogMessage.format("Switched to RunAs authentication %s", runAs));
			}
			// need to revert to token.Authenticated post-invocation
			return new InterceptorStatusToken(origCtx, true, attributes, object);
		}
		this.logger.trace("Did not switch RunAs authentication since RunAsManager returned null");
		// no further work post-invocation
		return new InterceptorStatusToken(SecurityContextHolder.getContext(), false, attributes, object);

	}
```

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202401150050112.png" alt="image-20240115005032442" style="zoom:50%;" />

以上这是配置文件中定义的

````java
private void attemptAuthorization(Object object, Collection<ConfigAttribute> attributes,
			Authentication authenticated) {
		try {
      // this.accessDecisionManager 尝试获取安全管理器，可以自定义，否则的话使用默认的AffirmativeBased
			this.accessDecisionManager.decide(authenticated, object, attributes);
		}
		catch (AccessDeniedException ex) {
			if (this.logger.isTraceEnabled()) {
				this.logger.trace(LogMessage.format("Failed to authorize %s with attributes %s using %s", object,
						attributes, this.accessDecisionManager));
			}
			else if (this.logger.isDebugEnabled()) {
				this.logger.debug(LogMessage.format("Failed to authorize %s with attributes %s", object, attributes));
			}
			publishEvent(new AuthorizationFailureEvent(object, attributes, authenticated, ex));
			throw ex;
		}
	}
````

AbstractAccessDecisionManager 有以下实现：

+ AffirmativeBased： 一票通过，如果多个投票器中有一个人通过，那么就是通过
+ ConsensusBased： 少数服从多数
+ UnanimousBased： 一票否决
+ 可自定义策略：自定义的时候需要将自定义的策略加入的管理器中

```java
// 定义投票器
public AffirmativeBased(List<AccessDecisionVoter<?>> decisionVoters) {
		super(decisionVoters);
	}
	@Override
	@SuppressWarnings({ "rawtypes", "unchecked" })
	public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)
			throws AccessDeniedException {
		int deny = 0;
    // 逐个遍历投票器
		for (AccessDecisionVoter voter : getDecisionVoters()) {
      // 投票器 投票
			int result = voter.vote(authentication, object, configAttributes);
			switch (result) {
			case AccessDecisionVoter.ACCESS_GRANTED:
				return;
			case AccessDecisionVoter.ACCESS_DENIED:
				deny++;
				break;
			default:
				break;
			}
		}
		if (deny > 0) {
			throw new AccessDeniedException(
					this.messages.getMessage("AbstractAccessDecisionManager.accessDenied", "Access is denied"));
		}
		// 检查是否允许都弃权
		checkAllowIfAllAbstainDecisions();
	}
```

![image-20240115005646110](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202401150056170.png)

# 自定义鉴权

自定义投票器

```java
/**
 * 自定义投票类 用于鉴权
 * 处理一些自定义的配置,比如配置在数据库的接口权限
 *@author peilizhi
 *@date 2024/1/14 21:28
 **/
public class CustomDecisionVoter implements AccessDecisionVoter<FilterInvocation> {
    @Override
    public boolean supports(ConfigAttribute attribute) {
        // 角色必须以ROLE_为前缀
        //return (attribute.getAttribute() != null) && attribute.getAttribute().startsWith(BaseConstant.ROLE_PREFIX);
        return true;
    }
    @Override
    public boolean supports(Class<?> clazz) {
        return true;
    }
    /**
     * 鉴权
     * @param authentication 用户
     * @param object 实际处理的
     * @param attributes 需要的权限
     * @return
     */
    @Override
    public int vote(Authentication authentication, FilterInvocation object, Collection<ConfigAttribute> attributes) {
        // 没有通过认证的话,就没有权限
        if (authentication == null) {
            return ACCESS_DENIED;
        }
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();

        // 获取用户实际的权限
        Set<String> permissions = loginUser.getPermissions();
        if (CollectionUtils.isEmpty(permissions)) {
            // 用户没有权限
            return ACCESS_DENIED;
        }
        if (permissions.contains("*:*")) {
            // 高管不校验权限
            return ACCESS_GRANTED;
        }
        // 获取请求的接口
        String requestUrl = object.getHttpRequest().getRequestURI();
        // 首先设置弃权,因为如果不是本投票器能处理的权限时就弃权
        int result = ACCESS_ABSTAIN;
        // 获取用户实际具有的权限
        if (permissions.contains(requestUrl)) {
            result = ACCESS_GRANTED;
        }
        return result;
    }
}
```



配置类

```java
 /**
     * 自定义投票器
     * @return
     */
    @Bean
    public CustomDecisionVoter customDecisionVoter() {
        return new CustomDecisionVoter();
    }
    /**
     * 自定义鉴权管理器
     * @return
     */
    @Bean
    public AccessDecisionManager accessDecisionManager() {
        // 构造一个新的AccessDecisionManager 放入两个投票器
        List<AccessDecisionVoter<?>> decisionVoters = Arrays.asList(new WebExpressionVoter(), customDecisionVoter());
        return new UnanimousBased(decisionVoters);
    }

@Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
http
                // 配置请求认证方式
                .authorizeRequests(auth -> auth
                        // /test/echo 任何用户都可以访问
                        .antMatchers("/test/echo", "/login-username", "/captchaImage", "/login-email").permitAll()
                        // /test/admin 只有ROLE_ADMIN角色可以访问,security 默认配置角色的前缀是ROLE_
                        .antMatchers("/admin/**").hasRole("ADMIN")
                        // /test/normal 只有ROLE_CUSTOM_NORMAL角色可以访问
                        .antMatchers("/normal/**").hasRole("CUSTOM_NORMAL")
                        // 对于登录login 验证码captchaImage  logout 允许匿名访问
                        .antMatchers("/login-username", "/captchaImage", "/login-email").anonymous()
                        // 测试接口任意访问
                        .antMatchers("/test/**").anonymous()
                        // 任何请求都需要认证用户
                        .anyRequest().authenticated()
                        // 自定义鉴权管理器
                        .accessDecisionManager(accessDecisionManager()));
       return http.build();
    }

```



# 注解鉴权一

配置类

@EnableGlobalMethodSecurity(securedEnabled = true) 开启方法上的鉴权

```java
@Configuration
@EnableWebSecurity
// 开启全局方法权限
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SecurityConfig {
}
```

@Secured 指定这个接口需要的角色

```java
/**
     * @Secured 表示想要请求这个方法的时候用户需要具有CUSTOM_ROLE_NORMAL 权限
     * 本周会执行两次鉴权，第一次在Filter中做的,鉴权的内容时对配置文件里面定义的权限
     * 第二次时在执行Controller 方法的时候,在处理@Secured 注解时对角色的鉴权
     */
    @GetMapping("/hello")
    @Secured("ROLE_CUSTOM_NORMAL")
    public GreetingDO greetingDO(@RequestParam(value = "name", defaultValue = "world") String name) {
        // 每次请求atomicInteger 都增加1
        return new GreetingDO(atomicInteger.incrementAndGet(), String.format(TEMPLATE, name));
    }
```

