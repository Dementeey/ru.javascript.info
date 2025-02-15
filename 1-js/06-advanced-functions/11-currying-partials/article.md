libs:
  - lodash

---

# Каррирование и частичное применение

До сих пор мы говорили только о привязывании `this`. Давайте шагнём дальше.

Мы можем привязать не только `this`, но и аргументы. Это делается редко, но иногда может быть полезно.

Полный синтаксис `bind`:

```js
let bound = func.bind(context, arg1, arg2, ...);
```
Это позволяет привязать контекст, например, `this` и начать передавать аргументы функции.

Например, у нас есть функция умножения `mul(a, b)`:

```js
function mul(a, b) {
  return a * b;
}
```

Давайте воспользуемся `bind`, чтобы создать функцию `double` на её основе:

```js run
function mul(a, b) {
  return a * b;
}

*!*
let double = mul.bind(null, 2);
*/!*

alert( double(3) ); // = mul(2, 3) = 6
alert( double(4) ); // = mul(2, 4) = 8
alert( double(5) ); // = mul(2, 5) = 10
```

Вызов `mul.bind(null, 2)` создаёт новую функцию `double`, которая передаёт вызов `mul`, фиксируя `null` как контекст и `2` -- как первый аргумент. Следующие аргументы передаются "как есть".

Это называется [частичное применение](https://ru.wikipedia.org/wiki/Частичное_применение) -- мы создаём новую функцию, фиксируя некоторые из существующих параметров.

Обратите внимание, что в данном случае мы на самом деле не используем `this`. Но для `bind` это обязательный параметр, так что мы должны передать туда что-нибудь вроде `null`.

В следующем коде функция `triple` умножает значение на три:

```js run
function mul(a, b) {
  return a * b;
}

*!*
let triple = mul.bind(null, 3);
*/!*

alert( triple(3) ); // = mul(3, 3) = 9
alert( triple(4) ); // = mul(3, 4) = 12
alert( triple(5) ); // = mul(3, 5) = 15
```

Для чего мы обычно создаём частично применённую функцию?

Польза от этого в том, что возможно создать независимую функцию с понятным названием (`double`, `triple`). Мы можем использовать её и не передавать каждый раз первый аргумент, т.к. он зафиксирован с помощью `bind`.

В других случаях частичное применение полезно, когда у нас есть очень общая функция и для удобства мы хотим создать её частный вариант.

Например, у нас есть функция `send(from, to, text)`. Потом внутри объекта `user` мы можем захотеть использовать её частный вариант: `sendTo(to, text)`, который отправляет текст от имени текущего пользователя.

## Частичное применение без контекста

Что если мы хотим зафиксировать некоторые аргументы, но не привязывать `this`?

Встроенный `bind` не позволяет этого. Мы не можем просто опустить контекст и перейти  к аргументам.

К счастью, несложно создать частично применённую функцию, которая привязывает только аргументы.

Вот так:

```js run
*!*
function partial(func, ...argsBound) {
  return function(...args) { // (*)
    return func.call(this, ...argsBound, ...args);
  }
}
*/!*

// Usage:
let user = {
  firstName: "John",
  say(time, phrase) {
    alert(`[${time}] ${this.firstName}: ${phrase}!`);
  }
};

// добавляем частично применённый метод, который говорит что-нибудь в настоящее время за счёт фиксирования первого аргумента
user.sayNow = partial(user.say, new Date().getHours() + ':' + new Date().getMinutes());

user.sayNow("Hello");
// Что-то вроде этого:
// [10:00] John: Hello!
```

Результатом вызова `partial(func[, arg1, arg2...])` будет обёртка `(*)`, которая вызывает `func` с:
- Тем же `this`, который она получает (для вызова `user.sayNow` -- это будет `user`)
- Затем передаёт ей `...argsBound` -- аргументы из вызова `partial` (`"10:00"`)
- Затем передаёт ей `...args` -- аргументы, полученные обёрткой (`"Hello"`)

Благодаря оператору расширения `...` это реализовать очень легко, не правда ли?

Также есть готовый вариант [_.partial](https://lodash.com/docs#partial) из библиотеки lodash.

## Каррирование

Иногда люди смешивают упомянутое выше частичное применение функции с другой возможностью, называемой "каррирование". Это ещё одна интересная техника работы с функциями, которую мы должны упомянуть здесь.

[Каррирование](https://ru.wikipedia.org/wiki/Каррирование) -- это трансформация функций таким образом, чтобы они принимали  аргументы не так `f(a, b, c)`, а вот так `f(a)(b)(c)`. В JavaScript мы обычно делаем обёртку, чтобы хранить оригинальную функцию.

Каррирование не вызывает функцию. Оно просто трансформирует её.

Давайте создадим вспомогательную функцию `curry(f)`, которая выполняет каррирование бинарной функции `f`. Другими словами, `curry(f)` для бинарной функции f(a, b)` трансформирует её в `f(a)(b)`.

```js run
*!*
function curry(f) { // curry(f) выполняет каррирование
  return function(a) {
    return function(b) {
      return f(a, b);
    };
  };
}
*/!*

// использование
function sum(a, b) {
  return a + b;
}

let carriedSum = curry(sum);

alert( carriedSum(1)(2) ); // 3
```

Как вы видите, реализация состоит из множества обёрток.

- Результат `curry(func)` -- обёртка `function(a)`.
- Когда она вызывается как `sum(1)`, аргумент сохраняется в лексическом окружении и возвращается новая обёртка `function(b)`.
- После чего `sum(1)(2)`, наконец, вызывает `function(b)`, предоставляя `2` и передаёт вызов к оригинальной мультиарной функции `sum`. 

Более продвинутая реализация каррирования, как [_.curry](https://lodash.com/docs#curry) из библиотеки lodash, делает нечто более сложное. Она возвращает обёртку, которая позволяет выполнить функцию, если переданы все аргументы, в противном же случае -- возвращает частично применённую функцию.

```js
function curry(f) {
  return function(...args) {
    // if args.length == f.length (количество аргументов, принимаемое f).
    //   тогда передаёт вызов f
    // в противном случае возвращает частично применённую функцию, которая фиксирует первые аргументы
  };
}
```

## Каррирование? Зачем?

Чтобы понять выгоду, нам определенно нужен пример из реальной жизни.

Продвинутое каррирование позволяет вызывать функцию, как обычно, так и с частичным применением.

Например, у нас есть функция логирования `log(date, importance, message)`, которая форматирует и выводит информацию. В реальных проектах у таких функций есть много других полезных возможностей, например, посылать логи по сети, здесь для простоты используем `alert`:

```js
function log(date, importance, message) {
  alert(`[${date.getHours()}:${date.getMinutes()}] [${importance}] ${message}`);
}
```

А теперь давайте применим к ней каррирование!

```js
log = _.curry(log);
```

После этого `log` продолжает работать нормально:

```js
log(new Date(), "DEBUG", "some debug");
```

...Но также работает вариант с каррированием:

```js
log(new Date(), "DEBUG", "some debug"); // log(a,b,c)
log(new Date())("DEBUG")("some debug"); // log(a)(b)(c)
```

Давайте сделаем удобную функцию для логов с текущим временем:

```js
// todayLog будет частичным применением функции log с фиксированным первым аргументом
let logNow = log(new Date());

// используем её
logNow("INFO", "message"); // [HH:mm] INFO message
```

А теперь удобная функция для логов отладки с текущим временем:

```js
let debugNow = logNow("DEBUG");

debugNow("message"); // [HH:mm] DEBUG message
```

Итак:
1. Мы ничего не потеряли после каррирования: `log` всё так же можно вызывать нормально.
2. Мы смогли создать частично применённые функции, как сделали для логов с текущим временем.

## Продвинутая реализация каррирования

В случае, если вам интересны детали (не обязательно!), вот "продвинутая" реализация каррирования, которую мы могли бы использовать выше.

Она очень короткая:

```js
function curry(func) {

  return function curried(...args) {
    if (args.length >= func.length) {
      return func.apply(this, args);
    } else {
      return function(...args2) {
        return curried.apply(this, args.concat(args2));
      }
    }
  };

}
```

Примеры использования:

```js
function sum(a, b, c) {
  return a + b + c;
}

let curriedSum = curry(sum);

alert( curriedSum(1, 2, 3) ); // 6, всё ещё можно вызывать нормально
alert( curriedSum(1)(2,3) ); // 6, каррирование первого аргумента
alert( curriedSum(1)(2)(3) ); // 6, каррирование всех аргументов
```

Новое `curry` может выглядеть сложным, но на самом деле его легко понять.

Результат `curry(func)` -- это обёртка `curried`, которая выглядит так:

```js
// func -- функция, которую мы трансформируем
function curried(...args) {
  if (args.length >= func.length) { // (1)
    return func.apply(this, args);
  } else {
    return function pass(...args2) { // (2)
      return curried.apply(this, args.concat(args2));
    }
  }
};
```

Когда мы запускаем её, есть два пути:

1. Вызвать сейчас: если количество переданных аргументов `args` совпадает с количеством аргументов при объявлении функции (`func.length`) или больше, тогда вызов просто переходит к ней.
2. Частичное применение: в противном случае `func` не вызывается сразу. Вместо этого, возвращается другая обёртка `pass`, которая снова применит `curried`, передав предыдущие аргументы вместе с новыми. Затем при новом вызове, мы опять получим, либо новое частичное применение (если аргументов не достаточно), либо, наконец, результат.

Например, давайте посмотрим, что произойдёт в случае `sum(a, b, c)`. У неё три аргумента, так что `sum.length = 3`.

Для вызова `curried(1)(2)(3)`:

1. Первый вызов `curried(1)` запоминает `1` в своём лексическом окружении и возвращает обёртку `pass`.
2. Обёртка `pass` вызывается с `(2)`: она берёт предыдущие аргументы (`1`), объединяет их с тем, что получила сама `(2)` и вызывает `curried(1, 2)` со всеми ними.

    Так как число аргументов всё ещё меньше 3-х, `curry` возвращает `pass`.
3. Обёртка `pass` вызывается снова с `(3)`. Для следующего вызова `pass(3)` берёт предыдущие аргументы (`1`, `2`) и добавляет к ним `3`, делая вызов `curried(1, 2, 3)` -- наконец 3 аргумента, и они передаются оригинальной функции.

Если всё ещё не понятно, просто распишите последовательность вызовов на бумаге.

```smart header="Только функции с фиксированным количеством аргументов"
Для каррирования необходима функция с известным фиксированным количеством аргументов.
```

```smart header="Немного больше, чем каррирование"
По определению, каррирование должно превращать `sum(a, b, c)` в `sum(a)(b)(c)`.

Но, как было описано, большинство реализаций каррирования в JavaScript более продвинуты: они также оставляют вариант вызова функции с несколькими аргументами.
```

## Итого

- Когда мы фиксируем некоторые аргументы какой-нибудь существующей функции, результатом (менее универсальным) будет функция *частично применённая*. Чтобы получить частичное применение, мы можем использовать `bind`, но есть и другие пути.

    Частичное применение удобно, когда мы не хотим повторять один и тот же аргумент много раз. Например, когда у нас есть функция `send(from, to)`, и `from` всё время будет одинаков для нашей задачи, мы можем создать частично применённую функцию и дальше работать с ней.

- *Каррирование* -- это трансформация, которая превращает вызов `f(a, b, c)` в `f(a)(b)(c)`. В JavaScript реализация обычно позволяет вызывать функцию обоими вариантами: либо нормально, либо возвращает частично применённую функцию, если количество аргументов недостаточно.

    Каррирование подходит тогда, когда мы хотим легко использовать частичное применение. Как мы видели в примерах с логами: универсальная функция `log(date, importance, message)` после каррирования возвращает нам частично применённую функцию, когда вызывается с одним аргументом, как `log(date)` или двумя аргументами, как `log(date, importance)`.
