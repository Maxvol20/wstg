---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование IMAP/SMTP-инъекций

|ID          |
|------------|
|WSTG-INPV-10|

## Обзор

Эта угроза затрагивает все приложения, взаимодействующие с почтовыми серверами (IMAP/SMTP), как правило, приложения web-почты. Целью этого теста является проверка способности вводить произвольные команды IMAP/SMTP на почтовые серверы из-за того, что входные данные не были должным образом нейтрализованы.

Метод IMAP/SMTP-инъекций эффективен, если почтовый сервер не доступен напрямую из Интернета. Там, где такое взаимодействие с почтовым сервером возможно, рекомендуется провести прямое тестирование.

IMAP/SMTP-инъекции позволяют получить доступ к почтовому серверу, который в противном случае был бы недоступен напрямую из Интернета. В некоторых случаях эти внутренние системы не обладают таким же уровнем защищённости инфраструктуры, что и web-серверы. Поэтому почтовые сервера могут быть более уязвимыми для атак со стороны конечных пользователей (см. схему, представленную на рисунке 1).

![IMAP SMTP Injection](images/Imap-smtp-injection.png)\
*Рисунок 4.7.10-1: Взаимодействие с почтовыми серверами с использованием метода IMAP/SMTP-инъекций*

На рисунке 1 показан поток трафика, обычно наблюдаемый при использовании технологий web-почты. На этапах 1 и 2 пользователь взаимодействует с клиентом web-почты, тогда как на шаге 2' тестировщик обходит клиент web-почты и напрямую взаимодействует с почтовыми серверами.

Этот метод позволяет выполнять самые разнообразные действия и атаки. Возможности зависят от типа и объёма инъекции, а также от тестируемой технологии почтового сервера.

Вот некоторые примеры атак с использованием метода IMAP/SMTP-инъекций:

- Эксплуатация уязвимостей в протоколах IMAP/SMTP
- Обход ограничений приложений
- Обход процесса анти-автоматизации
- Утечки информации
- Почтовый ретранслятор/спам-рассылки

## Задачи тестирования

- Найти точки для IMAP/SMTP-инъекции.
- Разобраться в потоках информации и структуре развёртывания системы.
- Оценить воздействие инъекций.

## Как тестировать

### Выявление уязвимых параметров

Чтобы обнаружить уязвимые параметры, тестировщик должен проанализировать способность приложения обрабатывать входные данные. Тестирование контроля входных данных требует, чтобы тестировщик отправлял фиктивные или вредоносные запросы на сервер и анализировал ответ. В защищённом приложении ответом должна быть ошибка с некоторым соответствующим действием, сообщающим клиенту, что что-то пошло не так. В уязвимом приложении вредоносный запрос может быть обработан внутренним приложением, которое ответит сообщением `HTTP 200 OK`.

Важно отметить, что отправляемые запросы должны соответствовать тестируемой технологии. Отправка строк SQL-инъекций для сервера MS SQL на сервер MySQL приведёт к ложноположительным ответам. В данном случае мы должны отправлять вредоносные команды IMAP, поскольку мы тестируем именно этот протокол.

Специальные параметры IMAP, которые обычно используются:

| На сервере IMAP     | На SMTP-сервере |
|------------------------|--------------------|
| Authentication         | Emissor email     |
| операции с почтовыми ящиками (list, read, create, delete, rename) | Destination email |
| операции с сообщениями (read, copy, move, delete) | Subject   |
| Disconnection          | Message body       |
|                        | Attached files     |

В данном примере тестируется параметр mailbox путём манипулирования запросами с параметром:

`http://<webmail>/src/read_body.php?mailbox=INBOX&passed_id=46106&startMessage=1`

Можно попробовать выполнить следующие действия:

- Присвоить параметру пустое значение:

`http://<webmail>/src/read_body.php?mailbox=&passed_id=46106&startMessage=1`

- Заменить значение на случайное:

`http://<webmail>/src/read_body.php?mailbox=NOTEXIST&passed_id=46106&startMessage=1`

- Добавить другие значения к параметру:

`http://<webmail>/src/read_body.php?mailbox=INBOX PARAMETER2&passed_id=46106&startMessage=1`

- Добавить нестандартные специальные символы (например, `\`, `'`, `"`, `@`, `#`, `!`, `|`):

`http://<webmail>/src/read_body.php?mailbox=INBOX"&passed_id=46106&startMessage=1`

- Исключить параметр:

`http://<webmail>/src/read_body.php?passed_id=46106&startMessage=1`

Конечный результат вышеупомянутого тестирования дает тестировщику три возможные ситуации:
S1 — приложение возвращает код/сообщение об ошибке;
S2 — приложение не возвращает код/сообщение об ошибке, но не выполняет запрошенную операцию;
S3 — приложение не возвращает код/сообщение об ошибке и нормально выполняет запрошенную операцию.

Ситуации S1 и S2 представляют собой успешную IMAP/SMTP-инъекцию.

Целью злоумышленника является получение ответа S1, так как это показатель того, что приложение уязвимо для инъекций и дальнейших манипуляций.

Предположим, что пользователь извлекает заголовки электронной почты с помощью следующего HTTP-запроса:

`http://<webmail>/src/view_header.php?mailbox=INBOX&passed_id=46105&passed_ent_id=0`

Злоумышленник может изменить значение параметра INBOX, вставив символ `"` (%22 в URL-кодировке):

`http://<webmail>/src/view_header.php?mailbox=INBOX%22&passed_id=46105&passed_ent_id=0`

В этом случае ответ приложения может быть таким:

```txt
ERROR: Bad or malformed request.
Query: SELECT "INBOX""
Server responded: Unexpected extra arguments to Select
```

Ситуацию S2 сложнее успешно протестировать. Необходимо использовать слепую инъекцию команд, чтобы определить, уязвим ли сервер.

С другой стороны, последняя ситуация (S3) не имеет отношения к данному абзацу.

> Список уязвимых параметров
>
> - Затронутая функциональность
> - Тип возможной инъекции (IMAP/SMTP)

### Понимание потоков информации и структуры развёртывания клиента

После выяснения всех уязвимых параметров (например, `passed_id`) тестировщик должен определить, какой уровень инъекции возможен, а затем разработать план тестирования для дальнейшей эксплуатации приложения.

В данном тестовом примере мы обнаружили, что параметр приложения `passed_id` уязвим и используется в следующем запросе:

`http://<webmail>/src/read_body.php?mailbox=INBOX&passed_id=46225&startMessage=1`

Используя следующий тестовый пример (предоставление буквенного значения, когда требуется числовое):

`http://<webmail>/src/read_body.php?mailbox=INBOX&passed_id=test&startMessage=1`

вызовет следующее сообщение об ошибке:

```txt
ERROR : Bad or malformed request.
Query: FETCH test:test BODY[HEADER]
Server responded: Error in IMAP command received by server.
```

В этом примере сообщение об ошибке вернуло имя выполненной команды и соответствующие параметры.

В других ситуациях сообщение об ошибке содержит название выполняемой команды, но чтение соответствующего [RFC](#ссылки) позволит разобраться, какие ещё команды можно выполнить.

Если приложение не выдаёт содержательного сообщения об ошибках, необходимо проанализировать затронутую функциональность, чтобы вывести все связанные с ней команды (и параметры). Например, если в функции создания почтового ящика был обнаружен уязвимый параметр, логично предположить, что уязвимой командой IMAP является `CREATE`. Согласно RFC, команда `CREATE` принимает один параметр, который указывает имя создаваемого почтового ящика.

> Список затронутых команд IMAP/SMTP
>
> - Тип, значение и количество параметров, ожидаемых для затронутых команд IMAP/SMTP

### Инъекция команд IMAP/SMTP

Как только тестировщик определил уязвимые параметры и проанализировал контекст, в котором они выполняются, следующим этапом является эксплуатация функциональности.

Этот этап имеет два возможных исхода:

1. Инъекция возможна в неаутентифицированном состоянии: затронутая функциональность не требует аутентификации пользователя. Доступные для инъекций (IMAP) команды ограничиваются `CAPABILITY`, `NOOP`, `AUTHENTICATE`, `LOGIN`, и `LOGOUT`.
2. Инъекция возможна только в аутентифицированном состоянии: для успешной эксплуатации требуется полная аутентификация пользователя, прежде чем можно будет продолжить тестирование.

В любом случае типичная структура IMAP/SMTP-инъекции выглядит следующим образом:

- Header: окончание ожидаемой команды;
- Body: инъекция новой команды;
- Footer: начало ожидаемой команды..

Важно помнить, что для выполнения IMAP/SMTP-команды предыдущая команда должна быть завершена CRLF-последовательностью (`%0d%0a`).

Предположим, что на этапе [выявления уязвимых параметров](#выявление-уязвимых-параметров) злоумышленник обнаруживает, что параметр `message_id` в следующем запросе является уязвимым:

`http://<webmail>/read_email.php?message_id=4791`

Предположим также, что в результате анализа, проведённого на этапе [понимания потоков информации и схемы развёртывания клиента](#понимание-потоков-информации-и-схемы-развёртывания-клиента), команда и аргументы, связанные с этим параметром, определены как:

`FETCH 4791 BODY[HEADER]`

В этом сценарии структура IMAP-инъекции будет следующей:

`http://<webmail>/read_email.php?message_id=4791 BODY[HEADER]%0d%0aV100 CAPABILITY%0d%0aV101 FETCH 4791`

которая сформирует следующие команды:

```sql
???? FETCH 4791 BODY[HEADER]
V100 CAPABILITY
V101 FETCH 4791 BODY[HEADER]
```

где:

```sql
Header = 4791 BODY[HEADER]
Body   = %0d%0aV100 CAPABILITY%0d%0a
Footer = V101 FETCH 4791
```

> Список затронутых IMAP/SMTP-команд
>
> - Инъекция произвольной IMAP/SMTP-команды

## Ссылки

### Технические руководства

- [RFC 0821 "Simple Mail Transfer Protocol"](https://tools.ietf.org/html/rfc821)
- [RFC 3501 "Internet Message Access Protocol - Version 4rev1"](https://tools.ietf.org/html/rfc3501)
- [Vicente Aguilera Díaz: "MX Injection: Capturing and Exploiting Hidden Mail Servers"](http://www.webappsec.org/projects/articles/121106.pdf)
