# Dependencies
* Starter JPA
* PostgreSQL Driver
* Lombok
# Modify application.yaml
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost/employee
    username: root
    password: root
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```

# Define Entities
```java
@Data
@Builder
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String name;
    private String email;
    private String phone;
}
```
# Define Search criteria
Simple blueprint for key, value and ops
```java
public record SearchCriteria(String key,
                             String operation,
                             String value) {
}

```

# Define Repository
Define the repository add JpaSpecificationExecutor<T> in there.
```java
public interface EmployeeRepository extends JpaRepository<Employee, Long>,
                                            JpaSpecificationExecutor<Employee> {
}
```

# Pattern 1
implement the Specification interface
```java
public class EmployeeSpecification implements Specification<Employee> {

    SearchCriteria searchCriteria;

    @Override
    public Predicate toPredicate(Root<Employee> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder) {
        if (searchCriteria.equals("like")) {
            return criteriaBuilder.equal(root.get(searchCriteria.key()), "%" + searchCriteria.value() + "%");
        }
        return criteriaBuilder.equal(root.get(searchCriteria.key()), searchCriteria.value());
    }
}
```

# Usage
```java
public class EmployeeService {
    @Autowired
    EmployeeRepository employeeRepository;

    public void findAllEmployee() {
        EmployeeSpecification specification = new EmployeeSpecification(new SearchCriteria("name", "", "john"));
        List<Employee> list = employeeRepository.findAll(specification);
    }
}
```
Also, we can use sort and paging and multiple predicates.
```java
public class EmployeeService {
    @Autowired
    EmployeeRepository employeeRepository;

    public void findAllEmployee() {
        Sort sort = Sort.sort(Employee.class)
                .by(Employee::getEmail)
                .ascending();

        Pageable page = PageRequest.of(0, 10, sort);

        EmployeeSpecification nameSpec = new EmployeeSpecification(new SearchCriteria("name", "like", "john"));
        EmployeeSpecification emailSpec = new EmployeeSpecification(new SearchCriteria("email", "=", "john@mail.com"));

        List<Employee> list = employeeRepository
                .findAll(nameSpec.and(emailSpec), page)
                .toList();
    }
}
```

# Pattern 2
```java
public interface EmployeeSpecificationDsl {

    public static Specification<Employee> byExactEmail(Employee employee) {
        return new Specification<Employee>() {
            @Override
            public Predicate toPredicate(Root<Employee> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder) {
                return criteriaBuilder.equal(root.get("name"), employee);
            }
        };
    }
}
```
Or simply replace this with lambda
```java
public interface EmployeeSpecificationDsl {

    static Specification<Employee> byExactEmail(String email) {
        return (root, query, criteriaBuilder) -> criteriaBuilder.equal(root.get("name"), email);
    }

    static Specification<Employee> byNameLike(String name) {
        return (root, query, criteriaBuilder) -> criteriaBuilder.like(root.get("name"), "%" + name + "%");
    }
}
```
Now use like this,
```java
public class EmployeeService {
    @Autowired
    EmployeeRepository employeeRepository;

    public List<Employee> findAllEmployee2(String email, String name) {
        Sort sort = Sort.sort(Employee.class).by(Employee::getEmail).ascending()
                .and(Sort.sort(Employee.class).by(Employee::getName).ascending());

        Pageable page = PageRequest.of(0, 10, sort);

        Stream<Employee> stream = employeeRepository.findAll(EmployeeSpecificationDsl.byExactEmail(email)
                                                    .and(EmployeeSpecificationDsl.byNameLike(name))
                                    , page).stream();
        return stream.toList();
    }
}
```
