---
title: spring security 초기 설정
date: 2023-05-28 09:00:00 +0900
categories: [개인프로젝트]
tags: [GettingThingsDone,spring security]     # TAG names should always be lowercase
published: true
---

의존성 추가 후 평소와 같이 포스트맨을 통해 테스트를 해보려는데 토큰 생성?외에 뭔가..반응이 없다.  
localhost로 띄워보니 왠 로그인 페이지가 떴다.  
![login](/assets/img/security login.png)

요청을 해도 예상한대로 응답이 오질 않고, 아예 먹통인 상황...  
```plaintext
서버 기동시 스프링 시큐리티의 초기화 작업 및 보안 설정이 이뤄진다.  
별도의 설정이나 구현을 하지 않아도 기본적인 웹보안 기능이 현재 시스템에 연동된다. 
모든 요청은 인증되어야 자원에 접근이 가능하고  
인증 방식은 폼 로그인 방식과 httpBacis 로그인 방식을 제공한다.

기본 계정이 제공되지만 매번 생성되는 랜덤문자열과 계정으로 로그인 하기는 까다롭기에  
application.properties에 기본 name과 password를 설정한다.  
```
이런 설명글을 보고 name과 password를 설정하고 로그인을 해보았고,  
응답이 오는 것을 일단 확인 하였다.
![payload](/assets/img/security payload.png)

현재 시점에서는 계정,권한을 추가하거나 db연동을 위해선 세부적 보안 기능이 필요하다고한다.    

앞서 설정한 username과 password로 로그인을 성공하였다면  


![basic](/assets/img/security basic.png)
위 그림에서  
서버에서 SessionID생성후 인증결과를 담은 인증 토큰을 생성하고 저장한 단계까지 온것이다.   


자세한 설정을 위해 `configSecuriy` 클래스를 작성하려 하는데,,, 
참고한 블로그에서 본  `WebSecurityConfigurerAdapter` 을 상속받아 쓰는 방식은 현재 프로젝트 버전에서는 더 이상 지원되지 않는 것을 확인하였다. 


`SecurityFilterChain` 라는 메서드를 사용해봤는데, 
```java
@Configuration
@EnableWebSecurity 
public class SecurityConfig {

    private final JwtTokenProvider jwtTokenProvider;

    public SecurityConfig(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .httpBasic().disable()
                .csrf().disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeHttpRequests()
                .requestMatchers("/login").permitAll()
                .requestMatchers("/getSession").hasRole("USER")
                .anyRequest().authenticated() 
                .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```

위와 같이 빈등록을 해야 동작한다.  
위 코드에 대한 설명은 아래와 같다. 

```java
@Configuration
@EnableWebSecurity //웹보안 활성화를위한 annotation
public class SecurityConfig {

    private final JwtTokenProvider jwtTokenProvider;

    public SecurityConfig(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }
		@Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .httpBasic().disable() // restAPI이므로 비인증시 로그인폼화면으로 리다이렉트 되는 기본설정 사용안함. 
                .csrf().disable() //restAPI이므로 csrf보안 disable처리 
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) //jwt token으로 인증하므로 세션은 필요없으므로 생성안함
                .and()
                .authorizeHttpRequests()// 다음 요청에 대한 보안검사(사용권한 체크) 시작
                .requestMatchers(HttpMethod.GET,"/member").permitAll() // 해당요청은 검사 안함
                .requestMatchers("/login").permitAll() // 
								.anyRequest().hasRole("USER") // 그 외 나머지 요청은 모두 인증된 회원만 접근 가능하다. 
                //.anyRequest().authenticated() //어떤 요청에도 보안검사를 한다.
                .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);// jwt toekn필터를 id/password인증 필터 전에 넣는다. 
        return http.build();
    }

    public PasswordEncoder passwordEncoder() {
		//JWT를 사용하기 위해서는 기본적으로 password encoder가 필요한데, 
		//여기서는 Bycrypt encoder를 사용했다.
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```