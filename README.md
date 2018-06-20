# 1. OAuth2에 관하여 
1-1 OAuth2
--------
> * 웹, 모바일 및 데스크톱 어플리케이션에서 안전하고 표준화된 방법으로 안전하게 인증할 수 있는 개방형 프로토콜  
> * 3rd party를 위한 범용적인 인증 표준  

1-2 OAuth2 인증 절차
-----------------
<img src="https://user-images.githubusercontent.com/28287122/41580422-ced6860a-73d5-11e8-9519-9646c93ad6c2.PNG" width="70%">

1-3 OAuth2 인증 방식(Grant Type)
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
 

1-4 Resource Owner Password Credentials flow 사용법
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

1-5 JWT token
---------------
현재 프로젝트에서 사용한 access token의 형태이다.  
JWT(Jason Web Token)은 claim 기반의 token으로 token 자체에 모든 정보를 가지고 있어 별도의 token 저장소가 필요없다는 장점이 있다. 
본 프로젝트에서는 access token을 client인 웹 브라우저의 `Local Storage`에 저장해서 관리하고 있다.  

다음은 인증 서버에서 `JSON`의 형태로 응답을 준 것이다.
```json
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mjk0Njg2MzcsInVzZXJfbmFtZSI6InNldWxraSIsImp0aSI6IjI4YWY4Yjk2LWU5NzItNDc5ZC04ZjU1LWNjNGU2YjM1ZmYzNiIsImNsaWVudF9pZCI6ImFwaWdhdGV3YXkiLCJzY29wZSI6WyJyZWFkIiwid3JpdGUiXX0.HEIaKHyZk-VqNR673eNKrWM9bh_ezWU846CjkBAve44",
    "token_type": "bearer",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mzk4MjIyMzcsInVzZXJfbmFtZSI6InNldWxraSIsImp0aSI6IjQyYTAxNjExLTE0YmUtNDM2MC04MmMyLWJmODVkYTA0NTljMSIsImNsaWVudF9pZCI6ImFwaWdhdGV3YXkiLCJzY29wZSI6WyJyZWFkIiwid3JpdGUiXSwiYXRpIjoiMjhhZjhiOTYtZTk3Mi00NzlkLThmNTUtY2M0ZTZiMzVmZjM2In0.AuRTX7iVlifPdt9OF00DfrK4WsTEkgxJmohlES8nq_4",
    "expires_in": 14399,
    "scope": "read write",
    "jti": "28af8b96-e972-479d-8f55-cc4e6b35ff36"
}
```
위에서 `"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mjk0Njg2MzcsInVzZXJfbmFtZSI6InNldWxraSIsImp0aSI6IjI4YWY4Yjk2LWU5NzItNDc5ZC04ZjU1LWNjNGU2YjM1ZmYzNiIsImNsaWVudF9pZCI6ImFwaWdhdGV3YXkiLCJzY29wZSI6WyJyZWFkIiwid3JpdGUiXX0.HEIaKHyZk-VqNR673eNKrWM9bh_ezWU846CjkBAve44"` 부분이 JWT 형태로 발급된 `access token`이다.  

JWT는 `.`를 기준으로 3가지 부분으로 나누어진다.  
![image](https://user-images.githubusercontent.com/28287122/41631098-b63378e0-746d-11e8-8f82-42bbec3d49be.png)
위의 그림에서 볼 수 있듯이 빨간 부분인 `HEADER`와 보라색 부분인 `PAYLOAD`, 파란색 부분의 `VERIFY SIGNATURE` 세 부분으로 나누어진다. 
* *HEADER*  
 JWT token을 어떻게 해석해야 하는지 명시하는 부분이다.  

* *PAYLOAD*(JWT Claim Set)  
 실제 token의 body로 token의 실질적인 정보를 가지고 있는 부분이다.  
 위의 사진에서 decode된 오른쪽의 `PAYLOAD` 부분을 확인하면 token의 만료 시간과 token 발급을 요청한 사용자와 client에 관한 정보와  
 해당 token에 부여된 권한에 대한 정보를 나타낸다.  

* *VERIFY SIGNATURE*  
 JWT token의 JWT Claim Set 부분이 위변조 되었는지 검증하기 위한 부분이다.

# 2. OAuth2 Service 제작
2-1 pom.xml
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
> 1. 위의 `dependency`는 spring boot `1.4.4.RELEASE` version을 기준으로 작성되었다.  
> spring boot version 변경 시 dependency 호환 문제가 발생할 수 있으로 maven 공식 홈페이지를 참고하여 수정한다.
> 2. `MYSQL` dependency version은 설치된 MYSQL version과 일치해야 한다.  
> `MYSQL 8.0` version 이상에서는 시간대 설정 문제가 발생할 수 있으니 별도로 처리해 주어야한다.

2-2 OAuth2Application.java (main class)
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
> * `@EnableAuthorizationServer` 어노테이션을 추가한다.
> * `public void configure(AuthorizationServerEndpointsConfigurer endpoints)`  
>   * `.accessTokenConverter(jwtAccessTokenConverter())` - JWT token converter를 이용하여 access token을 convert 하겠다는 것이다.
>   * `.userDetailsService(userDetailsService);` - userDetailsService를 이용한다는 것으로  
> `private CustomedUserDetailsService userDetailsService;` - spring security의 `userDetailService`를 필요에 맞게 변경한 것- 으로 `userDetailsService`를 이용하겠다는 것이다.  
> `userDetailsService`는 spring security에서 로그인 처리를 위해 제공하는 패키지이다.
> * `public TokenStore tokenStore()` 과 `public JwtAccessTokenConverter jwtAccessTokenConverter()` JWT 관련 설정이다.
>   * `converter.setSigningKey("123");` - JWT의 SingingKey를 "123"으로 고정 약속한 것으로  
> 후에 `key-pool`을 이용하여 생성된 key로 설정해도 된다.
> * ` public FilterRegistrationBean corsFilter()` - cross domain 문제를 해결해주는 부분이다.

#### 2. OAuth2AuthorizationServerConfig
```java
 @Configuration
    protected static class JwtOAuth2AuthorizationServerConfiguration extends OAuth2AuthorizationServerConfiguration {

        private final JwtAccessTokenConverter jwtAccessTokenConverter;
        public JwtOAuth2AuthorizationServerConfiguration(BaseClientDetails details,
                                                         AuthenticationManager authenticationManager,
                                                         ObjectProvider<TokenStore> tokenStoreProvider,
                                                         AuthorizationServerProperties properties,
                                                         JwtAccessTokenConverter jwtAccessTokenConverter
                                                        ,ClientDetailsService clientDetailsService
                                                         ) {
            super(details, authenticationManager, tokenStoreProvider, properties);
            this.jwtAccessTokenConverter = jwtAccessTokenConverter;
        }

        
        
        @Override
        public void configure(AuthorizationServerEndpointsConfigurer endpoints)
                throws Exception {
            super.configure(endpoints);
            endpoints.accessTokenConverter(jwtAccessTokenConverter);
        }
        
       
        @Override 
        public void configure(ClientDetailsServiceConfigurer clients) throws Exception{
        	//clients.withClientDetails(clientDetailsService);
        	clients.inMemory()
        		.withClient("apigateway")
        		.secret("apigateway12")
        		.authorizedGrantTypes("authorization_code", "password", "client_credentials", "implicit", "refresh_token")
        		.authorities("ROLE_MY_CLIENT")
        		.scopes("read", "write")
        		.accessTokenValiditySeconds(60*60*4)
        		.refreshTokenValiditySeconds(60*60*24*120);
        
      
        	
        	
        }
```
> * ` public void configure(ClientDetailsServiceConfigurer clients) `  
> Client에 관한 정보를 저장하는 곳이다/  
> 위의 userDetailsService와 동일하게 DB로 그 정보를 관리해도 된다.  
> 현재 프로젝트에서 client는 웹 브라우저 하나로 별도의 DB로 관리하지 않는다.
> * 다른 부분 - JWT token에 관한 설정 부분이다.

2-3 bootstrap.yml
--------------
```yml
server:
    port: 8081


spring:
    application:
        name: auth-service
    jpa:
        hibernate:
            ddl-auto: update
            show-sql: true
  
    datasource:
        url: jdbc:mysql://192.168.10.178:3306/memberapi2
        tomcat:
            connection-properties: useUnicode=true;characterEncoding=utf-8;serverTimezone=UTC;
        username: sangmin
        password: tkdals12
        driver-class-name: com.mysql.jdbc.Driver

security:
        authorization:
        token-key-access: isAuthenticated()
```
> * server.port 원하는 port로 지정할 수 있다.  
> * `spring.application` , `spring.jpa`, `spring.datasource`는 사용자 정보를 저장하고 있는 MYSQL과 연동해주는 부분이다.  
> `spring.datasource` 부분만 제작한 DB에 맞게 수정해주면 된다.
>


# 3. 사용자(User or Member) 관리 service  
프로젝트 초기에는 사용자 관리 서비스(user service)와 인증 서비스(Auth service)를 분리하여 구상하였지만 두 서비스를 분리하지 않기로 하였다.  
**User Service**는 REST API 형태로 제작하였다.


# 4. References
* [JWT token](https://blog.outsider.ne.kr/1160).
* [Spring Boot OAuth2 Tutorial](https://spring.io/guides/tutorials/spring-boot-oauth2/).
* [Spring Boot OAuth2 example1 - 간단한 OAuth2 서비스](https://brunch.co.kr/@sbcoba/1).
* [Spring Boot OAuth2 example2 - client와 로그인 기능까지](http://www.baeldung.com/rest-api-spring-oauth2-angularjs).
* [Spring Security Login](https://xmfpes.github.io/spring/spring-security/).
* [Spring Security CORS 설정](https://oddpoet.net/blog/2017/04/27/cors-with-spring-security/).
* [Spring boot RESTFul Web Service - 회원 관리 부분](http://www.namooz.com/2016/12/10/spring-boot-restful-web-service-example-get-post-put-delete-patch/).
