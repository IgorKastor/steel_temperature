# Предсказание температуры стали
## Аннотация
**Описание технологического процесса**

<img src="https://sun9-53.userapi.com/impg/QHffYmzLddVaJks8tIi3r2HPIB5Cqr5ualML6A/JXDRXAJVQl8.jpg?size=500x550&quality=95&sign=68b6c24b6edc388933c641f314179913&type=album" 
     align="left" width="250" alt="Схема установки">

Схема установки: 
```
1. ковш 
2. трайб-аппарат 
3. крышка
4. электроды
5. воронка подачи сыпучих присадок
6. аварийная фурма
```
    
Сталь обрабатывают в металлическом ковше (1) вместимостью около 100 тонн. Расплавленную сталь заливают в ковш и подогревают до нужной температуры графитовыми электродами (4). Сталь *легируют*, т.е. изменяют ее состав, подавая куски сплава из бункера для сыпучих материалов (5) или проволоку через специальный трайб-аппарат (2).

Перед тем как первый раз ввести легирующие добавки, измеряют температуру стали и производят химический анализ. Потом температуру на несколько минут повышают, добавляют легирующие материалы и продувают сплав инертным газом. Затем его перемешивают и снова проводят измерения. 

Цикл повторяется до достижения целевого химического состава и оптимальной температуры плавки, после чего расплавленная сталь отправляется на доводку металла или поступает в машину непрерывной разливки

**Цель, метрика.** Нужна модель, которая предскажет температуру полученной стали ($MAE < {6.8}^\circ С$).

**Задачи:** 
1. Разведочный анализ: просмотр общей информации по таблицам, изучение распределений, выявление пропусков и подозрительных (возможно, аномальных) выбросов.
2. Уточняющие вопросы заказчику по результатам п.1.
3. Предобработка данных: удаление аномалий, заполнение пропусков, генерация новых признаков, агрегирование по ключу.
4. Объединение предикторов в один датафрейм, изучение корреляций, дополнительная предобработка с учетом используемых моделей ML.
5. Разделение на обучающую и тестовую выборки, масштабирование.
6. Построение прогнозной модели: подбор гиперпараметров, оценка метрики на кросс-валидации, выбор лучшей модели.
7. Проверка качества лучшей модели на тестовой выборке, исследование важности признаков, проверка на адекватность с помощью константной модели.
8. Составление итогового отчета

**Данные:**

- ```data_arc_new.csv``` — данные об электродах;
- ```data_bulk_new.csv``` — данные о подаче сыпучих материалов (объём);
- ```data_bulk_time_new.csv``` — данные о подаче сыпучих материалов (время);
- ```data_gas_new.csv``` — данные о продувке сплава газом;
- ```data_temp_new.csv``` — результаты измерения температуры;
- ```data_wire_new.csv``` — данные о проволочных материалах (объём);
- ```data_wire_time_new.csv``` — данные о проволочных материалах (время)

Во всех файлах столбец `key` содержит номер партии. В файлах может быть несколько строк с одинаковым значением `key`, они соответствуют разным итерациям обработки. Данные о времени (начало и конец процесса) в файлах не синхронизированы. Значение *NaN* в данных об объеме соответствует 0.

## Результаты
### Cравнение решения и плана
План работы над проектом заключался в следующем: разведочный анализ, уточняющие вопросы заказчику по его результатам, предобработка данных и их подготовка к обучению, построение прогностической модели и проверка ее качества, составление итогового отчета.

Все пункты плана были выполнены без существенных отклонений.

### Трудности проекта 
Основные трудности были связаны с пониманием особенностей технологического процесса:
- каков возможный диапазон температуры сплава стали?
- как часто измеряют температуру?
- какие возможны сбои при работе нагревательной установки? и т.п. 

Данные моменты удалось прояснить после обращения к заказчику.  

### Ключевые шаги решения 
Развернутый план решения задачи:
1. Разведочный анализ: 
    - просмотр общей информации о признаках, 
    - изучение их распределений, 
    - выявление пропусков и подозрительных выбросов.
2. Предобработка данных: 
    - удаление аномалий, 
    - заполнение пропусков, 
    - генерация новых признаков, 
    - агрегирование.
3. Объединение признаков в один датафрейм:
    - изучение корреляций,
    - выделение предикторов (признаков для обучения).
4. Разделение на обучающую и тестовую выборки:
    - масштабирование,
    - дополнительная предобработка обучающей выборки.
5. Построение прогностических моделей: 
    - подбор гиперпараметров, 
    - оценка метрики на кросс-валидации, 
    - выбор лучшей модели.
6. Проверка качества лучшей модели на тестовой выборке:
    - проверка на адекватность с помощью константной модели,
    - исследование важности признаков.
    
### Подготовка данных 
#### Краткие результаты разведочного анализа
Изначально имелась информация более, чем о 3 тыс партий сплава стали:
- данные о нагревательных электродах (время и мощность нагрева);
- данные о подаче сыпучих и проволочных материалов (время, объем);
- данные о продувке сплава газом (объем);
- результаты измерения температуры партии стали (начальная, промежуточная, конечная).

Оказалось, что только по 2,3 тыс партий имелась информация о конечной температуре - она была целевой переменной, именно ее значение нужно было научиться предсказывать. Эти данные после заполнения пропусков и исключения аномалий использовали для машинного обучения.

Мы выяснили, что чаще всего добавляли 3-5 сыпучих материалов и 1-2 проволочных. У редко и часто добавляемых материалов средний объем существенно различается. Распределения суммарных объемов обоих типов материалов близки к нормальным, хотя есть небольшой пик малого объема по сыпучим материалам. Вообще, объемы сыпучих материалов обычно в 4-5 раз больше, чем прволочных, усредненный по количеству добавок объем сыпучего материала также больше (в 1,5-2 раза), чем проволочного. 

Распределения признаков, связанных с нагревом и продувкой сплава - нормальные. 

Имелись единичные аномальные (слишком низкие) значения температуры сплава и реактивной мощности электродов (отрицательное значение). Аномалии исключили. По объемам отдельных материалов и времени нагрева были замечены единичные, но значительные выбросы (см. ниже). 

#### Генерация новых признаков
Данные об измерении температуры и циклах нагрева после добавления очередной порции материа были агрегированы - получили обобщенные значения для каждой партии (ковша со сплавом стали). При этом, исходя из технологии и физических параметров процесса,  были выделены дополнительные признаки:
- число замеров температуры в партии;
- начальная и конечная температура партии;
- количества добавленных материалов (сыпучих, проволочных) на каждую партию;
- суммарные объемы добавленных материалов;
- средний объем добавленного материала;
- средняя полная мощность, время и суммарная тепловая энергия, переданная партии при нагреве.

Таким образом получили предварительный датафрейм с 36-ю признаками.

### Признаки, которые использовали для обучения
Полученые признаки были изучены с помощью коэффициента ранговой корреляции Спирмена $\rho$, поскольку отдельные признаки имели дискретное распределение. Признаки, очень слабо коррелирующие с таргетом - конечной температурой - исключили ($|\rho|<0.05$). Та также искючили мультиколлинеарность - удалили признаки с заметным взаимным $|\rho|>0.7$.

Осталось 14 признаков, наиболее значимых для определения конечной температуры сплава (перечислены по убыванию значения $|\rho|$:

1. Начальная температура сплава        
2. Объем пров. мат-ла *Wire 1*              
3. Объем сып. мат-ла *Bulk 6*              
4. Объем сып. мат-ла *Bulk 12*
5. Объем пров. мат-ла *Wire 2*               
6. Объем сып. мат-ла *Bulk 4*               
7. Значение тепловой энергии     
8. Средний объем сып. мат-в в партии (*Bulk_avg*)            
9. Число замеров температуры в партии           
10. Объем сып. мат-ла *Bulk 11*             
11. Число сып. мат-в в партии (*Bulk_count*)           
12. Объем пров. мат-ла *Wire 7*               
13. Объем сып. мат-ла *Bulk 14*              
14. Объем сып. мат-ла *Bulk 7*  

После разделения данных на обучающую и тестовую выборки (в соотношении *3:1*) у признаков *Bulk 7, -12, -14, Bulk_avg, Wire 1* и у тепловой энергии были исключены замеченные ранее выбросы, которые способны были сбить модель линейной регрессии. Их удалили из только из обучающей выборки по процентилю *99,9*.

### Обучение моделей
Были использованы стандартные модели для задачи регрессии, обучение велось методом рандомизированного поиска по гиперпараметрам (`sklearn.model_selection.RandomizedSearchCV` с аргументами `random_state=310723`, `n_jobs=-1`. Были получены следующие результаты:

Модель | Итерации | Папки CV | MAE | Обучение, с | Вычисление предсказаний, с
:-|:-|:-|:-|:-|:-
Линейная регрессия | 1 | 5 | 6.711 | 3.6 |	0.00
Случайный лес | 550 | 3 | 6.683 | 139.2 | 0.06
CatBoostRegressor | 5 | 5 | 6.613 | 38.8 |	0.01

По скорости и метрике лучшая модель - градиентный бустинг на деревьях решений (`CatBoostRegressor`).


### Описание лучшей модели 
*CatBoost* строит деревья так, что все вершины одного уровня имеют одинаковый предикат, причем одинаковые сплиты во всех вершинах одного уровня позволяют избавиться от ветвлений, что ускоряет применение модели и усиливает регуляризацию. 

Гиперпараметры, испытанные при обучении: 
- глубина: *4, 5, 6*;
- темп обучения: *0.005, 0.01*.

Параметры лучшей модели: темп обучения `learning_rate=0.01`, глубина `depth=5`.

Остальные параметры были определены моделью автоматически (по умолчанию), в том числе: метрика бустинга `eval_metric=RMSE`, число итераций `iterations=1000`, максимальное число листьев в ярусе `max_leaves=32`, число разбиений значений признаков `border_count=254` и др. 

Судя по корреляции в исходных данных, начальная температура должна была стать главным предиктором, и по значимости для модели она оказалась на 1-м месте. На 2-м месте как предиктор - значение тепловой энергии, хотя по коэффициенту корреляции она была только на 7-м месте. 

Признак *Wire 1* находится на 3-м месте по значимости и на 2-м по коэффициенту корреляции. Признак *Bulk 12* находится на 4-м месте по обеим оценкам. Выше видели, что это одни из самых частых добавок.

Признак *Bulk 6* на 3-м месте по корреляции и на 6-м по прогностической значимости. Возможно, это связано с тем, что он отрицательно коррелирует с целевой переменной. При этом ни по частоте добавления, ни по объему он изначально не выделялся

### Полученная метрика
У разработанной модели на тестовой выборке получено значение *MAE=6.31*, что удовлетворяет требованию *MAE<6.8*, при этом на тестовой выборке результат получился даже на 5% лучше, чем при обучении. Может быть, это связано с небольшим объемом тестовой выборки (менее 600 записей) и отсутствием в ней дополнительных малопредсказуемых выбросов. Также наша модель на 18% точнее предсказывает температуру, чем константная.
 
### Рекомендации по улучшению решения
- желательно было бы использовать больший объем исторических данных;
- следует регулярно обучать модель по мере изменения технологических компонентов процесса;
- можно детальнее обсудить с заказчиком значимость отдельных признаков с целью уменьшения их количества - тогда, возможно, получится использовать более простую модель (линейная регрессия, отдельное решающее дерево)