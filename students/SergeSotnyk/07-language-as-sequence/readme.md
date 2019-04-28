# Мова як послідовність

## Класифікатор
Дані:

- Виберіть будь-який відкритий корпус та згенеруйте тренувальні дані для моделі. 
Тренувальними даними буде набір склеєних речень. Візьміть до уваги, що склеєних 
речень може бути кілька (зазвичай 2, але буває і 3-4), а перше слово наступного 
речення може писатися з великої чи малої літери.
- Зберіть (чи знайдіть у відкритому доступі) базу енграмів. Візьміть до уваги, 
що відкриті бази енграмів зазвичай містять статистику, зібрану на реченнях, а 
отже вони можуть не містити енграми на межі речень.

Тестування:

- Напишіть бейзлайн та метрику для тестування якості.
- Для тестування використайте корпус 
[run-on-test.json](https://github.com/vseloved/prj-nlp-2019/blob/master/tasks/07-language-as-sequence/run-on-test.json). 

Класифікатор:

- Виділіть ознаки, які впливають на те, чи є слово на межі речень. Подумайте про 
правий/лівий контекст, написання слова, граматичні ознаки (чи може речення 
закінчитись на сполучник?), енграми (чи часто це слово і наступне йдуть поруч?), 
складники та залежності тощо.
- Побудуйте класифікатор на основі логістичної регресії з використанням виділених 
ознак, який анотує послідовно слова у реченні на предмет закінчення речення.
- Спробуйте покращити якість роботи класифікатора, змінюючи набір чи комбінацію ознак.

**Важливо**: оційнюйте якість свого класифікатора в першу чергу на своїх даних 
(train/test або кросвалідація), щоб не підігнати ознаки під надану тестову вибірку.

Запишіть ваші спостереження та результати в окремий файл.

## Хід рішення
### Перевіряємо статистичні характеристики наданого корпусу run-on-test.json

Запускаємо скрипт rot.py:

```
C:\Users\ssotn\Anaconda3\envs\nlp\python.exe D:/git-nlp/ss-prj-nlp-2019/students/SergeSotnyk/07-language-as-sequence/rot.py
Real sentences: 355
Missed ends: 155
Started from lowercase: 80
Total sentences: 200
``` 

### Робимо собі аналогічний корпус для тренування

Один мій однокурсник поділився архівом статей NYT за 2000й рік. Файл NY-Times-2000.zip 
занадто великий для того, щоб я його залив його тут (приблизно 600 МБ), але могу надати 
його особисто.

Утілита **process_nyt2000** вибирає з цього корпусу речення, які закінчуються на крапку 
(під капотом іде очистка корпуса від зайвого тексту за допомогою readability, 
конвертація в текст з якою не всі конвертори добре справляються, та деякі 
ручні маніпуляції для остаточної очистки). На виході маємо файл jsonl (цей формат обрано)
через те, що з json розміром більше гігабайта досить незручно робити), у якому кожна
строка - це набір токенів речення.

```
C:\Users\ssotn\Anaconda3\envs\nlp\python.exe D:/git-nlp/ss-prj-nlp-2019/students/SergeSotnyk/07-language-as-sequence/process_nyt2000.py
Load spacy...
...Done!
100%|██████████| 109990/109990 [1:16:38<00:00, 26.04it/s]
Total proper sentences: 2230373
Total articles: 109990
```

Наступна утілита **make_train_dev_ny2000.py** робить з эталонних речень 
run-on-sentences з приблизно такими ж статистичними особливостями, як і в тестовому
наборі, та розділяє їх на 2 сети - тренувальний, та валідаційний, на якому ми
будемо підбирати налаштування алгоритму. 

```
C:\Users\ssotn\Anaconda3\envs\nlp\python.exe D:/git-nlp/ss-prj-nlp-2019/students/SergeSotnyk/07-language-as-sequence/make_train_dev_ny2000.py
100%|██████████| 6049450/6049450 [01:57<00:00, 51413.51it/s]
Start reading file "data/nyt2000-sents.train.jsonl"
Has read 1320752 lines, shuffling in memory
Start writing back"
Done!
Start reading file "data/nyt2000-sents.dev.jsonl"
Has read 1321908 lines, shuffling in memory
Start writing back"
Done!
```

Тепер, створюємо статистику колокацій (біграм) по нашому корпусу. В цей словник
зберігаємо біграми, які знаходяться лише в межах одного речення. Для того, щоб 
класифікатор не занадто опирався на цей признак, видаляємо зі словника ті біграми,
котрі зустрічаються лише один раз. Для цього, запускаємо утілиту **prepare_colocations**:

```
C:\Users\ssotn\Anaconda3\envs\nlp\python.exe D:/git-nlp/ss-prj-nlp-2019/students/SergeSotnyk/07-language-as-sequence/prepare_colocations.py
100%|██████████| 2230373/2230373 [00:47<00:00, 46892.79it/s]
Collected 2150888
Stored
```

Тепер, все підготувавши, можемо классифікувати. Цей процес можна подивитися у 
ноутбуці [train_classify.ipynb](train_classify.ipynb). Після багатьох експериментів,
маю такі результати:

### Dev set:
```
              precision    recall  f1-score   support

       False      0.992     0.996     0.994   2639384
        True      0.795     0.693     0.740     65029

   micro avg      0.988     0.988     0.988   2704413
   macro avg      0.894     0.844     0.867   2704413
weighted avg      0.988     0.988     0.988   2704413
```

### Test set:
```
              precision    recall  f1-score   support

       False      0.992     0.988     0.990      4542
        True      0.676     0.755     0.713       155

   micro avg      0.980     0.980     0.980      4697
   macro avg      0.834     0.871     0.852      4697
weighted avg      0.981     0.980     0.981      4697
```

## Що можна поліпшити
0. Додати ще фічі. Але то вже треба добре подумати, що ще додати. Не думаю, що цей
шлях може покращити дуже сильно.
0. Застосувати якийсь нелінійний классифікатор - ті ж самі нейронні мережі. Також,
нейромережі можуть самі собі придумати нові фічи, до яких я не дійшов.
0. Опробувати стекінг, беггінг, бустінг - це те, що я ще не пробував.
0. Створити треніровочний набір на іншому корпусі з іншими статистичними 
властивостями.
0. Застосувати інший словник біграм слів, додати туди якісь плейсхолдери.