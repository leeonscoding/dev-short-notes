# Define the schema
Here is a many to many relationship. A student can multiple courses and A course can have multiple students.
```sql
CREATE TABLE student
  (
     id    SERIAL,
     name  VARCHAR(100),
     email VARCHAR(100),
     phone VARCHAR(20),
     PRIMARY KEY(id)
  );

CREATE TABLE course
  (
     id          SERIAL,
     name      VARCHAR(100),
     duration  varchar(20),
     
     PRIMARY KEY(id)
  );

  
CREATE TABLE student_course_map
  (
     id         SERIAL,
     course_id  INTEGER,
     student_id INTEGER,
     PRIMARY KEY(id)
  );

ALTER TABLE student_course_map
  ADD FOREIGN KEY(student_id) REFERENCES student(id);
ALTER TABLE student_course_map
  ADD FOREIGN KEY(course_id) REFERENCES course(id);
```

# Insert some data
Insert into student table
```sql
INSERT INTO student
VALUES     (1,
            'jhon',
            'jhon@mail.com',
            '123');

INSERT INTO student
VALUES     (2,
            'rony',
            'rony@mail.com',
            '156');
```
Insert into course table
```sql
INSERT INTO course
VALUES     (1,
            'Java',
            '1hr');

INSERT INTO course
VALUES     (2,
            'J2EE',
            '3hr');

INSERT INTO course
VALUES     (3,
            'Spring Boot',
            '1.5hr');

INSERT INTO course
VALUES     (4,
            'Docker',
            '30min'); 
```
Insert into student_course_map table
```sql
INSERT INTO student_course_map
VALUES     (1,
            2,
            1);

INSERT INTO student_course_map
VALUES     (2,
            3,
            1);

INSERT INTO student_course_map
VALUES     (3,
            1,
            2);

INSERT INTO student_course_map
VALUES     (4,
            2,
            2); 
```
# Inner join
```sql
SELECT s.name,
       s.email,
       s.phone,
       c.name     AS coursename,
       c.duration AS "course duration"
FROM   student_course_map sc
       INNER JOIN student s
               ON s.id = sc.student_id
       INNER JOIN course c
               ON c.id = sc.student_id;
```
We can interchange various join mehtods instead od inner join.