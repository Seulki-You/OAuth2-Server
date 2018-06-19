# OAuth2-Server
OAuth2
--------
> * 웹, 모바일 및 데스크톱 어플리케이션에서 안전하고 표준화된 방법으로 안전하게 인증할 수 있는 개방형 프로토콜  
> * 3rd party를 위한 범용적인 인증 표준  


OAuth2 인증 절차
-----------------
<img src="https://user-images.githubusercontent.com/28287122/41580422-ced6860a-73d5-11e8-9519-9646c93ad6c2.PNG" width="70%">

OAuth2 인증 방식(Grant Type)
----------------------------
### 1. Authorization Code flow
 * 웹 서버에서 인증 받을 때 가장 많이 이용되는 방식 
 * 클라이언트가 access token을 받기 위해 authorization code를 이용 

### 2. Implicit Grant flow
 * 클라이언트 사이드에서 OAuth2 인증하는 방식  
 * 추가적인 인증 코드 교환 단계 없이 엑세스 토큰이 즉시 반환되는 단순화 된 흐름
 * 보안사의 문제로 일반적으로 권장 되지 않음
 * refresh token 발급 안함  

### 3. Resource Owner Password Credentials flow
 * 사용자(resource owner)의 아이디와 비밀번호로 access token을 발급함

### 4. Client Credentials flow
 * 클라이언트 정보(Client ID와 Client Secret)로 access tokendmf 발급함
 * 사용자의 정보를 이용보다는 클라이언트 자체 정보를 이용할 때 이용 

### 5. Refresh Token
 * 기존에 발급된 refresh token이 존재할때 access token을 재발급 받을 때 이용
 

Resource Owner Password Credentials flow 사용법
-----------------------------------------------
#### HTTP reqeust
* Method : POST  

* Headers : 
> { Authorization : <Basic Auth 방식으로 인코딩된 Client ID와 Client Secret>,  
> Content-Type : "application/x-www-form-urlencoded" }  

* Body :  
> { grant_type : "password",  
> username : <인증 서버에 저장된 사용자 ID>,  
> password : <인증 서버에 저장된 해당 사용자 ID의 비밀번호>,  
> scope : <인증 서버에 지정한 권한으로 여러개를 띄어쓰기 기준으로 입력 가능함>  
> }  

#### HTTP reqeust 응답 결과 
![image](https://user-images.githubusercontent.com/28287122/41580400-bea9d732-73d5-11e8-8da0-6011967c8ab6.png)

pom.xml
-------
```xml
<dependency>
 <groupId>org.springframework.security.oauth</groupId>
 <artifactId>spring-security-oauth2</artifactId>
</dependency>
		
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-security</artifactId>
</dependency>
		
<dependency>
 <groupId>org.springframework.security</groupId>
 <artifactId>spring-security-core</artifactId>
</dependency>
		
<dependency>
 <groupId>org.springframework.security</groupId>
 <artifactId>spring-security-web</artifactId>
</dependency>
		
<dependency>
 <groupId>org.springframework.security</groupId>
 <artifactId>spring-security-config</artifactId>
</dependency>
		
<dependency>
 <groupId>org.springframework.security</groupId>
 <artifactId>spring-security-jwt</artifactId>
</dependency>
  
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>		
		
<dependency>
 <groupId>mysql</groupId>
 <artifactId>mysql-connector-java</artifactId>
 <version>8.0.11</version>
</dependency>

```

> **주의**
> 1. 위의 `dependency`는 spring boot `1.4.4.RELEASE` version을 기준으로 함 -> spring boot version 변경 시 호환 문제 발생
> 2. `MYSQL` dependency version은 설치된 MYSQL version과 일치해야 함. 

OAuth2Application.java (main class)
------------------------------------
#### 1. OAuth2AuthorizationServerConfig
```java
    @EnableAuthorizationServer
    @Configuration
    protected static class OAuth2AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    	
    	@Autowired
    	@Qualifier("authenticationManagerBean")
    	private AuthenticationManager authenticationManager;
    	
    	@Autowired
    	private CustomedUserDetailsService userDetailsService;
        
    	@Override
    	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    		endpoints.tokenStore(tokenStore())
    			.accessTokenConverter(jwtAccessTokenConverter())
    			.authenticationManager(authenticationManager)
    			.userDetailsService(userDetailsService);
    	}
    	
    	@Override
    	public void configure(AuthorizationServerSecurityConfigurer oauthServer) throws Exception{
    		oauthServer
    			.tokenKeyAccess("permitAll()")
    			.checkTokenAccess("isAuthenticated()");
    	}
    	
        @Bean
        public TokenStore tokenStore() {
            return new JwtTokenStore(jwtAccessTokenConverter());
        }

        @Bean
        public JwtAccessTokenConverter jwtAccessTokenConverter() {
        	final JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        	converter.setSigningKey("123");
            return converter;
        }
        
        @Bean 
        public FilterRegistrationBean corsFilter() {
        	UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
            CorsConfiguration config = new CorsConfiguration();
            config.setAllowCredentials(true);
            config.addAllowedOrigin("*");
            config.addAllowedHeader("*");
            config.addAllowedMethod("*");
            source.registerCorsConfiguration("/**", config);
            FilterRegistrationBean bean = new FilterRegistrationBean(new CorsFilter(source));
            bean.setOrder(Ordered.HIGHEST_PRECEDENCE);
            return bean;
        }
        
    }

```

> * OAuth2 설정관련 부분
> * `@EnableAuthorizationServer` 어노테이션 추가
> * `public void configure(AuthorizationServerEndpointsConfigurer endpoints)`  
>   * `.accessTokenConverter(jwtAccessTokenConverter())` - JWT token을 access token으로 사용
>   * `.userDetailsService(userDetailsService);` - userDetailsService를 이용, spring security에서 제공하는 로그인 모듈 이용
>   * `private CustomedUserDetailsService userDetailsService;` - spring security의 `userDetailService`를 필요에 맞게 변경한 것을 
> * `public TokenStore tokenStore()` 과 `public JwtAccessTokenConverter jwtAccessTokenConverter()` JWT 설정관련
>   * `converter.setSigningKey("123");` - JWT의 SingingKey를 "123"으로 고정 약속함.
> * ` public FilterRegistrationBean corsFilter()` - cross domain 문제처리 filter


