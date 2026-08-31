# Chloros Python SDK Довідка

**Версія:**

1.2.0**Створено:**

29.07.2026 19:19 ·**Переглянуто:**

30.08.2026**Пакет:** `chloros-sdk` (PyPI)**Аудиторія:** Оптимізовано для використання LLM; зрозуміле для людини.**Сфера застосування:** Усі загальнодоступні класи, функції та допоміжні функції, що надаються `import chloros_sdk`, із прикладами, які можна скопіювати та вставити, що охоплюють обробку зображень, керування однією камерою, синхронізовані масиви, датчики DAQ та автоматизацію проектів.

Якщо вам потрібні лише основні моменти, перейдіть до:
- [Встановлення та швидкий старт](#installation)
- [Smart-Connect для масивів LATTICE](#smart-connect-for-lattice-cameras)
- [Сесії датчиків DAQ](#daq-sensor-sessions)
- [Автоматизація проєкту](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Архітектура за 60 секунд

SDK — це тонкий шар Python, накладений на бекенд Chloros (той самий сервер Flask, що використовується в настільному графічному інтерфейсі та CLI). Для автоматизації потрібно імпортувати `chloros_sdk` і викликати методи високого рівня; «під капотом» кожен виклик перетворюється на запит HTTP до локального бекенду на порту 5000 — `http://127.0.0.1:5000/api/...` (навмисно не `localhost`, який спочатку перетворюється на `::1` на Windows і займає ~2 с на запит до бекенду, що підтримує лише IPv4). Бекенд володіє пулом апаратного забезпечення — камерами, датчиками DAQ, профілями вирівнювання, буферами кадрів — тому скрипти SDK можуть співіснувати з графічним інтерфейсом, не конкуруючи за послідовні порти чи пропускну здатність мережевих карт.

Ви будете використовувати три інтерфейси:

1. **`ChlorosLocal` + вільні функції** (`process_folder`, `process_lattice_capture`) — конвеєр обробки зображень. Обробіть всю папку, виконавши калібрування, дебейерінг та експорт індексів одним викликом Python.
2. **Обробники Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — відкриття постійного сеансу зв’язку з бекендом для апаратного забезпечення, що працює в режимі реального часу. Такий самий потік «smart-prep», як і в графічному інтерфейсі: перевірка мережі, автоматичний вибір рівня, PTP, ініціалізація AE, налаштування тригера GPIO.
3. **`ChlorosProject` / `open_project`** — завантаження збереженого проєкту (папка з `cameras.json` + `sensors.json` + `project.json`), одночасне підключення всіх пристроїв та запис даних за допомогою іменованих дескрипторів.

Поверхні 1 і 2 **автоматично запускають локальний бекенд**, якщо він ще не перебуває у режимі очікування (той самий вбудований бінарний файл, який запускає GUI/CLI) — тому простий скрипт працює з нової оболонки без попереднього запуску бекенду. Передайте `auto_start_backend=False`, щоб відключити цю функцію (наприклад, при вказівці на віддалений бекенд, який ніколи не запускається). Див. [Автозапуск бекенду](#backend-auto-start). Surface 3 поводиться інакше: `open_project()` не приймає параметр `auto_start_backend`, а `connect_all()` ніколи не запускає бекенд — він один раз перевіряє `http://127.0.0.1:5000` один раз, і, якщо ніхто не відповідає, без попередження переходить до прямого (без бекенда) `lattice_sdk`. Лише `proj.process()` та `stream(..., overlays=True)` ліниво створюють `ChlorosLocal()` (який виконує автоматичний запуск).

Усі три вимагають авторизації: запустіть `chloros-cli login` один раз на машині або увійдіть через графічний інтерфейс робочого столу. Виклики SDK без дійсної сесії викликають помилку `ChlorosAuthenticationError`.

Вимоги:
- Python 3.7+ (згідно з інформацією в пакеті; розроблено/тестовано на версії 3.10)
- Локально встановлений Chloros Desktop (бінарний файл бекенду входить до складу інсталятора)
- Активний Chloros+ вхід. Мінімальний рівень SDK/CLI повинен бути рівня **Copper**або вище (Copper / Bronze / Silver / Gold); безкоштовний**Iron**не надає доступу до SDK/CLI. Це правило застосовується**на стороні сервера**: кожен запит із позначкою SDK/CLI повинен містити як активну сесію, так і платний тарифний план, інакше сервер повертає код `403` із `error_code: PLAN_UPGRADE_REQUIRED` (відображається як `ChlorosLicenseError` за допомогою `ChlorosLocal`, а як `ChlorosConnectError` — за допомогою `connect_*`). Викликач, що вийшов із системи, отримує `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — ці два коди відрізняються, оскільки повторне виконання `chloros-cli login` виправляє перший, але не може виправити другий.
- Офлайн-використання підтримується протягом пільгового періоду тарифного плану: рівень доступу зчитується з кешу перевірки сервером (5 хв) або з кешу підписаних ліцензій, прив’язаних до конкретного комп’ютера (30 днів для місячних тарифних планів, до закінчення терміну дії передплати для річних). Коли цей пільговий період закінчується, тарифний план переходить у безкоштовний режим, і доступ за кодами SDK/CLI доступ припиняється, доки комп’ютер не зможе хоча б раз підключитися до сервера. `chloros-cli status` (`GET /api/license-status`) залишається доступним на безкоштовному рівні, тому причина очевидна — це єдиний маршрут SDK/CLI маршрут, який не підпадає під обмеження тарифного плану.
- Windows 10/11 64-бітна версія, **Ubuntu 22.04 LTS або новіша**, або Jetson (JetPack 6). Ubuntu 20.04**не** підтримується: залежності `.deb` походять від того, з чим пов’язаний бекенд, включаючи `libc6 (>= 2.34)`, а в дистрибутиві Focal використовується glibc 2.31.

---

## Встановлення

Python SDK — це тонкий шар Python, побудований на базі бекенду Chloros. Для будь-чого, що виходить за межі кількох робочих процесів, пов’язаних виключно зі збором даних (DAQ), вам потрібно **локально встановити настільний пакет Chloros** (інсталятор Windows або Linux `.deb`) — саме він забезпечує бінарний файл бекенду, середовище виконання Arena SDK для камер LATTICE та набори для калібрування.

Останні завантаження: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Крок 1 — Встановіть пакет платформи Chloros

#### Windows (.exe)

1. Завантажте `Chloros-Setup-x.y.z.exe` зі сторінки завантажень.
2. Запустіть інсталятор і дотримуйтесь вказівок майстра. Шлях інсталяції за замовчуванням — `C:\Program Files\MAPIR\Chloros\`.
3. Запустіть Chloros хоча б один раз і увійдіть у систему за допомогою свого облікового запису Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Крок 2 — Встановіть Python SDK

**Інсталятор Chloros постачається з відповідним пакетом SDK.** Кожен інсталятор Windows та файл .deb Linux розміщує на диску `chloros_sdk-X.Y.Z-py3-none-any.whl`, який точно відповідає версії графічного інтерфейсу / CLI / версії бекенду. Вам не доведеться стежити за оновленнями на PyPI, щоб залишатися в синхронізації.

#### Windows

Інсталятор автоматичнозапускає `pip install` з використанням вбудованого файлу wheel та вашого системного Python (бажано використовувати запускач `py.exe`, у разі відсутності — `python -m pip`). Ніяких дій не потрібно — `import chloros_sdk` працюватиме у вашому середовищі Python після успішної інсталяції. Якщо на комп’ютері немає Python, інсталятор без попередження пропускає цей крок, і графічний інтерфейс та CLI продовжують працювати.

#### Linux (.deb)

Пакет .deb розміщує файл wheel у `/usr/lib/chloros/sdk/`. `postinst` виводить точну команду — дистрибутиви, що дотримуються PEP 668, за замовчуванням відхиляють глобальні записи pip, тому ми не виконуємо автоматичну інсталяцію:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Для розгортань Jetson у режимі «air-gap» цей процес відбувається повністю в автономному режимі — колесо вже знаходиться на диску.

#### Публічний PyPI

Для хостів, що використовують лише pip (без встановленого настільного пакета Chloros; робочі процеси з віддаленим бекендом або виключно для DAQ):

```bash
pip install chloros-sdk
```

PyPI оновлюється на основі збірок інсталятора з версією релізу, тому опублікований файл wheel відповідає останньому стабільному релізу. Розробницькі збірки (наприклад, `1.1.4.dev1`) поширюються лише через вбудований інсталятор у вигляді файлу wheel.

#### Перевірка

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Потрібна підписка на Chloros+.** Усі виклики SDK вимагають активного логіна Chloros+. Запустіть `chloros-cli login user@example.com 'YourPassword'` один раз на кожній машині; облікові дані зберігаються у кеші `~/.chloros/`.

### Чи потрібен мені пакет для настільних комп’ютерів?

Самого лише пакета pip **не** достатньо для більшості робочих процесів. Ось що потрібно для кожної поверхні SDK:

| Поверхня SDK | Чи потрібен пакет для настільних ПК? | Чому |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Так** | Автоматично запускає бінарний файл бекенду на `/usr/lib/chloros/chloros-backend` (Linux) або `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Так**(локально)**/ Ні**(віддалено) | Потрібні клієнти Pure HTTP через бекенд. Локальний бекенд → потрібен пакет для настільних ПК. Віддалений бекенд → `backend_url=`**через тунель** (див. Режим віддаленого бекенду — вбудовані бекенди прив’язуються лише до петлі зворотного зв’язку). |
| `ChlorosProject` / `open_project` | **Так** | Передає збережені проєкти через бекенд. |
| Прямі класи LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Так** | Потрібна нативна середовище виконання Arena SDK, що входить до складу пакету для настільних ПК. В іншому випадку `CAMERA_AVAILABLE` є `False` під час імпорту. |
| Класи прямого збору даних (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Ні** | Чистий Python через pyserial/bleak/zeroconf. Середовище, що використовує лише pip, може керувати DAQ від початку до кінця. |

### Режим віддаленого бекенду (хост, що використовує лише pip, через тунель)

> **Поставлений бекенд недоступний через локальну мережу.** Продуктивні
> збірки прив’язуються лише до петлі зворотного зв’язку (обидві родини петлі зворотного зв’язку) і категорично відхиляють
> єдиний режим, що не є петлею зворотного зв’язку (`CHLOROS_CLOUD_MODE`), тому
> `backend_url="http://<lan-ip>:5000"` **не може працювати з встановленим
> Chloros** — цей шаблон завжди працював лише з бекендом source/dev
> . Щоб запустити бекенд на іншій машині, самостійно переадресуйте його
> порт loopback і вкажіть SDK на тунель:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Бездисплейні хости, хости CI та роботизовані хости можуть використовувати одну машину з повною інсталяцією робочого столу як «Chloros сервер», а на всіх інших — `pip install chloros-sdk`; однак транспорт між ними — це налаштований користувачем тунель, описаний вище, а не пряме LAN-з’єднання URL.

> **Відоме обмеження — `ChlorosLocal` не підтримує роботу виключно через pip.** `ChlorosLocal(backend_url=BACKEND)` наразі визначає локальний бінарний файл бекенду у своєму конструкторі *до* перевірки URL і видає помилку `ChlorosBackendError` («Chloros бекенд не знайдено…») у разі відсутності встановленого пакету для робочого столу — навіть за наявності доступного віддаленого бекенду. Лише поверхня Smart Connect, наведена вище (`connect_camera` / `connect_array` / `connect_daq_sensor`, а також `analyze_array_network` та `list_*` / `discover_*`) працюють на хості, що підтримує лише pip.

### Робочий процес «Тільки DAQ» (хост, що підтримує лише pip)

Якщо вам потрібні лише датчики DAQ і ви не використовуєте камери LATTICE чи обробку зображень, пакет pip є самодостатнім:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Не потрібно ні бекенду, ні .deb, ні Chloros+ для роботи з DAQ безпосередньо на апаратному рівні.

---

## Швидкий старт

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Головний індекс API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Обробка зображень — `ChlorosLocal`

Головний клас конвеєра. Запускає бекенд при першому використанні, створює та налаштовує проекти, відстежує хід виконання, повертає підсумки після завершення.

### Конструктор

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

### Методи

| Метод | Опис |
| --- | --- |
| `create_project(project_name, camera=None)` | Створити новий проєкт (за бажанням із шаблоном камери, таким як `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Імпортує зображення у форматах RAW/TIF/JPG/DNG **та записи датчика освітленості `.daq`**. Повертає `count` (зображення) та `scan_count` (записи). Видає попередження лише у випадку, якщо у папці немає ні того, ні іншого. |
| `export_light_sensor(daq=True, csv=True)` | Записувати відкалібровані `.daq` + `.csv` для кожного запису датчика освітленості в проєкті у файл `<project>/Light Sensor/`X. Див. [Записи датчика освітленості](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Налаштуйте параметри обробки. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Запустіть конвеєр. Повертає `{"status": "complete", "async": False}`, а також ключ `summary`, якщо бекенд його надає — див. [Підсумок та підказки після виконання](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Перевірити стан бекенду. |
| `logout()` | Очистити кешовані облікові дані. |
| `shutdown_backend()` | Припинити роботу бекенду (якщо його було запущено за допомогою SDK). |
| `discover_cameras()` | Виявити камери LATTICE **через бекенд цього екземпляра** (`/api/camera/discover`). Повертає список словників (`serial`, `model`, `ip`, …) — такої самої структури, як і в GUI/CLI. Порожній список, якщо нічого не знайдено або бекенд недоступний. |
| `camera_capture(output_dir, format="tiff", **settings)` | Захоплення одного кадру**через бекенд**(автоматично запускається цим дескриптором), щоб він отримав ту саму підготовку, що й GUI/CLI (за замовчуванням 12 біт, повторне використання пулу, вбудовані метадані калібрування). Визначте ціль за допомогою `serial=` або `device_index=`; передайте `exposure`/`gain`/`pixel_format`/`preset` як `**settings`. Повертає словник метаданих попередньої версії (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Видавати кадри попереднього перегляду, складені методом накладання, з об’єднаної камери — легкий MJPEG-клієнт через маршрут `/api/camera/<serial>/stream-annotated` бекенду (зебра / сітка / перехрестя / гістограма / пікінг / точка, намальована на стороні сервера). `decode=True` видає масиви BGR; `False` видає необроблені JPEG байти. Також доступно на рівні проекту як `ChlorosProject.stream(overlays=True)`. |

Використовуйте як менеджер контексту для гарантованого очищення:

```python
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

### Записи датчика освітленості — відкалібровані `.daq` + `.csv`

Дані з DAQ-U / DAQ-M / DAQ-E можна записувати **без** їхнього калібрувального пакета. Саме
це роблять загальнодоступні [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
записувачі (`record_daq.py`) роблять за замовчуванням: вони записують необроблені дані датчиків і проставляють мітку на
файл, щоб Chloros отримав заводську калібровку цього датчика **за серійним номером** — спочатку з локального кешу,
а потім із хмари MAPIR — і застосовує її під час імпорту.

Chloros записує результат назад у вигляді двох продуктів на кожен запис, під
`<project>/Light Sensor/`:

| Продукт | Що це |
| --- | --- |
| `<name>_calibrated.daq` | Архів, придатний для повторної обробки — та сама схема, що й у запису в режимі реального часу, але тепер із зазначенням пакета, який його створив. Повторне імпортування **не** призводить до повторної калібрування. |
| `<name>_calibrated.csv` | Спектральна інтенсивність випромінювання в Вт/м²/нм за власною сіткою довжин хвиль датчика, один рядок на одне зчитування, а також фотометричні стовпці (загальна потужність, фотопічний/скотопічний люкс, PPFD та його розподіл на синій/зелений/червоний, пікова довжина хвилі). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Тільки датчики без набору (DAQ-A).** Сирі спектральні імпульси датчика — *не* інтенсивність випромінювання. Див. нижче. |

`process()` виконує цей експорт як один із етапів своєї роботи. Для цього **не** потрібні зображення:
самостійний політ світлового датчика є повноцінним робочим процесом, і такий проєкт за своєю суттю не містить
жодного зображення.

**Записи DAQ-A експортуються у вигляді необроблених значень.** Сімейство DAQ-A з’явилося раніше, ніж система
і не має пакета для завантаження — натомість він калібрується в польових умовах за допомогою
мішені відбиття, тому він ніколи не потребував такого пакета. Ці записи експортуються
під префіксом `_raw`, а не `_calibrated`: інша назва файлуімен, а не прапорця
всередині файлу, оскільки інформація має зберегтися при надсиланні електронною поштою у вигляді простого імені. У
заголовку `.csv` вказано `raw spectral sensor counts (NOT irradiance)` і міститься попередження, що
значення є порівнянними **в межах** меж файлу — саме для цього їх і використовує калібрування на основі цільових значень
— а не між датчиками. Фотометричні стовпці, що залежать від потужності (загальна потужність,
фотопічний/скотопічний люкс, PPFD), повертаються як **NULL**, а не інтегровані на основі підрахунків.

DAQ-U / DAQ-M / DAQ-E, пакет для якого просто не вдалося завантажити, все одно **пропускається**,
а не записується у сирому вигляді: у цьому випадку пакет існує, і «повторне підключення та переобробка» є дійсно слушною порадою.

Старі записи **v1.01 / v1.02** (такі записує DAQ-A-SD) не містять епохи для кожного зчитування,
а лише час запису файлу. Програма зіставлення зображення↔низхідного потоку все ще відхиляє їх — зіставлення
кадр із часом запису призвело б до непомітної помилки — але експортер їх зчитує, і
CSV виводить `clock=daq_created_on`, тож продукт вказує, за яким годинником він працює.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Запис, пакет калібрування якого неможливо отримати (офлайн або датчик без
калібрування у файлі), відображається у `skipped` **із зазначенням причини**. Вона ніколи
не записується як «відкалібрований» файл, що містить необроблені дані підрахунку — підключіться до Інтернету та
запустіть процедуру повторно, і експорт завершиться.

### Зворотні виклики про хід виконання

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Підсумок та поради після виконання

Після завершення `process()` завантажує `GET /api/processing-summary` і додає тіло як `result["summary"]`. Завантаження відбувається за принципом «найкращих зусиль» і ніколи не блокує успішне повернення — якщо підсумок недоступний, `process()` повертається до простої форми `{"status": "complete", "async": False}`. Кожен запис у `summary["hints"]` — повні речення з пропозиціями щодо усунення проблеми, наприклад, чому запуск дав нульовий результат — також повторно виводиться як Python `UserWarning`, тож запуски з нульовим результатом є самодіагностичними, навіть якщо ви ніколи не перевіряєте словник:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` — це частина, придатна для машинного зчитування:

| Ключ | Що підраховується |
| --- | --- |
| `models` | Групи камер у запуску. |
| `images_in_groups` | Джерельні зображення у цих групах. |
| `targets_found` | Виявлені мішені відбиття. |
| `images_calibrated` | Зображення, за допомогою яких було проведено калібрування циклу. |
| `exported_files` | **Файли зображень, створені в ході циклу.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Записи світлових датчиків, які навмисно підраховано окремо — вони походять з іншого етапу та існують для циклів, у яких зображень взагалі немає, тому їхнє об’єднання могло б створити враження, ніби цикл, що передбачав лише збір даних (DAQ), експортував зображення. |

Поряд із ними: `summary["output_dirs"]` (усі каталоги, до яких було записано дані),
`summary["light_sensor_export"]`, `summary["stopped"]` (має значення «true», коли користувач перервав
запуск, тому часткові підрахунки не трактуються як завершений запуск із недостатнім виходом), та
`summary["groups"]` (розбивка за групами).

`exported_files` записується конвеєром **під час запису**, а не сканується з
об’єктів зображення проєкту згодом. Паралельні та GPU-стратегії створюють власні об’єкти зображення
(у робочих підпроцесах для GPU-шляхів), тому старе сканування повідомляло
`0 file(s) written` для кожного такого запуску, а потім видавав підказку про нульові експорти — у запусках,
де все працювало нормально. Якщо ви створюєте скрипт на основі цього номера, то зараз успішний паралельний запуск
повідомляє про ненульову кількість.

Пропуски датчика Light-sensor повідомляють причину, яку зчитувач фактично встановив для кожного файл —
нечитабельна схема, відсутній пакет, помилка запису — **дублюються**, тож двадцять файлів,
пропущених з однієї причини, трактуються як одна причина, а не як двадцять її повторень.

> **`process()` не виникає, коли запуск не створює жодних зображень.** Це єдине місце, де SDK та
> CLI навмисно відрізняються: `chloros-cli process` трактує «запитувалися результати, але жоден не був
> записаний» як помилку і завершується з ненульовим кодом, тоді як SDK повертається у звичайному режимі та повідомляє про
> цю ситуацію через `summary` / підказки. Якщо ваш конвеєр повинен зупинятися при порожньому запуску, перевірте це
> самостійно — перегляньте `summary` (або порахуйте файли у папці проєкту), замість того щоб покладатися на
> на відсутність винятку. Типовими причинами є те, що вхідна папка не була розпізнана як
> джерело зйомки, а також пропущені продукти, які не застосовуються для наявних камер (наприклад, яскравість лише від камер RGB
>).

### Функції для зручності

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Підтримувані значення

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Радіометричний вихід (мультиспектральний конвеєр LATTICE)

Рівень експорту мультиспектральних даних (M3C/M3M) конвеєра `process` у форматі LATTICE — `reflectance` (за замовчуванням), `radiance`, `sensor-response` або `all` (кожен відповідний режим для кожного зображення) — відповідає налаштуванню обробки проекту **«Радіометричний вихід»**. Для `configure()` передбачено спеціальне ключове слово:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Розширений обхідний шлях — запис ключа `"Radiometric output"` проекту через `custom_settings` — все ще працює, але пам’ятайте, що він замінює весь блок налаштувань (див. попередження нижче):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (за замовчуванням) ділить яскравість камери на **потік даних DAQ, що відповідає часовій мітці**, який автоматично визначається з записаного `.daq` (DAQ-U/M/E)**або власного `.csv` для DAQ-M**, знайденого разом із зображеннями; будь-який пакет калібрування для конкретної камери або DAQ, якого немає локально,**автоматично завантажується з AWS** під час першого використання. CLI надає доступ до цього у вигляді перемикачів для продуктів за типом у `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **замінює** весь обчислений блок налаштувань (за замовчуванням він ігнорує інші ключові слова та перевірку `configure()`). При його використанні включіть усі ключі `Project Settings`, які вас цікавлять, як у наведеному вище прикладі.

---

## Smart-Connect для камер LATTICE

Постійні сесії бекенду для апаратного забезпечення в режимі реального часу. Використовуються ті самі кінцеві точки, що й у графічному інтерфейсі, тому поведінка є однаковою у SDK, CLI та графічному інтерфейсі.

### Окрема камера — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Підпис `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Методи

| Метод | Опис |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Зчитування вузлів GenICam; повертає `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Запис вузлів за дружнім ім’ям (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Запис **одного** кадру. Повертає список із одного елемента, що містить словники метаданих кадру. (Функція серійного запису/запису декількох кадрів була видалена — якщо вам потрібна серія кадрів, викликайте `capture()` у циклі.) |
| `disconnect()` | Звільнення з пулу. Не виконує жодних дій, якщо ми підключилися до вже відкритої сесії. |

Елементи керування експортом `capture()` (така сама модель, як у масиві + графічний інтерфейс):

- `processing` / `levels` — `processing="all"` зберігає всі відповідні типи експорту; `levels=["raw","radiance"]` зберігає лише ці (перекриває `processing`). Пропустіть обидва для використання значень за замовчуванням бекенду.
- `force_daq=True` — зберігає призначене значення з DAQ/DLS як додатковий файл `.daq` навіть під час збору лише необроблених даних, щоб кадр можна було пізніше переобробити у відбиття/індекс. Не виконує жодних дій, якщо DAQ не підключено.

### Синхронізований масив — `ArraySession` (Smart-Prep)

`connect_array` є **рекомендованою точкою входу** для конфігурацій із кількома камерами. Ця функція виконує повний цикл обробки Smart-Prep через графічний інтерфейс користувача:

1. **Аналіз мережі** (`/api/camera/array/recommend`) — визначає найбільший розмір кадру, який відповідає рівню sim-emit без втрати кадрів.
2. **Автоматичний вибір рівня** — `sim-capture-sim-emit`, якщо кабель це витримує; інакше — `sim-capture-ftd-stagger` або `slip-emit-and-capture`.
3. **Автоматичне зменшення**— без попередження зменшує розмір кадру / збільшує об’єднання, коли канал не може підтримувати запитувану роздільну здатність.**Ця запобіжна міра не поширюється на сукупне перевищення пропускної здатності**: надмірну кількість камер для каналу неможливо виправити шляхом зменшення розміру кадрів — див. [Перевищення пропускної здатності](#over-subscription-the-per-cam-floor).
4. **PTP увімкнено** за замовчуванням — міжопераційні мітки часу мають точність до мікросекунд.
5. **Автоматичний вибір формату пікселів для кожної камери** — камери RGB → `BayerRG8`, мультиспектральні → `BayerRG12`.
6. **Ініціалізація AE** — знімки поточного стану AE кожної камерипоточний стан AE кожної камери, щоб підключення не скидало експозицію під час роботи.
7. **Конфігурація тригера GPIO** — `connect_array` активує кожну камеру (`TriggerMode=On`, `TriggerSource=Line2`), тому імпульс головного пристрою керував підлеглими через кабель M8. Цей крок стосується лише масиву: окрема камера, запущена за допомогою `LatticeCamera`, замість цього працює у вільному режимі.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` Підпис

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Значення `force_tier`:
- `"sim-capture-sim-emit"` — справжня синхронність (усі камери спрацьовують на одному фронті тактового сигналу).
- `"sim-capture-ftd-stagger"` — гнучке часове зсунення (камери спрацьовують із невеликим часовим зсувом, тому пакети передаються по каналу послідовно).
- `"slip-emit-and-capture"` — послідовний запис для кожної камери (без часової синхронізації; єдиний варіант, коли жоден розмір кадру не відповідає симуляції).

`wire_ceiling_mbps` замінює **стабільний пропускний бюджет хоста** у МБ/с — єдине
число, від якого залежить розподіл ресурсів усього масиву. Залиште значення `None`, щоб використовувати автоматично визначене
значення. Зменшуйте його, коли масив повідомляє про кадри з пошкодженням GVSP: автоматичне значення виводиться
зоголошеної швидкості з’єднання мережевого адаптера, що завищує показники USB-адаптерів, вузьких смуг PCIe та
завантажених спільних мереж — і це завищення проявляється у вигляді пошкоджених кадрів, а не у вигляді
видимо повільного з’єднання. Це значення зберігається у блоці захоплення масиву проекту, тому
при повторному відкритті або пізнішому запуску `connect_array` воно відновлюється, як і будь-яке інше налаштування масиву.
Див. [Стан масиву](#array-health--which-subsystem-is-losing-frames).

#### Надмірна підписка (мінімальний поріг на камеру)

Механізм регулювання Sim-emit розподіляє між камерами частку бюджету каналу, безпечного щодо колізій, мінімальне значення якого становить **8 МБ/с на камеру**(`per_cam_floor_bps`). Як тільки `N × floor` перевищує верхню межу, безпечну від колізій, масив**перевищує пропускну здатність каналу**— режимом відмови є втрата пакетів GVSP, а не зниження частоти кадрів — і жодних засобів виправлення, пов’язаних із розміром кадру, не існує:**об’єднання та ROI зменшують кількість байтів на кадр, а не кількість байтів за секунду за алгоритмом розподілу**, яку порівнює агрегована перевірка. Практичні верхні межі для повної роздільної здатності на хості 1 Гб/с:**6 камер при 1500 MTU, 9 з джамбо-кадрами** (`max_cams_collision_safe` у відповіді на аналіз вказує граничне значення для вашого каналу). Способи виправлення: зменшення кількості камер, використання джамбо-кадрів на всьому шляху або використання швидшої мережевої карти.

- Відповіді `analyze_array_network()` та `/api/camera/array/connect` містять `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` та `per_cam_floor_bps`. Коли `oversubscribed` має значення «true», проекція **обнуляє поля fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`), замість того щоб повідомляти оманливу швидкість — повільну, але таку, що працює.
- `POST /api/camera/array/connect` приймає параметр тіла `pin_resolution` (**лише для HTTP — не для SDK як аргументу-ключа**; `connect_array` його не надає). Фіксація усуває захисну мережу, пов’язану з покроковим зменшенням розміру блоків, тому підключення з перевищенням квоти та встановленим `pin_resolution`**категорично відхиляється** з помилкою, що вказує на всі можливі способи виправлення. Без фіксації підключення продовжується з поступовим зменшенням, але з попередженням, що зменшення не зможе очистити сукупність.
- Лабораторна-лазівка: встановіть `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` у середовищі бекенду, щоб знизити рівень відмови до гучного попередження — ви все одно підключитеся і приймете втрату пакетів.

#### Стан масиву — яка підсистема втрачає кадри

`GET /api/camera/array/<array_id>/capability` несе активний блок `health` у
підключеному масиві, що переоцінюється у **10-секундному** ковзному вікні. Він розділяє втрату кадрів
на дві причини, що потребують протилежних виправлень, замість одного показника «неповноти», який
не вказує на жодну з них:

| Поле | Що це означає | Яка підсистема |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (за кожним послідовним портом) | Кадр **прибув, але мав структурні помилки**— втрата пакетів GVSP. |**Мережа**: пропускна здатність лінії, темп передачі, кільце прийому мережевої карти, MTU |
| `never_arrived_rate_pct` (за серійним номером) | Кадр **взагалі не надійшов**— камера не спрацювала або з неї нічого не вийшло. |**Тригер / синхронізація**: кабель M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Найгірша швидкість передачі даних для кожної камери. | — |
| `per_cam_rate_pct` | Сукупний показник неповноти за камерою (обидві причини разом). | — |
| `stable_for_seconds` | Як довго кожна камера залишалася на рівні нижче 0,01 %. | — |

Поряд із `health` у цьому ж записі вказано значення, на якому зависає весь обсяг виділених ресурсів:

| Поле | Що це означає |
| --- | --- |
| `wire_ceiling_mbps` | Поточний стабільний пропускний бюджет хоста, МБ/с. |
| `wire_ceiling_source` | Звідки взято це число, словами — наприклад, `USB-capped 200 MB/s (was theoretical 1062; …)` або `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, коли `wire_ceiling_mbps=` встановив це значення. |
| `nic_is_usb` | `true` для USB-адаптера Ethernet. |

Для цієї кінцевої точки немає обгортки SDK — читайте його безпосередньо:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Інтерпретація:** значення `gvsp_corrupt_rate_pct`, відмінне від нуля, при значенні `never_arrived_rate_pct`, рівному 0, означає, що
запуск і синхронізація кабелю працюють ідеально, а 100 % втрат припадає на мережевий шлях — знизьте
`wire_ceiling_mbps` і підключіться заново. Зворотна картина вказує на кабель синхронізації або
лінію тригера.

> **`target_fps` не є показником пошкоджених кадрів.** Темп GevSCPD задається одноразово під час
> підключенні, тому зниження частоти тригера змінює робочий цикл, а не
> швидкість одночасного випромінювання пакетів. Виміряне 5-кратне скорочення навантаження не дало поліпшення, тоді як
> зниження максимальної пропускної здатності лінії з 240 до 200 Мб/с зменшило частку пошкоджених кадрів на цій же установці з 10,4 % до
> 0,00 %.

> **Функція автоматичного скорочення потоку в середині передачі недоступна у прошивці TRI032S.** Масив, що працює, не може
> самостійно виправити цю ситуацію; від’єднайте та під’єднайте знову, щоб модуль вибору під час підключення перепланував роботу з урахуванням
> нової верхньої межі.

**Швидкість USB-адаптера Ethernet обмежується зондом до 200 МБ/с** незалежно від його
номінальних характеристик: таблиця ефективності, яка перетворює швидкість каналу на стабільне значення,
базується на PCIe, а USB-мережева карта оголошує свою швидкість Ethernet-каналу, будучи обмеженою
USB-шиною та її драйвером. Обмеження є абсолютним, а не частковим — USB-адаптер 1 Гбіт/с
досягає ~80 МБ/с і не підпадає під це обмеження.

#### Методи `ArraySession`

| Метод | Опис |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Одна синхронізована група захоплення. Повертає `CaptureResult` (список словників кадрів + `.skipped`). Експортні елементи керування наведено нижче. |
| `capture(..., smart=True)` | **Розумний запис** — чекає, поки AE стабілізується на всіх камерах, а потім запускає запис. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Найшвидше захоплення: тільки необроблені дані + призначене значення DAQ (+ вільний комбінований індекс). Відповідає кнопці «Fastest Capture» (Найшвидше захоплення) у графічному інтерфейсі. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Одноразовий / безперервний / інтервальний режим в одному обмеженому циклі. Повертає `list[CaptureResult]`.**Потрібно `count` та/або `duration_s`**, щоб завершити роботу (SDK не підтримує Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Почати запис комбінованого індексного перегляду в режимі реального часу у формат відео/GIF → `RecorderHandle`. Один комбінований записник на масив. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Запуск серії знімків у форматі raw-Bayer із високою частотою кадрів → `RecorderHandle`. Повторна обробка в автономному режимі за допомогою `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Офлайн-переобробка збереженої серії знімків у форматі RAW у відкалібровані відео. Блокує виконання до завершення (`wait=True`) і повертає `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Опитування автономного завдання збірки: `{running, result, error, burst_dir}`. |
| `disconnect()` | Звільнення всього масиву. |

`capture()` — керування експортом (та сама кінцева точка, що й у GUI/CLI):

- `processing` / `levels` — `processing="all"` (або `levels=["raw","radiance",…]`) зберігає кожен відповідний тип експорту для кожної камери; одне значення `processing` зберігає лише цей рівень.
- `aligned=True` — вирівнює несирий експорт кожного елемента за [профілем вирівнювання](#array-alignment) масиву (з сумісною реєстрацією); сирі дані залишаються невирівняними, але містять перетворення в метаданих. Якщо масив не має профілю, використовується невирівняне значення (з попередженням, що відображається в `alignment` результату).
- `render_index=False` — пропускає накладення індексу рослинності для кожної камери; за замовчуванням відображає його там, де це налаштовано.
- `force_daq=True` — зберігає призначене значення DAQ/DLS як додатковий файл `.daq`, навіть якщо жоден обраний рівень цього не потребує.

**Стиснення TIFF (регулятор доступний лише для HTTP):**`ArraySession.capture()` не надсилає ключ `compression`, тому застосовується значення за замовчуванням бекенду — `POST /api/camera/array/capture` зчитує параметр тіла `compression`, `"deflate"` за замовчуванням (zlib L1 без втрат + горизонтальний предиктор, ~4,1 МБ на кадр у повній роздільній здатності). `"none"` записує дані в нестисненому вигляді (~6,3 МБ/кадр) із**приблизно в 5 разів швидшим записом** — обидва формати є безвтратними та читаються однаково під час імпорту. SDK не надає для цього жодного аргументу; альтернативним варіантом є `chloros-cli lattice array-capture --compression none` або необроблений HTTP. DEFLATE також утримує GIL Python, тому стиснені записи не паралелізуються між потоками запису для кожної камери — тривалий 8-камерний запис у повній роздільній здатності зі швидкістю сенсора вимагає `compression: "none"`. Деталі: [CLI Довідка → array-capture](cli-reference.md).**Перевизначення експорту для окремих елементів (лише HTTP):**ця ж кінцева точка також приймає `exclude_serials` (список — виключення елементів із збереженого набору; масив все одно спрацьовує як одна синхронізована група, а виключені елементи повертаються у `excluded`), `serial_levels` (перевизначення на рівні камери `{serial: [level tokens]}`) та `serial_index` (`{serial: bool}` — перевизначення накладення індексу для кожної камери). Це параметри тіла, що відповідають параметрам графічного інтерфейсу, і**поки що не є аргументами ключа SDK**; елементи, яких немає в мапах, використовують значення для всього масиву `levels` / `render_index`.

##### Перевірка пропущених камер — `CaptureResult.skipped`

`ArraySession.capture()` повертає `CaptureResult`, який є підкласом `list`класу: проходьте його, індексуйте, застосовуйте до нього `len()` — усі існуючі шаблони продовжують працювати. Новий код може перевіряти атрибут `.skipped`, щоб побачити, які камери були виключені та чому. Найпоширенішим випадком є RGB у масиві зі змішаними фільтрами, коли ви запитуєте `processing="radiance"` або `"reflectance"` — інтенсивність випромінювання на піксель за байєром не має сенсу для широкосмугового датчика, тому сервер пропускає ці камери, замість того щоб видавати безглузді дані.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Токени причин відповідають шаблону `<level>-not-applicable-to-rgb-cam` (один запис на кожен пропущений рівень, кожен із яких містить `level`). Пропуски, пов’язані з коефіцієнтом відбиття, — це `reflectance-skipped-no-fresh-dls` (немає нових даних про нисхідне випромінювання), `reflectance-skipped-bound-daq-unavailable (…)` (неможливо недоступний), а також `dls-uncalibrated-band-<nm>` — діапазон переважно лежить поза межами радіометрично відкаліброваного діапазону світлового датчика DAQ(~374–974 нм), тому абсолютне розділення за відбивною здатністю на основі DAQ відхиляється, і кадр переходить у режим, що повністю залежить від чутливості датчика. Серед серійних моделей це викликає лише F988; підтримуваним режимом роботи цієї камери є робочий процес із використанням панелі відбивної здатності.

Рівні `processing`:

| Рівень | Вихід |
| --- | --- |
| `"raw"` | Одноканальний Байєр (монохромні камери: один діапазон) безпосередньо з датчика. |
| `"debayered"` *(SDK за замовчуванням)* | 3-канальний BGR через білінійне демозаїкування (монохромні камери: 1-канальний відтінок сірого). |
| `"radiance"` | float32 Вт/м²/sr/нм через повний радіометричний ланцюг. Тільки мультиспектральний режим — камери RGB пропускаються. |
| `"reflectance"` | uint16 0..32768 (сумісний з Pix4D); для абсолютного еталону потрібне підключення до системи збору даних у реальному часі. Тільки мультиспектральний режим. |
| `"display"` | Повний ланцюг, що відповідає попередньому перегляду в графічному інтерфейсі (CCM + WB + гамма згідно з профілем камери). |
| `"all"` | **Один файл на відповідний рівень** для кожної камери (відповідає налаштуванню в графічному інтерфейсі «Записати все» / CLI за замовчуванням). Повернутий файл `CaptureResult` містить один словник кадрів на `(cam, level)`, із зазначенням рівня в кожному словнику; рівні, що не застосовуються, відображаються у файлі `.skipped`. Значення DAQ, використане для будь-якого кадру відбиття, зберігається як додатковий файл `.daq`. |

> **Примітка — значення за замовчуванням відрізняється від CLI.** За замовчуванням `ArraySession.capture()` встановлює `processing="debayered"`; команда `chloros-cli lattice array-capture` за замовчуванням за замовчуванням на `processing="all"`. Явно передайте `processing="all"` з SDK, щоб віддзеркалити багаторівневе збереження CLI/GUI.

### Режими зйомки та записуючі пристрої

Поверхня масиву відображає панель зйомки графічного інтерфейсу: режими «Одноразовий», «Безперервний», «Інтервальний» та «Найшвидший затвор», а також два записуючі пристрої (композитне відео в реальному часі та серія необроблених кадрів → офлайн-переробка).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**— це цикл «Безперервний/Інтервальний» SDK. Оскільки немає `Ctrl+C` для припинення циклу зі скрипта, ви**повинні** передати `count` та/або `duration_s` (цикл зупиняється при досягненні будь-якого з них). `interval_s` вимірюється від початку кожного проходу (відповідно до графічного інтерфейсу). Решта аргументів kwargs передаються безпосередньо до `capture()`.
- **`record`** має *рівень моніторингу*: він фіксує комбінований індекс у реальному часі так, як він відображається, тому комбінований потік має бути відкритим, щоб кадри надходили. Один реєстратор композитного сигналу на масив (викликає виняток, якщо такий вже працює).
- **`burst` → `build_video`** призначений для *аналізу*: `burst` записує необроблені кадри + маніфест для кожного кадру + один `.daq` для кожного окремого значення DLS у рамках `<output>/bursts/<base>/` із повною швидкістю циклу зчитування (без ланцюга, без exiftool, без попереднього перегляду). `build_video` синхронізує кожен кадр за часом з найближчим `.daq` і повторно запускає ланцюжок «яскравість/відбивна здатність/індекс» конвеєра імпорту. `products` — це список `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (за замовчуванням: комбінований індекс). `burst().stop()` також автоматично запускає побудову комбінованого індексу з максимальною точністю, результат якої повертається як `build_job` у підсумковому результаті.

#### `RecorderHandle`

Повертається функціями `ArraySession.record()` та `ArraySession.burst()`. Використовуйте його як менеджер контексту для автоматичного зупинення при виході з області дії або керуйте ним вручну.

| Елемент | Опис |
| --- | --- |
| `job_id` | Ідентифікатор завдання бекенду (str). |
| `kind` | `"composite"` (від `record`) або `"raw"` (з `burst`). |
| `start_stats` | Словник, повернений викликом `start`. |
| `result` | `None` під час виконання; кінцевий словник результатів зупинки після зупинки. |
| `stats(timeout=10.0)` | Статистика поточного завдання (кількість записаних кадрів, фактична частота кадрів, час, що минув). |
| `stop(timeout=60.0)` | Зупинка записувача; повертає та кешує кінцевий результат. Ідемпотентна операція (другий виклик повертає кешований результат). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Приєднання до вже підключеного масиву — `attach_array`

Якщо масив уже активний (його відкрив графічний інтерфейс або попередня сесія SDK викликала `connect_array`), використовуйте `attach_array`, щоб отримати дескриптор до нього, замість того, щоб підключатися заново. <sn><id>У такій ситуації</id></sn> `connect_array` завжди видає помилку «Камера  <sn>вже є у масиві<id>», оскільки POST-запит `/array/connect` для елемента впулу не є ідемпотентним; `attach_array` зчитує `/api/camera/array/list` і здійснює збіг за array_id або серійними номерами.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Шаблон: Скрипти SDK, що працюють у тому ж орендарі, що й графічний інтерфейс робочого столу, повинні спочатку спробувати `attach_array`, а якщо у пулі ще немає масиву, перейти до `connect_array`.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Важливо — вихід з context-manager ДІЙСНО призводить до роз&#x27;єднання.**`ArraySession.disconnect()` завжди надсилає POST-запит до `/array/disconnect`; тут немає захисного механізму «приєднано, але не належить», як у `CameraSession` / `DAQSensorSession`. Якщо ви використовуєте спільний простір з графічним інтерфейсом і не хочете розбирати масив при виході з області дії,**не використовуйте блок `with`** — збережіть дескриптор у звичайній змінній і пропустіть явне використання `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Допоміжний інструмент для аналізу мережі

Корисно перед відкриттям масиву — дозволяє передбачити, чи підійдуть ваші запропоновані налаштування:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` є одним із `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (інакше — `error`). `auto_capped_fps` означає, що запитувана роздільна здатність відповідає кільцю RX лише при обмеженій частоті спрацьовування — збережіть роздільну здатність і перейдіть від `target_fps=result["recommended"]["recommended_target_fps"]` до `connect_array` (див. [Приклад 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Як читати проекцію** (та сама модель, що й на панелі «Налаштування масиву» в графічному інтерфейсі):

- **Пакетні дані (`frame_bytes_total`) підсумовуються для кожної камери окремо у реальному піксельному форматі кожної камери.**Монохромні камери**M3M**передають Mono12 (2 біта/піксель) незалежно від переданого параметра `pixel_format`, тому кадр у повній роздільній здатності з 4 камер становить**~25 МБ** при використанні трьох монокамер, а не ~12,6 МБ, як передбачалося при припущенні, що всі камери працюють у 8-бітовому режимі. Бекенд визначає формат кожної камери за її моделлю.
- **Пропускна здатність (`burst_fits_nic_ring`) враховує навантаження на вихід**, а не співвідношення «повний пакет проти кільця»: sim-emit працює, коли хост спорожнює приймальне кільце швидше, ніж камери його заповнюють. Хост 10G + камери 1 GbE**допускають** повну роздільну здатність навіть тоді, коли пакет перевищує ємність кільця; хост 1 GbE блокує (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` — це консервативна верхня межа послідовного зчитування** — `max(readout+emit, N×emit)` із-камери обмежена до камерного каналу 1 GbE, незалежно від експозиції. Наприклад, ~2,8 кадрів на секунду для масиву з 4 камер з повною роздільною здатністю та 12-бітним форматом (відповідає виміряним у режимі виконання ~2,7–3,0). Повна модель: [CLI Довідка → Модель частоти кадрів масиву та серійної зйомки](cli-reference.md#array-fps--burst-model).
- **Надмірна підписка (`oversubscribed: true`) означає, що мінімальне значення N × на камеру перевищує безпечний для уникнення колізій верхній межа** — поля частоти кадрів (`achievable_fps_max` / `fps_bright` / `fps_dark`) показують 0, і автоматичне стиснення/об’єднання не може це виправити (вони зменшують кількість байтів на кадр, а не кількість байтів за секунду з заданим темпом). Вирішити проблему можна зменшенням кількості камер, використанням джамбо-фреймів або встановленням швидшої мережевої карти; `max_cams_collision_safe` повідомляє про верхню межу (6 камер з повною роздільною здатністю на 1 GbE при 1500 MTU, 9 — з джамбо-фреймами). У відповіді також містяться коди `aggregate_demand_bps`, `collision_safe_ceiling_bps` та `per_cam_floor_bps` (8 МБ/с). Див. [Перепідписка](#over-subscription-the-per-cam-floor).

### Виявлення та перелік

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Масиви LATTICE виконують безперервну автоекспозицію у фоновому режимі одразу після підключення, але для збіжності налаштувань щойно наведеної сцени потрібен деякий час. **Функція Smart-Capture** — це зручне рішення: вона зчитує значення експозиції кожної камери, чекає, поки масив стабілізується у всьому вікні, а потім запускає зйомку. Це еквівалент графічного інтерфейсу: кнопка «розумна» кнопка зйомки настільної програми викликає ту саму кінцеву точку серверної частини.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

При керуванні через `ChlorosProject` (наступний розділ) ви отримуєте більше налаштувань:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Політика «smart-AE» за замовчуванням є консервативною. Зменште значення `exposure_tolerance_pct` для вимогливої радіометричної роботи; збільште — для швидкозмінних сцен, де вам потрібно лише «достатньо точне» значення.

---

## Сеанси датчиків DAQ

Постійний пул бекенду для спектральних датчиків (DAQ-U через USB, DAQ-M через BLE, DAQ-E через Ethernet). Віддзеркалює функціонал камери: розумне виявлення, повторне використання пулу, ідемпотентне підключення.

### Розумне виявлення (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Пріоритет: Ethernet → BLE → USB. Передайте будь-яку явну підказку, щоб зафіксувати транспортний протокол.

### Зафіксований канал передачі

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Методи `DAQSensorSession`

| Метод | Опис |
| --- | --- |
| `status(timeout=10.0)` | Зведена інформація про запис у пулі (стан потокової передачі/запису, діапазон довжин хвиль, SHA калібрування, час інтеграції, frame_avg, стан AE). |
| `latest(n=1, timeout=10.0)` | Повертає до N найсвіжіших кадрів спектра. |
| `stream_start()` / `stream_stop()` | Відновити / призупинити потокову передачу (дескриптор залишається відкритим). |
| `record_start(output_dir=None, device_name=None)` | Початок запису файлу .daq. Повертає шлях до файлу. Відхиляється для DAQ-U/M без калібрувального пакета AWS (DAQ-E є винятком). |
| `record_stop()` | Зупинити запис. Повертає `{path, rows}`. |
| `disconnect()` | Вивільнення з пулу. Не виконує жодних дій для підключених, але не належних користувачеві дескрипторів. |

> **Профілі корекції верхньої межі (`cap_id`) не є регулятором SDK.** `connect_daq_sensor()` / `DAQSensorSession` не надають доступу до параметра `cap_id` або методу `set_cap`. Виберіть профіль корекції обмеження для флоту за допомогою CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) або `/api/daq` HTTP (`/api/daq/connect` та `/api/daq/<id>/cap-id` підтримують `cap_id`).

### Виявлення — пошук адреси для підключення

`discover_daq_sensors()` сканує USB / BLE / ETH у пошуках датчиків, які ви *можете* відкрити. Це аналог `discover_lattice_cameras()` для DAQ і єдиний спосіб отримати **MAC-адресу BLE пристрою DAQ-M** — DAQ-E має ім’я хосту, а DAQ-U — COM-порт, але MAC-адреса не вказана ні на пристрої, ні в списку ОС.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Поле | Опис |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-порт / MAC-адреса BLE / ім’я хосту — передається до `connect_daq_sensor` як `port=` / `mac=` / `eth_host=`. |
| `display` | Зрозуміла для людини мітка. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` або `None` для порту, який сканування може(послідовні USB-адаптери неможливо розрізнити без зонда, тому невідомі елементи відображаються, а не приховуються). |
| `extra` | Деталі за кожним типом передачі даних (оголошене ім’я BLE, виробник USB, IP-адреса/прошивка DAQ-E/…). Порожні значення опускаються. |

| Параметр | За замовчуванням | Опис |
| --- | --- | --- |
| `transports` | усі три | Послідовність (або рядок у форматі CSV), що обмежує сканування. Варто вказувати, коли ви знаєте, чого хочете — BLE є найповільнішим компонентом. |
| `scan_timeout` | 5 | Вікно сканування для кожного транспорту в секундах; сервер обмежує значення до 1–20. |
| `timeout` | 60,0 | HTTP для всього виклику (як і в інших місцях у SDK). |
| `auto_start_backend` | `True` | Запустити локальний бекенд, якщо жоден не працює. Ніколи не запускається для віддаленого `backend_url`. |

> **Датчики, які вже відкриті в пулі, не відображаються.** Підключений периферійний пристрій BLE припиняє розсилку оголошень, а відкритий COM-порт неможливо перевірити, тому функція discovery перелічує те, що *доступне для підключення*. Очікується порожній результат одразу після підключення пристрою — використовуйте `list_daq_sensors()` для того, що ви вже маєте. Транспорти, сканування яких неможливо виконати (не встановлено bleak / zeroconf), пропускаються, а не викликають помилку, тому машина без Bluetooth все одно отримує відповіді щодо USB та ETH.

### Список

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Спільне використання з графічним інтерфейсом / CLI

Якщо в графічному інтерфейсі вже відкрито датчик, виклик `connect_daq_sensor(port="COM3")` із Python повертає дескриптор із позначкою `already_connected=True`. `disconnect()` сеансу тоді не виконує жодних дій, тому ваш скрипт SDK не вириває датчик з-під GUI під час виходу з області дії.

### Класи прямого керування апаратним забезпеченням (без бекенду)

`daq_sdk` реекспортується функцією `chloros_sdk`, тому ви також можете керувати датчиками наскрізно в процесі роботи без використання бекенду:

> **Доступність:**`daq_sdk` постачається разом із настільною інсталяцією Chloros,**але не** з пакетом PyPI — `pip install chloros-sdk` надає вам `lattice_sdk`, але залишає `chloros_sdk.DAQ_AVAILABLE == False`. Перевірте цей прапорець перед використанням цих класів; на хості, де встановлено лише pip, керуйте датчиком через [`connect_daq_sensor()`](#daq-sensor-sessions), який не потребує локальних транспортних бібліотек.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Віддавайте перевагу шляху smart-connect (`connect_daq_sensor`), якщо вам потрібне спільне володіння з графічним інтерфейсом; використовуйте класи прямого підключення для скриптів без графічного інтерфейсу, які володіють датчиком виключно.

---

## Автоматизація проєктів — `ChlorosProject`

Збережений проект Chloros — це папка, що містить `cameras.json` + `sensors.json` + `project.json`. `open_project` завантажує маніфест, а `connect_all` підключає всі збережені пристрої до мережі із збереженими налаштуваннями — у тому самому стані апаратного забезпечення, який би створив графічний інтерфейс.

### Мінімальний приклад

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Або як менеджер контексту:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Методи `ChlorosProject`

| Метод | Опис |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Виявляє та підключає кожне збережене пристрої. Повертає звіт про підключення для кожного класу. Використовує запущений бекенд, якщо такий перебуває у режимі прослуховування на `127.0.0.1:5000`; в іншому випадку без попередження переходить на пряме (без бекенда) `lattice_sdk` управління пристроями — він ніколи не запускає бекенд. |
| `disconnect_all()` | Закриває всі з’єднання. |
| `capture_all(output_dir=".")` | Один кадр з кожної камери + масив + спектр з кожного датчика. |
| `stream(camera, overlays=False, fps=10.0)` | Генератор, що видає кадри BGR `numpy` з вказаної камери (або масиву). `overlays=False` — це прямий цикл захоплення `lattice_sdk` (масиви генерують словники `{serial: frame}`). `overlays=True` проходить через `ChlorosLocal.camera_stream()` → канал MJPEG бекенду `/api/camera/<serial>/stream-annotated`, при цьому збережений блок камери `ui.overlay` передається як параметри запиту. Вимагає режиму бекенду та **автономної камери**: камера в прямому режимі генерує `RuntimeError` (бекенд не може отримати камеру, якою володіє цей процес), а масив викликає `NotImplementedError` (накладає композицію для кожної камери — передає елемент за іменем). Еквівалент одноразового виклику: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Виконати вирівнювання для кожного масиву, що наразі підключений. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Виконати конвеєр калібрування / індексації для зображень проєкту (обгортає `ChlorosLocal.process`; ці чотири є **єдиними** допустимими ключовими аргументами — `indices=` тощо викликають виняток `TypeError`; встановлює індекси за допомогою `ChlorosLocal.configure()`). Ліниво створює `ChlorosLocal()`, який автоматично запускає бекенд. |

Атрибути:
- `proj.cameras` — `Dict[str, CameraHandle]` з ключем за іменем ТА серійним номером.
- `proj.arrays` — `Dict[str, ArrayHandle]` з ключем за іменем та array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` з ключем за іменем та slot_id.
- `proj.config` — `project.json["config"]` — словник.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Рівні обробки.** `capture()`, `grab()` та `frame_stream()` всі приймають один і той самий токен `processing`
, а ланцюжок є кумулятивним — кожен рівень виконує все, що знаходиться вище нього:

| Рівень | Вихідні дані | Примітки |
| --- | --- | --- |
| `raw` | 1-канальний Байєр, рідний для датчика | Без демозаїки. На цьому рівні накладення недоступні. |
| `debayered` | 3-канальний BGR (**за замовчуванням**) | Білінійна демозаїка. Єдиний рівень, що працює без режиму бекенду. |
| `radiance` | float32, Вт/м²/ср/нм | Повний радіометричний ланцюг: демозаїка + 3×3 розкладання (multispec) + DSNU + вирівнювання поля + шкала NIST, з вирахуванням експозиції × коефіцієнта підсилення, щоб значення були абсолютними. |
| `reflectance` | uint16, 32768 = 1,0 | Яскравість, поділена на інтенсивність опромінення, що спускається (ρ = π·L/E). Потрібне зчитування DLS/DAQ — див. примітку нижче. |
| `display` | 8-бітний, на зразок sRGB | Рендеринг, еквівалентний графічному інтерфейсу: CCM + баланс білого + гамма через активний колірний профіль камери. |

Будь-що, крім `debayered` вимагає режиму бекенду; камера в прямому режимі генерує
`NotImplementedError`. `reflectance` потребує придатного значення інтенсивності опромінення знизу — кінцева точка кадру автоматично підтягує
об’єднані дані DAQ автоматично в слот DLS камери, але без прив’язаного DAQ ланцюг відхиляє
вихідний показник відбиття та чесно позначає пониження в повернутих метаданих, замість того, щоб мовчки
повертати продукт нижчої якості.

> **Шкала DN відбиття — не задавайте її жорстко.** Відбиття LATTICE використовує `32768` = ρ 1,0 і позначає
> XMP `Chloros:PixelScale=32768`; Survey3 використовує `65535` = ρ 1.0 і не містить
> тегів `Chloros:*`. Прочитайте тег і розділіть на нього. Він визначений у домені uint16, тому залишається
> `32768` для кожного формату, що змінює масштаб (16-бітний TIFF, 8-бітний PNG/JPG, 32-бітний відсоток) — спочатку нормалізуйте
> збережений тип даних спочатку до uint16 (×257 з 8-бітного, ×65535 з float). Єдиний виняток:
> запис із 8-бітним джерелом, записаний як 8-бітний TIFF, *обрізається*, а не масштабується, тому жоден масштаб його не описує
> — у цьому випадку Chloros повністю пропускає `PixelScale` та кортеж MicaSense. Відсутність
> тегу у файлі відбиття LATTICE слід трактувати як «відсутність дійсного масштабу», а не як значення за замовчуванням.

> **EXIF передається під час експорту.** `process()` копіює блок GPS вихідного знімка
> **та його ExifIFD** у кожен продукт, тому експортовані файли містять `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` та `CameraSerialNumber`, а також
> геореференціювання. `FocalLength` — це те, на основі чого Pix4D обчислює відстань між точками зйомки; без нього
> реконструкція повертається до вкрай неправильного масштабу (у одному з виміряних випадків ділянка площею 411 м
> перетворилася на ділянку площею 47,8 км). Цей файл навмисно не є `-all:all`: структурні теги IFD0 порушують
> вихідні дані LATTICE, а `ExifImageWidth`/`Height` виключені, оскільки вони описують вихідне
> зображення, а не експортований растр.

Підпрапори етапу зйомки (стосуються радіометричних рівнів — `radiance`, `reflectance`, `display`):

| Прапор | За замовчуванням | Значення |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + плоске поле + 3x3 розкладання + радіометрична шкала NIST. |
| `apply_white_balance` | `True` | WB LUT. Підтримка DLS, коли DAQ прив’язаний до камери. |
| `apply_index` | `False` | Оцінка індексу рослинності. |
| `index_expression` | `None` | Формула заміщення. Непорожня → автоматично вмикає індекс. |
| `annotated` | `False` | Накладення елементів інтерфейсу (зебра/сітка/пікінг). Недоступно для `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Тип повернення — `CapturePathMap`, а не `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` — це `Dict[str, Union[str, List[str]]]`: однорівневий
> `processing` надає кожному послідовному елементу один шлях, тоді як багаторівневий (`"all"` або
> явний список `levels`) надає йому **впорядкований список** усіх продуктів, збережених для цієї
> камери. Комбінований композит у реальному часі, якщо він транслюється, надходить під додатковим
> ключем `"combined"`, а не під серійним номером. Код, що розрахований на `str`, дає збій у
> формі списку без будь-якої перевірки типу— у анотації вказано `Dict[str, str]`
> ще деякий час після випуску форми списку, саме тому існує цей псевдонім. Нормалізуйте
> форму, якщо вам потрібна плоска форма:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Вирівнювання масивів

`ArrayHandle` надає доступ до повної поверхні вирівнювання. Профілі за замовчуванням є сесійними— для збереження даних явно викличте `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Вирівнювання під час підключення

`connect_all(align=...)` може автоматично вирівнювати кожен масив під час підключення:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Якщо параметр не вказано, використовується `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Пряме апаратне забезпечення (без бекенду)

Якщо ви хочете повністю усунути залежність від бекенду (CI, безголові роботи, вбудовані), імпортуйте `lattice_sdk` та `daq_sdk` безпосередньо — обидва вони реекспортуються `chloros_sdk`. Захист на `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` входить до складу пакета PyPI (але потребує наявності середовища виконання Arena SDK), тоді як `daq_sdk` постачається лише з інсталяцією для настільних комп’ютерів.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Пресети та тригер

Три з чотирьох пресетів працюють у режимі **free-run**: камера безперервно експонує, і
`capture()` повертає наступний кадр. `triggered` є винятком — він готує
камеру до апаратного тригера на лінії 2, тому нічого не знімає, доки такий тригер не з’явиться.

| Пресет | Тригер | Використовувати, коли |
| --- | --- | --- |
| `default` | вільний режим | загальне використання |
| `high_speed` | вільний режим | 8-бітний, обмеження до 60 кадрів/с, коротка експозиція |
| `high_quality` | вільний режим | 12біт, без обмеження fps — звичайний вибір для фотозйомки |
| `triggered` | **у режимі готовності, лінія 2** | камера підключена до синхрокабелю M8, і її запускає якийсь зовнішній пристрій |

Якщо вибрати `triggered` (або самостійно встановити `trigger_mode="On"`) без
сигналу на лінії 2, кожен `capture()` завершиться через перевищення часу очікування — що є правильним, оскільки ви попросили
камеру чекати. Повідомлення SDK пояснює це, коли це трапляється; див.
[SC_ERR_TIMEOUT під час зйомки](#direct-hardware-backend-free).

> **Примітка — повідомлення «GVSP probe» / `SC_ERR_TIMEOUT -1011` під час підключення не є помилками.**&gt; Під час підключення SDK намагається домовитися про**джамбо-фрейми** (пакети GVSP розміром 9000 байт) для підвищення пропускної здатності. На прямому кабельному з’єднанні «точка-точка» (наприклад, локальна адреса `169.254.x.x`) мережа зазвичай не може передавати джамбо-фрейми, тому цей тест завершується з перевищенням часу очікування, і в журнал записуються такі рядки:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Це **запланований резервний варіант**: SDK автоматично повертається до стандартних пакетів розміром 1500 байт, і камера продовжує підключатися у звичайному режимі (наступні рядки `[chunk-enable …]` є частиною звичайної послідовності підключення). Запис відео все ще працює.
>
> Ви можете пропустити цей тест, але **це не просто засіб приглушення логів — він вимикає джамбо-фрейми.** Камера відповідає на ping-запити з параметром «Don&#x27;t-Fragment» лише для пакетів розміром до 1500 байтів, незалежно від якості вашої мережі, тому сам тест ping ніколи не виявить джамбо-фрейми; лише цей тест може це зробити. Вимкніть його, і камера буде назавжди використовувати стандартні пакети розміром 1500 байт у будь-якій мережі:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Це доцільно лише в мережі, про яку ви *знаєте*, що вона не підтримує джамбо-фрейми, де це економить приблизно одну секунду часу підключення на кожну камеру. Оскільки це реальна, а не косметична зміна, SDK тепер повідомляє про це під час використання:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Не змінюйте налаштування, якщо у вас немає на те причини.** Якщо залишити цю опцію увімкненою, під час кожного підключення система автоматично вимірюватиме реальні параметри вашої мережі: підключіть пристрій до комутатора, що підтримує пакети «джамбо», і під час наступного підключення система самостійно вибере режим «джамбо» — без жодних налаштувань та перезапуску.
>
> Якщо ви *хочете* пропускну здатність у режимі «джамбо», увімкніть режим «джамбо» на всьому шляху (MTU мережевої карти 9000 + комутатор, що їх пропускає), або зафіксуйте його за допомогою `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, якщо знаєте, що канал підтримує це — хоча краще використовувати `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` для окремих команд, ніж встановлювати це назавжди, оскільки зафіксований розмір пропускає тестування та припиняє адаптацію до мережі, що знаходиться перед ним. **Кожен** пристрій на шляху повинен пропускати пакети розміру «джамбо» — включаючи будь-які PoE-розгалужувачі або інжектори, що є типовою причиною, через яку система, яка в іншому випадку підтримує пакети розміру «джамбо», не може їх передавати.

> **`SC_ERR_TIMEOUT -1011` під час `capture()` / `grab*()` — це інша проблема; у цьому випадку йдеться про справжню помилку.**&gt; Зазначене вище стосується лише помилки `-1011`, зареєстрованої**зондом часу підключення**. Та сама помилка, що виникає під час**запису**, означає, що камера підключилася нормально, але не надсилає жодних зображень:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Ознакою цього є камера, *канал керування* якої працює нормально — виявлення працює, налаштування та записи `[chunk-enable …]` виконуються успішно — тоді як *кожен* кадр перевищує ліміт часу.
>
> **Зазвичай це відбувається через те, що камера налаштована на апаратний тригер.** У випадках `trigger_mode="On"` та `trigger_source="Line2"` камера взагалі нічого не передає, доки на синхронізаційному кабелі M8 не з’явиться електричний фронт. Якщо кабель, що керує цією лінією, відсутній, кожне захоплення даних чекає вічно. Камера не поламана, а мережа працює нормально — вона робить саме те, що їй було наказано.
>
> Коди `CameraSettings()` та пресети `default` / `high_speed` / `high_quality` працюють у режимі вільного ходу, і запис, який завершується через перевищення часу очікування під час готовності пояснюють ситуацію, замість того щоб просто виводити код `-1011`. `PRESETS["triggered"]`, згідно з дизайном, активує Line2.
>
> Щоб примусити будь-яку камеру працювати у вільному режимі:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Якщо з `trigger_mode="Off"` все одно відбувається перевищення часу очікування, камера дійсно не передає дані — надішліть нам журнал та `ip link show`.

#### Колірні профілі (RGB попередній перегляд у реальному часі) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` вибирає колірний профіль дисплея для **попереднього перегляду у реальному часі** на камерах RGB (багатоспектральні камери ігнорують це налаштування):

| Профіль | Значення |
| --- | --- |
| `raw` | Повністю обходити радіометричний ланцюг. |
| `linear` | DSNU + flat + WB, без CCM, без гами. |
| `natural` | Лінійний + виміряний CCM + гамма sRGB, лише з «дешевою» обробкою (згладжування кольоровості + знебарвлення світлих ділянок) — реалістичне значення за замовчуванням. |
| `enhanced` | `natural` плюс повна обробка з дотриманням хаб-паритету (усунення ореолів, вібранс, локальний контраст CLAHE). Більш насичений вигляд за приблизно **подвійну вартість обробки на кадр**, отже, нижча частота кадрів у режимі LIVE. |
| `custom_temp` | `natural`, але баланс білого зафіксовано на `custom_cct_k` кельвінів (DLS ігнорується; обмежується діапазоном 2000–10000 К на стороні бекенду). |

Цей профіль є регулятором швидкості/зовнішнього вигляду, що працює **лише в режимі попереднього перегляду в реальному часі**: збережені знімки завжди отримують повне насичене оздоблення незалежно від обраного профілю, тому вибір `natural` для економії часу обробки кадру не знижує якість того, що записується на диск. Невідомий профіль активує `ValueError`; коли бекенд chloros доступний, зміна також надсилається до нього методом POST, щоб наступний кадр попереднього перегляду відображав її (користувачі direct-SDK без бекенда все одно отримують зміну налаштувань).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Монохромні (M3M) камери та `Calibration`

Монохромна камера **M3M** (`M3M-<lens>-F<wavelength>`) має єдинийсмуга: одна площина відтінків сірого, без мозаїки Байєра, без матриці спектрального перехресного впливу 3×3. `Calibration` розпізнає її та виставляє прапор `is_mono`. Відбивна здатність все ще застосовується як радіометрична карта для кожної смуги(матриця розкладу — одинична), але багатосмугові математичні обчислення на одній камері дають сенс, а не безглузді результати:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Щоб побудувати індекс рослинності на основі монохроматичного обладнання, об’єднайте кілька камер M3M з різними довжинами хвиль у вирівняний багатосмуговий стек (див. [Вирівнювання масиву](#array-alignment)) та обчисліть індекс для всього цього масиву, а не для однієї камери.

Прямий режим DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Прийнятні ключі `apply_sensor_settings`**— саме `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; застарілий на користь `cap_id`), `filter_model` (DAQ-M) та `cap_id` (усі типи DAQ; `None`/`""`/`"none"` = «голий» датчик, без корекції верхньої межі). Невідомі ключі**тихо ігноруються** — наприклад, `{"integration_time": 64}` нічого не робить (має бути `integration_time_ms`). Повертає `{"applied": [...], "errors": {...}}` і ніколи не генерує виняток.

`chloros_sdk` реекспортує лише основну поверхню, використану вище. Повний публічний `daq_sdk` API (22 імена) додає наступне — імпортуйте їх безпосередньо з `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Винятки

Визначте базовий клас для обробки «будь-яких помилок Chloros»:

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

> `ChlorosAuthenticationError` та `ChlorosConfigurationError` експортуються на верхньому рівні разом з рештою; їх також можна імпортувати з `chloros_sdk.exceptions`, як показано.

Ієрархія:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Приклади «від початку до кінця»

### 1. Обробка папки з налаштованою смугою прогресу

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Масив LATTICE у реальному часі → Відбивна здатність + еталон DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Кампанія збору даних на основі проєкту

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Потік кадрів з декількох камер → Конвеєр NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Скрипт зйомки без інтерфейсу (без бекенду) безпосередньо на апаратному рівні

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Перевірка можливостей перед підключенням масиву з 4 камер

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Еквівалент рецепта запису (чистий Python)

Мова DSL рецептів CLI має прямий еквівалент у Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Автоматичний запуск бекенду

Точки входу smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, та `discover_lattice_cameras` — є «тонкі» клієнти HTTP, які припускають, що бекенд прослуховує порт `127.0.0.1:5000` (за замовчуванням для інтерфейсу Smart-Connect — URL). Якщо графічний інтерфейс або CLI вже запущено, то один із них працює. У випадку запуску з простого скрипта таких умов може не бути — тому ці функції **автоматично запускають вбудований бінарний файл бекенду** (безвіконний, так само, як це робить `ChlorosLocal`) перед своїм першим викликом, а потім чекають до `backend_startup_timeout`, поки він запуститься.

Правила:

- **Запускається лише локальний URL.** `backend_url`, що вказує на `localhost` / `127.0.0.1` / `[::1]` є допустимим; будь-який інший хост вважається машиною когось іншого і ніколи не запускається.
- **Бекенд залишається запущеним для повторного використання** (так само, як і CLI) — при завершенні роботи вашого скрипта не відбувається автоматичного вимкнення. При повторному запуску скрипта використовується той самий активний бекенд.
- **Відключіть цю функцію за допомогою `auto_start_backend=False`** у будь-якому з цих викликів (наприклад, якщо ви вказали віддалений бекенд або самостійно керуєте життєвим циклом бекенда).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Якщо не вдається знайти або запустити вбудований бінарний файл, наступний виклик HTTP генерує корисний, **залежний від платформи** виклик `ChlorosConnectError` замість простого трасування відмови у з’єднанні — у випадку Windows він вказує на настільну програму або команду `chloros-cli`; у випадку Linux (без графічного інтерфейсу) — до команди `chloros-cli` або до `.deb`.

---

## Середовище та заголовки

SDK позначає кожен виклик бекенду HTTP за допомогою `X-Chloros-Client: sdk`. Бекенд застосовує правила ліцензування SDK/CLI (потрібен вхід у систему **та** платний тарифний план Chloros+) замість безкоштовного тарифу, доступного через графічний інтерфейс. Це налаштовується автоматично під час імпорту — вам не потрібно нічого робити.

`http://localhost` та `http://127.0.0.1` розпізнаються як локальний бекенд. Виклики до інших хостів (наприклад, до вашої власної аналітичної служби) залишаються без змін.

Переопределите бекенд URL, передавши `backend_url=` (або `api_url=` на `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(`backend_url`, що не є петлевим, досягає лише бекенда source/dev — вбудовані бекенди прив’язуються лише до петлі; див. «Режим віддаленого бекенда» для схеми тунелювання.)

---

## Версії та сумісність

- Версія SDK надається як `chloros_sdk.__version__`.
- Поведінка виводів SDK прив’язана до версії бекенду, що входить до комплекту. Поєднання старішого SDK із новішим бекендом зазвичай працює (кінцеві точки сумісні з майбутніми версіями), але поєднання новішого SDK зі старішим бекендом може спричинити появу помилок `404` на нових кінцевих точках — оновлюйте настільну програму відповідно.
- Інтерфейс Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) та кінцева точка аналізу мережі повертають стабільні схеми JSON; нові поля додаються.

---

## Вказівки щодо усунення несправностей

- **`ChlorosAuthenticationError: Login required`** → Запустіть `chloros-cli login EMAIL PASSWORD` один раз на цьому комп’ютері або увійдіть через настільну програму Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Функція Smart-Connect автоматично запускає локальний бекенд, тому це повідомлення з’являється лише тоді, коли не вдається знайти або запустити бінарний файл із комплекту (наприклад, на хості, де встановлено лише pip, без настільного пакета). Повідомлення залежить від платформи: на Windows відкрийте настільну програму або запустіть будь-яку команду `chloros-cli`; на Linux виконайте команду `chloros-cli` (графічного інтерфейсу немає) або встановіть `.deb`. Для віддаленого бекенду передайте `backend_url=` (та `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** під час імпорту → `lattice_sdk` не вдалося завантажити (зазвичай це пов’язано з тим, що DLL-файли виконання Arena SDK не встановлені). Поверхня, не пов’язана з камерою, все ще працює.
- **Функція Array connect повертає роздільну здатність, нижчу за нативну**→ Функція smart-prep бекенду автоматично зменшує розмір кадру, щоб він вписався в канал передачі. Скористайтеся `analyze_array_network()`, щоб з’ясувати причину, після чого або оновьте канал зв’язку, або прийміть зменшення розміру, або передайте `force_tier="slip-emit-and-capture"` для послідовного збору даних. Захисна функція зменшення розміру**не** не охоплює сукупне перевищення пропускної здатності (`oversubscribed: true`, поля fps 0): надмірну кількість камер для каналу не можна виправити за допомогою бінінгу/ROI — зменшіть кількість камер, увімкніть джамбо-кадри або перейдіть на швидшу мережеву карту (див. [Надмірна передплата](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` повідомляє, що кільце прийому мережевої карти дуже мале (~0,26 МБ) / шлюзи підключення з повідомленням «FRAMES WILL DROP»** → Кільце прийому мережевої карти хоста перебуває у стані за замовчуванням (часто скидається до 32 після оновлення драйвера мережевої карти). На адаптері Realtek USB 10GbE встановіть `ReceiveBufferLen=256` та `PendingReceives=64` (з підвищеним рівнем), а потім перезапустіть бекенд, щоб він перечитав кільце. Повна процедура: [CLI Довідка → Налаштування та оптимізація мережевої карти хоста](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Хост зависає під час перезапуску/вимкнення, згодом виникають помилки WMI `Invalid class` / мережева карта не вмикається** → Застарілий драйвер USB 10GbE спричиняє помилку `DRIVER_POWER_STATE_FAILURE` (синій екран `0x9F`). Оновіть драйвер адаптера до актуальної версії (≥ 2026) та повторно застосуйте налаштування приймального кільця. Див. [CLI Довідка → Налаштування та оптимізація мережевої карти хоста](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Відбиття відхилено** → Для вимірювання відбиття в абсолютній шкалі до камери (або масиву) має бути підключений активний модуль збору даних (DAQ). Зв’яжіть через графічний інтерфейс або використовуйте `processing="radiance"` (Вт/м²/ср/нм), що не вимагає спареного датчика.
- **Збір даних за допомогою `smart=True` триває довше, ніж очікувалося** → Збіжність AE залежить від динаміки сцени; збільште значення `exposure_tolerance_pct` або скоротіть `stability_window_s`, якщо вам потрібен швидший (менш стабільний) спрацьовування.

---

## Див. також

- [Довідка щодо CLI](cli-reference.md) — кожна підкоманда CLI відповідає виклику SDK.
- [Посібник із датчиків DAQ](../daq/README.md) — правила підключення, калібрування та запису даних для конкретних датчиків.
- Онлайн-документація: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
