---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Калібрувальні мішені

MAPIR пропонує різні калібрувальні мішені для широкого спектра застосувань. Компактна модель T4-R50, зображена нижче, містить 4 панелі, для яких було виміряно світловідбивання в діапазоні від 250 до 2 500 нм.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Дифузні еталони T4 мають такі криві відбивання: [завантажити дані тут](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Коефіцієнт відбиття T4 :: 250–2500 нм</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Коефіцієнт відбиття T4 :: 400–1000 нм</p></figcaption></figure>Дифузні еталонні мішені T4P мають такі криві відбиття, [завантажити дані тут](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Коефіцієнт відбиття T4P :: 250–2500 нм</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P Коефіцієнт відбиття :: 400–1000 нм</p></figcaption></figure>Якщо подивитися на графік відбиття, можна побачити, що значення представлені у вигляді залежності довжини хвилі (вісь x) від відсотка відбиття (вісь y). Коли ми знімаємо зображення калібрувальної мішені, ми встановлюємо взаємозв’язок між значенням пікселя та відсотком відбиття в межах спектра, до якого чутливі окремі діапазони датчика камери.

Це означає, що для кожного зображення, знятого нашими камерами, ви можете використовувати фотографію наших еталонів відбиття, таких як [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) або [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), для калібрування зображень за коефіцієнтом відбиття. Після калібрування кожен піксель зображення відповідає відсотку відбиття.

Для **Survey3** , якщо ви експортуєте відкалібровані зображення у форматі Chloros як звичайні JPG або TIFF, то відсоток відбиття обчислюється шляхом ділення значення пікселя на глибину кольору формату зображення. Отже, для JPG слід ділити на 255, а для TIFF — на 65 535. Ви також можете вибрати формат виводу PERCENT у Chloros, і тоді кожен піксель матиме значення у відсотках від 0,0 до 1,0 (відбивання від 0% до 100%). Просто майте на увазі, що деякі програми для роботи із зображеннями не підтримують зображення у відсотках (з плаваючою комою), а також що такі файли займають багато місця при зберіганні.

{% hint style="info" %}
**Коефіцієнт відбиття LATTICE використовує інший масштаб пікселів.** Коефіцієнт відбиття LATTICE зберігається з DN 32768 = 100% відбиття (а не 65535), і кожен файл містить тег XMP `Chloros:PixelScale`, що вказує його масштаб. Прочитайте тег і розділіть значення на нього, замість того щоб припускати постійну величину — див. [Формати вихідних зображень](output-image-formats.md).
{% endhint %}

## Калібрувальні мішені для камер LATTICE

З камерами LATTICE калібрувальна мітка для коефіцієнта відбиття є **необов’язковою**: Chloros може замість цього пов’язувати коефіцієнт відбиття з інтенсивністю опромінення, що падає вниз, виміряною світловим датчиком DAQ (ρ = π·L/E). Орієнтир обирається за допомогою налаштування джерела відбиття (Налаштування проекту в графічному інтерфейсі; `--reflectance-source` у CLI; `reflectance_source` у SDK):

| Значення | Поведінка |
| --- | --- |
| `auto` *(за замовчуванням)* | Ціль у кадрі, що пройшла перевірку якості (QA), є **абсолютним еталоном**; якщо цілі немає або перевірка якості не пройшла, Chloros переходить на використання коефіцієнта поділу DAQ для низхідного потоку. |
| `target` | Тільки ціль — без заміни даними DAQ. |
| `daq` | Авторитет даних DAQ — вимірювання вниз по потоку завжди є еталоном. |

Додаткові параметри поведінки мішеней для LATTICE:

* **Геометрія мішеней** — підтримуються панелі з маркуванням ArUco, панелі з фіксованою областю інтересу (ROI) та смугові мішені; геометрія визначається конфігурацією мішеней у проєкті.
* **Дані вимірювань мішеней за одиницями** — `--target-reflectance-dir DIR` вказує на каталог сканованих даних відбиття мішеней, виміряних за одиницями (`<serial>.csv`, пошук за серійним номером/QR-кодом одиниці мішені). У разі промаху Chloros використовує номінальні спектри T3/T4P.
* **Часова фіксація** — виявлена мішень калібрує кадри навколо себе та утримується між спостереженнями мішені.

Повна семантика прапорців та приклади наведені в [Довідці щодо CLI](reference/cli-reference.md) (див. «Перемикачі експорту для окремих продуктів»).

### F988

«Відбивна здатність F988 калібрується за допомогою панелі відбивної здатності в кадрі: діапазон лежить за межами каліброваного діапазону світлового датчика DAQ, тому Chloros застосовує ваші останні дані зйомки панелі та зберігає їх між спостереженнями панелі».

Якщо F988 запускається з калібруванням лише за допомогою DAQ, Chloros відхиляє значення відбивної здатності, отримані за допомогою DAQ для цього діапазону, і вказує причину (причина пропуску `dls-uncalibrated-band-988`); робочий процес із використанням панелі є підтримуваним варіантом.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
