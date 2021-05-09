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
Builder dbevenius:deno is trusted
Selected run image dbevenius/ubi8-s2i-deno:0.2
Pulling image dbevenius/ubi8-s2i-deno:0.2
0.2: Pulling from dbevenius/ubi8-s2i-deno
Digest: sha256:1c94fccb55506f3a34b3e2143ed390120b5c2dbf97c8f2c6622a0e9333c30a1a
Status: Image is up to date for dbevenius/ubi8-s2i-deno:0.2
Creating builder with the following buildpacks:
-> danbev.buildpack.deno@0.0.1
Provided Environment Variables
  MAIN=src/welcome.ts
  PERMISSIONS=--allow-read=/etc
Using build cache volume pack-cache-library_deno-example_latest-4ac570e96879.build
Running the creator on OS linux with:
Container Settings:
  Args: /cnb/lifecycle/creator -daemon -launch-cache /launch-cache -log-level debug -cache-dir /cache -run-image dbevenius/ubi8-s2i-deno:0.2 -process-type web deno-example
  System Envs: CNB_PLATFORM_API=0.4
  Image: pack.local/builder/697675676c616a766d76:latest
  User: root
  Labels: map[author:pack]
Host Settings:
  Binds: pack-cache-library_deno-example_latest-4ac570e96879.build:/cache /var/run/docker.sock:/var/run/docker.sock pack-cache-library_deno-example_latest-4ac570e96879.launch:/launch-cache pack-layers-vjpuwalrxg:/layers pack-app-ewdfbfujwy:/workspace
  Network Mode: 
===> DETECTING
======== Output: danbev.buildpack.deno@0.0.1 ========
Detecting Deno application...
---> Using TypeScript config -c tsconfig.json
======== Results ========
pass: danbev.buildpack.deno@0.0.1
Resolving plan... (try #1)
danbev.buildpack.deno 0.0.1
===> ANALYZING
Analyzing image "ace00a2c29123cd769ba06a95777a9a8c11c06d59decdcdae67cb6e58b2b28f3"
===> RESTORING
===> BUILDING
Deno Buildpack: 
deno 1.5.1
v8 8.7.220.3
typescript 4.0.3
main: src/welcome.ts
tsconfig: 
Bundle file:///workspace/src/welcome.ts
Download https://deno.land/x/date_fns@v2.15.0/getMinutes/index.js
Download https://deno.land/std@0.74.0/fs/exists.ts
Download https://deno.land/x/date_fns@v2.15.0/getHours/index.js
Download https://deno.land/x/date_fns@v2.15.0/toDate/index.js
Download https://deno.land/x/date_fns@v2.15.0/_lib/requiredArgs/index.js
Check file:///workspace/src/welcome.ts
===> EXPORTING
no project metadata found at path 'project-metadata.toml', project metadata will not be exported
Reusing layers from image with id 'ace00a2c29123cd769ba06a95777a9a8c11c06d59decdcdae67cb6e58b2b28f3'
Layer 'slice-1' SHA: sha256:b6dc892c2761d64f808a66735513aa304493ac6a0e228ed67ff1175f64f68829
Reusing 1/1 app layer(s)
Reusing tarball for layer "launcher" with SHA: sha256:576e5f5696c6fcae9b679dca450d7ab127176f5c529bc9790b42abb9e3658cfa
Reusing layer 'launcher'
Layer 'launcher' SHA: sha256:576e5f5696c6fcae9b679dca450d7ab127176f5c529bc9790b42abb9e3658cfa
Reusing tarball for layer "config" with SHA: sha256:743848cdbf04ba82d525c7a3c834e1fe5d3785ab496494715d08189222be408e
Reusing layer 'config'
Layer 'config' SHA: sha256:743848cdbf04ba82d525c7a3c834e1fe5d3785ab496494715d08189222be408e
Reusing tarball for layer "process-types" with SHA: sha256:83d85471d9f8a3834b4e27cf701e3f0aef220cc816d9c173c7d32cd73909a590
Reusing layer 'process-types'
Layer 'process-types' SHA: sha256:83d85471d9f8a3834b4e27cf701e3f0aef220cc816d9c173c7d32cd73909a590
Adding label 'io.buildpacks.lifecycle.metadata'
Adding label 'io.buildpacks.build.metadata'
Adding label 'io.buildpacks.project.metadata'
Setting CNB_LAYERS_DIR=/layers
Setting CNB_APP_DIR=/workspace
Setting CNB_PLATFORM_API=0.4
Setting CNB_DEPRECATION_MODE=quiet
Prepending /cnb/process and /cnb/lifecycle to PATH
Setting default process type 'web'
Setting ENTRYPOINT: '/cnb/process/web'
*** Images (ace00a2c2912):
      deno-example

*** Image ID: ace00a2c29123cd769ba06a95777a9a8c11c06d59decdcdae67cb6e58b2b28f3
Successfully built image deno-example

```
During the detaction phase this buildpack will detect TypeScript configuration
file and use that during the buildphase.

During the build phase buildpack will use Deno's `bundle` command to compile the
TypeScript into JavaScript to make the execution of the final application faster
as this will not be neccessary at runtime.

After the image has been created we can then run it using the following command:
```console
$ docker run -t deno-example
Welcome to Deno(14:23) 🦕
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
