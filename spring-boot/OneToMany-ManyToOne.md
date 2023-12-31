# Setup database and logs
```java
spring.datasource.url=jdbc:postgresql://localhost/test
spring.datasource.username=my_test
spring.datasource.password=test

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

spring.jpa.hibernate.ddl-auto=update

```
# Unidirectional Many to one
## Assume, a post can have multiple comments. In JPA, the codes may look like this

```java
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String title;
    private String content;

	// No args constructor
	// Getters and setters
}
```

```java
@Entity
public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String content;

    @ManyToOne
    @JoinColumn(name = "my_post_id", foreignKey = @ForeignKey(name = "FK_MY_POST_ID"))
    private Post post;
	
	// No args constructor
	// Getters and setters
}
```

In the debugger console window we can see
```bash
Hibernate: 
    create table comment (
        id serial not null,
        content varchar(255),
        post_id integer,
        primary key (id)
    )
Hibernate: 
    create table post (
        id serial not null,
        content varchar(255),
        title varchar(255),
        primary key (id)
    )
Hibernate: 
    alter table if exists comment 
       add constraint FKs1slvnkuemjsq2kj4h3vhx7i1 
       foreign key (post_id) 
       references post
```

We can see the foreign key name is 'post_id'. This is generated by the hibernate. Also Hibernate generates a constraint 'FKs1slvnkuemjsq2kj4h3vhx7i1'.

We can set those name using @JoinColumn
```java
    @ManyToOne
    @JoinColumn(name = "my_post_id", foreignKey = @ForeignKey(name = "FK_MY_POST_ID"))
    private Post post;

```

Now see the console
```bash
Hibernate: 
    create table comment (
        id serial not null,
        my_post_id integer,
        content varchar(255),
        primary key (id)
    )
Hibernate: 
    create table post (
        id serial not null,
        content varchar(255),
        title varchar(255),
        primary key (id)
    )
Hibernate: 
    alter table if exists comment 
       add constraint FK_MY_POST_ID 
       foreign key (my_post_id) 
       references post
```
So, the foreign key name is 'my_post_id' and the constraint name is 'FK_MY_POST_ID'.

# Using Unidirectional ManyToOne
```java
@Autowired
PostRepository postRepository;

@Autowired
CommentRepository commentRepository;

@Override
public void run(String... args) throws Exception {
	Post post1 = new Post();
	post1.setTitle("1st post");
	post1.setContent("dummy content");

	Post post2 = new Post();
	post2.setTitle("1st post");
	post2.setContent("dummy content");

	postRepository.save(post1);
	postRepository.save(post2);

	Comment comment1 = new Comment();
	comment1.setContent("1st comment");

	Comment comment2 = new Comment();
	comment2.setContent("2nd comment");

	Comment comment3 = new Comment();
	comment3.setContent("3rd comment");

	Comment comment4 = new Comment();
	comment4.setContent("4th comment");

	comment1.setPost(post1);
	comment2.setPost(post1);
	comment3.setPost(post2);

	commentRepository.save(comment1);
	commentRepository.save(comment2);
	commentRepository.save(comment3);
	commentRepository.save(comment4);
}
```
Here I've saved 2 posts and 4 comments. First 2 comments belongs to post 1 and 3rd comment belongs to post 2. And the 4th comment belongs to nothing.

The sample log is
```bash
Hibernate: 
    insert 
    into
        comment
        (content,my_post_id) 
    values
        (?,?)
```
[image of the comment table]
Now, delele the first post
```java
postRepository.delete(post1);
```
We will get an Exception
```bash
[ERROR: update or delete on table "post" violates foreign key constraint "fk_my_post_id" on table "comment"
```

# Unidirectional OneToMany
```java
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String title;
    private String content;

    @OneToMany
    List<Comment> commentList;
	
	// No args constructor
	// Getters and setters
}
```

```java
@Entity
public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String content;
	
	// No args constructor
	// Getters and setters
}
```
In the log,
```bash
Hibernate: 
    create table comment (
        id serial not null,
        content varchar(255),
        primary key (id)
    )
Hibernate: 
    create table post (
        id serial not null,
        content varchar(255),
        title varchar(255),
        primary key (id)
    )
Hibernate: 
    create table post_comment_list (
        comment_list_id integer not null unique,
        post_id integer not null
    )
Hibernate: 
    alter table if exists post_comment_list 
       add constraint FKbtle4p40d3oqwyny88k028w7q 
       foreign key (comment_list_id) 
       references comment
Hibernate: 
    alter table if exists post_comment_list 
       add constraint FKr4ysm7jrt2f6torg4nx7ljny4 
       foreign key (post_id) 
       references post
```
The colums hirearchy is like this
```bash
table comment:
	columns:
		id
		content
	constraints:
		comment_pkey

table post:
	columns:
		id
		content
		title
	constraints:
		post_pkey

table post_comment_list:
	columns:
		comment_list_id
		post_id
	constraints:
		FKbtle4p40d3oqwyny88k028w7q
		FKr4ysm7jrt2f6torg4nx7ljny4
		post_comment_list_comment_list_id_key
```
The insert commands
```bash
Hibernate: 
    insert 
    into
        comment
        (content) 
    values
        (?)
```
Those sql queries are executes simultaneously.
```bash
Hibernate: 
    insert 
    into
        post
        (content,title) 
    values
        (?,?)
Hibernate: 
    insert 
    into
        post_comment_list
        (post_id,comment_list_id) 
    values
        (?,?)
```
[image of comment, post and post_comment_list]
Now, delele the first post
```java
postRepository.delete(post1);
```
The log says.
```bash
Hibernate: 
    select
        p1_0.id,
        p1_0.content,
        p1_0.title 
    from
        post p1_0 
    where
        p1_0.id=?
Hibernate: 
    select
        c1_0.post_id,
        c1_1.id,
        c1_1.content 
    from
        post_comment_list c1_0 
    join
        comment c1_1 
            on c1_1.id=c1_0.comment_list_id 
    where
        c1_0.post_id=?
Hibernate: 
    delete 
    from
        post_comment_list 
    where
        post_id=?
Hibernate: 
    delete 
    from
        post 
    where
        id=?
```
[important image comment, also add post and post_comment_list]

The comments are remains but post and post_comment_list are updated.

We need to delete the comments as well. We have to modify the association like this
```java
	@OneToMany(cascade = CascadeType.MERGE, orphanRemoval = true)
    List<Comment> commentList;
```
The log is
```bash
Hibernate: 
    select
        p1_0.id,
        p1_0.content,
        p1_0.title 
    from
        post p1_0 
    where
        p1_0.id=?
Hibernate: 
    select
        c1_0.id,
        c1_0.content 
    from
        comment c1_0 
    where
        c1_0.id=?
Hibernate: 
    select
        c1_0.id,
        c1_0.content 
    from
        comment c1_0 
    where
        c1_0.id=?
Hibernate: 
    select
        c1_0.post_id,
        c1_1.id,
        c1_1.content 
    from
        post_comment_list c1_0 
    join
        comment c1_1 
            on c1_1.id=c1_0.comment_list_id 
    where
        c1_0.post_id=?
Hibernate: 
    delete 
    from
        post_comment_list 
    where
        post_id=?
Hibernate: 
    delete 
    from
        comment 
    where
        id=?
Hibernate: 
    delete 
    from
        comment 
    where
        id=?
Hibernate: 
    delete 
    from
        post 
    where
        id=?
```
[images post, comment]

# Bidirectional OneToMany or ManyToOne
First create a bean in the Comment class
```java
    @ManyToOne
    private Post post;
```
If we want a specific foreign key name then use @JoinColumn
```java
    @ManyToOne
    @JoinColumn(name = "custom_post_id")
    private Post post;
```
In log,
```bash
Hibernate: 
    alter table if exists comment 
       add constraint FK9vxj85128fstj3pd5sq4kkllc 
       foreign key (custom_post_id) 
       references post
```
Here, the bean name 'post' is important. We will use this in the Post class in mappedBy field.
```java
    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)
    List<Comment> commentList = new ArrayList<>();
```
The output will be the same.
```bash
Hibernate: 
    create table comment (
        id serial not null,
        post_id integer,
        content varchar(255),
        primary key (id)
    )
Hibernate: 
    create table post (
        id serial not null,
        content varchar(255),
        title varchar(255),
        primary key (id)
    )
Hibernate: 
    alter table if exists comment 
       add constraint FKs1slvnkuemjsq2kj4h3vhx7i1 
       foreign key (post_id) 
       references post
```
Also, we need to add some helper methods in the Post class for avoiding common errors. We have to define those in the One side. Here is the Post class.
```java
    public void addComment(Comment comment) {
        commentList.add(comment);
        comment.setPost(this);
    }

    public void removeComment(Comment comment) {
        commentList.remove(comment);
        comment.setPost(null);
    }
```
Now run the application
```java
    Post post1 = new Post();
    post1.setTitle("1st post");
    post1.setContent("dummy content");

    Comment comment1 = new Comment();
    comment1.setContent("1st comment");

    Comment comment2 = new Comment();
    comment2.setContent("2nd comment");

    post1.addComment(comment1);
    post1.addComment(comment2);

    postRepository.save(post1);
```
For delete
```java
    post1.removeComment(comment2);
	postRepository.save(post1);
```

