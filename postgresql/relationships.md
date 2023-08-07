# One to One
* Add a primary key in table 1.
* Add a foreign key in table 2 referencing the primary key in table 1.
* Make the foreign key unique using unique constrain.
```sql
create table employee (
    id serial,
    name varchar(100),
    primary key(id)
);

create table address (
    id serial,
    city varchar(200),
    employee_id integer references employee(id),
    unique(employee_id),
    primary key(id)
);
```
* Now insert some employee
```sql
insert into employee values (default, 'john');
insert into employee values (default, 'amir');
insert into employee values (default, 'razzak');
```
* Fetch ids
```sql
select id from employee;
```
* Now insert some address using those ids
```sql
insert into address values (default, 'dhaka', 1);
insert into address values (default, 'dhaka', 2);
```
* See employee with id 1 has an address and employee with id 2 has an address. Both user ids are unique. Let's add an address with id 2.
```sql
insert into address values (default, 'sylhet', 2);
```
* We will get this exception
```
error: duplicate key value violates unique constraint "address_employee_id_key"
```

# One to Many
* Add a primary key in table 1.
* Add a foreign key in table 2 referencing the primary key in table 1.
* The primary key is the one side and the foreign key is the many side.
```sql

```