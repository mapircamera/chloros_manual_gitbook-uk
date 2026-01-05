# CLI : Командний рядок

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** забезпечує потужний доступ через командний рядок до механізму обробки зображень Chloros, що дозволяє автоматизувати, створювати скрипти та працювати без монітора у ваших робочих процесах з обробки зображень.

### Ключові особливості

* 🚀 **Автоматизація** - Скриптова пакетна обробка декількох наборів даних
* 🔗 **Інтеграція** - Вбудовування в існуючі робочі процеси та конвеєри
* 💻 **Безголовна робота** - Робота без графічного інтерфейсу
* 🌍 **Багатомовність** - Підтримка 38 мов
* ⚡ **Паралельна обробка** — динамічне масштабування відповідно до потужності вашого процесора (до 16 паралельних робочих процесів)

### Вимоги

| Вимога          | Деталі                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Операційна система** | Windows 10/11 (64-біт)                                              |
| **Ліцензія**          | Chloros+ ([потрібен платний план](https://cloud.mapir.camera/pricing)) |
| **Пам&#x27;ять**           | Мінімум 8 ГБ оперативної пам&#x27;яті (рекомендується 16 ГБ)                                  |
| **Інтернет**         | Потрібен для активації ліцензії                                     |
| **Місце на диску**       | Залежить від розміру проекту                                              |

{% hint style=&quot;warning&quot; %}
**Вимоги до ліцензії**: Для CLI потрібна платна підписка Chloros+. Стандартні (безкоштовні) плани не мають доступу до CLI. Відвідайте [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), щоб оновити.
{% endhint %}

## Швидкий старт

### Встановлення

CLI автоматично включено до інсталятора Chloros:

1. Завантажте та запустіть **Chloros Installer.exe**

2. Завершіть роботу майстра встановлення
3. CLI встановлено в: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Інсталятор автоматично додає `chloros-cli` до системного PATH. Після інсталяції перезапустіть термінал.
{% endhint %}

### Перша настройка

Перед використанням CLI активуйте ліцензію Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Основне використання

Обробіть папку з налаштуваннями за замовчуванням:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Довідник команд

### Загальна синтаксис

```
chloros-cli [global-options] <command> [command-options]
```

***

## Команди

### `process` - Обробити зображення

Обробка зображень у папці з калібруванням.

**Синтаксис:**

```bash
chloros-cli process <input-folder> [options]
```

**Приклад:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Параметри команди обробки

| Параметр                | Тип    | За замовчуванням        | Опис                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Шлях    | _Обов&#x27;язкове_     | Папка, що містить мультиспектральні зображення RAW/JPG                                         |
| `-o, --output`        | Шлях    | Те саме, що й вхідні дані  | Папка виводу для оброблених зображень                                                     |
| `-n, --project-name`  | Строка  | Автоматично згенерована | Ім&#x27;я проекту за замовчуванням                                                                    |
| `--vignette`          | Прапорець    | Увімкнено        | Увімкнути корекцію віньєтування                                                             |
| `--no-vignette`       | Прапорець    | -              | Вимкнути корекцію віньєтування                                                            |
| `--reflectance`       | Прапорець    | Увімкнено        | Увімкнути калібрування відбиття                                                         |
| `--no-reflectance`    | Прапорець    | -              | Вимкнути калібрування відбиття                                                        |
| `--ppk`               | Прапорець    | Вимкнено       | Застосувати корекції PPK з даних датчика освітленості .daq                                      |
| `--format`            | Вибір  | TIFF (16-біт)  | Формат виводу: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Ціле число | Автоматично           | Мінімальний розмір цілі в пікселях для виявлення калібрувальної панелі                          |
| `--target-clustering` | Ціле число | Автоматично           | Поріг кластеризації цілей (0-100)                                                    |
| `--exposure-pin-1`    | Строка  | Немає           | Блокування експозиції для моделі камери (контакт 1)                                                 |
| `--exposure-pin-2`    | Строка  | Немає           | Блокування експозиції для моделі камери (контакт 2)                                                 |
| `--recal-interval`    | Ціле число | Авто           | Інтервал повторної калібрування в секундах                                                      |
| `--timezone-offset`   | Ціле число | 0              | Зсув часового поясу в годинах                                                               |

***

### `login` - Аутентифікація облікового запису

Увійдіть за допомогою своїх облікових даних Chloros+, щоб увімкнути обробку CLI.

**Синтаксис:**

```bash
chloros-cli login <email> <password>
```

**Приклад:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Спеціальні символи**: використовуйте одинарні лапки навколо паролів, що містять такі символи, як `$`, `!` або пробіли.
{% endhint %}

**Вихідні дані:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Очистити облікові дані

Очистіть збережені облікові дані та вийдіть зі свого облікового запису.

**Синтаксис:**

```bash
chloros-cli logout
```

**Приклад:**

```powershell
chloros-cli logout
```

**Вихідні дані:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style=&quot;info&quot; %}
**Користувачі SDK**: Python SDK також надає програмний метод `logout()` для очищення облікових даних у скриптах Python. Детальнішу інформацію див. у [документації Python SDK](api-python-sdk.md#logout).
{% endhint %}

***

### `status` - Перевірка стану ліцензії

Відображення поточного стану ліцензії та автентифікації.

**Синтаксис:**

```bash
chloros-cli status
```

**Приклад:**

```powershell
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

### `export-status` — перевірка ходу експорту

Моніторинг ходу експорту потоку 4 під час або після обробки.

**Синтаксис:**

```bash
chloros-cli export-status
```

**Приклад:**

```powershell
chloros-cli export-status
```

**Випадок використання:** Викличте цю команду під час обробки, щоб перевірити хід експорту.***

### `language` - Керування мовою інтерфейсу

Перегляд або зміна мови інтерфейсу CLI.

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

```powershell
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

| Код    | Мова              | Назва мовою оригіналу      |
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
| `nl`    | Голландська                 | Nederlands       |
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

{% hint style=&quot;success&quot; %}
**Автоматичне збереження**: Ваші мовні налаштування зберігаються в `~/.chloros/cli_language.json` і зберігаються протягом усіх сеансів.
{% endhint %}

***

### `set-project-folder` - Встановити папку проекту за замовчуванням

Змінити розташування папки проекту за замовчуванням (спільне з GUI).

**Синтаксис:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Приклад:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Показати папку проекту

Відобразити поточне розташування папки проекту за замовчуванням.

**Синтаксис:**

```bash
chloros-cli get-project-folder
```

**Приклад:**

```powershell
chloros-cli get-project-folder
```

**Вихідні дані:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Скинути до стандартного значення

Скинути папку проекту до стандартного розташування.

**Синтаксис:**

```bash
chloros-cli reset-project-folder
```

***

## Глобальні параметри

Ці параметри застосовуються до всіх команд:

| Параметр          | Тип    | За замовчуванням       | Опис                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Шлях    | Автоматично виявлений | Шлях до виконуваного файлу бекенду                       |
| `--port`        | Ціле число | 5000          | Номер порту бекенду API                          |
| `--restart`     | Прапорець    | -             | Примусовий перезапуск бекенду (завершення існуючих процесів) |
| `--version`     | Прапорець    | -             | Показати інформацію про версію та вийти                |
| `--help`        | Прапорець    | -             | Показати довідкову інформацію та вийти                   |

**Приклад із глобальними опціями:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Посібник із налаштування обробки

### Паралельна обробка

Chloros+ CLI **автоматично масштабує**паралельну обробку відповідно до можливостей вашого комп&#x27;ютера:**Як це працює:**

* Виявляє ядра процесора та оперативну пам&#x27;ять
* Розподіляє робочі процеси: **2× ядра процесора** (використовує гіперпотоковість)
* **Максимум: 16 паралельних робочих процесів** (для стабільності)**Рівні системи:**

| Тип системи   | Процесор        | Оперативна пам&#x27;ять      | Робочі процеси  | Продуктивність     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Високий рівень**  | 16+ ядер  | 32+ ГБ   | До 16 | Максимальна швидкість   |
| **Середній** | 8-15 ядер | 16-31 ГБ | 8-16     | Відмінна швидкість |
| **Низький**   | 4-7 ядер  | 8-15 ГБ  | 4-8      | Хороша швидкість      |

{% hint style=&quot;success&quot; %}
**Автоматична оптимізація**: CLI автоматично визначає характеристики вашої системи та налаштовує оптимальну паралельну обробку. Ручне налаштування не потрібне!
{% endhint %}

### Методи дебайєризації

CLI використовує **Висока якість (швидше)** як стандартний і рекомендований алгоритм дебайєризації:

| Метод                      | Якість | Швидкість | Опис                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Висока якість (швидший)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Алгоритм з урахуванням країв (за замовчуванням, рекомендований) |

### Корекція віньєтування

**Що робить:** Коректує падіння освітлення на краях зображення (темніші кути, характерні для зображень з камери).

* **Увімкнено за замовчуванням** - Більшість користувачів повинні залишити цю функцію увімкненою
* Використовуйте `--no-vignette`, щоб вимкнути

{% hint style=&quot;success&quot; %}
**Рекомендація**: завжди вмикайте корекцію віньєтування, щоб забезпечити рівномірну яскравість по всьому кадру.
{% endhint %}

### Калібрування відбиття

Перетворює необроблені значення датчика в стандартизовані відсотки відбиття за допомогою калібрувальних панелей.

* **Увімкнено за замовчуванням** — необхідне для аналізу рослинності.
* Потрібні калібрувальні панелі на зображеннях.
* Використовуйте `--no-reflectance`, щоб вимкнути.

{% hint style=&quot;info&quot; %}
**Вимоги**: Для точного перетворення відбиття переконайтеся, що калібрувальні панелі правильно експоновані та видимі на ваших зображеннях.
{% endhint %}

### Корекції PPK

**Що робить:** Застосовує корекції Post-Processed Kinematic за допомогою даних журналу DAQ-A-SD для підвищення точності GPS.

* **За замовчуванням вимкнено**
* Для увімкнення використовуйте `--ppk`
* Потрібні файли .daq у папці проекту з MAPIR DAQ-A-SD світлового датчика.

### Формати виводу

<table><thead><tr><th width="197">Формат</th><th width="130.20001220703125">Розрядність</th><th width="116.5999755859375">Розмір файлу</th><th>Найкраще підходить для</th></tr></thead><tbody><tr><td><strong>TIFF (16-біт)</strong> ⭐</td><td>16-бітне ціле число</td><td>Великий</td><td>ГІС-аналіз, фотограмметрія (рекомендовано)</td></tr><tr><td><strong>TIFF (32-біт, відсоток)</strong></td><td>32-бітне з плаваючою комою</td><td>Дуже велике</td><td>Науковий аналіз, дослідження</td></tr><tr><td><strong>PNG (8-бітний)</strong></td><td>8-бітне ціле число</td><td>Середній</td><td>Візуальний огляд, обмін в Інтернеті</td></tr><tr><td><strong>JPG (8-біт)</strong></td><td>8-бітне ціле число</td><td>Невеликий</td><td>Швидкий попередній перегляд, стиснене виведення</td></tr></tbody></table>***

## Автоматизація та скрипти

### Пакетна обробка PowerShell

Автоматична обробка декількох папок з наборами даних:

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

### Windows Пакетний скрипт

Простий цикл для пакетної обробки:

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

### Python Скрипт автоматизації

Розширена автоматизація з обробкою помилок:

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

1. **Вхідні дані**: папка, що містить пари зображень RAW/JPG
2. **Виявлення**: CLI автоматично сканує підтримувані файли зображень
3. **Обробка**: Паралельний режим масштабується відповідно до ядер вашого процесора (Chloros+)
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

Типовий час обробки 100 зображень (12 МП кожне):

| Режим              | Час      | Апаратне забезпечення                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Паралельний режим** | 5-10 хв  | i7/Ryzen 7, 16 ГБ ОЗУ, SSD (до 16 робочих процесів) |
| **Паралельний режим** | 10–15 хв | i5/Ryzen 5, 8 ГБ ОЗУ, HDD (до 8 робочих процесів)   |

{% hint style=&quot;info&quot; %}
**Порада щодо продуктивності**: Час обробки залежить від кількості зображень, роздільної здатності та технічних характеристик комп’ютера.
{% endhint %}

***

## Усунення несправностей

### CLI Не знайдено

**Помилка:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Рішення:**

1. Перевірте місце встановлення:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Використовуйте повний шлях, якщо його немає в PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Додайте до PATH вручну:
   * Відкрийте «Властивості системи» → «Змінні середовища»
   * Відредагуйте змінну PATH
   * Додайте: `C:\Program Files\Chloros\resources\cli`
   * Перезапустіть термінал.

***

### Не вдалося запустити бекенд.**Помилка:**

```

Backend failed to start within 30 seconds
```

**Рішення:**

1. Перевірте, чи бекенд уже працює (спочатку закрийте його).
2. Перевірте, чи брандмауер Windows не блокує.
3. Спробуйте інший порт:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Примусово перезапустіть бекенд:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Проблеми з ліцензією / автентифікацією**Помилка:**

```

Chloros+ license required for CLI access
```

**Рішення:**

1. Перевірте, чи маєте ви активну підписку Chloros+
2. Увійдіть за допомогою своїх облікових даних:

```powershell
chloros-cli login user@example.com 'password'
```

3. Перевірте стан ліцензії:

```powershell
chloros-cli status
```

4. Зверніться до служби підтримки: info@mapir.camera

***

### Не знайдено зображень**Помилка:**

```

No images found in the specified folder
```

**Рішення:**

1. Перевірте, чи папка містить підтримувані формати (.RAW, .TIF, .JPG).
2. Перевірте, чи шлях до папки правильний (використовуйте лапки для шляхів із пробілами).
3. Переконайтеся, що у вас є права на читання для папки
4. Перевірте, чи правильні розширення файлів

***

### Обробка зупиняється або зависає**Рішення:**

1. Перевірте вільний простір на диску (переконайтеся, що його достатньо для виводу)
2. Закрийте інші програми, щоб звільнити пам&#x27;ять
3. Зменште кількість зображень (обробляйте їх партіями)

***

### Порт вже використовується**Помилка:**

```

Port 5000 is already in use
```

**Рішення:**

Вкажіть інший порт:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Поширені запитання

### З: Чи потрібна ліцензія для CLI?

**В:**Так! Для CLI потрібна платна**ліцензія Chloros+**.

* ❌ Стандартний (безкоштовний) план: CLI вимкнено
* ✅ Платні плани Chloros+: CLI повністю увімкнено

Підпишіться на: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Питання: Чи можна використовувати CLI на сервері без графічного інтерфейсу?**Відповідь:** Так! CLI працює повністю без графічного інтерфейсу. Вимоги:

* Windows Server 2016 або пізніша версія
* Встановлений Visual C++ Redistributable
* Достатня кількість оперативної пам&#x27;яті (мінімум 8 ГБ, рекомендується 16 ГБ)
* Одноразова активація ліцензії GUI на будь-якому комп&#x27;ютері

***

### Питання: Де зберігаються оброблені зображення?**Відповідь:**За замовчуванням оброблені зображення зберігаються в**тій самій папці, що й вхідні**, у підпапках за моделями камер (наприклад, `Survey3N_RGN/`).

Використовуйте опцію `-o`, щоб вказати іншу папку для збереження:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Питання: Чи можна обробляти кілька папок одночасно?**В:** Не безпосередньо за допомогою однієї команди, але ви можете використовувати скрипти для послідовної обробки папок. Див. розділ [Автоматизація та скрипти](CLI.md#automation--scripting).***

### П: Як зберегти вихідні дані CLI у файлі журналу?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Пакетна обробка:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Питання: Що станеться, якщо натиснути Ctrl+C під час обробки?**Відповідь:** CLI:

1. Плавне зупинення обробки
2. Вимкне бекенд
3. Вийде з кодом 130

Частково оброблені зображення можуть залишитися у вихідній папці.

***

### Питання: Чи можна автоматизувати обробку CLI?**Відповідь:** Звичайно! CLI призначений для автоматизації. Дивіться [Автоматизація та скрипти](CLI.md#automation--scripting) для прикладів PowerShell, Batch та Python.***

### Питання: Як перевірити версію CLI?**Відповідь:**

```powershell
chloros-cli --version
```

**Вихідні дані:**

```

Chloros CLI 1.0.2
```

***

## Отримання допомоги

### Довідка командного рядка

Перегляньте довідкову інформацію безпосередньо в CLI:

```powershell
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

### Приклад 1: Основна обробка

Обробка з використанням стандартних налаштувань (віньєтка, відбивання):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Приклад 2: Високоякісний науковий результат

32-бітний плаваючий TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Приклад 3: Швидка обробка попереднього перегляду

8-бітний PNG без калібрування для швидкого перегляду:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Приклад 4: Обробка з корекцією PPK

Застосування корекцій PPK з відбиванням:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Приклад 5: Індивідуальне розташування вихідних даних

Обробка на інший диск із зазначеним форматом:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Приклад 6: Робочий процес автентифікації

Повний робочий процес автентифікації:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Приклад 7: Багатомовне використання

Зміна мови інтерфейсу:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
