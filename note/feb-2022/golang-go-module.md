# Golang: Go Module

## How to use "go get" with a private repository

* If you encounter an error like this:

```
fatal: could not read Username for 'https://git.private.com': terminal prompts disabled
	Confirm the import path was entered correctly.
	If this is a private repository, see https://golang.org/doc/faq#git_https for additional information.
```

*   Visit at [https://golang.org/doc/faq#git\_https](https://golang.org/doc/faq#git\_https)

    * Make sure to use SSH to interact with VCS.
    * Open gitconfig file

    ```
    sudo vim ~\.gitconfig
    ```

    * Add the following lines to the end of the file:

    ```
    [url "ssh://git@git.private.com/"]
        insteadOf = https://git.private.com/
    ```
*   And then add a private repo to GOPRIVATE which is like a proxy:

    ```
    go env -w GOPRIVATE=ggit.private.com
    ```
* Some useful links:
  * [Go, FAQ](https://golang.org/doc/faq#git\_https)
  * [Gitlab, Dependency Management in Go](https://docs.gitlab.com/ee/development/go\_guide/dependencies.html)
  * [Stackoverflow, Add GOPRIVATE](https://stackoverflow.com/questions/60579900/how-to-fix-invalid-version-and-could-not-read-username-in-go-get-of-a-priv)

## How does "go module" work

* Go is source-based instead of artifact-based.
  * Artifact-based: package repository and source code repository are stored in different places. Such as npm for Node.js
  * Source-based: package and source code are stored in the same place.
  * [Gitlab, Dependency Management in Go](https://docs.gitlab.com/ee/development/go\_guide/dependencies.html)
* Since go 1.9 we can use the go module
  * [Go blog, Go module series - Using go modules](https://go.dev/blog/using-go-modules)
* Separate directory for new major version of go module
  * This is the highest recommendation from the Go team because it prevents the diamond dependency problem.
  * It also can separate branches for the major version, but not recommend by Go team. And I don't know why.
  * [Go blog, V2 go modules](https://go.dev/blog/v2-go-modules)
* Further reading:
  * [Russ Cox, Semantic Import Versioning](https://research.swtch.com/vgo-import)
  * [GO Wiki, Modules](https://github.com/golang/go/wiki/Modules#modules)
