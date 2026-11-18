
## 1. Şema haritası (Model 1 çıktısı)

```json
{
  "veritabani": "magaza.db",
  "tablolar": [
    {
      "tablo": "siparis",
      "satir_sayisi": 15230,
      "kolonlar": [
        {"ad": "siparis_id", "anlam": "id", "tip": "tam_sayi", "eksik_oran": 0.00, "anomali": false},
        {"ad": "mus_id", "anlam": "id", "tip": "tam_sayi", "eksik_oran": 0.00, "anomali": false},
        {"ad": "sat_tut", "anlam": "para_tutari", "tip": "ondalikli", "eksik_oran": 0.12, "anomali": true, "anomali_sayisi": 37},
        {"ad": "tarih", "anlam": "tarih", "tip": "tarih", "eksik_oran": 0.01, "anomali": false},
        {"ad": "sehir", "anlam": "sehir", "tip": "metin", "eksik_oran": 0.05, "anomali": false}
      ],
      "iliskiler": [
        {"kolon": "mus_id", "hedef_tablo": "musteri", "hedef_kolon": "id", "skor": 0.94}
      ]
    },
    {
      "tablo": "musteri",
      "satir_sayisi": 2100,
      "kolonlar": [
        {"ad": "id", "anlam": "id", "tip": "tam_sayi", "eksik_oran": 0.00, "anomali": false},
        {"ad": "yas", "anlam": "yas", "tip": "tam_sayi", "eksik_oran": 0.03, "anomali": true, "anomali_sayisi": 4}
      ],
      "iliskiler": []
    }
  ]
}
```

| Alan | Açıklama |
|---|---|
| `veritabani` | Kaynağın adı (veritabanı ya da dosya adı) |
| `tablolar[].tablo` | Tablo adı (CSV gibi tek tablolu kaynaklarda dosya adı) |
| `tablolar[].satir_sayisi` | Tablodaki satır sayısı |
| `kolonlar[].ad` | Kolonun kaynaktaki adı |
| `kolonlar[].anlam` | Model 1'in tahmin ettiği anlamsal tür (etiket listesi aşağıda) |
| `kolonlar[].tip` | Veri tipi |
| `kolonlar[].eksik_oran` | Boş değer oranı (0-1) |
| `kolonlar[].anomali` | Aykırı değer var mı (true/false) |
| `kolonlar[].anomali_sayisi` | Yalnızca `anomali: true` ise bulunur |
| `iliskiler[]` | Tablolar arası tahmin edilen ilişkiler |
| `iliskiler[].skor` | İlişki tahmininin güveni (0-1) |

## 2. İstek (Model 3 çıktısı, Model 2 girdisi)

Kullanıcının günlük Türkçe sorusu yapılandırılmış isteğe çevrilir.

```json
{"intent": "toplam", "hedef": "sat_tut", "filtre": {}}
```

```json
{"intent": "trend", "hedef": "sat_tut", "zaman": "tarih", "aralik": "ay", "filtre": {"yil": 2024}}
```

```json
{"intent": "en_cok", "hedef": "adet", "grupla": "urun_adi", "sirala": "azalan", "limit": 5}
```

```json
{"intent": "ortalama", "hedef": "sat_tut", "grupla": "sehir"}
```

```json
{"intent": "korelasyon", "kolonlar": ["reklam_butcesi", "sat_tut"]}
```

```json
{"intent": "dagilim", "hedef": "yas", "kutu_sayisi": 10}
```

```json
{"intent": "karsilastir", "hedef": "sat_tut", "grupla": "yil", "degerler": [2023, 2024]}
```

Takip sorusu ("peki geçen yıl?"):

```json
{"intent": "trend", "hedef": "sat_tut", "zaman": "tarih", "aralik": "ay", "filtre": {"yil": "gecen_yil"}, "onceki_istekten_devam": true}
```

Rapor isteği ("rapor hazırla"):

```json
{"intent": "rapor", "kapsam": "tum_veri"}
```

```json
{"intent": "rapor", "kapsam": "secili", "bolumler": ["trend", "dagilim"]}
```

| Alan | Açıklama |
|---|---|
| `intent` | Zorunlu. Analizin türü (liste aşağıda) |
| `hedef` | Üzerinde hesap yapılacak kolon |
| `grupla` | Gruplama kolonu |
| `zaman` | Zaman kolonu |
| `aralik` | Zaman aralığı (ay, yil vb.) |
| `filtre` | Süzme koşulları, sözlük (boş olabilir) |
| `sirala` | `azalan` veya `artan` |
| `limit` | Kaç satır istendiği |
| `kutu_sayisi` | Dağılım için aralık sayısı |
| `kolonlar` | Birden fazla kolon gereken analizler (korelasyon) |
| `degerler` | Karşılaştırılacak değerler |
| `onceki_istekten_devam` | Takip sorusu ise `true` |
| `kapsam` | Rapor için: `tum_veri` veya `secili` |
| `bolumler` | `kapsam: secili` ise istenen bölümler |

## 3. Analiz sonucu (Model 2 çıktısı)

Grafikli sonuç:

```json
{
  "durum": "tamam",
  "plan": {"analiz": "zaman_serisi_toplam", "kolonlar": {"x": "tarih", "y": "sat_tut"}},
  "sonuc": {"kolonlar": ["ay", "toplam"], "satirlar": [["Ocak", 1250], ["Şubat", 980], ["Mart", 1410]]},
  "grafik": {"tip": "cizgi", "x": "ay", "y": "toplam", "baslik": "2024 aylık satış"},
  "uyarilar": ["sat_tut kolonunun %12'si boş, sonuç kalan veriyle hesaplandı"]
}
```

Çubuk grafik:

```json
{
  "durum": "tamam",
  "sonuc": {"kolonlar": ["urun_adi", "adet"], "satirlar": [["Kalem", 5400], ["Defter", 4100]]},
  "grafik": {"tip": "cubuk", "x": "urun_adi", "y": "adet", "baslik": "En çok satan ürünler"}
}
```

Dağılım grafiği (korelasyon):

```json
{
  "durum": "tamam",
  "sonuc": {"korelasyon": 0.82, "n": 1180},
  "grafik": {"tip": "dagilim", "x": "reklam_butcesi", "y": "sat_tut"}
}
```

Sadece tablo (grafik gerekmeyen sonuç):

```json
{
  "durum": "tamam",
  "sonuc": {"kolonlar": ["toplam"], "satirlar": [[1284500.75]]},
  "grafik": null
}
```

Hata (veride böyle bir kolon yok):

```json
{
  "durum": "hata",
  "hata": {"kod": "KOLON_YOK", "mesaj": "'kar_marji' kolonu bulunamadı", "oneri": ["sat_tut", "maliyet"]}
}
```

| Alan | Açıklama |
|---|---|
| `durum` | `tamam` veya `hata` |
| `plan` | Model 2'nin seçtiği analiz ve kolonlar (isteğe bağlı) |
| `sonuc` | Tablo: `{kolonlar, satirlar}`. Tek değerli sonuçlarda (korelasyon gibi) sözlük olabilir |
| `grafik` | `{tip, x, y, baslik}` ya da `null` |
| `uyarilar` | Eksik/anormal veri uyarıları (isteğe bağlı) |
| `hata` | Yalnızca `durum: hata` ise: `kod`, `mesaj`, `oneri` |

## 4. Son cevap (Model 3 çıktısı)

Normal cevap:

```json
{
  "soru": "Hangi ay en çok satış yaptık?",
  "cevap_metni": "En çok satış Mart ayında, toplam 1.410 TL ile yapıldı.",
  "durum": "dogrulandi",
  "grafik_goster": true
}
```

Eksik veri uyarılı cevap:

```json
{
  "soru": "Toplam satışımız ne kadar?",
  "cevap_metni": "Toplam satış 1.284.500 TL. Not: satış kolonunun %12'si boş olduğu için sonuç kalan kayıtlarla hesaplandı.",
  "durum": "dogrulandi_uyarili",
  "grafik_goster": false
}
```

Veride olmayan bilgi:

```json
{
  "soru": "Kâr marjımız nedir?",
  "cevap_metni": "Bu bilgi veride bulunmuyor. Satış tutarı ve maliyet kolonları üzerinden bir analiz yapabilirim.",
  "durum": "veride_yok"
}
```

Belirsiz soru (geri sorma):

```json
{
  "soru": "Satışlar nasıl gidiyor?",
  "cevap_metni": "Hangi dönemi ve hangi tutarı kastediyorsunuz: sipariş tutarı mı, net tutar mı?",
  "durum": "belirsiz",
  "secenekler": ["sat_tut", "net_tut"]
}
```

Doğrulama başarısız, cevap yeniden üretildi:

```json
{
  "soru": "En çok satan ürün?",
  "cevap_metni": "En çok satan ürün Kalem, toplam 5.400 adet.",
  "durum": "yeniden_uretildi",
  "deneme": 2
}
```

| `durum` | Anlamı | Ek alan |
|---|---|---|
| `dogrulandi` | Cevap sonuçla tutarlı | `grafik_goster` |
| `dogrulandi_uyarili` | Tutarlı, eksik veri uyarısı eklendi | `grafik_goster` |
| `veride_yok` | İstenen kolon/bilgi veride yok | - |
| `belirsiz` | Soru net değil, geri soruldu | `secenekler` |
| `yeniden_uretildi` | Doğrulama başarısız, cevap yeniden üretildi | `deneme` |

`cevap_metni` her durumda bulunur.

## 5. Veri profili (otomatik analiz)

Veri seti yüklenince, kullanıcı soru sormadan üretilir. Arayüzün "veri profili ve
anomali raporu" ekranı bunu gösterir. İstatistikleri profilleyici (pandas) hesaplar,
Model 1 yalnızca `anlam` etiketini tahmin eder.

```json
{
  "kaynak": {"ad": "satislar.csv", "tur": "csv"},
  "genel": {
    "satir_sayisi": 15230,
    "kolon_sayisi": 5,
    "bos_hucre_orani": 0.036,
    "tekrar_eden_satir": 12
  },
  "kolonlar": [
    {
      "ad": "sat_tut", "anlam": "para_tutari", "tip": "ondalikli",
      "eksik_oran": 0.12, "anomali": true, "anomali_sayisi": 37,
      "istatistik": {"min": -999, "max": 98000, "ortalama": 420.3, "medyan": 310.0, "std": 512.4},
      "anomali_ornekleri": [-999, 98000]
    },
    {
      "ad": "sehir", "anlam": "sehir", "tip": "metin",
      "eksik_oran": 0.05, "anomali": false,
      "istatistik": {"benzersiz_sayisi": 81, "en_sik": [["Kayseri", 2100], ["Ankara", 1800]]}
    }
  ],
  "korelasyonlar": [
    {"kolon_a": "reklam_butcesi", "kolon_b": "sat_tut", "deger": 0.82}
  ],
  "onerilen_grafikler": [
    {
      "grafik": {"tip": "cubuk", "x": "sehir", "y": "adet", "baslik": "Şehirlere göre sipariş sayısı"},
      "sonuc": {"kolonlar": ["sehir", "adet"], "satirlar": [["Kayseri", 2100], ["Ankara", 1800]]}
    },
    {
      "grafik": {"tip": "histogram", "x": "sat_tut", "y": "adet", "baslik": "Sipariş tutarı dağılımı"},
      "sonuc": {"kolonlar": ["aralik", "adet"], "satirlar": [["0-500", 9000], ["500-1000", 4100]]}
    }
  ],
  "uyarilar": [
    "sat_tut kolonunun %12'si boş",
    "sat_tut kolonunda 37 aykırı değer var"
  ]
}
```

| Alan | Açıklama |
|---|---|
| `kaynak` | Yüklenen dosya (`tur`: csv, json, veritabani) |
| `genel` | Tüm veriye ait özet sayılar |
| `kolonlar[]` | Format 1'deki alanlar + `istatistik` |
| `istatistik` | Sayısal kolon: min, max, ortalama, medyan, std. Metin kolonu: benzersiz_sayisi, en_sik |
| `anomali_ornekleri` | Aykırı bulunan değerlerden birkaç örnek |
| `korelasyonlar[]` | Sayısal kolon çiftlerinin korelasyonu (-1 ile 1) |
| `onerilen_grafikler[]` | Her eleman Format 3'teki `grafik` + `sonuc` ikilisidir |
| `uyarilar` | Eksik/anormal veri uyarıları |

## 6. Rapor ("rapor hazırla" isteğinin çıktısı)

Rapor isteği Format 2'deki `intent: rapor` ile gelir. Model 2 birkaç analizi çalıştırır,
Model 3 metinleri yazar, doğrulama katmanı sayıları kontrol eder. Her bölüm Format 3'teki
`sonuc` (tablo) ve `grafik` yapısını kullanır.

```json
{
  "durum": "tamam",
  "baslik": "Satış Verisi Analiz Raporu",
  "kaynak": "satislar.csv",
  "ozet": "15.230 siparişin toplam tutarı 1.284.500 TL. En çok sipariş Kayseri'den geliyor.",
  "bolumler": [
    {
      "baslik": "Genel bakış",
      "metin": "Veri 5 kolon ve 15.230 satırdan oluşuyor. Kolonların %3,6'sı boş.",
      "tablo": {"kolonlar": ["kolon", "eksik_oran"], "satirlar": [["sat_tut", 0.12], ["sehir", 0.05]]},
      "grafik": null
    },
    {
      "baslik": "Aylık satış trendi",
      "metin": "En çok satış Mart ayında, 1.410 TL ile yapıldı.",
      "tablo": {"kolonlar": ["ay", "toplam"], "satirlar": [["Ocak", 1250], ["Şubat", 980], ["Mart", 1410]]},
      "grafik": {"tip": "cizgi", "x": "ay", "y": "toplam", "baslik": "Aylık satış"}
    }
  ],
  "uyarilar": ["sat_tut kolonunun %12'si boş, sonuçlar kalan veriyle hesaplandı"]
}
```

| Alan | Açıklama |
|---|---|
| `durum` | `tamam` veya `hata` |
| `baslik`, `kaynak` | Rapor başlığı ve veri kaynağı |
| `ozet` | Raporun 1-2 cümlelik özeti (Model 3) |
| `bolumler[].baslik` | Bölüm başlığı |
| `bolumler[].metin` | Bölümün doğal dil açıklaması (Model 3) |
| `bolumler[].tablo` | Format 3'teki `sonuc` ile aynı yapı, yoksa `null` |
| `bolumler[].grafik` | Format 3'teki `grafik` ile aynı yapı, yoksa `null` |
| `uyarilar` | Eksik/anormal veri uyarıları |