---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование «пазла» сессии

|ID          |
|------------|
|WSTG-SESS-08|

## Обзор

Перегрузка переменных сессии (также известная как «пазл» сессии) — это уязвимость на уровне приложения, которая позволяет злоумышленнику выполнять различные вредоносные действия, включая, помимо прочего:

- Обход действующих механизмов принудительной аутентификации и выдачу себя за законного пользователя.
- Повышение привилегии учётной записи злоумышленника в окружении, которое в противном случае считалось бы защищённым.
- Пропуск этапов отбора в многоэтапных процессах, даже если процесс включает все обычно рекомендуемые ограничения на уровне кода.
- Манипуляцию значениями на стороне сервера косвенными методами, которые невозможно предсказать или обнаружить.
- Проведение традиционных атак там, где они ранее были невозможны или даже считались безопасными.

Эта уязвимость возникает, когда приложение использует одну и ту же переменную сессии более чем для одной цели. Злоумышленник потенциально может получить доступ к страницам в порядке, непредусмотренном разработчиками, таким образом, что переменная сессии назначается в одном контексте, а используется в другом.

Например, злоумышленник может использовать перегрузку переменных сессии, чтобы обойти механизмы принудительной аутентификации приложений, проверяя существование переменных сессии, содержащих значения, связанные с учётными данными, которые обычно сохраняются в сессии после успешной процедуры аутентификации. Это означает, что злоумышленник сначала получает доступ к месту в приложении, которое задаёт контекст сессии, а затем — к привилегированным местам, которые проверяют этот контекст.

Например, вектор атаки обхода аутентификации может быть реализован путём доступа к общедоступной точке входа (например, к странице восстановления пароля), которая присваивается той же переменной сессии на основе фиксированных значений или вводимых пользователем данных.

## Задачи тестирования

- Определить все переменные сессии.
- Нарушить логику процесса создания сессии.

## Как тестировать

### Тестирование методом чёрного ящика

Эту уязвимость можно обнаружить и эксплуатировать, перебрав все переменные сессии, используемые приложением, с учётом контекста, в котором они допустимы. В частности, это возможно путём доступа к последовательности точек входа с последующим изучением точек выхода. В случае тестирования методом чёрного ящика эта процедура сложна и требует некоторой удачи, поскольку каждая отдельная последовательность может привести к другому результату.

#### Примеры

Очень простым примером может быть функция сброса пароля, которая в точке входа может запросить у пользователя некоторую идентифицирующую информацию, такую как имя пользователя или адрес электронной почты. Затем эта страница может присвоить переменной сессии эти идентифицирующие значения, которые получены непосредственно от клиента, из запросов или определены на основе полученных входных данных. На данном этапе в приложении могут быть страницы, которые отображают личные данные клиента на основе текущего экземпляра сессии. Таким образом злоумышленник может обойти процедуру аутентификации.

### Тестирование методом серого ящика

Наиболее эффективный способ обнаружения этих уязвимостей — проверка исходного кода.

## Меры защиты

Переменные сессии следует использовать только для одной предназначенной им цели.

## Ссылки

- [Session Puzzles](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/puzzlemall/Session%20Puzzles%20-%20Indirect%20Application%20Attack%20Vectors%20-%20May%202011%20-%20Whitepaper.pdf)
- [Session Puzzling and Session Race Conditions](http://sectooladdict.blogspot.com/2011/09/session-puzzling-and-session-race.html)
- [Session Puzzling Attacks (a.k.a. “Session Variable Overloading”)](https://appcheck-ng.com/session-puzzling-attacks-a-k-a-session-variable-overloading)