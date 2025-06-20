---
layout: post
title:  "My MacDventure Part 2: Burning ISOs on a Mac"
date:   2025-07-15 01:53:35 +0000
categories: My MacDventure
---

# Burning ISOs On A Mac




## Intro

So there I was again trying to feel at home on the Mac.

The biggest challenge with migrating from Linux to Mac is the lack of tools and ways to do things that are taken for granted on a Linux system.

For example the popular `gpg` command line tool is not present on the Mac.

So, your typical workflows, such as downloading and validating an ISO, are very likely to be derailed.

Thankfully containerization may help alleviate some of those challenges.

This installment of my Macdventure is intended to provide another map sheet
for navigating around the Mac hurdles.

Due to lack of native GPG tooling on the Mac our workaround will be to use the official Rocky 9 container, mount our downloads inside and validate them from there.

Let's get to it.




## Download the ISO and Signatures

1.  Let's start by creating a diretory where ISO and checksum-related
    files will be donwloaded

    ```zsh
    mkdir -p ~/iso_test/debian
    ```

1.  Download Debian ISO checksums, checksum signatures, and the ISO itself.

    ```zsh
    curl -O --output-dir ~/iso_test/debian https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/SHA256SUMS
    curl -O --output-dir ~/iso_test/debian https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/SHA256SUMS.sign
    curl -L -O --output-dir ~/iso_test/debian https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/debian-12.11.0-amd64-DVD-1.iso
    ```

    **Note `-L` option to follow redirects to the ISO.**
    **It is needed due to high chance of being redirected to the closes mirror**




## Getting Around the Lack of GPG Utility

Due to lack of native GPG tooling on the Mac our workaround will be to use the official Rocky 9 container, mount our downloads inside and validate them from there.

1.  Start rockylinux container and mount the directory cotnaing downloaded files

    ```zsh
    podman run --rm -ti --name debian --user 65534:65534 -v $HOME/iso_test/debian:/dvd docker.io/library/rockylinux:9.3 
    ```

1. Once inside the container change into the directory with downloaded files.

    ```bash
    cd /dvd
    ```

1.  Check the GPG key ID with which the checksum file was sigend.
    The key ID will be the last 16 characters of the finterprint.

    ```bash
    gpg --verify SHA256SUMS.sign 
    ```

    ![GPG key ID](/assets/images/debian_iso/gpg_key_id.png)

1.  Verify that the key is one of the official debian keys posted
    [here](https://www.debian.org/CD/verify).

    ![debian keys](/assets/images/debian_iso/debian_gpg_key_listings.png)

1.  Download the correct GPG key.

    ```bash
    gpg --keyserver hkps://keyring.debian.org:443 --recv-keys 0xDA87E80D6294BE9B
    ```

    ![download GPG key](/assets/images/debian_iso/download_gpg_key.png)

1.  Validate signatures.  Note the `using RSA key` and `Good signature from` messages.

    ```bash
    gpg --verify SHA256SUMS.sign 
    ```
    
    ![validate checkum signature](/assets/images/debian_iso/validate_cheskum_signature.png)

1.  Validate the ISO.

    ```bash
    sha256sum -c --ignore-missing SHA256SUMS
    ```

    ![validate ISO](/assets/images/debian_iso/validate_iso.png)




## Burn the ISO

1.  Insert the USB stick into the, when the window pops up click Ignore

1.  Check disk device and make sure it's not mounted.  In my case the
    results of the command below, listed the USB stick on which the ISO
    will be burned as `disk4`.

    ```zsh
    diskutil list
    mount | grep <identifier> 
    ```

1.  Finally burn the ISO.
    **Be very careful with specifying the output `of=` device!!!**
    **There is a reason why `dd` is nicknamed "data destroyer."**

    ```zsh
    sudo dd if=/Users/<username>/iso_test/debian/debian-12.11.0-amd64-DVD-1.iso of=/dev/disk4 bs=4M status=progress 
    ```