# Dağıtık Mesaj Kayıt Servisi 
Bu proje; birden fazla sunucunun bir küme halinde çalıştığı bir hibrit iletişim altyapısıdır. Sistem dış dünyadan gelen standart TCP metin mesajlarını merkezi bir lider sunucu (gateway) üzerinden kabul eder. Alınan bu veriler küme içerisindeki sunucular arasında gRPC ve Protobuf teknolojileri kullanılarak tüm üyelere anlık olarak dağıtılır (broadcast).

__1) TCP Server İstemciden Gelen Mesajları Pars Etme__

İstemci __SET <message_id> <message>__ ve __GET <message_id>__ olmak üzere text tabanlı iki istek yapabilmektedir.İstemciden gelen verileri parse ederek komutun türü (SET veya GET) ve beraberindeki parametreler (ID, Mesaj) birbirinden ayrıştırılır. Bu veriler bir liste içinde tutulur. 

__SET:__ Verilen anahtar ile veriyi eşleştirip sisteme kaydeder ve geriye başarı onayı (OK) döner.

__GET:__ Belirtilen anahtara ait veriyi sorgular ve veri varsa değeri, yoksa hata mesajını (NOT_FOUND) döndürür.

Bu kod, sistemin "Lider" düğümü üzerinde çalışarak istemciden gelen talepleri karşılayan ana arayüz görevini görür.

__2) Diskte Mesaj Saklama (Buffered/Unbuffered IO Yaklaşımı)__

__3) gRPC Mesaj Modeli (Protobuf Nesnesi)__

__4) Tolerance=1 ve 2 için Dağıtık Kayıt__

__5) Hata Toleransı n (Genel Hâl) ve Load Balancing__

__6) Crash Senaryoları ve Recovery__




