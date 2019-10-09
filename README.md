## Recompiled Debian Packages for Raspbian Cross Compilation

Draft: How the packages got recompiled:

### Raspbian Stretch

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

### Raspbian Buster

1. Upgrade the edi-raspbian setup to buster, within the role "repositories" disable the cross compiler
repository and raspbian_buster.list and within "development_tools" deactivate the installation of armhf packages and cross compilers
(set gcc_bootstrap_environment to True).
Important: The setup will pretend to be a Raspbian (see /usr/lib/os-release).
The command "lsb_release -is" should show the result "Raspbian".
The debian/rules2 file of gcc-8 will set the CPU options accordingly!
2. Execute the command "sudo edi -v lxc configure raspbian-buster-cross buster-cross.yml".
3. Make sure that no deb and deb-src entries in /etc/apt/sources.list.d/ point to Raspbian.
4. Recompile cross-toolchain-base for arm6hf within its own subdirectory:
    * Make sure that there are no cross compiler packages nor armhf packages installed.
    * apt-get source cross-toolchain-base
    * cd cross-toolchain-base-33
    * sudo apt-get build-dep cross-toolchain-base
    * debuild --set-envvar=CROSS_ARCHS=armhf -us -uc
5. Install debs from previous step and also collect *.deb from the cross-toolchain-base-XX sub folder.
    * sudo dpkg -i ../*.deb
6. Recompile gcc-8-cross for arm6hf:
    * apt-get source gcc-8-cross
    * cd gcc-8-cross-26
    * sudo apt-get build-dep gcc-8-cross (make sure that debs from previous step are not overwritten)
    * debuild --set-envvar=CROSS_ARCHS=armhf -us -uc
7. Recompile gcc-8 (source from Debian stretch) on Raspberry Pi to get rid of +rpi1 version mismatch.
Including testing this takes about seven days.
    * wget -qO - https://ftp-master.debian.org/keys/archive-key-9.asc | sudo apt-key add -
    * add buster_src.list from Debian.
    * apt-get source gcc-8
    * cd gcc-8-8.3.0
    * sudo apt-get build-dep gcc-8
    * sudo apt-get install devscripts build-essential lintian
    * DEB_BUILD_OPTIONS='parallel=1' debuild -us -uc
    * Fetch the rebuilt libraries plus gcc-8-base_*_armhf.deb.
8. Upload all recompiled debs to get-edi/raspbian-cross-compiler
9. Uncomment the tasks that yet got excluded in step 1 (set gcc_bootstrap_environment to False).
10. Use edi-raspbian according to step 2 to generate a cross compilation container.
11. Test the cross compilation container according to https://www.get-edi.io/Cross-Compiling-for-Raspbian/ and run the
resulting binaries on a Pi 1.
