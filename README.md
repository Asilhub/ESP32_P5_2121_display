# P5 2121 Carwash Display — ESP32 + HUB75 (64x32, 1/16 scan)

Avtomoyka shoxobchalari uchun LED tablo dasturi. ESP32 mikrokontrolleri HUB75 RGB LED panelni boshqaradi va Serial port orqali keladigan JSON buyruqlarni ko'rsatadi: xizmat nomi, qolgan vaqt, balans, valyuta, xatolik holati.

Ko'p tilli: lotin va kirill alifbolari, UTF-8 dekoder, proporsional 16x10 shrift.

**Versiya:** 1.0.0  
**Panel:** P5 2121-3264-16S-M5 (64x32, 1/16 scan)  
**Kontroller:** ESP32 (original, 2016)

> Bu repo **faqat P5 2121-3264-16S-M5** panel uchun. HYP5-1921-64x32-8S (1/8 scan) panel uchun [ESP32_P5_display](https://github.com/Asilhub/ESP32_P5_display) reposidagi firmware ishlatiladi. Ikkalasi bir-birining o'rniga ishlamaydi — [6-bo'limga](#6-8s-versiyasidan-farqlar) qarang.

---

## 1. Apparat va ulanish

### Pinout

ESP32 va HUB75 razyomi orasidagi ulanish:

| HUB75 pin | ESP32 GPIO | Vazifasi |
| :--- | :---: | :--- |
| **R1** | 18 | Qizil, yuqori yarim |
| **G1** | 17 | Yashil, yuqori yarim |
| **B1** | 16 | Ko'k, yuqori yarim |
| **R2** | 15 | Qizil, quyi yarim |
| **G2** | 19 | Yashil, quyi yarim |
| **B2** | 21 | Ko'k, quyi yarim |
| **A** | 4 | Satr tanlash A |
| **B** | 22 | Satr tanlash B |
| **C** | 14 | Satr tanlash C |
| **D** | 13 | Satr tanlash D — **shart!** Ulanmasa panelning yarmi yonmaydi |
| **E** | 5 | Satr tanlash E (32 qatorli panelda ishlatilmaydi, ulanmasa ham bo'ladi) |
| **LAT / STB** | 26 | Latch |
| **OE** | 25 | Output Enable |
| **CLK** | 27 | Taktlash |
| **GND** | GND | Umumiy yer — kamida 2 ta GND simini ulang |

Pinlarni o'zgartirish kerak bo'lsa: [`p5_2121.ino`](p5_2121.ino) faylining boshidagi `#define` bloki.

### Quvvat

Panel ESP32 dan emas, **alohida 5 V manbadan** oziqlanadi.

- 64x32 P5 panel to'liq oq rangda ~3.5–4 A tortadi. Kamida **5 V / 5 A** blok qo'ying.
- ESP32 va panelning **GND** lari birlashtirilgan bo'lishi shart.
- Dasturda yorqinlik `setBrightness8(120)` qilib qo'yilgan (255 dan). Bu tokni cheklaydi va panelni qizib ketishdan saqlaydi. Ko'proq yorqinlik kerak bo'lsa manba quvvatini ham oshiring.

---

## 2. Dasturni yuklash

### A) Tayyor firmware bilan

Kompilyatsiya qilish shart emas. Fayllar [`release/`](release/) papkasida va [Releases](https://github.com/Asilhub/ESP32_P5_2121_display/releases) sahifasida.

**Eng oson — bitta fayl:** `p5_2121_v1.0.0_FULL.bin` → **`0x0`** manziliga.

Brauzer orqali (hech narsa o'rnatmasdan): Chrome'da [ESP Tool](https://espressif.github.io/esptool-js/) → **Connect** → COM port → Flash Address `0x0` → `p5_2121_v1.0.0_FULL.bin` → **Program**.

**4 ta alohida fayl bilan** ([`release/parts/`](release/parts/)):

| Fayl | Manzil (Address) | Tavsif |
| :--- | :--- | :--- |
| `bootloader.bin` | `0x1000` | ESP32 bootloader |
| `partitions.bin` | `0x8000` | Partition table |
| `boot_app0.bin` | `0xe000` | OTA boot app info |
| `firmware.bin` | `0x10000` | Asosiy dastur kodi |

**1. Windows'da avtomatlashtirilgan skript bilan:**

```cmd
cd release
flash.bat COM3
```

*(Skript avtomatik tarzda 4 ta faylni kerakli manzillarga yozadi)*

**2. Espressif Flash Download Tool (GUI) orqali:**

| Fayl yo'li | Address | Belgilash |
| :--- | :--- | :---: |
| `bootloader.bin` | `0x1000` | ✅ |
| `partitions.bin` | `0x8000` | ✅ |
| `boot_app0.bin` | `0xe000` | ✅ |
| `firmware.bin` | `0x10000` | ✅ |

- **Chip:** ESP32
- **WorkMode:** develop
- **SPI Speed:** 80 MHz
- **SPI Mode:** DIO
- **Flash size:** 32Mbit (yoki 4MB)

**3. Qo'lda esptool buyrug'i:**

```bash
esptool --chip esp32 -p COM3 -b 921600 write_flash 0x1000 parts/bootloader.bin 0x8000 parts/partitions.bin 0xe000 parts/boot_app0.bin 0x10000 parts/firmware.bin
```

yoki bitta fayl bilan:

```bash
esptool --chip esp32 -p COM3 -b 921600 write_flash 0x0 p5_2121_v1.0.0_FULL.bin
```

> Flash qilishdan oldin Arduino IDE **Serial Monitor**ini yoping — aks holda port band bo'ladi (`Could not open COM3, the port is busy`).

### B) Manbadan kompilyatsiya qilish

Arduino IDE da **ESP32 Dev Module** boardini tanlang.

Kerakli kutubxonalar (Library Manager orqali):

| Kutubxona | Versiya |
| :--- | :--- |
| ESP32 HUB75 LED MATRIX PANEL DMA Display | **3.0.14** |
| GFX_Lite | 2.0.0 |
| ArduinoJson | 7.x |
| esp32 board core | 3.3.7 |

> **Muhim:** HUB75 kutubxonasining **3.0.13 dan eski** versiyalari ishlamaydi. `VirtualMatrixPanel_T` va `ScanTypeMapping` faqat 3.0.13+ da mavjud.

> Arduino IDE sketch papkasi nomi `.ino` fayl nomi bilan bir xil bo'lishini talab qiladi. Repo klonlangandan keyin papkani `p5_2121` deb nomlang yoki IDE taklif qilganda rozi bo'ling.

---

## 3. JSON protokoli

Serial port, **115200 baud**, har bir buyruq `\n` (yangi qator) bilan tugaydi.

### Kalitlar

| Kalit | Turi | Tavsif |
| :--- | :--- | :--- |
| `type` | matn | Rejim yoki ko'rsatiladigan matn (lotin) |
| `typeuz` / `textuz` | matn | Xuddi `type` kabi, lotin alifbosi |
| `typekg` / `textkg` | matn | Kirill rejimi (qirg'izcha) |
| `typekr` / `textkr` | matn | Kirill rejimi |
| `value` | son | Balans yoki vaqt — **MMSS formatida**, pastdagi izohga qarang |
| `colorR1` `colorG1` `colorB1` | 0–255 | Yuqori qator / matn rangi |
| `colorR2` `colorG2` `colorB2` | 0–255 | Quyi qator / raqam rangi |

Kalitlar shu tartibda tekshiriladi: `typekr` → `textkr` → `typekg` → `textkg` → `typeuz` → `textuz` → `type`. Birinchi topilgani ishlatiladi.

### `value` maydoni — MMSS, soniya emas

`formatTime()` funksiyasi qiymatni `/100` va `%100` qiladi:

| Yuborilgan | Ekranda |
| :---: | :---: |
| `300` | `03:00` |
| `230` | `02:30` |
| `45` | `00:45` |
| `1230` | `12:30` |

Ya'ni 2 daqiqa 30 soniya uchun `230` yuboriladi, `150` emas. Soniya qismi 59 dan oshmasligi kerak (`175` yuborilsa ekranda `01:75` chiqadi).

### Rejimlar

| Kalit so'z | Rejim | Ekranda |
| :--- | :--- | :--- |
| `MECANUZ`, `CARWASH`, `KGCARWASH`, `KG` | Logotip | `MECANUZ`, nafas oluvchi oq rang |
| `PP<matn>PP` | Valyuta | Tepada `value` raqami, pastda `<matn>` |
| `KG<matn>KG` | Valyuta | Xuddi shunday, avto-tarjimasiz |
| `SUM` | Raqam | Faqat `value` raqami, markazda |
| `SOM`, `СОМ` | Valyuta | Tepada raqam, pastda `SOM` / `СОМ` |
| `TEST` | Sinov | Butun alifbo va raqamlar skroll qiladi |
| boshqa har qanday matn | Matn + vaqt | Tepada matn, pastda `MM:SS` |

**Avto-tarjimalar:**

- `PPsumPP` yoki `PPsomPP` + kirill kaliti (`typekg`/`typekr`) → pastda `СОМ` chiqadi.
- Kirill rejimida `SHAMPUN` → `ШАМПУНЬ`, `PAUZA` → `ПАУЗА`.

**Xatolik rejimi:** buzilgan JSON kelsa ekranda qizil `ERROR` yozuvi miltillaydi va tepa/past chiziqlar qizil yonadi.

---

## 4. Misollar

```json
{"type":"MECANUZ"}
```
Bo'sh turgan holat — logotip nafas oluvchi oq rangda.

```json
{"typeuz":"PPsumPP","value":10000,"colorR1":255,"colorG1":180,"colorB1":0,"colorR2":0,"colorG2":255,"colorB2":0}
```
Balans: tepada yashil `10000`, pastda sariq `SUM`.

```json
{"typekg":"PPsumPP","value":10000}
```
Xuddi shunday, lekin pastda kirillcha `СОМ`.

```json
{"typeuz":"KOPIK","value":300,"colorR1":0,"colorG1":200,"colorB1":255,"colorR2":255,"colorG2":255,"colorB2":255}
```
Xizmat ishlayapti: tepada `KOPIK`, pastda `03:00`.

```json
{"typeuz":"PAUZA","value":45,"colorR1":255,"colorG1":255,"colorB1":0}
```
Pauza, `00:45` qoldi.

```json
{"typekg":"ШАМПУНЬ","value":230}
```
To'g'ridan-to'g'ri kirill matn, `02:30`.

```json
{"typeuz":"SUV","value":0}
```
Vaqt tugadi — 1 soniyadan keyin vaqt yo'qoladi, matn markazga ko'chadi.

```json
{"typeuz":"SUM","value":1500000}
```
Katta raqam, markazda.

```json
{"type":"TEST"}
```
Barcha harf va raqamlarni tekshirish uchun skroll.

---

## 5. Ma'lum cheklovlar

Bular mavjud xatti-harakat, mijozga oldindan aytilishi kerak:

1. **`Ө`, `Ү`, `Ң` harflari ekranga chiqmaydi.** Shriftda ular bor (60, 62, 68-indekslar), lekin `getFontIndex()` funksiyasi ularni xaritalamaydi. Natijada `КӨБҮК` → `КБК`, `ЧАҢ` → `ЧА` bo'lib chiqadi. Qirg'iz tili to'liq kerak bo'lsa, `getFontIndex()` ga uch qator qo'shish kifoya.

2. **Kirill rejimida lotin `SH` → `Ш` bo'lmaydi.** O'girish harfma-harf ishlaydi: `SHAMPUN` → `СХАМПУН`. Shu sababli `SHAMPUN` va `PAUZA` uchun kodda maxsus holat yozilgan. Boshqa so'zlar uchun **to'g'ridan-to'g'ri kirill yozib yuborish** ishonchliroq.

3. **`MECANUZ` va `ERROR` rejimlarida rang parametrlari e'tiborga olinmaydi** — animatsiya har kadrda rangni qayta yozadi.

4. **Matn kengligi 64 px.** Undan uzun matn avtomatik skroll qiladi. `MECANUZ` va `SHAMPUN` aynan 63 px — zo'rg'a sig'adi, chetlarida bo'shliq qolmaydi.

5. **Ekran balandligi 32 px.** Ikki qatorli rejimlarda kirill `Ц`, `Щ` harflarining dumlari quyi chiziqqa tegishi mumkin.

---

## 6. 8S versiyasidan farqlar

Ikkala panel ham 64x32 P5, lekin qatorlarni boshqarish usuli (scan) har xil:

| Nima | HYP5-1921-64x32-8S | P5 2121-3264-16S-M5 (shu repo) |
| :--- | :--- | :--- |
| Scan | 1/8 (four-scan) | 1/16 (standart) |
| Manzil liniyalari | A, B, C | A, B, C, **D** |
| Piksel xaritalash | `FOUR_SCAN_32PX_HIGH` | `STANDARD_TWO_SCAN` (xaritalashsiz) |
| DMA konfiguratsiya | `HUB75_I2S_CFG(128, 16, 1)` — eni×2, bo'yi÷2 | `HUB75_I2S_CFG(64, 32, 1)` — panel o'lchamida |
| Firmware | [ESP32_P5_display](https://github.com/Asilhub/ESP32_P5_display) | shu repo |

> **8S firmware'ini 2121 panelga yuklasangiz:** D liniyasi hech qachon yoqilmaydi va rasm 8S uchun qayta joylashtiriladi. Natijada panelning yarmi yonmaydi, harflarning faqat bir qismi ko'rinadi. Aksincha, bu firmware'ni 8S panelga yuklasangiz rasm aralashib ketadi.

Qolgan hammasi (shrift, JSON protokoli, animatsiyalar, pinout, yorqinlik, I2S 10 MHz) 8S versiyasi bilan bir xil.

---

## 7. Fayl tuzilmasi

```
p5_2121/
├── p5_2121.ino                       Asosiy dastur
├── font16x10.h                       Proporsional shrift, 81 ta belgi
├── README.md                         Shu hujjat
└── release/
    ├── p5_2121_v1.0.0_FULL.bin       Tayyor firmware (0x0 ga yoziladi)
    ├── flash.bat                     Windows uchun flash skripti
    └── parts/
        ├── bootloader.bin            0x1000
        ├── partitions.bin            0x8000
        ├── boot_app0.bin             0xe000
        └── firmware.bin              0x10000
```

### Shrift haqida

[`font16x10.h`](font16x10.h) — 16 px balandlik, 10 px maksimal kenglik, 81 ta belgi.

- **Proporsional:** har bir belgining chap va o'ng tomonidagi bo'sh ustunlar ish vaqtida kesib tashlanadi, shuning uchun `1` va `:` kabi belgilar kam joy egallaydi.
- **Tarkibi:** `0-9`, `A-Z`, `: - . , $ % + / =`, hamda to'liq kirill alifbosi.
- **Amaldagi balandlik:** lotin bosh harflari 13 px (0–12 qatorlar), raqamlar va kirill harflari 14 px (0–13 qatorlar). `Ң` 15 px, `Ц` va `Щ` dumi bilan 16 px gacha tushadi.
