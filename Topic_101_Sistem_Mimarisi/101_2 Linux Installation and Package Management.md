# 101.2 Sistem Önyüklemesi
**Ağırlık: 3**

**Ana Bilgi Alanları :**

-   **Önyükleyiciye ortak komutlar ve önyükleme sırasında çekirdeğe seçenekler sağlama.**
-   **BIOS/UEFI'den önyükleme tamamlanana kadar olan önyükleme sırasının bilgisini gösterme.**
-   **SysVinit ve systemd anlayışı.**
-   **Upstart bilinci.**
-   **Günlük dosyalarında önyükleme olaylarını kontrol etme.**

**Kullanılan dosyalar, terimler ve yardımcı programların kısmi listesi**

-   **dmesg**
-   **journalctl**
-   **BIOS**
-   **UEFI**
-   **bootloader**
-   **kernel**
-   **initramfs**
-   **init**
-   **SysVinit**
-   **systemd**

|LPIC|01.1 Lesson 1 |
|-|-|
|Sertifika|LPIC-1|
|Versiyon|5.0|
|Konu|101.2 Sistemi Önyüklemesi|
|Versiyon|5.0|
|Hedef|101.1 Donanım Ayarlarını Belirleme ve Yapılandırma |
|Ders|1 / 1| 

## **Giriş**

Makineyi kontrol etmek için, işletim sisteminin ana bileşeni olan kernel (çekirdek), BIOS veya UEFI gibi önceden yüklenmiş bir firmware tarafından yüklenen bir önyükleyici programı tarafından yüklenmelidir. Önyükleyici, kernel'i yüklemek için kullanılabilen parametreleri özelleştirmek üzere yapılandırılabilir, örneğin kök dosya sisteminin bulunduğu bölüm veya işletim sisteminin hangi modda çalışacağı gibi. Kernel yüklendikten sonra, kernel, donanımı tanımlayarak ve yapılandırarak önyükleme sürecine devam eder. Son olarak, kernel, sistem hizmetlerini başlatma ve yönetme sorumluluğuna sahip olan yardımcı programı çağırır.

>**NOT**
> Bu ders içinde gerçekleştirilen bazı Linux dağıtımlarındaki komutlar, kök (root) ayrıcalıklarını gerektirebilir.

## **BIOS ve UEFI**

x86 makineleri tarafından önyükleyiciyi çalıştırmak için yürütülen prosedürler, BIOS veya UEFI kullanıp kullanmadıklarına bağlı olarak farklıdır. BIOS, Basic Input/Output System kısaltmasıyla, anakart üzerine bağlı olan bir non-volatile bellek çipinde depolanan ve bilgisayar her açıldığında yürütülen bir programdır. Bu tür bir program firmware olarak adlandırılır ve depolama konumu, sistemin sahip olabileceği diğer depolama cihazlarından ayrıdır. BIOS, BIOS yapılandırma yardımcı programında tanımlanan sıraya uygun olarak ilk depolama cihazındaki ilk 440 baytın önyükleyicinin ilk aşaması (aynı zamanda bootstrap olarak da adlandırılır) olduğunu varsayar. Depolama cihazlarının standart DOS bölüm şemasını kullanan MBR (Master Boot Record) olarak adlandırılan ilk 512 baytı, önyükleyicinin ilk aşamasının yanı sıra bölüm tablosunu da içerir. Eğer MBR doğru veriyi içermiyorsa, sistem alternatif bir yöntem kullanılmadıkça başlatılamaz.

Genel olarak, BIOS'a sahip bir sistemde başlatma işlemi için yapılan ön işlemler şunlardır:

1. Makine açıldığında, **POST** (power-on self-test) işlemi basit donanım arızalarını tanımlamak için yürütülür.
2. BIOS, video çıkışı, klavye ve depolama ortamları gibi temel bileşenleri etkinleştirerek sistem yüklemesini başlatır.
3. BIOS, önyükleyicinin ilk aşamasını MBR'den (BIOS yapılandırma yardımcı programında tanımlanan ilk cihazın ilk 440 baytı) yükler.
4. Önyükleyicinin ilk aşaması, boot seçeneklerini sunan ve çekirdeği yükleyen sorumlu olan önyükleyicinin ikinci aşamasını çağırır.

UEFI, Unified Extensible Firmware Interface kısaltmasıyla, BIOS'tan bazı temel noktalarda farklılık gösterir. BIOS gibi, UEFI de bir firmware'dir ancak bölümleri tanımlayabilir ve bunlarda bulunan birçok dosya sistemini okuyabilir. UEFI, MBR'ye dayanmaz, yalnızca anakartta bulunan non-volatile belleğine (NVRAM) kaydedilen ayarları dikkate alır. Bu tanımlamalar, otomatik olarak yürütülecek veya bir önyükleme menüsünden çağrılacak olan UEFI uyumlu programların, EFI uygulamalarının, konumunu belirtir. EFI uygulamaları, önyükleyiciler, işletim sistemi seçicileri, sistem teşhis ve onarım araçları vb. olabilir. Bunlar geleneksel bir depolama cihazı bölümünde ve uyumlu bir dosya sistemine sahip olmalıdır. Standart uyumlu dosya sistemleri blok cihazları için FAT12, FAT16 ve FAT32, optik ortamlar için ise ISO-9660'dir. Bu yaklaşım, BIOS ile mümkün olanlardan çok daha sofistike araçların uygulanmasına olanak tanır.

Genellikle, **UEFI** olan bir sistemde işletim sistemi öncesi başlatma adımları şunlardır:

1.  Makine açıldığında, POST (power-on self-test) işlemi basit donanım arızalarını tanımlamak için yürütülür.
2.  UEFI, video çıkışı, klavye ve depolama ortamları gibi temel bileşenleri etkinleştirerek sistem yüklemesini başlatır.
3.  UEFI'nin firmware'i, NVRAM'de depolanan tanımlamaları okur ve ESP bölümünün dosya sistemine depolanan önceden tanımlanmış EFI uygulamasını yürütmek için kullanır. Genellikle, önceden tanımlanmış EFI uygulaması bir önyükleyicidir.
4.  Eğer önceden tanımlanmış EFI uygulaması bir önyükleyici ise, çekirdeği yükleyerek işletim sistemini başlatır.

UEFI standardı ayrıca "**Secure Boot**" adlı bir özelliği destekler. Bu özellik, yalnızca imzalanmış EFI uygulamalarının, yani donanım üreticisi tarafından yetkilendirilmiş EFI uygulamalarının yürütülmesine izin verir. Bu özellik, kötü amaçlı yazılımlara karşı korumayı artırır ancak üretici garantisi kapsamında olmayan işletim sistemlerini kurmayı zorlaştırabilir.

### The Bootloader

x86 mimarisinde Linux için en popüler önyükleyici **GRUB** (**Grand Unified Bootloader**)'dir. BIOS veya UEFI tarafından çağrıldığında, GRUB başlatılabilir işletim sistemlerinin bir listesini görüntüler. Bazı durumlarda, liste otomatik olarak görünmez, ancak BIOS tarafından GRUB çağrılırken `Shift` tuşuna basarak manuel olarak çağrılabilir. UEFI sistemlerinde ise `Esc` tuşu kullanılmalıdır.

GRUB menüsünden yüklenmesi istenen kurulu çekirdek seçilebilir ve ona yeni parametreler iletilmesi mümkündür. Çoğu çekirdek parametresi `option=value` şablonunu takip eder. En kullanışlı çekirdek parametrelerinden bazıları şunlardır:

`acpi`

ACPI desteğini etkinleştirir ya da devre dışı bırakır. `acpi=off` ACPI desteğini devre dışı bırakacaktır.

`init`

Alternatif bir sistem başlatıcısı belirler. Örneğin, `init=/bin/bash`, Bash kabuğunu başlatıcı olarak belirleyecektir. Bu, kernel önyükleme sürecinden hemen sonra bir kabuk oturumu başlatacaktır.

`systemd.unit` 

Etkinleştirilecek systemd hedefini belirler. Örneğin, `systemd.unit=graphical.target`. Systemd, SysV için tanımlanan sayısal çalışma seviyelerini de kabul eder. Örneğin, çalışma seviyesi 1'i etkinleştirmek için sadece sayı 1 veya harf S (kısaltma için "single")'yi bir çekirdek parametresi olarak eklemek yeterlidir.

`mem` 

Sistemin kullanılabilir RAM miktarını ayarlar. Bu parametre, her bir konuğa ne kadar RAM'in kullanılabilir olacağını sınırlamak için özellikle sanal makineler için kullanışlıdır. Örneğin, `mem=512M` kullanmak, belirli bir konuk sistem için kullanılabilir RAM miktarını 512 megabayta sınırlayacaktır.

`maxcpus` 

Simetrik çok işlemcili makinelerde sisteme görünür olan işlemci (veya işlemci çekirdekleri) sayısını sınırlar. Sanal makineler için de kullanışlıdır. 0 değeri, çok işlemcili makineler için destekleri kapatır ve `nosmp` çekirdek parametresiyle aynı etkiye sahiptir. `maxcpus=2` parametresi, işletim sistemine kullanılabilir iki işlemciyi sınırlayacaktır.

`quiet` 

Çoğu önyükleme mesajını gizler.

`vga` 

Video modunu seçer. `vga=ask` parametresi, seçilebilecek mevcut modların bir listesini gösterecektir.

`root` 

Bootloader'da önceden yapılandırılmış olan kök bölümünden farklı olarak, kök bölümünü ayarlar. Örneğin, `root=/dev/sda3`.

`rootflags`

Kök dosya sistemi için bağlama seçenekleri.

`ro` 

Kök dosya sisteminin ilk bağlamasını salt-okunur yapar.

`rw` 

İlk bağlamada kök dosya sistemine yazma izni verir.

### Sistemi Başlatma

Kernel dışında, işletim sistemi beklenen özellikleri sağlayan diğer bileşenlere de bağlıdır. Bu bileşenlerin birçoğu, sistem başlatma süreci sırasında yüklenir ve basit kabuk betiklerinden daha karmaşık hizmet programlarına kadar çeşitlilik gösterir. Betikler, genellikle sistem başlatma süreci sırasında çalışacak ve sona erecek kısa süreli görevleri gerçekleştirmek için kullanılır. Hizmetler, `daemons` olarak da bilinir, işletim sisteminin içsel yönlerinden sorumlu olabileceği için her zaman etkin olabilirler.

Linux dağıtımlarına başlangıç betikleri ve farklı özelliklere sahip daemons'ları entegre etme yollarının çeşitliliği çok büyüktür. Bu, tarihsel olarak tüm Linux dağıtımlarının bakımını ve kullanıcılarının beklentilerini karşılayan tek bir çözümün gelişimini engelleyen bir gerçektir. Bununla birlikte, dağıtım bakımcılarının bu işlevi gerçekleştirmek için seçtikleri herhangi bir araç, en azından sistem hizmetlerini başlatma, durdurma ve yeniden başlatma yeteneğine sahip olacaktır. Bu eylemler genellikle yazılım güncellemesi sonrasında sistem tarafından otomatik olarak gerçekleştirilir, ancak sistem yöneticisi genellikle yapılandırma dosyasında değişiklik yaptıktan sonra hizmeti manuel olarak yeniden başlatması gerekebilir.

Aynı zamanda bir sistem yöneticisinin, koşullara bağlı olarak belirli bir dizi daemondan oluşan bir kümesi etkinleştirebilmesi de pratik olacaktır. Örneğin, sistem bakım görevlerini gerçekleştirmek için sadece bir minimum hizmet kümesini çalıştırmak mümkün olmalıdır.

>**NOT**
> Tam anlamıyla konuşmak gerekirse, işletim sistemi sadece kernel ve donanımı kontrol eden, tüm süreçleri yöneten bileşenlerden oluşur. Ancak yaygın olarak "işletim sistemi" terimi daha geniş bir anlamda kullanılır; burada, kullanıcının temel hesaplamalı görevleri gerçekleştirebileceği yazılım ortamını oluşturan ayrı programların bütününü ifade eder.
İşletim sisteminin başlatılması, önyükleyici kernel'i RAM'e yüklediğinde başlar. Ardından, kernel CPU üzerinde kontrolü ele alır ve işletim sisteminin temel yönlerini, temel donanım yapılandırması ve bellek adresleme gibi, tespit edip ayarlamaya başlar.

Kernel daha sonra `initramfs`'yi (initial RAM filesystem - başlangıç RAM dosya sistemi) açacaktır. Initramfs, önyükleme sürecinde geçici bir kök dosya sistemi olarak kullanılan bir dosya sistemini içeren bir arşivdir. Initramfs dosyasının temel amacı, kernelin işletim sisteminin "gerçek" kök dosya sistemine erişebilmesi için gereken modülleri sağlamaktır.

Kök dosya sistemi kullanıma hazır olduğunda, kernel **/etc/fstab**'de yapılandırılmış tüm dosya sistemlerini bağlar (mount) ve ardından ilk programı yani "**init**" adlı bir yardımcı programı çalıştırır. Init programı, tüm başlatma betiklerini ve sistem daemons'larını çalıştırmaktan sorumludur. Geleneksel init'in yanı sıra, systemd ve Upstart gibi bu tür sistem başlatıcılarının farklı uygulamaları bulunmaktadır. Init programı yüklendikten sonra, initramfs RAM'den kaldırılır.

`SysV standardı`

 SysVinit standardına dayanan bir hizmet yöneticisi, **runlevel** kavramını kullanarak hangi daemons ve kaynakların kullanılabilir olacağını kontrol eder. Runlevellar, dağıtım bakımcıları tarafından belirli amaçları yerine getirmek üzere tasarlanmış olan 0'dan 6'ya kadar numaralandırılmıştır. Tüm dağıtımlar arasında paylaşılan tek runlevel tanımlamaları, runlevel 0, 1 ve 6'dır.

`systemd` 

systemd, SysV komutları ve runlevellar için uyumluluk katmanına sahip modern bir sistem ve hizmet yöneticisidir. systemd'nin eşzamanlı bir yapıya sahip olması, hizmet etkinleştirme için soketler ve D-Bus kullanması, talep üzerine daemon yürütme, cgroups ile işlem izleme, anlık destek, sistem oturumu kurtarma, bağlama noktası kontrolü ve bağımlılık tabanlı bir hizmet kontrolü içermektedir. Son yıllarda, çoğu büyük Linux dağıtımı systemd'i varsayılan sistem yöneticisi olarak kademeli olarak benimsemiştir.

`Upstart` 

systemd gibi, Upstart da init'in bir alternatifidir. Upstart'ın odak noktası, sistem hizmetlerinin yükleme sürecini paralelleştirerek önyükleme sürecini hızlandırmaktır. Upstart, geçmiş sürümlerde Ubuntu tabanlı dağıtımlar tarafından kullanılmıştır, ancak günümüzde systemd'e yerini bırakmıştır.

### Başlatma İncelemesi

Önyükleme süreci sırasında hatalar meydana gelebilir, ancak bunlar işletim sistemini tamamen durdurmaya yetecek kadar kritik olmayabilir. Bununla birlikte, bu hatalar sistem beklentilerini tehlikeye atabilir. Tüm hatalar, hata ne zaman ve nasıl meydana geldiği hakkında değerli bilgiler içerdikleri için gelecekteki incelemeler için kullanılabilir. Hata mesajları oluşturulmasa bile, önyükleme süreci sırasında toplanan bilgiler, ayarlama ve yapılandırma amaçları için kullanışlı olabilir.

Çekirdeğin mesajlarını, önyükleme mesajları dahil olmak üzere sakladığı bellek alanına "**kernel ring buffer**" denir. Bu mesajlar, önyükleme sürecinde görüntülenmese bile, örneğin bir animasyon görüntülendiğinde, çekirdek halka tamponunda saklanır. Ancak, sistem kapatıldığında veya `dmesg --clear` komutu çalıştırıldığında çekirdek halka tamponu tüm mesajları kaybeder. Seçeneksiz olarak kullanıldığında, dmesg komutu çekirdek halka tamponundaki mevcut mesajları görüntüler:

```bash
$ dmesg

[ 5.262389] EXT4-fs (sda1): mounted filesystem with ordered data mode. Opts: (null)
[ 5.449712] ip_tables: (C) 2000-2006 Netfilter Core Team
[ 5.460286] systemd[1]: systemd 237 running in system mode.
[ 5.480138] systemd[1]: Detected architecture x86-64.
[ 5.481767] systemd[1]: Set hostname to <torre>.
[ 5.636607] systemd[1]: Reached target User and Group Name Lookups.
[ 5.636866] systemd[1]: Created slice System Slice.
[ 5.637000] systemd[1]: Listening on Journal Audit Socket.
[ 5.637085] systemd[1]: Listening on Journal Socket.
[ 5.637827] systemd[1]: Mounting POSIX Message Queue File System...
[ 5.638639] systemd[1]: Started Read required files in advance.
[ 5.641661] systemd[1]: Starting Load Kernel Modules...
[ 5.661672] EXT4-fs (sda1): re-mounted. Opts: errors=remount-ro
[ 5.694322] lp: driver loaded but no devices found
[ 5.702609] ppdev: user-space parallel port driver
[ 5.705384] parport_pc 00:02: reported by Plug and Play ACPI
[ 5.705468] parport0: PC-style at 0x378 (0x778), irq 7, dma 3
[PCSPP,TRISTATE,COMPAT,EPP,ECP,DMA]
[ 5.800146] lp0: using parport0 (interrupt-driven).
[ 5.897421] systemd-journald[352]: Received request to flush runtime journal from PID 1

```
dmesg komutunun çıktısı yüzlerce satır uzunluğunda olabilir, bu nedenle önceki liste, çekirdeğin systemd servis yöneticisini çağırdığını gösteren örnekleri içermektedir. Satırların başındaki değerler, çekirdek yüklenmeye başladığı andan itibaren geçen saniyeleri gösterir.

systemd tabanlı sistemlerde, `journalctl` komutu `-b`, `--boot`, `-k` veya `--dmesg` seçenekleri ile başlatma (initialization) mesajlarını gösterecektir. `journalctl --list-boots` komutu, geçerli önyükleme ile ilgili önyükleme numaralarının, kimlik karmaşıklıklarının ve ilk ile son karşılık gelen mesajların zaman damgalarını gösteren bir liste sunar:

```bash
$ journalctl --list-boots

 -4 9e5b3eb4952845208b841ad4dbefa1a6 Thu 2019-10-03 13:39:23 -03—Thu 2019-10-03 13:40:30 -03
 -3 9e3d79955535430aa43baa17758f40fa Thu 2019-10-03 13:41:15 -03—Thu 2019-10-03 14:56:19 -03
 -2 17672d8851694e6c9bb102df7355452c Thu 2019-10-03 14:56:57 -03—Thu 2019-10-03 19:27:16 -03
 -1 55c0d9439bfb4e85a20a62776d0dbb4d Thu 2019-10-03 19:27:53 -03—Fri 2019-10-04 00:28:47 -03
  0 08fbbebd9f964a74b8a02bb27b200622 Fri 2019-10-04 00:31:01 -03—Fri 2019-10-04 10:17:01 -03

```
systemd tabanlı sistemlerde önceki başlatma (initialization) kayıtları da saklanır, bu nedenle önceki işletim sisteminden oturumlarına ait mesajlar incelenebilir. `-b 0` veya `--boot=0` seçenekleri sağlanırsa, o zaman mevcut başlatma için mesajlar gösterilecektir. `-b -1` veya `--boot=-1` seçenekleri önceki başlatmadan gelen mesajları gösterecektir. `-b -2` veya `--boot=-2` seçenekleri ondan önceki başlatmadan gelen mesajları gösterecektir ve böyle devam eder. Aşağıdaki alıntı, çekirdeğin son başlatma süreci için systemd servis yöneticisini çağırdığını göstermektedir:

```bash
$ journalctl -b 0

oct 04 00:31:01 ubuntu-host kernel: EXT4-fs (sda1): mounted filesystem with ordered data
mode. Opts: (null)
oct 04 00:31:01 ubuntu-host kernel: ip_tables: (C) 2000-2006 Netfilter Core Team
oct 04 00:31:01 ubuntu-host systemd[1]: systemd 237 running in system mode.
oct 04 00:31:01 ubuntu-host systemd[1]: Detected architecture x86-64.
oct 04 00:31:01 ubuntu-host systemd[1]: Set hostname to <torre>.
oct 04 00:31:01 ubuntu-host systemd[1]: Reached target User and Group Name Lookups.
oct 04 00:31:01 ubuntu-host systemd[1]: Created slice System Slice.
oct 04 00:31:01 ubuntu-host systemd[1]: Listening on Journal Audit Socket.
oct 04 00:31:01 ubuntu-host systemd[1]: Listening on Journal Socket.
oct 04 00:31:01 ubuntu-host systemd[1]: Mounting POSIX Message Queue File System...
oct 04 00:31:01 ubuntu-host systemd[1]: Started Read required files in advance.
oct 04 00:31:01 ubuntu-host systemd[1]: Starting Load Kernel Modules...
oct 04 00:31:01 ubuntu-host kernel: EXT4-fs (sda1): re-mounted. Opts:
commit=300,barrier=0,errors=remount-ro
oct 04 00:31:01 ubuntu-host kernel: lp: driver loaded but no devices found
oct 04 00:31:01 ubuntu-host kernel: ppdev: user-space parallel port driver
oct 04 00:31:01 ubuntu-host kernel: parport_pc 00:02: reported by Plug and Play ACPI
oct 04 00:31:01 ubuntu-host kernel: parport0: PC-style at 0x378 (0x778), irq 7, dma 3
[PCSPP,TRISTATE,COMPAT,EPP,ECP,DMA]
oct 04 00:31:01 ubuntu-host kernel: lp0: using parport0 (interrupt-driven).
oct 04 00:31:01 ubuntu-host systemd-journald[352]: Journal started
oct 04 00:31:01 ubuntu-host systemd-journald[352]: Runtime journal
(/run/log/journal/abb765408f3741ae9519ab3b96063a15) is 4.9M, max 39.4M, 34.5M free.
oct 04 00:31:01 ubuntu-host systemd-modules-load[335]: Inserted module 'lp'
oct 04 00:31:01 ubuntu-host systemd-modules-load[335]: Inserted module 'ppdev'
oct 04 00:31:01 ubuntu-host systemd-modules-load[335]: Inserted module 'parport_pc'
oct 04 00:31:01 ubuntu-host systemd[1]: Starting Flush Journal to Persistent Storage...

```

İşletim sistemi tarafından başlatılan başlatma (initialization) ve diğer mesajlar, **/var/log/** dizini içindeki dosyalarda saklanır. Eğer kritik bir hata meydana gelirse ve işletim sistemi çekirdek ve initramfs'yi yükledikten sonra başlatma sürecine devam edemiyorsa, alternatif bir önyükleme ortamı kullanılarak sistem başlatılabilir ve ilgili dosya sistemine erişilebilir. Daha sonra, **/var/log/** altındaki dosyalar, başlatma sürecinin kesilmesine neden olan olası sebepler için araştırılabilir. journalctl komutunun `-D` veya `--directory` seçenekleri, systemd log mesajları için varsayılan konum olan **/var/log/journal/** dışındaki dizinlerdeki log mesajlarını okumak için kullanılabilir. systemd'nin log mesajları ham metin olarak saklanmadığından, bunları okumak için journalctl komutu gereklidir.

### Rehberli Egzersizler

1.  BIOS firmware'a sahip bir bilgisayarda, önyükleyici (bootstrap) ikili dosyası nerede bulunur?
    
2.  UEFI firmware, EFI uygulamaları olarak adlandırılan dış programlar tarafından sağlanan genişletilmiş özellikleri destekler. Ancak, bu uygulamaların kendi özel konumları vardır. EFI uygulamaları sistemde nerede bulunur?
    
3.  Önyükleyiciler, çekirdeği yüklemeden önce özel çekirdek parametrelerinin iletilmesine olanak tanır. Diyelim ki sistem, yanlış bilgilendirilmiş bir kök dosya sistemi konumu nedeniyle önyüklenemiyor. Doğru kök dosya sistemi, /dev/sda3 konumunda ise, bunu çekirdeğe bir parametre olarak nasıl iletebiliriz?
    
4.  Linux makinesinin önyükleme süreci şu mesajla sona erer:
    
    ```bash
    ALERT! /dev/sda3 does not exist. Dropping to a shell!
    
    ```
    
    Bu sorunun muhtemel nedeni nedir?
    

### Keşif Egzersizleri

1.  Bootloader, makinede birden fazla işletim sistemi yüklü olduğunda seçenek sunar. Ancak, yeni yüklenen bir işletim sistemi, sabit diskin MBR'sini üzerine yazabilir, önyükleyicinin ilk aşamasını silebilir ve diğer işletim sistemine erişilemez hale getirebilir. Bu, bir UEFI firmware'a sahip bir makinede neden gerçekleşmeyebilir?
2.  Uygun bir initramfs görüntüsü sağlanmadan özel bir çekirdek kurmanın yaygın bir sonucu nedir?
3.  Başlatma kaydı yüzlerce satır uzunluğunda olduğu için dmesg komutunun çıktısı genellikle bir sayfa komutuna - örneğin less komutuna - yönlendirilir, okumayı kolaylaştırmak için. dmesg komutuna, çıktısını otomatik olarak sayfalayan ve açıkça bir sayfa komutu kullanma ihtiyacını ortadan kaldıran hangi seçenek verilir?
5.  Çevrimdışı bir makinenin tüm dosya sistemini içeren bir sabit disk, çıkarıldı ve çalışan bir makineye ikincil bir sürücü olarak takıldı. Varsayalım ki bu sürücünün bağlama noktası /mnt/hd, journalctl komutu nasıl kullanılır ki /mnt/hd/var/log/journal/ konumundaki günlük dosyalarının içeriğini incelesin?

### Özet

Bu ders, standart bir Linux sisteminin önyükleme sırasını kapsar. Linux sisteminin önyükleme sürecinin nasıl çalıştığına dair doğru bilgi, sistemi erişilemez hale getirebilecek hataları önlemeye yardımcı olur. Ders, aşağıdaki konu alanlarını kapsar:

-   BIOS ve UEFI önyükleme yöntemlerinin nasıl farklılık gösterdiği.
-   Tipik sistem başlatma aşamaları.
-   Önyükleme mesajlarını kurtarma. İlgilenilen komutlar ve prosedürler şunlardır:
-   Ortak çekirdek parametreleri.
-   Önyükleme mesajlarını okumak için kullanılan komutlar: **dmesg** ve **journalctl**.

### Rehberli Egzersizlere Cevaplar

1.  BIOS firmware'a sahip bir makinede, önyükleyici (bootstrap) ikili dosyası nerede bulunur? İlk depolama cihazının MBR'sinde, BIOS yapılandırma yardımcı programında tanımlandığı şekilde.
    
2.  UEFI firmware, EFI uygulamaları olarak adlandırılan dış programlar tarafından sağlanan genişletilmiş özellikleri destekler. Ancak, bu uygulamaların kendi özel konumları vardır. EFI uygulamaları, genellikle FAT32 dosya sistemine sahip uyumlu bir depolama bloğunda bulunan EFI Sistem Bölümü'nde (ESP) saklanır.
    
3.  Önyükleyiciler, çekirdeği yüklemesinden önce özel çekirdek parametrelerini iletmeye izin verir. Diyelim ki sistem, yanlış bilgilendirilmiş bir kök dosya sistemi konumu nedeniyle önyüklenemiyor. Doğru kök dosya sistemi, /dev/sda3 konumunda ise, bunu çekirdeğe bir parametre olarak iletmek için hangi parametre kullanılmalıdır? root parametresi kullanılmalıdır, yani root=/dev/sda3.
    
4.  Bir Linux makinesinin önyükleme süreci aşağıdaki mesajla sona erer:
    
    ```bash
    ALERT! /dev/sda3 does not exist. Dropping to a shell!
    
    ```
    
    Bu sorunun muhtemel nedeni nedir? Çekirdek, /dev/sda3 olarak belirtilen kök dosya sistemini bulamadı.
    

### Keşif Alıştırmaları Cevaplar

1.  Önyükleyici (bootloader), makinede birden fazla işletim sistemi yüklü olduğunda seçenek sunar. Ancak, yeni yüklenen bir işletim sistemi, sabit diskin MBR'sini üzerine yazabilir, önyükleyicinin ilk aşamasını silebilir ve diğer işletim sistemine erişilemez hale getirebilir. Bu durum, UEFI firmware'a sahip bir makinede neden gerçekleşmez? UEFI makineleri, önyükleyicinin ilk aşamasını depolamak için sabit diskin MBR'sini kullanmaz.
2.  Uygun bir initramfs görüntüsü sağlanmadan özel bir çekirdek kurmanın yaygın bir sonucu nedir? Kök dosya sistemi, dış çekirdek modülü olarak derlendiyse erişilemez olabilir.
3.  Başlatma kaydı yüzlerce satır uzunluğunda olduğu için dmesg komutunun çıktısı genellikle bir sayfa komutuna - örneğin less komutuna - yönlendirilir, okumayı kolaylaştırmak için. dmesg komutuna, çıktısını otomatik olarak sayfalayan ve açıkça bir sayfa komutu kullanma ihtiyacını ortadan kaldıran hangi seçenek verilir? dmesg -H veya dmesg --human komutları, sayfalandırmayı varsayılan olarak etkinleştirir.
4.  Çevrimdışı bir makinenin tüm dosya sistemini içeren bir sabit disk, çalışan bir makineye ikincil bir sürücü olarak takıldı. Varsayalım ki bu sürücünün bağlama noktası /mnt/hd ise, journalctl komutu, /mnt/hd/var/log/journal/ konumundaki günlük dosyalarının içeriğini incelemek için nasıl kullanılır? journalctl -D /mnt/hd/var/log/journal veya journalctl --directory=/mnt/hd/var/log/journal komutlarıyla kullanılır.