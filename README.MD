MỤC LỤC
```
|PHẦN   | NỘI DUNG
-----------------------------
|01     | GIỚI THIỆU
|02     | CÀI OS
```


## PHẦN 01: GIỚI THIỆU

Thông số kĩ thuật của RASPI 3B
Hình ảnh: <br>
![alt text](image.png)
Thông số: <br>
![alt text](image-2.png)

Pinout của RASPI 3B <br>
![alt text](image-1.png)

Tính năng <br>
```cpp
- Broadcom BCM2837 64bit ARM Cortex-A53 Quad Core - Processor SoC chạy ở @ 1.2GHz
- 1 GB RAM
- 4 x Cổng USB2.0 hỗ trợ dòng điện ra đến 1.2A
- 40-chân GPIO có thể mở rộng
- Xuất Video/Audio thông qua cổng 3.5mm 4 cực, HDMI, CSI camera, or Raw LCD (DSI)
- Lưu trữ: MicroSD
- 10/100 Ethernet (RJ45)
- Onboard BCM43143 WiFi
- Bluetooth Low Energy (BLE) on board
- Low-Level Peripherals:
 + 27 x GPIO
 + UART
 + I2C bus
 + SPI bus với 2 chip
 + +3.3V
 + +5V
 + Ground
- Yêu cầu nguồn điện: 5V @ 2.4 A qua cổng MicroUSB
- Hỗ trợ Raspbian, Windows 10 IoT Core, OpenELEC, OSMC, Pidora, Arch Linux, RISC OS, và nhiều hơn thế nữa!
- Kích thước: 85mm x 56mm x 17mm
```

## PHẦN 02: CÀI OS CHO PI
### Cách 01:
```
Có màn hình
```
```
Nếu có màn hình thì mọi chuyện rất đơn giản
B1: Tải file image
B2: Install
```
### Cách 02:
```
Không có màn hình
```
```
Nếu không có màn hình thì theo hướng dẫn tại 2 video này là được:
Thiết bị cần:
+ USB TO TTL CH340
+ Config.txt
```
Video 1: https://youtu.be/G0X2rfaq2ac?si=KLWJzZsvNzv6CizV <br>
Video 2: https://youtu.be/3C7O6ABCVLo?si=fOzwM1jXLSyOi_dD 

## PHẦN 03: CMD cơ bản VIM
Setup
```cpp
sudo apt update
sudo apt-get install vim
```
```
Esc +
:wq: Lưu và thoát
:set number: Hiện dòng
:<LINE_NUMBER>: Con trỏ trỏ đến dòng <LINE_NUMBER>
```

## PHẦN 04: KERNEL MODULE BASIC
Thêm header
```cpp
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r)
```

Module đầu tiên
```cpp
#include <linux/module.h>   
#include <linux/kernel.h>   
#include <linux/init.h>     

//(insmod)
static int __init hello_init(void) {
    printk(KERN_INFO "Hello, world!\n");
    return 0;
}

//(rmmod)
static void __exit hello_exit(void) {
    printk(KERN_INFO "Goodbye, world!\n");
}

module_init(hello_init);
module_exit(hello_exit);

// Infor
MODULE_LICENSE("GPL");
MODULE_AUTHOR("HOCCUNGET");
MODULE_DESCRIPTION("HELLO WORLD");

```

Build module
```cpp
obj-m += hello.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

```cpp
sudo insmod hello.ko   : Nạp module
sudo rmmod hello       : Gỡ module
dmesg | tail           : Đọc Log
```

## PHẦN 05 CHARACTER DEVICE DRIVER

```
Dữ liệu được đọc tuần tự theo từng byte một (chuột, bàn phím, serial,…)
Ví dụ:
/dev/ttyS0 (serial port)
/dev/input/ (chuột, bàn phím)
```
![alt text](image-3.png)
Bản chất thì nó là
```
Cầu nối giữa hardware và user-space
Cung cấp giao diện file để ứng dụng(user space) truy cập thiết bị(hardware)
```

Các chương trình trên tầng user có thể truy cập vào thiết bị tương ứng với các file nằm trong thư mục /dev và thực hiện các system call thích hợp trên thiết bị, các lệnh này sau đó sẽ được chuyển tới driver đã được liên kết với device file của thiết bị đó.

Tóm lại là:
```
Viết một character device driver → xuất hiện file /dev/temp_sensor
```

Ví dụ thực tế
```
Bạn muốn đọc nhiệt độ bằng một phần cứng A qua I2C
```
```cpp
Viết một Character device driver
--> Khi đó sẽ xuất hiện /dev/temp_sensor

+ Driver mình viết: sẽ lo việc giao tiếp với phần cứng
+ App thì chỉ cần: int fd = open("/dev/temp_sensor", O_RDONLY);

```

Chốt lại
```
Một driver trong Linux kernel dùng để ánh xạ thiết bị (hardware/ảo) 
thành file trong /dev/, cho phép user-space giao tiếp với thiết 
bị bằng thao tác file (read/write)
```