---
layout: post
title:  "My MacDventure Part 1: Setting Up Container-based C/C++ development Environment"
date:   2025-06-17 01:53:35 +0000
categories: My MacDventure
---

So there I was, with the gift of a brand new MacBook Air. Over the past  
decade and a half, my primary daily drivers have ALL been Linux.  

However, recently I decided to try something new. That something new  
involved getting a MacBook Air.  

I've been looking for a good coding platform, and most of the professional  
software engineers I know RAVE about how great Macs are. Well, Macs are  
Unix after all, and Unix is very similar to Linux. Therefore, I should be  
right at home. What could go wrong? Right????  

I'll save screen space by not ranting in frustration and will instead focus  
on providing a productive guide to help fellow Linux users find a  
navigable path as they embark on their Macdventure.  
 
What better way to start than by setting up your container-based developer  
tools without relying on the App Store or other elements of the Macaverse?  

Docker will not be used, since it looks a bit fishy, or rather shrimpy,
on the Mac. I will be utilizing Podman instead.  

Podman implementation on Mac works slightly differently than on Linux.  
It runs inside a Fedora CoreOS (FCOS) 41 (as of this writing) virtual  
machine (VM).  
**Note the VM initialization steps in the instructions below.**



## Installing Podman

1.  Download Podman `.pkg` package from the official podman github
    [release page](https://github.com/containers/podman/releases)

1.  Download the checkums from the official podman github
    [release page](https://github.com/containers/podman/releases)
    and save the checksums in a file named
    `shasums` in the same directory as the podman .pkg package

1.  Verify the checksums:

    ```zsh
    sha256sum -c --ignore-missing shasums
    ```

1.  Install podman system-wide

    ```zsh
    sudo installer -pkg podman-installer-macos-arm64.pkg -target LocalSystem
    ```

## Initialize Podman VM

1.  Initialize podman FCOS VM

    ```zsh
    podman machine init devvm
    podman machine start devvm
    ```

1.  Shelling into FCOS

    ```zsh
    podman machine ssh <vm-name>
    ```

1.  Update FCOS VM, install vim or other packages as needed

    ```zsh
    sudo dnf update -y && dnf install vim
    ```

1.  Restart FCOS VM after update

    ```zsh
    podman machine stop devvm
    podman machine start devvm
    ```

1.  Optionally Install `podman-compose`

    ```zsh
    sudo dnf install vim podman-compose -y
    ```


## Setting Up a Debian-Based C/C++ Development Environment

1.  Create a Dockerfile with the following content

    ```Dockerfile
    FROM docker.io/library/debian:latest

    RUN apt update && apt upgrade -y && \
    apt install build-essential vim git cpputest gdb cmake -y && \
    useradd developer -m -s /bin/bash 

    USER 1000:1000
    WORKDIR /home/developer
    ENTRYPOINT /bin/bash
    ```

1.  Build a developement container image `mydebian:dev`

    ```zsh
    podman build -f Dockerfile.debian -t mydebian:dev .
    ```

1.  Create a local `code` directory on the Mac side which will be mounted into
    the `mydebian:dev` image at `/home/developer/code`.

    ```zsh
    mkdir code
    ```

1.  Start the container and mount local `code` directory into it

    ```zsh
    podman run -ti --rm --name mydebian -v $(pwd)/code:/home/developer/code mydebian:dev 
    ```

1.  While in the container, create a "hello world code snippet",
    named `hello.c` with the following content:

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #define MSG "Hello, Mac!\n"


    int main() {
      printf(MSG);
      exit(0);
    }
    ```

1.  Compile and run it.

    ```zsh
    gcc -o hello hello.c
    ./hello
    ```

That should be enough to get started coding in C/C++.

Cheers!