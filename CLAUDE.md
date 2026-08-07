# AKD Horizon Theme — Proje Kuralları

Bu repo, Shopify'ın resmi **Horizon** temasının (`Shopify/horizon`) `akderelianil/horizon` fork'udur. Amaç: marka tasarımına (renk, tipografi, bileşen değerleri) uydurmak, ama temanın kendi native ayar sistemi üzerinden — Horizon'un dosya/section yapısını bozmadan, gelecekteki resmi güncellemelerle uyumlu kalarak.

## Mağaza ve ortamlar

- Mağaza: `fsgaksesuar.myshopify.com`
- **LIVE**: mağazada yayında olan tema. Sadece kullanıcı onayıyla güncellenir/yayınlanır.
- **DEV**: unpublished/preview tema. Geliştirme ve önizleme burada yapılır.
- Ortamlar `shopify.theme.toml` içinde `[environments.development]` ve `[environments.live]` olarak tanımlıdır.
- Push komutları: `shopify theme push --environment=development` (serbest) / `shopify theme push --environment=live` (**yalnızca kullanıcı açıkça onay verirse**).

## Altın kural: Hardcode yasak, ayar referansı zorunlu

Hiçbir Liquid, section veya snippet dosyasına ham değer (hex renk kodu, px, font adı, vb.) **doğrudan yazılmaz**. Her görsel değer `config/settings_schema.json` içinde tanımlı bir setting'e bağlanır ve Liquid'de `settings.*` üzerinden referans verilir.

Horizon'un kendi deseni buna zaten örnek:
```jsonc
// config/settings_schema.json
{ "type": "color_palette", "id": "color_palette", "default": { "background": "#ffffff", "foreground": "#000000" } }
...
{ "type": "color", "id": "page_text_color", "default": "{{ settings.color_palette.foreground }}" }
```
```liquid
{{ settings.page_text_color }}
```

Marka özelleştirmesi yaparken:
- **Doğru**: `config/settings_data.json` içindeki `current` bloğunda ilgili setting id'sinin değerini (`color_palette`, `type_body_font`, `button_border_radius_primary`, vb.) marka değerine güncelle.
- **Doğru**: Yeni bir görsel seçenek gerekiyorsa önce `settings_schema.json`'a (Horizon'un zaten kullandığı `color`, `color_palette`, `font_picker`, `range`, `select` gibi resmi setting tiplerinden biriyle) ekle, sonra `settings_data.json`'da değerini ver.
- **Yanlış**: Bir section/snippet dosyasına `style="color:#123456"` veya `font-family: 'Poppins'` gibi sabit değer yazmak.
- **Yanlış**: Horizon'da olmayan üçüncü parti bir CSS kütüphanesi, custom setting tipi veya section deseni eklemek.

## Upstream uyumluluğu

- `main` branch = LIVE, `dev` branch = DEV/geliştirme.
- `upstream` remote → `https://github.com/Shopify/horizon.git`.
- Periyodik senkronizasyon: `git fetch upstream && git checkout main && git merge upstream/main`, sonra `dev`'e merge/rebase edilir.
- Horizon'un dosya/section/schema yapısı olabildiğince korunur; özelleştirmeler `settings_data.json` değerleri, `settings_schema.json` varsayılanları, `locales/` ve preset'lerle sınırlı tutulur ki upstream merge'leri minimum çakışmayla ilerlesin.

## Git iş akışı

- Tüm commit / push / pull / merge / PR işlemleri Claude Code tarafından yürütülür.
- Çalışma `dev` branch'inde yapılır; değişiklikler DEV temada önizlenip doğrulandıktan sonra PR ile `main`'e alınır.
- `main`'e merge veya LIVE temaya push öncesi kullanıcı onayı istenir.

## MCP araçları

- `shopify-dev-mcp` (`.mcp.json`): Liquid/tema doğrulama (theme-check), Shopify dokümantasyonu ve GraphQL şema erişimi — tema dosyası düzenlerken kullanılır.
- `mcp__claude_ai_Shopify__*` (Admin API): mağaza/ürün/koleksiyon/sipariş yönetimi için; tema dosyası düzenleme için değildir.
- claude.ai Design System projesi (DesignSync): marka renk paleti ve tipografi tokenlarının referans kaynağı. Kod üretmez — buradaki kararlar `settings_data.json`'a elle/Claude Code aracılığıyla işlenir.
