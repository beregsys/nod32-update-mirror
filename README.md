ESET Nod32 Update Mirror
=========

Скрипт создания и обновления зеркала антивируса **Eset Nod32** под Linux, как с бесплатных серверов, так и официальных (для обновления с официальных серверов требуется установленный 'unrar').

### Системные требования:

  - Bash (тестировался на версиях 4.1.11(2) и 4.2.24(1));
  - Apache для раздачи файлов и показа морды (опционально, тестировалось на версиях 1.3.x и 2.2.x);
  - Наличие и разрешение на запуск '**curl**' (проверка состояния источника), '**wget**' (скачивание файлов), _'unrar' (для работы с официальными зеркалами)_ и некоторые другие стандартные приложения.

![alt tag](http://oi60.tinypic.com/20f3w21.jpg)

### Общая структура:

Функционально данное решение состоит из нескольких файлов. Ниже приведена общая структура и её описание:
- **/nod32upd** _(Директория, где располагаются сами скрипты обновления)_
  - **get-nod32-key.sh** _(Скрипт получения рабочего ключа)_
  - **upd-nod32-mirror.sh** _(Скрипт обновления зеркала)_
  - **settings.cfg** _(Настройки скриптов)_
- **/webface** _(Директория страницы-заглушки для Apache)_
  - **/.webface** _(Директория с содержимым страницы-заглушки)_
    - **header.html** _(Часть документа ДО вывода листинга)_
    - **footer.html** _(Часть документа ПОСЛЕ вывода листинга)_
    - **.htaccess** _(Настройки директории содержимого страницы-заглушки)_
  - **.htaccess** _(Разграничение доступа к зеркалу и инициализация страницы-заглушки)_

### Установка:

  1. Качаем или клонируем крайнюю версию;
  2. Если раздаем через smb — создаем директорию и открываем к ней общий доступ с правами только на чтение;
  3. Если раздаем через http — создаем отдельный virtualhost или директорию, доступную «из вне», кладем в неё содержимое директории «/webface», проверяем (открываем в браузере; должно быть похоже на изображение что выше);
  4. Кладем в директорию, **не доступную** «из вне» директорию «/nod32upd». Открываем файл «settings.cfg», прописываем пути к директориям, перепроверяем;
  5. Даем права на запуск ("chmod +x ...") для «upd-nod32-mirror.sh» и «get-nod32-key.sh» и делаем пробный запуск сперва "./get-nod32-key.sh", после этого "./upd-nod32-mirror.sh";
  6. Изучаем вывод, анализируем причину «почему сразу не заработало», идем по ссылке [задать вопрос], исправляем, радуемся;
  7. Ставим задание в крон, запускаем 3..4 раза в сутки.

### Настройка:

Для изменения рабочих путей, опций работы и всего, что связано непосредственно с обновлением - правьте /nod32upd/settings.cfg. Для определения правил доступа к зеркалу при использовании Apache - /webface/.htaccess

### Некоторые особенности:

  1. Работает как с официальными серверами, так и "пиратскими";
  2. Успешно получает "бесплатные" ключи обновлений (**Внимание - данный функционал только для примера и обучения**) и поддерживает их в актуальном состоянии (скрипт **get-nod32-key.sh**). Скрипт возвращает в крайней строке пару "логин:пароль", или слово "error" в случае ошибки. Для более подробной информации смотри его исходники и ``./get-nod32-key.sh -h``
  3. Не требует вывода индекса (списка файлов) для того, чтоб забрать файлы обновлений;
  4. Разбирает 'update.ver' и создает новый (разные сервера по разному указывают пути к файлам обновлений - встречал полные пути, относительные, только имена файлов);
  5. Имеет возможность проверки под-директорий (иногда разные сервера хранят под разные версии обновления в разных под-директориях);
  6. Имеет возможность **не** скачивать сами файлы обновлений, а лишь поддерживать в актуальном виде 'update.ver'. Удобно, если раздающий обновления сервер имеет ограниченный трафик, или недостаточно ресурсов. Такая штука работает, на удивление, даже с официальными серверами - доступ у них по прямым ссылкам к файлам _без_ авторизации (скорее всего - баг, и будет через какое-то время исправлен);
  7. Всегда используется User-Agent самого антивируса (по крайней мере - очень похожий и проходящий валидацию). Более того, при каждом запуске скрипта некоторые его части (номера версий) случайным образом генерируются, так что идентифицировать скрипт в общей куче становится ещё сложнее;
  8. Настройки лимитов скорости и паузы между запросами вынесены в секцию настроек (удобно снизить нагрузку на канал и сервер);
  9. Подробные комментарии (на ломаном английском) и приятный, подробный вывод;
  10. Скачивает **только** обновленные файлы;
  11. Работает без проблем на Apache версий 1.3.x и 2.2.x, выводит приятную взгляду мордашку (в которую встраиваются ссылки на дистрибутивы, какая-либо информация для пользователей, etc.);

### Алгоритм работы в двух словах:

  1. При необходимости (опция 'UseGetKeysScript') получаем "бесплатный" ключ;
  2. Проверяем доступность серверов (запрашивая файл 'update.ver' по указанному адресу) по порядку, указанному в настройках;
  3. При подтверждении доступности сервера - скачиваем и разбираем файл 'update.ver' выковыривая имена файлов. После чего пишем новый 'update.ver' (без путей к файлам), и 'wget'-ом выкачиваем файлы, выковыренные ранее (**только** обновленные). Если необходимо (опция 'createLinksOnly') - создаем только файл 'update.ver' с ссылками на оригинальные расположения файлов, **не** скачивая сами файлы обновлений;
  4. Проверяем под-директории из списка, указанного в настройках (алгоритм как в предыдущем пункте);
  3. Убираем за собой, создаем файл-временную метку и 'robots.txt', запрещающий индексацию.

Код не самый лучший, но хорошо откомментирован и работает.

### История изменений

* **0.3.11** - Исправление ошибки с определением пути к файлу настроек (`$(pwd)` изменен на `$(dirname $0)`), исправлен путь проверки для валидации ключа (`../mod_000_loader_1080/..` на `../mod_000_loader_1082/..`), вместо умершего `www.nod327.net` используем `tnoduse2.blogspot.ru`, для отключения ограничений wget достаточно установить значения `wget_wait_sec` и `wget_limit_rate` пустыми (или закомментировать их вовсе);
* **0.3.9** - Решение проблемы валидации ключей (скрипт `get-nod32-key.sh`);
* **0.3.8** - Решение проблемы обновления с официальных зеркал для версий v5..v7;
* **0.3.7** - Рефакторинг, вынос настроек в отдельный файл, добавление логирования в скрипт обновления, незначительные доработки и изменения;
* **0.3.6** - Чертовски мощный апдейт. В комплект добавлен скрипт "**get-nod32-key.sh**" который самостоятельно получает валидный ключ к официальным серверам. Более того - исправлена ошибка с отсутствием в пути пары логин:пароль при выполнении двух условий - 'createLinksOnly' включен и обновляемся с официального зеркала;
* **0.3.4** - Поправки в GUI, 'footer.html' и 'header.html' перенесены в '.\.webface\', добавлена поддержка 'mod_geoip' в .htaccess, сам скрипт обновления не изменен;
* **0.3.3** - Незначительные поправки;
* **0.3.2** - Незначительные поправки;
* **0.3.1** - Добавлен "цветной" вывод сообщений, добавлена опция 'createLinksOnly' которая позволяет НЕ скачивать все файлы обновления целиком, а только лишь указывать ссылки на них в 'update.ver'. Улучшено комментирование кода, исправлена пара мелких ошибок, выявленных после теста на боевом сервере. Мелкие исправления;
* **0.3** - Полностью переписан **bash** скрипт, иной алгоритм работы;
* **0.2.5-sh** - **PHP** версия более не поддерживается, скрипт переписан на **bash**;
* **0.2.5** - Масса мелких исправлений в .htaccess и верстке;
* **0.2.4** - Релиз на гитхабе.

### Лицензия:

Copyright (c) 2014 github.com/tarampampam

Данная лицензия разрешает лицам, получившим копию данного программного обеспечения и сопутствующей документации (в дальнейшем именуемыми «Программное Обеспечение»), безвозмездно использовать Программное Обеспечение без ограничений, включая неограниченное право на использование, копирование, изменение, добавление, публикацию, распространение, сублицензирование и/или продажу копий Программного Обеспечения, также как и лицам, которым предоставляется данное Программное Обеспечение, при соблюдении следующих условий:

Указанное выше уведомление об авторском праве и данные условия должны быть включены во все копии или значимые части данного Программного Обеспечения.

ДАННОЕ ПРОГРАММНОЕ ОБЕСПЕЧЕНИЕ ПРЕДОСТАВЛЯЕТСЯ «КАК ЕСТЬ», БЕЗ КАКИХ-ЛИБО ГАРАНТИЙ, ЯВНО ВЫРАЖЕННЫХ ИЛИ ПОДРАЗУМЕВАЕМЫХ, ВКЛЮЧАЯ, НО НЕ ОГРАНИЧИВАЯСЬ ГАРАНТИЯМИ ТОВАРНОЙ ПРИГОДНОСТИ, СООТВЕТСТВИЯ ПО ЕГО КОНКРЕТНОМУ НАЗНАЧЕНИЮ И ОТСУТСТВИЯ НАРУШЕНИЙ ПРАВ. НИ В КАКОМ СЛУЧАЕ АВТОРЫ ИЛИ ПРАВООБЛАДАТЕЛИ НЕ НЕСУТ ОТВЕТСТВЕННОСТИ ПО ИСКАМ О ВОЗМЕЩЕНИИ УЩЕРБА, УБЫТКОВ ИЛИ ДРУГИХ ТРЕБОВАНИЙ ПО ДЕЙСТВУЮЩИМ КОНТРАКТАМ, ДЕЛИКТАМ ИЛИ ИНОМУ, ВОЗНИКШИМ ИЗ, ИМЕЮЩИМ ПРИЧИНОЙ ИЛИ СВЯЗАННЫМ С ПРОГРАММНЫМ ОБЕСПЕЧЕНИЕМ ИЛИ ИСПОЛЬЗОВАНИЕМ ПРОГРАММНОГО ОБЕСПЕЧЕНИЯ ИЛИ ИНЫМИ ДЕЙСТВИЯМИ С ПРОГРАММНЫМ ОБЕСПЕЧЕНИЕМ.

### Ссылки

* [Пост в блоге]
* [Пост на хабре]

[Пост в блоге]:http://tmblr.co/ZYW79o1CrHcIG
[Пост на хабре]:http://habrahabr.ru/post/232163/
[задать вопрос]:https://github.com/tarampampam/nod32-update-mirror/issues/new
