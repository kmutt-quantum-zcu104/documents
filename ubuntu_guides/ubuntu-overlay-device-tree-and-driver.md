# Ubuntu + AXI4-Lite Repetition Decoder

## 1. Overview

วิธีที่ทำสำเร็จจริงบน ZCU104 คือใช้ Ubuntu Linux ติดต่อกับ FPGA Repetition Decoder ผ่าน AXI4-Lite โดยไม่ใช้ Yocto ใน runtime

```text
Ubuntu userspace
      |
      | write()/read()
      v
/dev/repetition_decoder
      |
      v
repetition_decoder.ko
      |
      | ioremap()
      v
AXI4-Lite @ 0xB0000000
      |
      +-- 0x04 : Syndrome input
      |
      +-- 0x0C : Result output
      |
      v
FPGA Repetition Decoder
```

---

## 2. ตรวจสอบระบบ

ตรวจ model:

```bash
cat /proc/device-tree/model
```

ตรวจ kernel:

```bash
uname -r
```

ตรวจ Device Tree:

```bash
ls -l /sys/firmware/devicetree/base
```

ตรวจว่า `/axi` ใช้ address/size cells แบบ 2 cells:

```bash
cat /sys/firmware/devicetree/base/axi/#address-cells | xxd
cat /sys/firmware/devicetree/base/axi/#size-cells | xxd
```

ผลที่ใช้:

```text
#address-cells = 2
#size-cells    = 2
```

---

# 3. FPGA AXI4-Lite

กำหนด address:

```text
Base address = 0xB0000000
Size         = 0x1000
```

Registers:

```text
0xB0000004 = Syndrome input
0xB000000C = Decoder result
```

ทดสอบ hardware โดยตรงด้วย `devmem`:

```bash
sudo busybox devmem 0xB0000004 32 0x1
sudo busybox devmem 0xB000000C 32
```

ผลที่ทดสอบสำเร็จ:

```text
0x00000001
```

---

# 4. Device Tree Overlay

สร้าง directory:

```bash
mkdir -p ~/dev/dt-overlay
cd ~/dev/dt-overlay
```

สร้างไฟล์:

```bash
nano repetition-decoder-overlay.dts
```

ใช้ code:

```dts
/dts-v1/;
/plugin/;

/ {
    fragment@0 {
        target-path = "/axi";

        __overlay__ {
            repetition_decoder@b0000000 {
                compatible = "cpekmutt,repetition-decoder";

                reg = <0x0 0xb0000000
                       0x0 0x1000>;

                status = "okay";
            };
        };
    };
};
```

### สำคัญ

`/axi` ของระบบนี้มี:

```text
#address-cells = 2
#size-cells = 2
```

ดังนั้น `reg` ต้องเป็น:

```dts
reg = <0x0 0xb0000000
       0x0 0x1000>;
```

---

# 5. Compile Device Tree Overlay

```bash
cd ~/dev/dt-overlay

dtc -@ -I dts -O dtb \
    -o repetition-decoder-overlay.dtbo \
    repetition-decoder-overlay.dts
```

ตรวจไฟล์:

```bash
ls -lh repetition-decoder-overlay.dtbo
```

---

# 6. Load Device Tree Overlay

ตรวจว่า kernel มี configfs:

```bash
ls -l /sys/kernel/config
```

ควรมี:

```text
device-tree
```

ตรวจ:

```bash
ls -l /sys/kernel/config/device-tree
```

ควรมี:

```text
overlays
```

สร้าง overlay:

```bash
sudo mkdir /sys/kernel/config/device-tree/overlays/repetition_decoder
```

โหลด:

```bash
sudo sh -c 'cat /home/ubuntu/dev/dt-overlay/repetition-decoder-overlay.dtbo > /sys/kernel/config/device-tree/overlays/repetition_decoder/dtbo'
```

ตรวจ status:

```bash
cat /sys/kernel/config/device-tree/overlays/repetition_decoder/status
```

ผลที่ต้องการ:

```text
applied
```

---

# 7. ตรวจ Platform Device

```bash
ls /sys/bus/platform/devices/ | grep repetition
```

ควรได้:

```text
b0000000.repetition_decoder
```

ตรวจ compatible:

```bash
cat /sys/bus/platform/devices/b0000000.repetition_decoder/of_node/compatible
```

ควรได้:

```text
cpekmutt,repetition-decoder
```

ตรวจ `reg`:

```bash
sudo xxd /sys/bus/platform/devices/b0000000.repetition_decoder/of_node/reg
```

ผล:

```text
00000000: 0000 0000 b000 0000 0000 0000 0000 1000
```

หมายถึง:

```text
address = 0xB0000000
size    = 0x1000
```

---

# 8. Linux Kernel Driver

Directory:

```bash
mkdir -p ~/dev/repetition-driver
cd ~/dev/repetition-driver
```

สร้าง:

```bash
nano repetition_decoder.c
```

ใช้ code:

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/io.h>

#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/device.h>
#include <linux/uaccess.h>

#define DRIVER_NAME "repetition_decoder"
#define DEVICE_NAME "repetition_decoder"

#define AXI_BASE 0xB0000000
#define AXI_SIZE 0x1000

#define SYNDROME_REG 0x04
#define RESULT_REG   0x0C

static void __iomem *base;

static dev_t dev_num;
static struct cdev repetition_cdev;
static struct class *repetition_class;


/*
 * Userspace -> driver -> AXI4-Lite
 *
 * Write syndrome to 0xB0000004
 */
static ssize_t repetition_write(struct file *file,
                                const char __user *buf,
                                size_t count,
                                loff_t *ppos)
{
    u32 syndrome;

    if (count < sizeof(u32))
        return -EINVAL;

    if (copy_from_user(&syndrome,
                       buf,
                       sizeof(u32))) {
        return -EFAULT;
    }

    iowrite32(syndrome,
              base + SYNDROME_REG);

    return sizeof(u32);
}


/*
 * AXI4-Lite -> driver -> userspace
 *
 * Read result from 0xB000000C
 */
static ssize_t repetition_read(struct file *file,
                               char __user *buf,
                               size_t count,
                               loff_t *ppos)
{
    u32 result;

    if (count < sizeof(u32))
        return -EINVAL;

    result = ioread32(base + RESULT_REG);

    if (copy_to_user(buf,
                     &result,
                     sizeof(u32))) {
        return -EFAULT;
    }

    return sizeof(u32);
}


static const struct file_operations repetition_fops = {
    .owner = THIS_MODULE,
    .read  = repetition_read,
    .write = repetition_write,
};


static int repetition_probe(struct platform_device *pdev)
{
    int ret;

    dev_info(&pdev->dev,
             "repetition decoder probe\n");

    /*
     * Use direct ioremap().
     *
     * We do not use:
     *
     * devm_platform_ioremap_resource()
     *
     * because the runtime Device Tree overlay did not
     * provide a usable platform resource for this setup.
     */
    base = ioremap(AXI_BASE,
                   AXI_SIZE);

    if (!base) {
        dev_err(&pdev->dev,
                "failed to ioremap AXI registers at 0x%08X\n",
                AXI_BASE);
        return -ENOMEM;
    }

    dev_info(&pdev->dev,
             "AXI registers mapped at 0x%08X\n",
             AXI_BASE);

    ret = alloc_chrdev_region(&dev_num,
                              0,
                              1,
                              DEVICE_NAME);

    if (ret < 0)
        goto unmap;

    cdev_init(&repetition_cdev,
              &repetition_fops);

    repetition_cdev.owner = THIS_MODULE;

    ret = cdev_add(&repetition_cdev,
                   dev_num,
                   1);

    if (ret < 0)
        goto unregister_chrdev;

    repetition_class = class_create(THIS_MODULE,
                                    DEVICE_NAME);

    if (IS_ERR(repetition_class)) {
        ret = PTR_ERR(repetition_class);
        goto del_cdev;
    }

    if (IS_ERR(device_create(repetition_class,
                             NULL,
                             dev_num,
                             NULL,
                             DEVICE_NAME))) {
        ret = -EINVAL;
        goto destroy_class;
    }

    dev_info(&pdev->dev,
             "repetition decoder registered\n");

    dev_info(&pdev->dev,
             "device: /dev/%s\n",
             DEVICE_NAME);

    return 0;


destroy_class:
    class_destroy(repetition_class);

del_cdev:
    cdev_del(&repetition_cdev);

unregister_chrdev:
    unregister_chrdev_region(dev_num, 1);

unmap:
    iounmap(base);
    base = NULL;

    return ret;
}


static int repetition_remove(struct platform_device *pdev)
{
    device_destroy(repetition_class,
                   dev_num);

    class_destroy(repetition_class);

    cdev_del(&repetition_cdev);

    unregister_chrdev_region(dev_num,
                             1);

    if (base) {
        iounmap(base);
        base = NULL;
    }

    dev_info(&pdev->dev,
             "repetition decoder removed\n");

    return 0;
}


static const struct of_device_id repetition_of_match[] = {
    {
        .compatible = "cpekmutt,repetition-decoder",
    },
    { }
};

MODULE_DEVICE_TABLE(of,
                    repetition_of_match);


static struct platform_driver repetition_driver = {
    .probe  = repetition_probe,
    .remove = repetition_remove,

    .driver = {
        .name = DRIVER_NAME,
        .of_match_table = repetition_of_match,
    },
};


module_platform_driver(repetition_driver);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("CPE KMUTT");
MODULE_DESCRIPTION("AXI4-Lite Repetition Decoder Driver");
MODULE_VERSION("1.0");
```

---

# 9. Makefile

สร้าง:

```bash
nano Makefile
```

ใช้:

```makefile
obj-m += repetition_decoder.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

all:
	make -C $(KDIR) M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean
```

---

# 10. Compile Driver

```bash
cd ~/dev/repetition-driver

make clean
make
```

ตรวจ:

```bash
ls -lh repetition_decoder.ko
```

---

# 11. Load Driver

```bash
sudo insmod ~/dev/repetition-driver/repetition_decoder.ko
```

ตรวจ log:

```bash
sudo dmesg | grep repetition_decoder
```

ผลที่สำเร็จ:

```text
repetition_decoder b0000000.repetition_decoder:
repetition decoder probe

repetition_decoder b0000000.repetition_decoder:
AXI registers mapped at 0xB0000000

repetition_decoder b0000000.repetition_decoder:
repetition decoder registered

repetition_decoder b0000000.repetition_decoder:
device: /dev/repetition_decoder
```

ตรวจ device:

```bash
ls -l /dev/repetition_decoder
```

---

# 12. Userspace Test

สร้าง:

```bash
cd ~/dev
nano test.c
```

ใช้ code:

```c
#include <stdio.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>

int main(void)
{
    int fd;
    uint32_t syndrome;
    uint32_t result;

    printf("=== Repetition Decoder Test ===\n");

    fd = open("/dev/repetition_decoder", O_RDWR);

    if (fd < 0) {
        printf("Failed to open device: %s\n",
               strerror(errno));
        return 1;
    }

    printf("Device opened successfully.\n");

    syndrome = 0x1;

    printf("Syndrome : 0x%X (%u)\n",
           syndrome,
           syndrome);

    if (write(fd,
              &syndrome,
              sizeof(syndrome)) != sizeof(syndrome)) {

        printf("Write failed: %s\n",
               strerror(errno));

        close(fd);
        return 1;
    }

    printf("Syndrome written successfully.\n");

    if (read(fd,
             &result,
             sizeof(result)) != sizeof(result)) {

        printf("Read failed: %s\n",
               strerror(errno));

        close(fd);
        return 1;
    }

    printf("Result   : 0x%X (%u)\n",
           result,
           result);

    close(fd);

    printf("Test completed.\n");

    return 0;
}
```

Compile:

```bash
gcc test.c -o test
```

Run:

```bash
sudo ./test
```

ผลที่ทดสอบสำเร็จ:

```text
=== Repetition Decoder Test ===
Device opened successfully.
Syndrome : 0x1 (1)
Syndrome written successfully.
Result   : 0x1 (1)
Test completed.
```

---

# 13. Unload Driver

Unload:

```bash
sudo rmmod repetition_decoder
```

ตรวจ:

```bash
lsmod | grep repetition_decoder
```

ถ้าไม่มี output แปลว่า driver ถูก unload แล้ว

ตรวจ `/dev`:

```bash
ls -l /dev/repetition_decoder
```

โดยปกติ device จะหายไปหลัง driver ถูก unload

Load กลับ:

```bash
sudo insmod ~/dev/repetition-driver/repetition_decoder.ko
```

---

# 14. ตรวจ Driver Log

```bash
sudo dmesg | grep repetition_decoder
```

ถ้า driver ทำงานสำเร็จควรมี:

```text
repetition decoder probe
AXI registers mapped at 0xB0000000
repetition decoder registered
device: /dev/repetition_decoder
```

---

# 15. ตรวจ Overlay

ตรวจ:

```bash
cat /sys/kernel/config/device-tree/overlays/repetition_decoder/status
```

ต้องได้:

```text
applied
```

ตรวจ platform device:

```bash
ls /sys/bus/platform/devices/ | grep repetition
```

ต้องได้:

```text
b0000000.repetition_decoder
```

---

# 16. สรุปสิ่งที่ทำสำเร็จ

```text
[FPGA]
Repetition Decoder AXI4-Lite
Base = 0xB0000000
        |
        v
[Ubuntu]
        |
        +-- Device Tree Overlay       OK
        |
        +-- Platform Device          OK
        |
        +-- repetition_decoder.ko    OK
        |
        +-- ioremap()                OK
        |
        +-- /dev/repetition_decoder   OK
        |
        +-- test.c                    OK
```

การทดสอบสองวิธีให้ผลตรงกัน:

### วิธีที่ 1: devmem

```bash
sudo busybox devmem 0xB0000004 32 0x1
sudo busybox devmem 0xB000000C 32
```

ผล:

```text
0x00000001
```

### วิธีที่ 2: Linux Driver

```bash
sudo ./test
```

ผล:

```text
Syndrome : 0x1 (1)
Result   : 0x1 (1)
```

ดังนั้น flow ที่พิสูจน์แล้วคือ:

```text
Userspace
   |
   | write()
   v
/dev/repetition_decoder
   |
   v
Linux Kernel Driver
   |
   | iowrite32()
   v
0xB0000004
   |
   v
AXI4-Lite
   |
   v
FPGA Repetition Decoder
   |
   | result
   v
0xB000000C
   |
   | ioread32()
   v
Linux Kernel Driver
   |
   v
Userspace read()
```

## วิธีที่ไม่ได้ใช้ใน final setup

เราไม่ใช้:

```c
devm_platform_ioremap_resource(pdev, 0);
```

เพราะ runtime Device Tree Overlay setup นี้ทำให้เกิด:

```text
invalid resource
failed to map AXI registers
error -22
```

วิธีที่สำเร็จคือ:

```c
base = ioremap(0xB0000000, 0x1000);
```

และทดสอบใช้งานจริงแล้ว
