# Integrate Thymeleaf in a Spring Boot Project

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
    * **Spring WEB**(spring-boot-starter-web): Contains Spring MVC and embedded Tomcat container
    * **Thymeleaf**(spring-boot-starter-thymeleaf)
* Click the **GENERATE** Button
* Copy the compressed file to a desired location
* Unzip it
* Add the index.html in the src/main/resources/
    ```html
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Index html</title>
        </head>
        <body>
            <h1>This is a test</h1>
        </body>
    </html>
    ```
    As we have the thymeleaf starter dependency, it has the necessary auto configuration. It will automatically find the index.html file.
* Write a controller in a new java file. I've created the MvcController.java
    ```java
    @Controller
    @RequestMapping("/customer")
    public class MvcController {

        
    }
    ```
## View a single Customer
* Prepare the model class. This is a simple POJO class.
    ```java
    public class Customer {
        private int id;
        private String name;
        private String email;
        private String gender;

        public Customer() {
        }

        public Customer(int id, String name, String email, String gender) {
            this.id = id;
            this.name = name;
            this.email = email;
            this.gender = gender;
        }
        //Generate all getters and setters
    }
    ```
* Add this method in the MvcController class
    ```java
    @GetMapping("{id}")
    public ModelAndView customerDetails(@PathVariable int id) {
        Customer customer =new Customer(id, "test1", "test1@mail.com", "m");

        ModelAndView mv = new ModelAndView();
        mv.setViewName("customer-details");
        mv.addObject("customer", customer);

        return mv;
    }
    ```
    Here I've set the view name and add the object which is passed to the template.
* Create a template customer-details.html in the resources/templates directory and put this code
    ```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Customer Details</title>
    </head>
    <body>
        <p th:text="${customer.id}"></p>
        <h1 th:text="${customer.name}"></h1>
        <p th:text="${customer.email}"></p>
        <p th:text="${customer.gender}"></p>
    </body>
    </html>
    ```
    Here I have used the thymeleaf namespace in the html tag. I've accessed the passed model object by it's bean name. We can access the variable by using the standard ${...} tag.
## View the list of customers
* Prepare the controller code
    ```java
    @GetMapping("list")
    public ModelAndView customerList() {
        List<Customer> list = new ArrayList<>();
        list.add(new Customer(1, "test1", "test1@mail.com", "m"));
        list.add(new Customer(2, "test2", "test2@mail.com", "m"));
        list.add(new Customer(3, "test3", "test3@mail.com", "f"));

        ModelAndView mv = new ModelAndView();
        mv.setViewName("customer-list");
        mv.addObject("customers", list);

        return mv;
    }
    ```
    Here I've created a ModelAndView object and set the view name and the list object and send it to the view.
* The template code
    ```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Customer list</title>
    </head>
    <body>
        <ul th:each="customer: ${customers}">
            <li>
                <h2 th:text="${customer.name}"></h2>
                <p th:text="${customer.email}"></p>

                <p th:if="${customer.gender == 'm'}" th:text="man"></p>
                <p th:unless="${customer.gender == 'm'}" th:text="female"></p>
            </li>
        </ul>
    </body>
    </html>
    ```
    We can iterate using the th:each attribute. In this code I also cover the conditional statements th:if and th:unless.
## Form example
* First prepare the form page from controller
    ```java
    @GetMapping("createForm")
    public String createCustomer(Model model) {
        Customer customer = new Customer();
        model.addAttribute("customer", customer);

        List<String> genderList = List.of("m", "f");
        model.addAttribute("genderList", genderList);

        return "create-customer";
    }
    ```
    We have to pass a empty Customer object for binding. The values will be bind in the temaplate form using th:object attribute. Also pass the genderList object for populating a select input.
* Prepare the template
    ```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Create customer</title>
    </head>
    <body>
        <form action="#" th:action="@{/customer/save}" th:object="${customer}" method="post">
            <label for="id">Id</label>
            <input type="number" id="id" th:field="*{id}">

            <label for="name">Name</label>
            <input type="text" id="name" th:field="*{name}">

            <label for="email">Email</label>
            <input type="text" id="email" th:field="*{email}">

            <label for="gender">Gender</label>
            <select name="gender" id="gender" th:field="*{gender}">
                <option th:each="g: ${genderList}" th:value="${g}" th:text="${g}"></option>
            </select>

            <button type="submit">Submit</button>
        </form>
    </body>
    </html>
    ```
    Here th:field maps with the th:object. For example, *{id} is equivalent to ${customer.id}
    th:action is the path where this form will be submitted. I've defined it in the controller too.
* Code the action class
    ```java
    @PostMapping("save")
    public ModelAndView saveCustomer(@ModelAttribute("customer") Customer customer, Model model) {
        ModelAndView mv = new ModelAndView();
        mv.setViewName("customer-details");
        mv.addObject("customer", customer);

        return mv;
    }
    ```
    Here I've collect all the data using ModelAttribute and send it to the customer details page. I already have writter the code for customer-details above.
* Linking -
    * In templates(files in templates folder)
        ```html
        <a href="/customer/list" th:href="@{/customer/list}">customer list</a>
        ```
    * In index.html, as usual
        ```html
        <a href="/customer/list">customer list</a>
        ```
* Styling: create a style(here, mystyle.css) in the static directory. I've also created the styles directory under the static directory for keeping code clean. In template files, we have to include using the th:herf tag.
    ```html
    <link rel="stylesheet" th:href="@{/styles/mystyle.css}">
    ```



