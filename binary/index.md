---
title: Binary Buildpack
---

<strong></strong>

Use the binary buildpack for running arbitrary binary web servers.

For more information, see the [buildpack documentation](../index.html).

## Usage ##

Unlike most other Cloud Foundry buildpacks, you must specify the binary
buildpack in order to use it in staging your binary file.
On a command line, use `cf push APP-NAME` with the `-b` option to specify the
buildpack.

For example:

```bash
$ cf push my_app -b https://github.com/cloudfoundry/binary-buildpack.git
```

You can provide Cloud Foundry with the shell command to execute your binary in
the following two ways:

* **Procfile**: In the root directory of your app, add a `Procfile` that
specifies a `web` task:

   	```yaml
   	web: ./app
	```
* **Command line**: Use `cf push APP-NAME` with the `-c` option:

   	```bash
   	$ cf push my_app -c './app' -b binary-buildpack
   	```

## Compiling your Binary ##

Cloud Foundry expects your binary to bind to the port specified by the `PORT`
environment variable.

The following example in [Go](https://golang.org/) binds a binary to the PORT environment variable:

```go
package main

import (
	"fmt"
	"net/http"
	"os"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %s", "world!")
}

func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServe(":"+os.Getenv("PORT"), nil)
}
```

Your binary should run without any additional runtime dependencies on the cflinuxfs2 or lucid64 root filesystem (rootfs).
Any such dependencies should be statically linked to the binary.

To boot a docker container running the cflinuxfs2 filesystem, run the following
command:

```bash
$ docker run -it cloudfoundry/cflinuxfs2 bash
```

To boot a docker container running the lucid64 filesystem, run the following
command:

```bash
$ docker run -it cloudfoundry/lucid64 bash
```

To compile the above Go application on the rootfs, golang must be installed. `apt-get install golang` and `go build app.go` will produce an `app` binary.

When deploying your binary to Cloud Foundry, use `cf push` with the `-s` option to specify the root filesystem it should run against.

```bash
$ cf push my_app -s (cflinuxfs2|lucid64)
```

To run docker on Mac OS X, we recommend [boot2docker](http://boot2docker.io/).