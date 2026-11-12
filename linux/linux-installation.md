# Встановлення на Linux

Chloros поширюється для Linux у вигляді пакетів `.deb`, які встановлюють CLI, сервер-бекенд та Python SDK (для системи `python3`, а також відповідний за версією файл wheel для virtualenvs).

Імена файлів пакетів містять версію та архітектуру: `chloros_1.3.0_amd64.deb` для x86_64, а `chloros_1.3.0_arm64_jp6.deb` — для arm64 (NVIDIA Jetson, JetPack 6). У наведених нижче командах вкажіть ім’я файлу, який ви фактично завантажили.

{% hint style="info" %}
**Нове у версії 1.3.0: програми постачаються у розпакованому вигляді.** Бекенд та інтерфейс командного рядка (CLI) встановлюються як папки, `/usr/lib/chloros/backend/` та `/usr/lib/chloros/cli/`, замість окремих саморозпаковуваних файлів. Вони більше не розпаковуються у тимчасову папку під час кожного запуску, тому запускаються набагато швидше (на x86_64 бекенд раніше витрачав близько 100 секунд на розпакування під час кожного запуску) і більше не потребують кількох ГБ вільного місця в `/tmp`. Розмір amd64-версії `.deb` також становить приблизно половину від попереднього.
{% endhint %}

***

## Linux amd64 (x86_64)

### Системні вимоги

| Вимога | Мінімальна | Рекомендована |
| --- | --- | --- |
| **Дистрибутив** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Процесор** | x86_64 (Intel/AMD) | Intel Core i7 або кращий |
| **Оперативна пам'ять (RAM)** | 8 ГБ | 16 ГБ або більше |
| **Відеокарта** | Не потрібна (обробка на процесорі) | Відеокарта NVIDIA з 4 ГБ+ відеопам'яті (12 ГБ+ розблоковує `GPU_PARALLEL`, 7 ГБ+ утримує Texture Aware поза трактом обробки одного зображення) |
| **Місце на диску** | 2 ГБ вільного місця | SSD з 10 ГБ+ вільного місця |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

> **Ubuntu 20.04 та Debian 11 не підтримуються.** Список залежностей `.deb`
> сформовано на основі того, з чим фактично пов’язується бекенд Chloros, і це включає
> `libc6`, новішу за glibc 2.31, що постачається у дистрибутивах focal та bullseye, тому `apt` відмовляє у
> встановленні відразу, замість того, щоб дозволити збій пізніше під час виконання.

### Встановлення

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Використовуйте `apt install ./…`, а не `dpkg -i`.** `apt` вирішує залежності пакета, а також встановлює те, що він рекомендує: вимоги Python SDK, `python3-numpy` та `python3-requests`. `dpkg -i` не встановлює жодного з них. CLI та бекенд є автономними і працюють без них, але `import chloros_sdk` потребує їх; у вихідних даних інсталяції вказано всі відсутні компоненти. Залиште `./`, щоб `apt` встановив файл, а не шукав пакет із такою назвою. Пакет також заявляє `libgl1`, який завантажує вбудований OpenCV і якого може бракувати в мінімальному образі сервера.
{% endhint %}

Перевірте встановлення:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Системні вимоги

| Вимога | Мінімальна | Рекомендована |
| --- | --- | --- |
| **Платформа** | NVIDIA Jetson з JetPack 6 | Jetson Orin NX 16 ГБ або AGX Orin |
| **JetPack** | JetPack 6.x | Остання версія JetPack 6 |
| **Пам’ять (RAM)** | 8 ГБ (спільна для GPU/CPU) | 16 ГБ+ спільної пам’яті (12 ГБ+ — поріг для паралельної обробки) |
| **Накопичувач** | 2 ГБ вільного місця | SSD NVMe з 10 ГБ+ вільного місця |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

### Встановлення

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Така сама конфігурація, як і для amd64 `.deb` (включно з Python SDK, буферами прийому та сервісами бекенду), для Jetson Orin / Orin NX / Orin Nano на **JetPack 6** (L4T R36). Його PyTorch — це CUDA-збірка JetPack 6 для Orin, тому обробка на GPU, включаючи дебейер Texture Aware, виконується на GPU Jetson. Пакет не містить самого CUDA: він використовує бібліотеки CUDA та cuDNN, які встановлює JetPack 6 (`sudo apt install nvidia-jetpack` на образі, де їх немає). Якщо графічний процесор з підтримкою CUDA недоступний, обробка виконується на центральному процесорі, а `--debayer texture-aware` використовує стандартний дебейер (у логах бекенду з’являється один рядок `[DEBAYER] … needs a CUDA GPU …`, що пояснює причину). Інформацію про поведінку Jetson щодо пам’яті, теплового режиму та розгортання в польових умовах див. у [Посібнику з NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Встановлення Python SDK (усі версії Linux)**`.deb` автоматично встановлює SDK.** Обидва пакети (amd64 та Jetson) встановлюють його для системи `python3`, тому `import chloros_sdk` працює одразу після встановлення, без pip та без підключення до мережі. Його пакети (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` та завантажувачі проектів) розміщуються в `/usr/lib/python3/dist-packages`, що належить пакету: оновлення замінює їх, а видалення пакета — видаляє їх. Версія завжди відповідає встановленим вами CLI та бекенду.

Власні вимоги SDK, `python3-numpy` та `python3-requests`, є **рекомендованими, а не обов’язковими**: `sudo apt install ./chloros_<version>_<arch>.deb` встановлює їх, а `dpkg -i` — ні. У вихідних даних інсталяції вказується версія SDK, яку імпортує `python3`, будь-які відсутні вимоги, а також про будь-які файли `chloros_sdk`, що знаходяться в інших місцях шляху `python3` і замінюють копію з пакета (наприклад, ті, що були встановлені раніше за допомогою `pip install --user`).**У virtualenv**, який не бачить системних пакетів, встановіть у нього колесо з комплекту (повністю в автономному режимі):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**З PyPI** — лише для машини, на якій не встановлено пакет Chloros, чий SDK керує бекендом на іншій машині (див. [Режим віддаленого бекенду](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Додаткові опції для встановлення за допомогою pip (у virtualenv або з PyPI):

| Додаткова опція | Команда | Додає |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` для потокової передачі прогресу в режимі реального часу |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` для передачі даних через BLE (DAQ-M) |

Перевірте SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python-SDK взаємодіє з бекендом, який `.deb` встановлює через локальний HTTP-API (`http://127.0.0.1:5000`). `ChlorosLocal` та допоміжні програми підключення автоматично запускають його за потреби; `connect_all()` збереженого проєкту ніколи цього не робить, тому на машині без монітора запускайте бекенд як [службу](#always-on-ptp-for-headless-hosts). Завжди використовуйте літеральну IPv4-адресу, а не `localhost` — `localhost` може вирішуватися в `::1` і витрачати приблизно дві секунди на кожен запит.
{% endhint %}

***

## Первинне налаштування

### 1. Увійдіть

Для доступу до CLI та SDK необхідний платний тарифний план Chloros+ (**Copper** або вище), що контролюється на стороні сервера: користувач, який вийшов із системи, отримує `401 AUTH_REQUIRED`, а користувач безкоштовного тарифного плану (Iron) — `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Повноваження кешуються в `~/.chloros/user_session.json`.

{% hint style="info" %}
**При оновленні ваші дані для входу зберігаються; при видаленні вони очищуються.** Встановлення нової версії `.deb` поверх існуючої інсталяції (у тому числі через `chloros-cli update`) зберігає `~/.chloros/user_session.json` та кешовану ліцензію, тому завдання CLI/SDK без графічного інтерфейсу продовжують працювати. Видалення пакета (`sudo apt remove chloros`) очищає збережені дані входу та кеш ліцензії для кожного користувача на комп’ютері, тому для нової інсталяції знову знадобиться `chloros-cli login`.
{% endhint %}

### 2. Перевірте стан своєї ліцензії

```bash
chloros-cli status
```

`chloros-cli status` працює на будь-якому рівні (включно з безкоштовним), тому ви завжди можете дізнатися, чому доступ є або відсутній.

### 3. Запустіть діагностику системи

```bash
chloros-cli selftest
```

Сім перевірок виконуються послідовно, і команда завершується з ненульовим кодом, якщо хоча б одна з них завершилася невдало:

| # | Перевірка | Що вона перевіряє |
| --- | --- | --- |
| 1 | **Версія** | Інтерфейс командного рядка (CLI) повідомляє свою версію (`v1.3.0`). |
| 2 | **Порт доступний** | Порт 5000 вільний, *або* на нього вже відповів працездатний бекенд Chloros (що вважається успішним результатом). |
| 3 | **Запуск бекенда** | Бінарний файл бекенда запускається. |
| 4 | **Тест API (`/api/test`)** | Бекенд відповідає `status: ok`. |
| 5 | **Інформація про систему** | Виводить `GPU: <name>, CUDA: <bool>, PyTorch: <version>` з `/api/system-info`. |
| 6 | **Моделі шумозаглушувача** | Знаходить моделі `*.pth.enc` (у Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + фільтр шуму**| Функція Texture Aware дійсно працює — для цього потрібні CUDA**та** принаймні один файл моделі. |

Виконання завершується `N/7 checks passed`, із переліком усіх помилок за назвами.

### 4. Обробіть свій перший набір даних

```bash
chloros-cli process ~/datasets/flight001
```

***

## Файли та каталоги

### Для кожного користувача

Chloros зберігає свої облікові дані та конфігурацію CLI в одному міжплатформовому каталозі, **`~/.chloros/`** (у Windows — `%USERPROFILE%\.chloros\`). Два кеші, характерні для Linux, натомість дотримуються конвенцій XDG — вони використовують `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, якщо їх налаштовано.

| Шлях | Призначення |
| --- | --- |
| `~/.chloros/user_session.json` | Кеш сеансу входу, записаний `chloros-cli login` (зберігається під час оновлень пакета; очищується при видаленні пакета) |
| `~/.chloros/working_directory.txt` | Перезапис папки проекту за замовчуванням (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Налаштування мови CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Налаштування мови, спільне з графічним інтерфейсом Windows, — значення `language` тут має пріоритет над `cli_language.json` |
| `~/.chloros/update_cache.json` | Одногодинний кеш для перевірки оновлень під час запуску Linux/Jetson |
| `~/.chloros/backend.log` | Журнал серверної частини, коли серверна частина була запущена за допомогою CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Закешовані пакети калібрування LATTICE для кожної камери, індексовані за серійним номером та хешем пакета |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Додаткові налаштування користувача для профілів корекції капацитану DAQ |
| `~/.config/chloros/system_config.json` | Профіль апаратного забезпечення з Dynamic Compute Adaptation у кеші — видаліть його, щоб примусово виконати нове виявлення апаратного забезпечення |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Журнали сервера бекенду, по одному файлу на кожен запуск |
| `~/Documents/MAPIR/Chloros Projects/` | Папка проекту за замовчуванням, якщо не встановлено інших налаштувань |

### Системні

| Шлях | Призначення |
| --- | --- |
| `/usr/bin/chloros-cli` | Скрипт-обгортка — встановлює `LD_LIBRARY_PATH` для вбудованих нативних бібліотек (а також `TMPDIR` на `/mnt/ssd/tmp`, якщо така папка існує), після чого запускає власне бінарне файл |
| `/usr/bin/chloros-backend` | Скрипт-обгортка — те саме для бекенду, який він позначає як встановлену виробничу збірку |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Скомпільовані CLI-інтерфейс та бекенд, встановлені у розпакованому вигляді (нова функція у версії 1.3.0); програми мають назви `cli/chloros-cli` та `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Дворядкові скрипти-запускачі, які запускають вищезазначені програми — зберігаються за цими шляхами, оскільки SDK, CLI та старі скрипти шукають їх саме тут |
| `/usr/lib/chloros/arena_runtime/` | Середовище виконання Arena SDK, необхідне для камер LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Зашифровані моделі шумозаглушувачів, що використовуються дебейєром Texture Aware |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (а також `lattice_sdk`, `lattice_cam`, `arena_api` — завантажувачі проєкту) | Python SDK, встановлений для системи `python3` і належить пакету |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK у форматі wheel, що відповідає саме цій збірці, для virtualenv |
| `/usr/lib/chloros/exiftool` | Вбудований запускач ExifTool (символічне посилання на `/usr/local/bin/exiftool` створюється лише за відсутності системного exiftool). Починаючи з версії 1.3.0 скрипт ExifTool постачається з власною бібліотекою Perl у `/usr/lib/chloros/exiftool-dist/`, тому він працює без системного пакета `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Конфігурація каналу оновлень, що зчитується `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Встановлює `net.ipv4.ip_unprivileged_port_start = 319` так, щоб бекенд міг прив’язувати порти PTP без прав суперкористувача |
| `/etc/sysctl.d/60-chloros-gige.conf` | Збільшує буфери прийому сокетів ядра (`net.core.rmem_max`, `net.core.rmem_default`) до 32 МБ для потоків камер LATTICE та застосовує ці налаштування під час інсталяції (якщо значення вже встановлено на вищий рівень, воно залишається без змін). За замовчуванням ядра пакети з камер відкидаються в ядрі, і масив втрачає кадри та частоту кадрів; `chloros-cli lattice network` повідомляє про низький рівень буферів. Див. [Хости Linux — буфери прийому сокетів](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Вказує динамічному завантажувачу на `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Надає авторизованому користувачеві доступ до послідовного USB-мосту DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Служба бекенду, що працює постійно, яка запускає бекенд від імені вказаного вами користувача (встановлена, **не ввімкнена**) — саме її слід увімкнути, див. [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Той самий сервіс, що запускає бекенд під правами **root**(встановлено,**не ввімкнено**), залишено для того, щоб машина, на якій його ввімкнено, продовжувала працювати |
| `/var/log/chloros/` | Детальний журнал служби root, доступний для читання групі `adm` (видаляється під час повного видалення пакета) |
| `/usr/share/applications/chloros-cli.desktop` | Пункт меню програми «Chloros CLI», що відкриває термінал |

**Брандмауер (нова функція у версії 1.3.0).** Якщо під час встановлення активні `ufw` або `firewalld`, пакет відкриває UDP-порти `5002` (дані DAQ-E), `5353` (виявлення mDNS), `319` та `320` (PTP), а під час видалення пакета ці правила видаляються. Якщо брандмауер встановлено, але він неактивний, він нічого не додає і виводить список портів, які слід дозволити, якщо ви увімкнете його пізніше. Він не відкриває жодного TCP-порту: оновлення прошивки DAQ-E, під час якого датчик підключається до цього комп’ютера через TCP для збору зображення, завершується з помилкою `… device never connected back on TCP <port> …`, якщо активний брандмауер блокує це з’єднання.

## Розташування виконуваного файлу бекенду

CLI та SDK автоматично визначають бекенд:

| Компонент | Шлях |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Бекенд | `/usr/lib/chloros/chloros-backend` (програма запуску для `/usr/lib/chloros/backend/chloros-backend`) |

Перезаписати шлях до бекенду можна за допомогою прапора CLI `--backend-exe` або параметра конструктора SDK `backend_exe`, а порт — за допомогою `--port` (за замовчуванням `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` вказує на **`lattice`**,**`project`**та**`daq pool-*`** на віддалений бекенд. Основні команди (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) навмисно ігнорують його та завжди орієнтуються на `http://127.0.0.1:<port>`.
{% endhint %}

***

## Камери LATTICE та світлові датчики DAQ у Linux

Усі сімейства команд live-hardware працюють у Linux (amd64 та Jetson):

* **`chloros-cli lattice`** — виявлення, підключення, налаштування та зйомка з камер LATTICE та синхронізованих масивів. `.deb` об’єднує необхідне середовище виконання Arena SDK та реєструє його в динамічному завантажувачі.
* **`chloros-cli daq pool-*`** — підключення світлових датчиків DAQ-U/M/E через пул бекенду, потокова передача відкаліброваних спектрів, запис файлів `.daq` та обмін профілями капсул, а для DAQ-E — зміна його мережевих налаштувань, перезапуск та оновлення прошивки. `chloros-cli daq discover` сканує датчики, до яких можна підключитися, а `chloros-cli daq --help` виводить список підкоманд `pool-*`.
* **`chloros-cli project`** — запускає збережений проєкт (його камери, датчики та налаштування обробки) у режимі без монітора.
* **`chloros-cli time-sync`** — перевірити головний сервер PTP, на якому працює бекенд Chloros для камер LATTICE та датчиків DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` є обов’язковим для виконання команд `pool-latest`, `pool-stream`, `pool-record` та `pool-set-cap`; `pool-list` показує ідентифікатори, які наразі знаходяться у пулі.

{% hint style="info" %}
**`--eth-host` пропускає процес виявлення.** `pool-connect --eth` знаходить DAQ-E за допомогою mDNS, використовуючи резервний механізм ARP, який працює з «холодного» кешу ARP; передача IP-адреси або імені хоста датчика за допомогою `--eth-host` дозволяє підключитися до нього безпосередньо.
{% endhint %}

**Дозволи на послідовний порт DAQ-U** обробляються встановленим правилом udev (`uaccess` + група `dialout`). Якщо датчик, який вже був підключений, залишається недоступним, перезавантажте правила або підключіть його заново:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Повний набір команд дивіться у [довіднику CLI](../CLI.md).

### Постійно активний PTP для хостів без монітора

Пакет встановлює дві служби systemd для бекенду, жодна з яких не ввімкнена. На бездисплейному Jetson або сервері, де має постійно працювати синхронізація часу PTP для датчиків DAQ-E та камер LATTICE, або де скрипти SDK підключають збережені проєкти (`connect_all()` ніколи не запускає бекенд), увімкніть службу **від імені вашого користувача**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` запускає бекенд під цим користувачем: записи, які він створює, належать цьому користувачеві, його детальний журнал знаходиться у файлі `~/.cache/chloros/logs`, і він використовує обліковий запис `chloros-cli login` (`~/.chloros`), тому спочатку увійдіть у систему під цим користувачем. Під час встановлення пакета виведеться ця команда із вставленим вашим іменем користувача.

Без служби бекенд (а разом з ним і головний сервер PTP) працює лише тоді, коли його запускає щось — команда `chloros-cli` або скрипт SDK.

Модуль прив’язує бекенд до налаштувань середовища `127.0.0.1:5000` (параметри середовища `CHLOROS_HOST` / `CHLOROS_PORT` всередині модуля; перезапишіть їх за допомогою `sudo systemctl edit chloros-backend@.service`) і перезапускає його у разі збою через 5 секунд.

**Служба з правами root.**`chloros-backend.service` запускає бекенд як**root**. Він все ще встановлений, щоб машина, на якій його ввімкнено, продовжувала працювати, але все, що записує бекенд з правами root, належить користувачеві root, тому користувач не може видалити власні записи; його журнал знаходиться у файлі `/var/log/chloros`, доступ до якого має група `adm`. Щоб перейти до свого користувача:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Бекенд служби є спільним і ніколи не зупиняється клієнтом.** Скрипти `chloros-cli` та SDK використовують його так, як знаходять: `process` працює на ньому, завантажуючи власний проєкт замість останнього. У тих випадках, коли CLI, як правило, перезапускав би бекенд (`--restart` або `process`, імпорт якого нічого не знайшов), він називає модуль, виводить `sudo systemctl restart <unit>` і завершує роботу `1`, оскільки бекенд, зупинений таким чином, залишився б непрацюючим. Перезапустіть його самостійно за допомогою `sudo systemctl restart chloros-backend@$USER.service`. Бекенд служби ніколи не зупиняється самостійно у режимі очікування, і `chloros-cli backend stop` відхиляє його, називаючи команду `systemctl stop`.**Як PTP отримує свої порти.** PTP використовує UDP 319/320, обидва нижче звичайного мінімального рівня привілейованих портів 1024. `postinst` цього пакета записує `/etc/sysctl.d/60-chloros-ptp.conf` із `net.ipv4.ip_unprivileged_port_start = 319`, що дозволяє бекенду прив’язувати їх під час роботи під вашим обліковим записом. Починаючи з версії 1.3.0 пакет не встановлює жодних файлових можливостей (`setcap`) на бекенді, а `libcap2-bin` більше не є залежністю: на розпакованому бекенді файлова можливість завадила б його запуску, тому не додавайте його самостійно.

***

## Приклади скриптів Bash

{% hint style="info" %}
**Коди завершення, зручні для скриптів.**`chloros-cli process` завершує роботу з кодом `0` у разі успіху та**з кодом, відмінним від нуля, у разі невдачі — включаючи запуск, під час якого запитувалися зображення, але жодне не було записано** (він виводить `Processing finished but wrote no image products.` та вказує назву папки проєкту й типові причини). У разі успішного виконання повідомляється, скільки зображень було записано (`Image products written: N`, а також `(plus M clamp masks)`, якщо були записані маски). Коди завершення: `0` — успіх, `1` — невдача, `2` — помилка аргументу, `130` — перервано.
{% endhint %}

### Обробка декількох наборів даних

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Обробка з власними налаштуваннями

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Дійсних значень `--format` є рівно чотири, і вони містять пробіли — завжди вказуйте їх у лапках:

| Значення `--format` | Папка виводу |
| --- | --- |
| Значення `TIFF (16-bit)` *(за замовчуванням)* | `tiff16` |
| Значення `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` підтримує `standard` (за замовчуванням) або `texture-aware` (Chloros+).

### Автоматизована обробка за допомогою Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Приклад SDK-Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Усунення несправностей

### Не знайдено CLI після встановлення

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Доступ заборонено

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Бекенд не запускається після додавання файлових можливостей

Не запускайте `setcap` на бекенді. Починаючи з версії 1.3.0, пакет не встановлює жодних файлових можливостей, а порти PTP 319/320 визначаються налаштуванням `sysctl`, описаним у розділі [Як PTP отримує свої порти](#always-on-ptp-for-headless-hosts). Файлова здатність переводить розпакований бекенд у режим безпечного виконання, в якому він не може знайти власні бібліотеки, тому не запускається. Якщо ви додали таку здатність, видаліть її:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP не запускається / не може прив’язатися до порту 319

Переконайтеся, що мінімальне значення для непривілейованих портів було знижено, і, якщо це не було зроблено, застосуйте його знову для поточного завантаження:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Потім перевірте головний сервер:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Команди `lattice` завершуються з помилкою «Не знайдено драйвери камери LATTICE»

Бекенд не зміг завантажити середовище виконання камери LATTICE, яке було встановлено разом із ним. Усі команди `lattice` виконуються на бекенді; сам CLI не містить драйверів камер. Переінсталюйте Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` встановлює середовище виконання поряд із бекендом, у `/usr/lib/chloros/arena_runtime`, та реєструє його в динамічному завантажувачі. Щоб перевірити цю реєстрацію:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Не вдалося запустити бекенд

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Журнали бекенда щодо невдалого запуску містяться у файлі `~/.cache/chloros/logs/`. Якщо порт 5000 належить службі бекенда (`systemctl status chloros-backend@$USER.service`), перезапустіть службу, а не завершуйте її процес.

### CUDA не виявлено

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` повідомляє про те саме в одному рядку: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Відсутні спільні бібліотеки

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Повільний запуск на системах із SD-картками

Починаючи з версії 1.3.0 бекенд та CLI встановлюються у розпакованому вигляді, тому вони більше не розпаковуються у тимчасовий каталог під час кожного запуску. Тимчасові файли автоматично зберігаються в `/mnt/ssd/tmp`, якщо цей каталог існує; в іншому випадку ви можете вказати для них швидку файлову систему:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Оновлення Chloros на Linux

Команда `update` доступна лише для Linux/Jetson. Вона перевіряє версію, опубліковану в каналі оновлень, налаштованому в `/etc/chloros/update.conf`, і пропонує завантажити та встановити відповідну версію `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

У Linux/Jetson CLI також виконує неблокуючу перевірку оновлень під час кожного запуску (результат кешується протягом однієї години в `~/.chloros/update_cache.json`) та виводить `Update available: vX.Y.Z`, якщо існує новіша версія. Ваші налаштування, проєкти та дані для входу зберігаються після оновлення.

Оновлення пакета зупиняє всі сервіси бекенду на час інсталяції, тому скрипт, що виконується, втрачає доступ до камер та датчиків, а потім запускає заново кожен із тих, що увімкнені (той, що запущений вручну, але ніколи не вмикався, залишається зупиненим, як і після перезавантаження).

## Видалення

```bash
sudo apt remove chloros
```

Видалення зупиняє та вимикає всі сервіси бекенду, відновлює мінімальне значення для непривілейованих портів за замовчуванням (1024), видаляє файл налаштувань буфера прийому (поточні значення зберігаються до перезавантаження), символічне посилання на exiftool, що входить до комплекту, конфігурацію завантажувача Arena, правила брандмауера, додані пакетом, та встановлений ним Python-SDK, а також очищає кешовані облікові дані. Ваші проєкти та файли даних `~/.chloros/` залишаються без змін. `sudo apt purge chloros` також видаляє `/var/log/chloros`.

***

## Наступні кроки

* [Посібник з NVIDIA Jetson](nvidia-jetson-guide.md) — Оптимізація та розгортання, специфічні для Jetson
* [CLI: Командний рядок](../CLI.md) — посібник з CLI
* [API: Python SDK](../api-python-sdk.md) — посібник з SDK
* [Довідник CLI](../reference/cli-reference.md) та [Довідник SDK](../reference/sdk-reference.md) — вичерпні переліки команд/API для версії 1.3.0
* [Динамічна адаптація обчислювальних ресурсів](../processing-architecture/dynamic-compute-adaptation.md) — як Chlorosадаптується до вашого обладнання
