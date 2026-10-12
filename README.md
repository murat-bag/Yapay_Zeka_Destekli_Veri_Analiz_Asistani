# Yapay Zeka Destekli Veri Analiz Asistanı

## Projenin Amacı
Bir veritabanına bağlanan, tablo ve kolonların ne anlama geldiğini kendi eğittiğimiz derin öğrenme modelleriyle anlayan, günlük Türkçe sorulara göre analiz yapıp tablo/grafik üreten ve cevabı doğal dille cevap veren bir sistemdir. 

## Sistem Akışı
Veritabanı -> Model 1 (Şema anlama) -> Model 2 (Analiz + grafik/tablo planı) -> Model 3 (Soru-cevap dil modeli + kontrol) -> Arayüz


## İş Paylaşımı
Murat -> Model 1, Model 2 ve Arayüz Tasarımı
Görev: 
Tablo ve kolonların anlamını çıkarır (Model 1).
Bu anlam haritası ve istekle hangi analizin, hangi kolonlarla, hangi grafik/tabloyla yapılacağına karar verip planı çalıştırır (Model 2).
Veritabanı bağlantı ekranı, veri profili/anomali raporu ekranı, grafik ve tablo paneli, sohbet penceresi (Arayüz).


Canpolat -> Model 3 ve Arayüz Tasarımı
Görev:
Günlük Türkçe soruyu yapılandırılmış isteğe çevirir. Model 2'nin sonucunu doğal Türkçe cevaba çevirir, cevabı kontrol eder (Model 3).
Veritabanı bağlantı ekranı, veri profili/anomali raporu ekranı, grafik ve tablo paneli, sohbet penceresi (Arayüz).
