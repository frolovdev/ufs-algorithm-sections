Предположим что с бэкэнда на фронт изначально прилетели секции в следующем порядке (Note 1 не отражает порядок это просто id, оно будет выглядеть скорее всего вот так Notes > something)

```

⬡ Note 1 { sourceOfContent: 'orginalHtml' }

⬡ Note 2 { sourceOfContent: 'orginalHtml' }

⬡ Note 3 { sourceOfContent: 'orginalHtml' }

⬡ Note 4 { sourceOfContent: 'orginalHtml' }

```

## Изменение секций местами (swap) 

Агент может менять порядок секций, поэтому первое что он сделал - 
это поменял первую и вторую секцию местами, 
чтобы поменять секцию местами мы используем следующий алгоритм:

1. Всегда храним в массиве `originalSections` исходные секции пришедшие с бэкэнда

2. Находим обе секции, которые будут подвержены swapу. Меянем их местами -> на выходе получем массив секций с измененным порядком

Пример: меняем первую и вторую секцию местами

```

⬡ Note 2

⬡ Note 1

⬡ Note 3

⬡ Note 4

```

```

Оригинальный.      Полученный
Массив.            массив

⬡ Note 1           ⬡ Note 2          

⬡ Note 2           ⬡ Note 1

⬡ Note 3           ⬡ Note 3

⬡ Note 4          ⬡ Note 4

```

```

⬡ Note 2 { sourceOfContent: 'orginalHtml' }

⬡ Note 1 { sourceOfContent: 'orginalHtml' }

⬡ Note 3 { sourceOfContent: 'orginalHtml' }

⬡ Note 4 { sourceOfContent: 'orginalHtml' }

```

> Если хотим учитывать нумерацию - то после каждого свапа пересчитываем нумерацию двух задействованных нод

## Добавление новой секции

Теперь у агента следующее состояние на руках с прошлого шага:

```

⬡ Note 2 { sourceOfContent: 'orginalHtml' }

⬡ Note 1 { sourceOfContent: 'orginalHtml' }

⬡ Note 3 { sourceOfContent: 'orginalHtml' }

⬡ Note 4 { sourceOfContent: 'orginalHtml' }

```

Он решил что хочет добавить секцию 'Note 7' (7 это не номер просто произовльный айди могло быть bla-bla bla)

1. Предположим мы хотим вставить секцию `7` между 2 и 1 нодой, мы будем использовать следующий алгоритм:

2. Находим после какой секции мы хотим вставить новую ноду, в нашем случае это Note 1

3. Вставляем и получаем следующий результат, (id генерируем на фронте с помощью `nanoid` или любого генратора id)

```

⬡ Note 2 { sourceOfContent: 'orginalHtml' }

⬡ Note 7 { sourceOfContent: 'changedByAgent' }

⬡ Note 1 { sourceOfContent: 'orginalHtml' }

⬡ Note 3 { sourceOfContent: 'orginalHtml' }

⬡ Note 4 { sourceOfContent: 'orginalHtml' }

```

> Если хотим учитывать нумерацию - Так как размерность итогово массива поменялся мы должно пройти по всем нодам ниже вставленной (все после Note 7), и применить операцию инкремента нашей нумерации

Разберем наш пример по шагам:

начинаем итерацию со следующей ноды после Note 7 (помним что массив это над тип связанного списка - операцию next можно выразить через взятие элемента по инкременту текущего индекса в массиве)

Результатом будет следующее состояние:

```
⬡ Note 2 { sourceOfContent: 'orginalHtml' }

⬡ Note 7 { sourceOfContent: 'changedByAgent' }

⬡ Note 1 { sourceOfContent: 'orginalHtml' }

⬡ Note 3 , sourceOfContent: 'orginalHtml' }

⬡ Note 4 , sourceOfContent: 'orginalHtml' }
```

---


Далее агент посвапал еще немного секции и привел их в следующее состояние

```
⬡ Note 2 { sourceOfContent: 'orginalHtml' }

⬡ Note 7 { sourceOfContent: 'changedByAgent' }

⬡ Note 3 { sourceOfContent: 'orginalHtml' }

⬡ Note 1 { sourceOfContent: 'orginalHtml' }

⬡ Note 4 , sourceOfContent: 'orginalHtml' }
```

После этих измнений он решил заслать на backend текущие изменения, но его ждали следующие особенности, так как клиент (customer) дослал дополнительно новых данных то секции на бекенде приняли новое состояние, плюс по нашей логике одна из секций скрылась. (В нашем кейсе Note 1 ушла, но добавились Note 5, Note 6 и Note 8)


Смотрите пояснения внизу

```
Исходное состяние          Новое состяоние       Состояние клиента

⬡ Note 1                    ⬡ Note 2              ⬡ Note 2

⬡ Note 2                    ⬡ Note 3              ⬡ Note 7

⬡ Note 3                    ⬡ Note 5              ⬡ Note 3

⬡ Note 4                    ⬡ Note 6              ⬡ Note 1

                            ⬡ Note 4              ⬡ Note 4

                            ⬡ Note 8 
```


Исходное состяоние - состояние на момент первого создания html и секций из него

Новое состояние - состояние полученное в тот момент, когда клиент отправил запрос на сохранение документа -> в этот момент пересоздался html и пересоздались секции

Состояние клиента - То что клиент отправил с фронтенда: а именно
```
⬡ Note2  { sourceOfContent: 'orginalHtml' }->
⬡ Note 7 { sourceOfContent: 'changedByAgent' }->
⬡ Note 3 { sourceOfContent: 'orginalHtml' }->
⬡ Note 1 { sourceOfContent: 'orginalHtml' }->
⬡ Note 4 , sourceOfContent: 'orginalHtml' }->
⬡ Note 8 , sourceOfContent: 'orginalHtml' }->
```
Теперь чтобы отдать клиенту на запрос обновление актуальные секции мы предпримем следующие шаги:

## Обновление и мердж данных при put (UPDATE) запросе

Так как алгоритм мерджа нетривиальный - запишем матрицу состояний

 | sourceOfContent | is Note id of client arrray exist in  Backend array | binary representation | serial number |
 | --------------- | --------------------------------------------------- | --------------------- | ------------- |
 | html            | false                                               | 00                    | 1             |
 | html            | true                                                | 01                    | 2             |
 | agent           | false                                               | 10                    | 3             |
 | agent           | true                                                | 11                    | 4             |

На руках имеем новое состяоние бэкэнда и клиента, слева backendArray(`bA[]`), справа clientArray(`cA[]`)
```
⬡ Note2 { sourceOfContent: 'orginalHtml' } ⬡ Note2   { sourceOfContent: 'orginalHtml' }

⬡ Note 3 { sourceOfContent: 'orginalHtml' } ⬡ Note 7 { sourceOfContent: 'changedByAgent' }

⬡ Note 5 { sourceOfContent: 'orginalHtml' } ⬡ Note 3 { sourceOfContent: 'orginalHtml' }

⬡ Note 6 { sourceOfContent: 'orginalHtml' } ⬡ Note 1 { sourceOfContent: 'orginalHtml' }

⬡ Note 4 { sourceOfContent: 'orginalHtml' } ⬡ Note 4 { sourceOfContent: 'orginalHtml' }

⬡ Note 8 { sourceOfContent: 'orginalHtml' }
```

Распишем действия соответствующие таблице состояний и будем применять на каждый шаг алгоритм, итерируемся по `cA[]` (далее будет пошаговый разбор конкретно этого кейса)

1) секции нет в `bA[]` & `sourceOfContent` === 'html'  => ничего не делаем пропускаем
2) секции есть в `bA[]` & `sourceOfContent` === 'html'  => берем индекс из `cA[]` и вставляем в результирующий массив по индексу с контент из `bA[]`
3) секции нет в `bA[]` & `sourceOfContent` === 'agent'  => берем индекс из `cA[]` и вставялем в результирующий массив по индексу с контентом из `cA` 
4) секции есть в `bA[]` & `sourceOfContent` === 'agent' => берем индекс из `cA[]` вставляем по индексу в результирующий массив с контентом из `cA[]`


Применим теперь данный алгоритм на наш кейс. начинаем итерироваться по `cA[]`, заводим `resultArray`

1) `⬡ Note2 { sourceOfContent: 'orginalHtml' }`

есть в `bA[]` & `sourceOfContent` === 'orginalHtml' => используем правило (2)

Таким образом на первом шаге массив будет выглядеть так

`resultArray = [ {sourceOfContent: originalHtml, body: ba.body  } ]`

1) `⬡ Note7 { sourceOfContent: 'changedByAgent' }`

нет в `bA[]` & `sourceOfContent` === 'changedByAgent'  => правило (3)

```

resultArray = [ 
    {id: Note2 , sourceOfContent: originalHtml, body: ba.body }, 
    {id: Note7,  sourceOfContent: changedByAgent, body: ca.body} 
  ]

```

3) `⬡ Note 3 { sourceOfContent: 'orginalHtml' }`

есть в `bA[]` & `sourceOfContent` === 'orginalHtml' => правило 2

```

resultArray = [ 
    {id: Note2 , sourceOfContent: originalHtml, body: ba.body  }, 
    {id: Note7,  sourceOfContent: changedByAgent, body: ca.body},
    {id: Note3,  sourceOfContent: orginalHtml, body: ba.body},
  ]

```

4) `⬡ Note 1 { sourceOfContent: 'orginalHtml' }`

нет в `bA[]` & `sourceOfContent: 'orginalHtml'`  => просто скипаем (1)

```

resultArray = [ 
    {id: Note2 , sourceOfContent: originalHtml, body: ba.body  }, 
    {id: Note7,  sourceOfContent: changedByAgent, body: ca.body },
    {id: Note3,  sourceOfContent: orginalHtml, body: ba.body },
  ]

```

5) `⬡ Note 4 { sourceOfContent: 'orginalHtml' }`

есть в `bA[]` & `sourceOfcontent` = `originalHtml` => правило (2)



```

resultArray = [ 
    {id: Note2 , sourceOfContent: originalHtml, body: ba.body  }, 
    {id: Note7,  sourceOfContent: changedByAgent, body: ca.body },
    {id: Note3,  sourceOfContent: orginalHtml, body: ba.body },
    {id: Note4,  sourceOfContent: orginalHtml, body: ba.body},
  ]

```

Попутно на каждом шаге мы помечали элементы из `bA[]` как вычеркнутые - получим следующий результат

```
bA = [
  ⬡ Note2 {  sourceOfContent: 'orginalHtml',   marked }, 
  ⬡ Note 3 {  sourceOfContent: 'orginalHtml',  marked },
  ⬡ Note 5 {  sourceOfContent: 'orginalHtml' },
  ⬡ Note 6 {  sourceOfContent: 'orginalHtml' },
  ⬡ Note 4 {  sourceOfContent: 'orginalHtml',   marked }
  ⬡ Note 8 {  sourceOfContent: 'orginalHtml' },
]

```

Теперь мы идем по `bA[]` и подмердживаем все что не помечено по следующему алгоритму

Если не marked то берем прев ноду и вставляем сразу за нашим элемент в `resultArray`

Пробуем итерироватсья по новому `bA[]`:

1) пропускаем
2) пропускаем
3) `(⬡ Note 5.prev) => Note 3`

находим Note3 в result array и вставялем за ней

```

resultArray = [ 
    {id: Note2 , sourceOfContent: originalHtml, body: ba.body }, 
    {id: Note7,  sourceOfContent: changedByAgent, body: ca.body},
    {id: Note3,  sourceOfContent: orginalHtml, body: ba.body},
   { id: Note5, sourceOfContent: 'orginalHtml' },
    {id: Note4,  sourceOfContent: orginalHtml, body: ba.body},
  ]


```

4) `(⬡ Note 6).prev => Note 5`

находим Note5 в резалт арей и вставялем за ней



```

resultArray = [ 
    {id: Note2 , sourceOfContent: originalHtml, body: ba.body  }, 
    {id: Note7,  sourceOfContent: changedByAgent, body: ca.body },
    {id: Note3,  sourceOfContent: orginalHtml, body: ba.body },
    { id: Note5, sourceOfContent: 'orginalHtml' },
    { id: Note6, sourceOfContent: 'orginalHtml' },
    {id: Note4,  sourceOfContent: orginalHtml, body: ba.body},
  ]


```

5) `⬡ Note 4 ` skip
6) `(⬡ Note 8).prev => Note 4`


```
  resultArray = [ 
    {id: Note2 , sourceOfContent: originalHtml, body: ba.body  }, 
    {id: Note7,  sourceOfContent: changedByAgent, body: ca.body },
    {id: Note3,  sourceOfContent: orginalHtml, body: ba.body },
    { id: Note5, sourceOfContent: 'orginalHtml' },
    { id: Note6, sourceOfContent: 'orginalHtml' },
    {id: Note4,  sourceOfContent: orginalHtml, body: ba.body},
    { id: Note8, sourceOfContent: 'orginalHtml' },
  ]
```

> Если хотим учитывать нумерацию -  Проходим по всему массиву выставляя нумерацию


Конец алгоритма
