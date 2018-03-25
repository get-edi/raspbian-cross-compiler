## Recompiled Debian Packages for Raspbian Cross Compilation

Draft: How the packages got recompiled:

1. Install amd64 container with foreign architecture armhf enabled
(amd64: Debian stretch, armhf: Raspbian stretch)
2. Pretent to be Raspbian (/etc/os-release, /etc/lsb-release).
3. Recompile cross-toolchain-base for arm6hf:
    * Eventually remove cross and armhf related packages.
    * apt-get source cross-toolchain-base
    * cd cross-toolchain-base-18
    * sudo apt-get build-dep cross-toolchain-base
    * debuild --set-envvar=CROSS_ARCHS=armhf -us -uc
4. Install debs from previous step.
5. Recompile gcc-6-cross for arm6hf:
    * apt-get source gcc-6-cross
    * cd gcc-6-cross-24
    * sudo apt-get build-dep gcc-6-cross (make sure that debs from previous step are not overwritten)
    * debuild --set-envvar=CROSS_ARCHS=armhf -us -uc
6. Recompile gcc-6 (source from Debian stretch) on Raspberry Pi to get rid of +rpi1 version mismatch.
7. Install all recompiled packages in a cross container.
