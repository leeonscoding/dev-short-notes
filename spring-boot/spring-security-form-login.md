# Integrate Angular in a Spring Boot Project

## Initialize the project
* Go to the start.spring.io [page](https://start.spring.io/ "spring boot project generator")
* Select
    * **Build Tool**: Gradle - Groovy
    * **Language**: Java
    * **Spring Boot Version**: 3.1.0
    * **Packaging**: Jar
    * **Java Version**: 17
* Fill the metadata part as your own
* Select Dependecies
    * **Spring WEB**: Contains Spring MVC and embedded Tomcat container
* Click the **GENERATE** Button
* Copy the compressed file to a desired location
* Unzip it

## Spring Security configuration
* Create a new class. I've named the class MySecurityConfig. As this is a configuration class, we have to annotate the @Configuration annotation. Also, we need the @EnableWebSecurity annotation. So, we will get the Spring security and its Spring MVC integration support.
* We have to define several methods in that class like this.
    ```java
    @Configuration
    @EnableWebSecurity
    public class MySecurityConfig {

        @Bean
        public InMemoryUserDetailsManager users() {

        }

        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        }

        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }
    }
    ```
    Let's discuss one by one
* The userDetailsManager is the Authentication provider. It provides user details like username, password, role etc. Here I'm using the In-memory authentication provider.
    ```java
    @Bean
    public InMemoryUserDetailsManager users() {
        UserDetails user1 = User.builder()
                                .username("admin")
                                .password(passwordEncoder().encode("admin"))
                                .roles("ADMIN")
                                .build();

        UserDetails user2 = User.builder()
                                .username("test1")
                                .password(passwordEncoder().encode("1"))
                                .roles("USER")
                                .build();

        return new InMemoryUserDetailsManager(user1, user2);
    }
    ```
* The filterChain contains the rules for authorization by url, define urls for loging in and logging out etc.
    ```java
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(Customizer.withDefaults())
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/secure/**").hasAnyRole("USER", "ADMIN")
                        .requestMatchers("/admin/**").hasAnyRole("ADMIN")
                        .anyRequest().authenticated()
                )
                .formLogin(form -> form.loginPage("/login")
                            .defaultSuccessUrl("/secure/home", true)
                            .failureUrl("/error")
                            .permitAll()
                )
                .logout(logout -> logout
                            .logoutSuccessUrl("/logout")
                            .deleteCookies("JSESSIONID")
                            .permitAll()
                )
                .build();
    }
    ```
    * We are authenticating any requests starts with '/secure' with the user who has either USER or ADMIN roles. Also authenticating any requests starts with '/admin' with the ADMIN users.
    * Define the loging url. I've confiured the login page in Controller class.
    * After successful authentication, always go to the /secure/home url. Also configure this url in the controller class.
    * for a failed authentication, redirect to /error url.
    * Hitting the /logout url, a logged in user will logged out. While logging out delete the specified cookie.
* Also we need to define a password hashing encoder. He I've used the BCrypt.
    ```java
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    ```
    I've used this in the users() authentication provider.
* Let's define the User model for view-controller communication.
    ```java
    public class User {
        private String username;
        private String password;

        public User() {
        }

        public User(String username, String password) {
            this.username = username;
            this.password = password;
        }

        //Getters and Setters

    }
    ```
    This is nothing but a simple POJO class.
* Let's define our controller
    ```java
    @Controller
    public class MyController {

        @GetMapping(value = {"/login", "/logout"})
        public String loginPage(Model model) {
            User user = new User();
            model.addAttribute("user", user);
            return "login.html";
        }

        @GetMapping("/secure/home")
        public String home(Model model) {
            String username = SecurityContextHolder.getContext().getAuthentication().getName();
            model.addAttribute("username", username);
            return "home.html";
        }

        @GetMapping("/admin/dashboard")
        public String data(Model model) {
            String username = SecurityContextHolder.getContext().getAuthentication().getName();
            model.addAttribute("username", username);
            return "data.html";
        }
    }
    ```
    * For logging out, we also view the login page.
    * We can get the authenticated user details using _SecurityContextHolder.getContext().getAuthentication()_
## Template pages
* login.html
    ```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Login example</title>
    </head>
    <body>
        <h1>Log in</h1>

        <form action="#" th:action="@{/login}" th:object="${user}" method="post">

            <div class="form-group">
                <label for="username">Username</label>
                <input type="text" id="username" th:field="*{username}">
            </div>
            <div class="form-group">
                <label for="password">Password</label>
                <input type="password" id="password" th:field="*{password}">
            </div>
            <div class="form-group">
                <button type="submit">Submit</button>
            </div>

    </form>
    </body>
    </html>
    ```
* home.html
    ```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>This is home page</title>
    </head>
    <body>
        <h1>Home page</h1>
        <h2 th:text="|Welcome ${username}|"></h2>
        <p>For log out. Click <a href="/logout">here</a></p>
    </body>
    </html>
    ```
* data.html
    ```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Data page</title>
    </head>
    <body>
        <h1>This another secure page</h1>
        <h2 th:text="'Welcome '+${username}"></h2>
    </body>
    </html>
    ```
* error.html
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Error</title>
    </head>
    <body>
        <h1>There is a problem</h1>
    </body>
    </html>
    ```