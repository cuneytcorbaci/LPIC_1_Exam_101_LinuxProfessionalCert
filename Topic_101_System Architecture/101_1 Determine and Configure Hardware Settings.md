# 101.1 Determine and configure hardware settings
**Weight : 2**

**Key knowledge areas :**

-   **Enable and disable integrated peripherals.**
-   **Differentiate between the various types of mass storage devices.**
-   **Determine hardware resources for devices.**
-   **Tools and utilities to list various hardware information (e.g. lsusb, lspci, etc.).**
-   **Tools and utilities to manipulate USB devices.**
-   **Conceptual understanding of sysfs, udev and dbus.**

Partial list of the used files, terms and utilities

-   **/sys/**
-   **/proc/**
-   **/dev/**
-   **modprobe**
-   **lsmod**
-   **lspci**
-   **lsusb**

|LPIC            |101.1 Lesson 1                                     |                                       
|----------------|--------|------------------------------------------|
|Sertifika       |LPIC-1                                             |
|Versiyon        |5.0                                                |
|Konu            |101 Sistem Mimarisi                                |
|Versiyon        |5.0												 |
|Hedef			 |101.1 Donanım Ayarlarını Belirleme ve Yapılandırma |
|Ders            |1 / 1                                              |
|                |                                                   | 

### **Introduction**

Since the early years of electronic computing, business and personal computer manufacturers have integrated a variety of hardware parts in their machines, which in turn need to be supported by the operating system. That could be overwhelming from the operating system developer’s perspective, unless standards for instruction sets and device communication are established by the industry. Similar to the standardized abstraction layer provided by the operating system to an application, these standards make it easier to write and maintain an operating system that is not tied to a specific hardware model. However, the complexity of the integrated underlying hardware sometimes requires adjustments on how resources should be exposed to the operating system, so it can be installed and function correctly.

Some of these adjustments can be made even without an installed operating system. Most machines offer a configuration utility that can be executed as the machine is turned on. Until the mid 2000s, the configuration utility was implemented in the BIOS (Basic Input/Output System), the standard for firmware containing the basic configuration routines found in x86 motherboards. From the end of the first decade of the 2000s, machines based on the x86 architecture started to replace the BIOS with a new implementation called UEFI (Unified Extensible Firmware Interface), which has more sophisticated features for identification, testing, configuration and firmware upgrades. Despite the change, it is not uncommon to still call the configuration utility BIOS, as both implementations fulfill the same basic purpose.

**Note**
Further details on the similarities and differences between BIOS and UEFI will be covered in a later lesson.

#### Device Activation

The system configuration utility is presented after pressing a specific key when the computer is turned on. Which key to press varies from manufacturer to manufacturer, but usually it is Del or one of the function keys, such as F2 or F12 . The key combination to use is often displayed in the power on screen.

In the BIOS setup utility it is possible to enable and disable integrated peripherals, activate basic error protection and change hardware settings like IRQ (interrupt request) and DMA (direct memory access). Changing these settings is rarely needed on modern machines, but it may be necessary to make adjustments to address specific issues. There are RAM technologies, for example, that are compatible with faster data transfer rates than the default values, so it is recommended to change it to the values specified by the manufacturer. Some CPUs offer features that may not be required for the particular installation and can be deactivated. Disabled features will reduce power consumption and can increase system protection, as CPU features containing known bugs can also be disabled.

If the machine is equipped with many storage devices, it is important to define which one has the correct bootloader and should be the first entry in the device boot order. The operating system may not load if the incorrect device comes first in the BIOS boot verification list.
