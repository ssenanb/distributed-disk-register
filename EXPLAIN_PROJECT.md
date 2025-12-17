# Dağıtık Mesaj Kayıt Servisi 
Bu proje; birden fazla sunucunun bir küme halinde çalıştığı bir hibrit iletişim altyapısıdır. Sistem dış dünyadan gelen standart TCP metin mesajlarını merkezi bir lider sunucu (gateway) üzerinden kabul eder. Alınan bu veriler küme içerisindeki sunucular arasında gRPC ve Protobuf teknolojileri kullanılarak tüm üyelere anlık olarak dağıtılır (broadcast).

__1) TCP Server İstemciden Gelen Mesajları Pars Etme__

İstemci __SET <message_id> <message>__ ve __GET <message_id>__ olmak üzere text tabanlı iki istek yapabilmektedir. İstemciden gelen verileri parse ederek komutun türü (SET veya GET) ve beraberindeki parametreler (ID, Mesaj) birbirinden ayrıştırılır. Bu veriler bir map içinde tutulur. 

__SET:__ Verilen anahtar ile veriyi eşleştirip sisteme kaydeder ve geriye başarı onayı (OK) döner.

__GET:__ Belirtilen anahtara ait veriyi sorgular ve veri varsa değeri, yoksa hata mesajını (NOT_FOUND) döndürür.

Bu kod, sistemin "Lider" düğümü üzerinde çalışarak istemciden gelen talepleri karşılayan ana arayüz görevini görür.

__2) Diskte Mesaj Saklama (Buffered/Unbuffered IO Yaklaşımı)__  

Bu aşamada mesajların geçici bellek yerine kalıcı olarak disk üzerinde saklanması amaçlanmıştır. Sistem, SET ve GET komutları aracılığıyla her mesajı kendi ID'siyle ayrı bir dosya olarak oluşturmakta ve okumaktadır.

Mesajların diske yazılması ve diskten okunması sürecinde Buffered IO yaklaşımı kullanılmıştır. BufferedWriter ve BufferedReader sınıfları kullanılarak verileri önce bellek üzerindeki bir tamponda biriktirip toplu halde diske yazdığı için disk işlem saysının minimize edilmesi hedeflenmiştir. Bu yaklaşım sık ve küçük veri işlemlerinde disk I/O maliyetini azaltarak performans artışı sağlamaktadır. 

Unbuffered yaklaşımda ise FileOutputStream ve FileInputStream kullanılarak veriler byte tabanlı olarak doğrudan işletim sistemine iletilmektedir. Bu yöntem düşük seviyeli kontrol gerektiren durumlar için uygundur ve sık yapılan okuma, yazma işlemlerinde buffered yaklaşıma kıyasla daha fazla sistem çağrısına neden olmaktadır. Disk erişim sayısını azaltarak performans artışı sağlaması nedeniyle projede buffered IO yaklaşımı tercih edilmiştir. 

__3) gRPC Mesaj Modeli (Protobuf Nesnesi)__

Lider ve üyelerin birbirleriyle iletişim kurarken kullandıkları veri paketlerinin standartlaştırılması ve Java sınıflarının Protobuf (Protocol Buffers) üzerinden otomatik üretilmesi sağlanır. İletişim __family.proto__ dosyasında tanımlanan yapılandırılmış nesneler üzerinden yürütülür. 

Sistem dış dünya ile iç dünya arasında farklı diller konuşur:

__İstemci ↔ Lider:__ Basit ve okunabilir text tabanlı iletişim.

__Lider ↔ Üyeler:__ Yüksek performanslı .protobuf nesneleri.

Protobuf kullanımı ağ trafiğini minimize eder ve serileştirme hızını artırarak düğümler arası iletişimi optimize eder.

__StoredMessage:__ Diskte saklanacak veriyi temsil eden ve içerisinde ID ve text alanlarını barındıran yapıdır. Veri bütünlüğünü sağlamak için tek bir paket halinde kapsüllenir.

__Store (RPC):__ Liderin gönderdiği StoredMessage nesnesini üye düğümün diskine kaydetmesini sağlayan çağrı metodudur.

__Retrieve (RPC):__ Belirtilen ID'ye sahip verinin üye düğümden okunup Lidere geri döndürülmesini sağlayan sorgu metodudur.

__4) Tolerance=1 ve 2 için Dağıtık Kayıt__

Bu aşamada sisteme yedekleme mekanizması eklenerek veriye erişilebilirlik sağlanmıştır. Sistem __tolerance.conf__ dosyasındaki TOLERANCE değerine göre dinamik olarak şekillenir. Ayrıca lider "mesaj id hangi üyelerde var" bilgisini __Map<Integer, List<MemberId>>__ tipinde bir map'te tutar.

* TOLERANCE=1: Veri, lider dışında 1 yedek üyede tutulur. Toplamda 2 kayıt yapılır.

* TOLERANCE=2: Veri, lider dışında 2 yedek üyede tutulur. Toplamda 3 kayıt yapılır.

__SET Akışı (Veri Kaydetme)__

Lider bir SET isteği alır ve protobuf formatına dönüştürür sonrasında şu adımları gerçekleştirir:

1) Mesajı kendi diskine yazar ve bellek haritasına ekler.

2) Aktif üyeler arasından TOLERANCE sayısı kadar üye seçer.

3) Seçilen üyelere gRPC üzerinden __Store()__ isteği gönderir (Dönüştürülen nesne gönderilir).

4) Tüm yedeklerden başarılı yanıt gelirse istemciye __OK__ döner.

5) Herhangi bir yedek başarısız olursa istemciye hata döner __(NOT_FOUND)__.

__GET Akışı (Veri Okuma)__

Lider bir GET isteği aldığında şu adımları gerçekleştirir:

1) Lider veriyi önce kendi diskinden okumaya çalışır.

2) Eğer veri liderde yoksa liste üzerinden verinin kopyalandığı üyeleri belirler.

3) Listedeki üyelere sırayla __Retrieve()__ isteği atar. İlk başarılı yanıt istemciye iletilir.

__Örnek ile Kod İşleyişini Anlama__

TOLERANCE değeri 2 olarak ayarlandı ve biri lider diğer dördü üye olacak şekilde sistem başlatıldı. Lider 5555 portundan başlatıldı ve her yeni üye eklendiğinde üylerin port numarası 5556, 5557, 5558, 5559 olacak şekilde oluşturuldu.

<img src="https://github.com/ssenanb/distributed-disk-register/blob/main/leader_and_members" alt="Lider ve Üyeler" width="900"/>

Mesaj gönderimi için 6666 portundan GET ve SET komutları gönderildi. Her SET isteğinde belrtilen ID ve mesaj kaydedildi, diske yazıldı. Başarılı olması durumunda OK döndürüldü. GET isteği geldiğinde kaydedilen ID'nin mesajı konsola bastırıldı. Yanlış ve eksik komut gönderiminde NOT_FOUND döndürülür. Bu örnekte gönderilen komutlar şu şekildedir:

__SET 100 hello_world__

__GET 100__

<img src="https://github.com/ssenanb/distributed-disk-register/blob/main/commands_send" alt="Komut Gönderimi" width="900"/>

TOLERANCE değeri 2 olduğu için SET mesajı ile gönderilen mesaj hem lidere hem de 2 üyeye kaydedildi. Bu örnek için lider olan 5555 portuna, üye_1 olan 5556 portuna ve üye_2 olan 5557 portuna mesaj kaydedildi. İstemci ve lider arasında text tabanlı bir iletişim kurulurken üyeler ve lider arasında protobuf neseneleri ile iletişim kurulmuştur.

<img src="https://github.com/ssenanb/distributed-disk-register/blob/main/gRPC_send" alt="gRPC Gönderimi" width="900"/>

Bu işlem sonunda lider ve her üye mesajı Buffered IO ile diske yazmıştır. Kaydedilen ID'nin adı ile .msg dosyaları oluşmuştur. Aşağıda dosya biçimi ve içerikleri gösterilmiştir:

<img src="https://github.com/ssenanb/distributed-disk-register/blob/main/message_files" alt="Mesaj Dosyaları" width="500"/>

<img src="https://github.com/ssenanb/distributed-disk-register/blob/main/message_files_content" alt="Mesaj Dosya İçeriği" width="900"/>

__5) Hata Toleransı n (Genel Hâl) ve Load Balancing__

__6) Crash Senaryoları ve Recovery__

Burada sistemin dayanıklılığı test edilmiştir. Amaç, verilerin bulunduğu düğümlerden biri veya birkaçı çöktüğünde (crash), lider düğümün bu durumu fark edip kesinti yaşamadan veriyi hayatta kalan diğer düğümlerden getirebildiğini kanıtlamaktır.

1- Lider, verinin hangi düğümlerde olduğunu bilir (messageLocations Map).
2- GET isteği geldiğinde listedeki ilk üyeye bağlanmayı dener.
3- Eğer üye çökmüşse, lider hatayı yakalar (catch), loglar ve kullanıcıya hissettirmeden listedeki bir sonraki üyeye geçer.

__Test Senaryosu 1:__

1'i lider olmak üzere 4 üye ve Tolerance değeri 2 olarak sistem başlatıldı. İstemci aracılığıyla SET 500 TestVerisi komutu gönderilerek verinin Lider haricinde iki üyeye daha (Örn: Port 5557 ve 5558) kopyalanması sağlandı.



![test aşamaları] https://github.com/ssenanb/distributed-disk-register/blob/main/testsenaryosu.png













