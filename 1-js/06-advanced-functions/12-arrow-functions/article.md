# Повторяем стрелочные функции

Давайте вернёмся к стрелочным функциям.

Стрелочные функции это не просто "сокращение" чтобы меньше писать. У них есть ряд других полезных особенностей.

При написании JavaScript-кода часто возникают ситуации, когда нам нужно написать небольшую функцию, которая будет выполнена где-то ещё.

Например:

- `arr.forEach(func)` -- `func` выполняется `forEach` для каждого элемента массива.
- `setTimeout(func)` -- `func` выполняется встроенным планировщиком.
- ...и так далее.

Это очень в духе JavaScript - создать функцию и передать её куда-нибудь.

И в таких функциях мы обычно не хотим выходить из текущего контекста. Здесь как раз и полезна стрелочная функция.

## У стрелочных функций нет "this"

Как мы помним из главы <info:object-methods>, у стрелочных функций нет `this`. Если происходит обращение к `this`, его значение берётся снаружи.

Например, мы можем использовать это для итерации внутри метода объекта:

```js run
let group = {
  title: "Our Group",
  students: ["John", "Pete", "Alice"],

  showList() {
*!*
    this.students.forEach(
      student => alert(this.title + ': ' + student)
    );
*/!*
  }
};

group.showList();
```

Здесь внутри `forEach` использована стрелочная функции, таким образом `this.title` в ней будет иметь точно такое же значение, как в методе `showList`: `group.title`.

Если бы мы использовали "обычную" функцию, была бы ошибка:

```js run
let group = {
  title: "Our Group",
  students: ["John", "Pete", "Alice"],

  showList() {
*!*
    this.students.forEach(function(student) {
      // Error: Cannot read property 'title' of undefined
      alert(this.title + ': ' + student)
    });
*/!*
  }
};

group.showList();
```

Ошибка возникает потому, что `forEach` по умолчанию выполняет функции с `this`, равным `undefined`, и поэтому мы пытаемся обратиться к `undefined.title`.


Это не влияет на стрелочные функции, потому что у них просто нет `this`.

```warn header="Стрелочные функции нельзя использовать с `new`"
Отсутствие `this` естественно ведёт к другому ограничению: стрелочные функции не могут быть использованы как конструкторы. Они не могут быть вызваны с `new`.
```

```smart header="Стрелочные функции VS bind"
Существует тонкая разница между стрелочной функцией `=>` and обычной функцией вызванной с `.bind(this)`:

- `.bind(this)` создаёт "связанную версию" функции.
- Стрелка `=>` ничего не привязывает. У функции просто нет `this`. При получении значения `this` - оно, как обычная переменная, берётся из внешнего лексического окружения.
```

## Стрелочные функции не имеют "arguments"

У стрелочных функции также нет переменной `arguments`.

Это отлично подходит для декораторов, когда нам нужно пробросить вызов с текущими `this` и `arguments`.

Например, `defer(f, ms)` принимает функцию и возвращает обёртку над ней, которая откладывает вызов на `ms` миллисекунд:

```js run
function defer(f, ms) {
  return function() {
    setTimeout(() => f.apply(this, arguments), ms)
  };
}

function sayHi(who) {
  alert('Hello, ' + who);
}

let sayHiDeferred = defer(sayHi, 2000);
sayHiDeferred("John"); // Hello, John after 2 seconds
```

То же самое без стрелочной функции выглядело бы:

```js
function defer(f, ms) {
  return function(...args) {
    let ctx = this;
    setTimeout(function() {
      return f.apply(ctx, args);
    }, ms);
  };
}
```

Здесь мы были вынуждены создать дополнительные переменные `args` и `ctx`, чтобы функция внутри `setTimeout` могла получить их.

## Итого

Стрелочные функции:

- Не имеют `this`.
- Не имеют `arguments`.
- Не могут быть вызваны `new`.
- (У них также нет `super`, но мы про это не говорили. Про это будет в главе <info:class-inheritance>).

Все это потому что они предназначены для небольшого кода, который не имеет своего "контекста", а работает в текущем. И они отлично справляются с этой задачей.
