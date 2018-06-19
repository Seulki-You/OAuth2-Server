# OAuth2-Server
OAuth2
--------
> * 웹, 모바일 및 데스크톱 어플리케이션에서 안전하고 표준화된 방법으로 안전하게 인증할 수 있는 개방형 프로토콜  
> * 3rd party를 위한 범용적인 인증 표준  


OAuth2 인증 절차
-----------------
![authlogic](https://user-images.githubusercontent.com/28287122/41580422-ced6860a-73d5-11e8-9519-9646c93ad6c2.PNG)

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
> Method : POST  
> Headers : { Authorization : <Basic Auth 방식으로 인코딩된 Client ID와 Client Secret>,  
>             Content-Type : "application/x-www-form-urlencoded" }  
> Body : { grant_type : "password",
>          username : <인증 서버에 저장된 사용자 ID>,
>          password : <인증 서버에 저장된 해당 사용자 ID의 비밀번호>,
>          scope : <인증 서버에 지정한 권한으로 여러개를 띄어쓰기 기준으로 입력 가능함>
>        }  

#### HTTP reqeust 응답 결과 
![image](https://user-images.githubusercontent.com/28287122/41580400-bea9d732-73d5-11e8-8da0-6011967c8ab6.png)
