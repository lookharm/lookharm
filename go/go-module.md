# Go: Module

## How to use "go get" with a private repository

* If you have encountered an error like this it can be determined that you tried to use go get with the repository that you have lack of permissions:

```
fatal: could not read Username for 'https://git.private.com': terminal prompts disabled
	Confirm the import path was entered correctly.
	If this is a private repository, see https://golang.org/doc/faq#git_https for additional information.
```

* If you can access this repository but _go get_ don't. Then you should tell it to get this kind of repository via SSH.
*   Visit at [https://golang.org/doc/faq#git\_https](https://golang.org/doc/faq#git\_https)

    * Make sure to use SSH to interact with VCS.
    * Open _.gitconfig_ file.

    ```
    sudo vim ~\.gitconfig
    ```

    * Add the following lines to the end of the file:

    ```
    [url "ssh://git@git.private.com/"]
        insteadOf = https://git.private.com/
    ```
*   And then add a private repository's domain name to GOPRIVATE:

    ```
    go env -w GOPRIVATE=git.private.com
    ```
* Some useful links:
  * [Go, FAQ](https://golang.org/doc/faq#git\_https)
  * [Gitlab, Dependency Management in Go](https://docs.gitlab.com/ee/development/go\_guide/dependencies.html)
  * [Stackoverflow, Add GOPRIVATE](https://stackoverflow.com/questions/60579900/how-to-fix-invalid-version-and-could-not-read-username-in-go-get-of-a-priv)

## How does "go module" work

* Since Go1.9 we can use the _go module_.
* Go module is source-based instead of artifact-based.
  * Artifact-based: package repository and source code repository are stored in different places. Such as Node.js's npm.
  * Source-based: package repository and source code repository are stored in the same place.
* Versioning convention
  * Separate directory for a new major version of go module. For an example /v1 and /v2.
  * This is the highest recommendation from the Go team because it prevents the diamond dependency problem.
  * It also can separate branches for the major version, but they don't recommend it. And I don't know why.
* Further reading:
  * [Russ Cox, Semantic Import Versioning](https://research.swtch.com/vgo-import)
  * [GO Wiki, Modules](https://github.com/golang/go/wiki/Modules#modules)
  * [Gitlab, Dependency Management in Go](https://docs.gitlab.com/ee/development/go\_guide/dependencies.html)
  * [Go blog, Go module series - Using go modules](https://go.dev/blog/using-go-modules)
  * [Go blog, V2 go modules](https://go.dev/blog/v2-go-modules)
