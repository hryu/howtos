# UEFI and Trusted Firmware
## UEFI

    $ export WORKSPACE=~/uefi
    $ mkdir -p $WORKSPACE
    $ cd $WORKSPACE

    $ git clone https://github.com/tianocore/edk2.git
    $ git clone https://github.com/tianocore/edk2-platforms.git
    $ git clone https://github.com/tianocore/edk2-non-osi.git

    $ export PACKAGES_PATH=$PWD/edk2:$PWD/edk2-platforms:$PWD/edk2-non-osi

    $ . edk2/edksetup.sh
    $ make -C edk2/BaseTools

    $ export GCC5_AARCH64_PREFIX=aarch64-linux-gnu-
    $ build -a AARCH64 -t GCC5 -p edk2-platforms/Platform/ARM/VExpressPkg/ArmVExpress-FVP-AArch64.dsc

*Build/ArmVExpress-FVP-AArch64/DEBUG_GCC5/FV/FVP_AARCH64_EFI.fd*

> 참고
> https://github.com/tianocore/edk2-platforms/blob/master/Readme.md
