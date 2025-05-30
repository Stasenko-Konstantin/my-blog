+++
date = '2025-05-21'
title = 'Проблема Rust или как и почему я повернул стрелочку RIIR'
+++

<br><br>

Забавно, но первый же результат при запросе `riir` в гугле мне выдало это: <br>
[why not Rewrite It In Rust (RIIR)](https://github.com/ansuz/RIIR).

![](https://cdn.jsdelivr.net/gh/Stasenko-Konstantin/my-blog@main/static/images/03-arrow-of-riir.png)

Я люблю трогать различные языки программирования, 
пробовать изучать их, писать на них что-либо небольшое. Обычно 
коротким тыканьем
всё и кончается когда я понимаю что что-то в языке несколько диссонирует
со мной, с моим чувством и пониманием прекрасного относительно языков. 
С `Rust` было не так. Он мне нравится, но есть одно большое но.

Я разделяю языки на две составные части:
1) Язык как инструмент. Сюда входят: компилятор,
пакетный менеджер, библиотеки, линтеры, языковой сервер и 
все остальные удобства современной разработки.
2) Язык как язык. Сюда входят: синтаксис, фичи, 
возможности и ограничения самого языка.

`Rust` по обоим критериям очень хорош. Я обожаю `cargo`, `clippy`,
`RustRover` (хоть JetBrains и контора сами знаете кого); в этом плане 
работать с `Rust` - одно удовольствие. Но и как язык `Rust` даёт фору
очень и очень многим: отсутствие ООП, первоклассная поддержка итераторов,
ориентированность на выражения (обожаю когда 
можно писать типа `if smth { 1 } else { 2 }`), система трейтов, пусть и
переусложненные (по понятным причинам), но всё же макросы, офигенские енамы
(в любви к ним я уже признавался в 
[прошлой заметке]({{< ref "02-go-adt-enums" >}})). Всё это оставляет приятное
впечатление от использования языка. 

Но как я уже сказал, есть одно большое но и это но - borrow checker. 
Возможно это просто мой skill issue, но я предпочту бороться с компилятором
`Haskell`, нежели `Rust`, хотя ощущения очень похожие. Вообще, если обобщить
еще больше, то borrow checker это скорее даже симптом, а не сама проблема.
Дело в том что `Rust` заточен под безопасное байтодробство, он сам так
позиционируется по крайней мере, поэтому использовать его для повседневных
прикладных задач в которых нет никакой нужды в выжимании всех возможных 
скоростей, а также в беспрецедентной безопасности - по меньшей мере 
неэффективно с точки зрения скорости разработки. 

Я иногда встречаю различные статьи о том как отдельные разработчики или даже
целые команды отказываются от `Rust` и я вполне могу их понять. Я сам 
решил отказаться от него, благо я успел написать на нём не так уж и много 
кода: простенькое приложение командной строки для подсчета строк в файлах
удобным мне образом (уже переписал на `Go`), приложение командной строки
для поиска дубликатов изображений (на момент написания этой заметки 
пока только частично переписал на `Go` - библиотечку для нахождение 
[phash](https://github.com/Stasenko-Konstantin/phash), осталось
сам cli-интерфейс), а также парочку небольших вещей во время обучения в 
колледже которые нет никакого смысла переписывать.

Что я выбрал вместо `Rust`? Во-первых, конечно же `Go`; у меня даже в моём
github-профиле написано: "Some gopher", так что ничего удивительного. 
Во-вторых, я не буду говорить) Как я уже выше отмечал - я люблю тыкать разные
языки и обычно надолго с ними не задерживаюсь, так что большого смысла писать
о них я не вижу. 

Давайте теперь сравним `Rust` и `Go`! Если вспомнить два моих пункта оценки
языков, то:
1) Оба языка прекрасны как инструменты и рассматривать всевозможные мелкие 
детали здесь не вижу смысла, так как с ними двумя мне работать очень комфортно,
а это здесь главное.
2) Как язык `Go` разумеется проигрывает `Rust` по кол-ву фич, выразительности
и некоторым отдельным удобствам, но borrow checker перекрывает все эти
достоинства `Rust`, отчего `Go` получает огромную фору своими простотой и 
удобством.

Если говорить о простоте `Go` (но не легкости, 
[это разные понятия, об этом еще Рич Хикки говорил](https://www.youtube.com/watch?v=SxdOUGdseq4&pp=0gcJCdgAo7VqN5tD)),
то писать на нём это как писать на `Python`, но приятней так как у тебя
есть какая никакая, но опора в виде статической типизации, что позволяет
дольше оставаться в том самом состоянии дзен (не путать с [Zen of Python](https://peps.python.org/pep-0020/)), 
не впадая при этом слишком часто и надолго в утомляющий дебаг из-за всяких
мелочей, той же динамической типизации. Мысли буквально сами ложатся
на код и не расплываются из-за статической типизации; больше ни с каким
другим языком я такого пока не испытывал. `Rust` же ставит палки в колёса
практически на каждом шагу и чтобы сделать тривиальные вещи тебе приходится
делать невероятные кульбиты, прямо как в том самом меме:

![](https://cdn.jsdelivr.net/gh/Stasenko-Konstantin/my-blog@main/static/images/03-mental-gymnastics.jpg)

Если говорить о моём опыте RIFR (Rewrite It From Rust или типа того),
то библиотеке [urfave/cli](https://github.com/urfave/cli) на `Go` которую я
использовал при переписывании [count](https://github.com/Stasenko-Konstantin/count) 
конечно не сравниться с библиотекой [clap](https://github.com/clap-rs/clap) 
на `Rust` по выразительности, изящности и удобству, но мне не привыкать 
к избыточности `Go` и для моих маленьких вещичек это всё же намного удобнее 
и проще чем бесконечно бодаться о borrow checker.

В конце хотел бы затронуть одну немаловажную тему: пропаганду. Навязчивость
пропаганды `Rust` уже давно стала притчей во языцах, мемом и об этом уже кто 
только не пошутил и не поговорил, но я хочу затронуть один особый аспект её.
Как известно, если в одном месте прибыло, значит в другом убыло; я хочу сказать
что комьюнити `Rust` растёт же не из воздуха, этот язык каждый год называется
StackOverflow как самый любимый и многие программисты хотят перейти на него.
А если переходят с одного места на другое, то первое несколько запустевает, 
так? Это происходит в том числе из-за агрессивной пропаганды `Rust` которая
в своё время отвернула меня от того же `Go`; да, это была история любви не 
с первого, а со второго раза, ведь в первый раз я был убеждён в том что 
отсутствие дженериков это очень и очень плохо. Лишь потом, в силу своего
природного любопытства, я попробовал `Go` и он мне понравился настолько, 
что я уже несколько лет на нём пишу и агитирую друзей к переходу на него;
даже несколько книг им подарил)

В общем, подводя итоги, какова мораль всей этой истории? Я думаю её можно
сформулировать следующим образом:

*Каждой проблеме нужно трезво выбирать свой соответствующий инструмент и не 
отвлекаться на пропаганду, а думать своей головой.* <br>

Потому что если всё же отвлекаться на эту пропаганду, то можно потратить
кучу времени и сил непонятно на что, а в нашем мире, к сожалению, 
время это деньги.