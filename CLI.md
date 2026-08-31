# CLI : Командний рядок

> **Повний довідник:**[Довідник CLI](reference/cli-reference.md) містить опис**усіх параметрів усіх підкоманд** та оптимізований для AI-помічників — вставте його URL у свій помічник і попросіть надати робочу команду: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Порада для інструментів ШІ:** будь-яка сторінка цього посібника доступна у вигляді необробленого коду Markdown, якщо додати `.md` до її URL (наприклад, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), а `https://mapir.gitbook.io/chloros/llms.txt` індексує весь посібник для використання великими мовними моделями (LLM).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Що таке «CLI
»

`chloros-cli` — це інтерфейс командного рядка для того самого механізму обробки, який використовує настільна програмаChloros
. Це «тонкий» клієнтHTTP
, що працює через серверну частинуChloros
(локальний сервер на `127.0.0.1:5000`) — більшість команд запускають серверну частину автоматично, тому для скрипта достатньо одного виклику `chloros-cli process …`.

Програма працює на **Windows
10/11 (x64)**та**Linux
(x86_64, а також NVIDIA Jetson arm64 на JetPack 6)** у будь-якому терміналі, без необхідності використання графічного інтерфейсу. Перевірте встановлення за допомогою команди:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Короткий огляд сімейств команд:

* **Обробка та обліковий запис** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 мов — див. [Підтримувані мови](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (лише дляLinux
/Jetson)
* **Апаратне забезпечення в режимі реального часу** — `lattice` (керування камерою LATTICE, понад 45 підкоманд), `daq pool-*` (датчики освітлення DAQ), `time-sync` (PTP)
* **Автоматизація** — `project` (запуск збереженого проєкту «Chloros
» без графічного інтерфейсу, включаючи рецепти збору даних у форматі YAML)

Глобальні параметри, про які варто знати: `--port N` (порт бекенду, за замовчуванням `5000`), `-v/--verbose`, `--restart` (примусовий перезапуск бекенду), `--backend-exe PATH`. Повний перелік див. у [ДовідціCLI
](reference/cli-reference.md).

***

## Встановлення

CLI
**входить до складу інсталятораChloros** для всіх платформ — окремого завантаженняCLI
немає. Завантажте інсталятор зі сторінки [Завантаження](download.md).

###Windows


Інсталятор розміщуєCLI
за адресою:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

та додає цю папку до вашої системи `PATH` — **відкрийте новий термінал**після встановлення, щоб оновлений `PATH` був виявлений. Інсталятор також розміщує скрипти запуску (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) у кореневій папці інсталяції, а також**Chloros
CLI
** Ярлик у меню «Пуск», кожен з яких відкриває термінал із готовим до використання `chloros-cli`.

###Linux


Встановіть `.deb` для вашої архітектури:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Це встановлює `chloros-cli` до `/usr/bin/chloros-cli` (вже встановлено `PATH`) та бекенд до версії `/usr/lib/chloros/chloros-backend`, а також середовище виконання ArenaSDK
, необхідне для камер LATTICE. Детальніше див. [Linux
Installation](linux/linux-installation.md).

### Перевірка

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Вхід та ліцензування

CLI
(а такожPython
SDK
) доступ вимагає **платного тарифного плануChloros
+**— він доступний у будь-якому платному тарифному плані; у безкоштовному — ні. Обмеження застосовується**на стороні сервера** бекендом, а не бінарним файломCLI
: виклик без входу в систему відхиляється з кодом `401 AUTH_REQUIRED`, а виклик від користувача, що увійшов у систему, на безкоштовному тарифі — з кодом `403 PLAN_UPGRADE_REQUIRED`, незалежно від того, чи надходить він із `chloros-cli`,SDK
чи з саморобного клієнтаHTTP
. Оновлення за посиланням [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Увійдіть **один раз з кожного комп’ютера**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Паролі зі спеціальними символами**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` спотворюється оболонкою (CLI
виявляє це за кодом 401 і автоматично повторює спробу, але використання одинарних лапок повністю усуває цю проблему).
{% endhint %}

Сесія зберігається в кеші як `~/.chloros/user_session.json` і продовжує працювати в автономному режимі протягом пільгового періоду тарифного плану (30 днів для місячних тарифів, до закінчення терміну дії для річних). `chloros-cli status` працює навіть без платного тарифного плану, тому причина відмови завжди видна.

{% hint style="danger" %}
**Плануєте запуск безінтерфейсних завдань? Спочатку увійдіть у систему.**Команди запуску бекенду (`process`, `status`, `export-status`, …) без**кешованої сесії**не завершується з помилкою одразу — вона переходить в інтерактивний режим із запитом `Email:` / `Password:` через stdin. Тому автоматичне завдання cron або крок CI**зависне в очікуванні введення**. Перед плануванням будь-яких завдань запустіть `chloros-cli login EMAIL 'PASSWORD'` один раз на комп’ютері.
{% endhint %}

***

## Ваш перший запуск обробки

Вкажіть `process` на папку із записами — програма автоматично виявитьSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` або їх поєднання:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Потоки прогресу відображаються в реальному часі для кожного потоку конвеєра (виявлення, аналіз, обробка, експорт), а успішне виконання завершується повідомленням про кількість збережених зображень (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Куди зберігаються результати

`process` записує дані у **папку проєкту**, а не у вашу вхідну папку:

* Якщо `-o` відсутній: проект створюється у вашій папці проектів за замовчуванням (спільній із графічним інтерфейсом; керуйте нею за допомогою `get-project-folder` / `set-project-folder`, резервний варіант — `~/Chloros Projects`), назва якого визначається `-n/--project-name` або міткою часу (`YYYYMMDD_HHMMSS`), якщо параметр не вказано.
* З `-o PATH`: ця папка **є** папкою проекту. Якщо вона вже містить файл `project.json`, замість його перезапису створюється файл із суфіксом `_1`/`_2`…

Усередині проєкту продукти групуються **за камерою, а потім за форматом файлу**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Папка камери має назву `LATT-<sensor>-<lens>-F<filter>` для LATTICE (відповідає EXIF-даним знімка `Model`) та `<model>_<filter>` (наприклад, `Survey3N_RGN`) для «Survey3
». Папка формату має назву `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` або `tiff32` для `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Кожен експортований продукт зберігає ім’я файлу-джерела.**Експорт даних про яскравість з `capture_..._raw.tif` все ще має назву `capture_..._raw.tif` — він просто знаходиться в папці `tiff32/Radiance_Images/`.**Продукт ідентифікується за папкою, а не за іменем файлу**, тому використовуйте глобальний шаблон для пошуку папки, а не для суфікса `*radiance*`.
{% endhint %}

### Параметри, які ви насправді використовуватимете

| Параметр | За замовчуванням | Що він робить |
| --- | --- | --- |
| `-o, --output PATH` | папка проєкту за замовчуванням | Розташування папки проєкту (див. вище). |
| `-n, --project-name NAME` | мітка часу | Назва проєкту. |
| `--format FMT` | `TIFF (16-bit)` | Одне з `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | немає | Індекси рослинності для експорту (див. [Індекси рослинності](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = нейронний дебейер, повільніше, найвища якість (Chloros
+, графічний процесор NVIDIA). |
| `--vignette / --no-vignette` | увімкнено | Корекція віньєтування. |
| `--reflectance / --no-reflectance` | увімкнено | Калібрування відбиття; для LATTICE це також перемикач продукту відбиття. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Примусове визначення точки входу конвеєра для TIFF-файлів LATTICE. |

Щодо всього іншого — налаштування виявлення цілей, PPK, фіксатори експозиції, прапори вирівнювання масивів — див. [розділ `process` у довіднику «CLI
»](reference/cli-reference.md).

***

## Вибір того, що експортувати (продукти LATTICE)

Обробка LATTICE розподіляється на **усі відповідні продукти за один прохід**. Чотири перемикачі для кожного продукту**за замовчуванням увімкнені**; скористайтеся формою `--no-`, щоб вимкнути один із них:

| Перемикач | Продукт |
| --- | --- |
| `--debayered` | Лінійна демозаїка → `Debayered_Images/` |
| `--preview` | Попередній перегляд (баланс білого + гамма; розтягнення у фальшивих кольорах для мультиспектральних зображень) → `Preview_Images/` |
| `--radiance` | Яскравість у форматі float32, Вт/м²/ср/нм → `Radiance_Images/` (завжди `tiff32/`) |
| `--reflectance` | uint16 коефіцієнт відбиття, готовий для Pix4D → `Reflectance_Calibrated_Images/` |

RGB
головні камери завжди випромінюють лише дані після дебейєризації + попередній перегляд — випромінювання/відбиваність за окремими діапазонами не мають значення для широкосмугового датчика, тому ці перемикачі для них не діють.Survey3
`.raw` ігнорує перемикачі та дотримується стандартного шляху відбиваності/цільового шляху.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (за замовчуванням `auto`) вибирає еталон відбиття: `auto` створює [калібрувальну мітку](calibration-targets.md) у кадрі, що відповідає вимогам контролю якості як абсолютну еталонну величину та, за відсутності мітки, повертається до значення поділу світлового потоку, що спускається, датчика DAQ (ρ = π·L/E); `target` діє строго (без заміни даними DAQ); `daq` є авторитетним для DAQ. Скани мішеней, виміряні в одиницях виміру, можна надати за допомогою `--target-reflectance-dir`.

{% hint style="info" %}
**Зчитування пікселів відбиття:**значення DN, що відповідає ρ = 1,0, є**на джерело** — Файли LATTICE вказують `Chloros:PixelScale=32768` у XMP; файлиSurvey3
використовують 65535 (і не містять тегів `Chloros:*`). Зчитуйте тег і діліть на його значення, а не припускайте, що воно є постійним. Деталі та один навмисний крайній випадок без масштабу наведено в [ДовідціCLI
](reference/cli-reference.md).
{% endhint %}

**Обробка завжди починається з `raw`.** Похідні продукти (експортовані дані з дебейєрингу, радіанції та відбиття) ніколи не повертаються назад у конвеєр — їх повторне імпортування та обробка призвели б до подвійного застосування математичних операцій калібрування, томуChloros
пропускає їх і повідомляє про це. `--input-level` — це спеціально передбачений «запасний вихід» на випадок, коли вам дійсно потрібно примусово задати точку входу.

***

## Коли запуск завершується з помилкою

Починаючи з версії 1.2.0, `process` видає гучну помилку замість того, щоб «успішно завершитися», не показуючи жодних результатів:

* Запуск, який **запитував продукти, але не записав жодного**— лише `project.json` та `calibration_data.json` — виводить `Processing finished but wrote no image products.` і**завершується з ненульовим кодом**, тож скрипти можуть це виявити. Типові причини: вхідна папка не була розпізнана як запис (перевірте структуру та `--input-level`), або кожен запитуваний продукт був непридатним для цих камер (наприклад, запит на інтенсивність випромінювання/відбиття від камер, що підтримують лише режим «RGB
»).
* **Навмисне виконання лише з метаданими** (усі продукти вимкнені, без `--indices`) все одно вважається успішним — порожній вихідний образ у цьому випадку є правильним результатом.
* Виконайте запуск знову з параметром `--verbose` і перевірте журнал бекенду на наявність рядків `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, які пояснюють пропуски для окремих камер.

Коди завершення: `0` — успіх · `1` — загальна помилка · `2` — помилка аргументу · `130` — перервано за допомогою Ctrl+C.

***

## Індекси рослинності

Запустіть `--indices` із одним або кількома іменами пресетів; кожен індекс зберігається у власній папці `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 попередньо задані імена, які приймає `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Існує три списки індексів — не плутайте їх.**У випадаючому меню «Налаштування проєкту» графічного інтерфейсу користувача є 27 формул (додано `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — ці п’ять доступні лише в графічному інтерфейсі та**не** застосовуються до `--indices`). Команда `lattice index --preset` для роботи в режимі реального часу/офлайн використовує власний окремий список із 22 пресетами. Формули та математичні обчислення для діапазонів описані в розділі [Формули мультиспектральних індексів](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Датчики освітленості DAQ: короткий огляд

Сімейство `daq pool-*` керує спектральними датчиками DAQ «MAPIR
» (DAQ-U через USB, DAQ-M через BLE, DAQ-E через Ethernet) через постійний пул бекенду — графічний інтерфейс користувача (GUI),CLI
таSDK
використовують один спільний дескриптор у режимі реального часу. **`pool-*` — це підтримуваний шлях DAQ у поставленому пакеті «CLI
»**; інші підкоманди `daq`, на які ви можете натрапити, є внутрішньою поверхнеюMAPIR
, призначеною лише для джерела, і завершуються з явною помилкою, що вказує на `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` без `--duration` працює до `pool-record --stop`; каталог виводу за замовчуванням — `~/Documents/DAQ Live View/` **на машині бекенду**. Профіль корекції конденсатора вибирається під час підключення (`--cap-id`, за замовчуванням на сервері — `sunshine_cosine`) і може бути замінений у режимі реального часу на `pool-set-cap` — профілі обмеження та калібрований діапазон датчика описані в розділах цього посібника, присвячених DAQ.

{% hint style="warning" %}
**DAQ-E на хості з декількома мережевими картами:** перше автоматичне виявлення `pool-connect --eth` після завантаження може завершитися невдачею навіть за умови справності датчика. `--eth-host <ip-or-hostname>` є надійнішим варіантом — використовуйте його, коли виявлення не дає результатів.
{% endhint %}

***

## Камери LATTICE, PTP та автоматизація проектів

Сімейство `lattice` (понад 45 підкоманд) охоплює повний цикл роботи з камерами LATTICE: виявлення, окремі знімки, постійні синхронізовані масиви з використанням потоку підключення Smart-Prep у графічному інтерфейсі, попередній перегляд у браузері в режимі реального часу, вирівнювання, обчислення індексів та діагностика мережевих карт хоста. Приклад:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Крім того: `chloros-cli time-sync` надає звіт про головний сервер PTP, який працює на хостіChloros
(камери LATTICE та датчики DAQ-E підпорядковуються йому для синхронізації часових міток між пристроями), а `chloros-cli project` відкриває збережений проект «Chloros
» та керує його камерами, масивами та датчиками у бездисплейному режимі — включаючи скриптові рецепти збору даних у форматі YAML.

Ці три сімейства (`lattice`, `project`, `daq pool-*`) також є єдиними, що підтримують `CHLOROS_BACKEND_URL` для керування **віддаленим** бекендом; основні команди завжди спрямовані на локальну машину.

Повні покрокові інструкції наведено в розділах, присвячених LATTICE, цього посібника; всі параметри наведено в [Довідці щодоCLI
](reference/cli-reference.md).

***

## Усунення несправностей: 5 найпоширеніших

| Симптом | Виправлення |
| --- | --- |
| `Login required` або заплановане завдання зависає на запрошенні `Email:` | Запустіть `chloros-cli login EMAIL 'PASSWORD'` один раз на цій машині — команди без кешованої сесії будуть виконуватися в інтерактивному режимі, а не завершуватимуться з помилкою. |
| `backend unreachable` | Запустіть настільну програмуChloros
або запустіть бінарний файл серверної частини безпосередньо (`chloros-backend`). Якщо ви вказуєте `lattice`/`project`/`daq pool-*` на віддалений сервер, перевірте `CHLOROS_BACKEND_URL`. |
| Блокування підключення масиву: `FRAMES WILL DROP` / `Reduce ROI to enable` | Скидання кільця прийому мережевої карти хоста до значень за замовчуванням — найпоширеніша причина відмови раніше працюючої установки від підключення, зазвичай після оновлення драйвера мережевої карти. Запустіть `chloros-cli lattice network --fix` з терміналу з **підвищеними** правами (або встановіть `ReceiveBufferLen=256`, `PendingReceives=64`); див. розділ *Налаштування та оптимізація мережевої карти хоста* у довіднику. |
| Підкоманда `daq` завершує роботу з повідомленням: «потрібен повний пакет daq…» | Це очікувано для готових збірок — у скомпільованому пакетіCLI
міститься лише сімейство команд `daq pool-*`, яке охоплює підключення, потокову передачу, запис та вибір капсул. Використовуйте `pool-*` (або `chloros_sdk.connect_daq_sensor()` зPython
). |
| Jetson виводить попередження про використання підкачки перед обробкою великих папок | Додайте підкачку на основі файлів — набір команд «CLI
» виводить точні команди `fallocate`/`swapon`, які потрібно виконати. |

***

## Отримання допомоги

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Усі прапори, усі підкоманди:** [CLI
Довідка](reference/cli-reference.md)
* **ЕквівалентPython
:** [Python
SDK
](api-python-sdk.md) та [SDK
Довідка](reference/sdk-reference.md)
* **Підтримка:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
