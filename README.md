# algolotl-api

Algolotl'un API'si. Algolotl, Python öğreten oyunlaştırılmış bir kodlama uygulaması.

## Kim neye karar verir

| Alan | Karar veren |
|---|---|
| Klasör yapısı, kütüphaneler, veritabanı tasarımı, test yöntemi, görev dağılımı | Backend ekibi. Kararlar aşağıdaki "Teknik kararlar" bölümüne yazılır. |
| [`API-SOZLESMESI.md`](API-SOZLESMESI.md), `openapi.json`, `content.schema.json` | Backend ve frontend birlikte. Bu dosyaları değiştiren PR'lar frontend onayı olmadan birleşmez. |

Mobil uygulama ve içerik reposu bu API'ye yalnızca sözleşme üzerinden bağlıdır. Sözleşme bozulmadığı sürece API'nin içi istediğiniz gibi değişebilir.

## Teknoloji

Proje başında verilen kararlar:

- FastAPI
- PostgreSQL ve SQLModel
- Veritabanı değişiklikleri için Alembic
- Kod kalitesi için Ruff, testler için pytest
- Docker ile paketleme

## Yol haritası

İşler yukarıdan aşağıya sırayla ilerler. Aynı aşamadaki işler paralel yürüyebilir. Her işin ayrıntısı, ne zaman bitmiş sayılacağı ve hangi işe bağlı olduğu kendi issue'sunda yazar. Güncel durumu [Issues](https://github.com/Hasmolam/algolotl-api/issues) ve [Milestones](https://github.com/Hasmolam/algolotl-api/milestones) sayfalarından takip edin.

### 1 · Kurulum

Herkesin aynı şekilde çalıştığı iskelet, yerel ortam ve otomatik kontroller.

- [#1 Proje iskeletini kur ve teknik kararları README'ye yaz](https://github.com/Hasmolam/algolotl-api/issues/1)
- [#2 Docker Compose ile yerel geliştirme ortamı](https://github.com/Hasmolam/algolotl-api/issues/2)
- [#3 CI: her PR'da lint, test ve sözleşme kontrolü](https://github.com/Hasmolam/algolotl-api/issues/3)

### 2 · Ortak sözleşme

Kod yazılmadan önce mobil ve API'nin ortak dili belirlenir.

- [#4 Sözleşme kurallarını frontend ile birlikte kesinleştir](https://github.com/Hasmolam/algolotl-api/issues/4)
- [#5 MVP endpoint'lerinin sözleşmesi: şemalar, boş endpoint'ler ve openapi.json](https://github.com/Hasmolam/algolotl-api/issues/5)

### 3 · Temel

Veritabanı, içerik şeması, kimlik doğrulama ve test ortamı.

- [#6 Veritabanı kuralları ve ilk migration](https://github.com/Hasmolam/algolotl-api/issues/6)
- [#7 İçerik şeması ve content.schema.json](https://github.com/Hasmolam/algolotl-api/issues/7)
- [#8 Kimlik doğrulama: kayıt, giriş, token yenileme, çıkış](https://github.com/Hasmolam/algolotl-api/issues/8)
- [#9 Sağlık kontrolü ve uygulama ayarları endpoint'i](https://github.com/Hasmolam/algolotl-api/issues/9)
- [#10 Test ortamına (staging) ilk yayın](https://github.com/Hasmolam/algolotl-api/issues/10)

### 4 · İçerik

İçerik reposundaki derslerin API'ye aktarılması ve sunulması.

- [#11 İçerik içe aktarma komutu](https://github.com/Hasmolam/algolotl-api/issues/11)
- [#12 İçerik endpoint'leri: yol, ünite ve dersler](https://github.com/Hasmolam/algolotl-api/issues/12)

### 5 · Öğrenme akışı

Ders sonucu, ilerleme, XP, seri, sıralama ve hesap işlemleri.

- [#13 Ders sonucu gönderimi ve ilerleme](https://github.com/Hasmolam/algolotl-api/issues/13)
- [#14 XP ve günlük seri](https://github.com/Hasmolam/algolotl-api/issues/14)
- [#15 Haftalık sıralama](https://github.com/Hasmolam/algolotl-api/issues/15)
- [#16 Şifre sıfırlama ve hesap silme](https://github.com/Hasmolam/algolotl-api/issues/16)

### 6 · Yayına hazırlık

Güvenlik kontrolü, izleme ve yedek.

- [#17 Güvenlik gözden geçirmesi](https://github.com/Hasmolam/algolotl-api/issues/17)
- [#18 İzleme, hata takibi ve yedek](https://github.com/Hasmolam/algolotl-api/issues/18)

## Teknik kararlar

Henüz yazılmadı. Backend ekibi [#1](https://github.com/Hasmolam/algolotl-api/issues/1) ile doldurur.

## Güvenlik kuralları

- Şifreler Argon2id ile saklanır: en az 19 MiB bellek, 2 tur, 1 paralellik.
- Şifre en az 15, en fazla 128 karakterdir. Karakter türü zorunluluğu yoktur, sızdırılmış şifreler reddedilir.
- Erişim token'ı 15 dakika geçerlidir. Algoritma listesi sabittir, `iss`, `aud` ve `exp` her istekte doğrulanır.
- Yenileme token'ı her kullanımda yenilenir ve veritabanında özet olarak saklanır. Eski token tekrar kullanılırsa oturum iptal edilir. Aynı anda gelen iki yenileme isteği için kısa bir tolerans süresi vardır.
- Giriş denemeleri hem IP hem hesap bazında sınırlandırılır. Hata mesajları hangi bilginin yanlış olduğunu söylemez.
- Kayıtlı olmayan bir e-postayla giriş denendiğinde de şifre kontrolü kadar süre geçer, böylece yanıt süresinden hesabın var olup olmadığı anlaşılmaz.
- Kimlik doğrulama koduna dokunan her pull request iki kişi tarafından incelenir.

## Çalışma kuralları

- `main` dalına doğrudan push yapılamaz. Her değişiklik pull request ile gelir.
- Her pull request en az bir onay almadan birleşmez. Onaydan sonra yeni commit gelirse onay yenilenir.
- Sözleşme dosyalarını değiştiren pull request'ler ayrıca frontend onayı ister.
- Açık kalan yorumlar çözülmeden birleştirme yapılmaz.
- Dal adları `feature/kisa-aciklama` veya `fix/kisa-aciklama` biçimindedir.
- Pull request küçük tutulur ve tek bir işi yapar. Açıklamada ilgili issue yazılır (`Closes #8`).
- Birleştirme yalnızca "Squash and merge" ile yapılır, dal birleşince otomatik silinir.
- Şifre, anahtar ve token gibi gizli bilgiler repoya eklenmez. Bunlar ortam değişkenlerinde tutulur.

## Lisans

Tüm hakları saklıdır. Bu repo yalnızca görüntülenebilir, izinsiz kullanılamaz. Ayrıntılar [LICENSE](LICENSE) dosyasında.
