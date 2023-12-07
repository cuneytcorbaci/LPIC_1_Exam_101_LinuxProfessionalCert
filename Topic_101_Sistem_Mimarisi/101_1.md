# 101.1 Donanım Ayarlarını Belirleme ve Yapılandırma
**Ağırlık: 2**

**Ana Bilgi Alanları :**

-   **Entegre bileşenleri etkinleştirme ve devre dışı bırakma.**
-   **Çeşitli türlerdeki yığın depolama aygıtları arasındaki farkları ayırt etmek.**
-   **Aygıtlar için donanım kaynaklarını belirleme.**
-   **Çeşitli donanım bilgilerini listeleyen araçlar ve yardımcı programlar (örneğin, lsusb, lspci vb.)**
-   **USB cihazlarını manipüle etmek için araçlar ve yardımcı programlar.**
-   **Sysfs, udev ve dbus'un kavramsal anlayışı.**

Kullanılan dosyalar, terimler ve yardımcı programların kısmi listesi

-   **/sys/**
-   **/proc/**
-   **/dev/**
-   **modprobe**
-   **lsmod**
-   **lspci**
-   **lsusb**

|LPIC|01.1 Lesson 1 |
|-|-|
|Sertifika|LPIC-1|
|Versiyon|5.0|
|Konu|101 Sistem Mimarisi|
|Versiyon|5.0|
|Hedef|101.1 Donanım Ayarlarını Belirleme ve Yapılandırma |
|Ders|1 / 1| 

## **Giriş**

Elektronik bilgisayarların ilk yıllarından itibaren, iş ve kişisel bilgisayar üreticileri makinelerinde çeşitli donanım parçalarını entegre etmişlerdir. Bu parçalar da sırasıyla işletim sistemleri tarafından desteklenmelidir. Talimat setleri ve cihaz iletişimine yönelik standartlar endüstri tarafından oluşturulmadıkça, bu durum işletim sistemi geliştiricisi açısından çok zor olabilir. Bir uygulamaya işletim sistemi tarafından sağlanan standartlaştırılmış soyutlama katmanına benzer şekilde, bu standartlar, belirli bir donanım modeline bağlı olmayan bir işletim sistemi yazmayı ve sürdürmeyi daha kolay hale getirir. Ancak, temel donanımın karmaşıklığı, bazen kaynakların işletim sistemine nasıl sunulması gerektiğinde ayarlamalar yapmayı gerektirir; böylece doğru bir şekilde yüklenip çalışabilir.

Bazı ayarlamalar, kurulu bir işletim sistemi olmadan bile yapılabilir. Birçok makine, makine açılırken yürütülebilen bir yapılandırma yardımcı programı sunar. 2000'li yılların ortalarına kadar yapılandırma yardımcı programı, x86 anakartlarda bulunan temel yapılandırma rutinlerini içeren firmware için standart olan BIOS'ta (Temel Giriş/Çıkış Sistemi) uygulanmıştı. 2000'li yılların ilk on yılı sonlarından itibaren, x86 mimarisine dayalı makineler, BIOS'un yerine daha sofistike özelliklere sahip olan UEFI (Birleşik Genişletilebilir Firmware Arabirimi) adlı yeni bir uygulama ile değişmeye başladı. Ancak bu değişikliğe rağmen, her iki uygulama da temelde aynı amacı yerine getirdiği için yapılandırma yardımcı programına hala BIOS adı vermek yaygındır.

>**Not**
>BIOS ve UEFI arasındaki benzerlikler ve farklar hakkındaki daha fazla detay, sonraki bir ders kapsamında ele alınacaktır.


###  Aygıt Etkinleştirme

Sistem yapılandırma yardımcı programı, bilgisayar açıldığında belirli bir tuşa basıldığında görüntülenir. Hangi tuşun kullanılacağı üreticiden üreticiye değişir, ancak genellikle Del veya F2, F12 gibi fonksiyon tuşlarından biridir. Kullanılacak tuş kombinasyonu genellikle açılış ekranında gösterilir.

BIOS kurulum yardımcı programında entegre bileşenleri etkinleştirmek veya devre dışı bırakmak, temel hata korumasını etkinleştirmek ve IRQ (kesme talebi) ve DMA (doğrudan bellek erişimi) gibi donanım ayarlarını değiştirmek mümkündür. Bu ayarları değiştirmek, modern makinelerde genellikle gerekli olmasa da, belirli sorunları çözmek için ayarlamalar yapmak gerekebilir. Örneğin, bazı RAM teknolojileri, varsayılan değerlerden daha hızlı veri transfer hızlarına uyumlu olabilir; bu nedenle, üretici tarafından belirtilen değerlere değiştirmek önerilir. Bazı CPU'lar, belirli bir kurulum için gerekli olmayan özellikler sunabilir ve devre dışı bırakılabilir. Devre dışı bırakılan özellikler, güç tüketimini azaltabilir ve bilinen hataları içeren CPU özellikleri de devre dışı bırakılabilir, bu da sistem korumasını artırabilir.

Eğer makine çok sayıda depolama cihazıyla donatılmışsa, doğru bootloader'a sahip olan ve aygıtın başlangıç sırasında ilk giriş olması gerekeni tanımlamak önemlidir. Eğer BIOS başlangıç doğrulama listesinde yanlış cihaz öncelikli olarak gelirse, işletim sistemi yüklenmeyebilir.

### ****Linux'ta Aygıt İncelemesi****

Aygıtlar doğru bir şekilde tanımlandıktan sonra, bunlarla ilişkilendirilmesi gereken yazılım bileşenleri işletim sistemine aittir. Bir donanım özelliği beklenen şekilde çalışmıyorsa, sorunun tam olarak nerede olduğunu belirlemek önemlidir. Eğer bir donanım parçası işletim sistemi tarafından algılanmıyorsa, büyük olasılıkla parça ya da bağlı olduğu port arızalıdır. Donanım parçası doğru bir şekilde algılandığında, ancak hala düzgün çalışmıyorsa, sorun işletim sistemi tarafında olabilir. Bu nedenle, donanım ile ilgili sorunlarla uğraşırken atılacak ilk adımlardan biri, işletim sisteminin cihazı doğru bir şekilde algılayıp algılamadığını kontrol etmektir. Linux sisteminde donanım kaynaklarını tanımlamanın iki temel yolu vardır: özel komutları kullanmak veya özel dosyaları özel dosya sistemlerinin içinde okumak.

**İnceleme İçin Komutlar**

Linux sistemlerinde bağlı cihazları tanımlamak için iki temel komut bulunmaktadır:

**lspci**

PCI (Peripheral Component Interconnect) veriyolu üzerine bağlı olan tüm cihazları gösterir. PCI cihazları, anakarta bağlı bir disk kontrolcüsü gibi bir bileşen veya harici bir grafik kartı gibi bir PCI yuvasına takılmış bir genişletme kartı olabilir.

**lsusb**

Makineye bağlı olan USB (Universal Serial Bus) cihazlarını listeler. USB cihazları neredeyse her türlü amaç için bulunabilirken, USB arabirimi genellikle giriş cihazları - klavye, işaret cihazları - ve taşınabilir depolama ortamlarını bağlamak için kullanılır.

**lspci** ve **lsusb** komutlarının çıktısı, işletim sistemi tarafından tanımlanan tüm PCI ve USB cihazlarının bir listesini içerir. Ancak, her donanım parçasının karşılık gelen cihazı kontrol etmek için bir yazılım bileşenine ihtiyacı olduğundan, cihaz henüz tam olarak işlevsel olmayabilir. Bu yazılım bileşeni, bir çekirdek modülü olarak adlandırılır ve resmi Linux çekirdeğinin bir parçası olabilir veya diğer kaynaklardan ayrı olarak eklenmiş olabilir.

Linux işletim sistemine bağlı donanım cihazlarıyla ilgili çekirdek modülleri, diğer işletim sistemlerinde olduğu gibi "sürücüler" olarak da adlandırılır. Ancak Linux için sürücüler her zaman cihazın üreticisi tarafından sağlanmaz. Bazı üreticiler kendi ayrı yüklenen ikili sürücülerini sağlarken, birçok sürücü bağımsız geliştiriciler tarafından yazılmaktadır. Tarihsel olarak, örneğin Windows'ta çalışan parçaların, Linux için karşılık gelen bir çekirdek modülü olmayabilir. Günümüzde ise Linux tabanlı işletim sistemleri güçlü donanım desteğine sahiptir ve çoğu cihaz sorunsuz bir şekilde çalışır.

Donanımla doğrudan ilgili komutlar genellikle yürütülmek için kök (root) ayrıcalıklarına ihtiyaç duyar veya normal bir kullanıcı tarafından çalıştırıldığında sınırlı bilgi gösterir. Bu nedenle, tam işlevselliğe ve ayrıntılara erişmek için root kullanıcısı olarak giriş yapmak veya komutu **`sudo`** ile çalıştırmak gerekebilir.

Örneğin, aşağıdaki **lspci** komutunun çıktısı, birkaç tanımlanan cihazı gösterir:

```bash
$ lspci
01:00.0 VGA compatible controller: NVIDIA Corporation GM107 [GeForce GTX 750 Ti] (rev a2)
04:02.0 Network controller: Ralink corp. RT2561/RT61 802.11g PCI
04:04.0 Multimedia audio controller: VIA Technologies Inc. ICE1712 [Envy24] PCI MultiChannel I/O Controller (rev 02)
04:0b.0 FireWire (IEEE 1394): LSI Corporation FW322/323 [TrueFire] 1394a Controller (rev 70)

```

Bu tür komutların çıktısı onlarca satır uzunluğunda olabilir, bu nedenle önceki ve sonraki örnekler yalnızca ilgi alanıyla ilgili bölümleri içerir. Her satırın başındaki onaltılık sayılar, karşılık gelen PCI cihazının benzersiz adresleridir. lspci komutu, -s seçeneği ile belirli bir cihaz hakkında daha fazla detay gösterir, bu seçeneğin yanında `-v` seçeneği de kullanılmalıdır:

```bash
$ lspci -s 04:02.0 -v
04:02.0 Network controller: Ralink corp. RT2561/RT61 802.11g PCI
  Subsystem: Linksys WMP54G v4.1
  Flags: bus master, slow devsel, latency 32, IRQ 21
  Memory at e3100000 (32-bit, non-prefetchable) [size=32K]
  Capabilities: [40] Power Management version 2
  kernel driver in use: rt61pci

```

Şimdi çıktı, 04:02.0 adresindeki cihazın çok daha fazla detayını gösteriyor. Bu, bir ağ denetleyicisidir ve dahili adı Ralink corp. RT2561/RT61 802.11g PCI'dir. Alt sistem, cihazın marka ve modeli olan Linksys WMP54G v4.1 ile ilişkilidir ve teşhis amaçları için faydalı olabilir.

Çekirdek modülü, kullanılan çekirdek sürücüsünü gösteren "kernel driver in use" satırında tanımlanabilir; bu durumda modül rt61pci'dir. Toplanan tüm bilgilerden, aşağıdakileri varsaymak doğrudur:

1.  Cihaz tanımlandı.
2.  Eşleşen bir çekirdek modülü yüklendi.
3.  Cihaz kullanıma hazır olmalıdır.

Belirli bir cihaz için hangi çekirdek modülünün kullanıldığını doğrulamanın başka bir yolu, daha yeni sürümlerde bulunan **`-k`** seçeneği tarafından sağlanır.

```bash
**$ lspci -s 01:00.0 -k**
01:00.0 VGA compatible controller: NVIDIA Corporation GM107 [GeForce GTX 750 Ti] (rev a2)
  kernel driver in use: nvidia
  kernel modules: nouveau, nvidia_drm, nvidia

```

Seçilen cihaz için, bir NVIDIA GPU kartı, lspci'nin kullanılan modülü nvidia olarak adlandırdığını belirtir; bu bilgi, "kernel driver in use: nvidia" satırında yer alır ve tüm ilgili çekirdek modülleri "kernel modules: nouveau, nvidia_drm, nvidia" satırında listelenir.

lsusb komutu, lspci'ye benzer şekilde ancak yalnızca USB bilgilerini listeler:

```bash
$ lsusb
Bus 001 Device 029: ID 1781:0c9f Multiple Vendors USBtiny
Bus 001 Device 028: ID 093a:2521 Pixart Imaging, Inc. Optical Mouse
Bus 001 Device 020: ID 1131:1001 Integrated System Solution Corp. KY-BT100 Bluetooth Adapter
Bus 001 Device 011: ID 04f2:0402 Chicony Electronics Co., Ltd Genius LuxeMate i200 Keyboard
Bus 001 Device 007: ID 0424:7800 Standard Microsystems Corp.
Bus 001 Device 003: ID 0424:2514 Standard Microsystems Corp. USB 2.0 Hub
Bus 001 Device 002: ID 0424:2514 Standard Microsystems Corp. USB 2.0 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

```

lsusb komutu, mevcut USB bağlantı noktalarını ve bunlara bağlı cihazları gösterir. lspci ile benzer şekilde, -v seçeneği daha ayrıntılı çıktı gösterir. Belirli bir cihazı incelemek için, onun kimliğini `-d` seçeneğine sağlayarak seçilebilir:

```bash
$ lsusb -v -d 1781:0c9f
Bus 001 Device 029: ID 1781:0c9f Multiple Vendors USBtiny
Device Descriptor:
  bLength 18
  bDescriptorType 1
  bcdUSB 1.01
  bDeviceClass 255 Vendor Specific Class
  bDeviceSubClass 0
  bDeviceProtocol 0
  bMaxPacketSize0 8
  idVendor 0x1781 Multiple Vendors
  idProduct 0x0c9f USBtiny
  bcdDevice 1.04
  iManufacturer 0
  iProduct 2 USBtiny
  iSerial 0
  bNumConfigurations 1

```

`-t` seçeneği ile lsusb komutu, mevcut USB cihaz eşlemelerini hiyerarşik bir ağaç olarak gösterir:

```bash
$ lsusb -t
/: Bus 01.Port 1: Dev 1, Class=root_hub, Driver=dwc_otg/1p, 480M
  |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
	  |__ Port 1: Dev 3, If 0, Class=Hub, Driver=hub/3p, 480M
		  |__ Port 2: Dev 11, If 1, Class=Human Interface Device, Driver=usbhid, 1.5M
		  |__ Port 2: Dev 11, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
		  |__ Port 3: Dev 20, If 0, Class=Wireless, Driver=btusb, 12M
		  |__ Port 3: Dev 20, If 1, Class=Wireless, Driver=btusb, 12M
		  |__ Port 3: Dev 20, If 2, Class=Application Specific Interface, Driver=, 12M
		  |__ Port 1: Dev 7, If 0, Class=Vendor Specific Class, Driver=lan78xx, 480M
		|__ Port 2: Dev 28, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
		|__ Port 3: Dev 29, If 0, Class=Vendor Specific Class, Driver=, 1.5M

```

Bazı cihazların kendilerine ait modüllerle ilişkilendirilmiş olmaması mümkündür. Belirli cihazlarla iletişim, bir modül tarafından sağlanan aracı olmadan doğrudan uygulama tarafından yönetilebilir. Bununla birlikte,`lsusb -t` tarafından sağlanan çıktıda önemli bilgiler bulunmaktadır. Eşleşen bir modül mevcut olduğunda, cihazın satır sonundaki adı görünür, örneğin `Driver=btusb`. Aygıt Sınıfı, genel kategoriyi belirtir; örneğin, İnsan Arayüz Aygıtı, Kablosuz, Üreticiye Özgü Sınıf, Kütle Depolama gibi. Önceki listede bulunan btusb modülünü kullanan cihazı doğrulamak için, Bus ve Dev numaraları lsusb komutunun `-s` seçeneğine verilmelidir:

```bash
$ lsusb -s 01:20
Bus 001 Device 020: ID 1131:1001 Integrated System Solution Corp. KY-BT100 Bluetooth Adapter

```

Standart bir Linux sistemde herhangi bir zamanda yüklenmiş büyük bir çekirdek modül kümesine sahip olmak yaygındır. Bunlarla etkileşimde bulunmanın tercih edilen yolu, kmod paketi tarafından sağlanan komutları kullanmaktır. kmod, Linux çekirdek modülleri ile yaygın görevleri işlemek için tasarlanmış bir dizi araç içerir, örneğin ekleme, kaldırma, liste, özellikleri kontrol etme, bağımlılıkları ve takma adları çözme. Örneğin, lsmod komutu şu anda yüklenmiş tüm modülleri gösterir:

```bash
**$ lsmod**
Module Size           Used by
kvm_intel             138528 0
kvm                   421021 1 kvm_intel
iTCO_wdt              13480 0
iTCO_vendor_support   13419 1  iTCO_wdt
snd_usb_audio         149112 2
snd_hda_codec_realtek 51465 1
snd_ice1712           75006 3
snd_hda_intel         44075 7
arc4                  12608 2
snd_cs8427            13978 1 snd_ice1712
snd_i2c               13828 2 snd_ice1712,snd_cs8427
snd_ice17xx_ak4xxx    13128 1 snd_ice1712
snd_ak4xxx_adda       18487 2 snd_ice1712,snd_ice17xx_ak4xxx
microcode             23527 0
snd_usbmidi_lib       24845 1 snd_usb_audio
gspca_pac7302         17481 0
gspca_main            36226 1 gspca_pac7302
videodev              132348 2 gspca_main,gspca_pac7302
rt61pci               32326 0
rt2x00pci             13083 1 rt61pci
media                 20840 1 videodev
rt2x00mmio            13322 1 rt61pci
hid_dr                12776 0
snd_mpu401_uart       13992 1 snd_ice1712
rt2x00lib             67108 3 rt61pci,rt2x00pci,rt2x00mmio
snd_rawmidi           29394 2 snd_usbmidi_lib,snd_mpu401_uart

```

lsmod komutunun çıktısı üç sütuna ayrılmıştır:

-   **Module**
    
    Modül adı.
    
-   **Size**
    
    Modül tarafından işgal edilen RAM miktarı, byte cinsinden.
    
-   **Used by**
    
    Bağlı modüller.
    

Bazı modüllerin düzgün çalışması için diğer modüllere ihtiyaç duyduğu durumlar gibi, örneğin ses cihazları için modüllerde olduğu gibi, bazı modüller diğer modülleri gerektirebilir.

```bash
**$ lsmod | fgrep -i snd_hda_intel**
snd_hda_intel  42658 5
snd_hda_codec  155748 3 snd_hda_codec_hdmi,snd_hda_codec_via,snd_hda_intel
snd_pcm        81999 3 snd_hda_codec_hdmi,snd_hda_codec,snd_hda_intel
snd_page_alloc 13852 2 snd_pcm,snd_hda_intel
snd            59132 19
snd_hwdep,snd_timer,snd_hda_codec_hdmi,snd_hda_codec_via,snd_pcm,snd_seq,snd_hda_codec,snd_h
da_intel,snd_seq_device

```

Üçüncü sütun, "Used by," birinci sütundaki modülün düzgün çalışması için gereken modülleri gösterir. Linux ses mimarisine ait birçok modül, genellikle "snd" öneki ile başlayan şekilde birbirine bağımlıdır. Sistem teşhisleri sırasında sorunları ararken, mevcut belirli modülleri boşaltmak yararlı olabilir. modprobe komutu, çekirdek modüllerini yüklemek ve boşaltmak için kullanılabilir: bir modülü ve ilgili modülleri, kullanılan bir çalışan işlem tarafından kullanılmadıkları sürece, boşaltmak için modprobe -r komutu kullanılmalıdır. Örneğin, snd-hda-intel modülünü (HDA Intel ses cihazı için modül) ve ses sistemi ile ilgili diğer modülleri boşaltmak için:

```bash
# modprobe -r snd-hda-intel

```

Sistem çalışırken çekirdek modüllerini yüklemek ve boşaltmanın yanı sıra, çekirdek yüklenirken modül parametrelerini değiştirmek mümkündür; bu, komutlara seçenekler geçirmekten pek farklı değildir. Her modül belirli parametreleri kabul eder, ancak çoğu zaman varsayılan değerler önerilir ve ek parametrelere ihtiyaç duyulmaz. Ancak, bazı durumlarda bir modülün davranışını beklenen şekilde çalışacak şekilde değiştirmek için parametreleri kullanmak gerekebilir.

Tek argüman olarak modül adını kullanarak, modinfo komutu bir açıklama, dosya, yazar, lisans, kimlik, bağımlılıklar ve verilen modül için kullanılabilir parametreleri gösterir. Bir modül için özelleştirilmiş parametreler, bunları /etc/modprobe.conf dosyasına veya /etc/modprobe.d/ dizinindeki .conf uzantılı bireysel dosyalara ekleyerek kalıcı hale getirilebilir. -p seçeneği, modinfo komutunun tüm mevcut parametreleri göstermesini sağlar ve diğer bilgileri görmezden gelir:

```bash
# modinfo -p nouveau
vram_pushbuf:Create DMA push buffers in VRAM (int)
tv_norm:Default TV norm.
  Supported: PAL, PAL-M, PAL-N, PAL-Nc, NTSC-M, NTSC-J,
  hd480i, hd480p, hd576i, hd576p, hd720p, hd1080i.
  Default: PAL
  NOTE Ignored for cards with external TV encoders. (charp)
nofbaccel:Disable fbcon acceleration (int)
fbcon_bpp:fbcon bits-per-pixel (default: auto) (int)
mst:Enable DisplayPort multi-stream (default: enabled) (int)
tv_disable:Disable TV-out detection (int)
ignorelid:Ignore ACPI lid status (int)
duallink:Allow dual-link TMDS (default: enabled) (int)
hdmimhz:Force a maximum HDMI pixel clock (in MHz) (int)
config:option string to pass to driver core (charp)
debug:debug string to pass to driver core (charp)
noaccel:disable kernel/abi16 acceleration (int)
modeset:enable driver (default: auto, 0 = disabled, 1 = enabled, 2 = headless) (int)
atomic:Expose atomic ioctl (default: disabled) (int)
runpm:disable (0), force enable (1), optimus only default (-1) (int)

```

Örnek çıktı, nouveau modülü için mevcut tüm parametreleri gösterir; bu modül, NVIDIA GPU kartları için özel sürücüler yerine sunulan Nouveau Projesi tarafından sağlanan bir çekirdek modülüdür. Örneğin, modeset seçeneği, ekran çözünürlüğünün ve derinliğinin çekirdek alanında mı yoksa kullanıcı alanında mı ayarlanacağını kontrol etmeye izin verir. /etc/modprobe.d/nouveau.conf dosyasına options nouveau modeset=0 eklemek, modeset çekirdek özelliğini devre dışı bırakacaktır.

Eğer bir modül sorunlara neden oluyorsa, /etc/modprobe.d/blacklist.conf dosyası kullanılarak modülün yüklenmesi engellenebilir. Örneğin, nouveau modülünün otomatik olarak yüklenmesini engellemek için, blacklist nouveau satırı /etc/modprobe.d/blacklist.conf dosyasına eklenmelidir. Bu işlem, özel sürücü modülü nvidia yüklüyken ve varsayılan modül nouveau kullanılmak istenmiyorsa gereklidir.

>**Not**
>Sistemde varsayılan olarak zaten bulunan /etc/modprobe.d/blacklist.conf dosyasını değiştirebilirsiniz. Ancak, tercih edilen yöntem, yalnızca belirli bir çekirdek modülüne özgü ayarları içeren /etc/modprobe.d/<modül_adı>.conf adlı ayrı bir yapılandırma dosyası oluşturmaktır.


### Bilgi Dosyaları ve Aygıt Dosyaları

lspci, lsusb ve lsmod komutları, işletim sistemi tarafından depolanan donanım bilgilerini okumak için ön uçlar olarak çalışır. Bu tür bilgiler, /proc ve /sys dizinlerindeki özel dosyalarda saklanır. Bu dizinler, bir cihaz bölümünde değil, yalnızca çekirdeğin çalışma zamanı yapılandırmasını ve çalışan işlemlerle ilgili bilgileri depolamak için kullanılan RAM alanında bulunan dosya sistemlerine bağlama noktalarıdır. Bu tür dosya sistemleri geleneksel dosya depolama için tasarlanmamıştır, bu nedenle bunlara sahte dosya sistemleri denir ve yalnızca sistem çalışırken var olurlar. /proc dizini, çalışan işlemler ve donanım kaynaklarıyla ilgili bilgiler içeren dosyaları içerir. Donanımı incelemek için /proc dizinindeki önemli dosyalardan bazıları şunlardır:

**/proc/cpuinfo** İşletim sistemi tarafından bulunan CPU'lar hakkında detaylı bilgileri listeler.

**/proc/interrupts** Her CPU için her G/Ç cihazı için kesmelerin sayılarını listeleyen bir dosyadır.

**/proc/ioports** Kullanımdaki Giriş/Çıkış bağlantı noktalarının şu anda kayıtlı olanlarını listeleyen bir dosyadır.

**/proc/dma** Kullanımdaki DMA (doğrudan bellek erişimi) kanallarını listeleyen bir dosyadır.

/sys dizinindeki dosyalar, /proc'taki dosyalara benzer rolleri üstlenir. Ancak, /sys dizini, /proc'un çalışan işlemler ve yapılandırma dahil olmak üzere çeşitli çekirdek veri yapıları hakkındaki bilgileri içerirken, /sys dizini donanım ile ilgili cihaz bilgilerini ve çekirdek verilerini depolamak için özel bir amaç taşır.

Standart bir Linux sistemindeki cihazlarla doğrudan ilişkili başka bir dizin ise /dev'dir. /dev içindeki her dosya bir sistem cihazıyla ilişkilidir, özellikle depolama cihazlarıyla. Örneğin, bir miras IDE sabit diski, anakartın birinci IDE kanalına bağlandığında, /dev/hda dosyası tarafından temsil edilir. Bu diskteki her bölüm, /dev/hda1, /dev/hda2 en son bulunan bölüme kadar tanımlanır.

Taşınabilir cihazlar, karşılık gelen cihazları /dev dizininde oluşturan udev alt sistemi tarafından yönetilir. Linux çekirdeği, donanım tespiti olayını yakalar ve bunu udev sürecine ileterek ardından cihazı tanır ve önceden tanımlanmış kuralları kullanarak /dev dizininde dinamik olarak karşılık gelen dosyaları oluşturur.

Mevcut Linux dağıtımlarında, udev, makine gücü açıldığında (coldplug tespiti) zaten mevcut olan cihazların tanımlanması ve yapılandırılması, ve sistem çalışırken tanımlanan cihazların (hotplug tespiti) tanımlanması için sorumludur. Udev, /sys içinde bağlanan donanım ile ilgili bilgiler için sahte bir dosya sistem olan SysFS'ye dayanır.

**Not**

Hotplug terimi, sistem çalışırken bir cihazın tespiti ve yapılandırılması için kullanılan bir terimdir, örneğin bir USB cihazı takıldığında. Linux çekirdeği, sürüm 2.6'dan itibaren hotplug özelliklerini destekleyerek, bir cihazın bağlandığında veya bağlantısı kesildiğinde çoğu sistem busunu (PCI, USB vb.) hotplug olaylarını tetiklemesine izin verir.


Yeni cihazlar tespit edildikçe, udev, /etc/udev/rules.d/ dizininde depolanan önceden tanımlanmış kurallar arasında eşleşen bir kural arar. En önemli kurallar dağıtım tarafından sağlanır, ancak özel durumlar için yeni kurallar eklemek mümkündür.

### Depolama Cihazları

Linux'ta depolama cihazları, veri bloklarıyla okunup yazılan bu cihazlar için genellikle blok cihazları olarak anılır, çünkü veri blokları önbellek verisiyle farklı boyutlarda ve pozisyonlarda bu cihazlardan okunur ve yazılır. Her blok cihazı, /dev dizinindeki bir dosya tarafından tanımlanır; dosyanın adı, cihaz türüne (IDE, SATA, SCSI, vb.) ve bölümlerine bağlı olarak değişir. Örneğin, CD/DVD ve floppy cihazları, adları /dev'de buna göre belirlenecektir: İkinci IDE kanalına bağlı bir CD/DVD sürücüsü /dev/hdc olarak tanımlanacaktır (/dev/hda ve /dev/hdb birinci IDE kanalındaki master ve slave cihazları için ayrılmıştır) ve eski bir floppy sürücüsü /dev/fd0, /dev/fd1, vb. şeklinde tanımlanır.

Linux çekirdeği sürümü 2.4'ten itibaren, çoğu depolama cihazı, donanım türlerine bakılmaksızın SCSI cihazları gibi tanımlanmaktadır. IDE, SSD ve USB blok cihazları sd ön eki ile başlayacaktır. IDE diskleri için sd ön eki kullanılacaktır, ancak sürücünün bir master veya slave olup olmadığına bağlı olarak üçüncü harf seçilecektir (birinci IDE kanalında, master sda ve slave sdb olacaktır). Bölümler sayısal olarak listelenir. Yollar /dev/sda1, /dev/sda2, vb. ilk tanımlanan blok cihazının birinci ve ikinci bölümleri için kullanılır ve /dev/sdb1, /dev/sdb2, vb. ikinci tanımlanan blok cihazının birinci ve ikinci bölümlerini tanımlamak için kullanılır. Bu düzenin istisnası, bellek kartları (SD kartlar) ve NVMe cihazları (PCI Express veriyolu üzerine bağlı SSD'ler) ile ortaya çıkar. SD kartları için, /dev/mmcblk0p1, /dev/mmcblk0p2, vb. yolları, ilk tanımlanan cihazın birinci ve ikinci bölümlerini tanımlamak için kullanılır ve /dev/mmcblk1p1, /dev/mmcblk1p2, vb. yolları, ikinci tanımlanan cihazın birinci ve ikinci bölümlerini tanımlamak için kullanılır. NVMe cihazları, /dev/nvme0n1p1 ve /dev/nvme0n1p2 gibi bir önek alır.

## Yönlendirilmiş Egzersizler

1.  Eğer bir işletim sistemi, sisteme ikinci bir SATA diski ekledikten sonra başlatılamıyorsa, tüm parçaların kusurlu olmadığını bilerek, bu hatanın olası nedeni nedir?
    
2.  Eğer yeni edindiğiniz masaüstü bilgisayarınızın PCI veriyolu üzerine bağlı harici bir ekran kartının üretici tarafından reklamı yapılan kart olduğundan emin olmak istiyorsanız, ancak bilgisayar kasasını açmak garantiyi geçersiz kılacaktır. İşletim sistemi tarafından algılandığı şekliyle ekran kartının detaylarını listeleyebilmek için hangi komut kullanılabilir?
    
3.  Aşağıdaki satır, **`lspci`** komutu tarafından oluşturulan çıktının bir parçasıdır:
    
    ```bash
    03:00.0 RAID bus controller: LSI Logic / Symbios Logic MegaRAID SAS 2208 [Thunderbolt]
    (rev 05)
    
    ```
    
    Bu özel cihaz için kullanılan çekirdek modülünü belirlemek için hangi komutu çalıştırmalısınız?
    
4.  Bir sistem yöneticisi, sistem yeniden başlatılmadan önce bluetooth çekirdek modülü için farklı parametreleri denemek istiyor. Ancak, modprobe -r bluetooth komutuyla modülü boşaltma girişimi şu hatayla sonuçlanıyor:
    
    ```bash
    modprobe: FATAL: Module bluetooth is in use.
    
    ```
    
    Bu hatanın olası nedeni nedir?
    

## Keşif Alıştırmaları

1.  Üretim ortamlarında eski makinaları bulmak yaygın bir durumdur, özellikle bazı ekipmanların kontrol bilgisayarıyla iletişim kurmak için güncelliğini yitirmiş bir bağlantı kullanması durumunda. Örneğin, eski BIOS firmware'ine sahip bazı x86 sunucular, bir klavye algılanmadığında başlamayabilir. Bu özel sorun nasıl önlenir?
2.  Linux çekirdeği etrafında inşa edilmiş işletim sistemleri, x86 dışındaki birçok bilgisayar mimarisi için de mevcuttur, örneğin ARM mimarisine dayalı tek kartlı bilgisayarlar. Bir dikkatli kullanıcı, Raspberry Pi gibi bu tür makinelerde lspci komutunun eksik olduğunu fark edecektir. Bu, x86 makineleriyle karşılaştırıldığında bu eksikliği haklı çıkaran hangi farklılıktır?
3.  Birçok ağ yönlendirici, bir USB portuna harici bir cihazın bağlanmasına izin verir, örneğin bir USB sabit sürücü. Çoğu Linux tabanlı işletim sistemi kullandığı için, bir harici USB sabit sürücü /dev/ dizininde nasıl adlandırılır, varsayılan olarak başka bir konvansiyonel blok cihazı yönlendiricide bulunmuyorsa?
4.  2018'de Meltdown olarak bilinen donanım zafiyeti keşfedildi. Bu, birçok mimarinin neredeyse tüm işlemcilerini etkiler. Linux çekirdeğinin güncel sürümleri, mevcut sistemin bu zafiyetlere karşı savunmasız olup olmadığını bildirebilir. Bu bilgi nasıl elde edilir?

## Özet

Bu ders, Linux çekirdeğinin donanım kaynaklarıyla nasıl başa çıktığını, özellikle x86 mimarisinde, kapsayan genel kavramları ele alır. Ders aşağıdaki konuları içerir: • BIOS veya UEFI yapılandırma yardımcı programlarında tanımlanan ayarların işletim sisteminin donanım ile etkileşim şeklini nasıl etkileyebileceği. • Bir standart Linux sistemi tarafından sağlanan araçları kullanarak donanım hakkında bilgi edinme. • Dosya sistemlerinde kalıcı ve taşınabilir depolama cihazlarını tanımlama.

İncelenen komutlar ve prosedürler şunlardı: • Algılanan donanımı incelemek için kullanılan komutlar: lspci ve lsusb. • Çekirdek modüllerini yönetmek için kullanılan komutlar: lsmod ve modprobe. • Donanım ile ilgili özel dosyalar, ya /dev/ dizininde bulunan dosyalar ya da /proc/ ve /sys/ pseudo- dosya sistemlerinde bulunan dosyalar.

## Yönlendirilmiş Egzersizlere Yanıtlar

1.  Varsayalım ki bir işletim sistemi, sisteme ikinci bir SATA diski ekledikten sonra başlatılamıyor. Tüm parçaların kusurlu olmadığını bildiğimize göre, bu hatanın olası nedeni nedir?
    
    BIOS setup yardımcı programında önyükleme cihazı sırasının yapılandırılması gerekir, aksi takdirde BIOS önyükleyiciyi çalıştıramayabilir.
    
2.  Varsayalım ki yeni edinilmiş masaüstü bilgisayarınızın PCI veriyolu üzerine bağlı harici bir ekran kartının gerçekten üretici tarafından reklamı yapılan kart olduğundan emin olmak istiyorsunuz, ancak bilgisayar kasasını açmak garantiyi geçersiz kılacaktır. İşletim sistemi tarafından algılandığı şekliyle ekran kartının detaylarını listeleyen hangi komut kullanılabilir?
    
    **`lspci`** komutu, PCI veriyolu üzerine bağlı olan tüm cihazların detaylı bilgilerini listeleyecektir.
    
3.  Aşağıdaki satır, **`lspci`** komutu tarafından oluşturulan çıktının bir parçasıdır:
    
    ```bash
    03:00.0 RAID bus controller: LSI Logic / Symbios Logic MegaRAID SAS 2208 [Thunderbolt]
    (rev 05)

    ```
    
    Bu belirli cihaz için kullanılan çekirdek modülünü tanımlamak için hangi komutu çalıştırmalısınız?
    
    ```bash
    lspci -s 03:00.0 -v or lspci -s 03:00.0 -k
    
    ```
    
4.  Bir sistem yöneticisi, sistemi yeniden başlatmadan önce bluetooth çekirdek modülü için farklı parametreleri denemek istiyor. Ancak, modprobe -r bluetooth komutuyla modülü kaldırmaya yönelik her girişim aşağıdaki hatayla sonuçlanıyor:
    
    ```bash
    **modprobe: FATAL: Module bluetooth is in use.**
    
    ```
    
    Bu hatanın olası nedeni, bluetooth modülünün bir çalışan işlem tarafından kullanılıyor olmasıdır.
    

## Keşif Egzersizlerine Yanıtlar

-   Üretim ortamlarında eski makineler bulmak alışılmış bir durumdur, özellikle bazı ekipmanların kontrol bilgisayarı ile iletişim kurmak için eski bir bağlantı kullanması durumunda. Bu nedenle, bu eski makinelerin özel özelliklerine özel bir dikkat göstermek gereklidir. Örneğin, bazı x86 sunucuları eski BIOS firmware'ine sahiptir ve klavye algılanmadığında başlatılmaz. Bu özel sorunu nasıl önleyebiliriz?
    -   BIOS yapılandırma aracında, bir klavye bulunamadığında bilgisayarın kilitlenmesini devre dışı bırakma seçeneği bulunmaktadır.
-   Linux çekirdeği etrafında inşa edilen işletim sistemleri, x86 dışındaki birçok bilgisayar mimarisi için de mevcuttur, örneğin ARM mimarisine dayalı tek kartlı bilgisayarlar gibi. Bu tür makinelerde, örneğin Raspberry Pi gibi, lspci komutunun bulunmamasını haklı çıkaran x86 makinelerinden fark nedir?
    -   Çoğu x86 makinesinin aksine, Raspberry Pi gibi ARM tabanlı bir bilgisayarın bir PCI otobüsü yoktur, bu nedenle lspci komutu işe yaramaz.
-   Birçok ağ yönlendiricisi, USB portuna harici bir cihazın bağlanmasına izin veren bir USB portuna sahiptir, örneğin bir USB sabit disk. Bu tür yönlendiricilerin çoğu Linux tabanlı bir işletim sistemini kullanıyor, bu durumda harici bir USB sabit diski /dev/ dizininde nasıl adlandırırız, varsayalım ki sistemde başka geleneksel bir blok cihazı yok?
    -   Modern Linux çekirdekleri, USB sabit diskleri SATA cihazları olarak tanımlar, bu nedenle karşılık gelen dosya /dev/sda olacaktır, çünkü sistemde başka geleneksel bir blok cihazı bulunmamaktadır.
-   2018'de, Meltdown olarak bilinen donanım güvenlik açığı keşfedildi. Bu, birçok mimarinin neredeyse tüm işlemcilerini etkiler. Linux çekirdeğinin güncel sürümleri, mevcut sistemin bu açığa karşı savunmasız olup olmadığını bildirebilir. Bu bilgi nasıl elde edilir?
    -   /proc/cpuinfo dosyasında, karşılık gelen CPU için bilinen hataları gösteren bir satır vardır, örneğin bugs: cpu_meltdown.