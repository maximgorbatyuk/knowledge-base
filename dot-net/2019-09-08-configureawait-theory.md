# ConfigureAwait, кто виноват и что делать?

Источник: [habr](https://habr.com/ru/company/clrium/blog/463587/)

---

В своей практике я часто встречаю в *различном* окружении код вроде того, что приведен ниже:

```csharp

    [1] var x = FooWithResultAsync(/*...*/).Result;
    
    //или
    [2] FooAsync(/*...*/).Wait();
    
    //или
    [3] FooAsync(/*...*/).GetAwaiter().GetResult();
    
    //или
    [4] FooAsync(/*...*/)
        .ConfigureAwait(false)
        .GetAwaiter()
        .GetResult();
    
    //или
    [5] await FooAsync(/*...*/).ConfigureAwait(false)
    
    //или просто
    [6] await FooAsync(/*...*/)
    
```

Из общения с авторами таких строк, стало ясно, что все они делятся на три группы:

- Первая группа, это те, кому ничего не известно о возможных проблемах с вызовом `Result/Wait/GetResult`. Примеры (1-3) и, иногда, (6), типичны для программистов из этой группы;
- Ко второй группе относятся программисты, которым известно о возможных проблемах, но они не знают причин их возникновения. Разработчики из этой группы, с одной стороны, стараются избегать строк вроде (1-3 и 6), но, с другой, злоупотребляют кодом вроде (4-5);
- Третья группа, по моему опыту самая малочисленная, это те программисты, которые знают о том, как код (1-6) работает, и, поэтому, могут сделать осознанный выбор.

Возможен ли риск, и на сколько он велик, при использовании кода, как в приведенных выше примерах, зависит, как я отмечал ранее, от *окружения*.

## Риски и их причины

Примеры (1-6) делиться на две группы. Первая группа — код с блокировкой вызывающего потока. К этой группе относятся (1-4). Блокировка потока, чаще всего, плохая идея. Почему? Для простоты будем считать, что все потоки выделяются из некоторого пула потоков. Если в программе присутствует блокировка, то это может привести к выборке всех потоков из пула. В лучшем случае, это замедлит работу программы и приведет к неэффективному использованию ресурсов. В худшем же случае, это может привести к взаимоблокировке(deadlock), когда для завершения некоторой задачи, нужен будет дополнительный поток, но пул его не сможет выделить. Таким образом, когда разработчик пишет код вроде (1-4), он должен задуматься, на сколько вероятна описанная выше ситуация.

Но все становится гораздо хуже, когда мы работаем в окружении, в котором существует контекст синхронизации, отличный от стандартного. При наличии *особого* контекста синхронизации блокировка вызывающего потока повышает вероятность возникновения взаимоблокировки многократно. Так, код из примеров (1-3), если он выполняется в UI-потоке WinForms, практически гарантированно создает deadlock. Я пишу "практически", т.к. есть вариант, когда это не так, но об этом чуть позже. Добавление `ConfigureAwait(false)`, как в (4), не даст 100% гарантии защиты от deadlock'a. Ниже приведен пример, подтверждающий это:

```csharp
    [7]
    //Некоторый метод библиотечного / стороннего класса.
    async Task FooAsync()
    {
        // Delay взять для простоты. Может быть любой асинхронный вызов.
        await Task.Delay(5000);
    
        //Остальную часть кода метода объединим в метод
        RestPartOfMethodCode();
    }
    
    //Код в "конечной" точке использования, в данном случае, это WinForms приложение.
    private void button1_Click(object sender, EventArgs e)
    {
        FooAsync()
            .ConfigureAwait(false)
            .GetAwaiter()
            .GetResult();
    
        button1.Text = "new text";
    }
```

В статье ["Parallel Computing — It's All About the SynchronizationContext"](https://msdn.microsoft.com/en-us/magazine/gg598924.aspx) дается информация о различных контекстах синхронизации.

Для того, чтобы понять причину возникновения взаимоблокировки, нужно проанализировать код конечного автомата, в который преобразуется вызов async метода, и, далее, код классов MS. В статье ["Async Await and the Generated StateMachine"](https://www.codeproject.com/Articles/535635/Async-Await-and-the-Generated-StateMachine) приводится пример такого конечного автомата. Не буду приводить полный исходный код, генерируемого для примера (7), автомата, покажу лишь важные для дальнейшего разбора строки:

```csharp
    //Внутри метода MoveNext.
    //...
    // переменная taskAwaiter определена выше по коду.
    
    taskAwaiter = Task.Delay(5000).GetAwaiter();
    if(tasAwaiter.IsCompleted != true)
    {
        _awaiter = taskAwaiter;
        _nextState = ...;
    
        _builder.AwaitUnsafeOnCompleted<TaskAwaiter, ThisStateMachine>(ref taskAwaiter, ref this);
        return;
    }
```

Ветка `if` выполняется, если асинхронный вызов (`Delay`) еще не был завершен и, следовательно, текущий поток можно освободить. Обратите внимание на то, что в `AwaitUnsafeOnCompleted` передается taskAwaiter полученный от **внутреннего** (относительно `FooAsync`) асинхронного вызова (`Delay`).

Если погрузиться в дебри исходников MS, которые скрываются за вызовом `AwaitUnsafeOnCompleted`, то, в конечном итоге, мы придем к классу [SynchronizationContextAwaitTaskContinuation](https://referencesource.microsoft.com/mscorlib/R/d8b8d04cc476b392.html), и его базовому классу [AwaitTaskContinuation](https://referencesource.microsoft.com/mscorlib/system/threading/Tasks/TaskContinuation.cs.html#3f97ac52ec881e24), где и находятся ответ на поставленный вопрос.

Код этих, и связанных с ними, классов довольно запутан, поэтому, для облегчения восприятия, я позволю себе написать сильно упрощенный "аналог" того, во что превращается пример (7), но без конечного автомата, и в терминах TPL:

```csharp
    [8]
    Task FooAsync()
    {
        // Переменная methodCompleted вводится только для того, чтобы подчеркнуть, 
        // что метод завершается тогда, когда будет выполнен некоторый "маркирующий код".
        // В конечном автомате функцию, аналогичную строчке methodCompleted.WaitOne() данного кода,
        // выполняет метод SetResult класса AsyncTaskMethodBuilder,
        // объект которого храниться в поле конечного автомата.
        var methodCompleted = new AutoResetEvent(false);
    
        SynchronizationContext current = SynchronizationContext.Current;
        return Task.Delay(5000).ContinueWith(
            t=>
                {
                    if(current == null)
                    {
                        RestPartOfMethodCode(methodCompleted);
                    }
                    else
                    {
                        current.Post(state=>RestPartOfMethodCode(methodCompleted), null);
                        methodCompleted.WaitOne();
                    }
                },
                TaskScheduler.Current);
    }
    
    //
    // void RestPartOfMethodCode(AutoResetEvent methodCompleted)
    // {
    //     Тут оставшаяся часть кода метода FooAsync.
    //   methodCompleted.Set();
    // }
```

В примере (8) важно обратить внимание на то, что при наличии контекста синхронизации, весь код асинхронного метода, который идет после завершения внутреннего асинхронного вызова, **выполняется через этот контекст** (вызов `current.Post(...)`). Этот факт и **является причиной** возникновения взаимоблокировок. Например, если речь идет о WinForms-приложении, то контекст синхронизации в нем связан с UI-потоком. Если UI-поток заблокирован, в примере (7) это происходит через вызов `.GetResult()`, то оставшаяся часть кода асинхронного метода выполниться не может, а значит, асинхронный метод не может завершиться, и не может освободить UI-поток, что и есть deadlock.

В примере (7) вызов `FooAsync` был сконфигурирован через `ConfigureAwait(false)`, но это не помогло. Дело в том, что конфигурировать надо именно тот объект ожидания, который будет передан в `AwaitUnsafeOnCompleted`, в нашем примере это объект ожидания от вызова `Delay`. Другими словами, в данном случае, вызов `ConfigureAwait(false)` в клиентском коде не имеет смысла. Решить проблему можно если разработчик метода `FooAsync` изменит его следующим образом:

```csharp
    [9]
    async Task FooAsync()
    {
        await Task.Delay(5000).ConfigureAwait(false);
        //Остальную часть кода метода объединим в метод
        RestPartOfMethodCode();
    }
    
    private void button1_Click(object sender, EventArgs e)
    {
        FooAsync().GetAwaiter().GetResult();
    
        button1.Text = "new text";
    }
```

Выше мы рассмотрели риски возникающие с кодом первой группы — код с блокировкой (примеры 1-4). Теперь о второй группе (примеры 5 и 6) — код без блокировок. В этом случае возникает вопрос, когда вызов `ConfigureAwait(false)` оправдан? При разборе примера (7), мы уже выяснили, что конфигурировать надо тот объект ожидания, на основе которого будет построено продолжение выполнения. Т.е. конфигурация требуется (если вы приняли такое решение) только для **внутренних** асинхронных вызовов.

## Кто виноват?

Как всегда, правильным ответом будет "все". Начнем с программистов из MS. С одной стороны, разработчики Microsoft приняли решение, что, при наличии контекста синхронизации, работа должна вестись через него. И это логично, иначе зачем он еще нужен. И, как я полагаю, они ожидали, что разработчики "клиентского" кода **не** будут блокировать основной поток, тем более в том случае, когда контекст синхронизации на него завязан. С другой стороны, они дали очень простой инструмент чтобы "выстрелить себе в ногу" — слишком просто и удобно получать результат через блокирующие `.Result/.GetResult`, или блокировать поток, в ожидании завершения вызова, через `.Wait`. Т.е. разработчики MS сделали так, что "неправильное" (или опасное) использование их библиотек не вызывает каких-либо затруднений.

Но есть вина и на разработчиках "клиентского" кода. Она состоит в том, что, зачастую, разработчики не пытаются разобраться в своем инструменте и пренебрегают предупреждениями. А это прямой путь к ошибкам.

## Что делать?

Ниже я привожу мои рекомендации.

### Для разработчиков клиентского кода

1. Всеми силами избегайте блокировок. Другими словами, не смешивайте синхронный и асинхронный код без особой необходимости.
2. Если приходится делать блокировку, то определите, в каком окружении выполняется код:
    - Есть ли контекст синхронизации? Если да, то какой? Какие особенности в работе он создает?
    - Если контекста синхронизации "нет", то: Какова будет нагрузка? Какова вероятность что ваша блокировка приведет к "утечки" потоков из пула? Хватит ли того числа потоков, что создается на старте, по умолчанию, или надо выделить больше?
3. Если код асинхронный, то нужен ли вам конфигурировать асинхронный вызов через `ConfigureAwait`?

Принимайте решение на основе всей полученной информации. Возможно, вам надо пересмотреть подход к реализации. Возможно, вам поможет `ConfigureAwait`, а может он вам не нужен.

### Для разработчиков библиотек

1. Если вы полагаете, что ваш код может быть вызван из "синхронного", то обязательно реализуйте синхронный API. Он должен быть по-настоящему синхронным, т.е. вы должны пользоваться синхронным API сторонних библиотек.
2. `ConfigureAwait(true / false)`.

Тут, с моей точки зрения, необходим более тонкий подход чем обычно рекомендуют. Во многих статьях говорится, что в библиотечном коде, все асинхронные вызовы надо конфигурировать через `ConfigureAwait(false)`. Я не могу с этим согласиться. Возможно, с точки зрения авторов, коллеги из Microsoft приняли неверное решение при выборе поведения "по умолчанию" в отношении работы с контекстом синхронизации. Но они (MS), все же, оставили возможность разработчикам "клиентского" кода изменить это поведение. Стратегия, когда библиотечный код полностью покрывается `ConfigureAwait(false)`, изменяет поведение по умолчанию, и, что более важно, такой подход лишает разработчиков "клиентского" кода выбора.

Мой вариант заключается в том, чтобы, при реализации асинхронного API, в каждый метод API добавлять два дополнительных входных параметра: `CancellationToken token` и `bool continueOnCapturedContext`. И реализовывать код в следующем виде:

```csharp
    public async Task<string> FooAsync(
        /*другие аргументы функции*/,
        CancellationToken token, 
        bool continueOnCapturedContext)
    {
        // ...
        await Task.Delay(30, token).ConfigureAwait(continueOnCapturedContext);
        // ...
        return result;
    }
```

Первый параметр, `token` — служит, как известно, для возможности скоординированной отмены (разработчики библиотек этой возможностью, иногда, пренебрегают). Второй, `continueOnCapturedContext` — позволяет настроить взаимодействие с контекстом синхронизации внутренних асинхронных вызовов.

При этом, если асинхронный метод API будет сам частью другого асинхронного метода, то "клиентский" код сможет определить, как он должен взаимодействовать с контекстом синхронизации:

    // Пример вызова в асинхронном коде:
    async Task ClientFoo()
    {
        // "Внутренний" код ClientFoo учитывает контекст синхронизации, в то время как 
        // внутренний код FooAsync игнорирует контекст синхронизации.
        await FooAsync(
            /*другие аргументы функции*/,
            ancellationToken.None,
            false);
    
        // Код всех уровней игнорирует контекст.
        await FooAsync(
            /*другие аргументы функции*/,
            ancellationToken.None, 
            false).ConfigureAwait(false);
        //...
    }
    
    //В синхронном, с блокировкой.
    private void button1_Click(object sender, EventArgs e)
    {
        FooAsync(
            /*другие аргументы функции*/,
            _source.Token,
            false).GetAwaiter().GetResult();
    
        button1.Text = "new text";
    }

## В качестве заключения

Главный вывод из всего вышеизложенного заключается в следующих трех мыслях:

1. Блокировки, чаще всего, корень всех зол. Именно наличие блокировок может привести, в лучшем случае, к деградации производительности и неэффективному использованию ресурсов, в худшем — к deadlock-у. Прежде чем использовать блокировки подумайте, нужно ли это? Возможно, есть другой, приемлемый в вашем случае, способ синхронизации;
2. Изучайте инструмент, с которым работаете;
3. Если проектируете библиотеки, то старайтесь сделать так, чтобы их правильное использование было легким, почти интуитивным, а неправильное было сопряжено со сложностями.

Я постарался максимально просто объяснить риски связанные с async/await, и причины их возникновения. А также, представил мое видение решения этих проблем. Надеюсь, что это мне удалось, и материал будет полезен читателю. Для того чтобы лучше понять, как все работает на самом деле, надо, конечно, обратиться к исходникам. Это можно сделать через репозитории MS на [GitHub](https://github.com/microsoft/referencesource/tree/master/mscorlib/system/threading/Tasks) или, что даже удобнее, через сайт самого [MS](https://referencesource.microsoft.com/#mscorlib/system/threading/Tasks/).

*P.S.* Буду благодарен за конструктивную критику.

---

Ссылки по теме:

1. Как работает класс Task - [mdsn](https://docs.microsoft.com/ru-ru/dotnet/api/system.threading.tasks.task?view=netframework-4.8)
2. StyleCop - [CA2007. Не следует напрямую ожидать Task](https://docs.microsoft.com/ru-ru/visualstudio/code-quality/ca2007-do-not-directly-await-task?view=vs-2019)
3. [Parallel Computing - It's All About the SynchronizationContext](https://msdn.microsoft.com/en-us/magazine/gg598924.aspx)