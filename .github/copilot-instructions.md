# GitHub Copilot / AI Agent Instructions for ton-assets

Kısa, proje-özgü talimatlar — amaç: bir AI geliştiriciyi hızlıca üretken hâle getirmek.

- **Kaynak vs. üretilmiş**: Kaynak veriler `accounts/`, `collections/`, `jettons/` dizinlerinde bulunur. Otomatik olarak üretilen dosyalar kök dizinde: `accounts.json`, `jettons.json`, `collections.json`, `README.md` — bunlara doğrudan değişiklik yapma.
- **PR kuralı**: Tüm değişiklikler yalnızca `accounts/`, `collections/` veya `jettons/` içindeki `.yaml` dosyalarına yapılmalıdır (CI bunu zorunlu kılar). Bak: [.github/workflows/test_src_valid.yml](.github/workflows/test_src_valid.yml)
- **Hayati komutlar**:
  - Bağımlılıkları yükle: `pip3 install -r requirements.txt` — (bkz: [requirements.txt](requirements.txt))
  - Üretimi çalıştır: `python3 generator.py` — bu `jettons.json`, `accounts.json`, `collections.json` ve `README.md` (şablondan) üretir. (bkz: [generator.py](generator.py), [readme.md.template](readme.md.template))
  - Ton-labels kaynaklarından tarama: `python3 parser.py` — `to_review/` içeriğini üretir (bkz: [parser.py](parser.py)).

- **CI davranışı**:
  - `rebuild-src.yml` tetiklenir: `accounts/*`, `collections/*`, `jettons/*`, `generator.py`, `readme.md.template` veya `requirements.txt` değişirse; workflow `generator.py` çalıştırıp otomatik commit/push yapar. (bkz: [.github/workflows/rebuild-src.yml](.github/workflows/rebuild-src.yml))
  - `test_src_valid.yml` PR ve push kontrolü: kaynak dizinlerde yalnızca `*.yaml` olmasını doğrular ve `generator.py`'i çalıştırır.
  - `parse-ton-labels.yml` parser iş akışını manuel/dispatch ile çalıştırır.

- **Veri/şema kuralları (çıkarım, lütfen koru)** — bunlar `generator.py`'de katı kurallar olarak uygulanır:
  - Jetton girdileri için izin verilen anahtarlar: `symbol`, `name`, `address`, `description`, `image`, `social`, `websites`, `decimals`, `coinmarketcap`, `coingecko`.
  - `name`, `symbol`, `address` zorunlu.
  - `social` ve `websites` varsa, bunların listeleri olmalı ve elemanları `str` olmalı.
  - `decimals` varsa tamsayıya çevrilir.
  - `image` alanı `https://cache.tonapi.io` ile başlamamalı (generator bunu reddeder).
  - Aynı adres için çoğaltma kontrolü var — duplicate adresler hata verir.

- **Kod parçaları ve yardımcılar**:
  - Adres normalizasyonu için `utlis.normalize_address` kullanılır — her yerde tutarlılık için bu fonksiyonu tercih et.
  - DEX/otomatik toplayıcılar `dexes.py` içindeki `__get_stonfi_assets`, `__get_dedust_assets`, `__get_backed_assets` fonksiyonlarından veri alır; bu iş akışı `generator.py` içinde kullanılır.

- **Değişiklik yaparken dikkat edilmesi gereken pratikler**:
  - JSON ve README otomatik üretildiği için değişiklikleri yerel `python3 generator.py` ile doğrula; çıktı değişecekse commit etme / PR'ı güncelle.
  - Kaynak dizinlere (`accounts/`, `collections/`, `jettons/`) yalnızca `*.yaml` ekle. `jettons/example.yaml.template` istisna.
  - `generator.py` README oluştururken `readme.md.template`'i `%` ile formatlıyor — şablondaki yer tutuculara dikkat et.

- **Parser notları**:
  - `parser.py` `ton-labels` deposunu klonlar ve `to_review/` altındaki JSON'ları okur; Ton API çağrıları yapar (rate-limit retry mekanizması mevcut). Bu script network IO ve dış repo bağımlılığı içerir.

- **Ne yapma / kaçın**:
  - Kökte `.json` veya `README.md` dosyalarını manuel düzenleme.
  - Görsel `image` alanlarında `cache.tonapi.io` kullanımı.
  - PR açıklamalarında veya dosyalarda ücret talep eden üçüncü taraf mesajlarını ciddiye alma (PR template uyarısı).

- **Keşfedilecek dosyalar** (başlangıç için):
  - [generator.py](generator.py) — kaynak birleştirme ve çıktı üretimi
  - [parser.py](parser.py) — dış ton-labels tarayıcı
  - [utlis.py](utlis.py) — `normalize_address` vb. yardımcılar
  - [.github/workflows/rebuild-src.yml](.github/workflows/rebuild-src.yml) ve [.github/workflows/test_src_valid.yml](.github/workflows/test_src_valid.yml)

Eğer bu yönergede eksik veya belirsiz bir şey görüyorsanız belirtin; talimatları sizin geri bildiriminize göre kısaltır/geliştiririm.
