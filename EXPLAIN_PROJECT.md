# Dağıtık Mesaj Kayıt Servisi 
Bu proje; birden fazla sunucunun bir küme halinde çalıştığı bir hibrit iletişim altyapısıdır. Sistem dış dünyadan gelen standart TCP metin mesajlarını merkezi bir lider sunucu (gateway) üzerinden kabul eder. Alınan bu veriler küme içerisindeki sunucular arasında gRPC ve Protobuf teknolojileri kullanılarak tüm üyelere anlık olarak dağıtılır (broadcast).

__1) TCP Server İstemciden Gelen Mesajları Pars Etme__

__2)Diskte Mesaj Saklama (Buffered/Unbuffered IO Yaklaşımı)__
__3) gRPC Mesaj Modeli (Protobuf Nesnesi)__
__4) Tolerance=1 ve 2 için Dağıtık Kayıt__
__5) Hata Toleransı n (Genel Hâl) ve Load Balancing__
__6) Crash Senaryoları ve Recovery__




