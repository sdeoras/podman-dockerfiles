## podman dockerbuild
here are dockerfiles for building `podman` on centos and ubuntu.

* `go` is already installed and `GOPATH` is set to /go
* a copy of podman repo already exists inside `GOPATH`
* clone podman repo within `GOPATH` and run following commands to build:

```bash
cd $GOPATH/src/github.com/containers/libpod
make BUILDTAGS="selinux seccomp"
make install PREFIX=
```

