---
layout: post
title: "Running picamera2 on containers"
author: "Romero Galiza"
comments: true
---

## Running picamera2 on containers


When running picamera2, or more specifically: *libcamera*, from a container (Docker), you might end up with the following issue:

```
>>> from picamera2 import Picamera2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python3/dist-packages/picamera2/__init__.py", line 1, in <module>
    import libcamera
  File "/usr/lib/python3/dist-packages/libcamera/__init__.py", line 4, in <module>
    from ._libcamera import *
ModuleNotFoundError: No module named 'libcamera._libcamera'
``` 

Several posts on different sources (GitHub Issues, StackOverflow, Medium, etc) point to different solutions, such as compiling libcamera and making sure the Python bindings are generated, and so on. Most issues and their respective answers were reported between 2014-2019, at the time of writing (2023) these are made obsolete. Besides, there is a considerable amount of information missing in most documents reporting issues with `picamera2` when running from containers, such
as the base image that has been used and what other requirements have been met.

A current solution is to simply rely on pre-build images from Balena, I've tested he one below and the results were positive.

```
FROM balenalib/raspberry-pi-debian:latest

WORKDIR /example

RUN apt update && apt upgrade && apt install -y python3-picamera2 --no-install-recommends

COPY . .

# let's imagine my example.py exposes a http endpoint:
EXPOSE 8080/tcp

CMD ["python3", "example.py"]
``` 

Even though the example above (should) run successfully, from the same builder images such as `balenalib/raspberrypi4-64-python:3.11-bullseye` (given you're running on a RaspberryPi 4) also failed. I'm yet to investigate the differences between them.

It's worth mentioning that to access the Raspberry Pi camera bus from a container you must enable privileged mode (`--privileged`). As per the official Docker documentation has the following effects: the `--privileged` flag gives all capabilities to the container, and it also lifts all the limitations enforced by the device cgroup controller. In other words, the container can then do almost everything that the host can do.

```
$ docker run -d \
  --network="host" \
  --name="your-own-name-here" \
  --volume /dev/:/dev/ \
  --env UDEV=1 \
  --device /dev:/dev \
  --privileged \
  my-example-image
```

In the example above I'm explicitly mounting the `/dev` volume and devices.
