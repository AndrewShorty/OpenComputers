| Содержание |
| ----- |
| [О библиотеке](#О-библиотеке) |
| [Установка](#Установка) |
| [Глобальные функции](#Глобальные-функции) |
| [Дополнения библиотеки table](#Дополнения-библиотеки-table) |
| [Дополнения библиотеки string](#Дополнения-библиотеки-string) |
| [Дополнения библиотеки math](#Дополнения-библиотеки-math) |
| [Дополнения библиотеки filesystem (OpenOS)](#Дополнения-библиотеки-filesystem-openos) |

О библиотеке
======
AdvancedLua - библиотека, дополняющая стандартные библиотеки Lua и OpenOS отсутствующими, однако крайне необходимыми в быту функциями: быстрой сериализацией таблиц, определением текущего исполняемого скрипта, переносом строк, округлением чисел, получением сортированных файловых списков, различными методами обработки бинарных данных и т.п. 

Установка
======

Исходный код доступен по ссылке: https://github.com/IgorTimofeev/OpenComputers/blob/master/lib/advancedLua.lua

Для загрузки на компьютер вы можете воспользоваться стандартной утилитой **wget**:

    wget https://raw.githubusercontent.com/IgorTimofeev/OpenComputers/master/lib/advancedLua.lua -f

Глобальные функции
======

**getCurrentScript**( ): *string* path
-----------------------------------------------------------

Функция возвращает путь к текущему исполняемому скрипту. Для примера запустим файл **/Test/Main.lua** со следующим содержимым:

```lua
print("Путь к текущему скрипту: " .. getCurrentScript())
```
В результате на экране будет отображен тот самый путь:

```lua
Путь к текущему скрипту: /Test/Main.lua
```

**enum**( ... ): *table* result
-----------------------------------------------------------

Функция принимает строковые аргументы и возвращает таблицу с этими аргументами в виде ключей, а также с их порядковым номером в виде значений. Данная функция создана для удобства, когда нет желания вручную изменять значения полей на нужные:

```lua
local alignment = enum(
	"left",
	"center",
	"right"
)
```

В результате таблица alignment будет иметь следующий вид:

```lua
{
	left = 1,
	center = 2,
	right = 3
}
```

Дополнения библиотеки table
======

table.**serialize**( t, [ pretty, indentationWidth, indentUsingTabs, recursionStackLimit ] ): *string* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *table* | t | Таблица, которую необходимо сериализовать |
| *boolean* | pretty | Опциональный аргумент, сериализующий таблицу для наилучшего визуального восприятия человеком. По умолчанию имеет значение false |
| *int* | indentationWidth | Опциональный аргумент, отвечающий за ширину отступа в символах при сериализации в режиме **pretty** |
| *boolean* | indentUsingTabs | Опциональный аргумент, отвечающий за выбор символа отступа при сериализации в режиме **pretty**. По умолчанию имеет значение false |
| *int* | recursionStackLimit | Опциональный аргумент, отвечающий за ограничение стека рекурсии при сериализации таблиц большого размера |

Метод изначально создавался в качестве быстрой альтернативы **/lib/serialization.lua**, поставляемой OpenOS. Он преобразует содержимое таблицы в строку и крайне удобен для сохранения конфигов различного ПО в понятном для человека виде с сохранением исходной структуры таблицы. Для примера рассмотрим следующий код:

```lua
local myTable = {
	"Hello",
	"world",
	abc = 123,
	def = "456",
	ghi = {
		jkl = true,
	}
}

print("Обычная сериализация: " .. table.serialize(myTable))
print(" ")
print("Красивая сериализация: " .. table.serialize(myTable, true))
``` 

В результате выполнения скрипта на экране будет отображена сериализованная структура таблицы :

```lua
Обычная сериализация: {[1]="Hello",[2]="world",["abc"]=123,["def"]="456",["ghi"]={["jkl"]=true}}

Красивая сериализация: {
	[1] = "Hello",
	[2] = "world",
	abc = 123,
	def = "456",
	ghi = {
		jkl = true,
	}
}
```

Обращаю ваше внимание, что аргумент **pretty** выполняет несколько дополнительных проверок на тип ключей и значений таблицы, а также генерирует символы переноса строки после каждого значения. Поэтому используйте его только в том случае, когда читабельность результата стоит в приоритете над производительностью.

table.**unserialize**( text ): *table or nil* result, *string* reason
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *string* | text | Строка, созданная методом table.**serialize**()  |

Метод пытается десериализовать строковое представление lua-таблицы и вернуть результат. Если это невозможно, то возвращается nil и строка с причиной синтаксической ошибки. Для примера выполним простейшую десериализацию:

```lua
local result = table.unserialize("{ abc = 123 }")
```

В результате таблица result будет иметь следующий вид:

```lua
{
	abc = 123
}
```

table.**toFile**( path, ... )
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *string* | path | Путь к файлу, в который необходимо записать сериализованный результат |
| - | ... | Множество аргументов, принимаемых функцией table.**serialize**(...) |

Метод аналогичен table.**serialize**(...), однако вместо возвращения строкового результата записывает его в файл. Он крайне удобен для быстрого сохранения конфига ПО без излишних заморочек.

table.**fromFile**( path ): *string* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *string* | path | Путь к файлу, содержимое которого необходимо десериализовать |

Метод аналогичен table.**unserialize**(...), однако строковое содержимое он читает напрямую из существующего файла, возвращая десериализованный результат. Опять же, по большей части он применяется для удобной загрузки конфигов ПО.

table.**size**( t ): *int* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *table* | t | Таблица, число ключей которой необходимо вычислить |

Метод возвращает число ключей переданной таблицы. Отличается от варианта **#t** тем, что подсчитывает также ненумерические индексы

table.**contains**( t, object ): *boolean* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *table* | t | Таблица, в которой будет осуществлен поиск объекта |
| *var* | object | Объект, наличие которого в таблице необходимо проверить |

Метод определяет, присутствует ли объект в таблице и возвращает результат

table.**indexOf**( t, object ): *var* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *table* | t | Таблица, в которой будет осуществлен поиск объекта |
| *var* | object | Объект, индекс которого необходимо определить |

Метод возвращает индекс (ключ) переданного объекта. Тип индекса может быть различным в зависимости от структуры таблицы: к примеру, в таблице {abc = 123} число 123 имеет строковый индекс abc

table.**copy**( t ): *table* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *table* | t | Таблица, которую необходимо сдублирвать |

Метод рекурсивно копирует содержимое таблицы t в новую и возвращает результат. Обращаю внимание на то, что таблицы, ссылающиеся сами на себя, не поддерживаются (ограничение стека рекурсии по аналогии с table.**serialize**() пилить было оч оч лень, прости <3)

Дополнения библиотеки string
======

string.**limit**( s, limit, mode, noDots ): *string* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *string* | s | Строка, длину которой необходимо ограничить |
| *int* | limit | Максимальная длину строки |
| *string* | mode | Режим ограничения для вставки символа многоточия. Может принимать значения **left**, **center** или **right** |
| *boolean* | noDots | Режим ограничения строки классической обрезкой без использования символа многоточия |

Метод ограничивает строку, вставляя символ многоточия в необходимом месте и возвращая результат. Для примера запустим код:

```lua
print("Ограничение слева: " .. string.limit("HelloBeautifulWorld", 10, "left"))
print("Ограничение по центру: " .. string.limit("HelloBeautifulWorld", 10, "center"))
print("Ограничение справа: " .. string.limit("HelloBeautifulWorld", 10, "right"))
```

В результате на экране будет отображено следующее:

```lua
Ограничение слева: …ifulWorld
Ограничение по центру: Hello…orld
Ограничение справа: HelloBeau…
```

string.**wrap**( s, wrapWidth ): *table* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *string/string[]* | s | Строка либо массив строк, которые необходимо перенести по указанной длине |
| *int* | wrapWidth | Максимальная длина строки, на которую следует ориентироваться при переносе |

Метод осуществляет перенос строки с указанным ограничением по длине и возвращает результат в виде таблицы. Если размер отдельно взятого слова превышает указанную длину, то слово будет "разрезано" на составляющие части.

Также поддерживаются символы **\n** для автоматического переноса каретки на новую строку.  Для примера рассмотрим код:

```lua
local limit = 20
local text = "Привет, как дела? Сегодня отличный денек для выгула твоей вонючей псины, не так ли, Сэм?\n\nАх, ты уже не тот Сэм, с которым Фродо расплавил кольцо Саурона в самом сердце Роковой Горы"
local lines = string.wrap(text, limit)

print(string.rep("-", limit))
for i = 1, #lines do
	print(lines[i])
end
```
В результате на экране будет отображен текст:

```lua
--------------------
Привет, как дела?
Сегодня отличный
денек для выгула
твоей вонючей псины,
не так ли, Сэм?

Ах, ты уже не тот
Сэм, с которым Фродо
расплавил кольцо
Саурона в самом
сердце Роковой Горы
```


string.**unicodeFind**( ... ): ...
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| - | ... | Множество аргументов, аналогичных таковым для функции string.**find**(...) |

Метод аналогичен string.**find**(...), однако позволяет работать с юникодом. Незаменимая штука для русскоговорящей аудитории!

Дополнения библиотеки math
======

math.**round**( number ): *float* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *float* | number | Число, которое необходимо округлить |

Метод округляет число до ближайшего целого и возвращает результат

math.**roundToDecimalPlaces**( number, decimalPlaces ): *float* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *float* | number | Число, которое необходимо округлить |
| *int* | decimalPlaces | Число знаков после запятой у округляемого числа |

Метод округляет число до ближайшего целого, лимитируя результат до указаннонного числа знаков после запятой и возвращает результат

math.**shorten**( number, decimalPlaces ): *string* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | number | Число, которое необходимо визуально сократить |
| *int* | decimalPlaces | Число знаков после запятой у результата |

Метод преобразует входное число в строку с приставками "**K**", "**M**", "**B**" и "**T**" в завимости от размера числа. Для примера выполним код:

```lua
print("Сокращенный результат: " .. math.shortenNumber(13484381, 2))
```

В результате на экране отобразится следующее:

```lua
Сокращенный результат: 13.48M
```

math.**getDigitCount**( number ): *int* digitCount
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | number | Число, количество десятичных знаков которого необходимо вычислить |

Метод возвращает количество десятичных знаков в числе. К примеру, в числе **512** их **3**, в числе **1888** их **4**

Дополнения библиотеки bit32
======

bit32.**merge**( number1, number2 ): *int* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | number1 | Первое число для склейки |
| *int* | number2 | Второе число для склейки |

Метод "склеивает" два числа и возвращает результат. К примеру, вызов метода с аргументами **0xAA** и **0xBB** вернет число **0xAABB**

bit32.**numberToByteArray**( number ) : *table* byteArray
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | number | Число, которое необходимо преобразовать в байт-массив |

Метод извлекает составляющие байты из числа и возвращает таблицу с ними. К примеру, вызов метода с аргументом **0xAABBCC** вернет таблицу **{0xAA, 0xBB, 0xCC}**

bit32.**numberToFixedSizeByteArray**( number, arraySize ) : *table* byteArray
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | number | Число, которое необходимо преобразовать в байт-массив |
| *int* | arraySize | Фиксированный размер выходного массива |

Метод аналогичен bit32.**numberToByteArray**(), однако размер выходного массива указывается вручную. Если количество байт в числе меньше указанного размера, то выходной массив будет дополнен отсутствующими нулями, в противном случае массив заполнится лишь частью байт числа. К примеру, вызов метода с аргументами **0xAABBCC** и **5** вернет таблицу **{0x00, 0x00, 0xAA, 0xBB, 0xCC}**

bit32.**byteArrayToNumber**( byteArray ) : *int* number
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | number | Байт-массив, который необходимо преобразовать в число |

Метод преобразует байты из переданного массива в целое число. К примеру, вызов метода с аргументом **{0xAA, 0xBB, 0xCC}** вернет число **0xAABBCC**

bit32.**byteArrayToNumber**( byteArray ) : *int* number
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *int* | number | Байт-массив, который необходимо преобразовать в число |

Метод преобразует байты из переданного массива в целое число. К примеру, вызов метода с аргументом **{0xAA, 0xBB, 0xCC}** вернет число **0xAABBCC**


Дополнения библиотеки filesystem (OpenOS)
======

filesystem.**extension**( path ): *string* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *string* | path | Путь к файлу, расширение которого необходимо получить |

Метод возвращает строковое расширение файла по указанному пути. К примеру, для файла **/Test/HelloWorld.lua** будет возвращена строка **.lua**

filesystem.**hideExtension**( path ): *string* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *string* | path | Путь к файлу, расширение которого необходимо скрыть |

Метод скрывает расширение файла по указанному пути (если оно имеется) и возвращает строковый результат. К примеру, для файла **/Test/HelloWorld.lua** будет возвращена строка **/Test/HelloWorld**

filesystem.**isFileHidden**( path ): *boolean* result
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *string* | path | Путь к файлу, скрытость которого необходимо проверить |

Метод проверяет, является ли файл скрытым (т.е. начинается ли его название с символа "**.**") и возвращает результат. К примеру, для файла **/Test/.Hello.lua** будет возвращено **true**

filesystem.**sortedList**(path, sortingMethod, [ showHiddenFiles, filenameMatcher, filenameMatcherCaseSensitive ] ): *table* fileList
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *string* | path | Путь к директории, список файлов которой необходимо получить |
| *string* | sortingMethod | Метод сортировки файлов. Может принимать значения **name**, **type** и **date** |
| *boolean* | showHiddenFiles | Опциональный аргумент, отвечающий за включение в список скрытых файлов. По умолчанию имеет значение false |
| *string* | filenameMatcher | Опциональный аргумент, представляющий из себя регулярное выражение, по которому будут отсекаться файлы из списка. К примеру, выражение "**%d+%.lua**" будет включать в список только файлы с расширением **.lua** и имеющие в название исключительно цифры |
| *string* | filenameMatcherCaseSensitive | Опциональный аргумент, позволяющий игнорировать регистр символов при использовании аргумента **filenameMatcher** |

Метод получает список файлов из указанной директории в соответствии с переданными условиями, сортируя их определенным методом. Возвращаемый результат представляет собой классическую нумерически индексированную таблицу:

```lua
{
	"bin/",
	"lib/",
	"home/",
	"Main.lua"
}
```


filesystem.**readUnicodeChar**( fileHandle ): *string* char
-----------------------------------------------------------
| Тип | Аргумент | Описание |
| ------ | ------ | ------ |
| *handle* | fileHandle | Открытый файловый дескриптор в бинарном режиме (**rb**) |

Метод читает из файлового дескриптора юникод-символ, используя бинарные операции. Поскольку "чистый" Lua не позволяет работать с юникод-символами при чтении файлов в текстовом режиме, то этот метод крайне полезен при написании собственных форматов данных. Отмечу, что для успешного чтения вы должны быть уверены, что читаемые байт-последовательности из дескриптора **гарантированно** соответствует какому-либо юникод-символу