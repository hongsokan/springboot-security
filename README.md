# spring security 테스트

## 기본설정
### 1. DB (mysql)
```mysql
create database security;
```
### 2. application.yml 파일 기본설정 추가
```
server:
  port: 8080
  servlet:
    context-path: /
    encoding:
      charset: UTF-8
      enabled: true
      force: true

spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/security?serverTimezone=Asia/Seoul
    username: 
    password: 

  jpa:
    hibernate:
      ddl-auto: create #create update none
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: true
```
### 3. WebMvcConfig 설정
```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        MustacheViewResolver resolver = new MustacheViewResolver();

        resolver.setCharset("UTF-8");
        resolver.setContentType("text/html;charset=UTF-8");
        resolver.setPrefix("classpath:/templates/");
        resolver.setSuffix(".html");

        registry.viewResolver(resolver);
    }
}
```
### 4. SecurityConfig 설정
```java
@Configuration // IoC 빈(bean)을 등록
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder encodePwd() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(auth -> auth.disable());
        http.authorizeRequests()
                .requestMatchers("/", "/login", "/join").permitAll()
//                .requestMatchers("/").authenticated()
                .requestMatchers("/user/**").hasAnyRole("ROLE_USER")
                .requestMatchers("/manager/**").hasAnyRole("ROLE_MANAGER")
                .requestMatchers("/admin/**").hasAnyRole("ROLE_ADMIN")
                .and()
                .formLogin(login -> login
                        .loginPage("/login")    // [A] 커스텀 로그인 페이지 지정
                        .loginProcessingUrl("/loginProc")    // [B] submit 받을 url
                        .usernameParameter("username")    // [C] submit할 아이디
                        .passwordParameter("password")    // [D] submit할 비밀번호
                        .defaultSuccessUrl("/")
                );

        return http.build();
    }

    @Bean
    public RoleHierarchy roleHierarchy() {

        RoleHierarchyImpl hierarchy = new RoleHierarchyImpl();

        hierarchy.setHierarchy("ROLE_USER > ROLE_MANAGER\n" +
                "ROLE_MANAGER > ROLE_ADMIN");

        return hierarchy;
    }

}
```
