# Pocket Play UI++ (PPUI)

**Pocket Play UI++ (PPUI)** arayüz modunun bir fork'u. Orjinal sürüme göre tek fark:
karakter kaydı **"Etkiler"** listesindeki ikonları **vanilla `ui.menu` yöntemine** çeviren düzeltme.

| | |
|---|---|
| **Orjinal mod** | [`Renegade0/PocketPlayUI`](https://github.com/Renegade0/PocketPlayUI) — yazar: **Pecca** |
| **Bu fork** | [`NightBreather/PocketPlayUI`](https://github.com/NightBreather/PocketPlayUI) |
| **Temel sürüm** | `59ae27d` (*"more v2.7 compatibility fixes"*, PPUI `v2.3`) |
| **Değişen dosya** | `pocket_play_ui/override/UI.MENU` |
| **Ayrıntılı inceleme** | [`FORK.md`](FORK.md) · [`ppui-record-icons.tr.md`](https://github.com/NightBreather/bg-custom-portrait-icons/blob/main/docs/ppui-record-icons.tr.md) |

## Sorun

PPUI, vanilla'nın durum-ikonu listesini yorum satırına alıp durum satırlarını birleşik
`listItems` listesine taşır. Orada ikonu **sabit `bam 'STATES'`** ile çizer ve
`v.current == 0` için **103 (Haste) hack'ini** uygular. Sonuç:

- `STATDESC.2DA` 3. kolonu ile tanımlı **özel BAM'ler** (ör. `nbmoon1..4`) ve **N ≥ 190**
  indeksler, karakter kaydı listesinde **yanlış — çoğu zaman hep aynı (Haste)** — görünür.
- Motorun zaten sağladığı `statusEffects[k].bam` alanı **hiç kullanılmaz**.
- Yalnızca kayıt ekranı etkilenir; portre yanındaki ikonlar **doğru** kalır.

## Çözüm

`pocket_play_ui/override/UI.MENU` içinde iki değişiklik:

```lua
-- 1) durum satırları: motorun verdiği bam'i 3. eleman yap, if/else (haste exception) kaldır
for k, v in pairs(characters[currentID].statusEffects) do
    table.insert(listItems, {v.current, '    ' .. Infinity_FetchString(v.strRef), v.bam})
end

-- 2) ikon kolonu: sabit 'STATES' yerine motorun bam'i; ikonsuz satırlar fallback
bam            lua "listItems[rowNumber][3] or 'STATES'"
sequence    lua "listItems[rowNumber][1]"
```

Böylece ikon **motora** çözdürülür (vanilla davranışı): `STATES` satırı, `col-3` tek/çok
frame'li özel BAM'ler ve `N ≥ 190` indeksler doğru çizilir. Gerekçe tablosu ve ölçümler:
[`FORK.md`](FORK.md).

## Kurulum

Bu bir yama değil, **tam PPUI modudur**. Kendi `.tp2`'si değişmemiştir
(`COPY ~pocket_play_ui/override~ ~override~`):

```sh
cd "<oyun-dizini>"
weidu --language 0 --use-lang tr_TR --force-install 0 --no-exit-pause pocket_play_ui/pocket_play_ui.tp2
```

> Not: `UI-backup2.6.MENU` (PPUI'nin gönderdiği, oyunun yüklemediği referans yedek) eski
> davranışı içerir; bilinçli olarak dokunulmadı.

## Lisans ve atıf

Orjinal depoda lisans dosyası bulunamadı; mod **[Pecca](https://github.com/Renegade0)**'ya
aittir ve orjinal koşullar geçerlidir. Bu fork yalnızca yukarıdaki tek düzeltmeyi ekler.
