---
description: Lab-measured panels used to calibrate captured data in post-processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# Калібрувальні мішені

MAPIR пропонує різні калібрувальні мішені для широкого спектру застосувань. Компактна модель T4-R50, зображена нижче, містить 4 панелі, які були виміряні на світловідбивання в діапазоні від 250 до 2500 нм.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Дифузні еталонні мішені T4 мають такі криві відбиття, [завантажити дані тут](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Відбивання :: 250-2500 нм</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Відбивання :: 400-1000 нм</p></figcaption></figure>На графіку відбивної здатності ви можете побачити, що значення є довжиною хвилі (ось x) проти відсотка відбивної здатності (ось y). Коли ми знімаємо зображення калібрувальної мішені, ми створюємо взаємозв&#x27;язок між значенням пікселя та відсотком відбивної здатності в межах спектра, до якого чутливі кожна з смуг сенсора камери.

Це означає, що з кожним зображенням, знятим нашими камерами, ви можете використовувати фотографію наших мішеней відбиття, таких як [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) або [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), для калібрування зображень за відбиттям. Після калібрування кожен піксель зображення дорівнює відсотку відбиття.

Якщо ви виводите відкалібровані зображення в Chloros як типові JPG або TIFF, то відсоток відбиття обчислюється шляхом ділення значення пікселя на бітову глибину формату зображення. Отже, для JPG діліть на 255, а для TIFF діліть на 65 535. Ви також можете вибрати вихідний формат PERCENT у Chloros, і тоді кожен піксель буде мати значення від 0,0 до 1,0 (від 0% до 100% відбиття). Просто майте на увазі, що деякі програми для роботи з зображеннями не підтримують зображення у відсотках (з плаваючою комою), а також що вони займають багато місця на диску.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
