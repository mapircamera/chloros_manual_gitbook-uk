# CLI : Командний рядок

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** надає потужний доступ до механізму обробки зображень Chloros через командний рядок, що дозволяє автоматизувати, створювати скрипти та працювати в режимі без інтерфейсу для ваших робочих процесів з обробки зображень.

### Ключові особливості

* 🚀 **Автоматизація** — пакетна обробка декількох наборів даних за допомогою скриптів
* 🔗 **Інтеграція** — вбудовування в існуючі робочі процеси та конвеєри
* 💻 **Робота без графічного інтерфейсу** — запуск без графічного інтерфейсу користувача
* 🌍 **Багатомовність** — підтримка 38 мов
* ⚡ **Паралельна обробка** — [Динамічна адаптація обчислювальних потужностей](processing-architecture/dynamic-compute-adaptation.md) автоматично оптимізується під ваше обладнання

### Вимоги

| Вимога          | Деталі                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Операційна система** | Windows 10/11 (64-біт), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Ліцензія**          | Chloros+ ([потрібен платний план](https://cloud.mapir.camera/pricing)) |
| **Пам&#x27;ять**           | Мінімум 8 ГБ оперативної пам&#x27;яті (рекомендується 16 ГБ)                                  |
| **Інтернет**         | Потрібен для активації ліцензії                                     |
| **Місце на диску**       | Залежить від розміру проєкту                                              |

{% hint style="warning" %}
**Вимоги до ліцензії**: Для CLI потрібна платна підписка Chloros+. Стандартні (безкоштовні) плани не мають доступу до CLI. Відвідайте [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), щоб оновити.
{% endhint %}

## Швидкий старт

### Встановлення

#### Windows

CLI автоматично входить до складу інсталятора Chloros:

1. Завантажте та запустіть **Chloros Installer.exe**

2. Пройдіть майстер інсталяції
3. CLI встановлено в: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Інсталятор автоматично додає `chloros-cli` до системного PATH. Перезапустіть термінал після інсталяції.
{% endhint %}

#### Linux

Встановіть пакет `.deb` для вашої архітектури:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Детальні інструкції щодо налаштування Linux див. у розділі [Встановлення Linux](linux/linux-installation.md).

### Перше налаштування

Перед використанням CLI активуйте свою ліцензію Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Основне використання

Обробка папки з налаштуваннями за замовчуванням:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Довідник команд

### Загальна синтаксис

```
chloros-cli [global-options] <command> [command-options]
```

***

## Команди

### `process` - Обробка зображень

Обробка зображень у папці з калібруванням.

**Синтаксис:**

```bash
chloros-cli process <input-folder> [options]
```

**Приклади:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Параметри команди обробки

| Параметр                | Тип    | За замовчуванням        | Опис                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Шлях    | _Обов&#x27;язкове_     | Папка, що містить мультиспектральні зображення у форматі RAW/JPG                                         |
| `-o, --output`        | Шлях    | Те саме, що й вхідні  | Папка для виводу оброблених зображень                                                     |
| `-n, --project-name`  | Рядок  | Автоматично згенеровано | Індивідуальна назва проекту                                                                    |
| `--vignette`          | Прапорець    | Увімкнено        | Увімкнути корекцію віньєтування                                                             |
| `--no-vignette`       | Прапорець    | -              | Вимкнути корекцію віньєтування                                                            |
| `--reflectance`       | Прапорець    | Увімкнено        | Увімкнути калібрування відбиття                                                         |
| `--no-reflectance`    | Прапорець    | -              | Вимкнути калібрування відбиття                                                        |
| `--ppk`               | Прапор    | Вимкнено       | Застосувати корекції PPK на основі даних датчика освітленості .daq                                      |
| `--format`            | Вибір  | TIFF (16-біт)  | Формат виводу: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Ціле число | Авто           | Мінімальний розмір мішені в пікселях для виявлення калібрувальної панелі                          |
| `--target-clustering` | Ціле число | Авто           | Поріг кластеризації мішеней (0–100)                                                    |
| `--debayer`           | Вибір  | `standard`     | Метод дебейєра: `standard` або `texture-aware` (тільки Chloros+)                          |
| `--target`, `--targets` | Прапорець  | Вимкнено       | Шукати калібрувальні цілі лише у підпапці «target» або «targets» (прискорює обробку) |
| `--indices`           | Список    | Немає           | Індекси рослинності для обчислення (наприклад, `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Рядок  | Немає           | Зафіксувати експозицію для моделі камери (контакт 1)                                                 |
| `--exposure-pin-2`    | Рядок  | Немає           | Фіксація експозиції для моделі камери (контакт 2)                                                 |
| `--recal-interval`    | Ціле число | Автоматично           | Інтервал повторної калібрування в секундах                                                      |
| `--timezone-offset`   | Ціле число | 0              | Зсув часового поясу в годинах                                                               |

***

### `login` - Аутентифікація облікового запису

Увійдіть за допомогою своїх облікових даних Chloros+, щоб увімкнути обробку CLI.

**Синтаксис:**

```bash
chloros-cli login <email> <password>
```

**Приклад:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Спеціальні символи**: Використовуйте одинарні лапки навколо паролів, що містять символи, такі як `$`, `!`, або пробіли.
{% endhint %}

**Вихідні дані:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` — Очистити облікові дані

Очистіть збережені облікові дані та вийдіть зі свого облікового запису.

**Синтаксис:**

```bash
chloros-cli logout
```

**Приклад:**

```bash
chloros-cli logout
```

**Вихідні дані:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Користувачі SDK**: Python SDK також надає програмний метод `logout()` для очищення облікових даних у скриптах Python. Детальніше див. [документацію Python SDK](api-python-sdk.md#logout).
{% endhint %}

***

### `status` — Перевірка стану ліцензії

Відображення поточного статусу ліцензії та автентифікації.

**Синтаксис:**

```bash
chloros-cli status
```

**Приклад:**

```bash
chloros-cli status
```

**Вихідні дані:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` — Перевірка ходу експорту

Моніторинг ходу експорту потоку 4 під час або після обробки.

**Синтаксис:**

```bash
chloros-cli export-status
```

**Приклад:**

```bash
chloros-cli export-status
```

**Випадок використання:** Викличте цю команду під час виконання обробки, щоб перевірити хід експорту.***

### `language` — Управління мовою інтерфейсу

Перегляньте або змініть мову інтерфейсу CLI.

**Синтаксис:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Приклади:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Підтримувані мови (всього 38)

| Код    | Мова              | Назва мови      |
| ------- | --------------------- | ---------------- |
| `en`    | Англійська               | English          |
| `es`    | Іспанська               | Español          |
| `pt`    | Португальська            | Português        |
| `fr`    | Французька                | Français         |
| `de`    | Німецька                | Deutsch          |
| `it`    | Італійська               | Italiano         |
| `ja`    | Японська              | 日本語              |
| `ko`    | Корейська                | 한국어              |
| `zh`    | Китайська (спрощена)  | 简体中文             |
| `zh-TW` | Китайська (традиційна) | 繁體中文             |
| `ru`    | Російська               | Русский          |
| `nl`    | Нідерландська                 | Nederlands       |
| `ar`    | Арабська                | العربية          |
| `pl`    | Польська                | Polski           |
| `tr`    | Турецька               | Türkçe           |
| `hi`    | Хінді                 | हिंदी            |
| `id`    | Індонезійська            | Bahasa Indonesia |
| `vi`    | В&#x27;єтнамська            | Tiếng Việt       |
| `th`    | Тайська                  | ไทย              |
| `sv`    | Шведська               | Svenska          |
| `da`    | Данська                | Dansk            |
| `no`    | Норвезька             | Norsk            |
| `fi`    | Фінська               | Suomi            |
| `el`    | Грецька                 | Ελληνικά         |
| `cs`    | Чеська                 | Čeština          |
| `hu`    | Угорська             | Magyar           |
| `ro`    | Румунська              | Română           |
| `uk`    | Українська             | Українська       |
| `pt-BR` | Бразильська португальська  | Português Brasileiro |
| `zh-HK` | Кантонська             | 粵語             |
| `ms`    | Малайська                 | Bahasa Melayu    |
| `sk`    | Словацька                | Slovenčina       |
| `bg`    | Болгарська             | Български        |
| `hr`    | Хорватська              | Hrvatski         |
| `lt`    | Литовська            | Lietuvių         |
| `lv`    | Латвійська               | Latviešu         |
| `et`    | Естонська              | Eesti            |
| `sl`    | Словенська             | Slovenščina      |

{% hint style="success" %}
**Автоматичне збереження**: Ваші мовні налаштування зберігаються у файлі `~/.chloros/cli_language.json` і зберігаються протягом усіх сеансів.
{% endhint %}

***

### `set-project-folder` - Встановити папку проекту за замовчуванням

Змінити розташування папки проекту за замовчуванням (спільне з GUI на Windows).

**Синтаксис:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Приклади:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` — Показати папку проекту

Відобразити поточне розташування папки проекту за замовчуванням.

**Синтаксис:**

```bash
chloros-cli get-project-folder
```

**Приклад:**

```bash
chloros-cli get-project-folder
```

**Результат:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` — Скинути до значень за замовчуванням

Скинути папку проекту до розташування за замовчуванням.

**Синтаксис:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` — Виконати діагностику системи

Виконати 7 діагностичних перевірок для підтвердження конфігурації системи.

**Синтаксис:**

```bash
chloros-cli selftest
```

**Виконані діагностичні перевірки:**

1. Перевірка версії
2. Доступність порту (5000)
3. Запуск бекенду
4. Тест підключення API
5. Інформація про систему та виявлення графічного процесора
6. Перевірка моделей шумозаглушувача
7. Перевірка доступності CUDA

{% hint style="info" %}
**Корисно для усунення несправностей**: Запустіть `selftest` після інсталяції, щоб перевірити, чи правильно налаштована ваша система, особливо на Linux/Jetson, де може знадобитися перевірка налаштувань графічного процесора та CUDA.
{% endhint %}

***

### `update` — Перевірка на наявність оновлень (лише Linux)

Перевірте наявність та встановіть оновлення CLI на системах Linux.

**Синтаксис:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Параметр    | Опис                        |
| --------- | ---------------------------------- |
| `--check` | Тільки перевіряти наявність оновлень, не встановлювати |

{% hint style="info" %}
Ця команда доступна лише в Linux. У Windows оновлення надаються через інсталятор.
{% endhint %}

***

## Глобальні параметри

Ці параметри застосовуються до всіх команд:

| Параметр            | Тип    | За замовчуванням       | Опис                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Шлях    | Визначено автоматично | Шлях до виконуваного файлу бекенду                       |
| `--port`          | Ціле число | 5000          | Номер порту бекенду API                          |
| `--restart`       | Прапор    | -             | Примусовий перезапуск бекенду (завершує існуючі процеси) |
| `--version`       | Прапор    | -             | Показати інформацію про версію та вийти                |
| `--help`          | Прапор    | -             | Показати довідкову інформацію та вийти                   |

{% hint style="info" %}
**Автоматичне виявлення бекенду**: Шлях `--backend-exe` визначається автоматично для кожної платформи:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ручний)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Приклад із глобальними параметрами:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Посібник із налаштуваннями обробки

### Паралельна обробка та динамічна адаптація обчислень

Chloros 1.1.0 включає [Динамічну адаптацію обчислень](processing-architecture/dynamic-compute-adaptation.md) — механізм обробки **автоматично визначає ваше обладнання** та обирає оптимальну стратегію:

| Платформа | Стратегія | Робочі процеси | Конвеєр | Примітки |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 ГБ** | `GPU_SINGLE` | 1 | `tiled_gpu` | Ефективне використання пам&#x27;яті, послідовне |
| **Jetson Orin NX 16 ГБ** | `GPU_PARALLEL` | 3 | `fused_gpu` | Паралельна обробка на GPU |
| **Настільний ПК з 8 ГБ графічного процесора** | `GPU_SINGLE` | 3 | `tiled_gpu` | Хороша продуктивність настільного ПК |
| **Настільний ПК з графічним процесором 12 ГБ+** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Оптимальна продуктивність настільного ПК |
| **Система тільки з процесором** | `CPU_PARALLEL` | ядер — 1 | `cpu_fallback` | Графічний процесор не потрібен |

{% hint style="success" %}
**Не потрібно ручного налаштування!** Chloros автоматично визначає ваш процесор, графічний процесор, оперативну пам&#x27;ять та (на Jetson) теплові датчики, а потім автоматично налаштовує оптимальний конвеєр обробки.
{% endhint %}

### Методи дебайєризації

| Метод | CLI Прапор | Якість | Швидкість | Ліцензія |
| --- | --- | --- | --- | --- |
| **Стандартний (Швидкий, Середня якість)** | `--debayer standard` | Хороша | Швидкий | Безкоштовний / Chloros+ |
| **З урахуванням текстури (повільний, найвища якість)** | `--debayer texture-aware` | Найвища | Повільний | Тільки Chloros+ |

Метод дебайєра за замовчуванням — **Стандартний**. Метод**З урахуванням текстури** використовує модель шумозаглушення на основі штучного інтелекту/машинного навчання для отримання зображення найвищої якості, але вимагає ліцензії Chloros+ та графічного процесора NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Корекція віньєтування

**Що робить:** Коректує падіння освітленості на краях зображення (темні кути, характерні для зображень з камери).

* **Увімкнено за замовчуванням** — більшості користувачів слід залишити цю опцію увімкненою
* Використовуйте `--no-vignette`, щоб вимкнути

{% hint style="success" %}
**Рекомендація**: Завжди вмикайте корекцію віньєтування, щоб забезпечити рівномірну яскравість у всьому кадрі.
{% endhint %}

### Калібрування відбиття

Перетворює необроблені значення датчика на стандартизовані відсотки відбиття за допомогою калібрувальних панелей.

* **Увімкнено за замовчуванням** — Необхідно для аналізу рослинності
* Потрібні калібрувальні панелі на зображеннях
* Використовуйте `--no-reflectance` для вимкнення

{% hint style="info" %}
**Вимоги**: Переконайтеся, що калібрувальні панелі правильно експоновані та видимі на ваших зображеннях для точного перетворення відбиття.
{% endhint %}

### Корекції PPK

**Що робить:** Застосовує корекції Post-Processed Kinematic з використанням даних журналу DAQ-A-SD для підвищення точності GPS.

* **Вимкнено за замовчуванням**
* Використовуйте `--ppk` для увімкнення
* Потрібні файли .daq у папці проєкту від датчика освітлення DAQ-A-SD MAPIR.

### Формати виводу

<table><thead><tr><th width="197">Формат</th><th width="130.20001220703125">Розрядність</th><th width="116.5999755859375">Розмір файлу</th><th>Найкраще підходить для</th></tr></thead><tbody><tr><td><strong>TIFF (16-бітний)</strong> ⭐</td><td>16-бітне ціле число</td><td>Великий</td><td>Аналіз ГІС, фотограмметрія (рекомендовано)</td></tr><tr><td><strong>TIFF (32-біт, відсоток)</strong></td><td>32-бітне число з плаваючою комою</td><td>Дуже велике</td><td>Науковий аналіз, дослідження</td></tr><tr><td><strong>PNG (8-біт)</strong></td><td>8-бітне ціле число</td><td>Середній</td><td>Візуальний огляд, обмін через Інтернет</td></tr><tr><td><strong>JPG (8-біт)</strong></td><td>8-бітне ціле число</td><td>Малий</td><td>Швидкий попередній перегляд, стиснений вихідний файл</td></tr></tbody></table>***

## Автоматизація та скрипти

### Пакетна обробка PowerShell (Windows)

Автоматична обробка декількох папок з наборами даних у Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Скрипт пакетної обробки Windows (Windows)

Простий цикл для пакетної обробки на Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Пакетна обробка в Bash (Linux)

Обробка декількох папок з наборами даних на Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Скрипт автоматизації Python (кросплатформовий)

Розширена автоматизація з обробкою помилок (працює на Windows та Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Робочий процес обробки

### Стандартний робочий процес

1. **Вхідні дані**: Папка, що містить пари зображень у форматі RAW/JPG
2. **Виявлення**: CLI автоматично сканує файли зображень, що підтримуються
3. **Обробка**: Паралельний режим масштабується відповідно до кількості ядер вашого процесора (Chloros+)
4. **Вихідні дані**: Створює підпапки за моделями камер із обробленими зображеннями

### Приклад структури вихідних даних

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Орієнтовний час обробки

Типовий час обробки 100 зображень (по 12 МП кожне):

| Платформа | Режим | Орієнтовний час | Примітки |
| --- | --- | --- | --- |
| **Настільний ПК з 12 ГБ+ GPU** | `GPU_PARALLEL` | 5–10 хв | Найшвидший варіант |
| **Настільний ПК з 8 ГБ GPU** | `GPU_SINGLE` | 10–15 хв | Хороша продуктивність |
| **Jetson Orin NX 16 ГБ** | `GPU_PARALLEL` | 15–25 хв | Обчислення на периферії |
| **Jetson Nano 8 ГБ** | `GPU_SINGLE` | 30–60 хв | Обмежена пам&#x27;ять |
| **Тільки CPU** | `CPU_PARALLEL` | 20–40 хв | Не потрібно GPU |

{% hint style="info" %}
**Порада щодо продуктивності**: Час обробки залежить від кількості зображень, роздільної здатності, методу дебейєрингу та апаратного забезпечення. Дебейєринг з урахуванням текстури займає значно більше часу, ніж стандартний. Детальніше див. [Динамічна адаптація обчислень](processing-architecture/dynamic-compute-adaptation.md).
{% endhint %}

***

## Усунення несправностей

### CLI не знайдено

**Помилка Windows:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Рішення:**

1. Перевірте місце встановлення:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Використовуйте повний шлях, якщо файл відсутній у PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Додайте до PATH вручну:
   * Відкрийте Властивості системи → Змінні середовища
   * Відредагуйте змінну PATH
   * Додайте: `C:\Program Files\Chloros\resources\cli`
   * Перезапустіть термінал

**Linux Помилка:**

```
chloros-cli: command not found
```

**Linux Рішення:**

1. Перевірте інсталяцію:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Перезавантажте оболонку:

```bash
source ~/.bashrc
```

3. Перевірте права доступу:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Не вдалося запустити бекенд**Помилка:**

```

Backend failed to start within 30 seconds
```

**Рішення:**

1. Перевірте, чи бекенд вже працює (спочатку закрийте його)
2. Перевірте, чи брандмауер не блокує (Windows) або перевірте доступність порту (Linux: `lsof -i :5000`)
3. Спробуйте інший порт:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Примусово перезапустіть бекенд:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. У Linux перевірте, чи існує виконуваний файл бекенду:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Проблеми з ліцензією / автентифікацією**Помилка:**

```

Chloros+ license required for CLI access
```

**Рішення:**

1. Переконайтеся, що у вас є активна підписка Chloros+
2. Увійдіть, використовуючи свої облікові дані:

```bash
chloros-cli login user@example.com 'password'
```

3. Перевірте стан ліцензії:

```bash
chloros-cli status
```

4. Зверніться до служби підтримки: info@mapir.camera

***

### Не знайдено зображень**Помилка:**

```

No images found in the specified folder
```

**Рішення:**

1. Перевірте, чи папка містить підтримувані формати (.RAW, .TIF, .JPG)
2. Перевірте, чи правильний шлях до папки (використовуйте лапки для шляхів із пробілами)
3. Переконайтеся, що у вас є права на читання для цієї папки
4. Перевірте, чи правильні розширення файлів

***

### Обробка зупиняється або зависає**Рішення:**

1. Перевірте вільний простір на диску (переконайтеся, що його достатньо для вихідних даних)
2. Закрийте інші програми, щоб звільнити пам&#x27;ять
3. Зменште кількість зображень (обробляйте партіями)

***

### Порт вже використовується**Помилка:**

```

Port 5000 is already in use
```

**Рішення:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Поширені запитання

### З: Чи потрібна ліцензія для CLI?

**В:**Так! Для CLI потрібна платна**ліцензія Chloros+**.

* ❌ Стандартний (безкоштовний) план: CLI відключено
* ✅ Плани Chloros+ (платні): CLI повністю увімкнено

Підпишіться за адресою: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Питання: Чи можу я використовувати CLI на сервері без графічного інтерфейсу?**Відповідь:** Так! CLI працює повністю без графічного інтерфейсу. Це основний варіант використання на Linux.**Сервер Windows:**
* Сервер Windows 2016 або пізнішої версії
* Встановлений пакет Visual C++ Redistributable

**Сервер Linux:**
* Ubuntu 20.04+ / Debian 11+ (amd64) або JetPack 6 (arm64)
* Встановлення за допомогою пакета `.deb`

**Обидві платформи:**
* Мінімум 8 ГБ оперативної пам&#x27;яті (рекомендується 16 ГБ)
* Одноразова активація ліцензії: `chloros-cli login user@example.com 'password'`

***

### Питання: Де зберігаються оброблені зображення?**Відповідь:**За замовчуванням оброблені зображення зберігаються в**тій самій папці, що й вихідні**, у підпапках за моделями камер (наприклад, `Survey3N_RGN/`).

Використовуйте опцію `-o`, щоб вказати іншу папку для збереження:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Питання: Чи можна обробляти кілька папок одночасно?**Відповідь:** Не безпосередньо за допомогою однієї команди, але ви можете використовувати скрипти для послідовної обробки папок. Дивіться розділ [Автоматизація та скрипти](CLI.md#automation--scripting).***

### Питання: Як зберегти вихідні дані CLI у файл журналу?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Питання: Що станеться, якщо я натисну Ctrl+C під час обробки?**Відповідь:** CLI:

1. Плавно зупинить обробку
2. Вимкне серверну частину
3. Вийде з кодом 130

Частково оброблені зображення можуть залишитися у папці виводу.

***

### Питання: Чи можна автоматизувати обробку CLI?**Відповідь:** Звичайно! CLI розроблено для автоматизації. Дивіться [Автоматизація та скрипти](CLI.md#automation--scripting) для PowerShell (Windows), Batch (Windows), Bash (Linux) та Python (кросплатформовий).***

### З: Як перевірити версію CLI?**В:**

```bash
chloros-cli --version
```

**Результат:**

```

Chloros CLI 1.1.0
```

***

## Отримання допомоги

### Довідка командного рядка

Перегляньте довідкову інформацію безпосередньо в CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Канали підтримки

* **Електронна пошта**: info@mapir.camera
* **Веб-сайт**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ціни**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Повні приклади

### Приклад 1: Базова обробка

Обробка з використанням стандартних налаштувань (віньєтка, відбиття):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Приклад 2: Високоякісний науковий результат

32-бітне число з плаваючою комою TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Приклад 3: Швидка обробка попереднього перегляду

8-бітний PNG без калібрування для швидкого перегляду:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Приклад 4: Обробка з корекцією PPK

Застосування PPK-корекції з відбивною здатністю:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Приклад 5: Індивідуальне місце збереження

Обробка в іншому місці з певним форматом:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Приклад 6: Процес автентифікації

Повний процес автентифікації (однаковий на всіх платформах):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Приклад 7: Використання кількох мов

Зміна мови інтерфейсу (однакова на всіх платформах):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
