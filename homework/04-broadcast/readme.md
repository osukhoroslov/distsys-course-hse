# Надежная упорядоченная рассылка

В данной задаче вам предстоит реализовать механизм рассылки сообщений с описанными далее гарантиями по надежности и порядку доставки. Выполнение и тестирование решения будет происходить в фреймворке dslab-mp, уже знакомом вам по задаче 1.

Пусть имеется распределенное приложение типа чата, пользователи которого общаются путем обмена сообщениями. Для простоты будем считать, что число пользователей фиксировано, чат один и сообщения в нём доставляются всем пользователям. С каждым пользователем связан процесс, который занимается приемом сообщений от пользователя, рассылкой их остальным процессам в системе, а также доставкой своему пользователю полученных от других процессов сообщений. Процессы выполняются на разных узлах системы, связанных сетью. Как обычно, взаимодействие с пользователем (прием и доставка сообщений) реализуются в dslab-mp с помощью локальных сообщений. Также будем считать, что каждое отправляемое сообщение уникально в пределах рассматриваемого выполнения системы (например, в течение теста).

Процессы взаимодействуют путем обмена сообщениями по сети. Изначально вам доступна только отправка сообщения одному процессу (unicast), поверх чего вам надо реализовать рассылку сообщения всем процессам (broadcast). Передача сообщений по сети происходит через надежный транспорт, гарантирующий exactly once доставку. Однако узлы и процессы на них могут быть подвержены отказам типа внезапной остановки (падения), иногда в самый неподходящий момент, например во время рассылки сообщения. При этом, даже если процесс успел отправить сообщение до того как упал, но сообщение еще не было принято получателем, то доставка этого сообщения уже _не гарантируется_. Ведь, как мы уже знаем, реализация надежного транспорта требует непосредственного участия отправителя в процессе передачи. Будем считать, что отказавший процесс останавливается навсегда, то есть не возвращается в систему. Если в ходе выполнения процесс не отказал, будем называть его _корректным_. Также будем считать, что в ходе выполнения корректные процессы всегда составляют большинство. Например, если процессов (и узлов в системе) пять, то отказать могут не более двух. (Подсказка: это требуется для реализации свойства 4 ниже.)

Ваша реализация рассылки сообщений должна удовлетворять следующим свойствам:

1. **No Duplication:** Сообщения доставляются пользователю не более одного раза (нет повторов).
2. **No Creation:** Если пользователю доставлено сообщение _m_ от пользователя _u_, то _m_ было ранее отправлено _u_.
3. **Validity:** Если пользователь корректного процесса _n_ отправил сообщение _m_, то _n_ должен в конце концов доставить _m_ пользователю.
4. **Uniform Agreement:** Если сообщение _m_ было доставлено некоторым процессом (необязательно корректным), то _m_ будет в конце концов доставлено каждым корректным процессом.
5. **Causal Order:** Сообщения доставляются с сохранением причинного порядка - если процесс _n_ доставил сообщение _m_ от пользователя _u_, то перед этим _n_ должен доставить все сообщения, которые могли повлиять на _m_ (сообщения, отправленные или доставленные _u_ до отправки им _m_).

Часть из этих свойств уже обеспечивается в силу описанных выше предположений, остальные вам надо реализовать самому. Другие характеристики вашей реализации (накладные расходы, масштабируемость, производительность и т.п.) в требования задачи не входят.

Помимо реализации надо написать отчёт с описанием вашего алгоритма и обоснованием достижения им всех требуемых свойств. Строгих доказательств не требуется, достаточно чтобы ваше обоснование было понятным и убедительным. Также в отчёт можно включить описание возможных оптимизаций решения в плане других характеристик. Например, как обеспечить масштабируемость рассылки при росте числа узлов без или с минимальными жертвами требуемых свойств.

## Реализация

Для реализации и тестирования решения используется учебный фреймворк dslab-mp (см. материалы первого семинара). В папке задачи размещена заготовка для решения [solution.py](solution.py). Вам надо доработать реализацию `BroadcastProcess` так, чтобы проходили все тесты.

При инициализации процессу передается его уникальный id (он же является id локального пользователя), а также список id всех процессов в системе. Отправка сообщения пользователем реализована с помощью локального сообщения _SEND_ с полем `text`. Для отображения сообщения пользователю надо использовать локальное сообщение _DELIVER_ c аналогичным полем `text` (см. заготовку). Для взаимодействия между процессами вы можете использовать любые собственные типы сообщений с произвольной структурой.

Ваша реализация не должна делать предположений о максимальных временах доставки сообщений или пытаться определить жив ли некоторый процесс или упал. Позже в курсе мы научимся реализовывать детектор отказов, в этой задаче вам надо обойтись без него. Также попробуйте обойтись без таймеров или по крайней мере не делайте их бесконечно повторяющимися - иначе тесты зависнут.

## Тестирование

Перед запуском тестов убедитесь, что на вашей машине [установлен Rust](https://www.rust-lang.org/tools/install). 

Тесты находятся в папке `tests`. Для запуска тестов перейдите в эту папку и выполните команду: `cargo run --release -- -d`. Вывод тестов содержит трассы (последовательности событий во время выполнения каждого из тестов), а также финальную сводку. Доступные опции можно посмотреть с помощью флага `-h`, часть из них уже должна быть вам знакома по задаче 1. 

По умолчанию тесты запускаются на системе из пяти процессов. В GitLab CI число тестов `CHAOS MONKEY` увеличено до 100: `cargo run -- --m 100`. Тест `SCALABILITY` измеряет масштабируемость вашего решения как зависимость числа сетевых сообщений от числа процессов. Он не влияет на оценку, но может быть полезен при оптимизации масштабируемости решения на дополнительный балл.

Если вы найдете ошибки или требования из условий, которые не покрывают наши тесты, то вы можете получить за это бонусы. Для этого надо включить в отчёт описание ситуации, которую не ловят тесты, добавив при необходимости пример решения с ошибкой. За это полагается 1 балл. Если вы также реализуете тесты, которые ловят найденную проблему, или хотя бы опишите их логику, то получите еще 1 балл. Готовые тесты оформляйте как pull request в репозиторий курса.

## Оценивание

Компоненты задачи и их вклад в оценку:
- Отчёт с описанием вашего решения в файле `solution.md` - обязательно, без него проверка производиться не будет.
- Реализация удовлетворяет свойствам 1-4 (без Causal Order) - 5 баллов
- Реализация удовлетворяет всем требуемым свойствам - 7 баллов
- Отчёт содержит полное описание реализованного алгоритма - 1 балл
- Отчёт содержит понятное и убедительное обоснование достижения требуемых свойств - 1 балл
- Отчёт содержит описание нетривиальной/интересной оптимизации - 1 балл (засчитывается только при успешном выполнении всех предыдущих компонентов)

## Сдача

Следуйте стандартному [порядку сдачи заданий](../readme.md).
