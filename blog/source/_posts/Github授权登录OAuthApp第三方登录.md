---
title: Github授权登录OAuthApp第三方登录
date: 2021-01-01 09:13:03
tags: oauth
categories: 分享
---

### Github 授权登录 OAuth App 第三方登录    

项目开发过程中，多个子系统需要集成统一认证平台，统一登录，也就是第三方登录。目前最流行的协议是OAuth2.0。有很多平台都提供了 第三方登录功能比如 QQ， Github等。



首先创建 Oauth App  在[链接](https://github.com/settings/developers)中点击**New OAuth App**  然后填写

<!--more-->

![](https://cdn.jsdelivr.net/gh/axzc/picbed/blog/img/github_oauth_create.jpg)

- Homepage URL： 就是 我们自己网站的主页  
- Authorization callback URL：为回调地址 用来接收后续的 access_token  

**创建成功后 会得到client id和client secret后续会使用，注意Client Secret只会显示一次，要注意保存。**

#### 登录的具体流程  

![](https://cdn.jsdelivr.net/gh/axzc/picbed/blog/img/github_oauth_flow.jpg)

1. 点击登录后，客户端会发送一个get 请求。并且携带参数，其中一个参数就是redirect_url
2. github收到请求会回调redirect_url中的地址，并且返回一个code参数
3. 收到回调请求后，再发送一个POST请求 并且携带code参数  
4. 在github判断code 正确后会返回一个 access_token
5. 收到token后，发送get请求并且携带token
6. github 会根据access_token返回用户信息  

具体流程和参数可以查看[Authorizing OAuth Apps](https://developer.github.com/apps/building-oauth-apps/authorizing-oauth-apps/)

#### 代码实现  

 大致结构为：  

![](https://cdn.jsdelivr.net/gh/axzc/picbed/blog/img/github_oauth_structure.jpg)



拦截器

```java
@Component
public class TokenInterceptor implements HandlerInterceptor  {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        GithubUserDto user = (GithubUserDto) request.getSession().getAttribute("user");
        if (user !=null){
            // 如果 能够获取到user 说明已经登录 放行
            return true;
        }else{
            // 如果没有登录
            response.sendRedirect("https://github.com/login/oauth/authorize?client_id=your_client_id&redirect_uri=http://localhost:8080/callback&scope=user&state=1");
            return false;
        }
    }
}

```

两个实体类  

```java
@Component
@ConfigurationProperties(prefix = "github")
@Data
public class GithubParams {

    private String clientId;
    private String clientSecret;
    private String redirectUri;
    private String tokenUri;
    private String userUri;
    private String authorizeUri;

}

//--------------------------------------
@Component
@Data
public class GithubUserDto {

    private Long id;
    private String name;
    private String bio;
    private String email;

}

```

配置类  

```java
// 拦截器 配置类
@Configuration
public class InterceptorConfig extends WebMvcConfigurationSupport {

    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new TokenInterceptor()).addPathPatterns("/**").excludePathPatterns("/", "/login", "/callback", "/css/**", "/fonts/**", "/js/**");
        super.addInterceptors(registry);
    }
}
// ---------------------------------------
@Configuration
public class RestConfig {

    @Bean
    public RestTemplate createRestTemplate(){
        return new RestTemplateBuilder().build();
    }

}

```

IndexController

```java
@Controller
public class IndexController {

    @GetMapping("/index")
    public String index(){
        return "index";
    }
    
}

```

AuthorizeController

```java
@Controller
public class AuthorizeController {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private GithubParams githubParams;

    @GetMapping("/callback")
    public String callback(@RequestParam("code") String code,
                           @RequestParam("state") String state,
                           HttpSession httpSession){

        String url = githubParams.getTokenUri()+"?client_id="+githubParams.getClientId()+"&client_secret="+githubParams.getClientSecret()+"&code="+code;

        // 发送post 请求 获取 accessToken
        String result = restTemplate.postForObject(url, null, String.class);
        String accessToken = result.split("&")[0].split("=")[1];

        // 发送get 请求 获取用户信息
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.add("accept", "application/json");
        httpHeaders.add("Authorization", "token "+accessToken);
        HttpEntity<String> httpEntity = new HttpEntity<>(httpHeaders);
        ResponseEntity<String> exchange = restTemplate.exchange(githubParams.getUserUri(), HttpMethod.GET, httpEntity, String.class);
        String body = exchange.getBody();
        GithubUserDto githubUserDto = JSON.parseObject(body, GithubUserDto.class);
        httpSession.setAttribute("user", githubUserDto);
        System.out.println(githubUserDto);

        return "index";
    }

}
```

最后 配置文件  

```properties
github.clientId=你的clientId
github.clientSecret=你的clientSecret
github.redirectUri=http://localhost:8080/callback&scope=user&state=1
github.tokenUri=https://github.com/login/oauth/access_token
github.userUri=https://api.github.com/user
```

