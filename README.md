# 🎬 Letterboxd Film Data Analysis & Rating Prediction

Letterboxd film veri seti üzerine yapılmış uçtan uca bir veri analizi ve makine öğrenmesi projesi. Proje; veri temizleme, kapsamlı özellik mühendisliği, tahmin modeli ve interaktif bir Power BI dashboard'u içermektedir.

## 📌 Proje Özeti

~941.000 kayıt içeren 9 farklı CSV dosyası birleştirilerek film derecelendirmelerini etkileyen faktörler analiz edilmiş ve bir derecelendirme tahmin modeli geliştirilmiştir.

## 🔍 Yöntem

### 1. Veri Temizleme & EDA
- 9 CSV dosyasının birleştirilmesi ve tutarlılık kontrolü
- Eksik veri, aykırı değer ve veri tipi düzeltmeleri
- Keşifsel veri analizi ile film türü, tema, ülke ve stüdyo dağılımlarının incelenmesi

### 2. Feature Engineering
- Tür (genre), tema (theme), ülke (country) ve stüdyo (studio) alanları için **multi-hot encoding** (pandas ile)
- Modelin çoklu kategorik ilişkileri yakalayabilmesi için özel dönüşümler

### 3. Modelleme
- **RandomForestRegressor** ile derecelendirme tahmini
  - MAE: **~0.21**
  - R²: **~0.56**
- Karşılaştırma: Linear Regression baseline'ına göre daha iyi performans

### 4. Power BI Dashboard
- Çok sayfalı, interaktif dashboard
- DAX ile hesaplanmış tablolar (`UNION`, `SELECTCOLUMNS` kullanılarak one-hot encoded kolonların unpivot edilmesi)
- Görselleştirmeler: [tür/tema dağılımları, derecelendirme trendleri vb. — kendine göre doldurabilirsin]

### 5. Sunum
- Bulguların PowerPoint sunumuna dönüştürülmesi (pptxgenjs ile otomatik oluşturuldu)

## 🛠️ Kullanılan Teknolojiler

| Alan | Araçlar |
|---|---|
| Veri İşleme | Python, Pandas |
| Modelleme | Scikit-learn (RandomForestRegressor, Linear Regression) |
| Görselleştirme | Power BI, DAX |
| Sunum | PowerPoint (pptxgenjs) |

## 📊 Sonuçlar

- RandomForestRegressor modeli, ~0.56 R² skoru ile film derecelendirmelerindeki varyansın önemli bir kısmını açıklayabilmiştir
- Linear Regression baseline'ına kıyasla belirgin performans artışı sağlanmıştır

## 📁 Repo İçeriği

```
├── data/                  # Ham/işlenmiş veri (varsa)
├── notebooks/             # Jupyter/Colab notebook(lar)
├── presentation.pptx      # Proje sunumu
└── README.md
```

## 👥 Katkıda Bulunanlar

Bu proje bir grup çalışması olarak gerçekleştirilmiştir.

---

*Bu proje bir bitirme (graduation) projesi kapsamında hazırlanmıştır.*
