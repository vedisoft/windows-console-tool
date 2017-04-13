Пример быстрой интеграции Windows приложения с "Простыми звонками"
==========================================================

В данном примере мы рассмотрим интеграцию Windows-приложения собственной разработки с "Простыми звонками" посредством модуля "Простые звонки - Интеграция со сторонним Windows приложением". 


Шаг 1. Исходное Windows приложение
--------------------------

Наше исходное Windows приложение принимает неограниченное число аргументов и выводит их значения на экран в диалоговом окне.

output.js:

```js
var arguments = '';

for(i = 0; i < WScript.Arguments.length; i++) 
{
	arguments += WScript.Arguments(i) + ', ';	
} 

WScript.Echo(arguments);
```

Скопируйте файл output.js в папку C:\Temp\

Шаг 2. Установим и настроим модуль "Простые звонки - Интеграция со сторонним Windows приложением"
--------------------------------------

Скачаем [тестовый сервер и диагностическую утилиту](https://github.com/vedisoft/pz-developer-tools).

Запустим тестовый сервер:

```
> TestServer.exe
```
	
и подключимся к нему диагностической утилитой:

```
> Diagnostic.exe

[events off]> connect ws://localhost:10150 abc
* Далее приложение запросит ввести пароль, просто нажмите Enter
Успешно начато установление соединения с АТС
```

![Соединение установлено](https://github.com/vedisoft/windows-console-tool/blob/master/testserver-success.png)

Тестовое окружение настроено.

Скачаем модуль [Простые звонки - Интеграция со сторонним Windows приложением](http://prostiezvonki.ru/installs/ProstieZvonki_Integraciya_s_Windows_Prilozheniem.zip) на компьютер. Установим его в соответствие с [руководством](http://prostiezvonki.ru/documents/Ustanovka_nastroyka_Integraciya_s_Windows_Prilozheniem.pdf).

Откроем конфигурационный файл ProstieZvonki.cfg и зададим параметры для подключения к тестовому серверу:

```ini
[server]
; IP адрес или хостнейм сервера
serverHost=localhost

; Порт для подключения
serverPort=10150

; Если вы получаетесь к интернет-серверу Простых звонков, укажите true.
; Возможные варианты: true, false
useSSL=false

; Пароль для подключения к серверу
password=

; Интервал для переподключения к серверу в милисекундах
reconnectInterval=5000

[user]
; Внутренний (добавочный) номер пользователя
phone=100
```

Запустим ProstieZvonki-debug.exe. Если настройки для подключения заданы верно, то модуль успешно подключится к тестовому серверу:

Закроем консольное окно ProstieZvonki-debug.exe.


Шаг 3. Настроим вызов внешнего приложения в модуле "Простые звонки - Интеграция со сторонним Windows приложением"
--------------------------------------

Теперь научим модуль "Простые звонки - Интеграция со сторонним Windows приложением" вызывать внешнее приложение при получении событий:
- входящий звонок клиента на добавочный номер
- добавочный номер ответил на входящий звонок клиента
- добавочный номер завершил разговор с клиентом

Откроем конфигурационный файл ProstieZvonki.cfg, раскомментируем и зададим значения параметров incomingCallEventCmd, answeredCallEventCmd, finishedCallEventCmd:

```ini
[handlers]
; Обработчик события, которое возникает при входящем звонке на добавочный номер (зазвонил телефон). 
; Раскомментируйте параметр и укажите путь для запуска внешнего приложения. 
; Используйте кавычки, если путь для запуска внешнего приложения содержит пробелы.
incomingCallEventCmd="C:\Temp\output.js" INCOMING_CALL FROM=$FROM TO=$TO LINE=$LINE

; Обработчик события, которое возникает при ответе добавочного номера на входящий звонок (пользователь поднял трубку). 
; Раскомментируйте параметр и укажите путь для запуска внешнего приложения. 
; Используйте кавычки, если путь для запуска внешнего приложения содержит пробелы.
answeredCallEventCmd="C:\Temp\output.js" ANSWERED_INCOMING_CALL FROM=$FROM TO=$TO LINE=$LINE

; Обработчик события, которое возникает при завершении входящего или исходящего звонка на добавочном номере (сотрудник положил трубку). 
; Раскомментируйте параметр и укажите путь для запуска внешнего приложения. 
; Используйте кавычки, если путь для запуска внешнего приложения содержит пробелы.
finishedCallEventCmd="C:\Temp\output.js" FINISHED_CALL FROM=$FROM TO=$TO DATE=$DATE AUDIO=$AUDIO DURATION=$DURATION DIRECTION=$DIRECTION LINE=$LINE
```

Проверьте, что файл output.js находится в папке C:\Temp\

Шаг 3. Проверим, что модуль "Простые звонки - Интеграция со сторонним Windows приложением" вызывает внешнее приложение
--------------------------------------

Запустим ProstieZvonki-debug.exe. 

Чтобы проверить работу всплывающей карточки, создадим входящий звонок с номера 74951002030 на номер 100 с помощью диагностической утилиты Diagnostic.exe:

```
[events off]> Generate transfer 73430112233 101
```

Модуль "Простые звонки - Интеграция со сторонним Windows приложением" должен вызвать приложение output.js:
