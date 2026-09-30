# PPUI — "NightBreather" fork (durum ikonu / status-effect ikon düzeltmesi)

Bu depo, **Pocket Play UI++ (PPUI)** modunun bir fork'udur. Upstream'e göre **tek amaçlı**
fark: karakter kaydı **"Etkiler" (status effects)** listesindeki ikonları **vanilla `ui.menu`
yöntemine** çeviren düzeltme.

| | |
|---|---|
| **Upstream** | `Renegade0/PocketPlayUI` |
| **Taban commit** | `59ae27d` — *"more v2.7 compatibility fixes"* (PPUI `v2.3`) |
| **Değişen dosya** | `pocket_play_ui/override/UI.MENU` |
| **Ayrıntı + ölçümler** | `NightBreather/bg-custom-portrait-icons` → `docs/ppui-record-icons.tr.md` |

---

## Sorun

PPUI, vanilla'nın durum-ikonu listesini yorum satırına alıp durum satırlarını birleşik
`listItems` listesine taşımıştır. Orada ikonu **sabit `bam 'STATES'`** ile çizer ve
`v.current == 0` için **103 (Haste) hack'ini** uygular. Sonuç:

- `STATDESC.2DA` 3. kolonu ile tanımlı **özel BAM'ler** (ör. `nbmoon1..4`) ve **N ≥ 190**
  indeksler, karakter kaydı listesinde **yanlış — çoğu zaman hep aynı (Haste)** — görünür.
- Motorun zaten sağladığı `statusEffects[k].bam` alanı **hiç kullanılmaz**.
- Yalnızca kayıt ekranı etkilenir; portre yanındaki ikonlar **doğru** kalır.

Vanilla `ui.menu` (2.7) ikonu **motora** çözdürür; UI yalnızca bağlar:
```lua
bam       lua "statusEffects[rowNumber].bam"       -- çözülen BAM (col-3 adı ya da 'STATES')
sequence  lua "statusEffects[rowNumber].current"   -- o BAM içindeki dizi/frame
```

---

## Değişiklik (`pocket_play_ui/override/UI.MENU`)

**1) Durum satırlarına motorun verdiği BAM'i 3. eleman olarak ekle; if/else (haste exception) kaldır.**

```lua
-- ÖNCE (satır ~597-603)
for k, v in pairs(characters[currentID].statusEffects) do
    if v.current == 0 then --haste exception
        table.insert(listItems, {103, '    ' .. Infinity_FetchString(v.strRef)})
    else
        table.insert(listItems, {v.current, '    ' .. Infinity_FetchString(v.strRef)})
    end
end
```

```lua
-- SONRA (vanilla mantığı)
for k, v in pairs(characters[currentID].statusEffects) do
    table.insert(listItems, {v.current, '    ' .. Infinity_FetchString(v.strRef), v.bam})
end
```

**2) İkon kolonunda sabit BAM yerine motorun verdiği BAM'i kullan.**

```lua
-- ÖNCE (satır ~1285-1286)
bam            'STATES'
sequence    lua "listItems[rowNumber][1]"
```

```lua
-- SONRA
bam            lua "listItems[rowNumber][3] or 'STATES'"
sequence    lua "listItems[rowNumber][1]"
```

---

## Neden doğru

| Satır türü | `bam` | `sequence` | Sonuç |
|---|---|---|---|
| `STATES` satırı (`BAM_FILE = ****`) | `'STATES'` | `N + 65` | doğru ikon ✅ |
| `col-3` tek-frame custom (`nbmoon1`) | `nbmoon1` | `0` | BAM frame 0 = ikon ✅ |
| `col-3` çok-frame (`spwi510d`) | `spwi510d` | `current` | doğru frame ✅ |
| ikonsuz başlık/metin satırı (`'233'`) | `'STATES'` (fallback) | `'233'` | değişmez ✅ |

`'233'` satırlarında 3. eleman yoktur → `or 'STATES'` fallback devreye girer; mevcut
`enabled "listItems[rowNumber][1] ~= '233'"` kapısı ve `sequence` davranışı **aynen** korunur.

---

## Kurulum

Bu bir yama değil, **tam PPUI modunun kendisidir** (fork). PPUI'nin kendi `.tp2`'si değişmez:

```sh
cd "<oyun-dizini>"
weidu --language 0 --use-lang tr_TR --force-install 0 --no-exit-pause pocket_play_ui/pocket_play_ui.tp2
```

---

## Notlar / sınırlar

- `pocket_play_ui/override/UI-backup2.6.MENU` (PPUI'nin gönderdiği, **oyunun yüklemediği**
  referans yedek) eski davranışı içerir; bilinçli olarak dokunulmadı.
- Taban PPUI `v2.3`; `STATES.BAM`/`STATDESC.2DA` ölçümleri BG:EE **v2.7.3.0** içindir.
- Upstream depoda **lisans dosyası bulunamadı**; upstream koşulları geçerlidir.
