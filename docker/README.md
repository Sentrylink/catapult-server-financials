# NEM Catapult Server - Ubuntu

#### Building the toolchain

```
$ docker build -t nem-catapult-toolchain . -f docker/toolchain/Dockerfile
```

#### Compiling catapult

```
$ docker build -t nem-catapult . -f docker/server/Dockerfile
```
