# API sözleşmesi

> **Taslak.** Bu belge backend ve frontend tarafından [#4](https://github.com/Hasmolam/coding-game-api/issues/4) üzerinde birlikte kesinleştirilecek. Kesinleşince bu not kaldırılır.

Bu belge mobil uygulama ile API'nin ortak dilidir. İki taraf da buna göre kod yazar, böylece biri diğerini beklemeden çalışabilir ve sonradan uyumsuzluk çıkmaz.

## Neyi kapsar, neyi kapsamaz

**Kapsar:** Endpoint'ler, istek ve yanıt şekilleri, hata biçimi, kimlik doğrulama akışı, sürümleme ve alan adlandırma.

**Kapsamaz:** Klasör yapısı, kütüphaneler, veritabanı tasarımı ve test yöntemi. Bunlar backend ekibinin kararıdır.

## Sözleşme nasıl çalışır

Tek doğruluk kaynağı bu repodaki `openapi.json` dosyasıdır. Bu dosya elle yazılmaz. FastAPI'deki istek ve yanıt modellerinden üretilir ve commit'lenir.

1. **Sözleşme PR'ı.** Yeni veya değişen bir endpoint için önce istek ve yanıt modelleri yazılır. Endpoint içi boş olarak eklenir ve `501 Not Implemented` döner. Güncel `openapi.json` aynı PR'a girer.
2. **Frontend onayı.** `openapi.json`, bu belge ve `content.schema.json` değiştiğinde frontend onayı zorunludur. Bu kural `.github/CODEOWNERS` ile otomatik uygulanır.
3. **Paralel çalışma.** Sözleşme birleşince mobil ekip tipleri üretir ve sahte API ile ekranı yazar. Backend aynı anda asıl kodu yazar.
4. **Uygulama PR'ı.** `openapi.json` değişmiyorsa frontend onayı gerekmez.

CI, koddan üretilen OpenAPI ile commit'lenen `openapi.json` dosyasını karşılaştırır. Fark varsa PR birleşmez.

## Bozmama kuralı

- Var olan bir alan silinmez, adı veya tipi değişmez. Yeni alan eklenebilir.
- Mobil uygulama tanımadığı alanları ve enum değerlerini yok sayar, çökmez.
- Kırıcı bir değişiklik gerçekten gerekiyorsa önce bu belgeye PR açılır ve iki taraf onaylar. Gerekirse eski endpoint kalır, yanına yenisi eklenir.

Telefonlarda eski uygulama sürümleri uzun süre kalır. Bu kural onların bozulmasını önler.

## Temel kurallar

| Konu | Kural |
|---|---|
| Adres | Bütün endpoint'ler `/v1` altındadır. Tek istisna sağlık kontrolüdür (`/healthz`). |
| Alan adları | `snake_case`. Örnek: `display_name`, `next_cursor`. |
| Zaman | ISO 8601, UTC ve sonunda `Z`. Örnek: `2027-03-01T09:30:00Z`. Yalnızca tarih gereken yerde `2027-03-01`. |
| Kimlikler | Kullanıcı, oturum ve ders denemesi kimlikleri metin olarak UUID'dir. İçerik kimlikleri içerik reposundaki sabit metinlerdir, örneğin `py-u1-l2-q3`. Sıralı sayı kimlik dışarı verilmez. |
| Kimlik doğrulama | `Authorization: Bearer <access_token>` başlığı kullanılır. |
| Uygulama sürümü | Her istek `X-App-Version: 1.0.0` başlığını taşır. Sürüm artık desteklenmiyorsa `unsupported_app_version` hatası döner. |
| Hatalar | Bütün hatalar aşağıdaki ortak biçimdedir (RFC 9457). |
| Listeler | Sayfalı döner: `items` ve `next_cursor`. `next_cursor` değeri `null` ise son sayfadır. İstek: `?cursor=...&limit=20`. |
| Tekrar gönderim | Ders sonucu, uygulamanın ürettiği bir UUID ile gönderilir. Aynı kimlik ikinci kez gelirse yeni kayıt oluşmaz, ilk yanıt tekrar döner. İlk gönderim `201`, tekrarı `200` alır. |
| Hesaplamalar | XP, seri, ders kilidi ve doğruluk oranı sunucuda hesaplanır. Uygulama cevapları gönderir, puan göndermez. |
| Cevaplar | Anında geri bildirim için cevaplar ders içeriğiyle birlikte gelir. Kesin sonucu sunucu verir. |
| Saat | Seri ve haftalık sıralama Türkiye saatine (`Europe/Istanbul`) göre hesaplanır. |

## Kimlik doğrulama akışı

1. Giriş veya kayıt yanıtı `access_token` (15 dakika) ve `refresh_token` döner.
2. Bir istek `401` alırsa uygulama token'ı bir kez yeniler ve isteği tekrarlar. Yenileme de başarısız olursa giriş ekranına döner.
3. Her yenileme yeni bir `refresh_token` verir, eskisi geçersiz olur.
4. Aynı anda gelen iki yenileme isteği oturumu kapatmaz. Sunucu kısa bir tolerans süresi tanır, uygulama da aynı anda tek bir yenileme isteği gönderir.
5. Eski bir `refresh_token` tolerans süresi dışında tekrar kullanılırsa o oturum tamamen kapanır.

## Hata biçimi

Hatalar `application/problem+json` türünde döner. `code` alanı sabittir. Mobil uygulama bu alana göre karar verir, `title` ve `detail` alanlarındaki metne göre değil.

```json
{
  "type": "/problems/validation-error",
  "title": "Gönderilen bilgiler geçersiz",
  "status": 422,
  "code": "validation_error",
  "detail": "Bir veya daha fazla alan hatalı.",
  "errors": [
    { "field": "password", "code": "too_short", "message": "Şifre en az 15 karakter olmalı." }
  ]
}
```

Hata kodları:

| `code` | Durum | Ne zaman |
|---|---|---|
| `validation_error` | 422 | İstek alanları geçersiz |
| `invalid_credentials` | 401 | E-posta veya şifre yanlış. Hangisinin yanlış olduğu söylenmez. |
| `token_expired` | 401 | Erişim token'ının süresi doldu, yenileme denenmeli |
| `unauthorized` | 401 | Token yok veya geçersiz |
| `forbidden` | 403 | Bu işleme izin yok |
| `not_found` | 404 | Kaynak bulunamadı |
| `conflict` | 409 | Çakışma, örneğin e-posta zaten kayıtlı veya aynı istek hâlâ işleniyor |
| `rate_limited` | 429 | Çok fazla istek. `Retry-After` başlığına uyulur. |
| `unsupported_app_version` | 426 | Uygulama güncellenmeli |
| `internal_error` | 500 | Sunucu hatası |

## MVP endpoint listesi

| Metot ve yol | Ne yapar | PRD |
|---|---|---|
| `GET /healthz` | Sunucu ayakta mı | |
| `GET /v1/app-config` | En düşük desteklenen uygulama sürümü, içerik sürümü, sunucu saati | |
| `POST /v1/auth/register` | Kayıt | G-01 |
| `POST /v1/auth/login` | Giriş | G-01 |
| `POST /v1/auth/refresh` | Token yenileme | G-01 |
| `POST /v1/auth/logout` | Çıkış, oturumu kapatır | G-01 |
| `POST /v1/auth/password/forgot` | Şifre sıfırlama e-postası. E-posta kayıtlı olsa da olmasa da aynı yanıt döner. | G-14 |
| `POST /v1/auth/password/reset` | Yeni şifre belirleme | G-14 |
| `GET /v1/me` | Profil: ad, toplam XP, seri, tamamlanan ders sayısı | G-12 |
| `PATCH /v1/me` | Görünen adı güncelleme | |
| `DELETE /v1/me` | Hesabı ve bütün verileri silme | G-13 |
| `GET /v1/courses` | Dil listesi | G-15 |
| `GET /v1/courses/{course_id}/tree` | Ünite ve ders ağacı, her ders için kilitli, açık veya tamamlandı | G-02, G-06 |
| `GET /v1/lessons/{lesson_id}` | Dersin soruları, açıklamaları ve cevapları | G-03, G-04 |
| `POST /v1/me/lesson-attempts` | Ders sonucu gönderme. Yanıt: kazanılan XP, doğruluk oranı, güncel seri. | G-05 – G-09 |
| `GET /v1/me/lesson-attempts` | Ders geçmişi, sayfalı | G-07 |
| `GET /v1/leaderboards/weekly` | Haftalık sıralama: ilk N kişi ve kullanıcının kendi sırası | G-10 |

Ayrıntılı istek ve yanıt şekilleri [#5](https://github.com/Hasmolam/coding-game-api/issues/5) ile `openapi.json` dosyasına yazılacak.

## Değişiklik önermek

Bu belgede bir şeyi değiştirmek isteyen bir issue açar veya doğrudan PR gönderir. Sözleşme PR'larına `sözleşme` etiketi eklenir.
