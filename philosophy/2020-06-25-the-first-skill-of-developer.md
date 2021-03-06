# Главный навык программиста. Сергей Тепляков

Источник: [Блог Сергея Теплякова](http://sergeyteplyakov.blogspot.com/2019/12/blog-post.html)

---

Я заметил, что опытных специалистов из разных областей объединяет как минимум одна вещь: осторожность в суждениях.

Если вы спросите программиста, повара или фитнес тренера о чем-то довольно сложном, то ответ его будет довольно обтекаемым, либо же он задаст несколько наводящих вопросов, чтобы узнать контекст получше.

Нужно ли писать тесты? Важно ли качество кода? Какая лучшая степень прожарки мяса? С чем лучше всего сочетается белое вино? Нужно ли делать жим лежа со становой или турника будет достаточно?

Когда я рассматриваю источники информации по некоторой теме я ищу в авторе глубину и осторожность. Желание дать фундаментальные принципы, а не конкретные ответы на сложные вопросы. В этом плане старина Боб Мартин меня давно разочаровал. Когда-то он для меня был автором лучших книг и статей, но его категоричность, негибкость в суждениях и неумение смотреть на проблемы шире перевели его в категорию "анти"-авторов. В категорию людей, чье мнение скорее опасно для индустрии, чем полезно.

Моя нелюбовь к старине Бобу началась с его книги "Принципы, паттерны и методики гибкой разработки на языке C#" (вот [большая критическая статья](http://sergeyteplyakov.blogspot.com/2013/12/about-agile-principles-patterns-and.html) на эту тему), но дядюшка Боб периодически выдает и вот, после очередного его высказывания, мне захотелось подумать над поднятой им темой более подробно.

Итак, вот его твит:

> When you read code, the race, religion, politics, gender, and orientation of the author are irrelevant and invisible. The only thing you can tell about the author is their ability to write well organized code. Nothing else matters.

Это единственный твит, здесь нет контекста, нет продолжения от автора, так что многие вещи я буду додумывать. А учитывая мою нелюбовь к автору, я могу быть весьма предвзятым. Но поскольку сам автор не соизволил дополнить свою мысль, то я не вижу в этом ничего плохого.

Структура твита. Автор вначале говорит о расе, религии, ориентации и поле, как о несущественных вещах в контексте анализа кода. Да. С этим никто не будет спорить. Это так. Но потом он заявляет, что единственная вещь, которая говорит об авторе кода - это его умение писать хороший код и что ничего другое не важно.

Здесь используется довольно грязный трюк с подменой понятий и отсутствием причинно-следственной связи между двумя частями твита. Тот факт, что мы не должны судить об авторе кода по его "внешним атрибутам" не делает "хорошо структурированный код" важным. И с чего старина Боб решил, что должно какое-то одно главное качество, характеризующее специалиста?

Поскольку тема эта интересна, то давайте немного пофилософствуем по этому поводу.

"Что самое главное в жизни?", может спросить Боб. "Семья, конечно", ответит один. "А как же здоровье? Если нет здоровья, то радости от семьи будет не много.", возразит второй. "А как же призвание, а деньги? На одной семье, пусть и здоровой, далеко не уедешь! Нужно ж еще и кушать вкусно, да и делом интересным заниматься!", скажет третий. "Ну а помощь другим? Это ж тоже важно! А друзья? А карьера? А красивое тело?".

Мне кажется, что жизнь достаточно сложная штука, чтобы в ней была одна единственная главная вещь, всецело определяющая ее суть . Жизнь - это игра не совсем с фиксированной суммой. В разумных пределах можно делать больше и успевать больше. Семья может мотивировать заниматься другими делами, например, своим здоровьем и даже карьерой. А приходя с работы, которая тебя радуем, может оставаться больше сил, чтобы поиграть с ребенком и насладиться временем, проведенным наедине со своим партнером. Жизнь многогранна и не может всецело определяться чем-то одним.

Работа программиста несколько проще, чем жизнь в целом, но и она достаточно сложна и многогранна. Чего стоит хорошо структурированный код, который решает не ту проблему? А если автор хорошо структурированного кода токсичен и из-за него вся команда разваливается? А если хорошо структурированный код использует странные или даже неверные паттерны, многословен или дублирует код, написанный в другом месте?

Уметь писать хороший код очень полезно, но я не могу назвать этот навык главным и самым ценным навыком программиста. Если на то уж пошло, то умение читать реальный (читай плохой) код является более полезным навыком, ведь большую часть времени мы проводим в анализе существующего кода и поиска правильного места для вставки или изменения функционала.

Если же задаться вопросом о главном навыке (навыках?) специалиста из любой области, то я бы сказал, что это умение учиться и критически мыслить (да, и не быть м#$@ком). Эти два ключевых мета-навыка позволят развить любые другие.

Я заметил, что лучшие инженеры учатся постоянно. И не столько читая книги, сколько потребляя информацию в процессе решения реальных проблем. Они мыслят более масштабно, и не боятся нарушить статус-кво, задавая неудобные вопросы. А ту ли проблему мы решаем? А что же на самом деле нужно нашим пользователям? А не стоит ли заменить/переписать ключевой компонент? А не залипли ли мы с этими практиками, которые работали когда-то, а сейчас дают больше вреда, чем пользы?

Хороший специалист может увидеть проблему (в себе, в коде, архитектуре или подходах), подумать об альтернативах и изучить наиболее подходящее решение.

Мне может лишь в страшном сне привидеться, что вся команда состоит из однотипных людей. Людей с одним, даже самым лучшим единственным навыком. Хорошая команда - это разнообразие в лучшем понимании этого слова. Разнообразие характеров, навыков, подходов. Их должны объединять схожие принципы и ценности, но попытка грести всех под одну гребенку вызывает у меня лишь недоумение.

Мне тоже хочется найти простое решение таких сложных проблем, как разработка ПО. "Развивай навык Х и ты будешь отличным специалистом!". Но мир и разработка ПО достаточно сложны и многогранны, чтобы был один главный навык, который бы в одиночку смог бы решить все наши проблемы.
