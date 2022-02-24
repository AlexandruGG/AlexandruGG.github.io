## Run a MySQL Docker Image on an M1 Mac

As of the time of writing, the [MySQL 5.7 Docker image](https://hub.docker.com/_/mysql?tab=tags&page=1&name=5.7) is (in)famously not yet natively supported on the newer M1 MacBooks.

The official Docker Desktop for Apple silicon announcement [provides some info](https://docs.docker.com/desktop/mac/apple-silicon/) on how to run an Intel image under emulation, similar to other [Stack Overflow posts](https://stackoverflow.com/questions/65456814/docker-apple-silicon-m1-preview-mysql-no-matching-manifest-for-linux-arm64-v8) on the topic.

However, when using a custom Dockerfile image in a team setting, where you might have a mix of platforms, the above will not work out of the box. What we want instead is a single image that leverages Docker's multiple architectures support.

---

### Build a Multi-CPU Architecture Image

#### Background

Nowadays Docker makes it easier than ever to build multi-architecture images with its [Buildx Docker CLI plugin](https://github.com/docker/buildx). This tool is already included in Docker Desktop for Windows, macOS, and some Docker Linux packages - can be installed by following the README otherwise.

The Docker [multi-CPU architecture support docs](https://docs.docker.com/desktop/multi-arch/) provide more background and info, but if you just want to get straight to the solution, continue reading below.

#### Building & Publishing

Our custom MySQL image Dockerfile might look like this:

```Dockerfile
FROM --platform=linux/amd64 mysql:5.7

COPY sql_mode.cnf /etc/my.cnf.d/
COPY docker-entrypoint-initdb.d /docker-entrypoint-initdb.d/
```

We could create a Makefile command that builds and pushes our multi-arch image all in one go:

```Makefile
.PHONY: build
build:
    docker buildx create --name db-builder --use \
    && docker buildx build --platform linux/amd64,linux/arm64 -t myorg/mysql:5.7 --push . \
    && docker buildx rm
```

Let's unpack the operations being done here:

- we've added the `--platform=linux/amd64` option to our Dockerfile so that we can fetch and build the image on an M1 machine

- we're creating a new Buildx builder because the default builder cannot be used for this operation; this is also cleaned up at the end, but feel free to keep it if you envision doing many multi-arch builds (run `docker buildx ls` to see existing builders)

- we're building the image from the Dockerfile and making it available for the `linux/amd64` and `linux/arm64` (this is our M1) platforms, tagging it as `myorg/mysql:5.7` and pushing it to our [Docker Hub](https://hub.docker.com/) repository all in one go

#### Tips & Gotchas

- this works with other MySQL-based DB images not yet supported on M1 Macs, such as [Percona](https://hub.docker.com/_/percona)

- if you're using a more specific version in your Dockerfile, e.g. `FROM mysql:5.7.37`, the `--platform` option might not work; try the generic `mysql:5.7` which fetches the latest version instead

- when testing this for the first time you should create a separate tag if your team or CI actively relies on the image, to not cause any disruption if something does not work as intended. For example, you can tag it as `myorg/mysql:5.7-M1` and change your `docker-compose` files to use the new tag instead in a separate branch

#### Happy Dockering!
