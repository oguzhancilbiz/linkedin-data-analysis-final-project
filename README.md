# 🔍 LinkedIn İş İlanları Analizi & Maaş Tahmin Pipeline'ı (2023–2024)

> **Uçtan Uca Veri Analizi, İstatistiksel Hipotez Testleri ve Makine Öğrenmesi Tabanlı Canlı Maaş Tahmin Sistemi**

---

## 📌 Proje Özeti
Bu proje, LinkedIn üzerindeki **123.849 iş ilanını** kapsayan ilişkisel bir veri setini inceleyerek iş piyasasının maaş yapısını, sektörel ve coğrafi farklılıkları, uzaktan çalışma primlerini ve beceri (skill) bazlı maaş etkilerini ortaya koymayı amaçlamaktadır.

Veri setindeki ilanların **%71,6'sında (88.728 ilan) maaş bilgisi bulunmamaktadır**. Bu çalışmada, eksik maaş verilerini silmek veya görmezden gelmek yerine, **Random Forest Regressor** modeli ile tahmin edilerek analizin kapsayıcılığı **%100'e** çıkarılmıştır.

---

## 🛠️ Kullanılan Teknolojiler & Kütüphaneler
- **Veri İşleme & ETL:** Python, Pandas, NumPy
- **NLP / Metin Madenciliği:** Regular Expressions (`re`) ile yetenek bayraklandırma (SQL, Python, Cloud, ML, BI, Excel)
- **Görselleştirme:** Matplotlib, Seaborn, Plotly (Etkileşimli trendler)
- **İstatistiksel Testler:** SciPy (`Mann-Whitney U`, `Kruskal-Wallis`, `Chi-Square`, `Spearman Correlation`, `Levene Test`, `Q-Q Normality Plot`)
- **Makine Öğrenmesi & Pipeline:** Scikit-Learn (`Random Forest`, `Linear Regression`, `Isolation Forest`, `Cross-Validation`), `Joblib`

---

## 🗂️ Veri Seti Mimarisi & ETL Süreci
Proje, `postings.csv` merkezli olmak üzere 11 ilişkisel CSV dosyasının `LEFT JOIN` stratejisiyle birleştirilmesiyle oluşturulmuştur.

```text
[postings.csv (123.849 İlan)]
   ├── salaries.csv ──────────> (Saatlik/Aylık -> Yıllık USD Normalleştirme)
   ├── companies.csv ─────────> (Şirket Büyüklüğü, Konum, İsim)
   ├── employee_counts.csv ───> (En güncel anlık kaydın seçilmesi - Tekilleştirme)
   ├── benefits.csv ──────────> (İlan başına toplam yan hak adedi)
   ├── job_skills.csv ────────> (İlan başına aranan beceri sayısı)
   └── job_industries.csv ────> (Sektör kodlarının isimlerle eşlenmesi)
```

### 🧹 Kritik Veri Temizliği Adımları:
1. **Dtype Dönüşümleri:** Epoch milisaniye zaman sütunları `datetime` tipine çevrildi.
2. **Maaş Normalleştirmesi:** Saatlik ($2080 saat/yıl), aylık (12 ay), haftalık (52 hafta) maaşlar tek bir yıllık USD ($) tabanına getirildi.
3. **IQR Aykırı Değer Temizliği:** Sağa çarpık (right-skewed) maaş verisindeki 535 Milyon $'lık veri girişi hatası gibi uç değerler **IQR (Interquartile Range)** yöntemiyle tespit edildi (Üst sınır: $234.500). Aykırı değerlerin satırları silinmeyip `salary` hücresi maskelenmiştir.

---

## 🔬 Aykırı Değer (Outlier) Yöntemleri ve Senaryo Analizleri

Notebook bünyesinde **6 farklı outlier yöntemi** (IQR, Z-Score, Modified Z-Score, Winsorize, Percentile Trimming, Isolation Forest) ve **3 farklı mimari senaryo** gerçek verilerle kıyaslanmıştır:

| Yöntem / Senaryo | Kullanılan İlan | Kapsam | Model MAE | Sonuç & Değerlendirme |
|---|---|---|---|---|
| **Senaryo 1 (ML Yok)** | 35.121 | %28.4 | N/A | Yanlış örneklem sapması (selection bias) riski taşır. |
| **Senaryo 2 (Dropna)** | 35.121 | %28.4 | $27.967 | 88.728 ilan çöpe gider, tahmin üretilmez. |
| **Senaryo 3 (ML Pipeline - Seçilen)** | **123.849** | **%100** | **$27.967** | **%100 kapsam sağlanır, gerçek veriler `salary` sütununda korunur.** |

> **Outlier Kararı:** Sağa çarpık veride Z-Score'un $6.9M'lık hataları kaçırdığı kanıtlanmış; sıralama tabanlı **IQR x 1.5** yöntemi en doğru yaklaşım olarak seçilmiştir.

---

## 📈 Öne Çıkan Bulgular (Key Insights)

1. **Maaşı Belirleyen Ana Faktörler (Random Forest Feature Importance):**
   - **1. Şirket Prestiji & Bilinirliği** (`follower_count / employee_count`) — Maaşı belirleyen 1 numaralı faktör.
   - **2. Sektör** — Donanım ve Ağ Güvenliği sektörleri geleneksel yazılım şirketlerini geride bırakmaktadır.
   - **3. Lokasyon** — Washington (WA) eyaleti medyan maaşta California (CA)'yı geride bırakmıştır.

2. **Beceri Primleri (Skill Premiums):**
   - **Machine Learning (ML):** Piyasa medyanının **+$75.000** üzerinde prim taşır.
   - **Cloud / Python:** +$50.000 ile +$58.000 arası ek prim sağlar.
   - **Excel:** Piyasa medyanının altında (-$1.000) kalmaktadır; artık prim taşıyan bir beceri değil, asgari zorunluluktur.

3. **Uzaktan Çalışma (Remote) Etkisi:**
   - Uzaktan çalışma oranı post-pandemi döneminde **%12,3** seviyesine gerilemiştir.
   - Buna rağmen uzaktan çalışan pozisyonların medyan maaşı ofis pozisyonlarına kıyasla **+$34.108 daha yüksektir** (Mann-Whitney U testi, $p < 0.000001$ ile kanıtlanmıştır).

---

## 🚀 Canlı Tahmin Pipeline'ı (Production-Ready Inference)

Eğitilen **Random Forest** modeli ve kategorik dönüştürücüler `joblib` vasıtasıyla `.pkl` olarak kaydedilmiştir. Sisteme veri setinde hiç bulunmayan yeni bir ilan geldiğinde anında maaş ve band tahmini üretilmektedir:

```python
from joblib import load

# Modeli ve kategorik encoder'ları yükle
rf = load('linkedin_rf_model.pkl')
encoders = load('linkedin_encoders.pkl')

# Örnek Kullanım (Canlı Tahmin)
tahmin, band = tahmin_yap(
    sirket_buyuklugu=7, # Dev Şirket
    eyalet='WA', 
    sektor='Software Development', 
    uzaktan=1, 
    yanhak_sayisi=8, 
    beceri_sayisi=12, 
    python=1, ml=1, cloud=1, sql=1
)
# Çıktı: $168.450 -> Çok Yüksek (>$150K)
```

---

## 📂 Proje Klasör Yapısı

```text
├── linkedin_analiz.ipynb           # Ana Proje & Analiz Notebook'u
├── linkedin_rf_model.pkl           # Eğitilmiş Random Forest Modeli
├── linkedin_encoders.pkl           # Kategorik Encoder Sözlükleri
├── .gitignore                      # Git geçici dosya engelleme
└── README.md                       # Proje Dokümantasyonu
```

---

## 🛠️ Kurulum & Çalıştırma

```bash
# 1. Repoyu klonlayın
git clone https://github.com/KULLANICI_ADI/linkedin-veri-analizi.git
cd linkedin-veri-analizi

# 2. Gerekli kütüphaneleri yükleyin
pip install pandas numpy matplotlib seaborn plotly scipy scikit-learn joblib

# 3. Jupyter Notebook üzerinden akışı çalıştırın
jupyter notebook linkedin_analiz.ipynb
```