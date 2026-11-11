# Використання Chloros з AI-помічниками

Цей посібник призначений для двох аудиторій: людей та штучного інтелекту, з яким люди все частіше працюють. На кожній сторінці наведено точні значення, значення за замовчуванням та команди, які можна скопіювати й вставити, щоб помічник (Claude, ChatGPT, Copilot, агент кодування тощо) міг з першого разу написати робочу автоматизацію Chloros.

Версія Chloros: **

1.3.0**. Платформи CLI/SDK: Windows 10/11 x64 та Linux (x86\_64 / Jetson aarch64).

## Що надати вашому помічнику

| Ресурс | URL | Для чого |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Індекс усіх сторінок цього посібника у форматі, придатному для машинного зчитування. |
| **Довідник CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Повний набір команд `chloros-cli`: кожна команда, прапор, значення за замовчуванням, код завершення та правило щодо папки виводу. Написано для використання LLM. |
| **Довідка щодо SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Повний Python-API `chloros_sdk`: класи, сигнатури, винятки та приклади з роз’ясненнями. Написано для використання LLM. |
| **Будь-яка сторінка у вигляді необробленого Markdown** | додайте `.md` до URL-адреси сторінки | наприклад, `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` повертає сторінку у вигляді необробленого Markdown — ідеально підходить для вставлення у вікно контексту або отримання від агента. |

Посилання в посібнику: [Довідка щодо CLI](reference/cli-reference.md) · [Довідка щодо SDK](reference/sdk-reference.md).

{% hint style="info" %}
Ці дві довідкові сторінки є самодостатніми: помічник, який прочитав одну з них, не потребує решти посібника, щоб написати правильний скрипт.
{% endhint %}

## Дозвольте вашому асистенту запускати Chloros безпосередньо: `chloros-mcp`

Усе, що описано вище, передбачає, що ваш асистент *напише* скрипт або команду. За допомогою `chloros-mcp` асистент, який підтримує [Model Context Protocol](https://modelcontextprotocol.io) (MCP), наприклад Claude Desktop, Claude Code або Cursor, керує самою програмою Chloros. Ви задаєте запит простою мовою, і він викликає інструменти сервера:

* **Обробка зображень:** створення проєкту, імпорт папки з знімками, вибір налаштувань, обробка та звіт про результати, експортовані після запуску.
* **Управління апаратним забезпеченням:** підключення камер LATTICE, масивів та світлових датчиків DAQ, попередній перегляд, налаштування експозиції та кольору, зйомка, запис та оновлення прошивки.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Інструменти сервера охоплюють функції [Python SDK](api-python-sdk.md). Вони встановлюються разом із SDK, тому зазвичай нічого додаткового встановлювати не потрібно: у Windows інсталятор Chloros розміщує `chloros-mcp.exe` у папці `Scripts` вашого Python, а в Linux `.deb` розміщує файл `chloros-mcp` у папці `/usr/bin`.

**Код Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**додайте це до `claude_desktop_config.json` (відкрити можна через**Налаштування > Розробник > Редагувати конфігурацію**), а потім перезапустіть Claude Desktop.**Курсор** використовує той самий запис у `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Якщо клієнт повідомляє, що сервер не запустився, ймовірно, `chloros-mcp` відсутній у списку `PATH`, який він бачить: замість цього вкажіть повний шлях (`where chloros-mcp` у Windows, `which chloros-mcp` у Linux).

На тому самому комп’ютері має бути встановлено Chloros, і ви повинні увійти в систему там, у додатку або за допомогою `chloros-cli login`, на будь-якому платному рівні Chloros+ (Copper або вище). Сервер не може увійти в систему за вас.

{% hint style="info" %}
Налаштування клієнта для кожного додатка, параметри, кожен інструмент та поведінка під час виконання: [Сервер MCP (AI-помічники)](reference/sdk-reference.md#mcp-server-ai-assistants) у довіднику SDK.
{% endhint %}

## Приклади команд

Скопіюйте, заповніть `<placeholders>` та вставте у свій асистент.

### 1. Обробка папки з даними польоту у формат NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Пакетний моніторинг каталогу знімків

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Підключення масиву LATTICE та зйомка

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Запис спектрів світлових датчиків DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Створення скриптів для DAQ з командного рядка завжди здійснюється за допомогою сімейства команд `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Інші підкоманди `daq`, які може вигадати ваш помічник, недоступні в офіційних збірках і призводять до завершення роботи з помилкою.
{% endhint %}

## Чому скрипти, написані ШІ, добре працюють з Chloros

Кожен із цих прикладів є реальним, перевіреним поведінкою Chloros 1.3.0 — вони усувають класичні причини збою автоматизації, написаної машиною:

* **Відсутність складних налаштувань.**Допоміжні функції SDK для інтелектуального підключення (`connect_camera`, `connect_array`, `connect_daq_sensor`) та точки входу обробки (`ChlorosLocal`, `process_folder`)**автоматично запускають локальний бекенд**. Згенерованому скрипту не потрібно, щоб графічний інтерфейс був відкритий або сервер запускався вручну — достатньо лише встановити пакет для робочого столу або CLI.
* **Весь конвеєр — це один виклик.** `chloros_sdk.process_folder("path", indices=["NDVI"])` виконує імпорт → калібрування → вимірювання відбиття → експорт індексу від початку до кінця. Менша площа поверхні, менше місць, де згенерований скрипт може дати збій.
* **Запуски без вихідних даних проходять самодіагностику.** Після завершення роботи `process()` до результату додається звіт про запуск, а кожна підказка щодо обробки (наприклад, *чому* запуск не дав результату) також повторно виводиться у вигляді Python-скрипта `UserWarning` — тож навіть скрипт, який ніколи не перевіряє словник результатів, відображає діагностику.
* **CLI видає гучну помилку.**Запуск `chloros-cli process`, який запитував результати, але не записав жодного, виводить `Processing finished but wrote no image products.` і**завершується з ненульовим кодом**, тому скрипти оболонки та CI виявляють це за допомогою простої перевірки коду завершення. Успішні запуски повідомляють про `Image products written: N`, а також про `(plus M clamp masks)`, якщо були записані маски. Команди запису також вказують імена всіх файлів, які не вдалося записати, і завершують роботу з кодом `1`.
* **Відносні шляхи означають те, чого очікує скрипт.** У разі локального бекенду відносний шлях до виводу (`-o out/` у CLI, `output_dir="out/"` у SDK) визначається відносно папки, з якої запускається команда або скрипт.

Одна асиметрія, про яку повинен знати асистент: `process()` у SDK навмисно **не** генерує виняток під час запуску з нульовим результатом — замість цього він повідомляє про це через підсумок/підказки. Якщо конвеєр Python повинен зупинитися під час порожнього запуску, перевірте підсумок (як це робить рецепт 2).

## Застереження

* **Потрібен Chloros+ login.**CLI та SDK вимагають**платного** тарифного плану Chloros+, що контролюється на стороні сервера: запити завершуються з помилкою `401 AUTH_REQUIRED`, якщо ви не ввійшли в систему, та `403 PLAN_UPGRADE_REQUIRED` на безкоштовному тарифному плані. Перед запуском згенерованих скриптів виконайте команду `chloros-cli login` один раз на кожній машині. Див. [Вхід у Chloros+](chloros+-login.md).
* **Команди захоплення керують реальним обладнанням.** Команди `lattice` / `daq` / `project`, об’єкти сеансів SDK та інструменти `chloros-mcp` забезпечують підключення, передачу потокового відео та запуск фізичних камер і датчиків. Перегляньте згенерований скрипт перед його першим запуском і запускайте його в присутності оператора обладнання.
* **Виконуйте вибіркову перевірку вихідних даних.** Перед публікацією результатів перевірте папки з даними та кілька значень пікселів. Зокрема, TIFF-файли відбиття масштабуються залежно від джерела — ознайомтеся з тегом XMP `Chloros:PixelScale` (файли LATTICE містять його: 32768 = 1,0 відбиття; файли Survey3 його не містять і використовують 65535 = 1,0) замість того, щоб припускати наявність дільника. Обидві довідкові сторінки описують це в розділі «Зчитування пікселів відбиття».
* **Невеликі підводні камені, що заважають генеруванню коду:**`pool-record` записує у файлову систему**хоста-бекенду** (за замовчуванням — `~/Documents/DAQ Live View/`); на машинах із кількома мережевими інтерфейсами краще використовувати `daq pool-connect --eth-host <ip-or-hostname>`, а не автоматичне виявлення; використовуйте `http://127.0.0.1:5000` (ніколи `localhost`) скрізь, де з’являється URL-адреса сервера; скрипт, що обходить дерево вихідних даних проєкту, повинен пропускати папки `clamp_masks/` (їхні файли `*_clampmask.tif` є масками, а не зображеннями — див. [Формати вихідних зображень](output-image-formats.md#clamp-masks)); а в Windows, де вимкнено довгі шляхи, папки проєктів мають бути короткими — папка проєкту, довжина шляху до якої перевищує приблизно 130 символів, відхиляється ще до початку обробки.
