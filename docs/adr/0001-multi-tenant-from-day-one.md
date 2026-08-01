# ADR 0001: Şemayı baştan multi-tenant tasarlamak

## Durum

Kabul edildi.

## Bağlam

"Multi-tenant Organizations" ürün özellik listesinde "future module" olarak yer alıyor, yani MVP kapsamında UI/akış olarak sunulmayacak. Ancak veritabanı şemasının bunu destekleyip desteklemeyeceği ayrı bir soru: single-tenant başlayıp sonradan `organization_id` eklemek, üretimde veri geçmişi biriktikten sonra riskli ve maliyetli bir migration gerektirir (her tabloya kolon eklemek, mevcut satırları bir "default org"a atamak, tüm sorgu/yetkilendirme katmanını değiştirmek).

## Karar

Tüm domain tabloları (`uploads`, `analyses`, `analysis_findings`, `incident_reports`, `exports`, `audit_events`, vb.) baştan `organization_id` foreign key taşıyacak. `organizations` ve `memberships` tabloları Faz 1'de (Auth) kurulacak. Bugün her kullanıcı kayıt olduğunda kendisine ait bir "default organization" otomatik oluşturulur ve kendisi bu organizasyonun `owner`'ı olur — kullanıcı deneyiminde multi-tenant karmaşıklığı görünmez, ama şema ileride "invite teammate" / organizasyon paylaşımı eklendiğinde değişmeden kalır.

## Sonuçlar

- (+) İleride multi-tenant UI/akışları eklemek sadece yeni endpoint/UI gerektirir, şema migration'ı gerektirmez.
- (+) Yetkilendirme (authorization) sorguları baştan `organization_id` filtresiyle yazılır, sonradan "hangi sorguyu unuttuk" riski taşınmaz.
- (-) Bugün gereksiz görünen bir `organization_id` kolonu ve join'i her sorguda taşınır (ihmal edilebilir maliyet).
