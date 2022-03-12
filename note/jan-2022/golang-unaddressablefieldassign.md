# Golang: UnaddressableFieldAssign

**Golang: UnaddressableFieldAssign**

* left-hand side must **addressable** or **map index expression**, or the blank identifier (\_)**.**
* addressable: a variable, pointer indirection, or slice indexing operation; or a field selector of an addressable struct operand; or an array indexing operation of an addressable array.
* map index expression is not addressable eg. `tree[“one”]`, it’s a special case that can be assigned. Because it’s implemented in a heap so its location is not the same all the time.

**Program 1:**

```go
type B struct {
    C string
}

type A map[string]B

func main() {
    a := make(A)
    a["1"] = B{}
    a["1"].C = "c" //cannot assign to struct field a["1"].C in map
}
```

* The above program turns out an error even though `a[“1”]` is a map index expression that can be assigned a value but `a[“1”].C` is a map to structure that is not addressable so it can’t.
* In a simple case such as `map[string]bool` we can assign a value like this `visited[“x”] = true`.
* `a[“1”]` is just a copy value of a struct in the map.

**Program 2:**

```go
type B struct {
    C string
}

type A map[string]*B

func main() {
    a := make(A)
    a["1"] = &B{}
    a["1"].C = "c"
}
```

* The above program would be great at compiling without any error.
* a\[“1”] is a copy value of a pointer to a struct in the map. So the pointer is addressable.
* In Go spec failed to point out why a selector of the map to struct pointer is addressable.
* But I got it. The map just stores addresses not copies of the real struct, in the backend, it’s not involved with any data in a real struct.

**Forget it**

* I think all of this is the failure design of Golang.
* Is it a hard part for controlling all struct items that are stored in a map? Why can’t we do this?

**Further readings**:

* [https://utcc.utoronto.ca/\~cks/space/blog/programming/GoAddressableValues](https://utcc.utoronto.ca/\~cks/space/blog/programming/GoAddressableValues)
* [https://go.dev/ref/spec#Address\_operators](https://go.dev/ref/spec#Address\_operators)
* [https://go.dev/ref/spec#Assignments](https://go.dev/ref/spec#Assignments)
* [https://github.com/golang/go/issues/3117](https://github.com/golang/go/issues/3117)
