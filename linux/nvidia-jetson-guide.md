# Посібник з NVIDIA Jetson

Chloros на NVIDIA Jetson забезпечує обробку мультиспектральних зображень на периферії — у польових умовах, на безпілотних літальних апаратах та у віддалених установках. Chloros автоматично визначає вашу модель Jetson та оптимізує стратегію обробки відповідно до вашого обладнання.

***

## Підтримувані моделі Jetson

| Модель                | Оперативна пам&#x27;ять            | Стратегія обробки                                   | Рекомендоване використання                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 ГБ спільного | `GPU_PARALLEL` (4 робочі процеси)                            | Максимальна продуктивність, великі набори даних                      |
| **Jetson Orin NX**   | 8–16 ГБ спільного  | `GPU_PARALLEL` (3 робочі процеси, 16 ГБ) / `GPU_SINGLE` (8 ГБ) | Основна рекомендація для розгортання в повітрі та в польових умовах |
| **Jetson Orin Nano** | 8 ГБ спільної пам&#x27;яті     | `GPU_SINGLE` (1 робочий процес)                               | Початковий рівень периферійних обчислень                                 |
| **Jetson Nano**      | 4–8 ГБ спільної пам&#x27;яті   | `GPU_SINGLE` (1 робочий процес)                               | Початковий рівень, обмежена пам&#x27;ять                          |

{% hint style="info" %}
**Старі моделі Jetson** (TX2, TX1, Xavier NX) можуть не підтримуватися. Продуктивність буде залежати від доступної пам&#x27;яті графічного процесора та можливостей CUDA.
{% endhint %}

***

## Вимоги

* **JetPack 6.x** (рекомендується найновіша версія)
* **NVIDIA CUDA** (входить до складу JetPack)
* **Ліцензія Chloros+** (необхідна для доступу до CLI/SDK)

## Встановлення

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Загальні відомості про встановлення Linux див. у розділі [Встановлення Linux](linux-installation.md).

***

## Динамічна адаптація обчислень на Jetson

Chloros автоматично визначає вашу модель Jetson і вибирає оптимальну стратегію обробки. **Ручне налаштування не потрібне.**

### Як це працює

Під час запуску Chloros аналізує вашу систему:

1. **Визначає модель Jetson** за допомогою `/proc/device-tree/model`
2. **Зчитує доступну пам&#x27;ять GPU/спільну пам&#x27;ять**

3.**Вибирає стратегію обробки** (`GPU_PARALLEL`, `GPU_SINGLE` або `CPU_PARALLEL`)
4. **Автоматично встановлює кількість робочих процесів, тип конвеєра та розподіл пам&#x27;яті**

### Поведінка для кожної моделі

| Модель Jetson                | Стратегія       | Робочі процеси | Конвеєр                       | Паралельність |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 ГБ**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (ефективне використання пам&#x27;яті) | Послідовне  |
| **Jetson Orin Nano 8 ГБ**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Серіалізований  |
| **Jetson Orin NX 8 ГБ**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Серіалізовано  |
| **Jetson Orin NX 16 ГБ**     | `GPU_PARALLEL` | 3       | `fused_gpu` (повний шлях до графічного процесора)    | Паралельний  |
| **Jetson AGX Orin 32–64 ГБ** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Паралельний  |

{% hint style="success" %}
**Jetson Orin NX 16 ГБ** є ідеальним варіантом для розгортання на периферії — він підтримує стратегію `GPU_PARALLEL` із 3 одночасними робочими процесами, забезпечуючи справжню паралельну обробку на GPU в компактному форм-факторі.
{% endhint %}

Ключовою відмінністю між платформами є **пам&#x27;ять**. Jetson Nano з 8 ГБ спільної пам&#x27;яті повинен обробляти зображення по одному, використовуючи ефективний з точки зору пам&#x27;яті підхід з розбиттям на фрагменти, тоді як Orin NX з 16 ГБ може одночасно обробляти 3 зображення через графічний процесор, використовуючи злитий конвеєр з вищою пропускною здатністю.

Повний довідник з адаптації обчислень див. у розділі [Динамічна адаптація обчислень](../processing-architecture/dynamic-compute-adaptation.md).

***

## Управління тепловим режимом

Пристрої Jetson мають обмежений тепловий запас, особливо в закритих або повітряних установках. Chloros включає автоматичний моніторинг температури та регулювання потужності:

| Температура         | Дія                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | Нормальна робота — повна швидкість обробки          |
| **70°C** (Попередження)  | Автоматичне зменшення розміру пакета                   |
| **80°C** (Критичний) | Агресивне обмеження — зниження паралельності         |
| **90°C** (Вимкнення) | Повне зупинення обробки GPU — необхідне охолодження |

{% hint style="warning" %}
**Забезпечте належну вентиляцію та відведення тепла** для безперервної обробки, особливо в закритих польових корпусах або бортових системах. Терморегулювання зменшить пропускну здатність обробки для захисту апаратного забезпечення.
{% endhint %}

***

## Управління пам&#x27;яттю

Пристрої Jetson використовують **уніфіковану пам&#x27;ять** — графічний процесор (GPU) і центральний процесор (CPU) спільно використовують одну фізичну оперативну пам&#x27;ять (RAM). Це означає, що вказаний обсяг відеопам&#x27;яті (VRAM) (наприклад, 15,3 ГБ на Orin NX 16 ГБ) не є виділеною пам&#x27;яттю для графічного процесора; вона спільно використовується з операційною системою та іншими процесами.

### Рекомендації щодо обміну

Для великих наборів даних або обробки з використанням Texture Aware debayer, Chloros може рекомендувати створити простір для обміну:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Орієнтовний обсяг пам&#x27;яті на зображення:**

* Стандартний debayer: ~10 МБ на зображення
* Texture Aware debayer: ~15 МБ на зображення

Chloros автоматично розраховує необхідний обсяг пам&#x27;яті на основі розміру вашого набору даних і попереджає вас, якщо рекомендується використання підкачки.

### Резервний варіант при нестачі пам&#x27;яті (OOM)

Якщо під час обробки виявлено нестачу пам&#x27;яті:

1. Chloros автоматично зменшує кількість робочих процесів на графічному процесорі
2. Переходить з конвеєра `fused_gpu` на `tiled_gpu` (більш ефективний з точки зору використання пам&#x27;яті)
3. Продовжує обробку зі зниженою пропускною здатністю, а не виходить з ладу

***

## Впровадження в польових умовах

### Питання енергоспоживання

| Модель Jetson     | Типове енергоспоживання | Примітки                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 Вт              | USB-C або циліндричний роз’єм    |
| Jetson Orin Nano | 7–15 Вт              | Циліндричний роз&#x27;єм постійного струму          |
| Jetson Orin NX   | 10–25 Вт             | Циліндричний роз&#x27;єм постійного струму          |
| Jetson AGX Orin  | 15–60 Вт             | USB-C PD або циліндричний роз&#x27;єм |

Сплануйте енергоспоживання для тривалої обробки — пікове споживання енергії відбувається під час інтенсивного використання графічного процесора у потоці 3 (обробка).

### Рекомендації щодо зберігання даних

* **SSD NVMe** настійно рекомендується для розгортань на архітектурі arm64
* SD-карти занадто повільні для обробки — використовуйте їх лише як завантажувальний носій
* Плануйте 2–3 рази більше місця, ніж розмір вихідних даних зображення, для оброблених результатів

### Робота без монітора через SSH

Chloros CLI ідеально підходить для бездисплейних розгортань Jetson:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Автоматизована обробка за допомогою systemd

Створіть службу systemd для автоматизованої обробки:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Поєднайте з таймером systemd для запланованої обробки:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Приклади робочих процесів

### Базова обробка на Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK на Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Пакетна обробка декількох польотів

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Рекомендовані системи Jetson для польового використання

Для польового та повітряного використання розгляньте такі варіанти несучої плати Jetson Orin NX 16 ГБ:

* **Повітряне/дрон**: Системи з класом вібростійкості (MIL-STD), легкі (менше 300 г), з пасивним охолодженням
* **Польові умови**: Водонепроникні корпуси IP67/IP69K з підключенням камери PoE GigE
* **Мінімальні/економічні**: набори для розробників із додатковими корпусами

Зверніться до [MAPIR Служби підтримки](https://www.mapir.camera/community/contact), щоб отримати конкретні рекомендації щодо обладнання для вашого сценарію розгортання.

***

## Наступні кроки

* [Linux Встановлення](linux-installation.md) — Загальні відомості про встановлення Linux
* [Динамічна адаптація обчислювальних потужностей](../processing-architecture/dynamic-compute-adaptation.md) — Повний довідник стратегій обчислення
* [Конвеєр обробки](../processing-architecture/processing-pipeline.md) — Огляд 4-потокового конвеєра
* [CLI : Командний рядок](../CLI.md) — Повний довідник CLI
* [API : Python SDK](../api-python-sdk.md) — Повний довідник SDK
