Spring security's servlet support is based on servlet filters. There are multiple servlet filters which are called FilterChain. When a client sends a request, the container creates a FilterChain which contains the Filter.

# DelegatingFilterProxy
Spring provides a Filter implementation called DelegatingFilterProxy which acts as a bridge between the Servlet's container lifecycle and the Spring's ApplicationContext.

The servlet container allows registering Filter instances by using its own standards, but it can't be a spring defined bean. That's why we can register DelegatingFilterProxy through the standard servlet container mechanism but delegate all the work to a spring ban that implement the Filter.

The DelegatingFilterProxy looks up bean Filter0 from the ApplicationContext and then invoke bean Filter0. This DelegatingFilterProxy also helps to delay looking up the Filter0 bean, so the container can start first.

# FilterChainProxy

The FilterChainProxy is a special Filter provided by the spring security. It allows delegating many filter instances wrapped in a DelegatingFilterProxy. This is a bean.

# SecurityFilterChain

SecurityFilterChain is used by FilterChainProxy to determine which spring security Filter should be invoked for the current request.

The security filters in a SecurityFilterChain are beans, but they are registered with FilterChainProxy instead of DelegatingFilterProxy.

We can invoke Filter instances based upon the URL alone.

```
SecurityFilterChain 0(/api/*), .... , SecurityFilterChain N(/admin/*)

FilterChainProxy -> SecurityFilterChain(0...N)
```

Here, the SecurityFilterChain 0 will be invoked only when any requests start with '/api' is matched.

# Security Filters
The security filters are inserted into the FilterChainProxy with the SecurityFilterChain api. Common purposes of those filters are:
* authentication
* authorization
* exploit protection

The filters are executed at the correct order as we specify them

An example of Security filters
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.csrf(Customizer.withDefaults())
                .authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated())
                .httpBasic(Customizer.withDefaults())
                .formLogin(Customizer.withDefaults())
                .build();
    }

}
```
Here the order is
* csrf filter
* authentication filter
* authorization filter

# Logging
In application.properties
```
logging.level.org.springframework.security=TRACE
```

# Add a custom filter to the FilterChain
## Using Filter interface
```java
public class TenantFilter1 implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        String tenantId = req.getHeader("X-tenant-id");
        if(!tenantId.isEmpty() && tenantId.equals("11")) {
            chain.doFilter(request, response);
        }
        throw new AccessDeniedException("Not a valid tenant");
    }
}
```

## Using OncePerRequestFilter
```java
public class TenantFilter2 extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String tenantId = request.getHeader("X-tenant-id");
        if(!tenantId.isEmpty() && tenantId.equals("11")) {
            filterChain.doFilter(request, response);
        }
        throw new AccessDeniedException("Not a valid tenant");
    }
}
```

Now, add those in the SecurityFilterChain
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.csrf(Customizer.withDefaults())
                .authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated())
                .addFilterBefore(new TenantFilter1(), AuthorizationFilter.class)
                .httpBasic(Customizer.withDefaults())
                .formLogin(Customizer.withDefaults())
                .build();
    }

}
```
Or use the TenantFilter2 class.

Now make a request without header
```bash
curl http://localhost:8080
```
We will get an exception in the console.

Now make a request with the header
```bash
curl http://localhost:8080 -H "X-tenant-id: 11"
```

Now the console output is:
```bash
2023-08-28T13:44:45.225+06:00 DEBUG 14176 --- [nio-8080-exec-2] o.s.s.w.s.HttpSessionRequestCache        : Saved request http://localhost:8080/error?continue to session
2023-08-28T13:44:45.227+06:00 DEBUG 14176 --- [nio-8080-exec-2] s.w.a.DelegatingAuthenticationEntryPoint : Trying to match using Or [RequestHeaderRequestMatcher [expectedHeaderName=X-Requested-With, expectedHeaderValue=XMLHttpRequest], And [Not [MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@17003497, matchingMediaTypes=[text/html], useEquals=false, ignoredMediaTypes=[]]], MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@17003497, matchingMediaTypes=[application/atom+xml, application/x-www-form-urlencoded, application/json, application/octet-stream, application/xml, multipart/form-data, text/xml], useEquals=false, ignoredMediaTypes=[*/*]]], MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@17003497, matchingMediaTypes=[*/*], useEquals=true, ignoredMediaTypes=[]]]
2023-08-28T13:44:45.228+06:00 DEBUG 14176 --- [nio-8080-exec-2] s.w.a.DelegatingAuthenticationEntryPoint : Match found! Executing org.springframework.security.web.authentication.DelegatingAuthenticationEntryPoint@7ecb9e17
2023-08-28T13:44:45.228+06:00 DEBUG 14176 --- [nio-8080-exec-2] s.w.a.DelegatingAuthenticationEntryPoint : Trying to match using RequestHeaderRequestMatcher [expectedHeaderName=X-Requested-With, expectedHeaderValue=XMLHttpRequest]
2023-08-28T13:44:45.228+06:00 DEBUG 14176 --- [nio-8080-exec-2] s.w.a.DelegatingAuthenticationEntryPoint : No match found. Using default entry point org.springframework.security.web.authentication.www.BasicAuthenticationEntryPoint@7e44322f
```

We can see the request is saved in the session. If we don't want to save this in the session then

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.csrf(Customizer.withDefaults())
                .authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated())
                .httpBasic(Customizer.withDefaults())
                .formLogin(Customizer.withDefaults())
                .addFilterBefore(new TenantFilter1(), AuthorizationFilter.class)
                .requestCache(cache -> cache.requestCache(new NullRequestCache()))
                .build();
    }

}
```

# Configure Form authentication

```
@Bean
SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    return http.csrf(Customizer.withDefaults())
            .authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated())
            .httpBasic(Customizer.withDefaults())
            .formLogin(Customizer.withDefaults())
            .requestCache(cache -> cache.requestCache(new NullRequestCache()))
            .build();
}
```
The flow is:
* First, a user makes an unauthenticated request to the resource (/) for which it is not authorized.
* Spring Security’s AuthorizationFilter indicates that the unauthenticated request is Denied by throwing an AccessDeniedException.
* Since the user is not authenticated, ExceptionTranslationFilter initiates Start Authentication and sends a redirect to the login page with the configured AuthenticationEntryPoint. In most cases, the AuthenticationEntryPoint is an instance of LoginUrlAuthenticationEntryPoint.
* The browser requests the login page to which it was redirected.
* must render the login page.

When the username and password are submitted, the UsernamePasswordAuthenticationFilter authenticates the username and password. The UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter.

In SecurityFilterChain many things happen
*  When the user submits their username and password, the UsernamePasswordAuthenticationFilter creates a UsernamePasswordAuthenticationToken, which is a type of Authentication, by extracting the username and password from the HttpServletRequest instance.
* Next, the UsernamePasswordAuthenticationToken is passed into the AuthenticationManager instance to be authenticated. **The details of what AuthenticationManager looks like depend on how the user information is stored.**

If authentication fails, then Failure.
* The SecurityContextHolder is cleared out.
* RememberMeServices.loginFail is invoked. If remember me is not configured, this is a no-op.
* AuthenticationFailureHandler is invoked.

If authentication is successful, then Success.
* SessionAuthenticationStrategy is notified of a new login.
* The Authentication is set on the SecurityContextHolder.
* RememberMeServices.loginSuccess is invoked. If remember me is not configured, this is a no-op.
* ApplicationEventPublisher publishes an InteractiveAuthenticationSuccessEvent.
* The AuthenticationSuccessHandler is invoked. Typically, this is a SimpleUrlAuthenticationSuccessHandler, which redirects to a request saved by ExceptionTranslationFilter when we redirect to the login page.

## Configure a custom login page
In security config
```
http.csrf(Customizer.withDefaults())
    .authorizeHttpRequests(authorize -> authorize.anyRequest().authenticated())
    .httpBasic(Customizer.withDefaults())
    .formLogin(form -> form.loginPage("/login").permitAll())
    .requestCache(cache -> cache.requestCache(new NullRequestCache()))
    .build();
```
The login.html
```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
<head>
    <title>Please Log In</title>
</head>
<body>
<h1>Please Log In</h1>
<div th:if="${param.error}">
    Invalid username and password.</div>
<div th:if="${param.logout}">
    You have been logged out.</div>
<form th:action="@{/login}" method="post">
    <div>
        <input type="text" name="username" placeholder="Username"/>
    </div>
    <div>
        <input type="password" name="password" placeholder="Password"/>
    </div>
    <input type="submit" value="Log in" />
</form>
</body>
</html>
```
This is a thymeleaf page.

The MvcController
```java
package com.leeonscoding.springsecuritydemo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class MvcController {
    @GetMapping("/login")
    String login() {
        return "login";
    }
}
```

Also, we need a user/password storage, for a simple test, I've used the InMemoryUserDetailsManager
```
@Bean
public UserDetailsService users() {
    UserDetails user = User.builder()
            .username("user")
            .password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
            .roles("USER")
            .build();
    UserDetails admin = User.builder()
            .username("admin")
            .password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
            .roles("USER", "ADMIN")
            .build();
    return new InMemoryUserDetailsManager(user, admin);
}
```
We can test this using the browser also in curl
```bash
curl -X POST http://localhost:8080 -d "username=user&password=password"
```

In the thymeleaf login.html page, those things are configured.
* The form should perform a post to /login.
* The form needs to include a CSRF Token, which is automatically included by Thymeleaf.
* The form should specify the username in a parameter named username.
* The form should specify the password in a parameter named password.
* If the HTTP parameter named error is found, it indicates the user failed to provide a valid username or password.
* If the HTTP parameter named logout is found, it indicates the user has logged out successfully.

# Logout flow
If csrf is disabled like this
```java
@Bean
SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
	return http.csrf(AbstractHttpConfigurer::disable)
			.authorizeHttpRequests(authorize -> authorize
					.anyRequest()
					.authenticated())
			.formLogin(form -> form.loginPage("/login")
					.defaultSuccessUrl("/content", true)
					.permitAll()
			)
			.build();
}
```
Also, in this code, I've redirected to the /content after a successful login.
```java
    @GetMapping("/content")
    public String content() {
        return "content";
    }
```
The content.html looks like this
```html
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>This is a simple content</h1>
    <form action="/logout" method="post">
        <input type="submit" value="submit"/>
    </form>
</body>
</html>
```
Now test with both /GET and /Post /logout URL

/get using manullay hit in the browser. and /post using the 'submit' button.

This works because when you include the spring-boot-starter-security dependency or use the @EnableWebSecurity annotation, Spring Security will add its logout support and by default respond both to GET /logout and POST /logout.

Also, the csrf is disabled here. If enabled the /get will not work and in the /post we have to pass the csrf token.

If we request teh post /logout the LogoutHandler do the followings:
* Invalidate the HTTP session (SecurityContextLogoutHandler)
* Clear the SecurityContextHolderStrategy (SecurityContextLogoutHandler)
* Clear the SecurityContextRepository (SecurityContextLogoutHandler)
* Clean up any RememberMe authentication (TokenRememberMeServices / PersistentTokenRememberMeServices)
* Clear out any saved CSRF token (CsrfLogoutHandler)
* Fire a LogoutSuccessEvent (LogoutSuccessEventPublishingLogoutHandler)
Once completed, then it will exercise its default LogoutSuccessHandler which redirects to /login?logout.

## Custom logout path and redirect URL
```java
    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(authorize -> authorize
                        .anyRequest()
                        .authenticated())
                .formLogin(form -> form.loginPage("/login")
                        .defaultSuccessUrl("/content", true)
                        .permitAll()
                )
                .logout(logout -> logout
                        .logoutUrl("/lg")
                        .logoutSuccessUrl("/login")
                        .permitAll())
                .build();
    }
```

curl http://localhost:8080/ -u "user:password"
   
