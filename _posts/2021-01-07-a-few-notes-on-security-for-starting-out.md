---
layout: post
title: "A few notes on security for starting out"
# title: "Security concerns for software developers"
author: "João Maria Janeiro"
categories: posts
tags: [security, web dev, web]
image: security.jpeg
---

If you are starting out on software development, you might be wondering what type of security concerns you should be aware of before releasing or publishing a project.

In this post I'll address some of the most important ones, if you are a security expert or a long time software engineer then this post is probably not for you. Although if you are curious you have the index ahead where I state what will be covered here.

If you implement these principles it doesn't mean your website or platform will be completely impenetrable but you will know you have a lot of protective measures that will prevent the most common attacks and keep your users pretty safe.

Of course the type of security you need depends on the application you are building, what type of data you deal with, for instance if you are dealing with financial data security is a much bigger concern than if you are developing a website for a local store. Nevertheless your users should be safe and protected when using your website or platform.


## Index
* [Encrypt passwords on database](#encrypt-passwords-on-database)
* [SQL Injection](#sql-injection)
* [Error handling](#error-handling)
* [Authentication and Authorization](#authentication-and-authorization)
* [Cross-Site Scripting (XSS) and CSRF](#cross-site-scripting-(xss))
* [HTTPS](#https)
* [Resource consumption](#resource-consumption)


## Encrypt passwords on database
This is an issue that gets overlooked a lot, it is a crucial part. If an attacker is able to break into your database he must not be able to steal your users' passwords, this is a big risk. Even if you think your application is small and its security is not that important, this is one issue you must not overlook. It's important because a lot of people use the same email and password for a lot of services so if an attacker gets your users' passwords he might get access to your users' personal emails, or worse. So you should really encrypt passwords in the database. If you are using a framework like Django this is already taken care of for you, there is no action necessary but if you are building a server using something like Spring Boot and you are using JDBC with an external database then you need to encrypt it. The defacto standard encryption algorithm for passwords at the time is [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt), although you could also use PBKDF2 (Django's default) or scrypt. 

Bcrypt is available for most every framework or language, [npm](https://www.npmjs.com/package/bcrypt), [python](http://zetcode.com/python/bcrypt/), [Django](https://docs.djangoproject.com/en/3.1/topics/auth/passwords/#using-bcrypt-with-django) and [Spring Boot](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/crypto/bcrypt/BCrypt.html). If you use another one you can probably find it online as well. You need to encrypt when you store the password in the database on account creation and decrypt for login or other matching operations, unless you use homomorphic encryption which I won't get into here.
Using it is usually as simple as: 

```python
# Python example
import bcrypt
salt = bcrypt.gensalt()
hashed = bcrypt.hashpw(your_password, salt)
```

## SQL injection
This is one of the top risks you might encounter. SQL injection is what happens when a user is able to run queries on your database that he should not be able to. How can this happen?
This vulnerability happens when your inputs are concatenated like this:

```java
// JAVA example
import Gson;
String username = new Gson().fromJson(request.getReader(), String.class);
String query = "SELECT * FROM Users WHERE username = '" + username + "'";
```
So in this query a user could input the username as `hello'; DROP TABLE Users; --` and so the resulting query would be `SELECT * FROM Users WHERE username = 'hello'; DROP TABLE Users; --' ` and all users would be deleted, or, if your passwords were not encrypted a user could instead query the table for all users' emails and passwords and get them.

#### So how to fix this?
This is an easy problem to fix, you simply have to use safe(named) parameters, which you could do like this:

```python
# Python example
cursor.execute("SELECT email FROM Users WHERE username = '" + username + "'") # Wrong
cursor.execute("SELECT admin FROM Users WHERE username = %(username)s",	{'username':	username}) #Safe
```

```Java
// JAVA example
SqlParameterSource namedParameters = new MapSqlParameterSource().addValue("id", 1);
return namedParameterJdbcTemplate.queryForObject("SELECT FIRST_NAME FROM EMPLOYEE WHERE ID = :id", namedParameters, String.class);

// Or something like this
jdbcTemplate.update("UPDATE Users SET profile_id = 3 WHERE username = ?", username);
```
So always used parameterized statements! They will prevent many attacks! If you only protect your application against one vulnerability this is probably the one!

## Error handling
If a users tries to do something illegal or that generates an error it's important to decide what to show to the users, simply showing the error the database gives is a big security risk as you could be revealing information you shouldn't. Let's say an user was trying to login and inserts a correct username but a wrong password, it's important that you do not tell the user the username was correct but the password failed, you should just say something like 'username and password did not match', as you have probably noticed most services you use do. 

SQL could return a statement like this:

```sql
-- SQL error
Violation of PRIMARY KEY constraint 'id'. Cannot insert duplicate key in object 'Users.id'. The duplicate key value is (2).

-- OR something like
The UPDATE statement conflicted with the CHECK constraint "CK__Employee__Salary". The conflict occurred in database "StoreManagement", table "Employee", column 'Salary'.
```
This gives an attacker information on the structure and content of your database that you should not make public.

Handling errors can be done using a `try/catch`. In case of an error the catch clause will be triggered and you can define different outputs for different type of exceptions, or, for different type of errors in the same exception. 


## Authentication and Authorization
First it's important to differentiate authentication from authorizations as these get mixed up too often. Authentication is to confirm that someone actually is who he claims to be, authorization is the permission users have to access resources.


You might want to allow anonymous/non-authenticated users, you can see my [previous post](https://joao-maria-janeiro.github.io/sample/anonymous-users.html) for that.

Managing authenticated users first requires that you make sure only the user has access to his data, so no other user can temper with other users' data. This means that every query to the database must be written so that only the user that owns the data can read and change this data or some other application-level verifications to prevent users tampering with other users' data.


Besides this you can also define roles such as Admin, Editor,View, Children/Parent or others and, depending on the users’ roles you might allow, or not, certain access and actions to those users.

You can implement this all by hand or leverage what most frameworks have as built-in. In Spring you can do something like this:

```java
// Java Spring Boot example
@Override
protected void configure(HttpSecurity http) throws Exception {	
    http.authorizeRequests()
        .antMatchers("/home").access("hasRole('ROLE_USER')")
        .antMatchers("/user").access("hasRole('ROLE_USER')")
        .antMatchers("/admin").access("hasRole('ROLE_ADMIN')")
        .antMatchers("/inventory").access("hasRole('ROLE_SELLER')")
        ...
}
```

As for authentication you should have email verification and use [HTTPS](#https) if you are using the HTTP protocol or encrypt the users' details if you are using another protocol.

## Cross-Site Scripting (XSS)
A platform that lets users enter text, like a username or a comment on a post, stores it and later on displays it to other users, is potentially susceptible to a **cross-site scripting (XSS)**  attack. In an attack like this, an attacker enters code written is a client-side scripting language such as JavaScript, instead of writing a comment or a post. When another user views the malicious text, the browser executes the script, which can perform actions like sending private cookie information back to the attacker or even performing an action on a different web server that the user may be logged into.

For instance, let's say a user is logged into the his bank account at the time of the script execution. The script could send cookie information of the bank's login to the attacker, who could then use this information to access the victims' bank account. Or the script could actually access pages on the bank's web site, with the correct information and perform a money transfer. This could in fact actually be done without scripting using a ingle line of code such as 
```html  
<img src="https://bank.com/transfer?amount=2000&destinationaccount=1334">
```

This kind of latter vulnerability is called **Cross-site request forgery (CSRF)**.

To prevent against such attacks you should do the following:

* Prevent your site from launching XSS or CSRF attacks

    The easiest way is to disallow any HTML tag in text that is inputted by users, as well as URLs and JavaScript entities. If your site allows the user to input rich text such as HTML or URLs you will need to carefully review what you accept and what you don't, you can take a look at [this](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html).

* Prevent your site from getting attacked with XSS or CSRF from other sites
    * You should verify the origin URL from which the request was made and only allow from your site
    * NEVER use a GET request to perform an update, this is a really bad practice and will make your server very vulnerable
    * If you use a framework like Django, you can use the XSS/CSRF protection mechanisms they have




## HTTPS

![image](https://moilahosting.co.za/wp-content/uploads/2016/10/hebergement-securise-1.png){: style="width: 200px;"}

Every time you see this lock in your browser it means HTTPS. 


> Hypertext Transfer Protocol Secure (HTTPS) is an extension of the Hypertext Transfer Protocol (HTTP). It is used for secure communication over a computer network, and is widely used on the Internet. In HTTPS, the communication protocol is encrypted using Transport Layer Security (TLS) or, formerly, Secure Sockets Layer (SSL). The protocol is therefore also referred to as HTTP over TLS, or HTTP over SSL.
> The principal motivations for HTTPS are authentication of the accessed website, and protection of the privacy and integrity of the exchanged data while in transit. It protects against man-in-the-middle attacks, and the bidirectional encryption of communications between a client and server protects the communications against eavesdropping and tampering. In practice, this provides a reasonable assurance that one is communicating with the intended website without interference from attackers.
> -- <cite>[Wikipedia][1]</cite>

[1]: https://en.wikipedia.org/wiki/HTTPS


So HTTPS provides encryption on the HTTP messages, this is something really important for when users create an account or login and you send passwords to the server from your site. By using HTTPS you don't need to encrypt the users' details before sending it to the server, you can just send them as-is.



## Resource consumption
If your server performs some complex operations over user input you could be exposing yourself to a Denial-of-Service (DoS) attack. If you allow your server to run indefinitely to perform a single request, an attacker could just send a couple of request and really clog your server. To prevent this you should have a maximum time allowed per request, where you would give a request timeout error if it was not possible to perform the request in the maximum time allowed. 


```java
// Spring boot Java example
@RequestMapping(value = "/request-url",
    method = RequestMethod.POST)
@Timed
@Transactional(timeout = 120)
```



