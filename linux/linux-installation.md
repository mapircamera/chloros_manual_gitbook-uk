# Встановлення Linux

Chloros поширюється для Linux у вигляді пакетів `.deb`, які встановлюють CLI та сервер бекенду. Python SDK — це окремий пакет pip (також входить до складу `.deb` як wheel, що відповідає версії).

Імена файлів пакетів містять версію та архітектуру: `chloros_1.2.0_amd64.deb` для x86_64 та `chloros_1.2.0_arm64_jp6.deb` для збірок JetPack 6 Jetson. У наведених нижче командах вкажіть ім’я файлу, який ви фактично завантажили.

***

## Linux amd64 (x86_64)

### Системні вимоги

| Вимога | Мінімальна | Рекомендована |
| --- | --- | --- |
| **Дистрибутив** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Процесор** | x86_64 (Intel/AMD) | Intel Core i7 або кращий |
| **Оперативна пам’ять (RAM)** | 8 ГБ | 16 ГБ або більше |
| **Відеокарта** | Не потрібна (обробка на процесорі) | Відеокарта NVIDIA з 4 ГБ+ відеопам&#x27;яті (12 ГБ+ розблоковує `GPU_PARALLEL`, 7 ГБ+ утримує Texture Aware поза траєкторією одиночного зображення) |
| **Місце на диску** | 2 ГБ вільного місця | SSD з 10 ГБ+ вільного місця |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

> **Ubuntu 20.04 та Debian 11 не підтримуються.** Список залежностей `.deb`
> сформовано на основі того, до чого фактично лінкується бекенд Chloros, а це включає
> `libc6 (>= 2.34)`. У дистрибутивах Focal та bullseye встановлено glibc 2.31, тому `apt` відмовляється від
> встановлення відразу, замість того, щоб дозволити збій пізніше під час виконання.

### Встановлення

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` не вирішує залежності. Якщо він повідомляє про відсутні пакети, `sudo apt-get install -f` (або `sudo apt --fix-broken install`) завершує встановлення — це нормальний хід подій, а не помилка.
{% endhint %}

Перевірте встановлення:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Системні вимоги

| Вимога | Мінімальна | Рекомендована |
| --- | --- | --- |
| **Платформа** | NVIDIA Jetson з JetPack 6 | Jetson Orin NX 16 ГБ або AGX Orin |
| **JetPack** | JetPack 6.x | Остання версія JetPack 6 |
| **Пам&#x27;ять (RAM)** | 8 ГБ (спільна для GPU/CPU) | 16 ГБ+ спільної пам&#x27;яті (12 ГБ+ — поріг для паралельних робочих процесів GPU) |
| **Накопичувач** | 2 ГБ вільного місця | SSD NVMe з 10 ГБ+ вільного місця |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

### Встановлення

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Така сама конфігурація, як і для amd64 `.deb`, із збіркою CUDA, налаштованою для Jetson Orin / Orin NX / Orin Nano. Щодо пам’яті, теплового режиму та поведінки Jetson під час розгортання в польових умовах див. [Посібник NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Встановлення Python та SDK (усі Linux)

SDK — це клієнт, що повністю базується на Python HTTP для бекенду, тому один і той самий пакет працює як на amd64, так і на arm64. Два джерела:**З PyPI** — опублікований стабільний випуск:

```bash
pip install chloros-sdk
```

**З комплекту wheel** — гарантовано сумісний із CLI/backend, який ви щойно встановили (використовуйте це, якщо ваш `.deb` новіший за версію з PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Дистрибутиви PEP 668** (Ubuntu 23.10+, Debian 12+) не підтримують системні інсталяції за допомогою pip. Використовуйте `pip install --user …`, віртуальне середовище або `sudo pip install --break-system-packages …`. Інсталятор пакетів ніколи не встановлює автоматичновстановлює SDK у вашу системну Python — цей вибір залишається за вами.
{% endhint %}

Додаткові компоненти:

| Додатковий компонент | Команда | Додає |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` для потокової передачі прогресу в режимі реального часу |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` для передачі даних через BLE (DAQ-M) |

Перевірте SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` встановлює Chloros, CLI та бекенд. Python та SDK взаємодіють із цим бекендом через локальний HTTP та API (`http://127.0.0.1:5000`) та автоматично запускає його за потреби. Завжди використовуйте літеральну IPv4-адресу, а не `localhost` — `localhost` може перетворюватися на `::1`, що займатиме приблизно дві секунди на кожен запит.
{% endhint %}

***

## Первинне налаштування

### 1. Увійдіть

Для доступу до CLI та SDK необхідний платний тариф Chloros+ (**Copper** або вище), що контролюється на стороні сервера: користувач, який вийшов із системи, отримує `401 AUTH_REQUIRED`, а користувач безкоштовного тарифного плану (Iron) — `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Повноваження кешуються в `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Після кожної інсталяції або оновлення необхідно знову увійти в систему.** Скрипт `prerm`, що входить до складу пакета, навмисно очищає `~/.chloros/user_session.json` та кешовану ліцензію для кожного користувача на комп’ютері, щоб нова збірка завжди повторно перевіряла ліцензію, а не покладалася на застарілий кеш.
{% endhint %}

### 2. Перевірте стан своєї ліцензії

```bash
chloros-cli status
```

`chloros-cli status` працює на будь-якому рівні (включно з безкоштовним), тому ви завжди можете з’ясувати, чому доступ є або відсутній.

### 3. Запустіть діагностику системи

```bash
chloros-cli selftest
```

Виконується сім перевірок послідовно, і команда завершується з ненульовим кодом, якщо хоча б одна з них завершилася невдачею:

| # | Перевірка | Що вона підтверджує |
| --- | --- | --- |
| 1 | **Версія** | CLI повідомляє свою версію (`v1.2.0`). |
| 2 | **Порт доступний** | Порт 5000 вільний, *або* на нього вже відповів працездатний бекенд Chloros (що вважається успішним результатом). |
| 3 | **Запуск бекенду** | Бінарний файл бекенду запускається. |
| 4 | **Тест API (`/api/test`)** | Бекенд відповідає `status: ok`. |
| 5 | **Інформація про систему** | Виводить `GPU: <name>, CUDA: <bool>, PyTorch: <version>` з `/api/system-info`. |
| 6 | **Моделі шумозаглушувача** | Знаходить моделі `*.pth.enc` (на Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + фільтр шуму**| Функція Texture Aware дійсно працює — потребує CUDA**та** принаймні одного файлу моделі. |

Виконання завершується на `N/7 checks passed` із переліком усіх помилок за назвами.

### 4. Обробка першого набору даних

```bash
chloros-cli process ~/datasets/flight001
```

***

## Файли та каталоги

### Для кожного користувача

Chloros зберігає свої облікові дані та конфігурацію CLI в єдиному міжплатформовому каталозі **`~/.chloros/`** (на Windows, `%USERPROFILE%\.chloros\`). Два кеші, специфічні для Linux, натомість дотримуються конвенцій XDG — вони враховують налаштування `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, якщо вони встановлені.

| Шлях | Призначення |
| --- | --- |
| `~/.chloros/user_session.json` | Кеш сеансу входу, що записується `chloros-cli login` (очищується під час кожної інсталяції/оновлення пакета) |
| `~/.chloros/working_directory.txt` | Перезапис папки проекту за замовчуванням (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Налаштування мови CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Налаштування мови, спільне з графічним інтерфейсом Windows — значення `language` тут має пріоритет над `cli_language.json` |
| `~/.chloros/update_cache.json` | Одногодинний кеш для перевірки оновлень під час запуску Linux/Jetson |
| `~/.chloros/backend.log` | Журнал бекенду під час запуску бекенду за допомогою CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Закешовані пакети калібрування LATTICE для кожної камери, індексовані за серійним номером та хешем пакета |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Додаткові налаштування користувача для профілів корекції обмежень DAQ |
| `~/.config/chloros/system_config.json` | Профіль апаратного забезпечення з динамічної адаптації обчислювальних ресурсів (Dynamic Compute Adaptation), збережений у кеші — видаліть його, щоб примусово виконати нове виявлення апаратного забезпечення |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Журнали сервера бекенду, по одному файлу на кожен запуск |
| `~/Chloros Projects/` | Папка проєкту за замовчуванням, якщо не встановлено інших налаштувань |

### Системні

| Шлях | Призначення |
| --- | --- |
| `/usr/bin/chloros-cli` | Скрипт-обгортка — встановлює `LD_LIBRARY_PATH` для вбудованих нативних бібліотек, а потім запускає власне бінарне файл |
| `/usr/bin/chloros-backend` | Скрипт-обгортка — те саме, плюс `CHLOROS_PRODUCTION=1`, щоб механізм авторизації бекенду ніколи не міг самовільно вимкнути себе |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Скомпільовані бінарні файли |
| `/usr/lib/chloros/arena_runtime/` | Середовище виконання Arena SDK, необхідне для камер LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Зашифровані моделі шумозаглушувачів, що використовуються дебейєром Texture Aware |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK — набір, що відповідає саме цій збірці |
| `/usr/lib/chloros/exiftool` | Вбудований exiftool (символічне посилання на `/usr/local/bin/exiftool` створюється лише за відсутності системного exiftool) |
| `/etc/chloros/update.conf` | Конфігурація каналу оновлень, що зчитується `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Налаштовує `net.ipv4.ip_unprivileged_port_start = 319`, щоб бекенд міг прив’язати порти PTP без прав root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Направляє динамічний завантажувач на `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Надає авторизованому користувачеві доступ до послідовного USB-мосту DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Увімкнення постійно активного фонового сервісу (встановлено, **не ввімкнено**) |
| `/usr/share/applications/chloros-cli.desktop` | Пункт меню програми «Chloros CLI», що відкриває термінал |

## Розташування виконуваного файлу фонового сервісу

CLI та SDK автоматично визначають фоновий сервіс:

| Компонент | Шлях |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Бекенд | `/usr/lib/chloros/chloros-backend` |

Перезапишіть шлях до бекенду за допомогою прапора `--backend-exe` CLI або параметра конструктора `backend_exe` SDK, а порт — за допомогою `--port` (за замовчуванням `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` вказує на **`lattice`**,**`project`**та**`daq pool-*`** на віддаленому бекенді. Основні команди (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) навмисно ігнорують його та завжди орієнтуються на `http://127.0.0.1:<port>`.
{% endhint %}

***

## Камери LATTICE та світлові датчики DAQ на Linux

Усі сімейства команд live-hardware працюють на Linux (amd64 та Jetson):

* **`chloros-cli lattice`** — виявлення, підключення, налаштування та зйомка з камер LATTICE та синхронізованих масивів. `.deb` об’єднує необхідне середовище виконання Arena SDK та реєструє його в динамічному завантажувачі.
* **`chloros-cli daq pool-*`** — підключення світлових датчиків DAQ-U/M/E через пул бекенду, передавати відкалібровані спектри та записувати файли `.daq`. Скомпільований CLI постачається лише з сімейством `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — запускає збережений проєкт (його камери, датчики та налаштування обробки) у бездисплейному режимі.
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

`--sensor-id` є обов’язковим для `pool-latest`, `pool-stream`, `pool-record` та `pool-set-cap`; `pool-list` показує ідентифікатори, які наразі знаходяться в пулі.

{% hint style="info" %}
**Для першого підключення DAQ-E на машині з декількома мережевими інтерфейсами віддайте перевагу `--eth-host`.** Функція автоматичного виявлення сканує mDNS і може пропустити інтерфейс датчика через «холодний» кеш ARP, тому перше підключення `pool-connect --eth` після завантаження може завершитися невдачею, навіть якщо датчик працює бездоганно. Вказавши IP-адресу або ім’я хосту датчика, можна повністю пропустити процес виявлення.
{% endhint %}

**Дозволи на послідовний порт DAQ-U** регулюються встановленим правилом udev (`uaccess` + група `dialout`). Якщо датчик, який уже був підключений, залишається недоступним, перезавантажте правила або підключіть його заново:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Повний набір команд дивіться у [довідці щодо CLI](../CLI.md).

### Постійно активний PTP для хостів без монітора

Під час першої інсталяції модуль systemd `chloros-backend.service` генерується, але **не вмикається**. На бездисплейному Jetson або сервері, де синхронізація часу PTP має працювати безперервно для датчиків DAQ-E та камер LATTICE, увімкніть її:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Без цього PTP працює лише під час роботи бекенду Chloros — тобто під час активної сесії CLI/SDK.

Пристрій прив’язує бекенд до `127.0.0.1:5000` (параметри середовища `CHLOROS_HOST` / `CHLOROS_PORT` всередині пристрою; перезаписати на `sudo systemctl edit chloros-backend.service`) і перезапускає його у разі збою через 5 секунд.

**Як PTP отримує свої порти.** PTP використовує UDP 319/320, обидва нижче звичайного мінімального рівня привілейованих портів 1024. Пакет `postinst` записує `/etc/sysctl.d/60-chloros-ptp.conf` із значенням `net.ipv4.ip_unprivileged_port_start = 319`, що дозволяє бекенду прив’язувати їх під час роботи від імені вашого користувача. Він також застосовує `setcap cap_net_bind_service,cap_net_raw=+ep` до бінарного файлу бекенду як додатковий запобіжний захід — саме тому `libcap2-bin` є оголошеною залежністю цього пакета.***

## Приклади скриптів у Bash

{% hint style="info" %}
**Коди завершення, зручні для скриптів.**`chloros-cli process` завершує роботу з кодом `0` у разі успіху та**з кодом, відмінним від нуля, у разі невдачі — включаючи запуск, який запитував зображення, але не записав жодного** (він виводить `Processing finished but wrote no image products.` та вказує назву папки проєкту й типові причини). У разі успішного виконання повідомляється, скільки зображень було записано (`Image products written: N`). Коди завершення: `0` — успіх, `1` — збій, `2` — помилка аргументу, `130` — перервано.
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
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Дійсних значень `--format` є рівно чотири, і вони містять пробіли — завжди вказуйте їх у лапках:

| Значення `--format` | Папка виводу |
| --- | --- |
| `TIFF (16-bit)` *(за замовчуванням)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` приймає `standard` (за замовчуванням) або `texture-aware` (Chloros+).

### Автоматизована обробка за допомогою Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Приклад Python SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Усунення несправностей

### CLI не знайдено після встановлення

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
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Помилка «setcap failed» під час встановлення

`.deb` застосовує `cap_net_bind_service` до `/usr/lib/chloros/chloros-backend`, щоб мати змогу прив’язати PTP-порти 319/320 без прав суперкористувача. Якщо під час встановлення бракувало `libcap2-bin`, виклик пропускається. Встановіть його та переінсталюйте пакет:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP не запускається / не може прив’язати порт 319

Переконайтеся, що мінімальне значення для непривілейованих портів було знижено, і, якщо це не було зроблено, застосуйте його для поточного завантаження:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Потім перевірте головний сервер:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### «Не знайдено драйвери камер LATTICE»

Не вдається вирішити проблему з середовищем виконання Arena SDK. Переконайтеся, що конфігурація завантажувача, яку записує пакет, присутня та оновлена:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
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

Журнали бекенду щодо невдалого запуску містяться у файлі `~/.cache/chloros/logs/`.

### CUDA не виявлено

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` повідомляє про те саме в одному рядку: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Відсутні спільні бібліотеки

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Повільний запуск на системах із SD-картками

Скомпільовані бінарні файли розпаковуються у тимчасовий каталог під час кожного запуску. Якщо файл `/mnt/ssd/tmp` існує, Chloros використовує його автоматично; в іншому випадку вкажіть `TMPDIR` для швидкої файлової системи:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Оновлення Chloros на Linux

Команда `update` доступна лише на Linux/Jetson. Вона перевіряє версію, опубліковану в каналі оновлень, налаштованому в `/etc/chloros/update.conf`, і пропонує завантажити та встановити відповідну версію `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

На Linux/Jetson CLI також виконує неблокуючу перевірку оновлень під час кожного запуску (результат зберігається у кеші протягом однієї години в `~/.chloros/update_cache.json`) і виводить повідомлення `Update available: vX.Y.Z`, якщо існує новіша версія. Ваші налаштування та проєкти зберігаються після оновлення; після цього вам потрібно буде знову увійти в систему.

## Видалення

```bash
sudo apt remove chloros
```

Видалення зупиняє роботу `chloros-backend.service`, відновлює мінімальне значення для непривілейованих портів за замовчуванням (1024), видаляє символічне посилання на exiftool, що входить до комплекту, та конфігурацію завантажувача Arena, а також очищає кешовані облікові дані. Ваші проєкти та файли даних `~/.chloros/` залишаються без змін.

***

## Наступні кроки

* [Посібник з NVIDIA Jetson](nvidia-jetson-guide.md) — оптимізація та розгортання для Jetson
* [CLI : Командний рядок](../CLI.md) — посібник CLI
* [API : Python SDK](../api-python-sdk.md) — посібник SDK
* [CLI Довідник](../reference/cli-reference.md) та [SDK Довідник](../reference/sdk-reference.md) — вичерпний перелік команд/API для версії 1.2.0
* [Динамічна адаптація обчислень](../processing-architecture/dynamic-compute-adaptation.md) — як Chloros адаптується до вашого обладнання
