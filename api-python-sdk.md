# API : Python SDK

**Chloros Python SDK** забезпечує програмний доступ до механізму обробки зображень Chloros, що дозволяє автоматизувати процеси, створювати власні робочі процеси та безперешкодно інтегрувати їх у ваші додатки Python та дослідницькі конвеєри.

### Основні функції

* 🐍 **Нативний Python** - Чистий, Pythonic API для обробки зображень
* 🔧 **Повний доступ до API** - Повний контроль над обробкою Chloros
* 🚀 **Автоматизація** - Створення налаштованих робочих процесів пакетної обробки
* 🔗 **Інтеграція** - Вбудовуйте Chloros в існуючі додатки Python
* 📊 **Готовий до досліджень** - Ідеально підходить для наукових аналітичних конвеєрів
* ⚡ **Паралельна обробка** - Масштабується до ваших ядер процесора (Chloros+)

### Вимоги

| Вимога          | Деталі                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Повинно бути встановлено локально                                           |
| **Ліцензія**          | Chloros+ ([потрібен платний план](https://cloud.mapir.camera/pricing)) |
| **Операційна система** | Windows 10/11 (64-біт)                                              |
| **Python**           | Python 3.7 або вище                                                |
| **Пам&#x27;ять**           | Мінімум 8 ГБ оперативної пам&#x27;яті (рекомендується 16 ГБ)                                  |
| **Інтернет**         | Необхідний для активації ліцензії                                     |

{% hint style=&quot;warning&quot; %}
**Вимоги до ліцензії**: Python SDK вимагає оплаченої підписки Chloros+ для доступу до API. Стандартні (безкоштовні) плани не мають доступу до API/SDK. Відвідайте [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), щоб оновити.
{% endhint %}

## Швидкий старт

### Встановлення

Встановіть за допомогою pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Перша настройка**: Перед використанням SDK активуйте свою ліцензію Chloros+, відкривши Chloros, Chloros (браузер) або Chloros CLI і увійшовши за допомогою своїх облікових даних. Це потрібно зробити лише один раз.
{% endhint %}

### Основне використання

Обробіть папку за допомогою декількох рядків:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Повний контроль

Для складних робочих процесів:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Інструкція з встановлення

### Необхідні умови

Перед встановленням SDK переконайтеся, що у вас є:

1. **Chloros Desktop** ([завантажити](download.md))
2. **Python 3.7+** ([python.org](https://www.python.org))
3. **Активна ліцензія Chloros+** ([оновлення](https://cloud.mapir.camera/pricing))

### Встановлення за допомогою pip

**Стандартне встановлення:**

```bash
pip install chloros-sdk
```

**З підтримкою моніторингу прогресу:**

```bash
pip install chloros-sdk[progress]
```

**Встановлення для розробки:**

```bash
pip install chloros-sdk[dev]
```

### Перевірка встановлення

Перевірте, чи правильно встановлено SDK:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Перше налаштування

### Активація ліцензії

SDK використовує ту саму ліцензію, що й Chloros, Chloros (браузер) та Chloros CLI. Активуйте один раз через графічний інтерфейс або CLI:

1. Відкрийте **Chloros або Chloros (браузер)**і увійдіть на вкладку «Користувач» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Або відкрийте**CLI**.
2. Введіть свої облікові дані Chloros+ і увійдіть
3. Ліцензія зберігається в локальному кеші (зберігається після перезавантаження)

{% hint style=&quot;success&quot; %}
**Одноразова настройка**: Після входу через графічний інтерфейс або CLI, SDK автоматично використовує кешовану ліцензію. Додаткова автентифікація не потрібна!
{% endhint %}

{% hint style=&quot;info&quot; %}
**Вихід**: Користувачі SDK можуть програмно очистити кешовані облікові дані за допомогою методу `logout()`. Див. [метод logout()](#logout) у довіднику API.
{% endhint %}

### Перевірка з&#x27;єднання

Перевірте, чи SDK може підключитися до Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API Довідка

### Клас ChlorosLocal

Основний клас для локальної обробки зображень Chloros.

#### Конструктор

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Параметри:**

| Параметр                 | Тип | За замовчуванням                   | Опис                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL локального Chloros бекенду          |
| `auto_start_backend`      | bool | `True`                    | Автоматично запускати бекенд, якщо потрібно |
| `backend_exe`             | str  | `None` (автоматичне виявлення)      | Шлях до виконуваного файлу бекенда            |
| `timeout`                 | int  | `30`                      | Тайм-аут запиту в секундах            |
| `backend_startup_timeout` | int  | `60`                      | Тайм-аут для запуску бекенда (секунди) |

**Приклади:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Методи

#### `create_project(project_name, camera=None)`

Створити новий проект Chloros.

**Параметри:**

| Параметр      | Тип | Обов&#x27;язковий | Опис                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Так      | Назва проекту                                     |
| `camera`       | str  | Ні       | Шаблон камери (наприклад, «Survey3N\_RGN», «Survey3W\_OCN») |

**Повертає:** `dict` - Відповідь про створення проекту**Приклад:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Імпортувати зображення з папки.

**Параметри:**

| Параметр     | Тип     | Обов&#x27;язковий | Опис                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Так      | Шлях до папки із зображеннями         |
| `recursive`   | bool     | Ні       | Шукати в підпапках (за замовчуванням: False) |

**Повертає:** `dict` - Результати імпорту з кількістю файлів**Приклад:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Налаштування параметрів обробки.

**Параметри:**

| Параметр                 | Тип | За замовчуванням                 | Опис                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | «Висока якість (швидше)» | Метод дебайєризації                  |
| `vignette_correction`     | bool | `True`                  | Увімкнути корекцію віньєтки      |
| `reflectance_calibration` | bool | `True`                  | Увімкнути калібрування відбиття  |
| `indices`                 | список | `None`                  | Індекси рослинності для розрахунку |
| `export_format`           | str  | «TIFF (16-бітний)»         | Формат виводу                   |
| `ppk`                     | bool | `False`                 | Увімкнути корекції PPK          |
| `custom_settings`         | dict | `None`                  | Розширені налаштування        |

**Формати експорту:**

* `"TIFF (16-bit)"` - Рекомендовано для ГІС/фотограмметрії
* `"TIFF (32-bit, Percent)"` - Науковий аналіз
* `"PNG (8-bit)"` - Візуальний огляд
* `"JPG (8-bit)"` - Стислий вихід

**Доступні індекси:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 та інші.**Приклад:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Обробити зображення проекту.

**Параметри:**

| Параметр           | Тип     | За замовчуванням      | Опис                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Режим обробки: «паралельний» або «послідовний»   |
| `wait`              | bool     | `True`       | Очікування завершення                       |
| `progress_callback` | callable | `None`       | Функція зворотного виклику прогресу (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Інтервал опитування прогресу (секунди)   |

**Повертає:** `dict` - Результати обробки

{% hint style=&quot;warning&quot; %}
**Паралельний режим**: Потрібна ліцензія Chloros+. Автоматично масштабується відповідно до ядер вашого процесора (до 16 робочих процесів).
{% endhint %}

**Приклад:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Отримати поточну конфігурацію проекту.

**Повертає:** `dict` - Поточна конфігурація проекту**Приклад:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Отримати інформацію про стан бекенду.

**Повертає:** `dict` - Стан бекенду**Приклад:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Вимкнення бекенду (якщо він запущений SDK).

**Приклад:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Очистити кешовані облікові дані з локальної системи.

**Опис:**

Програмно виходить із системи, видаляючи кешовані облікові дані. Це корисно для:
* Перемикання між різними обліковими записами Chloros+
* Очищення облікових даних в автоматизованих середовищах
* Цілі безпеки (наприклад, видалення облікових даних перед видаленням програми)

**Повертає:** `dict` - Результат операції виходу з системи**Приклад:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style=&quot;info&quot; %}
**Необхідна повторна аутентифікація**: Після виклику `logout()` необхідно знову увійти за допомогою Chloros, Chloros (браузер) або Chloros CLI перед використанням SDK.
{% endhint %}

***

### Зручні функції

#### `process_folder(folder_path, **options)`

Однорядкова зручна функція для обробки папки.

**Параметри:**

| Параметр                 | Тип     | За замовчуванням         | Опис                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Обов&#x27;язковий        | Шлях до папки з зображеннями     |
| `project_name`            | str      | Автоматично згенерований  | Назва проекту                   |
| `camera`                  | str      | `None`          | Шаблон камери                |
| `indices`                 | list     | `["NDVI"]`      | Індекси для обчислення           |
| `vignette_correction`     | bool     | `True`          | Увімкнути корекцію віньєтування     |
| `reflectance_calibration` | bool     | `True`          | Увімкнути калібрування відбиття |
| `export_format`           | str      | &quot;TIFF (16-біт)&quot; | Формат виводу                  |
| `mode`                    | str      | `"parallel"`    | Режим обробки                |
| `progress_callback`       | callable | `None`          | Зворотний виклик прогресу              |

**Повертає:** `dict` - Результати обробки**Приклад:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Підтримка контекстного менеджера

SDK підтримує контекстні менеджери для автоматичного очищення:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Повні приклади

### Приклад 1: Основна обробка

Обробка папки з типовими налаштуваннями:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Приклад 2: Налаштований робочий процес

Повний контроль над процесом обробки:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Приклад 3: Пакетна обробка декількох папок

Обробка декількох наборів даних про польоти:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Приклад 4: Інтеграція дослідницького конвеєра

Інтеграція Chloros з аналізом даних:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Приклад 5: Налаштований моніторинг прогресу

Розширене відстеження прогресу з реєстрацією:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Приклад 6: Обробка помилок

Надійна обробка помилок для виробничого використання:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Приклад 7: Управління обліковим записом і вихід із системи

Управління обліковими даними за допомогою програм:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Приклад 8: Інструмент командного рядка

Створення власного інструменту CLI за допомогою SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Використання:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Обробка винятків

SDK надає специфічні класи винятків для різних типів помилок:

### Ієрархія винятків

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Приклади винятків

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Розширені теми

### Налаштування власного бекенду

Використовуйте власне розташування або конфігурацію бекенду:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Неблокуюча обробка

Почніть обробку та продовжуйте виконувати інші завдання:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Управління пам&#x27;яттю

Для великих наборів даних обробляйте їх пакетами:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Усунення несправностей

### Бекенд не запускається

**Проблема:** SDK не вдається запустити бекенд.**Рішення:**

1. Перевірте, чи встановлено Chloros Desktop:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Перевірте, чи брандмауер Windows не блокує
3. Спробуйте вручну вказати шлях до бекенду:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Ліцензія не виявлена**Проблема:** SDK попереджає про відсутність ліцензії**Рішення:**

1. Відкрийте Chloros, Chloros (браузер) або Chloros CLI і увійдіть.
2. Перевірте, чи ліцензія збережена в кеші:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Якщо виникають проблеми з обліковими даними, очистіть кеш облікових даних і увійдіть знову:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Зверніться до служби підтримки: info@mapir.camera

***

### Помилки імпорту**Проблема:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Рішення:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Час очікування обробки**Проблема:** Час очікування обробки закінчився**Рішення:**

1. Збільште час очікування:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Обробляйте менші партії
3. Перевірте доступний простір на диску
4. Контролюйте системні ресурси

***

### Порт вже використовується**Проблема:** Порт 5000 бекенду зайнятий**Рішення:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Або знайдіть і закрийте конфліктний процес:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Поради щодо продуктивності

### Оптимізація швидкості обробки

1. **Використовуйте паралельний режим** (потрібен Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Зменшіть роздільну здатність виводу** (якщо це прийнятно)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Вимкніть непотрібні індекси**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Обробляйте на SSD** (а не на HDD)***

### Оптимізація пам&#x27;яті

Для великих наборів даних:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Фонове оброблення

Звільніть Python для інших завдань:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Приклади інтеграції

### Інтеграція з Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## FAQ

### Питання: Чи потрібно для SDK підключення до Інтернету?

**В:** Тільки для початкової активації ліцензії. Після входу через Chloros, Chloros (браузер) або Chloros CLI ліцензія кешується локально і працює в автономному режимі протягом 30 днів.***

### Питання: Чи можу я використовувати SDK на сервері без графічного інтерфейсу?**Відповідь:** Так! Вимоги:

* Windows Server 2016 або пізніша версія
* Chloros встановлено (одноразово)
* Ліцензія активована на будь-якому комп&#x27;ютері (кешована ліцензія скопійована на сервер)

***

### Питання: У чому різниця між Desktop, CLI та SDK?

| Функція         | Desktop GUI | CLI Командний рядок | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Інтерфейс**   | Покажчик миші | Команда          | Python API  |
| **Найкраще підходить для**    | Візуальної роботи | Скриптів        | Інтеграції |
| **Автоматизація**  | Обмежена     | Хороша             | Відмінна   |
| **Гнучкість** | Базова       | Хороша             | Максимальна     |
| **Ліцензія**     | Chloros+    | Chloros+         | Chloros+    |***

### Питання: Чи можу я розповсюджувати програми, створені за допомогою SDK?**Відповідь:** Код SDK можна інтегрувати у ваші програми, але:

* Кінцевим користувачам потрібно встановити Chloros
* Кінцевим користувачам потрібні активні ліцензії Chloros+
* Для комерційного розповсюдження потрібна ліцензія OEM.

Зверніться до info@mapir.camera з питаннями щодо OEM.

***

### Питання: Як оновити SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### Питання: Де зберігаються оброблені зображення?

За замовчуванням, у шляху до проекту:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Питання: Чи можна обробляти зображення за допомогою скриптів Python, що виконуються за розкладом?**Відповідь:** Так! Використовуйте планувальник завдань Windows із скриптами Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Заплануйте щоденне виконання за допомогою планувальника завдань.

***

### Питання: Чи підтримує SDK асинхронний/очікуючий режим?**Відповідь:** Поточна версія є синхронною. Для асинхронної роботи використовуйте `wait=False` або запустіть в окремому потоці:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### Питання: Як перемикатися між різними обліковими записами Chloros+?**Відповідь:** Використовуйте метод `logout()`, щоб очистити кешовані облікові дані, а потім знову увійдіть за допомогою нового облікового запису:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Після виходу з облікового запису пройдіть аутентифікацію за допомогою нового облікового запису через графічний інтерфейс, браузер або CLI, перш ніж знову використовувати SDK.

***

## Отримання допомоги

### Документація

* **Довідка API**: Ця сторінка

### Канали підтримки

* **Електронна пошта**: info@mapir.camera
* **Веб-сайт**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ціни**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Приклад коду

Усі наведені тут приклади перевірені та готові до використання. Скопіюйте їх та адаптуйте до своїх потреб.

***

## Ліцензія**Пропрієтарне програмне забезпечення** - Copyright (c) 2025 MAPIR Inc.

SDK вимагає активної підписки на Chloros+. Несанкціоноване використання, розповсюдження або модифікація заборонені.
