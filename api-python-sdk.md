# API : Python SDK

{% hint style="info" %}
**Шукаєте повний опис API?** Ця сторінка є практичним посібником. Усі загальнодоступні класи, методи, точні сигнатури та приклади, які можна скопіювати та вставити, містяться у [Довіднику SDK](reference/sdk-reference.md), який оптимізовано для роботи з штучним інтелектом.**Працюєте з AI-асистентом?** Вставте цей URL у чат, щоб він мав повну актуальну версію Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Кожна сторінка цього посібника доступна у вигляді необробленого тексту у форматі Markdown за посиланням у вигляді слога з малими літерами + `.md`, а весь посібник індексований за посиланням `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` на PyPI) керує всіма можливостями настільної програми, починаючи з Python: пакетна обробка зображень, управління камерами та масивами LATTICE у режимі реального часу, сесії збору даних (DAQ) за допомогою світлочутливих датчиків та автоматизація збережених проєктів. Це тонкий шар над тим самим локальним бекендом, який використовують графічний інтерфейс користувача та CLI (HTTP на `127.0.0.1:5000`), тому поведінка є однаковою на всіх трьох платформах.

## Встановлення

Встановлення складається з двох етапів: спочатку встановлюється пакет Chloros для настільних комп’ютерів (він забезпечує внутрішню інфраструктуру обробки та апаратні середовища виконання), а потім — пакет Python.

**Крок 1 — Встановіть Chloros.** Windows: запустіть інсталятор для настільних ПК (шлях за замовчуванням — `C:\Program Files\MAPIR\Chloros\`) зі сторінки [Завантажити](download.md). Linux: встановіть пакет `.deb` ([Встановлення Linux](linux/linux-installation.md)).**Крок 2 — Встановіть SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Можливо, вам навіть не знадобиться pip: кожен інсталятор постачається з відповідним колесом SDK. Інсталятор Windows автоматично встановлює його у вашу систему Python; інсталятор Linux `.deb` розміщує його в `/usr/lib/chloros/sdk/` і виводить точну команду `pip install --user`. PyPI оновлюється під час випуску нових версій, тому `pip install chloros-sdk` відповідає останній стабільній версії.

**Крок 3 — Увійдіть у систему один раз на кожній машині:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Повноваження кешуються в `~/.chloros/` (на обох платформах). На Windows ви можете увійти аналогічним чином через вкладку «<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">» у настільному додатку. Для SDK потрібен платний тарифний план Chloros+ — див. [Вимоги до ліцензії](#license-requirement) нижче.

| Вимога | Деталі |
| --- | --- |
| **Встановлено Chloros** | Windows: інсталятор для настільних ПК; Linux: пакет `.deb` (містить бінарний файл бекенду) |
| **Python** | 3.7 або вище (розроблено/тестовано на версії 3.10) |
| **Операційна система** | Windows 10/11 64-біт, Ubuntu 22.04 LTS або новіша версія, або NVIDIA Jetson (JetPack 6) |
| **Ліцензія** | Активний обліковий запис Chloros+, будь-який платний тариф (Copper або вище) |

## Перемога за 60 секунд

Один виклик створює проєкт, імпортує папку, налаштовує обробку та запускає конвеєр — автоматично запускаючи бекенд, якщо він ще не працює:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(У Linux використовуйте шляхи Linux: `/home/user/drone_images/flight001`. SDK працює однаково на обох платформах.)

Обробляєте папку знімків LATTICE? Використовуйте обгорнуту програму, сумісну з LATTICE — вона застосовує правильні налаштування за замовчуванням (без виявлення цільової панелі, стандартний дебейер):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — повне керування конвеєром

Для будь-чого, що виходить за межі однорядкового сценарію, використовуйте `ChlorosLocal`. Він запускає бекенд під час першого використання (`auto_start_backend=True`), створює та налаштовує проєкти, відстежує хід виконання та повертає підсумковий звіт після завершення.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
Зберігайте `http://127.0.0.1:5000` за замовчуванням, а не замінюйте його на `localhost` — у випадку Windows, `localhost` спочатку перетворюється на `::1` і займає близько 2 секунд на кожен запит до бекенду, що підтримує лише IPv4.
{% endhint %}

Використовуйте його як менеджер контексту для гарантованого очищення:

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` підтримує такі ключові слова: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` та `custom_settings`. Основні значення:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Регулятори, характерні для LATTICE (`input_level`, `radiometric_output`, сімейство `array_alignment*`) описані разом із повними таблицями значень у [Довіднику SDK](reference/sdk-reference.md#supported-values).

### Відстеження прогресу

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Зчитування підсумків після виконання — та виявлення порожніх запусків

Після завершення `process()` додає звіт про обробку бекенду у вигляді `result["summary"]`. Кожен запис у `summary["hints"]` — це повне речення, що пояснює будь-які важливі моменти — наприклад, чому запуск не дав жодного результату — і кожна підказка також повторно надсилається як Python `UserWarning`, тому порожні запуски діагностуються автоматично, навіть якщо ви ніколи не перевіряєте словник:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` не генерується, коли запуск не створює зображень.** Це єдине місце, де SDK та CLI навмисно відрізняються: `chloros-cli process` трактує ситуацію «було запрошено продукти, але жоден не був записаний» як збій і завершує роботу з ненульовим кодом, тоді як SDK повертається у звичайному режимі та повідомляє про цю ситуацію через `summary` / підказки. Якщо ваш конвеєр повинен зупинятися при порожньому запуску, перевіряйте це самостійно — перегляньте `summary` (або порахуйте файли у папці проєкту), замість того щоб покладатися на виняток.
{% endhint %}

## Smart Connect — апаратне забезпечення в режимі реального часу

Три допоміжні програми відкривають постійні сесії в пулі апаратних ресурсів бекенду — тому самому пулі, який використовує графічний інтерфейс, тому скрипти SDK співіснують із настільною програмою, не конкуруючи за послідовні порти чи пропускну здатність мережі. Усі три автоматично запускають локальний бекенд, якщо жоден із них не працює.

### Одиночна камера LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Синхронізований масив — `connect_array`

`connect_array` є рекомендованою точкою входу для багатокамерних установок. Він виконує той самий потік підготовки, що й графічний інтерфейс користувача: аналіз мережі, автоматичний вибір рівня синхронізації, синхронізацію часу за протоколом PTP, вибір формату пікселів для кожної камери, ініціалізацію AE та активацію тригера GPIO. **Перший послідовний порт є головним** (він генерує апаратний імпульс запуску); решта — підлеглі.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Додайте `smart=True` до будь-якого масиву зйомки, щоб дочекатися стабілізації автоматичної експозиції на всіх камерах перед запуском. Щодо режимів зйомки (Одиночний / Серійний / Інтервальний / Найшвидший), реєстраторів, серійної зйомки у формат відео та вирівнювання масиву див. [Довідник SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Датчик освітленості DAQ — `connect_daq_sensor`

Без аргументів `connect_daq_sensor()` автоматично визначає канал передачі (у порядку пріоритету: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Кожен кадр містить значення 135-точкового `spectrum` (Вт/м²/нм після калібрування), прапор `is_saturated` та CIE `x`, `y`, `z`. Щоб прив’язати конкретний датчик або транспорт — надійний вибір на хостах із кількома мережевими інтерфейсами, де автоматичне виявлення Ethernet може пропустити справний DAQ-E з першої спроби — передайте одну явну підказку:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Зверніть увагу, що профілі корекції верхнього регістру (`cap_id`) **не** є регулятором SDK — замість цього вибирайте їх за допомогою `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Збережені проекти — `open_project`

Збережений проект Chloros зберігає підключене обладнання (`cameras.json` + `sensors.json` разом із `project.json`), а `chloros_sdk.open_project(path)` може одночасно відновити з’єднання з усім обладнанням та здійснювати запис даних за іменами пристроїв. Див. розділ [Автоматизація проектів](reference/sdk-reference.md#project-automation--chlorosproject) у довіднику.

## Що отримує користувач при встановленні лише за допомогою pip

Перед використанням апаратних поверхонь перевірте прапорці доступності на рівні модулів:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

На хості, де встановлено **лише** `pip install chloros-sdk` і відсутній пакет робочого столу Chloros:

* `ChlorosLocal`, `process_folder` та `process_lattice_capture` **не** працюють — їм потрібний бінарний файл бекенду, що входить до складу інсталятора для настільних комп’ютерів.
* Допоміжні програми smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) є суто клієнтами HTTP, тому вони працюють із сервером на іншій машині — але вбудовані сервери прив’язані лише до петлі зворотного зв’язку, тому вам доведеться самостійно налаштувати переадресацію порту (наприклад, `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) та передати `backend_url="http://127.0.0.1:5000"` разом із `auto_start_backend=False`. Див. [Режим віддаленого бекенду](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Класи LATTICE для прямого підключення до апаратного забезпечення (`LatticeCamera`, `CameraPool`, …) імпортуються, але потребують середовища виконання Arena SDK із настільного пакета — без нього `CAMERA_AVAILABLE` є `False`.
* `daq_sdk` (класи прямого збору даних) постачається разом із настільною версією, а не з пакетом PyPI, тому `DAQ_AVAILABLE` на хості, де використовується лише pip, є `False` — замість цього керуйте датчиками DAQ через `connect_daq_sensor()`, підключившись до (тунельованого) бекенду.

## Вимоги до ліцензії

Для доступу до SDK необхідний активний обліковий запис Chloros+ на будь-якому платному тарифному плані — **Copper або вище**(Copper / Bronze / Silver / Gold); безкоштовний тарифний план «Iron» не надає доступу до SDK/CLI. Перевірка відбувається**на стороні сервера**: кожен запит SDK повинен містити як активну сесію, так і платний тарифний план, інакше сервер повертає `403` / `PLAN_UPGRADE_REQUIRED` (генерується як `ChlorosLicenseError` функцією `ChlorosLocal`, а як `ChlorosConnectError` — допоміжними функціями `connect_*`). Викликач, що вийшов із системи, отримує `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — повторне виконання `chloros-cli login` вирішує перший випадок, але не другий.

Офлайн-використання працює протягом пільгового періоду плану: рівень зчитується з кешу перевірки сервером (5 хвилин) або з кешу підписаних, прив’язаних до комп’ютера ліцензій (30 днів для місячних планів; до закінчення терміну дії передплати для річних). Коли пільговий період закінчується, тариф переходить у безкоштовний режим, і доступ за кодом SDK припиняється, доки пристрій хоча б раз не підключиться до сервера. Код `chloros-cli status` залишається доступним у безкоштовному тарифі, тому причина завжди залишається видимою. Див. [Chloros+ Вхід](chloros+-login.md).

## Винятки

Використовуйте базовий клас для обробки «будь-яких помилок Chloros»:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

Усі винятки конвеєра (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) походять від `ChlorosError`. Один нюанс: `ChlorosConnectError` — генерується лише `connect_camera` / `connect_array` / `connect_daq_sensor` — походить від звичайного `Exception`, **а не** від `ChlorosError`, тому `except ChlorosError` його не виявить. Повна ієрархія наведена в [Довідці щодо SDK](reference/sdk-reference.md#exceptions).

## Див. також

* [Довідка щодо SDK](reference/sdk-reference.md) — повна поверхня API, оптимізована для AI-асистентів.
* [Довідка по CLI](reference/cli-reference.md) — кожна підкоманда CLI відповідає виклику SDK.
* [Завантажити](download.md) — інсталятори для Windows та Linux.
