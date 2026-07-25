## 1. authorization vs authentication 
- authentication - who you are
- authorization - what you can do
## 2. Spring Boot - A pile of code that does the boring part for you
    boring part includes: 1. routing http req 
                          2. talking to dbs 
## 3. annotation : MetaData that tells Spring boot that its a reuseable piece of code!
        - @RestController → "this class handles incoming web requests"
        - @Entity → "this class represents a row in a database table"
## 4. Spring Data JPA - map java classes to db tables
1. JPA

        Database Table:
        ---------------
        | id | name  |
        |----|-------|
        | 1  | Trupti|

        Java Class:
        ----------
        java class User {
                Long id;
                String name;
        }

        - One row in the table = One Java object.
        - java User user = new User(1L, "Trupti");

2. Spring Data JPA

        Instead of writing SQL,
        -> SELECT * FROM users WHERE id = 1;
        
        we simply write
        -> java userRepository.findById(1L);

        Spring automatically writes the SQL for us.


3. Repository

        A Repository is the layer that talks to the database.

        Example:
        java public interface UserRepository extends JpaRepository<User, Long>{
        }
        
        - Spring automatically creates methods like:

        a. userRepository.save(user);      // INSERT
        b. userRepository.findById(1L);    // SELECT
        c. userRepository.findAll();       // SELECT *
        d.userRepository.delete(user);    // DELETE

## 5. JWT = JSON Web Token. - An ID that enables server to remember us. 
1. You log in with username + password.
2. Server checks they're correct.
3. Server creates a signed "ID card" (the JWT) containing your username, and hands it to you.
4. Every future request, you show that ID card 
5. Server checks the signature is valid (nobody tampered with it) and trusts what's written on it.

- note: It is not encrypted — anyone can read what's inside it, they just can't fake or edit it.

## 5. "BCrypt" / password hashing - password -> scram password

1. instead of stroing the actual password, you store a scrambled password to avoid leaks 

        -> everytime a user types in password :
                password -> Scramble -> check with scramble


## 6. Filter/ MiddleWare - A security Guard ! 
        Request
        |
        Filter
        |
        Controller

        Example: 
        --------
        A user sends a request: http GET /profile
        The Filter checks:
        - Is there a JWT token?
        - Is the token valid?
        ✅ Let the request continue to the Controller.
        ❌ Stop the request and return **401 Unauthorized**.


## 7. DTO - Data Transfer Object
- A **DTO** is a simple Java class used to send and receive data through an API.
- It is **different from your database entity**.

Suppose your `User` entity looks like this:

```java
class User {
    String username;
    String email;
    String passwordHash;
}
```

- When sending data to the client, we **don't want to expose** `passwordHash`.

So we create a DTO:
```java
class UserDTO {
    String username;
    String email;
}
```
Now only `username` and `email` are sent.

Remember

- **Entity** → Represents the database table.
- **DTO** → Represents the API request/response.

---

## 8. Validation Annotations

- Automatically check if the incoming data is valid.

```java
class RegisterRequest {

    @NotBlank
    String username;

    @Email
    String email;

    @NotBlank
    String password;
}
```
What they do:
------------
- `@NotBlank` → Field should not be empty.
- `@Email` → Must be a valid email format.
- `@Valid` → Tells Spring to check all validation rules.

If the data is invalid, Spring automatically returns:
```
400 Bad Request
```
No need to write manual `if` checks.
---


# What are we building?
### 1. Register
```
POST /api/auth/register
```
- User signs up.
- Password is **hashed** before storing.
- Password is **never stored as plain text**.
---

### 2. Login
```
POST /api/auth/login
```
- Check username and password.
- If correct, return a **JWT token**.
---

### 3. JWT Filter
Every protected request passes through a **Filter**.
The filter checks if the JWT is valid.
- Valid ✅ → Continue.
- Invalid ❌ → Return **401 Unauthorized**.
---

### 4. Roles
Each user has a role.
```
USER
ADMIN
```
For now, it's just a label.
Later, we can use it to allow only admins to access certain APIs.
