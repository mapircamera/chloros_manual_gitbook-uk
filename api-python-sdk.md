# API: Python SDK

{% hint style="info" %}
**Шукаєте повну документацію по API?** Ця сторінка є практичним посібником. Усі загальнодоступні класи, методи, точні сигнатури та приклади, які можна скопіювати та вставити, містяться в [Довіднику SDK](reference/sdk-reference.md), який оптимізовано для AI-асистентів.**Працюєте з AI-асистентом?** Вставте цей URL-адресу в чат, щоб отримати повний актуальний Chloros API 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Кожна сторінка цього посібника доступна у форматі сирого Markdown за посиланням у вигляді її назви з малими літерами + `.md`, а весь посібник індексовано за посиланням `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` на PyPI) забезпечує виконання всіх функцій настільної програми з Python: пакетну обробку зображень, управління камерами та масивами LATTICE у режимі реального часу, сесії збору даних (DAQ) за допомогою світлових датчиків та автоматизацію збережених проєктів. Це тонкий шар над тим самим локальним бекендом, який використовують графічний інтерфейс та CLI (HTTP на `127.0.0.1:5000`), тому поведінка є однаковою у всіх трьох середовищах.

## Встановлення

SDK постачається разом із настільним пакетом Chloros, який також містить серверну частину для обробки даних та апаратні середовища виконання, і завжди відповідає його версії.

**Крок 1 — Встановіть Chloros.** Windows: запустіть інсталятор для настільних ПК (шлях за замовчуванням — `C:\Program Files\Chloros\`) зі сторінки [Завантажити](download.md). Linux: встановіть пакет `.deb` за допомогою `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Встановлення для Linux](linux/linux-installation.md)).**Крок 2 — Перевірка SDK** (Python 3.7+). Зазвичай нічого встановлювати не потрібно:

* **Windows:** інсталятор встановлює вбудований пакет SDK wheel у ваш системний Python (якщо Python не встановлено, цей крок пропускається).
* **Linux:** `.deb` встановлює SDK для системного `python3` у `/usr/lib/python3/dist-packages`. Вимоги до нього, `python3-numpy` та `python3-requests`, рекомендуються пакетом: `apt install` їх встановлює, а `dpkg -i` — ні.
* **A virtualenv** не бачить системних пакетів; встановіть у нього колесо з комплекту: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` у Linux або `chloros_sdk-*.whl` у папці `sdk` інсталяційної папки Windows (за замовчуванням — `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

На комп’ютері, де немає пакета Chloros, чий SDK керує бекендом на іншому комп’ютері (див. [Що дає встановлення лише за допомогою pip](#what-a-pip-only-install-gets)), встановіть його з PyPI: `pip install chloros-sdk`. PyPI оновлюється під час випуску нових версій, тому відповідає останньому стабільному випуску.

**Крок 3 — Увійдіть у систему один раз на кожній машині:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Повноваження кешуються в `~/.chloros/` (на обох платформах). У Windows ви можете також увійти через вкладку «Користувач» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> у настільній програмі. Для роботи SDK потрібен платний тарифний план Chloros+ — див. [Вимоги до ліцензії](#license-requirement) нижче.

| Вимога | Деталі |
| --- | --- |
| **Встановлено Chloros** | Windows: інсталятор для настільних ПК; Linux: пакет `.deb` (містить бінарний файл бекенду та SDK) |
| **Python** | 3.7 або вище (розроблено/тестовано на версії 3.10) |
| **Операційна система** | Windows 10/11 (64-бітна версія), Ubuntu 22.04 LTS або новіша, або NVIDIA Jetson (JetPack 6) |
| **Ліцензія** | Активний обліковий запис Chloros+, будь-який платний тариф (Copper або вище) |

## Перемога за 60 секунд

Один виклик створює проєкт, імпортує папку, налаштовує обробку та запускає конвеєр — автоматично запускаючи бекенд, якщо він ще не працює:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(У Linux використовуйте шляхи Linux: `/home/user/drone_images/flight001`. SDK працює однаково на обох платформах.)

Обробляєте папку з записами LATTICE? Використовуйте обгорнуту програму, сумісну з LATTICE — вона застосовує правильні налаштування за замовчуванням (без виявлення цільової панелі, стандартний дебейер):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — повне керування конвеєром

Для будь-яких завдань, що виходять за межі однорядкового команди, використовуйте `ChlorosLocal`. Він використовує бекенд, який уже працює (бекенд настільної програми), або запускає встановлений бекенд під час першого використання (`auto_start_backend=True`; у Windows — із папки `backend` у папці інсталяції Chloros, за замовчуванням — `C:\Program Files\Chloros\backend\`), створює та налаштовує проекти, відстежує хід виконання та повертає підсумковий звіт після завершення.

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
Зберігайте значення за замовчуванням `http://127.0.0.1:5000`, а не замінюйте його на `localhost` — у Windows `localhost` спочатку перетворюється на `::1`, що займає близько 2 секунд на кожен запит до бекенду, який підтримує лише IPv4.
{% endhint %}

Використовуйте його як менеджер контексту. Вихід із блоку припиняє використання бекенду цим екземпляром, але не сам бекенд: він продовжує працювати для наступного скрипта і зупиняється самостійно після 30 хвилин бездіяльності. `shutdown_backend()` зупиняє екземпляр, запущений цим кодом.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` підтримує такі ключові слова: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` та `custom_settings`. Основні значення:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Параметри, специфічні для LATTICE (`input_level`, `radiometric_output`, сімейство `array_alignment*`), описані разом із повними таблицями значень у [Довіднику SDK](reference/sdk-reference.md#supported-values).

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

Після завершення `process()` додає звіт про обробку бекенду у вигляді `result["summary"]`. Кожен запис у `summary["hints"]` — це повне речення, що пояснює все, що заслуговує на увагу — наприклад, чому запуск не дав жодного результату — і кожна підказка також повторно надсилається у вигляді Python-коду `UserWarning`, тому порожні запуски діагностуються автоматично, навіть якщо ви ніколи не перевіряєте словник:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` не генерує винятку, коли запуск не створює зображень.** Це єдине місце, де SDK та CLI навмисно відрізняються: `chloros-cli process` трактує ситуацію «було запрошено продукти, але жоден не було записано» як збій і завершує роботу з ненульовим кодом, тоді як SDK повертається у звичайному режимі та повідомляє про цей стан через `summary` / підказки. Якщо ваш конвеєр повинен зупинятися у разі порожнього запуску, перевіряйте це самостійно — аналізуйте `summary` (або підрахуйте кількість файлів у папці проєкту), а не покладайтеся на виняток.
{% endhint %}

## Smart Connect — апаратне забезпечення в режимі реального часу

Три допоміжні програми відкривають постійні сесії в пулі апаратного забезпечення бекенду — тому самому пулі, який використовують CLI, інші скрипти та вкладки камер у графічному інтерфейсі, тому скрипти SDK співіснують з ними, не конкуруючи за послідовні порти чи пропускну здатність мережі. Усі три автоматично запускають локальний бекенд, якщо жоден із них не працює. Панель DAQ графічного інтерфейсу не входить до складу пулу: закрийте датчик освітленості в графічному інтерфейсі, перш ніж відкривати його зі скрипта.

### Окрема камера LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Відносні шляхи вказують на місце виконання вашого скрипта (змінено у версії 1.3.0).** З локальним бекендом відносний шлях `output_dir` (включно зі стандартним `"output"`) — та інші шляхи, які відкриває бекенд (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — вирішуються відносно робочого каталогу вашого скрипта, тому `cam.capture("output/")` записує дані в `<current folder>/output/`. У версії 1.2.0 встановлений бекенд розпізнавав їх у власній папці інсталяції, що призводило до помилки «Доступ заборонено». У випадку віддаленого бекенду шляхи надсилаються так, як їх введено.
{% endhint %}

### Синхронізований масив — `connect_array`

`connect_array` є рекомендованою точкою входу для багатокамерних установок. Він виконує той самий потік підготовки, що й графічний інтерфейс: аналіз мережі, автоматичний вибір рівня синхронізації, синхронізацію часу за PTP, вибір формату пікселів для кожної камери, ініціалізацію AE та активацію тригера GPIO. **Перший послідовний порт є головним** (він генерує імпульс апаратного тригера); решта — підлеглі.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Додайте `smart=True` до будь-якого масиву знімків, щоб дочекатися стабілізації автоматичної експозиції на всіх камерах перед запуском.

Нова функція у версії 1.3.0 — `ArraySession` — також надає вам синхронізовані кадри безпосередньо, без запису файлів:

* `arr.grab()` повертає по одному кадру з кожної камери, отриманих за один тригер, у вигляді масивів numpy (`{serial: array}`); `.group_id` — це спільний тригер, а `.errors` позначає будь-яку камеру, що залишилася поза увагою.
* `arr.frame_stream(fps=5.0)` видає один такий набір за кожен такт для обробки в реальному часі у вашому власному конвеєрі.
* `arr.continuous("output/")` запускає власний режим безперервного збору даних бекенду (режим «Безперервний» у графічному інтерфейсі) і повертає `ContinuousHandle` разом із `status()`, `wait()` та `stop()` — використовуйте його замість циклу збору даних, коли потрібно зберегти кожен тригер із високою частотою.

Щодо режимів збору даних (Одноразовий / Безперервний / Інтервальний / Найшвидший), реєстраторів, передачі серій даних у відеоформат та вирівнювання масивів див. [Довідник SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Датчик освітленості DAQ — `connect_daq_sensor`

Без аргументів `connect_daq_sensor()` автоматично визначає транспортний протокол (у порядку пріоритету: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Кожен кадр містить 135-точковий `spectrum` (Вт/м²/нм після калібрування), прапор `is_saturated` та CIE `x`, `y`, `z`. Щоб зафіксувати конкретний датчик або канал передачі, передайте одну явну підказку:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Зверніть увагу, що профілі корекції динамічного діапазону (`cap_id`) **не** є параметром SDK — вибирайте їх за допомогою `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Збережені проєкти — `open_project`

Збережений проєкт Chloros зберігає підключене обладнання (`cameras.json` + `sensors.json` разом із `project.json`), а `chloros_sdk.open_project(path)` може одночасно відновити з’єднання з камерами, масивами та датчиками освітлення та керувати зйомкою за іменами пристроїв. Датчики освітлення повторно підключаються через запущений серверний модуль Chloros із збереженими налаштуваннями та лімітами. Див. розділ [Автоматизація проєктів](reference/sdk-reference.md#project-automation--chlorosproject) у довіднику.

## Що отримує установка, що складається лише з pip

Перед використанням апаратних поверхонь перевірте прапорці доступності на рівні модулів:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

На хості, де встановлено **лише** `pip install chloros-sdk` і відсутній пакет робочого столу Chloros:

* `ChlorosLocal`, `process_folder` та `process_lattice_capture` **не** працюють — їм потрібен бінарний файл бекенду, що входить до складу інсталятора для настільних систем.
* Допоміжні програми smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) є чистими HTTP-клієнтами, тому вони працюють із сервером на іншому комп’ютері — але вбудовані бекенди прив’язані лише до петлі зворотного зв’язку, тому вам доведеться самостійно переадресувати порт (наприклад, `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) та передати `backend_url="http://127.0.0.1:5000"` разом із `auto_start_backend=False`. Див. [Режим віддаленого бекенду](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Класи LATTICE, що безпосередньо взаємодіють з апаратним забезпеченням (`LatticeCamera`, `CameraPool`, …) імпортуються, але потребують середовища виконання Arena SDK з настільного пакета — без нього `CAMERA_AVAILABLE` є `False`.
* Датчики DAQ керуються лише через (тунельований) бекенд — `connect_daq_sensor()` або `proj.sensors` проекту; бекенд відкриває датчик на своїй машині.

## Асистенти штучного інтелекту — `chloros-mcp`

SDK містить `chloros-mcp` — сервер MCP, який дозволяє Claude Desktop, Claude Code, Cursor та іншим клієнтам MCP керувати Chloros: обробляти зображення та керувати камерами LATTICE, масивами та світловими датчиками DAQ. Його інструменти охоплюють усі можливості цього SDK. Він встановлюється разом із SDK, тому для налаштування Claude Code достатньо одного рядка:

```bash
claude mcp add chloros -- chloros-mcp
```

Інші клієнти: [Використання Chloros з AI-асистентами](ai-assistants.md). Параметри та всі інструменти: [Сервер MCP (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Вимоги до ліцензії

Для доступу до SDK необхідний активний обліковий запис Chloros+ на будь-якому платному тарифі — **Copper або вище**(Copper / Bronze / Silver / Gold); безкоштовний тарифний план «Iron» не надає доступу до SDK/CLI. Перевірка відбувається**на стороні сервера**: кожен запит до SDK повинен містити дані про активну сесію та платний тарифний план, інакше сервер поверне коди помилок `403` / `PLAN_UPGRADE_REQUIRED` (генерується як `ChlorosLicenseError` функцією `ChlorosLocal`, а як `ChlorosConnectError` — допоміжними функціями `connect_*`). Викликач, що вийшов із системи, отримує `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — повторне виконання `chloros-cli login` вирішує перший випадок, але не другий.

Офлайн-використання працює протягом пільгового періоду тарифного плану: рівень доступу зчитується з кешу перевірки сервером (5 хвилин) або з кешу підписаних ліцензій, прив’язаних до конкретного пристрою (30 днів для місячних тарифних планів; до закінчення терміну дії підписки для річних). Коли пільговий період закінчується, тариф переходить у безкоштовний режим, а доступ до SDK припиняється, доки комп’ютер хоча б раз не підключиться до сервера. Код `chloros-cli status` залишається доступним у безкоштовному тарифному плані, тому причина завжди залишається видимою. Див. [Вхід у Chloros+](chloros+-login.md).

## Винятки

Обробляйте базовий клас для обробки «будь-яких помилок Chloros»:

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

Усі винятки конвеєра (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) походять від `ChlorosError`. Один нюанс: `ChlorosConnectError` — генерується лише `connect_camera` / `connect_array` / `connect_daq_sensor` — походить від звичайного `Exception`, **а не** від `ChlorosError`, тому `except ChlorosError` його не виявить. Повна ієрархія наведена в [Довіднику SDK](reference/sdk-reference.md#exceptions).

## Див. також

* [Довідник SDK](reference/sdk-reference.md) — повний набір API, оптимізований для AI-асистентів.
* [Довідка щодо CLI](reference/cli-reference.md) — кожна підкоманда CLI відповідає виклику SDK.
* [Використання Chloros із AI-асистентами](ai-assistants.md) — рецепти налаштування та підказки для `chloros-mcp`.
* [Завантажити](download.md) — інсталятори для Windows та Linux.
