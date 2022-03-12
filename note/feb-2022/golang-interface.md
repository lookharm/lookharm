# Golang: Interface

## Interface

* For provide **duck typing**: type that can do many things like ducks.
* Interface just pair of value: pointer to the associated data and pointer to an itable (interface table)
* itable contains list of pointer to methods of an interface.
* This is the one thing that Russ Cox want if he can export one feature from Golang to another language.
* Further reading:
  * [Russ cox, Go Data Structures: Interfaces](https://research.swtch.com/interfaces)

## Embedding Interfaces in Go

* How to implement Interface
  * Inject all methods that have the same signature in an interface.
  * Embedding Interfaces
* Embedding Interfaces
  * Interface is a set of methods.
  * We called concrete value implement same interface if it it has a subset of methods equal methods in an interface.
  *   Embedding struct in another struct:

      ```go
        type A struct {

        }
        func (a A) F1() {}

        type B struct {
            A
        }
      ```

      * Struct B would inherit method F1() from struct A
*   Embedding interface in another struct:

    ```go
      type A interface {
          F1()
      }

      type B struct {
          A
      }
    ```

    * Struct B would inherit method F1() from struct A, struct B has all methods in an interface A. So we can called struct B is implement an interface A.
    * But F1() just point to nil value because we don't actually implement F1(), if we call F1() it will panic at run-time.
* Use-case:
  * unit-test:
    *   We do not want to implement all method that we do not want to test. So we can implement the interface like this:

        ```go
        type User struct {
            Name string
        }

        type Repository interface {
            GetUsers() []User
            CreateUser(User)
        }

        type mkRepository struct {
            Repository
            getUsers func() []User
        }

        func (m *mkRepository) GetUsers() []User {
            return m.getUsers()
        }

        func Test_GetUsers() {
            var r Repository
            r = &mkRepository{
                getUsers: func() []User {
                    return []User{User{Name: "a"}, User{Name: "b"}}
                },
            }
            users := r.GetUsers()
            fmt.Println(users)
        }
        ```

        * We want to test only GetUsers() so we just implement only GetUsers()
* Further reading:
  * [Golang By Example, Embedding Interfaces in Go](https://golangbyexample.com/embedding-interfaces-go)
