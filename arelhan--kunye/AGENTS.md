<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Künye QR Sorgulama Sistemi

Bu proje, Türkiye Sağlık Bakanlığı künye numaralarını sorgulayarak QR kod bilgilerini görüntüleyen bir Python Flask web uygulamasıdır.

## Proje Yapısı
- **Backend**: Python Flask REST API
- **Frontend**: HTML/CSS/JavaScript SPA
- **Veri Kaynağı**: sbu2.saglik.gov.tr QR sistemi
- **CORS Çözümü**: Flask-CORS ile proxy server

## Kod Yazım Kuralları
- Türkçe değişken ve fonksiyon isimleri kullan
- BeautifulSoup4 ile HTML parsing
- Responsive design için CSS Grid/Flexbox
- Modern JavaScript (ES6+) kullan
- Error handling her yerde olmalı
- Logging ve debugging desteği ekle

## API Endpoints
- `/api/kunye/<kunye_no>` - Künye bilgilerini getir
- `/health` - Sunucu durumu kontrolü
- `/api/test` - Test verileri

## Test Künye Numaraları
- 1008947258 (Ana test verisi)
- 1234567890 (Alternatif test)
- 9876543210 (Hata testi)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arelhan)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/arelhan)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
