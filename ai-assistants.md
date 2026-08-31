# Використання Chloros разом з штучним інтелектом

Цей посібник призначений для двох аудиторій: людей та штучного інтелекту, з яким люди дедалі частіше працюють. На кожній сторінці наведено точні значення, значення за замовчуванням та команди, які можна скопіювати й вставити, щоб помічник (Claude, ChatGPT, Copilot, агент для програмування тощо) міг з першої спроби написати працездатну автоматизацію Chloros.

Версія Chloros: **

1.2.0**. Платформи CLI/SDK: Windows 10/11 x64 та Linux (x86_64 / Jetson aarch64).

## Що передати вашому помічнику

| Ресурс | URL | Для чого він призначений |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Індекс кожної сторінки цього посібника у форматі, придатному для машинного зчитування. |
| **CLI Довідка** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Повний набір команд `chloros-cli`: кожна команда, прапор, значення за замовчуванням, код завершення та правило щодо папки виводу. Написано для використання великими мовними моделями (LLM). |
| **SDK Довідник** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Повний опис `chloros_sdk` Python API: класи, сигнатури, винятки та практичні приклади. Написано для використання в LLM. |
| **Будь-яка сторінка у вигляді необробленого Markdown** | додайте `.md` до сторінки URL | наприклад `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` повертає сторінку у вигляді необробленого Markdown — ідеально підходить для вставлення у вікно контексту або отримання від агента. |

Посилання в посібнику: [CLI Довідка](reference/cli-reference.md) · [SDK Довідка](reference/sdk-reference.md).

{% hint style="info" %}
Ці дві довідкові сторінки є самодостатніми: помічник, який прочитав одну з них, не потребує решти посібника, щоб написати правильний скрипт.
{% endhint %}

## Готові рецепти

Скопіюйте, заповніть `<placeholders>` та вставте у свій асистент.

### 1. Обробка папки з даними польоту в NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Пакетний моніторинг каталогу записів

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Підключення масиву LATTICE та запис

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Запис спектрів світлового датчика DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Створення скриптів для DAQ з командного рядка завжди здійснюється через сімейство `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Інші підкоманди `daq`, які може вигадати ваш помічник, недоступні в офіційних збірках і призводять до завершення роботи з помилкою.
{% endhint %}

## Чому скрипти, написані ШІ, добре працюють із Chloros

Кожен із цих випадків є реальним, перевіреним проявом поведінки Chloros 1.2.0 — вони усувають класичні причини збою автоматизації, написаної машиною:

* **Відсутність складних налаштувань.**Інтелектуальні допоміжні функції SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) та точки входу обробки (`ChlorosLocal`, `process_folder`)**автоматично запускають локальний бекенд**. Згенерованому скрипту не потрібно, щоб графічний інтерфейс був відкритий або сервер запускався вручну — йому потрібно лише, щоб на робочому столі був встановлений пакет CLI.
* **Весь конвеєр — це один виклик.** `chloros_sdk.process_folder("path", indices=["NDVI"])` виконує імпорт → калібрування → визначення відбивної здатності → експорт індексу від початку до кінця. Менша площа поверхні — менше місць, де згенерований скрипт може дати збій.
* **Запуски без вихідних даних мають функцію самодіагностики.** Після завершення роботи `process()` до результату додається звіт про запуск, а кожна підказка щодо обробки (наприклад, *чому* запуск не дав результату) також повторно надсилається як Python `UserWarning` — тож навіть скрипт, який ніколи не перевіряє словник результатів, відображає діагностику.
* **CLI завершується з гучною помилкою.**Запуск `chloros-cli process`, який запитував результати, але не вивів жодних, виводить `Processing finished but wrote no image products.` і**завершується з ненульовим кодом**, тому скрипти оболонки та CI виявляють це за допомогою простої перевірки коду завершення. Успішні запуски повідомляють про `Image products written: N`.

Одна асиметрія, про яку повинен знати асистент: `process()` у SDK навмисно **не** генерує помилку під час запуску без продуктів — замість цього він повідомляє про це через підсумок/підказки. Якщо конвеєр Python повинен зупинитися у разі порожнього запуску, перевірте підсумок (рецепт 2 це робить).

## Застереження

* **Chloros+ — необхідний вхід у систему.**CLI та SDK вимагають**платного** рівень Chloros+, що контролюється на стороні сервера: запити завершуються з помилкою `401 AUTH_REQUIRED`, якщо ви не ввійшли в систему, та `403 PLAN_UPGRADE_REQUIRED` на безкоштовному рівні. Перед запуском згенерованих скриптів запустіть `chloros-cli login` один раз на кожній машині. Див. [Chloros+ Вхід](chloros+-login.md).
* **Команди запису керують реальним обладнанням.** Команди `lattice` / `daq` / `project` та об’єкти сеансу SDK забезпечують підключення, передачу потокового відео та запуск фізичних камер і датчиків. Перегляньте згенерований скрипт перед його першим запуском і запустіть його в присутності оператора обладнання.
* **Виконайте вибіркову перевірку вихідних даних.** Перед публікацією результатів перевірте папки з даними та кілька значень пікселів. Зокрема, TIFF-файли відбиття масштабуються залежно від джерела — ознайомтеся з тегом XMP `Chloros:PixelScale` (LATTICE: 32768 = відбиваність 1,0; Survey3: 65535) замість того, щоб припускати ділитель. Обидві довідкові сторінки описують це в розділі «Зчитування пікселів відбиваності».
* **Невеликі підводні камені, що заважають генеруванню коду:**`pool-record` записує у файлову систему**хоста бекенду** (за замовчуванням — `~/Documents/DAQ Live View/`); на машинах з кількома мережевими інтерфейсами краще використовувати `daq pool-connect --eth-host <ip-or-hostname>` замість автоматичного виявлення; та використовуйте `http://127.0.0.1:5000` (ніколи `localhost`) скрізь, де з’являється бекенд URL.
