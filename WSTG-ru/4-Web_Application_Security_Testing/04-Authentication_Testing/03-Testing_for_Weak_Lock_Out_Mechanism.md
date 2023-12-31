---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование механизма блокировки

|ID          |
|------------|
|WSTG-ATHN-03|

## Обзор

Механизмы блокировки учётных записей используются для сдерживания атак методом перебора. Вот некоторые из атак, с которыми можно бороться с помощью механизма блокировки:

- Атака по угадыванию пароля или имени пользователя для входа.
- Угадывание кода при двухфакторной аутентификации или ответов на «секретные вопросы».

Механизмы блокировки учётных записей требуют баланса между защитой от несанкционированного доступа и защитой от отказа в легитимном доступе. Учётные записи обычно блокируются после 3-5 неудачных попыток и могут быть разблокированы только через определённый период времени с помощью механизма разблокировки самостоятельно или после вмешательства администратора.

Несмотря на простоту проведения атак методом перебора, результат успешной атаки опасен, поскольку злоумышленник получает полный доступ к учётной записи пользователя, а вместе с ней и ко всем функциям и сервисам, к которым у него есть доступ.

## Задачи тестирования

- Оценить способность механизмов блокировки учётных записей противостоять возможности подбора пароля методом перебора.
- Оценить устойчивость механизма разблокировки к несанкционированной разблокировке учётной записи.

## Как тестировать

### Механизм блокировки

Чтобы проверить надёжность механизмов блокировки, вам потребуется доступ к учётной записи, которую вы хотите или можете позволить себе заблокировать. Если у вас есть только одна учётная запись, с помощью которой вы можете войти в web-приложение, проведите этот тест в конце тестирования, чтобы избежать потери времени из-за блокировки.

Чтобы оценить способность механизма блокировки учётной записи противостоять возможности подбора пароля методом перебора, попробуйте войти, используя неправильный пароль несколько раз, прежде чем использовать правильный для проверки того, что учётная запись была заблокирована. Пример теста может быть следующим:

1. Попытайтесь войти в систему с неправильным паролем 3 раза.
2. Войдите в систему с правильным паролем, тем самым подтвердив, что механизм блокировки не срабатывает после 3 неуспешных попыток аутентификации.
3. Попытайтесь войти в систему с неправильным паролем 4 раза.
4. Войдите в систему с правильным паролем, тем самым показывая, что механизм блокировки не срабатывает после 4 неуспешных попыток аутентификации.
5. Попытайтесь войти в систему с неправильным паролем 5 раз.
6. Попытайтесь войти в систему с правильным паролем. Приложение выдаёт сообщение «Ваша учётная запись заблокирована», тем самым подтверждая, что учётная запись блокируется после 5 неуспешных попыток аутентификации.
7. Попробуйте войти в систему с правильным паролем через 5 минут. Приложение выдаёт сообщение «Ваша учётная запись заблокирована», тем самым показывая, что механизм блокировки не разблокируется автоматически через 5 минут.
8. Попробуйте войти в систему с правильным паролем через 10 минут. Приложение выдаёт сообщение «Ваша учётная запись заблокирована», тем самым показывая, что механизм блокировки не разблокируется автоматически через 10 минут.
9. Успешно войдите в систему с правильным паролем через 15 минут, показывая, что механизм блокировки автоматически разблокируется после 10-15 -минутного периода.

CAPTCHA может препятствовать атакам методом перебора, но может иметь свои недостатки, поэтому не должна заменять механизм блокировки. Механизм CAPTCHA можно обойти, если он реализован неправильно. Недостатки CAPTCHA включают в себя:

1. Легко решаемая задача, арифметическая или ограниченный набор вопросов.
2. CAPTCHA проверяет наличие кода HTTP-ответа вместо его статуса (успех или неудача).
3. Логика CAPTCHA на стороне сервера по умолчанию соответствует успешному решению.
4. Ответ на CAPTCHA не проверяется на стороне сервера.
5. Поле для ввода или параметр CAPTCHA обрабатываются вручную и подтверждаются неправильно или экранируются.

Для оценки эффективности CAPTCHA:

1. Оцените задания в CAPTCHA и попытайтесь автоматизировать их решение в зависимости от сложности.
2. Попытайтесь отправить запрос, не решая CAPTCHA через имеющийся интерфейс пользователя.
3. Попытайтесь отправить запрос с заведомо неправильным ответом на CAPTCHA.
4. Попытайтесь отправить запрос на сервер с использованием прокси, не решая CAPTCHA (предполагая, что значения по умолчанию можно передать через код на стороне клиента и т.д.).
5. Попытайтесь пофаззить точки входа данных для CAPTCHA (при наличии) с распространёнными векторами полезной нагрузки для инъекций или последовательностями специальных символов.
6. Проверьте, может ли быть решением для CAPTCHA альтернативный текст изображения(й), имя файла(ов) или значение в соответствующем скрытом поле.
7. Попытайтесь повторно передать ранее выявленные правильные ответы.
8. Проверьте, не приводит ли очистка файлов cookie к обходу CAPTCHA (например, если CAPTCHA отображается только после нескольких неудачных решений).
9. Если CAPTCHA является частью многоступенчатого процесса, попробуйте просто получить доступ или выполнить шаг, следующий после CAPTCHA (например, если CAPTCHA является первым шагом в процессе входа в систему, попробуйте просто отправить сразу второй шаг [имя пользователя и пароль]).
10. Проверьте наличие альтернативных методов, в которых может не применяться CAPTCHA, таких как точка входа для API, предназначенная для облегчения доступа к мобильным приложениям.

Повторите этот процесс для всех возможных функций, для которых может потребоваться механизм блокировки.

### Механизм разблокировки

Чтобы оценить устойчивость механизма к несанкционированной разблокировке учётной записи, запустите этот механизм и найдите в нём изъяны. Типичные механизмы разблокировки могут включать секретные вопросы или отправляемую по электронной почте ссылку для разблокировки. Такая ссылка должна быть уникальной и одноразовой, чтобы злоумышленник не смог угадать или воспроизвести её и провести пакетные атаки методом перебора.

Обратите внимание, что механизм разблокировки следует использовать только для разблокировки учётных записей. Это не то же самое, что механизм восстановления пароля, но он может следовать тем же правилам безопасности.

## Меры защиты

Применять механизмы разблокировки учётной записи в зависимости от уровня риска. Варианты в порядке увеличения доверия:

1. Временная блокировка и разблокировка.
2. Авторазблокировка (например, отправка электронного письма для разблокировки на зарегистрированный адрес электронной почты).
3. Ручная разблокировка администратором.
4. Ручная разблокировка администратором после положительной идентификации пользователя.

Факторы, которые следует учитывать при реализации механизма блокировки учётных записей:

1. Каков для приложения риск угадывания пароля методом перебора?
2. Достаточно ли CAPTCHA для снижения этого риска?
3. Используется ли механизм блокировки на стороне клиента (например, JavaScript)? (Если это так, отключите код на стороне клиента для тестирования.)
4. Количество неудачных попыток входа в систему до блокировки. Чем ниже порог блокировки, тем чаще могут быть заблокированы действительные пользователи. Чем выше порог блокировки, тем больше попыток даётся злоумышленнику для взлома учётной записи до её блокирования. В зависимости от цели приложения типичным порогом блокировки является диапазон от 5 до 10 неудачных попыток.
5. Как происходит разблокировка?
    1. Вручную администратором. — Наиболее безопасный метод, но он может доставлять неудобства пользователям и отнимать «драгоценное» время у администратора.
        1. Обратите внимание, что у самого администратора также должен быть способ восстановления на случай блокировки его учётной записи.
        2. Если целью злоумышленника является блокировка учётных записей всех пользователей web-приложения, то этот механизм разблокировки может привести к атаке типа «отказ в обслуживании».
    2. По истечении определённого периода времени. — Какова продолжительность блокировки? Достаточно ли её для защиты приложения? Блокировка от 5 до 30 минут может стать неплохим компромиссом между сдерживанием атак методом перебора и причинением неудобств законным пользователям.
    3. С помощью механизма самообслуживания. — Как указывалось ранее, механизм авторазблокировки должен быть достаточно безопасным, чтобы злоумышленник не мог самостоятельно разблокировать учётные записи.

## Ссылки

- См. статью OWASP по атакам [методом перебора](https://owasp.org/www-community/attacks/Brute_force_attack).
- [Памятка OWASP по забытым паролям](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html).
