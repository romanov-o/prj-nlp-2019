# Курсовий проект
Для свого курсового проекту визначте остаточні метрики і напишіть програму, яка їх 
реалізує. Покажіть приклад роботи програми на іграшкових даних (до 10 прикладів 
реальних чи штучних даних).

# Метрики
В моєму проекті є дві речі, якість яких треба оцінювати: 
- знайдений список ключових слів
- речення автоматично сконструйованної аннотації

Судячи з https://arxiv.org/pdf/1801.04470.pdf (Simple Unsupervised Keyphrase 
Extraction using Sentence Embeddings), якість списку ключових слів вимірюють за 
допомогою скорів 

- Precision
- Recall
- F1

Для оцінки якості аннотацій, використовують ті ж самі метрики, що і для оцінки
якості перекладів:

- BLEU (https://en.wikipedia.org/wiki/BLEU)
- ROUGE (<https://en.wikipedia.org/wiki/ROUGE_(metric)>)
- METEOR (https://en.wikipedia.org/wiki/METEOR)

Є ще декілька 
([CIDEr: Consensus-based Image Description Evaluation](http://arxiv.org/pdf/1411.5726.pdf),
[SPICE: Semantic Propositional Image Caption Evaluation](https://arxiv.org/abs/1607.08822)) які менш поширені і які я поки не хочу 
застосовувати у моєму проекті.

METEOR здається найбільш адекватною метрикою через те, що вона дозволяє використовувати 
не тільки ті ж самі слова, але і їх синоніми (для їх пошуку використовується 
WordNet). Але через це ж метрика є мовозалежною і більшість реалізацій опрацьовують
лише англійську. Звичайно, її можна доробити майже для всякої росповсюдженої в Wiktionary 
мови, але поки (на першому етапі) я вирішив для неанглійських текстів використовувати 
BLEU та ROUGE-N.

## Code

Задля підрахування метрик написаний код, що знаходиться в пакеті keysum_evaluator. 
Там є модулі для розрахунку як prec, recall, f1 для ключових слів, так і для 
розрахунку метрик BLEU, ROUGE1..4 для анотацій. Оскільки корпуси зазвичай мають
дані або для ключових слів, або для анотацій, але не для обох цих типів 
переробленого тексту, то, які метрики треба рахувати, задається параметрами.

Сам процес розрахунку за допомогою пакета, можна подивитися у файлі 
[evaluate_minicorpora.py](evaluate_minicorpora.py)

Наявна версія модуля keysum_eval підтримує тільки англійскі тексти, але в інтерфейсі 
закладено підтримку багатомовних корпусів.

## Корпуси

### Один документ

Один текст в корпусі має таку структуру:

```
@KEYWORDS
ключові, слова, через кому
або на різніх, рядках

@SUMMARY
У цьому розділі
йдуть речення аннотації
По одному на строку

@LANG
en
```
 
Кожна секція починається зі строки з '@' та ідентифікатором секції. Після 
ідентифікатора може йти що завгодно, то ж можна далі поставити двокрапку, чи ще шось.
Регістр ідентифікатора неважливий.
 
У @Summari є ще один синоним - @Digest. Це зроблено для того, щоб легше було корегувати
файли, які зроблено із застосуванням сервису [getdigest.com](https://getdigest.com)
 
Секції можуть бути відсутні. Тоді для списку ключових слів та анотації за 
замовчуванням використовується пустий список. Як ідентифікатор язику - англійська ('en').
 
Секції з незнайомими ідентифікаторами пропускаються.
 
### Структура корпусу
 
Щоб документ міг буті опрацьованим, функції розрахунку треба передати corpus_reader, 
який очікує:
 
* Ридеру передається два шляхи, в яких знаходяться тільки файли з анотаціями
* Список файлів (кількість та імена) мають бути однаковими
 
До цієї домашньої роботи я додав два корпуси:
 
- minicorpus_sum - корпус на базі перших 8 документів BBC NEWS. "Ідеальні" анотації 
взято саме із корпусу, як і оригінальні тексти. Для створення автоматичної анотації
використовувався он-лайн сервіс [getdigest.com](https://getdigest.com), з настройкою
компресії тексту 25%.
 
- minicorpus_kw - корпус на базі перших 8 документів NUC. "Ідеальний" список 
ключових слів взято також із корпусу NUC. А для створення автоматичного списку
також застосовували сервис [getdigest.com](https://getdigest.com). Нажаль, 
настройку кількості ключових слів там було спрятано, то ж взяли як є.
 
## Результати

Середні арифметичні значення всіх метрик для двох корпусів:
 
``` 
C:\Users\ssotn\Anaconda3\envs\nlp\python.exe D:/git-nlp/ss-prj-nlp-2019/students/SergeSotnyk/04-nlp-full-circle/project/evaluate_minicorpora.py

'minicorpus_sum' metrics: {'rouge_1': 0.5644124126853372, 'rouge_2': 0.50555971686626, 'rouge_3': 0.464391350557414, 'rouge_4': 0.4300323366910759, 'bleu': 32.38553763212366}
'minicorpus_kw' metrics: {'precision': 0.40094693284636473, 'recall': 0.3480706482469783, 'f1': 0.36279817050728513}
```