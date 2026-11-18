# Librix — Proje Özellikleri

Librix, kitap koleksiyonunu, üyeleri, ödünç işlemlerini, stokları ve
rezervasyon sırasını yönetmek için geliştirilmiş masaüstü kütüphane
uygulamasıdır. Masaüstü istemci, Django REST API ile haberleşir.

## Teknoloji ve bileşenler

- **Masaüstü arayüzü:** Python ve PyQt5
- **API:** Django ve Django REST Framework
- **Oturum açma:** JSON Web Token (JWT); istemcide erişim ve yenileme token'ı
- **Veritabanı:** SQLite (`API/db.sqlite3`)
- **Medya:** Kitap görselleri `API/media/` altında saklanır
- **Ana uygulama alanları:** `User`, `Book`, `OduncAlmaSistemi`,
  `Reservation` ve `Admin_Panel`

## Kullanıcı rolleri

### Yönetici

Masaüstü menüsünde ana sayfa, kitaplar, üyeler, ödünç/iade, raporlar, ayarlar,
rezervasyonlar ve stok ekranları bulunur. Yönetici kitap kataloğunu ve
üyelerin ödünç süreçlerini yönetebilir.

### Normal üye

Üye menüsünde kitap kataloğu, **Kitaplarım** ve profil/ayarlar ekranları
bulunur. Üye kendi ödünç geçmişini ve rezervasyon durumlarını görüntüleyebilir.

Üye arayüzünde kitap seçip ödünç talebi veya rezervasyon oluşturmak için bir
işlem düğmesi bulunmuyor. API'de üye oturumuyla kullanılabilen
`POST /take-book/<kitap_id>/` uç noktası mevcut; bu işlem masaüstü arayüzüne
bağlı değildir.

## Uygulamadaki özellikler

### Kitap kataloğu

- Kitapları listeleme ve ad/barkod üzerinden arama
- Kitap ayrıntılarını görüntüleme
- Kitap ekleme, düzenleme, yayına alma/yayından kaldırma ve silme
- Barkod, başlık, yazar, kategori, dil, fiyat, açıklama ve stok bilgileri
- Kitap görselleri için veri modeli ve medya depolama desteği
- Kategori ve yazar kayıtlarının listelenmesi; aktif/pasif durumlarının
  yönetilmesi
- Kategori/yazar adına ve durumuna göre filtreleme; ilişkili kitap sayılarının
  gösterilmesi
- Kitapla ilişkili yazar veya kategorinin silinmesi, kitap bağlantıları
  varken engellenir
- Kitap dili kayıtlarının listelenmesi

### Üye yönetimi

- API üzerinden üye kaydı oluşturma; kayıt sırasında kullanıcı adı, soyadı,
  telefon, kimlik numarası ve eşleşen parola alanları kullanılır
- Kayıt API'si yeni hesabı normal üye rolünde oluşturur
- Yönetici arayüzünde üyeleri listeleme, arama ve bir üyenin kitap geçmişini
  görüntüleme
- Üyeyi pasif hâle getirme
- Üye profilinde kullanıcı adı, soyadı, telefon, e-posta ve rol bilgilerini
  görüntüleme

Masaüstü istemcisinde kayıt olma, parola sıfırlama, profil bilgilerini
düzenleme veya pasif üyeyi yeniden etkinleştirme ekranı bulunmuyor.

### Ödünç ve iade

- Yönetici adına doğrudan ödünç kaydı oluşturma
- Bekleyen ödünç kaydını onaylayıp kitabı teslim edilmiş duruma getirme
- Kitabı iade alma ve iade edilen adedi stoğa ekleme
- Kitap/üye arama, kayıtları sayfalama ve durumları görüntüleme
- Son teslim tarihini yönetici tarafından belirleme
- Üye başına **aynı anda bir bekleyen veya iade edilmemiş ödünç kaydı**
  kuralı
- Ödünç verme sırasında aktif kitap ve yeterli stok kontrolü
- Teslim/iade işlemlerinin veritabanı işlemi içinde yürütülmesi

Önemli iş kuralı: Üye mevcut kitabını iade etmeden ikinci bir kitabı teslim
alamaz. Kontrol, bekleyen (`pending`) ve üyede olan (`borrowed`) kayıtları
aktif sayar. İkinci kitap için rezervasyon kuyruğuna girebilir; ancak aktif
ödünç kaydı varken bu rezervasyon teslim edilemez. Kural hem doğrudan ödünç
oluşturma/onayında hem rezervasyon tesliminde uygulanır.

Ödünç durumları: `pending` (bekliyor), `borrowed` (üyede), `returned`
(iade edildi) ve `cancelled` (iptal).

### Rezervasyon ve stok

- Stokta olmayan kitap için bekleme sırasına rezervasyon ekleme
- Yönetici tarafından bir üye adına rezervasyon oluşturma
- Bekleyen rezervasyonu hazır duruma getirme; aynı kitaptaki eski sıranın
  önce işlenmesi
- Hazır kitabı teslim ederek ödünç kaydı oluşturma
- Bekleyen veya hazır rezervasyonu iptal etme
- Hazır rezervasyon iptal edilince ayrılmış adedi stoğa geri ekleme
- Kitap iadesinden sonra bekleme sırasındaki rezervasyonları stok elverdiği
  ölçüde otomatik olarak hazır duruma yükseltme
- Stok listesinde arama; tüm kitaplar, stokta olanlar, az stoklular ve stoksuz
  kitaplar filtreleri
- Toplam, az stoklu (1–2 adet) ve stoksuz kitap özetleri
- Yönetici için stok miktarını güncelleme ve üye adına rezervasyon başlatma

Rezervasyon durumları: `waiting` (sırada), `ready` (teslime hazır),
`completed` (teslim tamamlandı) ve `cancelled` (iptal).

`/take-book/<kitap_id>/` uç noktası, stok varsa ve üyenin aktif ödünç kaydı
yoksa kitabı doğrudan ödünç verir; aksi durumda rezervasyon oluşturur.
Oluşturulan doğrudan ödünç için varsayılan süre 14 gündür. Rezervasyon
tesliminde son tarih yönetici tarafından girilebilir; girilmezse 14 gün
sonrası varsayılan alınır.

### Raporlar ve ana sayfa

- Aktif kitap ve üye sayıları
- Ödünçteki ve bekleyen kitap sayıları
- Son teslim tarihi geçmiş kitapların ve gecikme günlerinin listesi
- En çok ödünç alınan ilk beş kitabın listesi
- Son ödünç işlemlerinin ana sayfada gösterilmesi
- Raporların arka plan iş parçacığında yüklenmesi ve yenileme durumu

Gecikme raporu kayıtları tarihe göre hesaplar. Otomatik e-posta/bildirim,
gecikme cezası veya otomatik ödünç uzatma özelliği görünmüyor.

### Üyenin “Kitaplarım” ekranı

- Üyeye ait ödünç kayıtlarını listeleme
- Bekleyen, ödünçte ve iade edilmiş/okunmuş kitap adetlerini özetleme
- Bekleyen ve hazır rezervasyonları gösterme
- Ödünç ve rezervasyon durumlarını, kayıt tarihini ve son teslim tarihini
  görüntüleme
- API'deki kitap geçmişi durum filtreleri: `all`, `pending`, `borrowed`,
  `returned`

Arayüzde “okunan” sayısı, iade edilmiş kitap kayıtlarından türetilir; ayrı bir
okuma/puanlama takibi değildir.

### Ayarlar ve masaüstü tercihleri

- Profil bilgilerini API'den yenileme
- Açılışta gösterilecek sayfayı seçme
- İade işlemi öncesi masaüstünde onay sorulmasını açıp kapatma
- Tercihleri bilgisayardaki `QSettings` ile saklama
- Giriş/çıkış akışı ve role göre menü gösterimi

## API uç noktaları

Geliştirme ortamındaki varsayılan adres `http://127.0.0.1:8000` şeklindedir.
Korunan uç noktalar JWT erişim token'ını
`Authorization: Bearer <token>` başlığında bekler.

### Kimlik ve üye

| Yöntem | Yol | İşlev |
|---|---|---|
| POST | `/login/` | JWT erişim ve yenileme token'ı alma |
| POST | `/refresh/` | Erişim token'ını yenileme |
| POST | `/register/` | Normal üye hesabı oluşturma |
| GET | `/profile/` | Oturum açan hesabın profil bilgileri |
| GET | `/profile/kitaplarim/?status=<durum>` | Üyenin ödünç geçmişi |
| GET | `/user-list/` | Üye listesi |
| PATCH | `/user-deactive/<id>/` | Üyeyi pasifleştirme |

### Kitap ve katalog

| Yöntem | Yol | İşlev |
|---|---|---|
| GET | `/book-all/` | Kitap listesi |
| GET | `/book-get/<slug>/` | Kitap ayrıntısı |
| GET | `/search/<name>/` | Kitap arama |
| POST | `/book-add/` | Kitap ekleme |
| PATCH | `/book-update/<slug>/` | Kitap güncelleme |
| DELETE | `/book-delete/<slug>/` | Kitap silme |
| GET | `/categori-all/` | Kategori listesi |
| GET | `/categori-get/<slug>/` | Kategori ayrıntısı |
| POST | `/categori-add/` | Kategori ekleme |
| PATCH | `/categori-update/<slug>/` | Kategori güncelleme |
| DELETE | `/categori-delete/<slug>/` | Kategori silme |
| GET | `/writer-all/` | Yazar listesi |
| GET | `/writer/get/<id>` | Yazar ayrıntısı |
| GET | `/writer/search/<name>/` | Yazar arama |
| POST | `/writer/add/` | Yazar ekleme |
| PATCH | `/writer/update/<id>/` | Yazar güncelleme |
| DELETE | `/writer/delete/<id>/` | Yazar silme |
| GET | `/language-all/` | Dil listesi |

### Ödünç işlemleri

| Yöntem | Yol | İşlev |
|---|---|---|
| GET | `/odunc-all/` | Ödünç kayıtlarını listeleme |
| GET | `/admin/<user_id>/books/` | Bir üyenin ödünç kayıtları |
| GET | `/admin/odunc-get/<id>/` | Ödünç kaydı ayrıntısı |
| POST | `/admin/odunc-create/` | Üye adına ödünç kaydı oluşturma |
| PATCH | `/admin/odunc-update/<id>/` | Bekleyen kaydı teslim edildi olarak onaylama |
| POST | `/admin/odunc-return/<loan_id>/` | Ödünç kaydını iade alma |
| GET | `/admin/odunc-list-user/` | Üyeleri ödünç özetiyle listeleme |

`/odunc-all/` dışında, bazı ödünç listeleme/oluşturma/güncelleme yollarının
`/admin/` öneki olmayan ikinci bir eşlemesi de bulunuyor.

### Rezervasyon

| Yöntem | Yol | İşlev |
|---|---|---|
| GET | `/reservation/` | Oturum açan üyenin rezervasyonları |
| POST | `/take-book/<book_id>/` | Kitabı almayı deneme; gerekirse rezervasyon |
| GET | `/admin/reservations/` | Rezervasyon listesini yönetme |
| POST | `/admin/reservations/create/` | Üye adına rezervasyon oluşturma |
| POST | `/admin/reservations/<id>/ready/` | Kitabı ayırıp hazır yapma |
| POST | `/admin/reservations/<id>/issue/` | Hazır rezervasyonu teslim etme |
| POST | `/admin/reservations/<id>/cancel/` | Rezervasyonu iptal etme |
| POST | `/return/user/<user_id>/book/<book_id>/` | Üyenin kitabını iade alma |

## Çalıştırma

API ve masaüstü istemcisi ayrı süreçlerde çalıştırılır:

```powershell
cd API
python manage.py migrate
python manage.py runserver
```

İkinci terminalde:

```powershell
cd FRONTEND
python main.py
```



## 📸 Ekran Görüntüleri

### 1. Kullanıcının Aldığı Kitaplar

<img width="1920" height="1020" alt="kullanicinin_aldigi_kitaplar" src="https://github.com/user-attachments/assets/a2849675-b17d-4a84-a699-17232b2897fa" />

### 2. Kullanıcılar

<img width="1920" height="1020" alt="kullanicilar" src="https://github.com/user-attachments/assets/1901c1bc-232d-4b22-b2dd-b62ec162bb83" />

### 3. Kitap Detayları

<img width="1920" height="1020" alt="kitap_detay" src="https://github.com/user-attachments/assets/90c2aa06-7f75-42c2-8fa8-0ae10bf1caa5" />

### 4. Kitaplar

<img width="1920" height="1020" alt="kitaplar" src="https://github.com/user-attachments/assets/fcec60a4-e530-4d53-84a5-b1cae20f7ff4" />

### 5. Ana Sayfa

<img width="1920" height="1020" alt="Anasayfa" src="https://github.com/user-attachments/assets/4960053d-448f-4ff1-b667-7828e5559d68" />

### 6. Kitap Verme

<img width="1920" height="1020" alt="kitap_verme" src="https://github.com/user-attachments/assets/293d3c3d-61b5-426c-82d8-2b08bb5c0598" />

### 7. Giriş

<img width="1122" height="758" alt="login" src="https://github.com/user-attachments/assets/9ed978bc-f1c1-4e06-9b29-420329748a03" />

### 8. Kitap Düzenleme

<img width="1920" height="1020" alt="kitap_duzenle" src="https://github.com/user-attachments/assets/4c5f146f-e4c4-4f61-8520-14d23ef50fcd" />

### 9. Stok

<img width="1920" height="1020" alt="stok" src="https://github.com/user-attachments/assets/3977a216-71f9-436c-b79f-1e48dd4f3e03" />

### 10. Rezervasyonlar

<img width="1920" height="1020" alt="rezarvaayonlar" src="https://github.com/user-attachments/assets/8cb47bde-59f5-49a8-91f2-1924c5a34819" />

### 11. Ayarlar

<img width="1920" height="1020" alt="ayarlar" src="https://github.com/user-attachments/assets/f652e642-fb5d-452b-934c-e878f5edf2b1" />

### 12. Raporlar

<img width="1920" height="1020" alt="raporlar" src="https://github.com/user-attachments/assets/49dc26e9-3159-428a-9628-7aa84845940f" />

### 13. Ödünç / İade

<img width="1920" height="1020" alt="odunc_iade" src="https://github.com/user-attachments/assets/c6d07c4c-bca0-49c0-9bdd-e825939a23be" />



