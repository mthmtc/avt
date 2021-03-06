# 1. Описание

Мы предлагаем решить две задачи:
 1. Определить есть ли в объявлении контактная информация 
 2. Найти положение контактной информации в описании объявлении

Первая задача обязательная. Вторая - со звездочкой, можете решить ее если останется время или желание :)

## 1.1 Датасет
Для обучения и инференса обоих задач у вас есть следующие поля:
* `title` - заголовок,
* `description` - описание,
* `subcategory` - подкатегория,
* `category` - категория,
* `price` - цена,
* `region` - регион,
* `city` - город,
* `datetime_submitted` - дата размещения.

Таргет первой задачи: `is_bad`. Для второй разметка не предоставляется.

Есть два датасета: `train.csv` и `val.csv`. 
В датасетах могут встречаться (как и, к сожалению, в любых размечаемых данных) некорректные метки.

`train.csv` содержит больше данных, однако разметка в нём менее точная.

В `val.csv` существенно меньше данных, но более точная разметка.

Тестовый датасет, на котором мы оценим решение, будет больше похож на `val.csv`.

`val.csv` находится в папке `./data`. 
`train.csv` можно качать скриптом `./data/get_train_data.sh` или перейдя по 
[ссылке](https://drive.google.com/file/d/1LpjC4pNCUH51U_QuEA-I1oY6dYjfb7AL/view?usp=sharing) 

## 1.2 Задача 1
В первой задаче необходимо оценить вероятность наличия в объявлении контактной информации. 
Результатом работы модели является `pd.DataFrame` с колонками:
* `index`: `int`, положение записи в файле;
* `prediction`: `float` от 0 до 1.

Пример:

|index  |prediction|
|-------|----------|
|0|0.12|
|1|0.95|
|...|...|
|N|0.68|

В качестве метрики качества работы вашей модели мы будем использовать усредненный `ROC-AUC` по каждой категории объявлений.

## 1.3 Задача 2

Во второй задаче необходимо предсказать начало и конец контактной информации в описании (`description`) объявления. 
Например:
* для строки `Звоните на +7-888-888-88-88, в объявлении некорректный номер`: (11, 26),
* для строки `Звоните на +7-888aaaaa888aaaa88a88, в объявлении некорректный номер`: (11, 33),
* для строки `мой tg: @ivanicki_i на звонки не отвечаю`: (8, 18),
* для строки `мой tg: ivanicki_i на звонки не отвечаю`: (8, 17),
* если в описании объявления (поле `description`) контактов нет, то (None, None)
* если в описании объявления (поле `description`) более одного контакта (`Звоните не 89990000000 или на 89991111111`), то (None, None).

Результатом работы модели является `pd.DataFrame` с колонками:
* `index`: `int`, положение записи в файле;
* `start`: `int` or `None`, начало маски контакта;
* `end`: `int` or `None`, конец маски контакта.\
(`start` < `end`)
  
Пример:

|index  |start|end|
|-------|----------|-----|
|0|None|None|
|1|0|23
|2|31|123
|...|...|
|N|None|None


Для этой задачи метрикой будет усредненный IoU (`Intersection over Union`) по текстам объявлений.

# 2. Запуск решения

Ваш код для обучения и инференса моделей должен располагаться в папке `./lib`. 

Чтобы мы могли проверить ваше решение необходимо изменить метод `process` класса `Test` в файле `./lib/run.py`. 

В нем происходит инференс вашей модели на тестовых данных. 

Метод должен возвращать два датафрейма с ответами к задачам 1 и 2 соответственно.

Вы можете получить доступ к валидационным, трейновым (если файл скачан) и тестовым данным с помощью методов 'val', 'train' и 'test'.


Для того чтобы было легче разобраться как происходит запуск моделей мы подготовили константные 
"модели" (`./lib/model.py`), которые примеряются в `./lib/run.py` для формирования финального ответа.

Форматы тестового файла (в нем будет отсутствовать стобец `is_bad`), ответов зачач 1 и 2 приведены выше. 
После прогона будут запущены минимальные чекеры на соответствие ответов описанному формату

Решение будет проверяться в автоматическом режиме. 
Перед отправкой решения вам необходимо убедиться что все работает корректно запустив команду 
`docker-compose -f docker-compose.yaml up` в корне данного репозитория. 
Весь локальный код репозитория мапится в папку `/app` контейнера, локальная папка `./data` мапится в `/data` контейнера.
После этого запускается команда `python lib/run.py --debug`.
Чтобы все заработало у вас в системе должны быть установлены `docker` и `docker-compose`.

Вы можете добавить нужные библиотеки в файл `requirements.txt` или напрямую в `Dockerfile`.

Во время инференса моделей у контейнера не будет доступа в интернет. 

Обратите внимание, что в контейнере по умолчанию используется python3.8.

Удачи :)

# 3. Ресурсы

Ресурсы контейнера:
* 4 Гб оперативной памяти
* 2 ядра CPU
* 1 GPU, 2 Гб памяти

Ограничение на время работы:
* 60 000 объектов должны обрабатываться не более 180 минут для предсохраненной модели на CPU и 30 минут на GPU.

**Важно, чтобы всё, что нужно для запуска run.py, было в репозитории.**\
Часто решающие предлагают перед запуском вручную скачать архив с весами модели, в таком случае нужно чтобы веса скачивались и распаковывались при сборке контейнера либо обучение происходило в пайплайне.

# Baseline

Текущий бэйзлайн, который надо побить для первой части - 0.91.