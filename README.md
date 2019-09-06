# Yet another brace reader (os): Чтение "скобкофайлов" 1С (yabr.os)

Приложение oscript для чтения скобочных файлов 1С, аналог [обработок 1С (yabr.1c)](https://github.com/ArKuznetsov/yabr.1c).

## Общие сведения

- Чтение и обработка данных выполняется последовательно (построчно)
- Это эксперимент по реализации конвейерной обработки данных (pipeline), т.е. каждая порция данных, полученная в текущей обработке передается в указанный(-ные) в настройках обработчик(-и) для последующе обработки
- Настройки описываются в формате JSON (см. [Файл настроек](#jsonsettings))
- разработка ведется в формате EDT

## Обрабатываемые форматы

- Журнал регистрации 1С (LGP) и словарь журнала регистрации (LGF)
- Настройки информационных баз из файла настроек кластера 1С (1CV8Clst.lst)
- Список версий хранилища из отчета по версиям хранилища (MXL)
- Результаты замера производительности (PFF)

## Команды

- **process** (p) - выполняет обработку данных настройкам из файла (.json)
  - _<Путь>_ - путь к файлу настроек (по умолчанию ./ybrsettings.json)
  - _--work-dir_ _<путь к рабочему каталогу (по умолчанию: текущий каталог)>_
- **test** (t) - выполняет чтение указанного файла в скобочном формате 1С, выводит результаты в консоль (возможен замер времени)
  - _<Путь>_ - путь к файлу в скобочном формате 1С
  - _--start-row_  _<начальная строка для чтения (по умолчанию 1)>_
  - _--mesure-rate_ _<частота замера скорости выполнения (по умолчанию 100)>_

## Управляющие обработки

### МенеджерОбработкиДанных.os

Управляющая обработка-менеджер, читает настройки, запускает и управляет обработкой данных.

### yabr.os

Обработка для интерактивного указания файла настроек и запуска обработки данных.

## <a id="api"></a> Стандартный программный интерфейс обработки

**Функция ПринимаетДанные()** - признак возможности обработки, принимать входящие данные

**Функция ВозвращаетДанные()** - признак возможности обработки, возвращать обработанные данные

**Функция ОписаниеПараметров()** - возвращает структуру с описанием параметров обработки

**Функция МенеджерОбработкиДанных()** - возвращает ссылку на вызывающую/управляющую обработку - менеджер обработки данных

**Процедура УстановитьМенеджерОбработкиДанных(Знач НовыйМенеджерОбработкиДанных)** - устанавливает ссылку на вызывающую/управляющую обработку - менеджер обработки данных

**Функция Идентификатор()** - возвращает идентификатор обработки, установленный при инициализации в менеджере обработки данных

**Процедура УстановитьИдентификатор(Знач НовыйИдентификатор)** - устанавливает идентификатор обработки, вызывается при инициализации в менеджере обработки данных

**Функция ПараметрыОбработкиДанных()** - возвращает значения параметров обработки данных

**Процедура УстановитьПараметрыОбработкиДанных(Знач НовыеПараметры)** - устанавливает значения параметров обработки данных

**Функция ПараметрОбработкиДанных(Знач ИмяПараметра)** - возвращает значение указанного параметра обработки данных

**Процедура УстановитьПараметрОбработкиДанных(Знач ИмяПараметра, Знач Значение)** - устанавливает значение указанного параметра обработки

**Процедура УстановитьДанные(Знач ВходящиеДанные)** - устанавливает данные для обработки

**Процедура ОбработатьДанные()** - выполняет обработку данных

**Функция РезультатОбработки()** - возвращает результаты обработки данных

**Процедура ЗавершениеОбработкиДанных()** - выполняет действия при окончании обработки данных и оповещает обработку-менеджер о завершении обработки данных

## Обработчики данных

### ПоискНастроекКластера.os

Ищет в стандартном расположении или в указанных каталогах файлы настроек кластера серверов 1С (1cv8lst.lst) и передает найденные файлы далее на обработку.

#### Параметры

- **ОписанияКластеров** - массив путей к каталогам, в которых будет выполняться поиск файлов настроек кластеров 1С. Если не указан, то выполняется поиск в стандартных расположениях (%ProgramFiles%\1Cv8\srvinfo).

### ЧтениеКаталога.os

Читает список файлов из указанного каталога по указанной маске и передает для дальнейшей обработки по одному.

#### Параметры

- **Путь** - Путь к каталогу для файлов для обработки. Значение по умолчанию: <каталог запуска обработки>;
- **МаскиФайлов** - Маски поиска файлов, разделенные "";". Значение по умолчанию: "*.*";
- **ИскатьВПодкаталогах** - Флаг поиска в подкаталогах. Значение по умолчанию: Ложь;
- **СортироватьПоИмени** - Флаг сортировки найденных файлов по имени. Значение по умолчанию: Истина.

### ЧтениеСкобкоФайла.os

Читает скобочный файл в иерархию структур и массивов:

- чтение выполняется последовательно по строкам
- ошибки формата файла не проверяются и игнорируются, что сможет прочитать. то и получится
- элементы простых типов (число/строка) записываются в массив текущего уровня как есть
- элементы - массивы (ограниченные "{...}") "оборачиваются" в структуру вида:

```txt
    |-> Элемент
    |      |- Родитель    - родительский элемент (Неопределено - для корневого элемента)
    |______|________|
    |      |- Уровень     - уровень элемента в иерархии (0 - корневой элемент)
    |      |- Индекс      - индекс элемента в родительском элементе
    |      |- НачСтрока   - номер начальной строки диапазона, на основании которому создан текущий и подчиненные элементы
    |      |- КонСтрока   - номер конечной строки диапазона, на основании которому создан текущий и подчиненные элементы
    |      |- Значения    - массив дочерних элементов
    |_______________|
```

- Для закрывающихся скобок ("}") выполняется обратный вызов МенеджерОбработкиДанных.ПродолжениеОбработкиДанных (МенеджерОбработкиДанных.os) для передачи прочитанных данных на дальнейшую обработку

#### Параметры

- **ПутьКФайлу** - (обязательный, если обработчик стоит первым в цепочке) Имя файла, файл, поток данных для чтения;
- **УровниЗаписей** - Массив уровней записей (по вложенности), которые будут переданы на дальнейшую обработку. Значение по умолчанию: [1];
- **НачальнаяСтрока** - Номер строки входящих данных с которой будет начато чтение. Значение по умолчанию: 1;
- **КаталогПозицийЧтения** - Путь к каталогу для сохранения позиций начала чтения. Если не указан, то позиции чтения сохраняться не будут;
- **ЧастотаЗамераСкоростиВыполнения** - Количество прочитанных записей для которых будет выполнен расчет скорости чтения данных. Если указан 0, то расчет скорости выполняться не будет. Значение по умолчанию: 0.

### ЧтениеСпискаИБ.os

Принимает на вход данные в том виде как их возвращает обработка чтения "скобкофайлов" и обрабатывает элементы данных как запись настройки информационной базы в файле настроек кластера 1С.

#### Параметры

- **УровеньЭлементов** - (служебный, не требует установки) Номер уровня вложенности элементов в структуре данных, прочитанной из скобкофайла, которые будут прочитаны и обработаны. Значение: 3;
- **ИндексЭлементаРодителя** - (служебный, не требует установки) Индекс родительского элемента в структуре данных, подчиненные элементы которого будут прочитаны и обработаны. Значение: 2;
- **ПутьКФайлу** - Путь к файлу списка ИБ кластера 1С. Требуется для вывода информации о каталоге кластера 1С.

### ЧтениеСловаряЖР.os

Принимает на вход данные в том виде как их возвращает обработка чтения "скобкофайлов" и обрабатывает каждый элемент данных как запись словаря текстового журнала регистрации.

#### Параметры

- **УровеньЭлементов** - (служебный, не требует установки) Номер уровня вложенности элементов в структуре данных, прочитанной из скобкофайла, которые будут прочитаны и обработаны. Значение: 1;
- **ИндексЭлементаРодителя** - (служебный, не требует установки) Индекс родительского элемента в структуре данных, подчиненные элементы которого будут прочитаны и обработаны. Значение: 0;

### ЧтениеЖР.os

Принимает на вход данные в том виде как их возвращает обработка чтения "скобкофайлов" и обрабатывает каждый элемент данных как запись текстового журнала регистрации.

#### Параметры

- **УровеньЭлементов** - (служебный, не требует установки) Номер уровня вложенности элементов в структуре данных, прочитанной из скобкофайла, которые будут прочитаны и обработаны. Значение: 1;
- **ИндексЭлементаРодителя** - (служебный, не требует установки) Индекс родительского элемента в структуре данных, подчиненные элементы которого будут прочитаны и обработаны. Значение: 0;
- **Словари** - (обязательный) Словари данных журнала регистрации 1С. Задается в виде структуры <имя словаря>:[<ключ элемента>:<значение элемента>];
- **ВыгружатьКолонки** - Содержит список колонок с указанием необходимости их выгрузки. Задается в виде структуры <имя колонки>:true|false;
- **РазмерПорцииОбработки** - Количество записей, которое будет прочитано прежде чем они будут переданы на дальнейшую обработку. Значение по умолчанию: 1;
- **ИмяФайлаЖурнала** - Имя обрабатываемого файла журнала регистрации. Требуется для вывода информации об имени файла журнала;
- **ИнформационнаяБаза** - Описание или имя информационной базы, чей журнал обрабатывается. Требуется для вывода информации об информационной базе 1С.;

### ЧтениеОтчетаПоВерсиямХранилища.os

Принимает на вход данные в том виде как их возвращает обработка чтения "скобкофайлов" и обрабатывает как элементы данных табличного документа (MXL) с отчетом по версиям хранилища 1С.

#### Параметры

- **УровеньЭлементов** - (служебный, не требует установки) Номер уровня вложенности элементов в структуре данных, прочитанной из скобкофайла, которые будут прочитаны и обработаны. Значение: 4.;

### ЧтениеЗамераПроизводительности.os

Принимает на вход данные в том виде как их возвращает обработка чтения "скобкофайлов" и обрабатывает каждый элемент данных как запись замера производительности 1С (PFF).

#### Параметры

- **УровеньЭлементов** - (служебный, не требует установки) Номер уровня вложенности элементов в структуре данных, прочитанной из скобкофайла, которые будут прочитаны и обработаны. Значение: 3;

### ФильтрДанных.os

Принимает на вход данные в виде структуры или массива структур, проверяет их на соответствие заданным условиям и если условия выполняются, передает на дальнейшую обработку.

#### Параметры

- **Условия** - структура описания условий фильтра данных;

#### Пример настройки фильтра

```json
    "ИмяОбработки":"ФильтрДанных",
    "Параметры":{
        "Условия":{
            "Группа":"ИЛИ",
            "Условия": [
                {
                    "ЛевоеЗначение":"&Важность",
                    "ВидСравнения":"ВСписке",
                    "ПравоеЗначение": [
                        "Ошибка"
                    ]
                },
                {
                    "Группа":"И",
                    "Условия": [
                        {
                            "ЛевоеЗначение":"&СтатусТранзакции",
                            "ВидСравнения":"Равно",
                            "ПравоеЗначение": "Зафиксирована"
                        },
                        {
                            "ЛевоеЗначение":"&ПредставлениеСобытия",
                            "ВидСравнения":"Содержит",
                            "ПравоеЗначение": "Данные"
                        }
                    ]
                }
            ]
        }
    },
```

### ВыводДанныхВКонсоль.os

Обработчик вывода данных, любые входящие данные преобразует в формат JSON и выводит в консоль.

#### Параметры

- **ОбрабатыватьКакМассив** - Истина - входящий массив будет обработан поэлементно; Ложь - входящий массив будет обработан как 1 значение. Значение по умолчанию: Истина.

### ВыводДанныхВФайлJSON.os

Обработчик вывода данных, любые входящие данные преобразует в формат JSON и выводит в указанный файл.

#### Параметры

- **ПутьКФайлу** - (обязательный) Путь к файлу JSON для сохранения данных;
- **ОбрабатыватьКакМассив** - Истина - входящий массив будет обработан поэлементно; Ложь - входящий массив будет обработан как 1 значение. Значение по умолчанию: Истина.

### ВыводДанныхВЭластик.os

Обработчик вывода данных, любые входящие данные преобразует в формат JSON и отправляет в индекс Elastic.

#### Параметры

- **Эластик_Сервер** - (обязательный) Адрес сервера http-сервиса Elastic.");
- **Эластик_Порт** - Порт сервера http-сервиса Elastic. Значение по умолчанию: 9200;
- **Эластик_Путь** - Относительный путь к REST API Elastic;
- **Эластик_Пользователь** - Имя пользователя сервиса Elastic;
- **Эластик_Пароль** - Пароль пользователя сервиса Elastic;
- **ШаблонЗаголовкаИндекса** - (обязательный) Шаблон для установки значения заголовка индекса (_index) при выгрузке;
- **ШаблонТипаИндекса** - (обязательный) Шаблон для установки значения типа индекса (_type) при выгрузке;
- **ШаблонИдИндекса** -(обязательный) Шаблон для установки идентификатора индекса (_id) при выгрузке;
- **РазмерПорцииОбработки** - Количество записей, которое будет добавлено в пакет для отправки в Elastic. Значение по умолчанию: 1;
- **СохранятьОбработанныеДанные** - Флаг необходимости сохранения обработанных данных в результатах обработки. Значение по умолчанию: Ложь;
- **СохранятьОтправленныеСтроки** - Флаг необходимости сохранения строк, отправленных в Elastic в результатах обработки. Значение по умолчанию: Ложь.

## <a id="jsonsettings"></a> Файл настроек (JSON)

### Структура файла настроек

Файл настроек описывает последовательность вызова обработчиков для обработки данных. обработчик = обработка 1С реализующая вышеуказанный [API](#api).

```txt
    |-> Описание обработчика
    |      |- ИдОбработчика     - строковый идентификатор обработчика (необязательный)
    |      |- ИмяОбработки      - имя класса обработчика
    |      |- ПутьКОбработке    - путь к oscript-файлу класса обработчика
    |      |- Параметры         - структура параметров обработки
    |      |      |- <просто параметр>                 - параметр простого типа
    |      |      |- <параметр из данных обработчика>  - параметр вычисляемый обработчиком
    |      |      |      |- ИдОбработчика              - идентификатор обработчика из которого будет получен параметр
    |      |      |      |- ФункцияПолученияЗначения   - имя функции получения значения параметра (по умолчанию: "ПолучитьРезультат")
    |      |- Обработчики      - массив обработчиков данных полученных на текущем уровне
    |             |- <Описание обработчика>*           - структура, аналогичная данной
    |____________________|
```

### Доступные подстановки

- **$settingsDir** - каталог файла настроек
- **$yabrDir** - каталог запуска скрипта
- **$workDir** - указанный при запуске рабочий каталог (по умолчанию: каталог файла настроек)

### Пример файла настроек

Пример файла настроек чтения словарей журнала регистрации

```json
{
    // путь к обработке чтения файлов скобочного формата
    "ИмяОбработки":"ЧтениеСкобкофайла",
    "Параметры":{
        // путь к файлу словарей журнала регистрации
        "ПутьКФайлу":"$settingsDir\\1CV8.lgf",
        // обрабатываемые уровни вложенности "скобкофайла"
        "УровниЗаписей":[1]
    },
    "Обработчики":[
        {
            // путь к обработке чтения файла словарей журнала регистрации, обработчик будет вызван, для каждой прочитанной записи файла словарей
            "ИмяОбработки":"ЧтениеСловаряЖР",
            "Обработчики":[
                {
                    // путь к обработке вывода произвольных данных в файл JSON, обработчик будет вызван, для каждой прочитанной записи словаря журнала регистрации
                    "ИмяОбработки":"ВыводДанныхВФайлJSON",
                    "Параметры":{
                        // путь к файлу для сохранения результатов
                        "ПутьКФайлу":"$yabrDir\\..\\testdata\\jornal-dic2jsonfile-result.json"
                    }
                }
            ]
        }
    ]
}
```

Больше примеров расположены в каталоге [examples](./examples).
