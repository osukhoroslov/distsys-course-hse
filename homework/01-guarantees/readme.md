# Гарантии доставки

В данной задаче вам предстоит реализовать различные гарантии доставки сообщений в распределенной системе.

В нашей системе есть два узла, и нам требуется организовать одностороннюю передачу текстовых сообщений между выполняющимися на узлах процессами. Процесс _sender_ будет принимать сообщения от локального пользователя _S_ и отправлять их по сети процессу _receiver_. Процесс receiver будет принимать сообщения от sender-а и доставлять их локальному пользователю _R_. Под доставкой подразумевается отправка локального сообщения, идентичного исходному сообщению от _S_.

Вам необходимо написать четыре реализации _sender_ и _receiver_, обеспечивающие следующие гарантии по доставке сообщений:

1. _Не более одного раза (at most once)._ Каждое сообщение от _S_ должно быть или доставлено _R_ ровно один раз или не доставлено вовсе. Иными словами, нельзя допускать повторные доставки сообщений. 
2. _Не менее одного раза (at least once)._ Каждое сообщение от _S_ должно быть доставлено _R_, при этом допускаются повторы.
3. _Ровно один раз (exactly once)._ Каждое сообщение от _S_ должно быть доставлено _R_ ровно один раз, то есть повторы не допускаются.
4. _Ровно один раз и с сохранением порядка (exactly once + ordered)._ Каждое сообщение от _S_ должно быть доставлено _R_ ровно один раз и в порядке их отправки _S_.

Процессы могут взаимодействовать друг с другом путем обмена сообщениями через сетевой транспорт со следующими характеристиками: доставка сообщений не гарантируется, доставленные сообщения не искажаются, сообщения могут дублироваться, сохранение порядка при приеме сообщений не гарантируется, все получаемые сообщения были кем-то отправлены (нет сообщений "из воздуха"). Также будем предполагать, что каждое сообщение рано или поздно достигнет получателя, если отправитель будет повторять попытки передать сообщение. В тестах это предположение соблюдается. Отказы узлов в данной задаче отсутствуют.

Независимо от типа гарантий, в случае если сеть ведет себя надёжно (нет потерь сообщений) все сообщения от _S_ должны доставляться _R_. Это исключает, например, тривиальную реализацию гарантии 1, которая не отправляет ничего по сети.

Ваша реализация не должна делать предположений об уникальности содержимого доставляемых сообщений. Например, в разные моменты времени могут быть отправлены два сообщения с идентичным текстом. В этом случае для гарантии 2 требуется доставить эти сообщения пользователю не менее двух раз, а для гарантий 3 и 4 - ровно два раза. Также не следует делать предположений о размере сообщений - он может быть произвольным.

Несложно заметить, что реализация последней, самой сильной гарантии покрывает первые три гарантии. Тем не менее мы просим вас реализовать каждую гарантию отдельно. Так вы наглядно увидите какие дополнительные ресурсы и накладные расходы требуются для поддержки той или иной гарантии. На практике не всегда требуются самые сильные гарантии, например может быть неважен порядок доставки и "платить" за него будет расточительно. Поэтому если вы реализуете только последнюю гарантию, то мы не зачтём вам остальные.

На максимальный балл требуется также оптимизировать ресурсы, потребляемые для обеспечения каждой из гарантий. Во-первых, это объем памяти, используемой процессами, то есть размер хранимого ими состояния. Тесты на эту часть задания сделаны так, что если просто запоминать все сообщения (или их идентификаторы), то памяти не хватит. Попробуйте организовать данные таким образом, чтобы не хранить лишнюю или неактуальную информацию (например, если доставлены все 100 предыдущих сообщений, то так ли необходимо хранить все 100 идентификаторов сообщений?). Во-вторых, это число и суммарный объем (трафик) сообщений, передаваемых по сети между _sender_ и _receiver_. Постарайтесь не передавать по сети лишних данных или отправлять слишком много сообщений, особенно если они могут быть отброшены получателем, то есть переданы по сети зря. В процессе оптимизации вы должны осознать возникающие при такой оптимизации компромиссы между потреблением памяти, нагрузкой на сеть и производительностью (например, скоростью доставки сообщений).

## Реализация

Для реализации и тестирования решения используется учебный фреймворк dslab-mp (см. материалы первого семинара). В папке задания размещена заготовка для решения [solution.py](solution.py). Вам надо доработать реализации классов `...Sender` и `...Receiver` для каждой гарантии так, чтобы они проходили все тесты.

### Sender

Сообщения от _S_ передаются процессу с помощью локальных сообщений, см. метод `on_local_message()`. Все сообщения имеют тип `MESSAGE` и одинаковую структуру - в единственном поле `text` содержится строка с текстом сообщения. Для взаимодействия с _receiver_-ом вы можете использовать сообщения произвольного типа и структуры. Приходящие от _receiver_-а сообщения следует обрабатывать в методе `on_message()`. Также вы можете устанавливать таймеры в любом из методов и обрабатывать их наступление в `on_timer()`.

### Receiver

Данный процесс не принимает локальные сообщения, поэтому метод `on_local_message()` не используется. Сетевые сообщения следует обрабатывать в методе `on_message()`. Также вы можете устанавливать таймеры в любом из методов и обрабатывать их наступление в `on_timer()`.

Важно правильно реализовать доставку сообщений локальному пользователю _R_, иначе тесты не будут проходить. Для этого вы должны отправить локальное сообщение с помощью метода `ctx.send_local()`. Сообщение должно быть полностью идентично исходному сообщению, принятому sender-ом от его пользователя _S_, то есть иметь тот же тип `MESSAGE` и поле `text` с тем же значением. Других полей в сообщении быть не должно.

## Тестирование

Перед запуском тестов убедитесь, что на вашей машине [установлен Rust](https://www.rust-lang.org/tools/install). 

Тесты находятся в папке `tests`. Для запуска тестов перейдите в эту папку и выполните команду: `cargo run --release -- -d`. Вывод тестов содержит трассы (последовательности событий во время выполнения каждого из тестов), а также финальную сводку. 

Доступные опции можно посмотреть с помощью флага `-h`. Опишем наиболее важные из них:
- Флаг `-d` включает вывод трасс. Его рекомендуется использовать при отладке решений.
- Опция `-m` задает количество запусков рандомизированных тестов (chaos monkey). Значение по умолчанию - 0. Как только ваше решение будет проходить основные тесты, установите значение в 10 и убедитесь, что эти тесты проходят. Далее можно проверить решение на 100 запусках (`-d` лучше убрать для скорости) - такое значение используется при проверке вашего решения в CI. (Обратите внимание, что эти тесты хоть и рандомизированные, но детерминированные - при одном значении seed результат будет всегда одинаковый. Так что не стоит пытаться заново отправлять то же решение в CI, надеясь что оно вдруг пройдет.)
- Флаг `-c` включает тесты на model checking (см. первый семинар), по умолчанию они выключены. Как только ваше решение будет проходить основные тесты, добавьте этот флаг и убедитесь, что эти тесты также проходят.
- Флаг `-o` включает тесты на потребление ресурсов (памяти и сети), по умолчанию они выключены. В этих тестах измеряются и выводятся максимальное потребление памяти объектами Sender и Receiver, число переданных по сети сообщений и их суммарный объем (трафик). Полученные значения сравниваются с пороговыми значениями, в которые укладывается оптимизированное решение. Как только ваше решение будет проходить основные тесты, chaos monkey и model checking, включите эти тесты и при необходимости займитесь оптимизацией решения.
- Опция `-t` позволяет прогнать только один конкретный тест, указав его имя (в точности как оно выводится в консоли, например `[AT MOST ONCE] NORMAL`).
- Опция `-g` позволяет прогнать только тесты для одной из гарантий, указав её сокращение (`AMO`, `ALO`, `EO`, `EOO`).
- Опция `-s` позволяет изменить используемый random seed (см. первый семинар). Можно использовать для дополнительной проверки вашего решения. В CI тесты запускаются со значением по умолчанию (123).

При тестировании в CI тесты запускаются так: `cargo run --release -- -m 100 -c -o`. На авторском решении выполнение всех тестов занимает менее 10 секунд.

Код тестов открыт и находится в `tests/src/main.rs`. Вы можете обращаться к нему и использовать информацию об условиях тестирования, например максимальной задержке в сети, в своем решении. Главное, чтобы гарантии обеспечивались всегда, вне зависимости от условий запуска, например вероятности потери сообщений в сети.

Если вы найдете ошибки или требования из условий, которые не покрывают наши тесты, то вы можете получить за это бонусы. Для этого надо включить в отчёт описание ситуации, которую не ловят тесты, добавив при необходимости пример решения с ошибкой. За это полагается 1 балл. Если вы также реализуете тесты, которые ловят найденную проблему, или хотя бы опишите их логику, то получите еще 1 балл. Готовые тесты оформляйте как pull request в репозиторий курса.

## Оценивание

Компоненты задачи и их вклад в оценку:
- Краткий отчёт с описанием вашего решения в файле `solution.md` - обязательно, без него проверка производиться не будет.
- Корректная реализация гарантии _at most once_ - 2 балла
  - проходят все тесты на эту гарантию без "OVERHEAD..."
- Корректная реализация гарантии _at least once_ - 2 балла
  - проходят все тесты на эту гарантию без "OVERHEAD..."
- Корректная реализация гарантии _exactly once_ - 2 балла
  - проходят все тесты на эту гарантию без "OVERHEAD..."
- Корректная реализация гарантии _exactly once + ordered_ - 2 балла
  - проходят все тесты на эту гарантию без "OVERHEAD..."
- Оптимизация потребляемых ресурсов - 2 балла*
  - проходят тесты "OVERHEAD..." для всех гарантий (1 балл)
  - в отчёте описаны возникающие при оптимизации компромиссы и обоснованы используемые подходы (1 балл)
  - *засчитывается только при успешном выполнении всех предыдущих компонентов

Штрафы:
- Для гарантии A используется реализация более сильной гарантии B - минус 1 балл за гарантию A

## Сдача

Следуйте стандартному [порядку сдачи заданий](../readme.md).

## ЧаВо

**Как измеряется потребление памяти в тестах на overhead и что в него входит?**

См. [здесь](https://github.com/osukhoroslov/dslab/blob/46de51c4e38f80671d82e30426d665e50b5d0691/crates/dslab-mp-python/src/lib.rs#L202). Данной функции передается объект Sender или Receiver. Измеряется потребление памяти структурами данных (атрибутами) внутри этих объектов. Функция вызывается периодически по ходу выполнения теста, и запоминается максимальное полученное значение, которое выводится в конце теста. Затраты на глобальные переменные и таймеры не учитываются. 

**Что можно и что нельзя использовать для хранения состояния процесса?**

Можно и нужно использовать атрибуты объектов Sender и Receiver.

Нельзя использовать:
- глобальные переменные (кроме иммутабельных данных типа констант),
- имена таймеров (не пытайтесь хранить в них содержимое сообщений, допускается хранить номера сообщений),
- отправку процессом самому себе сообщений с его состоянием.

Всё перечисленное будет рассматриваться как попытка обойти тесты (чит), и пункт с оптимизацией потребляемых ресурсов засчитан не будет. Авторское решение не использует подобных ухищрений и хранит всё в атрибутах процессов.

Также для прохождения тестов на overhead не требуются сторонние библиотеки типа numpy, сжатие и распаковка данных, специальные бинарные форматы и т.п. Авторское решение не использует ничего из перечисленного.

Если вы не понимаете, как пройти тесты без вышеописанных хаков, перечитайте условие, там есть намёк.

**Допускается ли падение тестов на overhead с другим random seed?**

Да. Пороги в тестах указаны не вообще для всех возможных выполнений (где потребление ресурсов может довольно сильно отличаться), а только для запусков с дефолтным seed.
