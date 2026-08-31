# Chloros CLI Довідка

**Версія:**

1.2.0**Створено:**

29.07.2026 19:19 ·**Переглянуто:**

30.08.2026**Аудиторія:** Оптимізовано для використання LLM; зрозуміле для людини.**Обсяг:** Усі підкоманди `chloros-cli`, доступні для користувачів, з опціями та прикладами, які можна скопіювати та вставити.

Цей документ є повним довідником щодо інструменту командного рядка `chloros-cli`, що постачається разом із MAPIR Chloros. Він навмисно є вичерпним, щоб LLM (або людина) міг скласти будь-який підтримуваний робочий процес на основі наведених нижче списків без необхідності перегляду вихідного коду.

Якщо вам потрібні лише основні моменти, перейдіть до:
- [П’ятихвилинний швидкий старт](#five-minute-quickstart)
- [Робочий процес першого підключення камери LATTICE](#lattice-camera-first-connect-workflow)
- [Робочий процес першого підключення датчика DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Режими зйомки, реєстратори та офлайн-переобробка](#capture-modes-recorders--offline-reprocess)

---

## Умовні позначення

- Усі команди мають префікс `chloros-cli`. На Windows бінарний файл має ім’я `chloros-cli.exe`; на Linux /Jetson — `chloros-cli`.
- Опціональні аргументи позначаються як `--flag`. Обов’язкові позиційні аргументи вказано без дужок.
- Якщо вказано значення за замовчуванням, у разі пропуску прапора використовується саме це значення.
- CLI — це «тонкий» клієнт HTTP, що працює через бекенд Chloros (сервер Flask на `127.0.0.1:5000`). Бекенд запускається автоматично більшістю команд. `CHLOROS_BACKEND_URL=<url>` вказує на **`lattice`**,**`project`**та**`daq pool-*`** на віддалений бекенд — основні команди (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) навмисно фіксують `http://127.0.0.1:<port>` та ігнорують його (літерал IPv4 дозволяє уникнути штрафу в розмірі ~2 с на запит, пов’язаного з перетворенням «Windows» на «`localhost`» → «`::1`»). Див. [Змінні середовища](#environment-variables).
- Для всіх викликів SDK / CLI необхідний вхід під обліковим записом Chloros+ (запустіть `chloros-cli login` один раз на кожній машині; кешується в `~/.chloros/`).
- У прикладах використовуються шляхи Linux; на Windows замініть `/home/user/...` на `C:/Users/.../...`.

---

## Загальний огляд

```
chloros-cli [global options] COMMAND [command options]
```

### Глобальні параметри

| Параметр | Опис |
| --- | --- |
| `--backend-exe PATH` | Переопределить автоматично виявлений виконуваний файл бекенду. |
| `--port N` | Порт бекенду HTTP (за замовчуванням: `5000`). |
| `-v, --verbose` | Увімкнути детальний вивід. |
| `--restart` | Примусовий перезапуск бекенда (завершує роботу всіх запущених екземплярів `backend_server.py`). |
| `--version` | Вивести версію (`Chloros CLI 1.2.0`). |
| `--help` | Показати довідку верхнього рівня. |

### Індекс команд

| Команда | Призначення |
| --- | --- |
| [`process`](#chloros-cli-process) | Обробити папку з записомSurvey3 або LATTICE від початку до кінця. |
| [`login`](#chloros-cli-login) | Авторизація цього комп’ютера за допомогою облікового запису Chloros+. |
| [`logout`](#chloros-cli-logout) | Очистити кешовані облікові дані. |
| [`status`](#chloros-cli-status) | Показати поточний стан ліцензії / автентифікації. |
| [`export-status`](#chloros-cli-export-status) | Відстеження ходу експорту Thread-4 у режимі реального часу під час виконання `process`. |
| [`language`](#chloros-cli-language) | Встановити або перелічити мову відображення CLI (підтримується 38 мов). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Папка проекту за замовчуванням (спільна з графічним інтерфейсом). |
| [`update`](#chloros-cli-update) | Перевірка наявності та встановлення оновлень CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Діагностика системи + тести на працездатність. |
| [`time-sync`](#chloros-cli-time-sync) | Стан та керування головним сервером PTP. |
| [`lattice`](#chloros-cli-lattice) | Керування камерами LATTICE та зйомка (понад 45 підкоманд). |
| [`daq`](#chloros-cli-daq) | Керування спектральними датчиками DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Відкриття та запуск збереженого проекту «Chloros» (камери + DAQ). |

---

## Встановлення

`chloros-cli` входить до складу інсталятора для настільних ПК «Chloros» на кожній підтримуваній платформі — окремого завантаження CLI немає. Встановлення пакета для платформи додає `chloros-cli` до вашого `PATH` разом із настільною програмою та бінарним файлом серверної частини, який вона використовує.

Останні завантаження: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Інсталятор також містить зручні скрипти запуску (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), які відкривають готову доCLI; вони описані в [CLI Посібнику користувача](../CLI.md) і тут не повторюються.

### Windows (.exe)

1. Завантажте інсталятор Windows зі сторінки завантажень.
2. Запустіть `Chloros-Setup-x.y.z.exe` і дотримуйтесь вказівок майстра. Шлях установки за замовчуванням — `C:\Program Files\Chloros\` (папка «CLI» розміщується в `C:\Program Files\Chloros\cli\`, яку інсталятор додає до змінної PATH).
3. Відкрийте новий термінал (`cmd.exe`, PowerShell або термінал «Windows»), щоб система змогла знайти оновлений файл `PATH`.

```powershell
chloros-cli --version
```

Інсталятор автоматично додає `chloros-cli.exe` до системного `PATH` та включає середовище виконання Arena SDK, необхідне для камер LATTICE.

### Linux amd64 (.deb)

Для Ubuntu 22.04 LTS або новіших версій / робочих станцій на базі Debian з архітектурою x86_64.

> **Ubuntu 20.04 не підтримується.** Список залежностей пакета сформовано на основі
> того, з чим фактично пов’язується бекенд, і це включає `libc6 (>= 2.34)`;
> focal постачає glibc 2.31. `apt` відмовляється від установки, замість того щоб допустити збій під час
> виконання.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Пакет .deb встановлює:
- `chloros-cli` до `/usr/bin/chloros-cli`
- Скомпільований бекенд до `/usr/lib/chloros/chloros-backend`
- Середовище виконання Arena SDK (для камер LATTICE)
- Моделі шумозаглушувачів, пакети калібрування та конфігурацію каналу оновлень

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Така сама структура, як і у файлу .deb для amd64, із збіркою CUDA, налаштованою для Jetson Orin / Orin NX / Orin Nano.

### Одноразова автентифікація на кожній машині

Кожна платформа вимагає одноразового входу за адресою Chloros+ перед тим, як почнуть працювати виклики SDK / CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Повноваження кешуються в `~/.chloros/user_session.json`.

### Перевірка встановлення

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Потрібна підписка Chloros+.**Для використання функції «CLI» необхідний активний тарифний план Chloros+.**Copper**— це базовий рівень Chloros+; кожен платний рівень Chloros+ надає доступ до CLI / SDK; лише безкоштовний рівень**Iron** не надає такого доступу. (Відповідність ідентифікаторів тарифних планів: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Оновлення за адресою [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Цей мінімальний рівень забезпечується серверною частиною, а не лише файлом CLI: запит із прапорами SDK / CLI без оплаченого тарифного плану відхиляється з кодом `403 PLAN_UPGRADE_REQUIRED`, незалежно від того, чи надходить він із `chloros-cli`, Python SDK чи з саморобного клієнта HTTP. Користувач, що вийшов із системи, отримує код `401 AUTH_REQUIRED`. Доступ працює в автономному режимі протягом пільгового періоду тарифного плану (30 днів для місячного плану, до закінчення терміну дії для річного) і припиняється після його закінчення; `chloros-cli status` продовжує працювати, щоб причина була видимою (це єдиний маршрут SDK / CLI, який не підпадає під обмеження за рівнем — `GET /api/license-status`).

---

## П’ятихвилинний швидкий старт

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Опрацюйте папку зображень за допомогою повного конвеєра Chloros (виявлення об’єктів → калібрування → віньєтування → відбивна здатність → експорт індексу).

### Короткий опис

```
chloros-cli process INPUT [OPTIONS]
```

### Аргументи позиції

| Аргумент | Опис |
| --- | --- |
| `INPUT` | Шлях до вхідної папки, що містить файли `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) або `.dng`. |

### Загальні параметри

| Параметр | За замовчуванням | Опис |
| --- | --- | --- |
| `-o, --output PATH` | нова папка з позначкою часу у вашому шляху до проекту за замовчуванням (`~/Chloros Projects`, якщо не налаштовано інакше) | Папка проєкту для створення або повторного використання. Якщо у папці вже міститься файл `project.json`, замість перезапису створюється файл-побратим `_1`/`_2`. |
| `-n, --project-name NAME` | auto (мітка часу) | Назва проєкту. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` використовує Chloros+ нейронний дебейер; повільніше, але з вищою якістю. |
| `--vignette / --no-vignette` | `--vignette` | Корекція віньєтування. |
| `--reflectance / --no-reflectance` | `--reflectance` | Калібрування відбивної здатності (використовує панель-мішень, якщо її знайдено, або калібрування NIST за серійним номером для LATTICE). Для мультиспектральних знімків LATTICE це також слугує перемикачем **продукту** відбивної здатності — див. [Перемикачі експорту за продуктом](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | вимк. | Застосувати PPK-корекції GNSS із файлів sidecar. |
| `--exposure-pin-1 MODEL` | вимкнено | Закріпити модель «pin-1» двокамерної установки «Survey3». |
| `--exposure-pin-2 MODEL` | вимкнено | Закріпити модель «pin-2». |
| `--recal-interval SECONDS` | 0 | Примусове повторне виконання математичних розрахунків калібрування кожні N секунд зйомки. |
| `--timezone-offset HOURS` | local | Перезаписати зміщення часового поясу, вбудоване у вихідні метадані. |
| `--format FORMAT` | `TIFF (16-bit)` | Один із `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | немає | Індекси рослинності (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Примусове встановлення точки входу в конвеєр для файлів LATTICE TIFF (файли Survey3 .raw не змінюються). Також передбачено «запасний вихід», який дозволяє обробляти знімки **без raw** — див. [Як виглядає папка з записами](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | увімкнено | Виводити лінійний результат дебейєризації (`Debayered_Images`). Див. [Перемикачі експорту для окремих продуктів](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | увімкнено | Виводити попередній перегляд на екрані (`Preview_Images`): RGB = баланс білого (джерело світла DAQ, якщо доступне, інакше — «сірий світ») + гамма; multispec = розтягнення у фальшивих кольорах. |
| `--radiance / --no-radiance` | увімкнено | Виводити яскравість у форматі float32 (`Radiance_Images`, Вт/м²/ср/нм). |
| `--reflectance-source {daq,target,auto}` | `auto` | Еталон для продукту відбиття LATTICE: `auto` = ціль у кадрі, що пройшла контроль якості, є абсолютним еталоном, резервний варіант — DAQ-низхідне випромінювання (ρ = π·L/E); `target` = суворий (без заміни даними DAQ); `daq` = авторитетні дані DAQ. Див. [Перемикачі експорту для окремих продуктів](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | немає | Каталог сканів **виміряного** відбиття цілі для кожної одиниці (`<serial>.csv`); у разі відсутності даних використовуються номінальні спектри T3/T4P. |
| `--array-alignment / --no-array-alignment` | увімкнено | Масиви LATTICE: застосовувати вирівнювання між модулями, зазначене у файлі XMP кожного знімка `Chloros:Alignment*`, до кожного обробленого продукту (дебейерінг / попередній перегляд / радіанс / відбиваність / індекс). Не виконується для зображень без цих тегів. |
| `--array-alignment-crop / --no-array-alignment-crop` | обрізка | Обрізати вирівняні експортовані дані до області загального перекриття масиву, щоб усі модулі мали одну площу; `--no-…` зберігає повне поле датчика (чорне заповнення за межами джерела). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Передискретизація для виправлення спотворень при вирівнюванні. `nearest` зберігає точні значення DN джерела (без змішування радіометричних значень між пікселями). |

### Параметри виявлення цілі

| Прапорець | Опис |
| --- | --- |
| `--min-target-size PIXELS` | Мінімальний розмір цільової панелі (px) для детектора. |
| `--target-clustering 0-100` | Чутливість кластеризації. |
| `--target / --targets` | Розглядати вхідну як папку, що містить лише панелі-цілі (пропустити виявлення оглядових знімків). |

### Приклади

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Перемикачі експорту для окремих продуктів (LATTICE мультиспектральний)

Обробка LATTICE розгалужується на **усі відповідні продукти за один прохід**. Чотири перемикачі для кожного типу — `--debayered`, `--preview`, `--radiance`, `--reflectance` —**за замовчуванням увімкнені**; скористайтеся формою `--no-<type>`, щоб вимкнути один із них. Головні камери RGB завжди видають лише дані з дебейерингом + попередній перегляд (без відбиття/відбиття), тому `--radiance`/`--reflectance` для них не діють. Перемикачі ігноруються для Survey3 `.raw` (які дотримуються стандартного шляху відбиття/цільового шляху). *(Старий прапор `--radiometric-output {reflectance,radiance,sensor-response}` було **вилучено** та замінено цими перемикачами; рівня `sensor-response` більше не існує.)*

| Продукт | Вихідні дані | Потрібна низхідна зйомка DAQ? |
| --- | --- | --- |
| `--debayered` | Лінійна демозаїка (`Debayered_Images`). | Ні |
| `--preview` | Попередній перегляд (`Preview_Images`): RGB = WB + гамма; multispec = розтягнення у фальшивих кольорах. | № |
| `--radiance` | float32 Вт/м²/ср/нм з повного радіометричного ланцюга (`Radiance_Images`). | № |
| `--reflectance` | uint16 коефіцієнт відбиття ρ (`32768` = 1,0), готовий до використання в Pix4D. | **Так**, якщо тільки його не закріплює ціль у кадрі, що пройшла контроль якості (див. нижче). |

`--reflectance-source` вибирає еталон відбивної здатності:**`auto`**(за замовчуванням) робить мітку в кадрі, що пройшла перевірку якості (QA),**абсолютним еталоном**— ланцюги емпіричних ліній, закріплені за міткою, перехресно оцінюються на вилучених панелях, і застосовується виміряний переможець — з переходом до поділу DAQ за напрямком на спуск (ρ = π·L/E), коли мітка відсутня або QA не пройшла;**`target`**є суворим (без заміни DAQ);**`daq`**відключає авторитет DAQ. Геометрія мішені (ArUco / фіксована область інтересу / смуга) береться з конфігурації мішеней проєкту; `--target-reflectance-dir DIR` зберігає**виміряні** скани для кожної одиниці (`<serial>.csv`), які шукаються за серійним номером/QR одиниці цілі, з номінальними спектрами T3/T4P як запасним варіантом.

Шлях відбиття DAQ автоматично визначає **відповідний за часовим штампом низхідний потік**із записаного**`.daq`**(DAQ-U/M/E)**або власного формату DAQ-M `.csv`**, знайденого разом із зображеннями. Якщо пакет калібрування для конкретної камери або DAQ не збережено в локальному кеші, конвеєр**автоматично завантажує його з AWS** під час першого використання (потрібен одноразовий доступ до Інтернету; зберігається у кеші під `~/.chloros/`).

#### Зчитування пікселів відбиття (Pix4D / Metashape / ваші власні скрипти)

Відбиття зберігається у вигляді цілого числа DN, і **значення DN, яке означає ρ = 1,0, залежить від вихідної камери**:

| Джерело | ρ = 1,0 відповідає | Як визначити |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (діапазон до ρ 2,0) | У файлі вказано XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (обрізано при ρ 1,0) | Відсутність тегів XMP `Chloros:*` — саме ця відсутність *і є* сигналом. |

**Прочитайте значення `Chloros:PixelScale` і розділіть на нього**, замість того щоб припускати, що воно є постійним. Тег визначений у домені uint16, тому він залишається незмінним `32768` у всіх форматах виводу, що змінюють масштаб — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` та `TIFF (32-bit, Percent)` — усі вони самоописані (спочатку нормалізуйте збережений тип даних назад до uint16: ×257 з 8-бітного, ×65535 з float).

> **Один випадок за задумом не містить масштабу.** Коли 8-бітний вихідний матеріал (BayerRG8) записується як 8-бітний TIFF, конвеєр *обрізає* до діапазону 0..255 замість перемасштабування, тому кожне значення вище ρ≈0,008 згладжується до 255, і файл не містить інформації про масштаб. Chloros навмисно опускає там як `Chloros:PixelScale`, так і `MicaSense:RadiometricCalibration`, і фіксує причину цього. **Якщо тег відсутній у файлі відбиття LATTICE, не слід припускати наявність масштабу — краще переекспортуйте у 16-бітний або 32-бітний формат**, замість того, щоб ділити пікселі, які ніколи не піддавалися діленню.

#### EXIF, що передається під час експорту

`process` копіює **блок GPS та його ExifIFD** вихідного знімка на кожен продукт, тому
експорт містить `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` та
`CameraSerialNumber` разом із геореференціюванням.

**`FocalLength` не є необов’язковим для фотограмметрії.** Pix4D обчислює відстань між точками на місцевості на основі
фокусної відстані та висоти над рівнем моря; за відсутності цього тегу програма використовує вкрай неправильний масштаб. Під час одного
політу над апельсиновим садом із 49 знімками відсутність тегу перетворила ділянку розміром 411 м × 160 м на реконструйовану
ділянку розміром 47,8 км × 13 км — ортофотографію розміром 455 МП, яка здебільшого складалася з порожніх даних, що потім трактувалося як проблема з мозаїкою та
проблема з BigTIFF, перш ніж хтось перевірив GSD. Якщо ваша ортофотокарта виходить у неправдоподібному
масштабі, спочатку запустіть `exiftool -FocalLength` на експортованому продукті.

Ця копія навмисно **не** є `-all:all`: структурні теги IFD0 порушують вихідні дані LATTICE під час
копіювання, а `ExifImageWidth` / `ExifImageHeight` виключено, оскільки вони описують
*джерело* зйомки — експорт, розмір якого коли-небудь змінювався, інакше містив би розміри,
, що суперечать його власному растру. XMP записується безпосередньо, а не копіюється, оскільки ExifTool
відкидає теги XMP одного й того ж виклику під час копіювання блоку XMP (що призвело б до втрати тегів калібрування MAPIR
).

### Де зберігаються результати

Результати записуються **у папці проєкту, згруповані за камерами, а потім за форматами файлів**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Папка для камери — `LATT-<sensor>-<lens>-F<filter>` для LATTICE (що відповідає EXIF-даним знімка
`Model`), а для Survey3 — `<model>_<filter>`; дві камери, що мають спільний сенсор і фільтр, але відрізняються
об&#x27;єктивами, зберігаються в окремих деревах, оскільки у них відрізняються віньєтування, поле зору та спотворення. Папка формату
папки відповідає `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, або `tiff32` для
`TIFF (32-bit, Percent)`.

> **Кожен експортований файл зберігає ім’я вихідного файлу.** Експорт у форматі Radiance з
> `capture_…_raw.tif` все одно називається `capture_…_raw.tif` — просто він знаходиться у
> `tiff32/Radiance_Images/`. **Продукт ідентифікується за папкою, а не за іменем файлу**, тому пошук за шаблоном
> `*radiance*.tif` нічого не знайде; замість цього слід шукати за каталогом.

### Записи датчика освітленості — відкалібровані `.daq` + `.csv`

`process` також обробляє записи `.daq` у вашій вхідній папці, і для цього йому **не**
потрібні жодні зображення: DAQ-U / DAQ-M / DAQ-E, запущений самостійно, забезпечує повний
збір даних, а папка, що містить лише файли `.daq`, є допустимим вхідним матеріалом.

DAQ може бути записуватися **без** його калібрування — саме це і роблять загальнодоступні
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) записуючі пристрої
(`record_daq.py`) роблять за замовчуванням: вони записують необроблені дані датчика та позначають файл таким чином, щоб
Chloros завантажував заводську калібрування цього датчика **за серійним номером** (спочатку з локального кешу,
потім із хмарного сховища MAPIR) та застосовує її. `process` записує результат назад:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` містить один рядок на кожне зчитування: мітку часу UTC, час інтеграції, загальну потужність,
фотопічний/скотопічний люкс, PPFD (та його розподіл на синій/зелений/червоний), пікову довжину хвилі, а потім
повний спектр за власною сіткою довжин хвиль датчика. `.daq` імпортує дані знову без
повторної калібрування.

У разі успіху запуск повідомляє про `Light-sensor products written: N (calibrated .daq + .csv)`.
Текст у дужках описує, що саме було записано, тобто:
`(RAW COUNTS — this sensor has no calibration bundle)` для датчика без пакета та
`(N calibrated, M raw counts)` для папки, що містить обидва. Власні заголовки бекенду
заголовки `[DAQ-EXPORT]` та `[RUN-SUMMARY]` формують свою формулювання таким самим чином — жоден із
цих трьох не може називати необроблений експорт каліброваним.

Запис DAQ-U / DAQ-M / DAQ-E запис, пакет калібрування якого неможливо отримати — ви
знаходитеся в автономному режимі або для цього датчика немає калібрування у файлі — **пропускається із зазначенням причини** у
рядку `[DAQ-EXPORT]` і ніколи не записується як «калібрований» файл, що містить необроблені дані.
Підключіться до Інтернету та запустіть процедуру повторно. Причина — це та, яку зчитувач фактично
встановив для цього файлу (нечитабельна схема, відсутність пакета, помилка запису), а у зведенні
результатів перелічено **окремі** причини — двадцять файлів, пропущених через одну причину, відображаються як одна
причина, а не двадцять її повторень.

#### Записи DAQ-A експортуються як необроблені дані підрахунку

Сімейство **DAQ-A** з’явилося до впровадження системи пакетів для кожного серійного номера і не має калібрувального пакета,
який потрібно завантажувати — натомість воно калібрується в польових умовах за допомогою мішені відбиття, саме
тому вона ніколи не потребувала такого пакета. Відмова від цих записів позбавила користувачів можливості
отримати їхні дані взагалі, тому вони експортуються під **іншою назвою**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Інша назва файлу, а не прапорець усередині файлу, оскільки ця інформація має зберегтися
під час надсилання електронною поштою у вигляді простої назви. Заголовок `.csv` вказує
`raw spectral sensor counts (NOT irradiance)` і попереджає, що значення є порівнянними
**всередині** файлу — саме для цього їх і використовує калібрування на основі цільових значень — а
не між датчиками. Фотометричні стовпці, що залежать від потужності (загальна потужність, фотопічний та
скотопічний люкс, PPFD), записуються як **NULL**, а не інтегруються з підрахунків, і у підсумку
зазначено `RAW COUNTS`, тому «експортовані» у журнал дані не можна трактувати як інтенсивність випромінювання.

Старі записи **v1.01 / v1.02** (їх записує DAQ-A-SD записує їх) не містять епохи для кожного зчитування,
лише час запису файлу. Програма зіставлення зображень↔низхідного випромінювання все ще відхиляє їх — зіставлення
кадру з часом запису було б помилковим, хоча це й непомітно — але експортер їх зчитує, і
CSV виводить `clock=daq_created_on`, тому продукт вказує, за яким годинником він працює.

### Примітки

- `process` автоматично визначає, чи є ваша папка типу «Survey3», «LATTICE» чи змішаною.
- Хід виконання передається через Server-Sent Events; на сторінці CLI відображається поточний прогрес для кожного потоку (виявлення, аналіз, обробка, експорт).
- Для Linux /Jetson, CLI перевіряє обсяг обміну пам’яті та може вивести попередження перед обробкою великих папок. Дебейер з урахуванням текстур також автоматично застосовує обмеження частоти графічного процесора на енергоефективних пристроях Jetson (Nano, Orin Nano).
- У разі успіху запуск повідомляє, скільки зображень було записано (`Image products written: N`).

#### Запуск, під час якого не записуються зображення, завершується з помилкою

Якщо ви запросили результати, а запуск не записав **жодного** — лише `project.json` та
`calibration_data.json` — `process` трактує це як збій: він виводить
`Processing finished but wrote no image products.` і **завершується з ненульовим кодом**, тому скрипт може
це виявити. У повідомленні вказано папку проєкту та типові причини:

- вхідна папка не була розпізнана як набір даних (перевірте структуру та `--input-level`), або
- усі запитувані параметри були пропущені як непридатні для цих камер (наприклад, запит на
  радіанцію/відбиваність від камер, що підтримують лише режим «RGB»).

Запустіть процес знову з параметром `--verbose` і перевірте журнал бекенду на наявність рядків `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
які пояснюють пропуски для окремих камер, що інакше не потрапляють у вихідні дані CLI.

Навмисний запуск лише з метаданими — усі продукти вимкнені та відсутність `--indices` — все одно вважається
**успішним**, оскільки порожній вихідний файл зображення є правильним результатом у цьому випадку.

Так само як і **запуск лише з датчиком освітленості**: папка із записами `.daq` за визначенням не містить зображень для експорту,
а запуск оцінюється за каліброваними файлами `.daq` / `.csv`, які були записані замість них.

---

## `chloros-cli login`

Автентифікуйте цей пристрій за допомогою облікового запису хмарного сервісу Chloros+. Дані для входу надійно зберігаються у кеші в `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Приклади

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (вилучаючи з пароля або дублюючи його частини). У разі помилки 401 CLI автоматично повторює спробу, додаючи `$$`, а потім — половину пароля без дублікатів; якщо повторна спроба завершується успіхом, система входить у систему та виводить правильний синтаксис з одинарними лапками для використання наступного разу.

> **Безграфічний режим/використання за допомогою скриптів: відсутність кешованої сесії означає інтерактивний запрос, а не швидку помилку.** Будь-яка команда, що запускає бекенд (`process`, `status`, `export-status`, `time-sync`, …) виконується без кешованої ліцензії/сесії переходить у режим інтерактивного запиту `Email:` / `Password:` на stdin перед продовженням роботи. Тому автозадача без кешованої сесії зависне в очікуванні вводу — запустіть `chloros-cli login EMAIL PASSWORD` один раз на кожній машині перед плануванням роботи в режимі без монітора.

---

## `chloros-cli logout`

Очищає кешовану сесію та примусово ініціює новий вхід при наступному виклику.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Відображає поточний рівень ліцензії (Iron/Copper/Bronze/Silver/Gold), автентифікованого користувача та кількість прив’язок до пристроїв.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Отримує поточний прогрес експорту Thread-4. Можна безпечно викликати **під час** виконання `process` з іншої оболонки.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Встановити мову відображення консолі «CLI» (підтримується 38 мов, включаючи CJK, RTL та індійські). На застарілих консолях, які не можуть відображати шрифти, автоматично використовується англійська мова.

```
chloros-cli language [LANG_CODE] [--list]
```

### Приклади

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Команди папки проєкту

Ці команди керують розташуванням папки проєкту за замовчуванням (спільне з графічним інтерфейсом).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Тільки для Jetson. Перевіряє `version_url` з `/etc/chloros/update.conf` і пропонує завантажити та встановити відповідний `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

На Linux /Jetson файл CLI також виконує **автоматичну перевірку оновлень при кожному запуску** (без блокування, ніколи не затримує виконання команди): він зчитує `/etc/chloros/update.conf`, кешує результат на 1 годину у `~/.chloros/update_cache.json`, і виводить `Update available: vX.Y.Z / Run: chloros-cli update`, якщо існує новіша версія. При будь-якій помилці та при Windows пропускається без повідомлення.

---

## `chloros-cli selftest`

Виконує 7-етапний тест на працездатність: версія, доступність портів, запуск бекенду, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), наявність моделі шумозаглушення, готовність CUDA + шумозаглушення.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Стан та керування головним сервером PTP. Хост «Chloros» запускає головний сервер PTP; камери LATTICE та блоки DAQ-E працюють у режимі підлеглих пристроїв для синхронізації часових міток між пристроями.

| Підкоманда | Опис |
| --- | --- |
| `status` | Показати стан головного сервера, пріоритети BMCA, ідентифікатор годинника. |
| `peers` | Перелік підлеглих пристроїв, виявлених через Delay_Req (камери + датчики DAQ-E). |
| `cameras` | Стан PTP для кожної камери (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Перезапуск процесу головного сервера. |
| `set-priority --priority1 N --priority2 N` | Переопределение приоритетов BMCA. |

### Приклади

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Керування камерами LATTICE. Кожна підкоманда проходить через бекенд Chloros; бекенд володіє пулом камер, тому наступні виклики CLI повторно використовують той самий відкритий дескриптор.

### Загальні параметри (спільні для більшості підкоманд)

| Параметр | Опис |
| --- | --- |
| `-d, --device N` | Індекс камери (за замовчуванням: 0). |
| `-s, --serial SN` | Конкретний серійний номер; замінює `--device`. |
| `--serials SN1,SN2,…` | Серійні номери, розділені комами, для роботи з декількома камерами. |
| `--all` | Працювати з кожною виявленою камерою. |
| `--exposure US` | Час експозиції в мікросекундах. |
| `--gain DB` | Коефіцієнт підсилення в дБ. |
| `--pixel-format FMT` | Наприклад: `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Розміри зображення. |
| `--preset {default,high_quality,high_speed,triggered}` | Застосувати попередньо встановлені налаштування. Усі режими працюють у вільному режимі, крім `triggered`, який переводить камеру в режим очікування апаратного сигналу на лінії 2 — якщо на цю лінію не подається сигнал, камера буде буде чекати нескінченно, а не здійснювати зйомку. |
| `-o, --output DIR` | Каталог виводу (за замовчуванням: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Розмір пакета GVSP. `auto` запускає проби ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Алгоритм першого підключення камери LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Довідник підкоманд

#### Виявлення та інформація

| Підкоманда | Призначення |
| --- | --- |
| `lattice info` | Перелік підключених камер (виробник, модель, серійний номер, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Аналіз хост-системи для визначення оптимальної конфігурації камери. `--no-discover` пропускає виявлення камер (швидше, аналіз лише мережевої карти). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Перевірка/виправлення налаштувань мережевої карти; оцінка пропускної здатності/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Мережеві можливості бекенду зі стабільною схемою + рекомендації щодо масиву (повертає `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` зберігає запитуване розділення, але обмежує цільову частоту кадрів — зчитайте `recommended.recommended_target_fps` і передайте його як цільове значення підключення; розглядайте це як успіх, а не як помилку. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Аналіз «що, якщо» без відкриття камер. **`--n-active` — це загальна кількість камер у мережі, а не лише у цьому масиві**— збільшуйте це значення, коли автономні камери передають потокове відео одночасно або коли бюджет пропускної здатності мережі розраховується на основі попиту, який(за замовчуванням: `len(--models)`). Завжди виводить сукупні рядки `Wire budget:` (затребувані МБ/с проти безпечної межі, що запобігає колізіям) та `Max cameras:`, а також позначає прапорцем `** OVER-SUBSCRIBED**`, коли масив перевантажує мережу — див. [Модель частоти кадрів та імпульсів масиву](#array-fps--burst-model). |
| `lattice gpu` | Показати стан графічного процесора. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Перевірити або оновити прошивку камери. Місцевий вибір `.fwa` зафіксовано: файл у `firmware/<MODEL_PREFIX>/`, що відповідає `MIN_FIRMWARE_VERSION` збірки, прошивається, якщо він присутній (лише найновіша версія як запасний варіант), тому новіший образ постачальника, розміщений на диску, залишається неактивним, доки цей параметр не буде змінено — новіші версії навмисно надходять на пристрої через підписаний маніфест AWS, що є кращим варіантом, якщо версія новіша. |
| `lattice presets [--apply NAME]` | Перелік або застосування попередніх налаштувань камери. |
| `lattice status` | Стан камери в режимі реального часу. |

#### Запис

| Підкоманда | Призначення |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Один кадр. **За замовчуванням зберігає всі типи експорту** (`--processing all`); див. [Рівні експорту знімків](#capture-export-levels-the-all-default). `--levels` зберігає явно вказану підмножину (перекриває `--processing`); `--force-daq` записує призначене значення DAQ як `.daq`-файл-супутник навіть під час зчитування лише необроблених даних. `--jpeg-quality` = якість JPEG 1–100 (за замовчуванням 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Потік даних на диск до натискання Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Попередній перегляд MJPEG у режимі реального часу в браузері. `--ae-damping` встановлює гасіння автоматичної експозиції (0,4–100). |

#### Налаштування датчика

| Підкоманда | Призначення |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Читання/запис будь-якого вузла GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Експозиція та AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Коефіцієнт підсилення та автоматичне підсилення. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Область інтересу (ROI) датчика та бінінг. |
| `lattice format [--set FMT] [--list]` | Формат пікселів. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Апаратний/програмний тригер. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (відсутність прапорців = одноразовий баланс білого) | Операції WB. Тільки камери з матрицею «RGB» / «Bayer»; на монохромних камерах M3M ця операція пропускається. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | Колірний конвеєр відображення «RGB». `natural` (за замовчуванням) — це економічний варіант обробки в режимі реального часу; `enhanced` додає усунення ореолів + яскравість + локальний контраст CLAHE для повного вигляду «hub-parity» при вартості обробки на кадр приблизно у 2 рази вищій, тому **реальна** частота кадрів нижча — збережені знімки у будь-якому разі завжди отримують повну обробку. RGB /Тільки для камер з матрицею Байєра; пропускається на монохромних M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Налаштування насиченості/контрасту дисплея (камери з фільтром RGB). Пропускається на монохромних M3M. |
| `lattice filter [--set NAME] [--list]` | Встановлення моделі фільтра камери (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Вимірювання потужності/температурних вузлів; увімкнути/вимкнути режим низького енергоспоживання в режимі очікування. |

#### Калібрування та датчики

| Підкоманда | Призначення |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Калібрування за допомогою мішені відбиття. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Команди вбудованого датчика світла, що падає зверху. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Застосувати корекцію віньєтування до існуючих зображень. |

#### Багатокамерний режим (тимчасові сесії)

| Підкоманда | Призначення |
| --- | --- |
| `lattice multi-info` | Перелік усіх камер із ролями синхронізації. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Один синхронізований кадр з кожної камери. За замовчуванням зберігає **усі типи експорту**, коли підключено постійний масив; тимчасовий варіант без масиву**проходить лише дебейєринг** (для решти спочатку запустіть `array-connect`). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Потік синхронізованих кадрів (тимчасовий). |
| `lattice multi-test [--count N]` | Тест синхронізації GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Автоматичне виявлення підключення GPIO «майстер/ведений». |

#### Вирівнювання

| Підкоманда | Призначення |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — а також регулятори детектора/збігу `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, регулятори RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, комбінація декількох кадрів `[--averaging mean\|median\|inlier_weighted]`, геометричні обмеження `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, просторові обмеження `[--roi X0,Y0,X1,Y1] [--mask PATH]` та перевизначення для кожного підлеглого `[--per-cam-override SN:KEY=VALUE]` (повторювані) | Обчислення профілю вирівнювання за даними камер у режимі реального часу. `--prefilter` за замовчуванням відповідає `gradient` (карта країв; відповідає вирівнювачу в графічному інтерфейсі/масиві — краї зберігаються у всіх спектральних діапазонах). `--matcher flann` дає результат при кількості ознак понад ~5000; `--averaging median` стійкий до одного невдалого знімка, `inlier_weighted` зважує за кількістю збігів; `--lock-scale` проектує на найближче обертання (без масштабу), `--lock-axis` обнуляє одну складову перенесення; `--mask` застосовується до кожної камери (для налаштувань окремо для кожної камери використовуйте `--per-cam-override`, наприклад, `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` відмовляється зберігати калібрування, якщо середньоквадратичне відхилення (RMS) репроекції перевищує поріг. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Знімати один вирівняний багатосмуговий кадр. `--bit-depth` за замовчуванням узгоджується з камерою; `--no-crop` зберігає повний кадр (заповнюючи порожні місця чорним); `--interpolation` (за замовчуванням `linear`) та `--border-mode`/`--border-value` (за замовчуванням `constant`/0) керують деформацією на процесорі — шлях через графічний процесор у будь-якому разі білінійним. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Багатосмугові кадри з вирівнюванням за потоком (ті самі параметри варпу, що й у `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Відобразити деталі профілю. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Змінити порядок шарів. |

#### Індекс / Математика рослинності

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Повний набір прапорців: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (повторювані), `--capture-level raw|debayered|radiance|reflectance|unknown` (перезаписати рівень запису, записаний у вихідному файлі TIFF; за замовчуванням: зчитувати з метаданих TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. До `--live` також застосовуються регулятори викривлення вирівнювання: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel` — у символах враховується регістр.** Символи повинні точно відповідати назвам каналів пресетів (у пресетах використовуються малі літери, наприклад, NDVI = `red`, `nir` — див. `--list-presets`), а частина з назвою діапазону повинна збігатися з назвою діапазону у вирівняному стеку (або бути індексом діапазону, що починається з 0, в автономному режимі). `--channel red=Red_660 --channel nir=NIR_850` працює; `--channel RED=660` завершується з помилкою `channel_map missing entries`.

#### Постійні з’єднання (Smart-Prep, потік, еквівалентний графічному інтерфейсу)

Ці команди зберігають камери відкритими у пулі бекенду під час викликів функції «CLI».

| Підкоманда | Призначення |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Додати одну камеру до пулу (одна камера, без масиву). |
| `lattice cam-disconnect [--serial SN] [--all]` | Звільнити. |
| `lattice cam-list` | Вивести список камер у пулі. |
| **`lattice array-connect`**|**Підключити постійний синхронізований масив (РЕКОМЕНДОВАНА точка входу).** Запускає повний цикл підготовки за допомогою графічного інтерфейсу. |
| `lattice array-disconnect [--array-id ID] [--all]` | Звільнити масив. |
| `lattice array-list` | Перелік підключених масивів. |
| `lattice array-status [--array-id ID]` | Кількість кадрів за секунду в режимі реального часу, PTP, остання помилка. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Один синхронізований знімок із масиву, що працює в режимі реального часу — Один / Безперервний / Інтервальний / Найшвидший. **За замовчуванням — `all`** (один файл для кожного відповідного типу експорту на кожну камеру). Пропущені камери (наприклад, RGB, виключені з вимірювання яскравості/відбиття) вказуються з кодом `Skipped: SN:<serial> (<reason>)`; показники DAQ, використані для вимірювання відбиття, зберігаються окремо та вказуються з кодом `DAQ: <path>`. Див. [Режими зйомки, записуючі пристрої та офлайн-переобробка](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Запис поєднаного індексного зображення в режимі реального часу у форматі відео/GIF (для моніторингу; потребує відкритого поєднаного потоку). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Серія знімків у форматі raw-Bayer із високою частотою кадрів (для аналізу; переобробка в автономному режимі). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Переобробка збереженої серії необроблених знімків у відкалібровані відео. |

##### Параметри `array-connect`

| Прапорець | За замовчуванням | Опис |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Автоматичне виявлення всіх камер LATTICE (потрібно ≥2) | Перший за порядковим номером є MASTER. Якщо не вказано, виявлення фільтрується за моделями LATTICE (`TRI032*`) і підключає їх усі. |
| `--line {Line0,Line2,Line3}` | `Line2` | Лінія синхронізації GPIO. |
| `--target-fps F` | auto | Частота спрацьовування тригера Master. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Переопределение вибору рівня. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | **Постійний пропускний здатність хоста, у МБ/с — значення, від якого залежить розподіл ресурсів усього масиву.** Зменшуйте його, коли масив повідомляє про кадри з пошкодженням GVSP: значення «auto» обчислюється на основі заявленої швидкості з’єднання мережевої карти, яка завищує показники USB-адаптерів, вузьких смуг PCIe та завантажених спільних мережевих інфраструктур. Зберігається у блоці збору даних масиву проєкту, тому повторне відкриття / CLI / SDK відновлює його. Див. [Стан масиву](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Апаратне об&#x27;єднання. |
| `--no-recommend` | off | Пропустити етап аналізу мережі. |
| `--no-ptp` | вимкнено | Вимкнути PTP (у цьому випадку мітки часу між камерами **не** можна порівнювати). |

### Smart-AE / Smart-Capture

Масиви LATTICE виконують безперервну автоекспозицію (AE) у фоновому режимі, щойно вони підключаються, але для збіжності параметрів щойно наведеної сцени потрібен деякий час. `array-capture --smart` — це **зручна функція**: вона чекає, поки автоекспозиція стабілізується на всіх камерах у масиві, а потім запускає зйомку. Використовуйте його, коли змінюєте сцену посеред сеансу.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

За замовчуванням політика стабілізації є консервативною: тайм-аут 5 с, вікно стабільності 1,5 с, допуск розкиду експозиції ±5 %. Налаштуйте за допомогою «SDK» (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) , якщо вам потрібна інша поведінка автоматизації.

### Рівні експорту знімків (за замовчуванням — `all`)

Починаючи з цього випуску, `lattice capture`, `lattice multi-capture` та `lattice array-capture` **за замовчуванням — `--processing all`** — один збережений файл на кожен тип експорту, що застосовується до кожної камери, відповідно до поведінки функції «Записати все» у графічному інтерфейсі. Рівні такі:

| Рівень | Вихідні дані | Застосовується до |
| --- | --- | --- |
| `raw` | Одноканальний Байєр (монохромні камери: одна смуга) безпосередньо з датчика. | Усі камери. |
| `debayered` | 3-канальна демозаїка BGR (монохромні камери: 1-канальна шкала сірих). | Усі камери. |
| `radiance` | float32 Вт/м²/ср/нм через повний радіометричний ланцюг. | Тільки мультиспектральні (M3C/M3M) — **пропускається для камер з фільтром «RGB»**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), готовий до роботи з Pix4D. | Тільки мультиспектральний, і **тільки якщо прив’язано DAQ + камера відкалібрована**; в іншому разі пропускається. |
| `preview` / `display` | Повний ланцюг попереднього перегляду в графічному інтерфейсі (CCM + WB + гамма відповідно до профілю камери). `lattice capture` називає це `preview`; `array-capture`/`multi-capture` використовуйте `display`. | Усі камери. |

Вкажіть один рівень, щоб зберегти лише цей (`--processing debayered`). Коли ви запитуєте `all`, рівні, що не застосовуються до даній кулачці, пропускаються (і фіксуються у звіті), а не сприймаються як помилка — для непідключеного або некаліброваного кулачка все одно отримуються значення `raw` / `debayered` / `preview`.

Для будь-якого кадру відбиття фактично використане значення DAQ для нисхідного випромінювання записується у **`.daq`** поруч із зображенням (щоб знімок можна було переобробити пізніше) і вказується у рядку `DAQ:`.

### Як виглядає папка із знімками

Кожен тип експорту розміщується у **власній підпапці** в каталозі `-o`, тому в багаторівневому знімку типи ніколи не змішуються:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` — це мітка часу зйомки, а `<serial>` — серійний номер камери, тож одна синхронізована група має спільну
мітку часу для всіх камер. **Зверніть увагу на одну асиметрію:** рівень `display` зберігається у папці
із назвою `preview/`, тоді як самі файли зберігають `_display` у назві — папка та суфікс відрізняються
лише для цього рівня. Невідомі рівні зберігаються у папці з їхньою власною назвою, і якщо підпапку
не вдається створити, файл записується в кореневу папку вихідних даних, а не втрачається.

**Повторна обробка папки знімків:**вкажіть `chloros-cli process` на**кореневу папку знімків**
(`output/`). `process` зазвичай імпортує лише вказану вами папку, але якщо ця папка не містить
зображень і має підпапки, програма автоматично переходить по ланцюжку — таким чином, підпапки кореневого рівня та
коренева папка `.daq` — вибираються за один раз. Кожен рівень знімка імпортується як одне зображення, причому
інші рівні доступні як режими, а не як окремі зображення для кожного рівня.

Пряме вказання **підпапки рівня** (наприклад, `output/raw/`) також працює. У цьому випадку коренева
папка `.daq` залишається поза увагою, тому скопіюйте або вкажіть дані з DAQ разом із ними, коли ви повторно отримуєте радіометричний
продукт із `raw/` — інакше збіг за часовим штампом не матиме з чим зіставитися.

**Обробка завжди починається з `raw`.** У межах кожного запису джерелом конвеєра є необроблений кадр;
`debayered`, `radiance`, `reflectance` та `preview` з’являються як режими перегляду, але ніколи не повертаються
назад через конвеєр. Повторна обробка похідного продукту призвела б до повторного застосування віньєтування, CCM та
математичні обчислення випромінювання, які вже вбудовані в його пікселі, тому «Chloros» відмовляється від цього, замість того щоб
проводити подвійну обробку. Два наслідки, про які варто знати:

- Рендери `index/` та `composite/` **ніколи** не обробляються. Це вихідні дані, а не знімки —
  рендер LUT «NDVI» не має змістовної інтерпретації яскравості.
- Папка знімків, експортована **без** `raw` (наприклад, `array-capture --processing reflectance`) не має
  легітимного джерела в конвеєрі. Ці записи імпортуються та відображаються нормально, але `process` пропускає
  їх і повідомляє про це:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Якщо вам дійсно потрібно пропустити похідний продукт — сесію хабу, записану з
  увімкненим `demosaic` або зі старої папки — `--input-level {raw,debayered,processed}` примусово встановлює точку
  входу та скасовує пропуск. Цей прапор є навмисним «запасним виходом»; `auto` (за замовчуванням)
  ніколи не обробляє запис, який не містить необроблених даних.

### Пропущені знімки в масивах зі змішаними фільтрами

Коли ви поєднуєте камери типу «RGB» та мультиспектральні камери в одному масиві, `array-capture --processing radiance` (або `reflectance`) зберігає мультиспектральні кадри та **пропускає** камери типу «RGB» — інтенсивність випромінювання за пікселем Байєра не має сенсу для широкосмугового датчика. Програма CLI явно виводить на екран кожен збережений файл (з рівнем експорту), кожне записане значення `.daq` та кожне пропущення, тому кількість файлів не єдивно:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Токени причин пропуску відповідають шаблону `<level>-not-applicable-to-rgb-cam`. Відбиття також може пропускатися з `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, а також за допомогою `dls-uncalibrated-band-<nm>`, коли діапазон переважно лежить поза радіометрично відкаліброваним діапазоном світлового датчика DAQ (~374–974 нм) — серед доступних моделей лише F988, для якої підтримується робочий процес з панеллю відбиття-панелі.

Використовуйте `--processing debayered` (або `display`), щоб включити кожну камеру незалежно від типу фільтра, або стандартний `all`, щоб отримати всі відповідні рівні для кожної камери за один раз.

---

## Режими зйомки, записуючі пристрої та офлайн-переробка

Усі вони працюють із **постійним масивом** (спочатку запустіть `array-connect`). Вони віддзеркалюють панель зйомки графічного інтерфейсу користувача.

### Режими `array-capture`

`array-capture` — це окрема команда з чотирма режимами затвора та набором параметрів експорту:

| Режим | Прапорець | Поведінка |
| --- | --- | --- |
| **Одноразовий** *(за замовчуванням)* | (немає) | Одна синхронізована група знімків, потім вихід. |
| **Безперервний** | `--continuous` | Послідовні проходи до `Ctrl+C`, `--count N` або `--duration S`. |
| **Інтервал** | `--interval S` | Один прохід кожні `S` секунд (відлічується від початку кожного проходу), ті самі межі. |
| **Найшвидший** | `--fastest` | Тільки необроблені дані + призначене значення DAQ + композитний індекс; пропускає обчислення яскравості/відбиття/відображення, щоб кадр завантажувався швидше. Передбачає використання `--processing raw --force-daq`. Пізніше переобробіть збережені дані `.daq` у калібровані продукти. |

Перемикачі експорту (можна поєднувати з будь-яким режимом; усі використовують спільний графічний інтерфейс та кінцеву точку SDK):

| Прапорець | Ефект |
| --- | --- |
| `--processing LEVEL` | Один рівень експорту або `all` (за замовчуванням). |
| `--levels L1,L2,…` | Явна підмножина типів експорту (наприклад, `raw,radiance,reflectance`); **перекриває `--processing`**. |
| `--aligned` / `--no-aligned` | Вирівнює несирі експортні дані кожного елемента масиву за [профілем вирівнювання](#alignment) (з співреєстрацією). Сирі дані залишаються невирівняними, але містять перетворення в метаданих. Якщо масив не має профілю, використовується невирівняне значення (з попередженням). |
| `--index` / `--no-index` | Зберегти / пропустити накладення індексу рослинності для кожної камери, якщо воно налаштоване. За замовчуванням: візуалізувати його. |
| `--force-daq` | Зберегти призначене показник DAQ/DLS як файл-супутник `.daq`, навіть якщо жоден обраний рівень цього не вимагає (наприклад, при знятті лише необроблених даних), щоб кадри можна було переобробити в відбиваність/індекс у автономному режимі. |
| `--smart` | Очікувати стабілізації AE на всіх камерах перед запуском (див. [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Стиснення пікселів за алгоритмом «TIFF». `deflate` (за замовчуванням) = безвтратне стиснення zlib L1 + горизонтальний предиктор, ~4,1 МБ на кадр у повній роздільній здатності; `none` = без стиснення, запис приблизно в 5 разів швидший із розміром ~6,3 МБ на кадр — використовуйте для досягнення максимальної стабільної швидкості, якщо дозволяє диск. Обидва варіанти є безвтратними та читаються однаково під час імпорту. |

> **Одноразовий запис TIFF + модель зі стабільною швидкістю.**Знімки записуються в**один**прохід у файлі TIFF, що містить пікселі + XMP + IFD0 «Make/Model» (виміряно на Mono12 у повній роздільній здатності: 36 мс у стислому форматі / 6,5 мс у нестислому, проти ~148 мс для старого методу «запис, а потім перезапис за допомогою ExifTool»); єдина робота, що залишається для ExifTool (доопрацювання EXIF-під-IFD) виконується в асинхронному фоновому процесі, і кадр вважається завершеним та готовим до імпорту навіть у разі, якщо цей процес ніколи не запускається. Зверніть увагу, що стиснення DEFLATE утримує GIL Python, тому стиснені записи**не**паралелізуються між потоками запису для кожної камери — стабільна зйомка з 8 камер у повному дозволі зі швидкістю сенсора (~10,4 кадрів/с) вимагає `--compression none`**і** диск класу NVMe (~500 МБ/с стабільної швидкості запису). Той самий параметр доступний як `compression` у `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — відео/GIF із комбінованим індексом (клас моніторингу)

Записує все, що відображається в **режимі комбінованого індексу в реальному часі**, на `.avi` (та, за бажанням, на `.gif`). Оскільки він підключається до композитного сигналу в реальному часі, комбінований потік має бути відкритим (наприклад, масив переглядається у графічному інтерфейсі), щоб кадри надходили. Він перевіряє хід запису кожні 2 с і зупиняється на `--duration`, `Ctrl+C`, або коли записник самостійно завершує роботу.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Прапорець | За замовчуванням | Опис |
| --- | --- | --- |
| `--array-id ID` | тільки масив | Цільовий масив (пропустити, якщо підключено лише один). |
| `-o, --output DIR` | `output` | Вихідний каталог (локальний для бекенду). |
| `--fps F` | `10` | Частота кадрів запису. |
| `--duration S` | до натискання Ctrl+C | Автоматична зупинка через `S` секунд. |
| `--gif` | вимкнено | Також записувати анімований GIF. |
| `--gif-only` | вимкнено | Записувати лише GIF (без `.avi`). |

### `array-burst` — серійний знімок у форматі raw-Bayer із високою частотою кадрів (аналітичного рівня)

Зчитує синхронізованийгруповий буфер циклу зчитування безпосередньо — **не потрібні ланцюжок калібрування, exiftool та попередній перегляд** — тому працює на повній швидкості зчитування камери. Записує необроблені кадри + маніфест для кожного кадру + один файл `.daq` для кожного окремого зчитування DLS під `<output>/bursts/<base>/`. Переобробіть в автономному режимі (наступна команда) або передайте `--build`, щоб зробити це негайно після зупинки.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Прапорець | За замовчуванням | Опис |
| --- | --- | --- |
| `--array-id ID` | тільки масив | Цільовий масив. |
| `-o, --output DIR` | `output` | Вихідний каталог (пакетний потік потрапляє в `<DIR>/bursts/<base>/`). |
| `--duration S` | до натискання Ctrl+C | Автоматична зупинка через `S` секунд. |
| `--max-frames N` | необмежено | Автоматична зупинка після `N` необроблених кадрів. |
| `--build` | вимкнено | Після зупинки негайно переобробити серію кадрів (так само, як `array-build-video`). |
| `--products …` | `combined:index` | З параметром `--build`: які відео створювати (див. нижче). |
| `--fps F` | `10` | З `--build`: частота кадрів вихідного відео. |
| `--save-tiffs` | вимкнено | З `--build`: також зберігати відкалібровані TIFF-файли для кожного кадру. |
| `--gif` | вимкнено | З `--build`: також записувати анімовані GIF-файли. |

### `array-build-video` — офлайн-переробка збереженої серії знімків

Збіг за часом кожного необробленого кадру з найближчим збереженим значенням `.daq` та пропускає його через **ту саму ланцюжок індексів яскравості / відбиття / ланцюг індексів, що й конвеєр імпорту**, створюючи одне або кілька відео.

`--products` — це список елементів `kind:level`, розділених комами, де `kind` ∈ `per_cam` | `combined`, а `level` ∈ `radiance` | `reflectance` | `index`. Простий `level` (без `kind:`) за замовчуванням дорівнює `per_cam`. Значенням за замовчуванням є `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Прапорець | За замовчуванням | Опис |
| --- | --- | --- |
| `--burst-dir DIR` | (обов’язковий) | Шлях до папки burst (`…/bursts/<base>/`). |
| `--products …` | Список `combined:index` | Список `kind:level`, як зазначено вище. |
| `--fps F` | `10` | Кількість кадрів на секунду (fps) вихідного відео. |
| `--save-tiffs` | вимкнено | Також зберігати відкалібровані TIFF-файли для кожного кадру разом із відео. |
| `--gif` | вимкнено | Також записувати анімовані GIF-файли. |

> **Оберіть правильний записник.** `array-record` призначений для *моніторингу* — він записує композитний сигнал у реальному часі так, як він відображається на екрані, і потребує відкритого потоку. `array-burst` → `array-build-video` призначений для *аналізу* — він зберігає необроблені дані датчика з повною швидкістю, а потім відтворює відкалібровані відео з яскравістю, відбивною здатністю та індексом, при цьому не вимагаючи перегляду в режимі реального часу.

### Монохромні (M3M) односмугові камери

Лінійка **M3M**є монохромним аналогом моделі Bayer**M3C**: одна вузькосмугова фільтрація інтерференції на камеру (наприклад, `M3M-<lens>-F<wavelength>`, `M3M-L87-F685`), тому сенсор забезпечує**один діапазон відтінків сірого** без мозаїки Байєра. Немає чого демозаїкувати, немає міжканальних перехресних перешкод, які потрібно розділяти, і немає балансу білого, який потрібно налаштовувати — весь колірний ланцюжок «RGB» просто не застосовується.

Що це означає для CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**виявляють монохромну камеру і**пропускають її, виводячи однорядкове повідомлення**, замість того, щоб застосовувати безглузді налаштування. Вони все одно нормально працюють з камерами RGB /Bayer M3C у тій самій сесії.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** все ще працюють — радіанс та відбивна здатність є *посмуговими* радіометричними картами і чітко визначені для однієї смуги. Монохромні кадри містять **ідентичну** матрицю чутливості датчика (без 3×3 розкладання), тому площина проходить через математику калібрування без змін.
- **Одна монохромна камера не може генерувати індекс рослинності.**NDVI / NDRE / тощо потребують щонайменше двох діапазонів (наприклад, Red + NIR). Щоб отримати індекс із монохромного обладнання, спрямуйте**кілька** камер M3M на різні довжини хвиль, об’єднайте їх у один багатодіапазонний стек і обчисліть індекс *саме для нього*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` символи повинні **точно** відповідати назвам каналів у пресеті (з урахуванням регістру; NDVI — це `red`,`nir` з малими літерами — див. `--list-presets`), а назва сторони діапазону вказує на діапазон у вирівняному стеку (в автономному режимі також допускаються індекси діапазонів, що починаються з 0, наприклад, `--channel red=0 --channel nir=1`).

Розпізнавальним елементом у всьому стеку є токен `M3M` у рядку моделі (він ніколи не з’являється у рядку `M3C`), який відображається у графічному інтерфейсі користувача/SDK як `is_mono`.

---

## Налаштування та оптимізація мережевої карти хоста (масиви LATTICE)

Камери LATTICE передають потік GVSP через Ethernet-адаптер хоста, тому для масивів із декількох камер **драйвер**та**розмір кільця прийому** мають таке ж значення, як і швидкість каналу. Неправильні налаштування відображаються у вигляді помилки `FRAMES WILL DROP` / `Reduce ROI to enable` на панелі «Налаштування масиву» (а також у вигляді `lattice network-analysis` / `analyze_array_network()` у «SDK»), навіть якщо самі камери працюють нормально.

### USB-адаптери 10GbE — Realtek RTL8157 («Realtek USB 10GbE Family Controller»)

| Елемент | Необхідне значення | Чому це важливо |
| --- | --- | --- |
| **Версія драйвера**|**≥ v10.67 (січень 2026 р.)**, INF `rtump64x64sta.inf` | Застарілий драйвер**2016**року (v10.65, `rtump64x64.inf`) неправильно обробляє вимкнення живлення та викликає помилки**`DRIVER_POWER_STATE_FAILURE` (синій екран `0x9F`)**під час вимкнення/перезапуску/переходу в режим сну. Процес переходу зависає (тайм-аут ~5 хв), користувач примусово вимикає живлення, а повторні некоректні вимкнення**пошкоджують репозиторій WMI**(PowerShell/інструменти починають давати збій із кодом `Invalid class`) та**блокують стек USB** під час наступного завантаження (мережева карта не вмикається; USB-накопичувачі перестають розпізнаватися). Оновіть драйвери з realtek.com (або у виробника USB-ключа), перш ніж розраховувати на коректні перезавантаження. |
| **Буфери прийому**— ключове слово `ReceiveBufferLen` |**256**(максимум для драйвера) | Кільце прийому мережевої карти. Значення за замовчуванням драйвера**32**залишає лише ~0,26 МБ корисного простору в кільці — це занадто мало для пакетів з декількох камер — тому панель масиву повідомляє про `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` і блокує з’єднання. При значенні**256**кільце є достатньо великим (**~13,5 МБ, виміряно на лабораторному хості 10GbE**), що надає конвеєру прийому (RX) реальний запас пропускної здатності для пакетних передач GVSP з декількох камер. (Чи вдасться конкретній конфігурації фактично *встановити з’єднання*, визначається двома перевірками — перевіркою допуску з урахуванням **вичерпання**та перевіркою**сукупної надмірної підписки** — а не просте порівняння обсягу пакета з розміром кільця; див. [Модель частоти кадрів масиву та пакетів](#array-fps--burst-model).) |
| **URB прийому**— ключове слово `PendingReceives` |**64** (макс.) | Блоки запитів USB у процесі передачі; збільшуйте разом із буферами прийому для поглинання пакетних передач. |
| **Jumbo Frame** — ключове слово `*JumboPacket` | **9014** | Необхідно для пакетів GVSP розміром 9000 байт (у 6 разів менше пакетів на кадр, ніж у форматі 1500). |

> ⚠️ **Оновлення драйвера мережевої карти СКИДАЄ ці розширені параметри до значень за замовчуванням.**Після оновлення або заміни драйвера мережевого адаптера**повторно застосуйте** параметри `ReceiveBufferLen=256` та `PendingReceives=64`, інакше панель масиву знову перейде в режим обмеження пропускної здатності, навіть якщо «у апаратному забезпеченні нічого не змінилося.» Це головна причина того, що раніше працююча система раптово відмовляється підключатися.

Застосуйте з **PowerShell із підвищеними правами** (замініть на назву вашого адаптера, наприклад, `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` підтримує USB-адаптери 10GbE.** Тепер програма визначає тип адаптера та налаштовує правильне ключове слово для кільця прийому: `*ReceiveBuffers`→2048 для мережевих карт PCIe (Intel I219 тощо), або `ReceiveBufferLen`→256 + `PendingReceives`→64 для контролера Realtek **USB** 10GbE (який не надає значення `*ReceiveBuffers`). Значення обмежуються максимальним значенням, що повідомляється кожним драйвером (`NumericParameterMaxValue`), тому ніколи не записуються значення, що виходять. Запустіть програму з терміналу з **підвищеними** правами; як і будь-яке налаштування на основі реєстру, зміна набуває чинності після перезапуску адаптера або перезавантаження системи. Наведені вище ручні команди `Set-NetAdapterAdvancedProperty` залишаються чудовою альтернативою — вони застосовуються на льоту (переприв’язують адаптер) без перезапуску.

### Основи мережі (усі з’єднання LATTICE)

- **Адресація:** локальна для каналу `169.254.0.0/16` (GigE Vision LLA). Хост отримує статичну адресу `169.254.x.x/16`; камери та DAQ-E самостійно призначають собі адреси в тому самому діапазоні. DHCP та шлюз не потрібні.
- **Розмір пакета:**краще використовувати розмір «джамбо» (9000), але дозвольте функції автовизначення самостійно його визначити — система повторно вимірює при кожному підключенні і вже обходить обмеження ICMP камери в 1500 байт за допомогою GVSP-пробування, тому встановлює розмір «джамбо» там, де кабель дійсно його пропускає. Встановлюйте фіксований розмір за допомогою `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` лише тоді, коли ви знаєте краще, ніж система автоматичного визначення, і віддавайте перевагу налаштуванню для кожної команди, а не постійному: фіксоване значення обходить тест, тому якщо канал насправді не може передати 9000,**кожен** запис завершиться тайм-аутом із `SC_ERR_TIMEOUT -1011` (див. [Змінні середовища](#environment-variables)).
- **Об’єм кільця RX масштабується залежно від `ReceiveBufferLen`:**за замовчуванням `32` корисний об’єм кільця становить ~0,26 МБ (занадто мало для будь-якої серії знімків з кількох камер); при максимальному значенні `256` він є великим (~13,5 МБ, виміряно на лабораторному хості 10GbE), що забезпечує реальний запас. Тож те, чи вдасться встановити з’єднання за даною конфігурацією, визначається перевіркою допуску з урахуванням навантаження**та** агрегована перевірка перевищення пропускної здатності, наведена нижче — а не просте порівняння обсягу пакетів із розміром кільця.

### Модель частоти кадрів масиву та пакетів

Як читати панель налаштувань масиву (та `lattice analyze-array` / `analyze_array_network` модуля «SDK»):

- **Потік даних підсумовується для кожної камери окремо у реальному піксельному форматі кожної камери.**Монохромні камери**M3M**передають**Mono12 (2 біта на піксель)**;**M3C**-камери з матрицею Байєра передають 8- або 12-бітний потік (TRI032S автоматично виводить BayerRG12 навіть при запиті на BayerRG8). Отже, кадр у повній роздільній здатності з 4 камер становить**~12,6 МБ, якщо всі 8-бітні, але ~25 МБ при використанні трьох 12-бітних монокамер**. Проекція визначає формат кожної камери за її моделлю (кеш ідентичності), тому пакет даних відповідає тому, що фактично передається по каналу — а не універсальному припущенню про BayerRG8.
- **Швидкість USB-адаптера Ethernet обмежена 200 МБ/с незалежно від заявлених характеристик.** Таблиця ефективності, яка перетворює швидкість каналу на стабільну величину, базується на PCIe; мережева карта USB вказує свою швидкість *Ethernet*, але обмежена шиною USB та її драйвером. USB-ключ 10GbE раніше показував ~1063 МБ/с «стабільною» — показник, який ніколи не перевірявся — а внаслідок цього 6–18 % кадрів були пошкоджені, хоча система й далі показувала нормальну цільову частоту кадрів. Мережеві адаптери, підключені через USB, тепер мають абсолютне обмеження у **200 МБ/с** (обмеженням є шина, тому швидкість не залежить від номінальних характеристик; USB адаптер 1 Гб/с забезпечує швидкість ~80 МБ/с і це на нього не впливає). `wire_ceiling_source` у записі можливостей вказує на це словами, а `nic_is_usb` позначає це прапорцем. Обидва параметри можна перезаписати за допомогою `--wire-ceiling-mbps`.
- **Підтримка — це, а не «цілий пакет проти кільця».** Одночасний пакет повинен вміститися лише в *тимчасовий затор* = `max(0, Σ per-cam arrival − host drain) × emit_window`, а не в цілому пакеті. У структурі «швидкий хост / повільна камера» (**PCIe**10G + 4× 1 GbE): надходження ≈ 320 МБ/с, вивід ≈ 1063 МБ/с) хост виводить дані швидше, ніж камери заповнюють їх, накопичення ≈ 0, тому симуляція випромінювання з повною роздільною здатністю**допускається**, хоча пакет розміром 25 МБ перевищує об’єм кільця 13,5 МБ. Підключіть ті самі чотири камери до**USB**-адаптера 10 Гб/с, і швидкість виведення становитиме 200 МБ/с, а не 1063 — надходження випереджає його, і втрати проявляються у вигляді пошкоджених кадрів, а не у вигляді зниження частоти кадрів. На хості 1 ГбЕ 31,25 МБ/с DLThr призводить до того, що надходження даних випереджає швидкість зчитування → система коректно**блокує** (для *цього* класу блоків слід зменшити область інтересу (ROI) або використовувати бінінг ≥ 2). Допуск здійснюється одним із **двох** шлюзів підключення — іншим є перевірка сумарної кількостіперевірки на недостатню кількість підписок, наведеної нижче.
- **Прогнозована частота кадрів (fps) є консервативною верхньою межею для послідовного зчитування.**Цикл зчитування хоста наразі витягує буфер кожної камери**послідовно**(приблизно по одному за-вікно передачі кожної камери), тому цикл обмежується `max(readout+emit, N × emit)`, причому передача з кожної камери обмежується**лінією доступу**камери (1 Гб/с ≈ 80 МБ/с), а не висхідним каналом хосту. Для 4-камерного масиву повної роздільної здатності 12-бітну матрицю, що становить**~2,8 кадрів/с**, що відповідає виміряним ~2,7–3,0 кадрів/с. Ця величина навмисно**не залежить від експозиції**, тому в умовах слабкого освітлення фактична частота кадрів може трохи опускатися нижче верхньої межі у міру подовження експозиції. Послідовне вилучення даних є справжнім обмежувачем частоти кадрів; його паралелізація підвищила б верхню межу до швидкості передачі даних за один цикл.
- **Сукупна надмірна підписка є серйозною перешкодою для з’єднання.**Мінімальна пропускна здатність на камеру становить**8 МБ/с**(`ARRAY_PER_CAM_FLOOR_BPS`), тому після досягнення цього мінімуму сукупний попит (`per_cam × N`) може перевищити**максимальну пропускну здатність каналу, безпечну щодо колізій**(`sustained × sim_emit_factor`). Практичні межі для повної роздільної здатності на 1 ГбЕ:**6 камер при 1500 MTU, 9 — з джамбо-кадрами**. Ця межа визначається виключно характеристиками каналу та мінімальним значенням — вона**не залежить від розміру кадру**, тому**об&#x27;єднання в групи та зменшення ROI НЕ допомагають** (вони зменшують кількість байтів на *кадр*, а не кількість байтів за *секунду* відповідно до темпу GevSCPD); єдиними рішеннями є зменшення кількості камер, відмова від джамбо-фрейміввід початку до кінця або швидший мережевий адаптер. Симптомом буде втрата пакетів GVSP, а не плавне зниження fps, тому `analyze-array` обнуляє показники досяжних fps і виводить `**OVER-SUBSCRIBED**`, а `array-connect` із зафіксованою роздільною здатністю **відмовляється встановлювати з’єднання** (в іншому випадку алгоритм «walk-down» групує кадри, що також не усуває цей тип блокування). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` знижує рівень відмови до гучного попередження для тестових робіт — див. [Змінні середовища](#environment-variables).

### Стан масиву — яка підсистема втрачає кадри

`GET /api/camera/array/<array_id>/capability` підключеного масиву містить активний
`health`-блок, який переоцінюється у **10-секундному** рухомому вікні. Він розділяє втрату кадрів
на дві причини, що потребують протилежних заходів усунення, замість того, щоб повідомляти про один «неповний»
показник, який не вказує на жодну з них:

| Поле | Що це означає | Яка підсистема |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (за послідовним портом) | Кадр **прийшов, але мав структурні помилки**— втрата пакетів GVSP. |**Мережа**: пропускна здатність каналу, темп передачі, кільце прийому мережевої карти, MTU |
| `never_arrived_rate_pct` (за послідовним номером) | Кадр **взагалі не надійшов**— камера не спрацювала, або з неї нічого не вийшло. |**Тригер / синхронізація**: кабель M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Найгірша частота передачі даних для кожної камери. | — |
| `per_cam_rate_pct` | Сукупний показник неповноти за камерою (обидві причини разом). | — |
| `stable_for_seconds` | Як довго кожна камера залишалася на рівні нижче 0,01 %. | — |

При перевищенні 5 % бекенд записує у журнал рядок `[array-health <id>] WARN` із зазначенням причини — при
першому порушенні, при зміні рівня серйозності, раз на хвилину, поки проблема існує, та один раз, коли
вона усувається. У пошкодженій половині записується `[gvsp-corrupt <SN>]` при першому збігу за камерою та
причиною, а потім підсумковий звіт кожні 60 с. Кожна оцінка все одно потрапляє до файлу журналу бекенду;
лічильники оновлюються для кожного буфера незалежно від того, що виводиться.

У цьому ж записі вказується загальний обсяг виділеної пам’яті:

| Поле | Що означає |
| --- | --- |
| `wire_ceiling_mbps` | Поточний бюджет пропускної здатності хоста, МБ/с. |
| `wire_ceiling_source` | Звідки взято це число, словами — наприклад, `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` або `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, коли `--wire-ceiling-mbps` (або поле ****Wire Budget**) встановлено це значення. |
| `nic_is_usb` | `true` для USB-адаптера Ethernet — див. обмеження 200 МБ/с вище. |

**Інтерпретація:** значення `gvsp_corrupt_rate_pct`, відмінне від нуля, при значенні `never_arrived_rate_pct`, що дорівнює 0,
означає, що запуск і синхронізація кабелю працюють ідеально, а 100 % втрат припадає на шлях мережі
— знизьте значення `--wire-ceiling-mbps` і підключіть заново. Зворотна картина вказує на
кабель синхронізації або лінію запуску.

> **`--target-fps` не є причиною пошкоджених кадрів.** Темп GevSCPD записується
> один раз під час підключення, тому зниження частоти тригера змінює робочий цикл, а не
> швидкість одночасних імпульсних пакетів. Виміряне 5-кратне скорочення навантаження не дало поліпшення;
> зниження максимальної пропускної здатності з 240 до 200 Мб/с зменшило рівень пошкоджених кадрів у тій самій установці з 10,4 %
> до 0,00 %.

> **Автоматичне скорочення потоку в процесі передачі недоступне у прошивці TRI032S.** Масив, що працює,
> не може самостійно виправити цю ситуацію; від&#x27;єднайте та під&#x27;єднайте знову, щоб механізм вибору часу підключення міг
> перепланувати роботу з урахуванням нової межі пропускної здатності.

### Симптом → усунення

| Симптом (Налаштування масиву / підключення / `analyze_array_network`) | Причина | Виправлення |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` скинуто до 32 (зазвичай після оновлення драйвера) | Встановіть `ReceiveBufferLen`→256, `PendingReceives`→64; знову відкрийте панель (перезапустіть серверну частину, якщо вона зберегла в кеші старий розмір кільця) |
| Зависання під час перезапуску/вимкнення; згодом `Invalid class` помилки WMI, мережева карта невключити, відсутні USB-накопичувачі | Старий драйвер Realtek USB 10GbE 2016 року → BSOD `0x9F` → примусове вимкнення живлення | Оновіть драйвер адаптера до версії ≥ v10.67 (2026), а потім знову застосуйте наведені вище налаштування кільця прийому |
| Підключення відбувається успішно, але повертається роздільна здатність нижче нативної | Smart-prep автоматично стискає кадр, щоб він помістився в канал | Оновіть канал / прийміть стиснення / `--force-tier slip-emit-and-capture` |
| Масив повідомляє про нормальну цільову частоту кадрів (fps), але забезпечує лише її частину; `health.gvsp_corrupt_rate_pct` відмінне від нуля, `never_arrived_rate_pct` 0 | Визначений хостом пропускний потенціал кабелю перевищує фактичну пропускну здатність (типово для USB-адаптера Ethernet, вузькій смузі PCIe або спільній структурі) | Підключіться знову з меншим значенням `--wire-ceiling-mbps` і повторно перевірте блок працездатності. **Не** `--target-fps` — Темп роботи GevSCPD фіксується під час підключення |
| Камери відсутні в опублікованих групах; `health.never_arrived_rate_pct` відмінний від нуля, `gvsp_corrupt_rate_pct` 0 | Шлях спрацьовування / синхронізації — камери не спрацьовують, це не проблема мережі | Перевірте кабель синхронізації M8 та `--line`; переконайтеся, що всі пристрої в групі перебувають у режимі охоронного режиму (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` перевищено в `analyze-array`, або відмова у підключенні з зафіксованим рішенням (`array over-subscribes the wire`) | Сумарний попит на камеру (мінімум 8 МБ/с × N камер) перевищує безпечний для уникнення колізій ліміт пропускної здатності каналу — 6 камер у повній роздільній здатності на 1 Гбіт/с при 1500 МТУ, 9 з використанням кадрів jumbo | Менша кількість камер, використання кадрів jumbo на всьому маршруті або швидша мережева карта. **ROI/біннінг НЕ допоможуть** (верхня межа не залежить від розміру кадру). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` перекриває налаштування на тестовому стенді (допускає втрату пакетів) |

---

## `chloros-cli daq`

Команди спектрального датчика. Два класи:
- **`pool-*`**— «тонкі» клієнти HTTP, які керують датчиком через постійний пул бекенду.**Це підтримуваний шлях, і єдиний, що присутній у поставленому пакеті CLI.** Бекенд володіє транспортом, тому графічний інтерфейс, скрипти CLI та SDK використовують один активний дескриптор замість того, щоб змагатися за послідовний порт.
- **Усе інше**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — прямий доступ до апаратного забезпечення, описаний нижче для повноти картини. Для їх роботи потрібен пакет `daq` Python, який**не входить до жодного з поставляних артефактів**: скомпільований CLI його не містить (`scripts/Build-CLI.ps1` встановлює `--nofollow-import-to=daq`, а транспорти `pyserial` / `bleak` / `zeroconf` разом із ним), а пакет PyPI SDK також його не містить. Вони працюють лише з вихідного коду, тому розглядайте їх як внутрішній шлях розробки MAPIR, а не як щось, до чого варто прагнути.
- **`discover` / `list`** поєднують обидва підходи: це команди для безпосереднього керування апаратним забезпеченням із вихідного коду, але в готовій збірці вони переключаються на `pool-discover`, і сканування виконує бекенд. Отже, сканування працює скрізь — це важливо, оскільки це єдиний спосіб дізнатися MAC-адресу BLE пристрою DAQ-M.

> **`chloros-cli daq --help`** (та `-h` / `help`) перелічують підкоманди `pool-*` — довідка навмисно направляється до пулу -клієнт, щоб вона відображала команди, які фактично виконуються. Якщо ви викликаєте прямупідкоманду, пов’язану з апаратним забезпеченням, у готовій збірці, вона завершується з явною помилкою, в якій вказується відсутній пакет і надається посилання на `pool-*`; ніщо не завершується без повідомлення. (`discover` / `list` є винятки — вони перенаправляють на `pool-discover` і просто працюють.)
>
> **Усе, що потрібно клієнту, доступне через `pool-*`** — підключення, потокова передача, запис відкаліброваних файлів `.daq` та зміна профілів конденсаторів. DAQ також можна керувати з Python за допомогою `chloros_sdk.connect_daq_sensor()`, який використовує той самий об’єднаний шлях.

### Алгоритм першого підключення датчика DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Довідка щодо `pool-*`

| Підкоманда | Призначення |
| --- | --- |
| `daq pool-connect` (smart-detect) | Відкрити датчик у пулі бекенду. |
| `daq pool-connect --port PORT` | DAQ-U на певному послідовному порту. |
| `daq pool-connect --ble` | DAQ-M через BLE, автоматичне сканування MAC-адреси. |
| `daq pool-connect --mac MAC` | DAQ-M через BLE за відомою MAC-адресою (передбачає `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E через Ethernet на відомому хості. |
| `daq pool-connect --eth` | DAQ-E через Ethernet, хост виявлено автоматично (mDNS + резервне використання ARP; працює з порожнім кешем ARP на Windows та Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Налаштування вікна інтеграції / стану AE. |
| `daq pool-connect --no-stream` | Підключитися, але ще не починати передачу даних (продовження з `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Профіль корекції обмеження. За замовчуванням на серверній стороні встановлено `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Сканувати кожен канал передачі даних на наявність датчиків, до яких можна підключитися, не здійснюючи підключення. **Ось як знайти MAC-адресу BLE пристрою DAQ-M.** `daq discover` / `daq list` автоматично перенаправляють сюди у готових збірках. Датчики, які вже відкриті у пулі, не відображаються — підключений DAQ-M припиняє розсилку — тому для них використовуйте `pool-list`. |
| `daq pool-list` | Показати всі датчики у пулі бекенду. |
| `daq pool-disconnect --sensor-id ID [--all]` | Звільнити. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Останні N спектральних кадрів. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Відновити / призупинити потокову передачу. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Запустити / зупинити запис у форматі .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Змінити профіль корекції конденсатора під час виконання. |

### Підкоманди прямого доступу до апаратного забезпечення (доступні лише у вихідному коді — відсутні в готових збірках)

> Наведено для повноти переліку. Для їх виконання потрібен пакет `daq` Python, а також `pyserial` / `bleak` / `zeroconf`, жодна з яких не входить до складу скомпільованої збірки CLI або репозиторію PyPI SDK — вони працюють лише після вивантаження вихідного коду з MAPIR. **Якщо ви використовуєте офіційну збірку Chloros, скористайтеся натомість наведеними вище командами `pool-*`**; вони охоплюють підключення, потокову передачу, запис та вибір камер.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Відкрити, підключитися та керувати збереженим проектом Chloros (папка з файлами `cameras.json` + `sensors.json` + `project.json`). Усе проходить через серверну частину, тому графічний інтерфейс та CLI відображають однаковий стан обладнання.

### Довідник підкоманд

| Підкоманда | Призначення |
| --- | --- |
| `project open PATH` | Вивести маніфест пристроїв проекту (камери, масиви, датчики). |
| `project devices PATH [--reconnect]` | Створити список або повторно запустити виявлення. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Підключити всі збережені камери / масиви / датчики. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Одноразовий знімок із вказаної камери або масиву. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Серія з N кадрів із вказаної камери або масиву (`-n/--count` за замовчуванням 5; `-i/--interval` інтервал між кадрами у секундах, за замовчуванням 0). Серії з масивів видаляють дублікати з повторюваних синхронізованих груп (контроль застарілості), щоб масив з частковим циклом не міг повернути N копій одного кадру; виводить результати за кожну ітерацію. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Потікна диск через завдання бекенду. `--poll-interval` = кількість секунд між опитуваннями `/stats` (за замовчуванням 2,0). |
| `project sensor read PATH NAME [--json]` | Останній кадр спектру. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Запис у форматі .daq. |
| `project run PATH RECIPE.yaml` | Виконати рецепт запису у форматі YAML/JSON. `--dry-run` перевіряє без виконання. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Обчислити вирівнювання для масиву — див. [таблицю прапорців нижче](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Вивести поточний профіль вирівнювання. |
| `project align clear PATH NAME` | Видалити профіль із кешу. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Зсунути на один трансформацію одного підлеглого. |
| `project align export PATH NAME --to FILE` | Зберегти профіль у файлі JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Завантажити збережений профіль. |

#### Параметри `project align calibrate`

| Параметр | За замовчуванням | Опис |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Метод вирівнювання. **Ці назви відрізняються від `lattice align-calibrate`**, який приймає скорочені форми `orb` / `akaze` / `phase`; ці дві команди не є взаємозамінними для цього прапора. |
| `--model {translation, rigid, affine, homography}` | `affine` | Трансформація моделі для припасування. |
| `--frames N` | `1` | Синхронізовані знімки кадрів для усереднення. |
| `--reference SN` | головна | Серійний номер опорної камери; всі інші камери деформуються відповідно до неї. |
| `--max-features N` | `5000` | Обмеження кількості ознак ORB. |
| `--ratio-threshold F` | `0.75` | Тест коефіцієнта Лоу. |
| `--ransac-threshold-px F` | `3.0` | RANSAC запороговому значенні. |
| `--min-matches N` | `15` | **Контроль якості** — відхилити розв’язок, якщо кількість збігів з внутрішніми точками менша за це значення. |
| `--max-reproj-err-px F` | `4.0` | **Поріг якості** — відхилити розв’язок, якщо помилка репроекції RMS перевищує це значення. |
| `--checkerboard RxC` | — | Геометрія плати для `--method checkerboard`, наприклад, `9x6`. |
| `--name PROFILE` | порожнє | Назва профілю, вбудована у збережений файл JSON. **Це не назва масиву** — це позиційний `NAME`. |

Ці два критерії якості є причиною того, що калібрування може успішно завершитися, але все одно
буде відхилено: профіль, який не проходить хоча б один з них, без попередження призведе до неправильної реєстрації кожного
наступного збору даних, тому його відхиляють, а не зберігають.

### Приклади

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL рецептів

`project run RECIPE.yaml` приймає файл у форматі YAML або JSON, що описує послідовність дій:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Підтримувані дії: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Дія `burst` приймає `name` (обов’язкове), `count` (за замовчуванням 5), `interval` (секунди, за замовчуванням 0), `output`, `format` та `settings` (такі самі налаштування для кожної камери, як у `apply`); масиви серійних знімків використовують той самий новийсинхронізований за групою сторожовий механізм, що й `project burst`.

Запустіть його:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Змінні середовища

| Змінна | Ефект |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Переопределяє внутрішній модуль «URL» (за замовчуванням — `http://127.0.0.1:5000`) — **враховується лише сімействами команд `lattice`, `project` та `daq pool-*`.** Основні команди (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) прив’язують `http://127.0.0.1:<port>` та ігнорують цю змінну (літерал IPv4 дозволяє уникнути штрафу у Windows `localhost`→`::1` ~2 с на запит), тому вони завжди націлюються на локальну машину. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` знижує рівень відмови у підключенні через перевищення квоти масиву (сумарний показник на-камеру &gt; безпечний від колізій верхній межі каналу з `pin_resolution`) до режиму «гучне попередження та продовження», допускаючи втрату пакетів GVSP. Використовується лише в тестових умовах — див. [Модель fps та спалахів масиву](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Встановлюється самим модулем «CLI»; вказує бекенду увімкнути паралельну обробку. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` пропускає пробу резервного переходу GVSP (лише результати ICMP). **Це вимикає підтримку джамбо-пакетів, а не просто приглушує записи в журналі** — камера відповідає на DF-пінги лише до 1500 на кожному шляху, тому цей тест є єдиним способом виявлення джамбо-пакетів. Економить ~1 с на камеру за кожне з’єднання; коштує ~1,45× від максимальної пропускної здатності каналу, якщо мережа *могла* б передавати пакети «джамбо». Файл SDK видає попередження при налаштуванні. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Фіксує розмір пакета GVSP на N байтів; повністю пропускає тестування. Краще використовувати налаштування для кожної команди (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`), ніж встановлювати його назавжди: фіксований розмір перестає адаптуватися до мережі, що знаходиться перед ним, і фіксація 9000 на шляху, який не підтримує пакети «джамбо», призведе до того, що **кожен** запис завершиться з помилкою `SC_ERR_TIMEOUT -1011` через перевищення часу очікування. |
| `TMPDIR` (Linux) | Перезаписати каталог вилучення файлів Nuitka. Команда «CLI» автоматично використовує `/mnt/ssd/tmp`, якщо він присутній. |

---

## Коди завершення

| Код | Значення |
| --- | --- |
| `0` | Успішно. |
| `1` | Загальна помилка (більшість помилок підкоманд). |
| `2` | Помилка аргументу. |
| `130` | Перервано за допомогою Ctrl+C. |

---

## Поради щодо усунення несправностей

- **&quot;Потрібен вхід&quot;** → Запустіть `chloros-cli login EMAIL PASSWORD` один раз на цьому комп’ютері.
- **&quot;backend unreachable&quot;** → Запустіть настільну програму Chloros або запустіть бінарний файл бекенду безпосередньо (`chloros-backend`), або перевірте `CHLOROS_BACKEND_URL`, якщо це віддалений доступ.
- **Команди `lattice` завершуються з помилкою «Не знайдено драйвери камери LATTICE»** → Не встановлено середовище виконання Arena SDK; CLI постачається разом із `win32api`, який можна завантажити на Windows, але бібліотека C є частиною інсталятора з графічним інтерфейсом.
- **У вікні «Array connect» / «Array Settings» з’являється повідомлення «FRAMES WILL DROP» або «Reduce ROI to enable»** → Кільце прийому мережевої карти хоста занадто мале (зазвичай після оновлення драйвера мережевої карти воно скидається до 32). Див. [Налаштування та оптимізація мережевої карти хоста](#host-nic-setup--tuning-lattice-arrays) — встановіть `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Система зависає під час перезапуску/вимкнення, потім WMI `Invalid class` / мережева карта не вмикається / відсутні USB-накопичувачі** → Застарілий драйвер USB-адаптера 10GbE спричиняє `DRIVER_POWER_STATE_FAILURE` (синій екран `0x9F`). Оновіть драйвер адаптера — див. [Налаштування та оптимізація мережевої карти хоста](#host-nic-setup--tuning-lattice-arrays).
- **Попередження про обмінну пам&#x27;ять Jetson** → Додайте обмінну пам&#x27;ять на основі файлів; файл «CLI» містить точні команди `fallocate` / `swapon`.
- **Відсутні прямі команди DAQ** → Очікується: поставлений `chloros-cli` навмисно виключає пакет `daq`, тому присутній лише `pool-*` (репозиторій PyPI SDK також його не містить). Використовуйте `pool-*`, який керує тим самим датчиком через бекенд, або `chloros_sdk.connect_daq_sensor()` з Python.

---

## Див. також

- [Python Довідник SDK](sdk-reference.md) — програмний еквівалент кожної команди CLI.
- [Посібник з датчиків DAQ](../daq/README.md) — підключення та калібрування конкретних датчиків.
- Онлайн-документація: `https://mapir.gitbook.io/chloros/cli`
