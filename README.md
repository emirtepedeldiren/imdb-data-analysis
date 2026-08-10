# 🎬 Bir Film Gişede Tutar mı?

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/emirtepedeldiren/imdb-data-analysis/blob/master/imdb_analizi_colab.ipynb)

TMDB 5000 Movies veri seti üzerinde keşifsel veri analizi ve gişe başarısı tahmini.
Huawei Student Developers — Veri Bilimi ve Makine Öğrenmesi bootcamp final projesi.

## Problem

Bir film **vizyona girmeden önce** bilinebilecek bilgilerle (bütçe, süre, tür, çıkış ayı,
yapım şirketi sayısı) gişe başarısını tahmin edebilir miyiz?

Hedef değişken:

```
hit = 1   eğer  revenue / budget >= 2
hit = 0   diğer durumlarda
```

2 kat eşiği, `budget` kolonunun pazarlama ve dağıtım masraflarını içermemesinden
kaynaklanıyor — sektörde bir filmin başabaş noktasını bütçesinin iki katında geçtiği
kabul edilir.

## Veri

[TMDB 5000 Movie Dataset](https://www.kaggle.com/datasets/tmdb/tmdb-movie-metadata) —
4.803 film, 20 kolon. Temizlik sonrası 3.215 film modellemede kullanıldı.

## Öne çıkan bulgular

- **Bütçe ile başarı arasındaki ilişki U şeklinde:** en ucuz filmlerin %66'sı, en pahalıların
  %60'ı hit oluyor; ortada sıkışanlar (medyan 28M $) %48 ile en kötü performansı gösteriyor
- **Horror en tutarlı tür:** 14M $ medyan bütçe, %67 hit oranı
- **Çıkış ayı çok önemli:** haziran %67, eylül %43 — eylül aynı zamanda en çok film çıkan ay
- **`popularity` ve `vote_count` kasıtlı olarak kullanılmadı** — bunlar çıkış sonrası oluşuyor,
  modele koymak veri sızıntısı olurdu

| Model | Test doğruluğu | Çapraz doğrulama |
|---|---|---|
| Baseline (hep "hit") | %56,1 | — |
| Lojistik Regresyon | %59,3 | — |
| Random Forest | %65,9 | **%62,0 (± %1,2)** |

## Dosyalar

| Dosya | Açıklama |
|---|---|
| `imdb_analizi_colab.ipynb` | Ana notebook — Google Colab için hazırlandı |
| `medium_article.md` | Medium yazısının taslağı |
| `figures/` | Notebook'un ürettiği grafikler |
| `tmdb_5000_movies.csv` | Veri seti |

## Çalıştırma

### Google Colab (önerilen)

1. [colab.research.google.com](https://colab.research.google.com) → **File → Upload notebook**
2. `imdb_analizi_colab.ipynb` dosyasını yükle
3. Hücreleri sırayla çalıştır — ikinci hücre `tmdb_5000_movies.csv` dosyasını isteyecek

Colab'de pandas, numpy, matplotlib, seaborn ve scikit-learn zaten kurulu olduğu için
ek kurulum gerekmiyor.

### Yerel

```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter
jupyter notebook imdb_analizi_colab.ipynb
```

> Not: Yerelde çalıştırırken `files.upload()` içeren hücreyi atla — CSV dosyası
> notebook'un yanında olduğu sürece `pd.read_csv` doğrudan okuyacaktır.
