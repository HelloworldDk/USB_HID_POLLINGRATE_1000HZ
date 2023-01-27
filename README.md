# USB_HID_POLLINGRATE_1000HZ
Tell you How To Make a 1000HZ USB HID Device With STM32 or Other USB Board  
教您使用STM32或其他支持USB的板子，制作1000HZ回报率（刷新率）的USB HID 设备！！！

# Chinese
我开发了一款USB 键盘游戏控制器，采用STM32制作，直接通过CubeMX生成Custom HID Devide，然后写了个107Key的纯NKEYROLLBACK（真全键无冲）键盘，当时测得刷新率只有120hz左右，非常头疼，查ST论坛，有人说HID规范要求必须间隔10MS，所以是正常现象，但是手上有一个USB手柄，刷新率确实巨高，让我非常羡慕，折腾一个月，各种上网查，然后终于知道问题出在哪里了！！！！

如何操作，让慢速的HID变成1000hz的？

1. 在CubeMX USB中间件的设置中（推荐，重生成不会还原） 或在USBD_CONF.c（不推荐，重新生成会还原），把CUSTOM_HID_FS_BINTERVAL改成 0x1（每次报告最短间隔1ms），关于0x0，电脑识别应该是跟0x1一样，完全为0的间隔显然是不现实的。  
2. 考虑每个包间隔1ms，但是，你的每次发送包大小不一定是满足你整个报告的大小！！！通过软件分析ConfigDesc，可以发现自动生成的程序，Endpoint address 0x1, Input, Interrupt, max packet size: 2 bytes, update interval: 2 1-millisecond frames，每包2字节，像NKRO键盘一个包至少107个bit的，显然一个包发不完，这样多个包一个报告，会浪费很多1ms，按照USB-DeviceFS规定，一个包最大64字节，于是我们直接把usbd_customhid.h中#define CUSTOM_HID_EPIN_SIZE                 0x02U改成0x40U，一个包发完整个报告，绝不浪费多个包时间发送  
3.打开usbHID抓包软件，可以发现你的设备已经达到1000HZ！！！！！
4.如果改完之后发现丢包情况或者卡包情况，请参阅文章底部Also Read（也请阅读）的第一条链接，您使用的可能是旧版未修复的HAL库（现在的库大概已经没有那个链接所述问题）。
5.如果还是不行，请检查程序的其他部分，我不知道是否还有其他地方影响速度，我了解的就这些了。
6.感谢阅读

# English 
Recently , I made a USB Keyboard Controller HID Device on STM32,based on STM32Cubemx generated Code. The keyboard is a N Key Rolling Back Keyboard,support pressing any key without any Conflict,but the RefleshRate(Polling Rate) is only 120 HZ.Someone says that "USB hid specification Requires device have 10 ms Interval between every packet".But i have a Gamepad that have about 1000HZ RefreshRate(Polling Rate),I envy it very much.After One Month searching on the internet and experiment,I found where is the problem finally.

What to do to make a 1000hz usb hid device?
1. Modify CUSTOM_HID_FS_BINTERVAL to 0x1 in STM32CubeMX's USB Middleware config(Recommended) or USBD_CONF.c(It is not recommended to modify it here, the regeneration program by CubeMX will overwrite it)
2. Considering you are sending data to the computer by packet,but your HID Report is larger than one packet size,because CubeMX default packet size is only 0x2 bytes,sending a 107 key keyboard data need more than one packet,it makes sending slow.What you need to do is to change it in CubeMX(?I don't know if it provides options that can be modified),or in the usbd_customhid.h ,the name of it is CUSTOM_HID_EPIN_SIZE,yes change 0x02U to 0x40U(64 bytes max packet size of USB FullSpeed Device,change to the number just fit the need is better)
3. Now you can program&dowload to the MCU and test the device, it is 1000HZ now.
4. If you find packet loss or card packet conditions after changing it, please refer to the first link Also Read at the bottom of the article, you may be using an old version of the unfixed HAL library (the current library probably no longer has the problem described in that link).
5. If it still doesn't work, check the rest of the program, I don't know if there are other places that affect the speed, that's all I know.
6. thanks for reading

# Also Read
1. https://community.st.com/s/question/0D53W00000HzPMBSA3/usb-hid-performance-optimization
2. https://www.techbang.com/posts/63583-what-is-101usb-polling-rate-of-computer-vocabulary-2000hz-3000hz-the-faster-the-better
