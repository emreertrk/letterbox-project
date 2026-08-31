# 🎬 Letterboxd Film Veri Seti — Temizleme, Feature Engineering & Power BI Dashboard

Letterboxd'un halka açık film veri setini (9 ayrı ham CSV dosyası) tek bir temiz, zengin özellik setine dönüştürüp; bu veriyi hem bir makine öğrenmesi modelinde hem de interaktif bir Power BI dashboard'unda kullanılabilir hale getiren uçtan uca bir veri mühendisliği projesi.

---

## 🎯 Projenin Amacı

Letterboxd verisi, film bilgilerinin (isim, tür, tema, dil, ülke, ekip, oyuncu, gösterim tarihleri...) birbirinden bağımsız 9 farklı CSV dosyasına dağılmış halde gelmesiyle bilinir. Bu proje:
1. Bu dağınık veriyi tek bir satır-başına-film tablosuna indirger,
2. Kategorik ve çok değerli (bir filmin birden fazla türü/teması/ülkesi olabilmesi gibi) alanları modele girecek sayısal özelliklere dönüştürür,
3. Sonucu hem analiz (Power BI) hem de tahminleme (regresyon modeli) için kullanılabilir hale getirir.

---

## 📦 Kullanılan Ham Veri Dosyaları

Projede aşağıdaki 9 CSV dosyası kullanıldı (her biri `id` üzerinden birbirine bağlanıyor):

| Dosya | İçerik | Notebook'ta Kullanımı |
|---|---|---|
| `movies.csv` | Film adı, çıkış yılı, süre, ortalama puan | Temel tablo — `tagline`/`description` çıkarıldı, kritik alanlarda (`name`, `date`, `minute`, `rating`) null olan satırlar silindi |
| `languages.csv` | Filmde konuşulan/birincil diller | En çok geçen 20 dil + "Other" kategorisine indirgendi, ağırlıklı (primary=2, spoken=1) pivot encoding |
| `genres.csv` | Film türleri | 19 türün tamamı one-hot (crosstab) encoding ile eklendi |
| `themes.csv` | Serbest metin tarzı ince temalar (109 benzersiz değer) | Elle hazırlanmış bir sözlükle ~28 üst kategoriye (Action_Theme, Horror_Theme, Crime_Theme vb.) gruplanıp one-hot encoding yapıldı |
| `studios.csv` | Yapımcı stüdyolar | Stüdyo sayısı, büyük stüdyo (Warner Bros, Universal, Disney vb. 8 majör stüdyo) flag'i, en yüksek stüdyo frekansı hesaplandı |
| `releases.csv` | Ülke bazlı gösterim tarihleri ve tipleri | Ülke/tip sayıları, ilk-son gösterim tarihi, gösterim tipi flag'leri (digital/physical/premiere/tv/theatrical), en çok geçen 25 ülke için one-hot, yıl/ay/çeyrek, sinema→dijital geçiş süresi |
| `crew.csv` | Ekip bilgisi (yönetmen, yazar, yapımcı vb.) | Rol bazlı kişi sayıları + elle seçilmiş ~40 usta yönetmenden (Kurosawa, Bergman, Nolan, Miyazaki vb.) biri var mı flag'i |
| `countries.csv` | Yapım ülkesi | En çok geçen 20 ülke için one-hot encoding |
| `actors.csv` | Oyuncu kadrosu | Oyuncu sayısı, benzersiz rol sayısı + elle seçilmiş ~180 tanınmış oyuncudan kaçının kadroda olduğu |

---

## 🔍 Metodoloji (Notebook Akışı)

### 1️⃣ Temel Tablo — `movies.csv`
- `tagline` ve `description` sütunları çıkarıldı (metinsel, modele doğrudan girmiyor)
- `name`, `date`, `minute`, `rating` alanlarından herhangi biri boşsa o film tamamen çıkarıldı
- Sonuç: **90.475 film**

### 2️⃣ Kademeli Left Join + Temizlik Döngüsü
Her bir ek dosya için aynı desen izlendi:
1. Ham veriden özellik tablosu türetildi (pivot/crosstab/groupby ile)
2. `df_final` ile `id` üzerinden **left join** yapıldı
3. Join sonrası oluşan NaN'lar kontrol edildi
4. Duruma göre: **anlamlı eksiklik** olan satırlar silindi (örn. dil bilgisi hiç olmayan film, stüdyo bilgisi tamamen eksik olan film) *veya* **"özelliği yok" anlamına gelen** NaN'lar 0 ile dolduruldu (örn. bir filmin belirli bir temaya sahip olmaması)

Bu döngü boyunca veri seti şu şekilde daraldı:
| Adım | Silinen Kayıt | Kalan Kayıt |
|---|---|---|
| movies.csv temizliği | — | 90.475 |
| Dil bilgisi tamamen eksik | 3.196 | 87.279 |
| Hiç tür (genre) bilgisi yok | 1.131 | ~86.148 |
| Stüdyo bilgisi tamamen eksik | 9.902 | ~76.246 |
| Ekip (crew) bilgisi eksik | 244 | 76.002 |
| Oyuncu bilgisi eksik | 1.623 | **74.623** |

> Not: Tema (`themes.csv`) ve ülke (`countries.csv`/`releases.csv`) join'lerinde oluşan NaN'lar silme değil, **0 ile doldurma** ile ele alındı — çünkü bu alanlarda NaN "o kategoriye ait değil" anlamına geliyor, veri eksikliği değil.

### 3️⃣ Final Veri Seti
- **74.623 film × 138 sütun**
- Sıfır NaN, sıfır tekrarlanan `id`, sıfır `_x`/`_y` merge kalıntısı sütunu (kontrol edildi)
- `powerbi_data.csv` olarak dışa aktarıldı — hem Power BI dashboard'unun hem de model eğitiminin veri kaynağı bu dosya

### 4️⃣ Modelleme (ayrı adımda)
`powerbi_data.csv` üzerinde:
- **RandomForestRegressor** ile puan (rating) tahmini — MAE ≈ 0.21, R² ≈ 0.56
- Karşılaştırma: Linear Regression baseline'ına göre daha iyi sonuç

### 5️⃣ Power BI Dashboard
- Çok sayfalı, `powerbi_data.csv` kaynaklı dashboard
- DAX ile hesaplanmış tablolar (`UNION`, `SELECTCOLUMNS`) kullanılarak one-hot encoded (tür/tema) sütunlar unpivot edilip normalize bir yapıya dönüştürüldü — filtreleme/segmentasyon bunun üzerinden çalışıyor

> 💡 **Öğrenilen ders:** `.pbix` dosyasından `pbixray` gibi araçlarla ham tablo agregasyonu almak, dashboard'daki görsellerle birebir örtüşmüyor — çünkü DAX ölçüleri (measures) ve örtük filtreler bu şekilde yansımıyor. Doğrulanmış sayılar için dashboard ekran görüntülerine güvenmek gerekiyor.

### 6️⃣ Sunum
Bulgular `pptxgenjs` ile otomatik oluşturulan bir PowerPoint sunumunda özetlendi (grafik XML uyumluluğu için `multiLvlStrRef → strRef` düzeltmesi gerekti).

---

## 🛠️ Kullanılan Teknolojiler

| Kategori | Araç / Kütüphane |
|---|---|
| Veri İşleme | Python, Pandas (merge, pivot_table, crosstab, groupby) |
| Modelleme | Scikit-learn (RandomForestRegressor, LinearRegression) |
| Görselleştirme (BI) | Power BI, DAX |
| Sunum | PowerPoint (pptxgenjs) |
| Ortam | Google Colab |

---

## 📁 Repo Yapısı

```
├── notebooks/
│   └── letterbox_project.ipynb   # Veri temizleme + feature engineering pipeline (movies → powerbi_data.csv)
├── presentation/
│   └── presentation.pptx         # Proje sunumu
└── README.md
```

> Not: `.pbix` dosyası boyutu (~300MB) GitHub'ın tek dosya sınırını aştığı için repoya eklenmedi. Dashboard'un ekran görüntüleri eklenebilir.

---

## 📊 Öne Çıkan Teknik Detaylar

- **Tema indirgeme:** Ham veride 109 benzersiz, serbest-metin tarzı tema etiketi vardı (örn. "Bloody vampire horror", "Gritty crime and ruthless gangsters"). Bunlar elle hazırlanmış bir sözlükle ~28 anlamlı üst kategoriye gruplanarak encoding'e uygun hale getirildi.
- **Ağırlıklı dil encoding:** Diller basit 0/1 değil, `type` alanına göre ağırlıklandırıldı (Primary/Language = 2, Spoken = 1) — bir filmin birincil dili ile yan dilleri arasındaki fark korunmuş oldu.
- **Kürasyonlu "prestij" flag'leri:** Yönetmen ve oyuncu listelerinde, sinema tarihinde öne çıkan isimlerden oluşan elle seçilmiş listeler kullanılarak `top50_director` ve `top_actor_count` gibi özellikler türetildi.
- **Aşamalı temizlik stratejisi:** Her join sonrası oluşan eksik veri, "gerçekten eksik" ile "0 anlamına gelen" ayrımı yapılarak farklı şekillerde ele alındı.

---

## 👥 Katkıda Bulunanlar

Bu proje bir bitirme (graduation) projesi kapsamında grup çalışması olarak gerçekleştirilmiştir.
