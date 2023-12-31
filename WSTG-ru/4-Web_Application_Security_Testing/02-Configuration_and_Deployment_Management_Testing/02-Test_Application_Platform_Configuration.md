---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование конфигурации платформы приложения

|ID          |
|------------|
|WSTG-CONF-02|

## Обзор

Правильная конфигурация элементов, составляющих архитектуру приложения, важна для предотвращения ошибок, которые могут поставить под угрозу безопасность всей архитектуры.

Анализ и тестирование конфигурации — важнейшая задача при разработке и сопровождении архитектуры. Это связано с тем, что различным системам, как правило, предоставляются универсальные конфигурации, которые могут не подходить для выполняемых ими задач на конкретном сайте, на котором они установлены.

Хотя типичная установка web-сервера и сервера приложений включает много функциональных возможностей (например, примеры приложений, документацию, тестовые страницы), — всё ненужное для эксплуатации должно быть удалено до развёртывания, чтобы избежать уязвимостей после установки.

## Задачи тестирования

- Убедиться, что удалены все известные ненужные файлы и изменены значения по умолчанию.
- Убедиться, что в рабочей среде не осталось отладочного кода и отключены инструменты для отладки.
- Проанализировать механизмы ведения журнала событий, установленные для приложения.

## Как тестировать

### Тестирование методом «чёрного ящика»

#### Примеры, известные файлы и каталоги

Многие web-серверы и серверы приложений при установке по умолчанию предоставляют примеры приложений и файлов для удобства разработчика и для проверки правильности работы сервера сразу после установки. Однако позже стало известно, что многие приложения по умолчанию уязвимы. Это имело место, например, в CVE-1999-0449 (отказ в обслуживании в IIS, когда устанавливался образец сайта Exair), CAN-2002-1744 (уязвимость обхода каталога в CodeBrws.asp в Microsoft IIS 5.0), CAN-2002-1630 (использование sendmail.jsp в Oracle 9iAS) или CAN-2003-1172 (обход каталога в образце исходного кода Apache Cocoon).

Подробный список известных файлов и образцов каталогов различных web-серверов и серверов приложений есть в CGI-сканерах, которые помогут быстро определить, есть ли подобные файлы. Однако единственный способ убедиться в этом — досконально изучить состав web-сервера или сервера приложений и определить, что относятся к приложению, а что нет.

#### Анализ комментариев

Программисты часто вставляют комментарии при разработке больших web-приложений. Однако комментарии, включённые в HTML-код, могут раскрывать внутреннюю информацию, которая не должна быть доступна злоумышленнику. Иногда даже исходный код закомментирован, поскольку функциональность больше не нужна, но этот комментарий непреднамеренно просачивается на HTML-страницы, предоставляемые пользователям.

Анализ комментариев должен проводится для того, чтобы определить возможность утечки чувствительной информации через комментарии. Этот анализ можно тщательно провести только посредством анализа статичного и динамического контента web-сервера и поиска файлов. Полезно просматривать сайт как в автоматическом, так и в ручном режиме и сохранять весь извлечённый контент. Затем в нём можно искать HTML-комментарии, имеющиеся в коде.

#### Конфигурация системы

Для предоставления специалистам по ИТ и безопасности подробной оценки соответствия целевых систем могут использоваться различные инструменты, документы или чек-листы по базовым уровням конфигурации или сравнительным показателям. Такие инструменты включают (но не ограничиваются):

- [CIS-CAT Lite](https://learn.cisecurity.org/cis-cat-lite)
- [Attack Surface Analyzer от Microsoft](https://github.com/microsoft/AttackSurfaceAnalyzer)
- [National Checklist Program от NIST](https://nvd.nist.gov/ncp/repository)

### Тестирование методом «серого ящика»

#### Анализ конфигурации

Конфигурация web-сервера или сервера приложений играет важную роль в защите содержимого сайта, и ее необходимо тщательно проверять, чтобы выявить распространённые ошибки конфигурации. Очевидно, что рекомендуемая конфигурация меняется в зависимости от политики сайта и функций, которые должны обеспечиваться программным обеспечением сервера. Однако в большинстве случаев, чтобы определить, надежно ли защищен сервер, необходимо следовать рекомендациям по настройке, предоставленным поставщиком программного обеспечения или внешними сторонами.

Дать универсальные рекомендации как должен быть настроен сервер невозможно, однако, например, для IIS следует учитывать следующее:

- Включайте только те серверные модули (расширения ISAPI в случае IIS), которые необходимы для приложения. Это уменьшает поверхность атаки, поскольку сервер уменьшается в размерах и сложности вследствие отключения программных модулей. Это также предотвращает влияние на сайт уязвимостей, если они присутствуют только в отключенных модулях.
- Обрабатывайте ошибки сервера (400-е и 500-е) с помощью пользовательских, а не стандартных страниц web-сервера. В частности, убедитесь, что ошибки приложения не доходят до конечного пользователя и что через эти ошибки не произойдет утечка кода, поскольку это поможет злоумышленнику. На самом деле об этом часто забывают, т.к. разработчикам эта информация нужна в среде тестирования.
- Убедитесь, что программное обеспечение сервера работает с минимальными привилегиями в операционной системе. Это предотвращает прямую компрометацию всей системы, хотя злоумышленник может повысить привилегии после запуска кода от имени web-сервера.
- Убедитесь, что программное обеспечение сервера правильно регистрирует как легитимный доступ, так и ошибки.
- Убедитесь, что сервер настроен для правильной обработки перегрузок и предотвращения атак типа «отказ в обслуживании». Убедитесь, что настройки производительности сервера оптимизированы.
- Никогда не предоставляйте неадминистративным учётным записям (за исключением `NT SERVICE\WMSvc`) доступ на чтение или на запись к applicationHost.config, redirection.config и administration.config. Включая `Network Service`, `IIS_IUSRS`, `IUSR`, или любую пользовательскую учётную запись, используемую пулами приложений IIS. Рабочие процессы IIS не предназначены для прямого доступа к какому-либо из этих файлов.
- Никогда не давайте общий доступ к applicationHost.config, redirection.config, and administration.config в сети. При использовании Shared Configuration предпочтительнее экспортировать applicationHost.config в другое место (см. раздел в документации IIS «Настройка разрешений для общей конфигурации»).
- Учтите, что все пользователи по умолчанию могут читать файл .NET Framework `machine.config` и имеют права администратора на файл `web.config`. Не храните чувствительную информацию в этих файлах, если она предназначена только для администратора.
- Шифруйте чувствительную информацию, которая должна быть доступна только рабочим процессам IIS, а не другим пользователям компьютера.
- Не предоставляйте доступ на запись к учётной записи, которую web-сервер использует для доступа к общему `applicationHost.config`. Она должна иметь доступ только на чтение.
- Используйте отдельную учётную запись для публикации applicationHost.config на общем ресурсе. Не используйте её для настройки доступа к общей конфигурации на web-серверах.
- Используйте надёжный пароль при экспорте ключей шифрования для общей конфигурации.
- Обеспечьте ограниченный доступ к ресурсу, содержащему общую конфигурацию и ключи шифрования. Если этот общий ресурс будет скомпрометирован, злоумышленник сможет прочитать и записать любую конфигурацию IIS для web-сервера, перенаправить трафик с вашего сайта на вредоносные источники, а в некоторых случаях получить контроль над всеми web-серверами, загрузив произвольный код в рабочие процессы IIS.
- Рассмотрите возможность защиты этого общего ресурса с помощью правил межсетевого экрана и IPSec, чтобы разрешать подключение к нему только web-серверам-участникам.

#### Ведение журнала событий

Ведение журнала является важным элементом безопасности архитектуры приложений, поскольку его можно использовать для обнаружения недостатков в приложениях (например, пользователи постоянно пытаются скачать файл, которого нет), а также для предотвращения постоянных атак со стороны мошенников. Журналы обычно неплохо формируются web- и прочими серверами. Но нечасто можно найти такие web-приложения, которые должным образом регистрируют свои действия в журнале, и даже когда они это делают, основная их цель — вывести отладочную информацию, которая нужны программистом для анализа конкретной ошибки.

В обоих случаях (журналы сервера и приложения) необходимо протестировать и проанализировать несколько вопросов исходя из содержимого журнала:

1. Содержат ли журналы чувствительную информацию?
2. Хранятся ли журналы на выделенном сервере?
3. Может ли использование журнала привести к отказу в обслуживании?
4. Как происходит ротация журналов? Достаточен ли срок их хранения?
5. Как анализируются журналы? Могут ли администраторы применять этот анализ для обнаружения целевых атак?
6. Как хранятся резервные копии журналов?
7. Контролируются ли данные перед записью в журнал (минимальная/максимальная длина, допустимые символы и т.д.)?

##### Чувствительная информация в журналах событий

Некоторые приложения могут, например, использовать запросы GET для передачи данных из формы, которые будут отображаться в журналах сервера. Это означает, что журналы сервера могут содержать конфиденциальную информацию (например, имена пользователей в качестве паролей или данные банковского счета). Эта информация может быть использована злоумышленником не по назначению, если он получил журналы, например, через административные интерфейсы или известные уязвимости web-сервера или неправильную конфигурацию (например, хорошо известный недостаток конфигурации `server-status` в HTTP-серверах на основе Apache).

Журналы событий часто содержат данные, которые могут быть полезны злоумышленнику (утечка информации) или могут быть использованы непосредственно в уязвимостях:

- Отладочная информация
- Трассировка стека
- Имена пользователей
- Названия компонентов системы
- Внутренние IP-адреса
- Персональные данные (например, email, почтовые адреса и номера телефонов, связанные с указанными лицами)
- Бизнес-данные

Кроме того, в некоторых юрисдикциях хранение конфиденциальной информации в файлах журналов, например персональных данных, может обязать предприятие применять к файлам журналов те же законы о защите данных, которые они применили бы к своим внутренним базам данных. И невыполнение этого требования, даже по незнанию, может повлечь за собой наказание в соответствии с действующими законами о защите данных.

Более широкий список конфиденциальной информации:

- Исходный код приложения
- Идентификаторы сессии
- Токены доступа
- Персональные данные и информацию, позволяющую установить личность (PII)
- Пароли для аутентификации
- Строки подключения к базам данных
- Ключи шифрования
- Данные о владельце банковского счета или платежной карты
- Информация с более высокой категорией конфиденциальности, чем разрешено хранить в системе ведения журнала
- Коммерческая тайна
- Информация, сбор которой является незаконным в соответствующей юрисдикции
- Информация, от сбора которой пользователь отказался или не давал согласия, например. использование запрета на отслеживание или когда срок действия согласия на её обработку истёк.

#### Размещение журнала

Обычно серверы формируют локальные журналы своих действий и ошибок, используя диск системы, на которой работают. Однако, если сервер скомпрометирован, его журналы могут быть стёрты злоумышленником, чтобы очистить все следы его атаки и методов взлома. Если это произойдёт, системный администратор не узнает, как произошла атака или где находится её источник. На самом деле, в большинство наборов инструментов для нападения входит log zapper, который способен очищать любые журналы, содержащие заданную информацию (например, IP-адрес злоумышленника), что обычно и делает rootkit на системном уровне.

Следовательно, разумнее хранить журналы в отдельном месте, а не на самом web-сервере. Это также упрощает агрегирование журналов из разных источников, относящихся к одному и тому же приложению (например, с фермы web-серверов), а также упрощает анализ журналов (который может сильно нагружать процессор), не затрагивая сам сервер.

#### Хранение журналов

Журналы могут привести к отказу в обслуживании, если они не хранятся должным образом. Любой злоумышленник с достаточными ресурсами может создать такое количество запросов, которое заполнит всё отведённое место для журналов, если это намеренно не запретить. Если сервер настроен неправильно, файлы журналов будут храниться в том же разделе диска, который используется для операционной системы или самого приложения. Если диск будет переполнен, операционная система или приложение могут выйти из строя, потому что они не смогут записывать на диск.

Обычно в системах UNIX журналы размещаются в /var (хотя иногда могут находиться и в /opt или /usr/local), и важно убедиться, что каталоги, в которых хранятся журналы, находятся в отдельном разделе диска. В некоторых случаях, а также для предотвращения воздействия на системные журналы, каталог журналов самого сервера (например, /var/log/apache на web-сервере Apache) должен храниться в выделенном разделе диска.

Это не означает, что размер журналов может неограниченно расти и заполнить всю файловую систему, в которой они находятся. Необходимо контролировать размер журналов, чтобы обнаружить это условие, поскольку оно может указывать на атаку.

Тестирование этого условия в эксплуатационных средах так же просто как и опасно, как запустить достаточное и постоянное количество запросов, чтобы убедиться, регистрируются ли они, и есть ли возможность переполнить ими раздел журнала на диске. В некоторых средах, где параметры QUERY_STRING регистрируются независимо от того, созданы ли они с помощью запросов GET или POST, можно моделировать большие запросы, которые будут заполнять место быстрее, поскольку, как правило, один запрос приводит к записи небольшого объёма данных, таких как дата и время, IP-адрес источника, запрос URI и результат сервера.

#### Ротация журналов

Большинство серверов (но немногие пользовательские приложения) проводят ротацию журналов, чтобы они не заполняли всю файловую систему, в которой находятся. При ротации предполагается, что содержащаяся в журналах информация нужна только в течении ограниченного периода времени.

Эта функция должна быть протестирована, чтобы убедиться, что:

- Журналы хранятся в течение времени, определённого в политике безопасности, не больше и не меньше.
- Журналы сжимаются после ротации (это удобно, т.к. на одном и том же доступном дисковом пространстве будет храниться больше журналов).
- Разрешения файловой системы для ротируемых файлов журнала такие же (или более строгие), что и для самих файлов журнала. Например, web-серверы пишут в журналы, которые они используют, но им не нужно писать в ротируемые журналы, таким образом разрешения файлов могут быть изменены только на чтение, чтобы предотвратить их изменение процессом web-сервера при ротации.

Некоторые серверы могут ротировать журналы при достижении заданного размера. Если это так, необходимо убедиться, что злоумышленник не сможет принудительно ротировать журналы, чтобы скрыть свои следы.

#### Контроль доступа к журналам

Информация журнала событий не должна быть видна конечным пользователям. Даже web-администраторы не должны иметь возможности просматривать такие журналы, поскольку это нарушает правило разделения обязанностей. Убедитесь, что механизм управления доступом, используемый для защиты необработанных журналов и приложения для просмотра или поиска в журналах, не связаны с механизмом управления доступом для ролей пользователей в других приложениях. Кроме того, данные журналов не должны быть доступны для просмотра пользователям, не прошедшим аутентификацию.

#### Анализ журнала событий

Анализ журналов может использоваться не только для извлечения статистики использования файлов на web-серверах (на что обычно нацелено большинство приложений анализа журналов), но и для определения того, имели ли место быть атаки на web-сервер.

Для анализа атак на web-сервер необходимо проверить файлы журнала ошибок сервера. Анализ должен быть сосредоточен на:

- 400-х (not found) сообщениях об ошибках. Большое их количество из одного и того же источника может указывать на то, что против web-сервера отработал CGI-сканер
- 500-х (server error) сообщениях об ошибках. Может свидетельствовать о том, что нарушитель злоупотребляет частями приложения, которые неожиданно выходят из строя. Например, первые этапы атаки с инъекцией SQL-кода будут выдавать 500-ю ошибку, если SQL-запрос построен неправильно и его выполнение в базе данных завершается сбоем.

Статистика или анализ журналов не должны формироваться или храниться на том же сервере, на котором хранятся сами журналы. В противном случае злоумышленник может из-за уязвимости web-сервера или его неправильной конфигурации получить к ним доступ и ту же информацию, что раскрывается в самих журналах.

## Ссылки

- Apache
    - Apache Security, by Ivan Ristic, O’reilly, March 2005.
    - [Apache Security Secrets: Revealed (Again), Mark Cox, November 2003](https://awe.com/mark/talks/apachecon2003us.html)
    - [Apache Security Secrets: Revealed, ApacheCon 2002, Las Vegas, Mark J Cox, October 2002](https://awe.com/mark/talks/apachecon2002us.html)
    - [Оптимизация производительности](https://httpd.apache.org/docs/current/misc/perf-tuning.html)
- Lotus Domino
    - Lotus Security Handbook, William Tworek et al., April 2004, available in the IBM Redbooks collection
    - Lotus Domino Security, an X-force white-paper, Internet Security Systems, December 2002
    - Hackproofing Lotus Domino Web Server, David Litchfield, October 2001
- Microsoft IIS
    - [Рекомендации по безопасности для IIS 8](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj635855(v=ws.11))
    - [CIS Microsoft IIS Benchmarks](https://www.cisecurity.org/benchmark/microsoft_iis/)
    - Securing Your Web Server (Patterns and Practices), Microsoft Corporation, January 2004
    - IIS Security and Programming Countermeasures, by Jason Coombs
    - From Blueprint to Fortress: A Guide to Securing IIS 5.0, by John Davis, Microsoft Corporation, June 2001
    - Secure Internet Information Services 5 Checklist, by Michael Howard, Microsoft Corporation, June 2000
- Red Hat (ранее Netscape) iPlanet
    - Guide to the Secure Configuration and Administration of iPlanet Web Server, Enterprise Edition 4.1, by James M Hayes, The Network Applications Team of the Systems and Network Attack Center (SNAC), NSA, January 2001
- WebSphere
    - IBM WebSphere V5.0 Security, WebSphere Handbook Series, by Peter Kovari et al., IBM, December 2002.
    - IBM WebSphere V4.0 Advanced Edition Security, by Peter Kovari et al., IBM, March 2002.
- Универсальное
    - [Памятка по журналам](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html), OWASP
    - [SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final) Руководство по безопасному журналированию, NIST
    - [PCI DSS v3.2.1](https://ru.pcisecuritystandards.org/_onelink_/pcisecurity/en2ru/minisite/en/docs/PCI_DSS_v3-2-1_RU.pdf) см. требование 10 и требование 4 в PA-DSS v3.2 (с октября 2022 разделён между двумя стандартами PCI Secure Software Standard и PCI Secure SLC Standard), PCI Security Standards Council

- Общее:
    - [Модули укрепления безопасности CERT: защита общедоступных web-серверов, 2000](https://resources.sei.cmu.edu/asset_files/SecurityImprovementModule/2000_006_001_13637.pdf)
