# Dağıtık Mesaj Kayıt Servisi 
Bu proje; birden fazla sunucunun bir küme halinde çalıştığı bir hibrit iletişim altyapısıdır. Sistem dış dünyadan gelen standart TCP metin mesajlarını merkezi bir lider sunucu (gateway) üzerinden kabul eder. Alınan bu veriler küme içerisindeki sunucular arasında gRPC ve Protobuf teknolojileri kullanılarak tüm üyelere anlık olarak dağıtılır (broadcast).

__1) TCP Server İstemciden Gelen Mesajları Pars Etme__

İstemci __SET <message_id> <message>__ ve __GET <message_id>__ olmak üzere text tabanlı iki istek yapabilmektedir. İstemciden gelen verileri parse ederek komutun türü (SET veya GET) ve beraberindeki parametreler (ID, Mesaj) birbirinden ayrıştırılır. Bu veriler bir liste içinde tutulur. 

__SET:__ Verilen anahtar ile veriyi eşleştirip sisteme kaydeder ve geriye başarı onayı (OK) döner.

__GET:__ Belirtilen anahtara ait veriyi sorgular ve veri varsa değeri, yoksa hata mesajını (NOT_FOUND) döndürür.

Bu kod, sistemin "Lider" düğümü üzerinde çalışarak istemciden gelen talepleri karşılayan ana arayüz görevini görür.

__2) Diskte Mesaj Saklama (Buffered/Unbuffered IO Yaklaşımı)__
Bu aşamada mesajların geçici bellek yerine kalıcı olarak disk üzerinde saklanması amaçlanmıştır. Sistem, SET ve GET komutları aracılığıyla her mesajı kendi ID'siyle ayrı bir dosya olarak oluşturmakta ve okumaktadır. Mesajların diske yazılması ve diskten okunması sürecinde Buffered IO yaklaşımı kullanılmıştır. BufferedWriter ve BufferedReader sınıfları kullanılarak verileri önce bellek üzerindeki bir tamponda biriktirip toplu halde diske yazdığı için disk işlem saysının minimize edilmesi hedeflenmiştir. Bu yaklaşım sık ve küçük veri işlemlerinde disk I/O maliyetini azaltarak performans artışı sağlamaktadır. Unbuffered yaklaşımda ise FileOutputStream ve FileInputStream kullanılarak veriler byte tabanlı olarak doğrudan işletim sistemine iletilmektedir. Bu yöntem düşük seviyeli kontrol gerektiren durumlar için uygundur ve sık yapılan okuma, yazma işlemlerinde buffered yaklaşıma kıyasla daha fazla sistem çağrısına neden olmaktadır. Disk erişim sayısını azaltarak performans artışı sağlaması nedeniyle projede buffered IO yaklaşımı tercih edilmiştir. 

__3) gRPC Mesaj Modeli (Protobuf Nesnesi)__

Lider ve üyelerin birbirleriyle iletişim kurarken kullandıkları veri paketlerinin standartlaştırılması ve Java sınıflarının Protobuf (Protocol Buffers) üzerinden otomatik üretilmesi sağlanır. İletişim __family.proto__ dosyasında tanımlanan yapılandırılmış nesneler üzerinden yürütülür. 

__StoredMessage:__ Diskte saklanacak veriyi temsil eden ve içerisinde ID ve Text alanlarını barındıran yapıdır. Veri bütünlüğünü sağlamak için tek bir paket halinde kapsüllenir.

__Store (RPC):__ Liderin gönderdiği StoredMessage nesnesini üye düğümün diskine kaydetmesini sağlayan çağrı metodudur.

__Retrieve (RPC):__ Belirtilen ID'ye sahip verinin üye düğümden okunup Lidere geri döndürülmesini sağlayan sorgu metodudur.

__4) Tolerance=1 ve 2 için Dağıtık Kayıt__

__5) Hata Toleransı n (Genel Hâl) ve Load Balancing__

__6) Crash Senaryoları ve Recovery__




