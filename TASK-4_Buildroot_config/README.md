
#Quy trình cài đặt Buidroot cho BBB

1.Tổng quan
Buildroot là framework mã nguồn mở giúp tự động hoá quá trình tạo ra:

    Root filesystem (rootfs)
    Toolchain
    Kernel
    Bootloader

Quy trình làm việc cơ bản gồm 4 bước:

    Toolchain – tạo bộ cross-compiler cho kiến trúc mục tiêu (ARM, RISC-V, v.v.)
    Kernel – build kernel image (zImage, dtb, modules)
    RootFS – tạo root filesystem (ext4, cpio, tar)
    Image – sinh ra file image hoàn chỉnh (sdcard.img, rootfs.ext4, ...)

