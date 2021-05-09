# Buildpack for Deno

Build pack to use Deno with Red Hat Universal Base Images (UBI).


### Create/Update the builder

This needs to be done whenever [detect](./bin/detect) or [build](./bin/build)
are updated.

```console
$ pack builder create dbevenius:deno --config ./builder.toml
Successfully created builder image dbevenius:deno
Tip: Run pack build <image-name> --builder dbevenius:deno to use this builder
```

Optionally trust this builder:
```
$ pack config trusted-builders add dbevenius:deno
```

### Usage in this buildpack

The following will use the [demo-example](https://github.com/danbev/deno-example).

```console
$ pack -v build deno-example --builder dbevenius:deno --path ../deno-example -e MAIN="src/welcome.ts" -e PERMISSIONS="--allow-read=/etc"
```

After the image has been created we can then run it using the following command:
```console
$ docker run -t deno-example
Welcome to Deno(7:2) 🦕
Does /etc/passwd exist: true
```

### Troubleshooting
On Fedora 31 I ran into the following issue when running the `pack build` command
and I'd made sure that the current user was a member of the `docker` group and
in addition verified that this change was made effective using both `newgrp` and
also logging in again but none worked. I finally found the following to work.
```console
  Network Mode: 
ERROR: failed to initialize docker client: failed to connect to docker socket: dial unix /var/run/docker.sock: connect: permission denied
ERROR: failed to build: executing lifecycle: failed with status code: 1
```
```console
$ getenforce
Enforcing
$ sudo setenforce Permissive
```
After this change I was able to use the `pack` command.
