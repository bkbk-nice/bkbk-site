---
slug: jwt
title: springsecurity+jwt
authors: [bkbk]
tags: [springsecurity,jwt]
---
  
:::caution 目录
**springsecurity** 
**jwt**        
**后端接口权限访问**  
::: -->
<!--truncate-->
 
### jwt、springsecurity配置

``` xml title="依赖"
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency> 
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
``` 

```java title="jwt工具类"  
public final class JwtUtil {
   
   //自定义secretKey
    private final static String secretKey = "xxxxxxxxxxxx";
    
    private final static Duration expiration = Duration.ofHours(2);

     
    public static String generate(String userName) {
        // 过期时间
        Date expiryDate = new Date(System.currentTimeMillis() + expiration.toMillis());

        return Jwts.builder()
                .setSubject(userName) // 将userName放进JWT
                .setIssuedAt(new Date()) // 设置JWT签发时间
                .setExpiration(expiryDate)  // 设置过期时间
                .signWith(SignatureAlgorithm.HS512, secretKey) // 设置加密算法和秘钥
                .compact();
    } 
    public static Claims parse(String token) { 

        if (StringUtils.isEmpty(token)) {
            return null;
        }
 
        Claims claims = null; 
        try {
            claims = Jwts.parser()
                    .setSigningKey(secretKey) // 设置秘钥
                    .parseClaimsJws(token)
                    .getBody();
        } catch (JwtException e) { 
            System.err.println("解析失败！");
        }
        return claims;
    }
}
```

```java title="SecuriConfig"
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private SecureServiceImpl secureService;

    @Autowired
    private LoginFilter loginFilter;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 关闭csrf和frameOptions，如果不关闭会影响前端请求接口（这里不展开细讲了，感兴趣的自行了解）
        http.csrf().disable();
        http.headers().frameOptions().disable();
        // 开启跨域以便前端调用接口
        http.cors();
        http.addFilterBefore(loginFilter, UsernamePasswordAuthenticationFilter.class);

        //禁用session
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);


        // 这是配置的关键，决定哪些接口开启防护，哪些接口绕过防护
        http.authorizeRequests()
                // 注意这里，是允许前端跨域联调的一个必要配置
                .requestMatchers(CorsUtils::isPreFlightRequest).permitAll()
                // 指定某些接口不需要通过验证即可访问。登陆、注册接口肯定是不需要认证的
                .antMatchers("/secure/login", "/secure/register").permitAll()
                // 这里意思是其它所有接口需要认证才能访问
                .anyRequest().authenticated()
                // 指定认证错误处理器
                 .and().exceptionHandling().authenticationEntryPoint(new MyEntryPoint());
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 指定UserDetailService和加密器
        auth.userDetailsService( secureService).passwordEncoder(passwordEncoder());
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }


    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

```java title="UserDetail" 
public class UserDetail extends User { 
    /**
     * 用户实体对象，要调取用户信息时直接获取这个实体对象
     */
    private Client client; 
    
    public UserDetail(Client client, Collection<? extends GrantedAuthority> authorities) {
        // 必须调用父类的构造方法，以初始化用户名、密码、权限
        super(client.getName(), client.getPassword(), authorities);
        this.client = client;
    }
} 
```
 
```java title="自定义失败处理" 
//在SecuriConfig中导入
 public class MyEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) throws IOException {
        response.setContentType("application/json;charset=utf-8");
        PrintWriter out = null;
        try {
            out = response.getWriter();
        } catch (IOException ex) {
            throw new RuntimeException(ex);
        }
        // 直接提示前端认证错误
        out.write("认证错误");
        out.flush();
        out.close();
    }
}
```
 
### controller service 使用

```java title="流程" 
 //controller
@RestController
@RequestMapping("secure")
public class SecureController { 
    @Autowired
    private SecureService secureService; 
    @PostMapping("/login")
    public ResultVo login(String name,String password){  
        return secureService.login(name,password);
    }  
    @Autowired
    private PasswordEncoder passwordEncoder; 
    @PostMapping("/register")
    public ResultVo register(String name ,String password) {
        Client client = new Client();
        // 调用加密器将前端传递过来的密码进行加密
        client.setName(name);
        client.setPassword((passwordEncoder.encode(password)));
        // 将用户实体对象添加到数据库 
        return secureService.register(client);
    } 
}
//service 必须  implement UserDetailsService ，重写loadUserByUsername
@Service
public class SecureServiceImpl implements SecureService  , UserDetailsService { 

    @Autowired
    private SecureMapper secureMapper; 
    @Autowired
    private PasswordEncoder passwordEncoder; 

    @Override
    public ResultVo login(String name, String password) {
        //登录校验 
        QueryWrapper<Client> queryWrapper = new QueryWrapper<Client>();
        queryWrapper.eq("name",name); 
        Client client = secureMapper.selectOne(queryWrapper); 
        if (client == null  ||  !passwordEncoder.matches(password, client.getPassword()) ){
            return ResultVo.fail("账号密码错误");
        } 
        //jwt生成token 
        ClientVo csvo = new ClientVo(); 
        csvo.setId(client.getId());
        csvo.setName(client.getName());
        csvo.setToken(JwtUtil.generate(client.getId().toString())); 
        return ResultVo.success(csvo);
    }

    @Override
    public ResultVo register(Client client) {
        // 从数据库中查询出用户实体对象
        QueryWrapper<Client> queryWrapper = new QueryWrapper<Client>();
        queryWrapper.eq("name",client.getName());

        Client client1 = secureMapper.selectOne(queryWrapper); 
        if (client1 != null) {
            return ResultVo.fail("已存在此客户");
        } 
        Date now = new Date(); 
        client.setCreateTime(now);
        client.setUpdateTime(now); 
        return ResultVo.success(secureMapper.insert(client));
    }

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {

        System.out.println("exe loaduserbyusername");

        // 从数据库中查询出用户实体对象
        QueryWrapper<Client> queryWrapper = new QueryWrapper<Client>();
        queryWrapper.eq("id",Integer.parseInt(s)); 
        Client client = secureMapper.selectOne(queryWrapper);
        // 若没查询到一定要抛出该异常，这样才能被Spring Security的错误处理器处理
        if (client == null) {
            throw new UsernameNotFoundException("没有找到该用户");
        }  
        //增加一个权限
        Set<GrantedAuthority> dbAuthsSet = new HashSet<>(); 
        dbAuthsSet.add(new SimpleGrantedAuthority("ROLE_client")); 
        return new UserDetail(client,dbAuthsSet);
    } 
}

```

```java title="接口权限注解配置"
@RestController
@RequestMapping("/client")
public class ClientController {  
    @Autowired
    private ClientService clientService; 

    @Secured("ROLE_client")
    @PostMapping("/uploadAvatar")
    public ResultVo uploadAvatar(MultipartFile file ,@RequestHeader("Authorization") String token)  {   
        Claims claims = JwtUtil.parse(token);
        if (claims != null) { 
            String clientid = claims.getSubject(); 
            return   clientService.uploadAvatar(file,Integer.parseInt(clientid)); 
        }else{
            return ResultVo.fail("token错误");
        } 
    }

    @Secured("ROLE_client")
    @GetMapping("/avatar")
    public ResultVo getAvatar(@RequestHeader("Authorization") String token)  {  
        Claims claims = JwtUtil.parse(token);
        if (claims != null) { 
            String clientid = claims.getSubject(); 
            return   clientService.getAvatar(Integer.parseInt(clientid)); 
        }else{
            return ResultVo.fail("token错误");
        } 
    }  
}

```


:::tip
**  @RequestHeader 注解取出header中token **  
** public ResultVo uploadAvatar(MultipartFile file ,@RequestHeader("Authorization") String token) **
:::


### http 请求过滤

``` java title="LoginFilter"
@Component
public class LoginFilter extends OncePerRequestFilter {
 
    @Autowired
    private SecureServiceImpl secureService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        // 从请求头中获取token字符串并解析 
        Claims claims = JwtUtil.parse(request.getHeader("Authorization"));
        if (claims != null) {
            // 从jwt中提取出之前存储好的用户名
            String clientid = claims.getSubject();
            // 查询出用户对象
            UserDetails user = secureService.loadUserByUsername(clientid);
            //user 中携带角色信息（ROLE）
            Authentication authentication = new UsernamePasswordAuthenticationToken(user,
                    user.getPassword(),
                    user.getAuthorities());
            // 将认证对象放到上下文中
            SecurityContextHolder.getContext().setAuthentication(authentication); 
        }
        chain.doFilter(request, response); 
    }
}

```

### 前端接收

```js title=" http.js  axios " 
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios' 

class HttpRequest { 
    private readonly baseUrl: string; 
    constructor() { 
        this.baseUrl = 'http://localhost:9000' 
    } 
    getInsideConfig() { 
        const config = { 
            baseURL: this.baseUrl,// 所有的请求地址前缀部分(没有后端请求不用写)   
            timeout: 80000, // 请求超时时间(毫秒) 
            withCredentials: true,// 异步请求携带cookie  
           headers: { 
            // 设置后端需要的传参类型 
            'Content-Type': 'application/json',
            // 'token': x-auth-token',//一开始就要token 
            // 'X-Requested-With': 'XMLHttpRequest', 
             }, 
        } 
        return config 
    } 
    // 请求拦截 
    interceptors(instance: AxiosInstance, url: string | number | undefined) { 
        instance.interceptors.request.use(config => { 
            // 添加全局的loading.. 
            // 请求头携带token 
            return config 
        }, (error: any) => { 
            return Promise.reject(error) 
        }) 
        //响应拦截  
        instance.interceptors.response.use(res => { 
            //返回数据 
            const { data } = res 
           // console.log('返回数据处理', res) 
            return data 
        }, (error: any) => { 
            console.log('error==>', error)  
            return Promise.reject(error) 
        }) 
    } 
    request(options: AxiosRequestConfig) { 
        const instance = axios.create() 
        options = Object.assign(this.getInsideConfig(), options) 
        this.interceptors(instance, options.url) 
        return instance(options) 
    } 
} 
const http = new HttpRequest() 
export default http
```

```js title="api.tx    post get"
import http from '../utils/http'

export function getCsclientlistPageByDynamics(params: any) {

        return http.request({

        url: '/cs-service/client/listPageByDynamics',

        method: 'get',

        params,

        headers: {
            Authorization: localStorage.getItem("token")
        } 
    }) 
}

export function postCsclientaddClient(params: any) { 

    return http.request({

        url: '/cs-service/client/addClient',

        method: 'post',

        data:params ,
            
        headers: {
            Authorization: localStorage.getItem("token")
        }

    }) 
}
```


```js title="调用"

import { editClient, updatePassword, getClient } from "../../api/api"


editClient(clientform).then((res: any) => {
    console.log(res);
    if (res.status == 0) {
        ElMessageBox.alert(res.message, '提示', {
        confirmButtonText: 'OK',
        })
        load()
    } else {
        ElMessageBox.alert(res.message, '提示', {
        confirmButtonText: 'OK',
        })
    }
    }).catch(() => {
         ElMessageBox.alert("服务器异常", '提示', { confirmButtonText: 'OK', })
});

```