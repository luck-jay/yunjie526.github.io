# 1、驱动程序编写

1. 点灯程序

   ```assembly
   .global _start /*全局标号*/
   
   _start:
           /* 使能所有外设时钟 */
           ldr r0, =0x020c4068 /* CCGR0 */
           ldr r1, =0xffffffff /* 要向CCGR0写入的数据 */
           str r1, [r0]        /* 将0XFFFFFFFF写入到CCGR0中 */
   
           ldr r0, =0x020c406c /* CCGR1 */
           str r1, [r0]        /* 将0XFFFFFFFF写入到CCGR1中 */
   
           ldr r0, =0x020c4070 /* CCGR2 */
           str r1, [r0]        /* 将0XFFFFFFFF写入到CCGR2中 */
   
           ldr r0, =0x020c4074 /* CCGR3 */
           str r1, [r0]        /* 将0XFFFFFFFF写入到CCGR3中 */
   
           ldr r0, =0x020c4078 /* CCGR4 */
           str r1, [r0]        /* 将0XFFFFFFFF写入到CCGR4中 */
   
           ldr r0, =0x020c407c /* CCGR5 */
           str r1, [r0]        /* 将0XFFFFFFFF写入到CCGR5中 */
   
           ldr r0, =0x020c4080 /* CCGR6 */
           str r1, [r0]        /* 将0XFFFFFFFF写入到CCGR6中 */
   
           /*
            * 配置GPIO1_IO03 PIN的复用为GPIO
            * IOMUXC_SW_MUX_CTL_PAD_PIO1_IO03=5
   		 * IOMUXC_SW_MUX_CTL_PAD_PIO1_IO03寄存器地址0x020E0068
            */
           ldr r0, =0x020E0068 /* IOMUXC_SW_MUX_CTL_PAD_PIO1_IO03 */
           ldr r1, =0x5        /* 要写入的数据 */
           str r1, [r0]        /* 将0XFFFFFFFF写入到CCGR0中 */
           /*
            * 配置GPIO1_IO03的电器属性
            * IOMUXC_SW_PAD_CTL_PAD_PIO1_IO03
            * IOMUXC_SW_PAD_CTL_PAD_PIO1_IO03地址为0x020E02F4
            * bit0:     0 低速率
            * bit5:3:   110 R0/6驱动能力
            * bit7:6:   10  100MHz速度
            * bit11:    0 关闭开路输出
            * bit12:    1 使能pull/kepper
            * bit13:    0 kepper
            * bit15:14: 00 100k下拉
            * bit16:    0 关闭Hyst
            */
           ldr r0, =0x020e02f4
           ldr r1, =0x10b0
           str r1, [r0]
           /**
            * 设置GPIO
            * 设置GPIO1_GDIR寄存器，设置GPIO1_GDIR寄存器，设置GPIO1_GPIO03为输出
            * GPIO1_GDIR 地址是0x0209c004
            * 设置GPIO1_GDIR寄存器bit3为1,
            */
           ldr r0, =0x0209c004
           ldr r1, =0x8
           str r1, [r0]
   
           /**
            * 打开LED, 也就是设置GPIO_IO03为0
            * GPIO1_DR寄存器地址为0x0209c000
            */
           ldr r0, =0x0209c000
           ldr r1, 0
           str r1, [r0]
   
   loop:
           b loop
   ```

2. 编译程序

   1. 将.c .s程序编译为.o

      ```shell
      arm-none-eabi-gcc -g -c leds.s -o leds.o
      ```

   2. 将所有的.o文件链接为elf格式的可执行文件

      ```shell
       $ arm-none-eabi-ld -Ttext 0x87800000 leds.o -o leds.elf     # 链接成elf文件
      ```

      > 链接：
      >
      > 链接就是将所有的.o文件链接在一起，并且链接到指定的地方。本实验链接的时候要指定链接的起始地址。链接起始地址就是代码运行的起始地址。
      >
      > 对于6ULL来说，链接起始地址应该指向RAM地址，RAM分为内部RAM和外部RAM也就是DDR。6ULL内部RAM地址范围**0x90000000～0x91FFFFFF**，也可以放到外部DDR中，对于I.MX6U-ALPHA开发板，512M字节DDR版本的核心板，DDR范围就是**0x80000000~0x9FFFFFFF**，对于256MB的DDR来说，那就是0x80000000~0x8FFFFFFF。

   3. 将elf文件转化成bin文件

      ```shell
       $ arm-none-eabi-objcopy -O binary -S -g leds.elf leds.bin   # 将elf文件转成bin文件
      ```

   4. 将elf文件转为汇编、反汇编

      ```shell
      $ arm-none-eabi-objdump -D leds.elf > led.dis
      ```

3. 烧写程序

   1. 调整拨码开关
   
      ![调整拨码开关](/home/pyj/git/yunjie526.github.io/linux/拨码开关SD卡启动.png)

