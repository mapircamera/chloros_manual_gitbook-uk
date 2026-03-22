# Встановлення Linux

Chloros поширюється для Linux у вигляді пакетів `.deb`, які встановлюють CLI та серверну частину. Python SDK встановлюється окремо за допомогою pip.

***

## Linux amd64 (x86_64)

### Системні вимоги

| Вимога | Мінімальна | Рекомендована |
| --- | --- | --- |
| **Дистрибутив** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Процесор** | x86_64 (Intel/AMD) | Intel Core i7 або кращий |
| **Пам&#x27;ять (RAM)** | 8 ГБ | 16 ГБ або більше |
| **Відеокарта** | Не потрібна (обробка на процесорі) | Відеокарта NVIDIA з 4 ГБ+ відеопам&#x27;яті |
| **Місце на диску** | 2 ГБ вільного місця | SSD з 10 ГБ+ вільного місця |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

### Встановлення

Завантажте пакет `.deb` та встановіть:

```bash
sudo dpkg -i chloros-amd64.deb
```

Перевірте встановлення:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Системні вимоги

| Вимога | Мінімальна | Рекомендована |
| --- | --- | --- |
| **Платформа** | NVIDIA Jetson з JetPack 6 | Jetson Orin NX 16 ГБ або AGX Orin |
| **JetPack** | JetPack 6.x | Остання версія JetPack 6 |
| **Пам&#x27;ять (RAM)** | 8 ГБ (спільна для GPU/CPU) | 16 ГБ+ спільна |
| **Накопичувач** | 2 ГБ вільного місця | SSD NVMe з 10 ГБ+ вільного місця |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

### Встановлення

Завантажте пакет JetPack 6 `.deb` та встановіть:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Перевірте встановлення:

```bash
chloros-cli --version
```

Детальну інформацію про налаштування Jetson, включаючи управління тепловим режимом та розгортання в польових умовах, див. у [Посібнику NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK Встановлення (Усі Linux)

Python SDK встановлюється окремо за допомогою pip і працює як на amd64, так і на arm64:

```bash
pip install chloros-sdk
```

Щоб включити опціональну підтримку потокового відстеження прогресу:

```bash
pip install chloros-sdk[progress]
```

Перевірте SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Пакет `.deb` встановлює Chloros CLI та бекенд. Python SDK — це окремий пакет pip, який взаємодіє з бекендом через локальний HTTP API.
{% endhint %}

***

## Каталоги конфігурації

Chloros на Linux відповідає [Специфікації базового каталогу XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Призначення | Linux Шлях | Windows Еквівалент |
| --- | --- | --- |
| **Конфігурація** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Дані / Проекти** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Кеш / Повноваження** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Розташування виконуваних файлів бекенду

Пакет `.deb` встановлює бекенд у стандартне місце. Пакет CLI та SDK автоматично визначають шлях до бекенду:

| Метод інсталяції | Шлях до бекенду |
| --- | --- |
| Пакет `.deb` | `/usr/lib/chloros/chloros-backend` |
| Ручний / користувацький | `/opt/mapir/chloros/backend/chloros-backend` |

Ви можете замінити шлях до бекенду за допомогою прапора `--backend-exe` CLI або параметра конструктора `backend_exe` SDK.

***

## Первинне налаштування

### 1. Активуйте свою ліцензію

Для доступу до CLI та SDK потрібна ліцензія Chloros+:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Перевірте стан своєї ліцензії

```bash
chloros-cli status
```

### 3. Обробіть свій перший набір даних

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Запустіть діагностику системи

Переконайтеся, що ваша система налаштована правильно:

```bash
chloros-cli selftest
```

Це запускає 7 діагностичних перевірок, включаючи версію, запуск бекенду, API підключення та доступність CUDA/GPU.

***

## Приклади скриптів Bash

### Обробка декількох наборів даних

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Обробка з власними налаштуваннями

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Автоматизована обробка за допомогою Cron

Додайте до свого crontab (`crontab -e`), щоб автоматично обробляти нові набори даних:

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

### CLI Не знайдено після встановлення

Якщо `chloros-cli` не знайдено після встановлення пакета `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Доступ заборонено

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
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

### CUDA не виявлено

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Відсутні спільні бібліотеки

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Оновлення Chloros на Linux

Використовуйте вбудовану команду оновлення, щоб перевірити наявність оновлень та встановити їх:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Наступні кроки

* [Посібник NVIDIA Jetson](nvidia-jetson-guide.md) — оптимізація та розгортання для Jetson
* [CLI : Командний рядок](../CLI.md) — Повний довідник команд CLI
* [API : Python SDK](../api-python-sdk.md) — Повний довідник SDK
* [Динамічна адаптація обчислень](../processing-architecture/dynamic-compute-adaptation.md) — Як Chloros адаптується до вашого обладнання
