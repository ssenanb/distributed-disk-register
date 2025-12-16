# Dağıtık Mesaj Kayıt Servisi 
Bu proje; birden fazla sunucunun bir küme halinde çalıştığı bir hibrit iletişim altyapısıdır. Sistem dış dünyadan gelen standart TCP metin mesajlarını merkezi bir lider sunucu (gateway) üzerinden kabul eder. Alınan bu veriler küme içerisindeki sunucular arasında gRPC ve Protobuf teknolojileri kullanılarak tüm üyelere anlık olarak dağıtılır (broadcast).

__1) TCP Mesajlarının Alınması ve Parse Edilemsi__

