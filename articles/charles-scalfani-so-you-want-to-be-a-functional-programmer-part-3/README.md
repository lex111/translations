# Итак, вы хотите научиться функциональному программированию (Часть 3)

*Перевод статьи [Charles Scalfani](https://medium.com/@cscalfani): [So You Want to be a Functional Programmer (Part 3)](https://medium.com/@cscalfani/so-you-want-to-be-a-functional-programmer-part-3-1b0fd14eb1a7) с [наилучшими пожеланиями от автора](https://twitter.com/cscalfani/status/933052963781722112).*

![Эволюция парадигм программирования](https://cdn-images-1.medium.com/max/800/1*AM83LP9sGGjIul3c5hIsWg.png)

Первый шаг к пониманию идей функционального программирования – самый важный и иногда самый сложный шаг. Но с правильным подходом никаких трудностей быть не должно.

Предыдущие части: [Часть 1](https://medium.com/devschacht/charles-scalfani-so-you-want-to-be-a-functional-programmer-part-1-6ef98e90d58d), [Часть 2](https://medium.com/devschacht/charles-scalfani-so-you-want-to-be-a-functional-programmer-part-2-ae095d9807b3).

## Композиция функций

![Программисты глазами здорового человека](https://cdn-images-1.medium.com/max/800/1*yGnDGRW4pTgmcDUi4oC8Uw.png)

Как все нормальные программисты, мы — ленивые. Мы не хотим постоянно собирать, тестировать и деплоить один и тот же код, который переписываем снова, и снова, и снова.

Мы всегда пытаемся найти пути, чтобы, решив какую-нибудь задачу однажды, использовать это решение в других случаях.

Повторное использование кода — хорошая затея, но в реальности этого тяжело добиться. Попробуйте сделать код слишком специализированным и у вас не получится использовать его снова. Попробуйте сделать его слишком общим и вам будет сложно использовать его даже в вашей первоочередной задаче.

То, что нам нужно — баланс между этими двумя положениями, способ создавать элементы поменьше с возможностью их многократного использования, которые мы будем применять как строительные блоки для конструирования более сложного функционала.

В функциональном программировании функции - наши строительные блоки. Мы пишем их для решения определённых задач, а потом складываем вместе, как блоки в Lego™.

Результат такого сложения называется ***композицией функций***.

Так как же это работает? Давайте начнём с двух JavaScript-функций:

```js
var add10 = function(value) {
    return value + 10;
};
var mult5 = function(value) {
    return value * 5;
};
```

Код получился слишком пространный, так что давайте перепишем его, используя [стрелочные функции](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions/Arrow_functions):

```js
var add10 = value => value + 10;
var mult5 = value => value * 5;
```

Так-то лучше. Теперь давайте представим, что нам нужна функция, принимающая значение и добавляющая к нему 10, после чего умножающая результат на 5. Мы могли бы написать:

```js
var mult5AfterAdd10 = value => 5 * (value + 10)
```

Даже несмотря на такой простой пример, нам бы не хотелось писать целую функцию с нуля. Во-первых, мы могли бы допустить ошибку и, например, забыть поставить круглые скобки.

Во-вторых, у нас есть одна функция, прибавляющая к значению 10, и другая, умножающая его на 5. Получается, мы пишем код, который уже написали.

Так что вместо этого давайте используем `add10` и `mult5` и соберём новую функцию:

```js
var mult5AfterAdd10 = value => mult5(add10(value));
```

Мы просто использовали существующие функции, чтобы получить `mult5AfterAdd10`, но есть и способ получше.

В математике ***f ∘ g*** – композиция функций (*прим. пер., или суперпозиция функций*) и читается она как «***применение функции f к результату функции g***» или более просто «***выполнение f после g***». Получается, что ***(f ∘ g)(x)*** — эквивалент вызова функции ***f*** после функции ***g*** со значением ***x*** или ещё проще: `f(g(x))`.

В нашем примере у нас `mult5` ∘ `add10` или «`mult5` ***после*** `add10`», отсюда и название нашей функции `mult5AfterAdd10`.

И это объясняет то, что мы сделали. Мы вызвали `mult5` после вызова `add10` с `value` или просто: `mult5(add10(value))`.

Поскольку JavaScript нативно не реализует возможность композиции функций, давайте взглянем на Elm:

```elm
add10 value =
    value + 10

mult5 value =
    value * 5

mult5AfterAdd10 value =
    (mult5 << add10) value
```

В Elm функции компонуются с помощью инфиксального оператора `<<`. Это даёт нам визуально понять как параметры из одной функции «перетекают» в другую. Сначала `value` попадает в `add10`, а затем её результат попадает в `mult5`.

Обратите внимание на скобки в `mult5AfterAdd10`, то есть именно на выражение `(mult5 << add10)`. Они там затем, чтобы быть уверенными, что `value` будет передана внутрь выражения после того, как функции в нём скомпонуются.

Вы можете скомпоновать столько функций, сколько захотите, например:

```elm
f x =
   (g << h << s << r << t) x
```

В этом случае `x` передаётся в `f`, чей результат передаётся в `r`, чей результат, в свою очередь, — в `s` и так далее. Если вы захотите сделать что-то подобное в JavaScript, это будет выглядеть примерно так: `g(h(s(r(t(x)))))` — просто какой-то ад из круглых скобок. 

## Бесточечная нотация

![Свободная точка или точка свободы](https://cdn-images-1.medium.com/max/800/1*g2pWcQJ0jOUf1WKbTDIktQ.png)

***Бесточечная нотация*** — стиль описания функций без предварительного указания входных параметров. По началу такой стиль будет казаться необычным, но по мере продолжения, когда вы достаточно разберётесь, вы оцените лаконичность такого подхода. 

Вы заметите, что в `mult5AfterAdd10` значение `value` определяется дважды. Один раз в списке параметров и один раз в момент использования.

```elm
-- Эта функция ожидает 1 входной параметр

mult5AfterAdd10 value =
    (mult5 << add10) value
```

Но этот параметр несущественен, так как `add10`, самая правая функция в композиции, ожидает тот же параметр. Следующая бесточечная версия — эквивалент предыдущей:

```elm
-- Эта функция тоже ожидает 1 входной параметр

mult5AfterAdd10 =
    (mult5 << add10)
```

В использовании такого бесточечного подхода существует множество преимуществ.

Во-первых, нам не нужно определять лишние параметры. И поскольку мы их не определяем, нам также не нужно придумывать им имена.

Во-вторых, код становится легче читать и анализировать, так как он более лаконичный. Это простой пример, но представьте функцию, принимающую больше параметров.

## «Тени в раю»

![Мост, который не совсем мост](https://cdn-images-1.medium.com/max/800/1*RE3Qxh6Bg9umzQ5dOrF6pw.png)

До сих пор мы разбирались, как работает композиция функций, и как следует определять функции при бесточечной нотации для лаконичности, чистоты и гибкости кода.

Теперь давайте попробуем использовать эти идеи при слегка ином сценарии и посмотрим, насколько они ему соответствуют. Представьте, что мы заменяем `add10` на `add`:

```elm
add x y =
    x + y

mult5 value =
    value * 5
```

Как нам создать `mult5AfterAdd10`, используя эти две функции?

Ладно, если вы действительно потратите время, раздумывая над этим вопросом, то вернётесь примерно с таким решением:

```elm
-- Это неверно !!!!

mult5AfterAdd10 =
    (mult5 << add) 10 
```

Но такой код не будет работать. Потому что `add` принимает два параметра.

Если это не так очевидно в Elm, попробуйте написать то же самое в JavaScript:

```js
var mult5AfterAdd10 = mult5(add(10)); // не работает
```

Этот код неверен, но почему?

Потому что в нём функция `add` принимает лишь один из своих двух параметров, после чего передаёт ошибочные результаты в `mult5`. Это будет приводить к неверным решениям.

По факту, в Elm компилятор не позволит вам даже написать такой нецелесообразный код (что является одним из больших плюсов языка Elm).

Давайте попробуем ещё раз:

```js
var mult5AfterAdd10 = y => mult5(add(10, y)); // не бесточечный стиль
```

Да, это не бесточечный стиль, но я тем не менее могу жить с таким решением. В то же время, теперь я уже не просто комбинирую функции. Я пишу новую функцию. Также, если задача станет гораздо сложнее, к примеру, если я захочу скомпоновать `mult5AfterAdd10` с какой-то другой функцией, я столкнусь с серьёзной проблемой.

Пока мы не можем «обвенчать» эти две функции, будет казаться, что идея композиции функций не обладает такой уж большой практической ценностью. И это очень плохо, потому что, на самом деле, это не так.

Что ж, что было бы действительно хорошо, так это если бы у нас была идея как *предварительно* передать нашей функции `add` один из её входных параметров и уже *позже* — второй параметр, когда будет вызвана `mult5AfterAdd10`.

Выходом из этого затруднительного положения является концепция ***каррирования***.

## Мой мозг!!!!

![Условная инструкция что делать, если вы вправду так сказали](https://cdn-images-1.medium.com/max/800/1*IK5485-iZaHeZRfP8aWmYg.png)

Пока что достаточно.

В последующих частях этой статьи я расскажу про стандартные функции в функциональном программировании (такие как `map`, `filter`, `fold` и так далее), прозрачность ссылок и ещё много о чём.

---

*Слушайте наш подкаст в [iTunes](https://itunes.apple.com/ru/podcast/%D0%B4%D0%B5%D0%B2%D1%88%D0%B0%D1%85%D1%82%D0%B0/id1226773343) и [SoundCloud](https://soundcloud.com/devschacht), читайте нас на [Medium](https://medium.com/devschacht), контрибьютьте на [GitHub](https://github.com/devSchacht), общайтесь в [группе Telegram](https://t.me/devSchacht), следите в [Twitter](https://twitter.com/DevSchacht) и [канале Telegram](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht).*

[Статья на GitHub](https://github.com/communar/translations/tree/master/articles/charles-scalfani-so-you-want-to-be-a-functional-programmer-part-3)
