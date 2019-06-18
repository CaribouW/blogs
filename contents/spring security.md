## spring security

### WEB 层过滤

#### 登录界面跳转

```java
@Configuration
@EnableWebSecurity
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter{
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login") // 1 设置登录认证界面
                .permitAll();        // 2 保证该接口不设置任何鉴权操作
    }		
}
```

#### 过滤网链配置

```java
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()                                             
            .antMatchers("/resources/**", "/signup","/about").permitAll() 
           .antMatchers("/admin/**").hasRole("ADMIN")                     
            .antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')")                      .anyRequest().authenticated()
            .and()
        // ...
        .formLogin();
}
```

#### 登出配置



### OAuth 2.0 Client

#### 加密操作

spring security内置了不少的密码加密代理类，使用方式比较简单

如下所示

###### Pbkdf2PasswordEncoder

```java
// Create an encoder with all the defaults
Pbkdf2PasswordEncoder encoder = new Pbkdf2PasswordEncoder();
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

###### BCryptPasswordEncoder

```java
// Create an encoder with strength 16
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(16);
String result = encoder.encode("myPassword");
assertTrue(encoder.matches("myPassword", result));
```

...

他们都继承于同一个加密父类 `PasswordEncoder`

