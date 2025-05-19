# Правила разработки

Это исключительно технические правила, описывающие как вы должны проводить вашу работу в репозитории.

!!! warning
    Вики находится на этапе заполнения, поэтому большая часть информации тут отсутствует. Её можно найти в [документации от официальных разработчиков SS14](https://docs.spacestation14.com), но описанные здесь пункты всё ещё должны следоваться.

## Правила PR

### Запрет master-ветки

Ветка из которой вы делаете Pull Request не должна являться master-веткой.

Если всё же получилось так, что вы уже опубликовали изменения под master-веткой, то вот как исправить ситуацию:

```bash
$ git checkout -B {feature-name}
$ git switch master
$ git reset --hard HEAD~{количество комитов на которое вам нужно откатиться назад}
```

### Регионы изменений

При добавлении совершенно новых систем, новых текстур или прототипов, вы должны их располагать в папках `Content.{Client/Server/Shared}/_Exodus`, `Resources/Prototypes/_Exodus`, `Resources/Locale/ru-RU/exodus`, `Resources/Audio/Exodus`, `Resources/Textures/Exodus`.

При необходимости изменить код в файлах официальных разработчиков, вам требуется обозначить изменённые участки добавлением соответствующего комментария формата `Exodus-{feature}`, одним из способов:

1. Однострочное изменение<br>
Если изменение затрагивает лишь одну строчку, то он добавляется на изменённую строку как: `// Exodus-Feature`.
2. Многострочный блок<br>
Если ваше изменение затрагивает блок кода больше чем на одну строку, то вам следует изменённые фрагменты обрамлять в пару комментариев: `// Exodus-Feature-Begin` в начале, `// Exodus-Feature-End` в конце.<br>
Например:
```cs
// Exodus-Feature-Begin
var a = 1 + 1;
Console.WriteLine($"1+1 will be equal to: {a}");
// Exodus-Feature-End
```

Также, при изменениях в сложной логике или в принципе сложных изменениях, крайне желательно наличие пояснений к вашим изменениям, как правило они указываются через символ `|` на строках обозначающих изменённые регионы.<br>
Например:
```cs
// Exodus-Feature-Begin | Make calculation of a to be done here, cuz way more effecient
// Moving this block to other parts will lead to {any other explanations}
var a = 1 + 1;
Console.WriteLine($"1+1 will be equal to: {a}");
// Exodus-Feature-End
```

### Лицензия

При публикации своего PR, вы должны согласиться с условиями лицензии CLA, я уверен, что её условия вам уже объяснили, однако, если нет, то её суть кратко описана [здесь](../#cla).

Файлы `.cs` и `.xaml` добавляемые в папки обозначенные в пункте "Регионы изменений" должны иметь в самом верху следующий комментарий: `© Space Exodus, An EULA/CLA with a hosting restriction, full text: https://raw.githubusercontent.com/space-exodus/space-station-14/master/CLA.txt`. Это дополнительное подтверждение того, что контент Space Exodus находится под CLA.

Примеры:

* `*.cs`:
```c#
// © Space Exodus, An EULA/CLA with a hosting restriction, full text: https://raw.githubusercontent.com/space-exodus/space-station-14/master/CLA.txt
```

* `*.xaml`:
```xaml
<!-- © Space Exodus, An EULA/CLA with a hosting restriction, full text: https://raw.githubusercontent.com/space-exodus/space-station-14/master/CLA.txt -->
```
