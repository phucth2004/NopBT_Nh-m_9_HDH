### Clone mã nguồn Buildroot
Vì chúng ta sẽ thực hiện phát triển với Buildroot, hãy clone mã nguồn Buildroot từ kho Git chính thức:
```bash
git clone https://gitlab.com/buildroot.org/buildroot.git
```
Sau khi tải xong, truy cập vào thư mục Buildroot mới được tạo:
```bash
cd buildroot
```
Chúng ta sẽ tạo một nhánh (branch) mới dựa trên phiên bản Buildroot 2025.02.6, đây là bản đã được kiểm thử cho khóa huấn luyện này
```bash
git checkout -b bootlin 2025.02.6
```
### Cấu hình Buildroot (Configuring Buildroot)

Nếu bạn mở thư mục configs/, bạn sẽ thấy có tệp beaglebone_defconfig, đây là một tệp cấu hình Buildroot sẵn sàng để sử dụng nhằm xây dựng hệ thống cho BeagleBone Black Wireless.
Tuy nhiên, vì chúng ta đang học cách làm việc với Buildroot, nên sẽ bắt đầu tạo một cấu hình mới hoàn toàn từ đầu.

### Bắt đầu công cụ cấu hình Buildroot
Chạy lệnh sau để mở giao diện cấu hình:
```bash
make menuconfig
```


Sau khi mở menuconfig , giao diện sẽ như sau:
![menuconfig](image-1.png)

Đến đây ta sẽ đi cấu hình từng mục phù hợp với board BeaglbonBlack nhé!!!

 Thực hiện cấu hình từng bước:

### 1. Target Options:
- Chúng ta biết rằng BeagleBone Black Wireless là nền tảng ARM (little endian), vì vậy chọn:
```bash
Target Architecture → ARM (little endian)
```
- Theo trang chính thức https://beagleboard.org/BLACK
, thiết bị này dùng Texas Instruments AM335x, dựa trên ARM Cortex-A8.
```bash
Target Architecture Variant → cortex-A8
```
- Có hai giao diện nhị phân (ABI) cho ARM: EABI và EABIhf.
Nếu không cần tương thích ngược, hãy chọn EABIhf vì hiệu suất tốt hơn:
```bash
Target ABI → EABIhf
```
#### Các giá trị còn lại giữ mặc định:

- **ELF là định dạng nhị phân**

- **VFPv3-D16 cho đơn vị dấu chấm động (FPU)**

- **ARM instruction set cho bộ lệnh (thay vì Thumb-2)**
![targetoption](image-2.png)

### 2. Build Options

- Không cần thay đổi gì, nhưng bạn có thể tham khảo và đọc phần mô tả (help) cho từng tùy chọn để hiểu rõ hơn.

### 3.Toolchain

- Theo mặc định, Buildroot sẽ tự xây dựng toolchain, nhưng việc này mất nhiều thời gian.
→ Chúng ta sẽ dùng external toolchain (có sẵn).
```bash
Toolchain type → External toolchain
```

Chọn:
```bash
Toolchain → Buidroot toolchains ( Buidroot sẽ giúp chúng ta tự động tạo ra một bộ toolchain( gcc, ldd, gdb) để sử dụng sau này
```

Hệ thống sẽ tự động chọn biến thể:
```bash
armv7-eabihf glibc bleeding-edge 2024.05-1
```
Các tùy chọn khác như C library , ... giữ nguyên.

Đây là lựa chọn phù hợp cho BeagleBone Black.


### 4. System Configuration

Trong hệ thống cơ bản này, chưa cần nhiều tùy chỉnh.
Bạn có thể đặt:

- System hostname (tên hệ thống)

- System banner (thông báo khi khởi động)

- Root password (mật khẩu root)
### 5.Kernel

Kích hoạt nhân Linux:
```bash
[*] Linux kernel
```


Sử dụng phiên bản cụ thể để đảm bảo tái lập build:
```bash
Kernel version → Custom version
Custom version → 6.12.47
```

Chọn cấu hình sẵn (defconfig) trong mã nguồn kernel:
```bash
Defconfig name → omap2plus
```

(Vì BeagleBone Black dựa trên dòng TI AM335x, được hỗ trợ trong OMAP2/3/4.)

Giữ định dạng nhị phân mặc định:
```bash
Kernel binary format → zImage
```

Bật Device Tree Blob (DTB):
```bash
[*] Build a Device Tree Blob (DTB)
In-tree Device Tree Source file names → ti/omap/am335x-boneblack
```

Kích hoạt OpenSSL cho host:
```bash
[*] Needs host OpenSSL
```

Hiển thị dấu * là đã bật nhé, nhấn dấu space để bật nhé.

### 6. Target Packages

- Menu này chứa hơn 3000+ gói Buildroot có thể chọn cài.

- Với hệ thống cơ bản, chỉ cần BusyBox (đã bật mặc định).

- Bạn có thể khám phá thêm các gói khác trong các bài lab sau.


Ở đây bạn có thể bật các tool cần dùng cho kernel ví dụ như mosquitto (MQTT) hay là ssh ,.... chẳng hạn.

### 7 .Filesystem Images

Giữ tùy chọn mặc định:
```bash
[*] tar the root filesystem
```
Việc flash root filesystem vào thẻ SD sẽ được thực hiện ở bước sau.

### 8. Bootloaders

Chọn U-Boot, bootloader phổ biến nhất cho ARM:
```bash
[*] U-Boot
```

Dùng hệ thống build kiểu Kconfig (phiên bản mới của U-Boot):
```bash
Build system → Kconfig
```

Phiên bản:
```bash
Custom version → 2024.04
```

Cấu hình board:
```bash
Board defconfig → am335x_evm
Custom make options → DEVICE_TREE=am335x-boneblack-wireless
```

U-Boot có 2 phần:
```bash
MLO: Bootloader giai đoạn đầu (SPL)

u-boot.img: Bootloader chính
```
Cấu hình tương ứng:
```bash
U-Boot binary format → u-boot.img
[*] Install U-Boot SPL binary image
U-Boot SPL binary image name → MLO
```

Khi đã thiết lập xong, bạn có thể lưu cấu hình lại và thoát.

##  Biên dịch hệ thống (Building)

Bạn có thể đơn giản chạy lệnh:
```bash
make -j4 
```
4 ở đây là số luồng của máy ảo giúp ta build nhanh hơn đó.


Tuy nhiên, để lưu lại toàn bộ log quá trình build (bao gồm cả đầu ra chuẩn và lỗi) vào một tệp đồng thời vẫn hiển thị trên terminal, ta sẽ dùng lệnh:
```bash
make 2>&1 | tee build.log
```

Lệnh này sẽ:
```text
2>&1 gộp luồng lỗi (stderr) vào luồng chuẩn (stdout),

tee vừa ghi log vào tệp build.log, vừa in ra màn hình để bạn theo dõi tiến trình biên dịch.
```

Trong khi quá trình biên dịch đang diễn ra (sẽ mất khá nhiều thời gian tùy vào cấu hình hệ thống và tốc độ mạng), bạn có thể chuẩn bị các bước tiếp theo để kiểm tra kết quả build trên thiết bị BeagleBone Black của mình.

### Quá trình build Buildroot sẽ tốn kha khá thời gian nên việc của chúng ta là chờ đợi , như máy mình là đợi tầm 45 phút là build xong.
 Sau khi build xong thì thứ ta quan tâm là distro các file cần thiết để boot lên Baeglebon, thường thì chúng sẽ nằm ở đây, các bạn gõ lệnh sau:
 ```bash
 ls -l output/images
 ```
 Các file bao gồm như sau:
 ```text
 buildroot/
├── output/
│   ├── build/               # Mã nguồn và file tạm của từng package (Linux kernel, BusyBox, U-Boot,…)
│   ├── host/                # Các công cụ Buildroot build để chạy trên máy host (toolchain, genimage,…)
│   ├── images/              #  Thư mục chứa toàn bộ file đầu ra cuối cùng
│   │   ├── rootfs.tar       # Root filesystem dạng tar (theo cấu hình bạn chọn)
│   │   ├── zImage           # Kernel image (nếu bạn bật build Linux kernel)
│   │   ├── am335x-boneblack-wireless.dtb  # Device Tree Blob
│   │   ├── MLO              # Bootloader SPL (giai đoạn 1)
│   │   └── u-boot.img       # Bootloader chính (giai đoạn 2)
│   ├── staging/             # Root filesystem tạm để cài đặt các package
│   └── target/              # Cây thư mục rootfs thật (chưa đóng gói, dạng copy ra SD nếu cần)
└── configs/
```
### Giải thích chi tiết
| Thư mục           | Vai trò                                                                                   | Ghi chú                                        |
| ----------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------- |
| `output/build/`   | Mỗi package (kernel, uboot, busybox, openssl, …) sẽ được giải nén và build tại đây        | Có thể xem log build chi tiết của từng package |
| `output/host/`    | Chứa các công cụ build chạy trên máy host (ví dụ `host-gcc`, `mkimage`, `genext2fs`, …)   | Được Buildroot tự động quản lý                 |
| `output/images/`  |  **Nơi chứa các file cuối cùng bạn sẽ flash lên thẻ SD hoặc dùng để khởi động thiết bị** | Đây là thư mục bạn quan tâm nhất               |
| `output/staging/` | Dạng rootfs trung gian — chưa dùng trực tiếp                                              | Buildroot dùng để copy vào `target/`           |
| `output/target/`  | Root filesystem đầy đủ của thiết bị (dạng thư mục)                                        | Có thể `chroot` hoặc tạo image từ đây          |


Xem danh sách file đầu ra:
```bash
ls output/images/
```

Kiểm tra kích thước các thành phần:
```bash
du -h output/images/
```

Giải nén root filesystem để xem bên trong:
```bash
mkdir rootfs
sudo tar xf output/images/rootfs.tar -C rootfs/
ls rootfs/
```
Sau khi đã có đầy đủ các file rồi thì ta sẽ tiến hành copy vào thẻ nhớ sd card để boot lên board nhé, các bạn chuẩn bị một thẻ nhớ từ  8GB trở lên nhé.

##Chuẩn bị thẻ SD (Prepare the SD card)

Để hệ thống chạy được trên BeagleBone Black, chúng ta cần chuẩn bị thẻ SD với hai phân vùng riêng biệt:

** Cấu trúc phân vùng cần có**

- Phân vùng 1 – Bootloader (FAT32):

Dùng để chứa các tệp khởi động:
```bash
MLO (U-Boot SPL – bootloader giai đoạn 1)

u-boot.img (bootloader chính)

zImage (Linux kernel)

am335x-boneblack-wireless.dtb (Device Tree)
```
Phân vùng này phải tuân thủ quy định của SoC AM335x, nên định dạng là FAT32.

- Phân vùng 2 – Root filesystem (ext4):

    Dùng để chứa toàn bộ root filesystem của hệ thống Linux.

Sử dụng định dạng ext4.

###  Xác định thiết bị thẻ SD

Đầu tiên, hãy xác định tên thiết bị mà hệ thống gán cho thẻ SD của bạn bằng lệnh:
```bash
cat /proc/partitions
```

Nếu bạn dùng đầu đọc thẻ SD tích hợp trên laptop, thường sẽ thấy tên như:
```bash
/dev/mmcblk0
```

Nếu bạn dùng đầu đọc SD qua USB, nó sẽ hiển thị dạng:

/dev/sdX  (ví dụ: /dev/sdb, /dev/sdc, ...)


###  Quy ước tên phân vùng

Nếu thẻ SD của bạn là /dev/mmcblk0,
thì các phân vùng bên trong sẽ là:

- **/dev/mmcblk0p1   → phân vùng 1 (boot)**
- **/dev/mmcblk0p2   → phân vùng 2 (rootfs)**

###  Các bước format thẻ SD
1. Tháo (unmount) tất cả phân vùng của thẻ SD

Ubuntu thường tự động mount các phân vùng, bạn cần tháo chúng ra trước:
```bash
sudo umount /dev/mmcblk0p*
```
2. Xóa sạch phần đầu thẻ SD

Điều này đảm bảo các phân vùng cũ không còn được hệ thống nhận nhầm:
```bash
sudo dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=16
```
3. Tạo hai phân vùng mới

Sử dụng công cụ cfdisk:
```bash
sudo cfdisk /dev/mmcblk0
```

Chọn dos làm kiểu bảng phân vùng (dos partition table)

Tạo phân vùng thứ nhất:
```bash
Dung lượng: 128 MB

Loại: Primary

Kiểu (Type): e (W95 FAT16)

Đánh dấu bootable
```
Tạo phân vùng thứ hai:
```bash
Dùng toàn bộ dung lượng còn lại

Loại: Primary

Kiểu (Type): 83 (Linux)

Thoát và lưu thay đổi trong cfdisk
```
4. Định dạng phân vùng boot (FAT32)
```bash
sudo mkfs.vfat -a -F 32 -n boot /dev/mmcblk0p1
```

Tham số:
```text
-a → kích hoạt auto-align

-F 32 → định dạng FAT32

-n boot → đặt nhãn (label) là boot
```

5. Định dạng phân vùng rootfs (ext4)
```bash
sudo mkfs.ext4 -L rootfs -E nodiscard /dev/mmcblk0p2
```

Giải thích:

-L rootfs: đặt tên volume là rootfs

-E nodiscard: tắt chế độ kiểm tra và loại bỏ block xấu, giúp tăng tốc format đáng kể (vì SD card thường không có bad block thật).

6. Kiểm tra kết quả

Sau khi hoàn tất, tháo thẻ SD ra và cắm lại — Ubuntu sẽ tự động mount hai phân vùng:
```bash
/media/$USER/boot
/media/$USER/rootfs
```

