---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Калібрувальні мішені

MAPIR пропонує різноманітні калібрувальні мішені для широкого спектра застосувань. Компактна модель T4-R50, зображена нижче, містить 4 панелі, для яких виміряно коефіцієнт відбиття світла в діапазоні від 250 до 2500 нм.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Дифузні еталони T4 мають такі криві відбиття: [завантажити дані тут](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Коефіцієнт відбиття T4 :: 250–2500 нм</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Відбивна здатність T4 :: 400–1000 нм</p></figcaption></figure>Дифузні еталонні мішені T4P мають такі криві відбиття, [завантажити дані тут](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P Відбивна здатність :: 250-2500 нм</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Коефіцієнт відбиття T4P :: 400–1000 нм</p></figcaption></figure>Якщо подивитися на графік відбиття, можна побачити, що значення представлені як довжина хвилі (вісь x) проти відсотка відбиття (вісь y). Коли ми знімаємо зображення калібрувальної мішені, ми створюємо взаємозв&#x27;язок між значенням пікселя та відсотком відбиття в межах спектра, до якого чутливі кожні діапазони датчика камери.

Це означає, що для кожного зображення, знятого нашими камерами, ви можете використовувати фотографію наших мішеней відбиття, таких як [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) або [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), щоб відкалібрувати зображення за відбиттям. Після калібрування кожен піксель зображення відповідає відсотку відбиття.

Якщо ви виводите відкалібровані зображення у форматі Chloros як звичайний JPG або TIFF, то відсоток відбиття обчислюється шляхом ділення значення пікселя на розрядність формату зображення. Отже, для JPG діліть на 255, а для TIFF — на 65 535. Ви також можете вибрати формат виводу PERCENT у Chloros, і тоді кожен піксель буде мати значення від 0,0 до 1,0 (від 0% до 100% відбиття). Просто майте на увазі, що деякі програми для роботи із зображеннями не підтримують зображення у форматі відсотків (з плаваючою комою), а також що вони займають багато місця на диску.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
