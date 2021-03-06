# О сложности. Блог С. Теплякова

Источник: [http://sergeyteplyakov.blogspot.com/2015/02/blog-post.html](http://sergeyteplyakov.blogspot.com/2015/02/blog-post.html)

## О сложности

Фред Брукс в своем «Мифическом человеко-месяце» еще 30 лет назад описал пару понятий, которые я считаю ключевыми в проектировании. Это понятия случайной (accidental) и естественной (essential) сложности.

Разница между ними в том, что естественная сложность является частью самой задачи и от нее невозможно избавиться. Случайная сложность привносится в процессе решения и обусловлена ошибками проектирования, плохим выбором языка программирования или базы данных, неудачной декомпозицией системы и просто грязным кодом. Все эти иерархии наследования, принципы проектирования, cohesion & coupling, все это направлено на уменьшение случайной сложности. На то, чтобы обуздать естественную сложность задачи и сделать ее максимально очевидной.

Мне кажется, очень важно осознавать разницу между этими понятиями.

Сколько раз вы ловили себя на мысли, что совершенно не можете понять, что же делает некоторый код? Вы видите кучу блоков if-else, switch, foreach, перехват и логирование исключений. Все это хорошо. Но, какую же задачу этот код решает на самом деле? Затем вы начинаете распутывать этот клубок, добавляете комментарии, выделяете новые методы, переносите обработку исключений на более высокий уровень, и оказывается, что решается очень простая задача, которую можно описать парой предложений.

Разработчик первоначального кода без особого труда поместил в свою толковую кибитку исходную задачу, и выразил ее решение в максимально простой форме ([easy but not simple](http://www.infoq.com/presentations/Simple-Made-Easy)). Затем, пришлось добавить вспомогательные вещи, такие как логирование, обработка ошибок и проверка граничных условий. С его точки зрения, задача и решение все еще оставалось «простым и понятным», поскольку часть его мозга все еще помнила о том, что же происходит в его коде.

Но все становится не так просто, когда за код берется кто-то другой или ты сам, через некоторое время, когда смысл задачи уже выветрился из кратковременной памяти. Тогда, весь этот «шум» начинает раздражать, поскольку ты уже не можешь сосредоточиться на самой задаче.

Мне сложно сказать, в чем причина столь резкого роста случайной сложности в проектах. Высокий интеллект разработчиков и подход типа «мне сейчас понятно это решение, значит и другим тоже оно будет понятно, а если нет, то мне плевать»? Нежелание вернуться к «завершенной» задаче и причесать ее? Боязнь срыва сроков и опасение, что адекватное решение потребует значительно больше усилий?

Любой, даже очень хорошо структурированный код, на первый взгляд может показаться сложным. Как бы мы не убирали случайную сложность, решение все еще может быть сложным. Код любой системы все равно будет сложным, это нормально. Здесь может проявляться синдром «это написано не мной» и «чукча не читатель кода, он писатель кода».

## Борьба со случайно сложностью

Но бороться со случайной сложностью можно и нужно. «Капитан» дает несколько советов:

- Посмотреть на код со стороны через некоторое время, когда задача выветрилась из кратковременной памяти (этот же подход отлично подходит для ревью любого своего материала – статей, книг или кода).
- Подумайте о том, сколько неявных допущений вы делаете при внесении исправления. Насколько эти допущения будут очевидны другому человеку.Посмотрите на сделанные изменения со стороны: сколько десятков переключений по коду вы сделали, чтобы изменить эту строку? Насколько это изменение будет очевидно новичку-кикбоксеру, который найдет вас в темном переулке, если ему попадет из-за вашего кода?
- Не бойтесь «срезать углы» в коде, но делайте это осознанно, понимая, сколько проблем «срезанный угол» принесет в будущем. Является ли увеличение искусственной сложности локальным или глобальным?
- Не бойтесь причесывать код или оставлять в нем комментарии. Даже если код не ваш и даже, если на проекте (как обычно) пожар. Если это не одноразовый проект, то такие инвестиции окупятся.
- Стремитесь к декларативному коду. Из кода должны быть очевидны моделируемые бизнес правила. Если это не так, то стоит хорошо задуматься над тем, насколько другому человеку будет очевидно, что же код делает.

В общем, постарайтесь хотя бы немного смотреть на код глазами другого человека. Попробуйте сделать так, чтобы ваши намерения были хоть немного понятными без литра пива и магии Вуду. А если они не понятны, то не стесняйтесь, черканите небольшой комментарий, они [именно для этого и предназначены](http://sergeyteplyakov.blogspot.com/2013/05/blog-post_20.html)!
