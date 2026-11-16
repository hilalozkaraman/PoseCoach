# PoseCoach — Yoga Hareketi Analizi ve Türkçe Geri Bildirim

Bilgisayarla Görü dersi ara sınav projesi, **Problem 1**: doğru yoga hareketini gösteren referans video ile kullanıcının aynı hareketi yaptığı videoyu karşılaştırır, farkı derece olarak ölçer ve nasıl düzeltileceğini Türkçe söyler.

Örnek çıktı (Tadasana / Açı 1):

> 30.–33. saniyeler arasında kollarını referanstaki gibi yukarıda tutamıyorsun: sağ kol yüksekliği ≈98° (referans ≈171°) … Kollarını yaklaşık 73° (sağ) ve 71° (sol) daha yukarı kaldır ve dirseklerini uzat.

En az 8 ölçüm güvenilir biçimde karşılaştırılabilmiş ve raporlanabilir sapma ya da tekrar sayısı farkı bulunmamışsa sistem olumlu geri bildirim verir:

> Hareketi doğru yaptın. Dirseklerin, kol kaldırma açın, kalça açıların ve dizlerin referans poza oldukça yakın, böyle devam et!

## Nasıl çalışır

1. **Pose çıkarımı:** MediaPipe Pose Landmarker (`pose_landmarker_full`, VIDEO modu) ile saniyede 2 karede 33 landmark.
2. **Ölçüm:** 3B world landmarklardan sağ/sol omuz, dirsek, kalça, diz ve ayak bileği açıları; 2B görüntü düzleminden gövde eğimi, omuz hizası, bilek yüksekliği ve ayak açıklığı.
3. **Zamansal hizalama:** Toleransa göre normalize edilmiş ölçüm serileri, hazır DTW kütüphanesi kullanılmadan, dinamik programlamayla kodlanmış Dynamic Time Warping (DTW) ile eşlenir.
4. **Sapma tespiti:** Tolerans art arda en az 3 karede aşılırsa sapma sayılır; ortalaması toleransın 1,5 katını geçen sapmalar raporlanır, aynı anda sapan eklemler tek olayda birleştirilir.
5. **Geri bildirim:** En önemli 3 olay için kural tabanlı Türkçe cümle: zaman aralığı, eklem, referans ve kullanıcı açısı, düzeltme yönü ve derecesi.

Toleranslar: dirsek/diz 15°, omuz/kalça/ayak bileği 20°, gövde eğimi 12°, omuz hizası 10°.

## Kurulum ve çalıştırma

Defter **Google Colab** için yazılmıştır (Drive bağlantısı kullanır).

1. Veri setini indirin (aşağıya bakın) ve video klasörünü ya da ZIP dosyasını Google Drive'a koyun.
2. `PoseCoach_Project.ipynb` dosyasını Colab'de açın.
3. 2. bölümdeki `VERI_YOLU` değerini Drive'daki klasörünüze göre değiştirin.
4. Hücreleri sırayla çalıştırın. 6. bölüm tek bir çifti, 8. bölüm bütün çiftleri analiz eder.

İlk çalıştırmada pose modeli otomatik indirilir. Her videonun ölçümleri Drive'da önbelleğe alınır; sonraki çalıştırmalar hızlıdır.

### Arayüz

Defterin sonundaki iki hücre, analiz fonksiyonlarının üzerine arayüz ekler. İkisi de önceki hücreler çalıştıktan sonra çalıştırılmalıdır.

- **Colab içi panel (ipywidgets):** Hareket ve kamera açısı seçilir; geri bildirim, sapma anının kareleri ve açı grafiği hücrenin altında görünür.
- **Web uygulaması (Gradio):** Hücre çalışınca `gradio.live` uzantılı geçici bir bağlantı verir; uygulama ayrı bir sekmede açılır. "Veri setinden seç" sekmesi panelle aynı işi yapar. "Kendi videonu yükle" sekmesi bir referans ve bir kullanıcı videosu alır (deneysel; ilk analiz video başına birkaç dakika sürebilir). Colab oturumu kapanınca bağlantı çalışmaz.

Test edilen ortam: Python 3.13, MediaPipe 0.10.35, OpenCV 4.14 (Google Colab, CPU). Paket listesi `requirements.txt` dosyasındadır.

## Veri seti

B. Kanorewala, Y. Suryawanshi, K. Patil, N. Gunjal, "Yoga for all: A Comprehensive Collection of Yoga Images and Videos dataset", Mendeley Data, V1, 2023, doi: [10.17632/jc4mmnvcdk.1](https://doi.org/10.17632/jc4mmnvcdk.1) (CC BY 4.0).

- 10 asana × 4 kamera açısı × Right/Wrong = 80 video, 40 çift.
- Yalnızca videolar kullanıldı; `Right` videolar referans, `Wrong` videolar kullanıcı videosu olarak alındı.
- Bazı kamera açısı / etiket adları elle düzeltildi.
- `Wrong` videolar gösterim niteliğindedir: kişi önce hareketin yanlış yapılışını gösterir, sonra doğrusunu yapar. Videonun tamamı kullanıcı videosu olarak alınır; sistem içindeki sapan bölümleri bulur.
- Videolar bu depoda **yer almaz**; yukarıdaki bağlantıdan indirilmelidir.

Beklenen adlandırma: dosya ya da klasör adında hareket adı, `Right` / `Wrong` ve `Angle 1`–`Angle 4` geçmelidir. Örnek:

```
Tadasana/Tadasana Right Steps/Tadasana Right Steps Angle 1.mp4
Tadasana/Tadasana Wrong Steps/Tadasana Wrong Steps Angle 1.mp4
```

## Çıktılar

Sonuçlar Drive'da `PoseCoach_CokluHareket_Sonuclar/<RUN_ADI>/` klasörüne yazılır:

| Dosya | İçerik |
|---|---|
| `video_katalogu.csv`, `video_ciftleri.csv` | Bulunan videolar ve Right/Wrong eşleşmeleri |
| `ornek_olcumler.csv`, `ornek_geri_bildirim.txt` | Seçilen tek çiftin ölçümleri ve geri bildirimi |
| `ornek_hareket_karsilastirmasi.png` | DTW ile hizalanmış açı eğrileri |
| `sapma_*.png` | Sapma anında referans ve kullanıcı kareleri |
| `tum_cift_ozetleri.csv`, `tum_eklem_sapmalari.csv` | Bütün çiftlerin özetleri ve eklem bazlı sapmaları |

## Sınırlılıklar

- Tek kameradan kestirilen 3B açılar, yatarak ve ters yapılan hareketlerde ve önden çekimde gürültülüdür.
- Toleranslar deneysel başlangıç değerleridir; klinik norm değildir. Bir sapmanın raporlanması için tolerans art arda en az 3 geçerli karede aşılmalı ve ortalama sapma toleransın en az 1,5 katı olmalıdır (omuz açısı için 20° × 1,5 = 30°).
- Saniyede 2 kare örneklendiği için yaklaşık 1,5 saniyeden kısa sapmalar yakalanmaz.
- Omurga şekli ölçülmez: 40 çiftin 2'sinde (Marjariasana, yandan çekim: Açı 2 ve 4) hiç sapma bulunamadı.
- El ve parmak biçimi ölçülmez (Tadasana'daki el biçimi farkı yakalanmadı).
- `Wrong` videolarda referansta olmayan fazladan bölümler bulunduğu için zamansal hizalama yaklaşık kalır; sistem bunu tekrar sayısı farkı uyarısıyla bildirir.
- Arayüzdeki video yükleme sekmesi sınırlı biçimde denenmiştir.
- Referans ve kullanıcı hareketi farklı tarafa (sağ/sol) yaparsa sistem bunu ayna olarak tanımaz.
- Tek bir `Right` video evrensel doğru poz anlamına gelmez.

## Kaynaklar

- Google MediaPipe Pose Landmarker (Python): https://developers.google.com/edge/mediapipe/solutions/vision/pose_landmarker/python
- Yoga for All veri seti: https://doi.org/10.17632/jc4mmnvcdk.1
- SciPy `cdist`: https://docs.scipy.org/doc/scipy/reference/generated/scipy.spatial.distance.cdist.html

Kod geliştirme, hata ayıklama ve rapor taslağında üretken yapay zekâ (ChatGPT, Claude) desteği alınmıştır.

## Yazar

Rümeysa Hilal Özkaraman — 503020260009
