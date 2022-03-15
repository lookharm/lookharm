# Go: Interface

## Interface

* For providing **duck typing**: the type that can do many things like ducks.
* The interface is a pair of values: a pointer to the associated data and a pointer to an _itable_ (interface table)
* _itable_ contains a list of pointers to methods that were defined within an interface.
* This is the one thing that Russ Cox wants if he can export one feature from Go to another language.
* Further reading:
  * [Russ cox, Go Data Structures: Interfaces](https://research.swtch.com/interfaces)

## Embedding Interfaces in Go

* 2 ways to implement Interface
  * 1\. Write down all methods in an interface.
  * 2\. Embedding Interfaces.
* Embedding Interfaces
  * Interface is a set of methods.
  *   Let's see if we embedded struct in another struct:

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

    * Struct B would inherit method F1() from struct A, struct B has all methods in an interface A. So struct B is implement an interface A.
    * But F1() doesn't point to any function because we don't actually implement F1() or write down any description of this function, if we call F1() it will panic at run-time.
* Use-case:
  * unit-test:
    *   We don't want to implement all method that we don't like to test. So we can implement the interface like this:

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
