#  Missense Genetik Varyantların Patojenite Sınıflandırması: Çok Girdili 1D-CNN ve Dinamik Eşik Optimizasyonu

Bu depo, insan genomundaki *missense* (yanlış anlamlı) varyantların hastalık yapıcı (Patojenik) veya iyi huylu (Benign) olma durumlarını tahmin etmek amacıyla geliştirilen **Çok Girdili Hibrit Derin Öğrenme** ve **Klasik Makine Öğrenmesi** modellerini içermektedir. Proje, klinik genetik tanıda hayati önem taşıyan "Yanlış Negatif" (FN) hatalarını en aza indirmek için tasarlanmıştır.

##  Projenin Özgün Değerleri ve Yenilikleri

*  **Çok Girdili (Multi-Input) 1D-CNN Mimarisi:** Sadece sayısal *in silico* skorlara (REVEL, CADD vb.) dayanmaz. Mutasyon bölgesindeki ±5 komşuluktaki **DNA nükleotidlerini (11x4)** ve **Amino Asit dizilimlerini (11x20)** Bire-Bir Kodlama (One-Hot) ile matrislere çevirerek biyolojik motifleri doğrudan okur.
*  **Dinamik Karar Eşiği (Threshold) Optimizasyonu:** Klasik "tek beden herkese uyar" mantığındaki 0.50 karar eşiği terk edilmiştir. Her algoritma için F1-Skorunu zirveye çıkaran ve klinik riskleri (FN) minimize eden modele özgü "Altın Eşikler" (Örn: CNN için 0.25, Rastgele Orman için 0.30) matematiksel olarak hesaplanmıştır.
*  **Sızıntı Kontrolü (Data Leakage Prevention):** Modelin genomik adresleri ezberlemesini önlemek için veri seti rastgele değil, `GroupShuffleSplit` kullanılarak **Gen Bazlı** ayrıştırılmıştır.

##  Teknik Yaklaşım ve Veri Hattı (Pipeline)

### 1. Veri Ön İşleme ve Zenginleştirme
* **Klinik Veriler:** ClinVar veritabanından yalnızca "uzman paneli tarafından onaylanmış" yüksek güvenilirlikli varyantlar kullanılmıştır.
* **Veri Zenginleştirme:** CFTR ve PAH gibi gen panellerindeki sınıf dengesizliği, gnomAD veritabanından çekilen sağlıklı popülasyon örneklemleriyle giderilmiştir.
* **Özellik Çıkarımı:** MyVariant.info API, NCBI API ve `pyfaidx` kütüphaneleri kullanılarak evrimsel korunmuşluk, biyokimyasal değişimler (Delta MW, Charge vs.) ve referans dizilimler dinamik olarak çekilmiştir.

### 2. Değerlendirilen Modeller
* **Klasik ML:** Rastgele Orman, Lojistik Regresyon, K-En Yakın Komşu, Basit Bayes, Karar Ağacı, YSA (MLP).
* **Derin Öğrenme:** Tabular, DNA ve Protein girdilerini aynı anda işleyen `Concatenate` tabanlı TensorFlow/Keras 1D-CNN modeli.

##  Öne Çıkan Sonuçlar
* **Rastgele Orman:** 0.30 dinamik eşiğinde **0.9646 F1-Skoru** ve **0.9885 ROC-AUC** ile genel en yüksek performansı göstermiştir.
* **Basit 1D-CNN:** 0.25 dinamik eşiğinde **0.9464 F1-Skoru** elde etmiş ve klinik güvenlik açısından en kritik hata olan *Yanlış Negatif (FN)* sayısını test setinde sadece **9 vakaya** kadar indirerek Duyarlılığı maksimize etmiştir.

##  Dosya Yapısı
```text
├── veri_ve_model_kodlari.ipynb   # Veri çekimi, ön işleme, model eğitimi ve eşik analizi notebook'u
├── clinvar.vcf.gz                # Ham ClinVar veri seti (Sıkıştırılmış)
├── hg38.fa                       # Referans insan genomu (DNA dizilimleri için)
├── PAH.csv / CTFR.csv            # gnomAD veri zenginleştirme dosyaları
└── README.md                     # Proje dokümantasyonu
## ⚙️ Kurulum ve Çalıştırma
```
### Gereksinimler
Projenin sorunsuz çalışması için **Python 3.8+** sürümü önerilir. Gerekli kütüphaneleri aşağıdaki komutla yükleyebilirsiniz:

```bash
pip install pandas numpy scikit-learn tensorflow matplotlib seaborn requests pyfaidx
```
## Çalıştırma Adımları
Depoyu bilgisayarınıza klonlayın.
hg38.fa (Referans Genom) ve clinvar.vcf.gz dosyalarının ana dizinde olduğundan emin olun.
Jupyter Notebook dosyasını (veri_ve_model_kodlari.ipynb) açın.
Hücreleri sırasıyla çalıştırarak API veri çekimi, model eğitimi ve görselleştirme adımlarını (Karmaşıklık matrisleri, eşik grafikleri) baştan sona simüle edebilirsiniz.

Not: NCBI ve MyVariant API'lerine yapılan toplu istekler (request) zaman alabilir. Notebook içerisinde sunucuyu yormamak adına .sleep() fonksiyonları eklenmiştir.

Geliştirenler: Melda Kahraman & Ege Ertekin
Bu proje Gazi Üniversitesi Bilgisayar Mühendisliği Veri Bilimi dersi kapsamında  geliştirilmiştir.
