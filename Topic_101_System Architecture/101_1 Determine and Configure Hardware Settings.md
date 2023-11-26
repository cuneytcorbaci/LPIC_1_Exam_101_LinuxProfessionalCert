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

### **Giriş**

Elektronik bilgisayarların ilk yıllarından itibaren, iş ve kişisel bilgisayar üreticileri makinelerinde çeşitli donanım parçalarını entegre etmişlerdir. Bu parçalar da sırasıyla işletim sistemleri tarafından desteklenmelidir. Talimat setleri ve cihaz iletişimine yönelik standartlar endüstri tarafından oluşturulmadıkça, bu durum işletim sistemi geliştiricisi açısından çok zor olabilir. Bir uygulamaya işletim sistemi tarafından sağlanan standartlaştırılmış soyutlama katmanına benzer şekilde, bu standartlar, belirli bir donanım modeline bağlı olmayan bir işletim sistemi yazmayı ve sürdürmeyi daha kolay hale getirir. Ancak, temel donanımın karmaşıklığı, bazen kaynakların işletim sistemine nasıl sunulması gerektiğinde ayarlamalar yapmayı gerektirir; böylece doğru bir şekilde yüklenip çalışabilir.

Bazı ayarlamalar, kurulu bir işletim sistemi olmadan bile yapılabilir. Birçok makine, makine açılırken yürütülebilen bir yapılandırma yardımcı programı sunar. 2000'li yılların ortalarına kadar yapılandırma yardımcı programı, x86 anakartlarda bulunan temel yapılandırma rutinlerini içeren firmware için standart olan BIOS'ta (Temel Giriş/Çıkış Sistemi) uygulanmıştı. 2000'li yılların ilk on yılı sonlarından itibaren, x86 mimarisine dayalı makineler, BIOS'un yerine daha sofistike özelliklere sahip olan UEFI (Birleşik Genişletilebilir Firmware Arabirimi) adlı yeni bir uygulama ile değişmeye başladı. Ancak bu değişikliğe rağmen, her iki uygulama da temelde aynı amacı yerine getirdiği için yapılandırma yardımcı programına hala BIOS adı vermek yaygındır.

**Not**

BIOS ve UEFI arasındaki benzerlikler ve farklar hakkındaki daha fazla detay, sonraki bir ders kapsamında ele alınacaktır.