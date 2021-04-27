# Spring Security

1. 启动日志中，可见DefaultSecurityFilterChain，这里面则是SecurityFilterChain
   - WebAsyncManagerIntegrationFilter
   - SecurityContextPersistenceFilter（内容持久化过滤器，登陆后，用sessionid来识别）
   - HeaderWriterFilter
   - UsernamePasswordAuthenticationFilter（用户密码验证过滤器）
   - JwtAuthorizationTokenFilter（JWT Token过滤器）
   - RequestCacheAwareFilter
   - SecurityContextHolderAwareRequestFilter
   - SessionManagementFilter
   - ExceptionTranslationFilter
   - FilterSecurityInterceptor
   
2. 对自动配置进行干预，可以继承 `WebSecurityConfigurerAdapter` 并实现它的 3 个 `configure` 方法

   - `void configure(AuthenticationManagerBuilder auth)`：配置认证管理器，开发者需要实现 `UserDetailsService` 接口，编写自定义认证逻辑，并将接口实现注册到 `Spring` 容器，在此方法中指定认证逻辑实现。
   - `void configure(HttpSecurity http)`：配置过滤器管理器，开发者在此方法中对默认的 `HttpSecurity` 进行修改
   - `void configure(WebSecurity web)`：请求忽略配置，开发者在此可以配置不需要进行安全认证的请求：
   
3. Spring Security Jwt实例（部分重要代码）

   - 配置文件

     ```java
     //配置文件注解
     @Configuration
     //开启Security注解
     @EnableWebSecurity
     //开放 @PreAuthorize 和 @PostAuthorize 注解，详情如下链接
     @EnableGlobalMethodSecurity(prePostEnabled=true)
     //继承 WebSecurityConfigurerAdapter 可干预 Security 配置
     public class SecurityConfig extends WebSecurityConfigurerAdapter {
         @Autowired
         private UserMapper userMapper;
         //自定义未登录结果
         @Autowired
         private RestfulAccessDeniedHandler restfulAccessDeniedHandler;
         //自定义未授权返回结果
         @Autowired
         private RestAuthenticationEntryPoint restAuthenticationEntryPoint;
     
         //配置过滤器管理器
         @Override
         protected void configure(HttpSecurity httpSecurity) throws Exception {
             httpSecurity.csrf()// 由于使用的是JWT，我们这里不需要csrf
                     .disable()
                     .sessionManagement()// 基于token，所以不需要session
                     .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                     .and()
                     .authorizeRequests()
                     .antMatchers(HttpMethod.GET, // 允许对于网站静态资源的无授权访问
                             "/",
                             "/*.html",
                             "/favicon.ico",
                             "/**/*.html",
                             "/**/*.css",
                             "/**/*.js",
                             "/swagger-resources/**",
                             "/v2/api-docs/**"
                     )
                     .permitAll()
                     .antMatchers("/login", "/register")// 对登录注册要允许匿名访问
                     .permitAll()
                     .antMatchers(HttpMethod.OPTIONS)//跨域请求会先进行一次options请求
                     .permitAll()
     //                .antMatchers("/**")//测试时全部运行访问
     //                .permitAll()
                     .anyRequest()// 除上面外的所有请求全部需要鉴权认证
                     .authenticated();
             // 禁用缓存
             httpSecurity.headers().cacheControl();
             // 在 UsernamePasswordAuthenticationFilter 拦截器前，添加JWT filter
             httpSecurity.addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
             //添加自定义未授权和未登录结果返回
             httpSecurity.exceptionHandling()
                     .accessDeniedHandler(restfulAccessDeniedHandler)//未登录
                     .authenticationEntryPoint(restAuthenticationEntryPoint);//未授权
         }
     
         //配置认证管理器
         @Override
         protected void configure(AuthenticationManagerBuilder auth) throws Exception {
             auth.userDetailsService(userDetailsService())
                     .passwordEncoder(passwordEncoder());
         }
     
         //密码加密方式
         @Bean
         public PasswordEncoder passwordEncoder() {
             return new BCryptPasswordEncoder();
         }
     
         //通过用户唯一值 username 把权限 permissionList 加入到 UserDetails
         @Bean
         public UserDetailsService userDetailsService() {
             //获取登录用户信息
             return username -> {
                 User user = userMapper.getOneByUserName(username);
                 if (user != null) {
                     List<Permission> permissionList = userMapper.getPermissionList(user.getId());
                     return new AdminUserDetails(user,permissionList);
                 }
                 throw new UsernameNotFoundException("用户名或密码错误");
             };
         }
     
         //新增的拦截器
         @Bean
         public JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter(){
             return new JwtAuthenticationTokenFilter();
         }
     
         //用于自定义筛选器
         @Bean
         @Override
         public AuthenticationManager authenticationManagerBean() throws Exception { 
             return super.authenticationManagerBean();
         }
     }
     ```
   
   
   
   - Jwt验证过滤器
   
     ```java
     //继承 OncePerRequestFilter，确保在一次请求只通过一次filter，好像是版本问题
     public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
         //Security 内置的UserDetailsService类，可获取userDetails
         @Autowired
         private UserDetailsService userDetailsService;
         //Jwt工具类
         @Autowired
         private JwtTokenUtil jwtTokenUtil;
         private String tokenHeader = "Authorization";
         private String tokenHead = "Bearer ";
     
         @Override
         protected void doFilterInternal(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, FilterChain filterChain) throws ServletException, IOException {
             // 获取头
             String authHeader = httpServletRequest.getHeader(this.tokenHeader);
             if (authHeader != null && authHeader.startsWith(this.tokenHead)){
                 //把前面一段截掉
                 String authToken = authHeader.substring(this.tokenHead.length());
                 //通过token获取用户名
                 String username = jwtTokenUtil.getUserNameFromToken(authToken);
                 //如果用户不为空，且SecurityContextHolder为空，则验证是否能登录
                 if (username != null && SecurityContextHolder.getContext().getAuthentication() == null){
                     //获取userDetails，里边包含了用户详情和所有权限
                     UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);
                     //验证token是否还有效
                     if (jwtTokenUtil.validateToken(authToken, userDetails)){
                         //由 UsernamePasswordAuthenticationToken 生成 authentication
                         UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails,null, userDetails.getAuthorities());
                         //把 authentication 存入到 SecurityContentHolder 中，存在则会通过验证
                         SecurityContextHolder.getContext().setAuthentication(authentication);
                     }
                 }
     
             }
             //通道继续
             filterChain.doFilter(httpServletRequest,httpServletResponse);
         }
     }
     ```
   
     

------

[Spring Security原理介绍、源码解析——认证过程](https://juejin.cn/post/6844903949296730125)

[Spring Security 源码分析（一）：过滤器链](https://wch853.github.io/posts/security/SpringSecurity%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9A%E8%BF%87%E6%BB%A4%E5%99%A8%E9%93%BE.html#websecurityconfigureradapter-%E9%85%8D%E7%BD%AE)

[@EnableGlobalMethodSecurity注解详解](https://blog.csdn.net/chihaihai/article/details/104678864)