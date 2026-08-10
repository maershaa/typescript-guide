# 🚀 Міграція React + Vite проєкту з JavaScript на TypeScript

> Практичні нотатки з поступової міграції React + Vite проєкту
> з JavaScript на TypeScript.
---

## Зміст

1. [Базові типи](#базові-типи-в-typescript)
2. [Складні типи](#складні-типи)
3. [Union та Intersection](#union-type-)
4. [Tuple](#tuple-кортеж)
5. [Спеціальні типи: any, unknown](#спеціальні-типи)
6. [Enum](#enum)
7. [Типізація функцій](#типізація-функцій)
8. [type vs interface](#коротко-про-type-і-interface)
9. [Type Guards](#type-guards)
10. [Робота з HTML-елементами](#робота-з-html-елементами)
11. [Index Properties](#index-properties-індексні-сигнатури)
12. [Дженерики](#дженерики-generics)
13. [Utility Types](#utility-types)
14. [React: типізація компонентів](#react-типізація-компонентів)
15. [React: типізація подій](#react-типізація-подій)
16. [React: children — JSX.Element vs ReactNode](#react-children--jsxelement-vs-reactnode)
17. [Встановлення TypeScript у проєкт](#встановлення-typescript)
18. [Практика: порядок перейменування файлів](#порядок-перейменування-файлів-на-практиці)
19. [Redux Toolkit: типізація slice, store, селекторів](#redux-toolkit-типізація-slice-store-селекторів)
20. [RTK Query: типізація API](#rtk-query-типізація-api)

---

# Базові типи в TypeScript

> TypeScript вміє виводити (infer) типи автоматично, тому примітивні типи
> часто можна не вказувати явно.

## Примітивні типи

```ts
let name: string = 'Іван';
let age: number = 25;
let isOnline: boolean = true;
let emptyValue: null = null;
let notDefined: undefined = undefined;
```

---

# Складні типи

## Object

> Для об'єктів краще явно описувати їхню структуру, а не використовувати
> `object` чи `{}`. Тип `object` каже лише «це не примітив», але нічого не
> знає про поля всередині — тому TypeScript не зможе підказати автодоповнення
> або знайти помилку в назві властивості.

```ts
const obj: object = {};
const emptyObj: {} = {};

let user: {
  name: string;
  age?: number; // age — необов'язкова властивість
} = {
  name: 'Tom',
  age: 30,
};
```

---

## Array

### Масив рядків

```ts
let arrString: string[] = ['a', 'b', 'c'];
```

### Масив різних типів

```ts
let mixed: (number | string)[] = [1, 'two'];
```

### Масив об'єктів

```ts
type User = {
  name: string;
  age: number;
};

let users: User[] = [
  { name: 'Tom', age: 30 },
  { name: 'Jack', age: 25 },
  { name: 'Alice', age: 32 },
];
```

---

# Union Type (`|`)

Дозволяє вказати, що значення може мати **один з кількох типів**.

```ts
let mixedType: string | number | boolean;

mixedType = 'hello'; // ✅
mixedType = 42; // ✅
mixedType = true; // ✅
```

Використовується, коли заздалегідь відомо кілька допустимих варіантів,
наприклад статус запиту: `'loading' | 'success' | 'error'`.

---

# Intersection Type (`&`)

Об'єднує кілька типів в один — значення має відповідати **всім** типам
одразу.

```ts
type Employee = {
  name: string;
  id: number;
};

type Manager = {
  employees: Employee[];
};

type CEO = Employee & Manager;

const boss: CEO = {
  name: 'Anna',
  id: 1,
  employees: [{ name: 'Tom', id: 2 }],
};
```

Тепер `CEO` зобов'язаний містити властивості обох типів.

---

# Tuple (Кортеж)

Кортеж — це масив із **фіксованою кількістю елементів**, де заздалегідь
відомий тип кожного елемента.

```ts
let tupleType: [string, boolean];

tupleType = ['hello', true]; // ✅ OK
tupleType = [true, 'hello']; // ❌ Неправильний порядок типів
tupleType = ['hello', true, true]; // ❌ Зайвий елемент
```

Підходить для зберігання фіксованих наборів даних, наприклад:

- координати (`[x, y]`)
- дата (`[день, місяць, рік]`)
- відповідь API (`[data, error]`)

---

# Спеціальні типи

## `any`

Повністю вимикає перевірку типів — TypeScript перестає стежити за змінною
взагалі.

```ts
let value: any = 'Hello';

value = 10;
value = true;
```

Використовувати рекомендується лише в крайніх випадках (наприклад, як
тимчасова заглушка під час міграції), оскільки `any` знищує весь сенс
TypeScript.

## `unknown`

Схожий на `any`, але безпечніший: значення можна присвоїти будь-чому, але
**використати його не можна**, поки не перевіриш тип.

```ts
let value: unknown = 'Hello';

// ❌ Помилка: Type 'unknown' is not assignable to type 'string'
let text: string = value;
```

Перед використанням потрібно звузити (narrow) тип:

```ts
if (typeof value === 'string') {
  let text: string = value; // ✅ тут TS вже знає, що це string
}
```

**Правило:** якщо не впевнений, який тип прийде ззовні (наприклад, відповідь
API) — використовуй `unknown`, а не `any`.

---

# Enum

`enum` — це набір іменованих констант.

### Рядковий enum

```ts
enum UserStatus {
  Active = 'ACTIVE',
  Inactive = 'INACTIVE',
  Banned = 'BANNED',
}

let status: UserStatus = UserStatus.Active;
```

### Числовий enum

```ts
enum HttpCodes {
  OK = 200,
  BadRequest = 400,
  Unauthorized = 401,
}

const respond = (status: HttpCodes) => {
  // ...
};

respond(HttpCodes.OK);
```

> 💡 У сучасних React-проєктах enum часто замінюють на union-тип із рядків
> (`type UserStatus = 'ACTIVE' | 'INACTIVE' | 'BANNED'`) — це простіше і не
> додає зайвого коду в збірку.

---

# Типізація функцій

## Типізація параметрів

```ts
const sum = (a: number, b: number) => {
  return a + b;
};
```

## Типізація значення, що повертається

```ts
const sum = (a: number, b: number): number => {
  return a + b;
};
```

TypeScript зазвичай сам виводить тип результату, але явно вказувати його
корисно — так функція не зможе випадково почати повертати щось інше.

## Приклад із масивом об'єктів

```ts
type User = {
  id: number;
  name: string;
};

const getUserNames = (users: User[]): string[] => {
  return users.map(user => user.name);
};
```

Тут:

- `users: User[]` — функція приймає масив користувачів;
- `: string[]` — повертає масив рядків.

## `void`

`void` використовується, щоб позначити, що функція **нічого не повертає**.
Зазвичай застосовується для колбеків і обробників подій.

```ts
const logMessage = (message: string): void => {
  console.log(message);
};

const doSomething = (callback: () => void) => {
  callback();
};

doSomething(() => {
  console.log('Callback function!');
});
```

## `never`

`never` — тип функції, яка **ніколи не завершується нормально**: або завжди
кидає помилку, або йде в нескінченний цикл.

```ts
// Функція, яка завжди кидає помилку
const throwError = (message: string): never => {
  throw new Error(message);
};

// Функція з нескінченним циклом
const infiniteLoop = (): never => {
  while (true) {}
};
```

Відмінність від `void`: `void` — функція завершилась, просто нічого не
повернула. `never` — функція взагалі не може завершитись штатно.

---

# Коротко про `type` і `interface`

Обидва використовуються для опису структури даних.

## Type

Частіше використовується для:

- Union (`|`)
- Intersection (`&`)
- аліасів примітивів
- кортежів

```ts
type User = {
  name: string;
  age: number;
};
```

## Interface

Частіше використовують для опису об'єктів, пропсів React-компонентів і
класів.

```ts
interface User {
  name: string;
  age: number;
}
```

### Що обрати?

Для більшості React-проєктів обидва варіанти підходять.

Загальне правило:

- `interface` — опис об'єктів і пропсів;
- `type` — все інше (Union, Intersection, Tuple тощо).

---

# Type Guards

Type Guards у TypeScript — це інструменти, які допомагають TypeScript
зрозуміти, з яким саме типом ми працюємо всередині `if`, коли змінна описана
через Union Type.

Основні інструменти:

- `typeof` — перевірка примітивного типу
- `in` — перевірка наявності властивості в об'єкті
- `instanceof` — перевірка, чи є об'єкт екземпляром класу
- User-Defined Type Guards — власні функції-перевірки

### `typeof`

```ts
const printId = (id: string | number) => {
  if (typeof id === 'string') {
    console.log(id.toUpperCase()); // тут TS знає, що id — string
  } else {
    console.log(id.toFixed(2)); // тут TS знає, що id — number
  }
};
```

### `in`

```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

const makeSound = (animal: Cat | Dog) => {
  if ('meow' in animal) {
    animal.meow();
  } else {
    animal.bark();
  }
};
```

### `instanceof`

Працює з класами: перевіряє, чи був об'єкт створений цим класом.

```ts
class ApiError extends Error {}

const handleError = (error: Error) => {
  if (error instanceof ApiError) {
    console.log('Це помилка API');
  }
};
```

### User-Defined Type Guards

Власні функції-перевірки, які повертають спеціальний тип `arg is Type`. Це
корисно, коли звичайної перевірки `typeof`/`in` недостатньо.

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

const isFish = (pet: Fish | Bird): pet is Fish => {
  return (pet as Fish).swim !== undefined;
};

const move = (pet: Fish | Bird) => {
  if (isFish(pet)) {
    pet.swim(); // TS точно знає, що це Fish
  } else {
    pet.fly();
  }
};
```

---

# Робота з HTML-елементами

Під час роботи з DOM у TypeScript часто потрібно вказати конкретний тип
елемента — `document.getElementById` повертає `HTMLElement | null`, який
нічого не знає про `.value` чи інші специфічні властивості інпута.

```ts
const input = document.getElementById('inputEmail') as HTMLInputElement;
```

Є й інший синтаксис приведення типу — через кутові дужки:

```ts
const input = <HTMLInputElement>document.getElementById('inputEmail');
```

⚠️ Другий варіант **не підходить для `.tsx` файлів**, тому що TypeScript
плутає його з JSX-розміткою. У React-проєктах завжди використовуй `as`.

---

# Index Properties (індексні сигнатури)

Дозволяють описати об'єкт, у якого заздалегідь невідомі точні назви ключів,
але відомий їхній тип і тип значень.

```ts
type IndexType = {
  [prop: string]: string;
};

const colors: IndexType = {
  primary: '#fff',
  secondary: '#000',
  // будь-яка кількість ключів-рядків зі значеннями-рядками
};
```

---

# Дженерики (Generics)

Дженерики дозволяють писати функції та класи, які працюють із різними
типами, не втрачаючи при цьому строгу типізацію.

```ts
const identity = <T>(arg: T): T => {
  return arg;
};

const output1 = identity('myString'); // T = string
const output2 = identity(100); // T = number
```

> 💡 У `.tsx` файлах після `T` ставлять кому (`<T,>`), інакше TypeScript
> сплутає дженерик із відкриваючим JSX-тегом.

## Кілька дженериків

```ts
const merge = <T, U>(objA: T, objB: U) => {
  return Object.assign(objA, objB);
};

const merged = merge({ name: 'Alisa' }, { age: 28 });
console.log(merged);
// { name: "Alisa", age: 28 }
```

## Дженерики з `keyof`

`keyof` бере тип об'єкта і перетворює його на union із назв його ключів. Це
дозволяє обмежити `key` лише реально наявними полями об'єкта.

```ts
const extractValue = <T extends object, U extends keyof T>(obj: T, key: U) => {
  return obj[key];
};

extractValue({ name: 'John' }, 'name'); // ✅ 'name' — існуючий ключ
extractValue({ name: 'John' }, 'age'); // ❌ Помилка: 'age' немає в об'єкті
```

## Generic Classes

Дозволяють створювати класи, які можуть працювати з різними типами даних,
зберігаючи при цьому строгу типізацію — конкретний тип задається під час
створення екземпляра класу.

```ts
class DataStorage<T> {
  private data: T[] = [];

  addItem(item: T) {
    this.data.push(item);
  }

  getItems() {
    return [...this.data];
  }
}

const textStorage = new DataStorage<string>();
textStorage.addItem('Hello');
textStorage.addItem('World');
console.log(textStorage.getItems()); // ['Hello', 'World']
textStorage.addItem(1); // ❌ Error: number не підходить під string

const numberStorage = new DataStorage<number>();
numberStorage.addItem(1);
numberStorage.addItem(2);
console.log(numberStorage.getItems()); // [1, 2]
numberStorage.addItem('TEXT'); // ❌ Error: string не підходить під number
```

---

# Utility Types

Вбудовані типи-помічники, які беруть наявний тип і перетворюють його на новий
за певним правилом. Дуже зручні для типізації API та форм.

## `Partial<T>`

Робить усі властивості типу `T` **необов'язковими**. Чудово підходить для
часткового оновлення об'єкта (наприклад, метод `PATCH`).

```ts
type User = {
  id: number;
  name: string;
  email: string;
};

const updateUser = (id: number, changes: Partial<User>) => {
  // changes може містити будь-яку частину полів User, а може бути й порожнім {}
};

updateUser(1, { name: 'New Name' }); // ✅ не потрібно передавати всі поля
```

## `Readonly<T>`

Робить усі властивості в типі `T` доступними лише для читання. Після
створення об'єкта їх не можна змінити.

```ts
type User = {
  id: number;
  name: string;
  email: string;
};

const aliceReadonly: Readonly<User> = {
  id: 1,
  name: 'Alice',
  email: 'alice@example.com',
};

aliceReadonly.name = 'Bob';
// ❌ Error: Cannot assign to 'name' because it is a read-only property.
```

## `Pick<T, K>`

Формує новий тип лише з указаних полів. Часто використовується для
складання типів, наприклад під час роботи з API, звідки може прийти багато
полів, а потрібні не всі.

```ts
type User = {
  id: number;
  name: string;
  email: string;
};

type UserBasicInfo = Pick<User, 'id' | 'name'>;
// { id: number; name: string }
```

## `Omit<T, K>`

Дозволяє створити новий тип на основі типу `T`, виключивши набір
властивостей, указаних у `K`.

```ts
type Person = {
  name: string;
  age: number;
  location: string;
};

type PersonWithoutLocation = Omit<Person, 'location'>;
// { name: string; age: number }
```

## `Record<K, T>`

Дозволяє описати об'єкт, у якого ключі заздалегідь відомі, а значення мають
один і той самий тип.

- `K` — множина можливих ключів.
- `T` — тип кожного значення.

```ts
type User = {
  id: number;
  name: string;
};

const users: Record<number, User> = {
  1: { id: 1, name: 'Alisa' },
  2: { id: 2, name: 'Ivan' },
};
```

Або з конкретним набором рядкових ключів:

```ts
type Status = 'loading' | 'success' | 'error';

const labels: Record<Status, string> = {
  loading: 'Завантаження...',
  success: 'Готово',
  error: 'Помилка',
};
```

## `ReturnType<T>`

Дозволяє отримати тип, який повертає функція. Обов'язково використовується
разом із `typeof`, тому що `ReturnType` очікує _тип функції_, а не саму
функцію.

```ts
const greeting = () => {
  return 'Hello, world!';
};

type Greeting = ReturnType<typeof greeting>; // string

const multiply = (a: number, b: number) => {
  return a * b;
};

type MultiplyResult = ReturnType<typeof multiply>; // number
```

Зручно, коли тип результату функції складний і не хочеться дублювати його
вручну в окремому `type`.

## `Parameters<T>`

Дістає типи параметрів функції у вигляді кортежу. Корисно, коли потрібно
перевикористати «форму» аргументів іншої функції, не переписуючи її вручну —
наприклад, щоб обгорнути функцію у власну обгортку з тими самими аргументами.

```ts
type MyFunctionType = (a: string, b: number, c: boolean) => void;

type MyParametersType = Parameters<MyFunctionType>;
// [string, number, boolean]

// Практичний приклад: обгортка-логер навколо наявної функції
const originalFn = (name: string, age: number) => {
  console.log(name, age);
};

const withLogging = (...args: Parameters<typeof originalFn>) => {
  console.log('Виклик з аргументами:', args);
  originalFn(...args);
};
```

## `NonNullable<T>`

Прибирає `null` і `undefined` з типу `T`. Корисний, коли потрібно
гарантувати, що далі в коді значення точно не порожнє.

```ts
type MaybeString = string | null | undefined;

type DefiniteString = NonNullable<MaybeString>;
// string

const printLength = (value: DefiniteString) => {
  console.log(value.length); // можна не перевіряти на null — TS вже впевнений
};
```

## Індексний доступ до типу поля — `Type['key']`

Не зовсім «утилітний тип» у звичному сенсі, але вкрай корисний прийом:
дозволяє взяти тип конкретного поля з іншого типу/інтерфейсу, замість того
щоб дублювати його вручну.

```ts
interface Contact {
  id: string;
  name: string;
}

// замість того щоб писати id: string руками
const deleteContact = (id: Contact['id']) => {
  // ...
};
```

Якщо завтра `id` в `Contact` зміниться на інший тип — усі місця, які
посилаються на `Contact['id']`, підхоплять зміну автоматично, без ручного
пошуку й заміни по всьому проєкту.

---

# React: типізація компонентів

## Чи потрібно використовувати `FC<Props>`?

Раніше типізація React-компонента через `React.FC<Props>` вважалася
стандартом:

```tsx
import { FC, JSX } from 'react';

type SectionProps = {
  title?: string;
  children: JSX.Element;
};

const Section: FC<SectionProps> = ({ title, children }) => {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
};

export { Section };
```

Сьогодні в спільноті радять **не використовувати `FC`**, а типізувати пропси
напряму в параметрі функції:

```tsx
type SectionProps = {
  title?: string;
  children: JSX.Element;
};

const Section = ({ title, children }: SectionProps) => {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
};

export { Section };
```

**Чому так краще:**

- `FC` не дає явної переваги — тип значення, що повертається (`JSX.Element`),
  TS і так коректно виводить сам.
- У старих версіях React-типів `FC` неявно додавав `children` до пропсів,
  навіть якщо компонент їх не приймав — джерело плутанини.
- `FC` ускладнює роботу з дженерик-компонентами.

Це правило діє **незалежно від того, є в компонента пропси чи ні** — навіть
без пропсів не потрібно писати `const Component: FC = () => {...}`, достатньо
`const Component = () => {...}`.

---

## Явно вказувати тип у `useState<T>` чи довіряти інференції?

Правило просте: **не дублюй те, що TS і так виведе з початкового значення.**

Якщо початкове значення однозначно визначає тип — анотація зайва:

```ts
const [name, setName] = useState<string>(''); // <string> тут зайвий
const [name, setName] = useState(''); // TS сам виведе string
```

Явна анотація **обов'язкова**, коли реальний тип ширший, ніж те, що можна
вивести з початкового значення:

```ts
// Без <..> TS виведе тип як просто '' (widening вимкнено), а не потрібний union
const [gender, setGender] = useState<'male' | 'female' | ''>('');
```

Так само обов'язкова анотація для `null`-початкових значень — без неї TS
виведе тип змінної як буквально `null`, і покласти туди об'єкт буде не можна:

```ts
const [user, setUser] = useState<User | null>(null);
```

---

# React: типізація подій

| React-обробник           | Тип події           | Коли використовувати           |
| ------------------------ | -------------------- | ------------------------------ |
| `onChange`                | `ChangeEvent<T>`     | input, select, textarea        |
| `onSubmit` / `onReset`   | `FormEvent<T>`\*     | `<form>`                       |
| `onClick`                 | `MouseEvent<T>`      | кнопки, посилання, будь-який елемент |
| `onKeyDown`/`Up`/`Press` | `KeyboardEvent<T>`   | інпути, документ               |
| `onFocus`/`onBlur`       | `FocusEvent<T>`      | інпути                         |

\* Починаючи з `@types/react` v19.2.10 `FormEvent`/`FormEventHandler`
позначені як `@deprecated` на користь `SubmitEvent`/`SubmitEventHandler`.
Старий тип поки продовжує працювати, але краще одразу використовувати новий:

```ts
const handleFormSubmit = (e: SubmitEvent<HTMLFormElement>) => {
  e.preventDefault();
  // ...
};
```

**Загальне правило вибору:**

- Обробник — це JSX-проп (`onX={...}`) → бери тип з `React.XEvent`.
- Працюєш з `ref.current.addEventListener(...)` напряму, минаючи JSX → бери
  нативний DOM-тип, без префікса `React.`.

---

## `*Element` vs `*Attributes`

| Тип                        | Що описує                                                                                                                | Де використовується                                                                             |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `HTMLInputElement`          | Реальний `<input>`-елемент у DOM: властивості (`value`, `checked`, `disabled`) і методи (`focus()`, `select()`)          | У `ChangeEvent<HTMLInputElement>`, у `useRef<HTMLInputElement>(null)`                             |
| `InputHTMLAttributes<T>`    | Набір HTML-атрибутів, які можна передати в JSX `<input>` (`placeholder`, `value`, `onChange`, `type`, `disabled`)         | Власний компонент-обгортка над `<input>`, щоб прийняти стандартні пропси без ручного перерахування |
| `HTMLFormElement`           | Реальний `<form>`-елемент: методи `.reset()`, `.submit()`, властивість `.elements`                                       | `FormEvent<HTMLFormElement>` / `SubmitEvent<HTMLFormElement>`, `useRef<HTMLFormElement>(null)`    |
| `FormHTMLAttributes<T>`     | Атрибути для JSX `<form>` (`onSubmit`, `action`, `method`, `noValidate`)                                                  | Власний компонент-обгортка над `<form>`                                                            |
| `HTMLButtonElement`         | Реальний `<button>`-елемент: `.disabled`, `.form`, `.type`                                                               | `useRef<HTMLButtonElement>(null)`, події на кнопці                                                 |
| `ButtonHTMLAttributes<T>`   | Атрибути для JSX `<button>` (`onClick`, `disabled`, `type`)                                                              | Власний компонент-кнопка                                                                           |
| `HTMLSelectElement`         | Реальний `<select>`-елемент: `.value`, `.selectedIndex`, `.options`                                                      | `ChangeEvent<HTMLSelectElement>`, `useRef<HTMLSelectElement>(null)`                                |
| `SelectHTMLAttributes<T>`   | Атрибути для JSX `<select>` (`onChange`, `multiple`, `value`)                                                            | Власний компонент-обгортка над `<select>`                                                          |

**Правило вибору однією фразою:** якщо працюєш з подією (`ChangeEvent`,
`SubmitEvent`) чи `ref` — використовуєш `*Element`. Якщо пишеш власний
перевикористовуваний компонент і розширюєш його пропсами рідного HTML-тега
(`extends InputHTMLAttributes<...>`) — використовуєш `*Attributes`.

Приклад перевикористовуваного `<Input />`:

```tsx
interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label: string;
}

const Input = ({ label, ...rest }: InputProps) => (
  <div>
    <label>{label}</label>
    <input {...rest} />
  </div>
);
```

`InputHTMLAttributes<HTMLInputElement>` дає всі стандартні пропси `<input>`
«безкоштовно» — не потрібно вручну писати `value?: string`,
`onChange?: ...`, `placeholder?: string` і так далі.

---

# React: `children` — `JSX.Element` vs `ReactNode`

| Тип                     | Що дозволяє                                                                                                                     | Що забороняє                                                                                            |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `JSX.Element`             | Рівно один готовий React-елемент (`<div/>`, `<Component/>`)                                                                        | Рядки, числа, `null`, `undefined`, `boolean`, масиви елементів, умовний рендер (`{cond && <div/>}`)          |
| `ReactElement<P>`        | Те саме, що `JSX.Element`, але можна вказати тип пропсів конкретного елемента (`ReactElement<ButtonProps>`)                        | Те саме, що й `JSX.Element`                                                                                  |
| `ReactNode`               | Взагалі все, що React здатен відрендерити: елементи, рядки, числа, `null`, `undefined`, `boolean`, масиви, фрагменти, портали      | Практично нічого не забороняє — «дозволити все»                                                              |
| `PropsWithChildren<T>`   | Не окремий тип, а утиліта: автоматично додає `children?: ReactNode` до пропс-типу                                                  | —                                                                                                             |

## Коли що використовувати

**`ReactNode`** — найчастіший вибір для універсальних обгорток-контейнерів
(layout, Card, Modal), куди реально може прийти що завгодно:

```tsx
interface CardProps {
  children: ReactNode;
}
// <Card>Просто текст</Card>       — ок
// <Card>{count}</Card>            — ок, число
// <Card><div/><div/></Card>       — ок, кілька елементів
```

**`JSX.Element`** — коли компонент вимагає рівно один конкретний елемент, і
потрібно, щоб TS забороняв передавати «сміття» (рядок помилково, `null` з
умовного рендера). Гарний приклад — `PrivateRoute`, де принципово важливо,
щоб прийшов саме захищений маршрут:

```tsx
interface PrivateRouteProps {
  children: JSX.Element;
}

const PrivateRoute = ({ children }: PrivateRouteProps) => {
  const isLoggedIn = useSelector(selectIsLoggedIn);
  return isLoggedIn ? children : <Navigate to="/auth" replace />;
};
```

**`PropsWithChildren<T>`** — скорочення, щоб не писати `children: ReactNode`
руками щоразу:

```tsx
type Props = PropsWithChildren<{ title: string }>;
// еквівалентно
type Props = { title: string; children?: ReactNode };
```

⚠️ `children` у `PropsWithChildren` **опціональний**. Якщо `children` у
компоненті обов'язковий (як у `Section` чи `PrivateRoute`) —
`PropsWithChildren` без змін не підходить, потрібен явний
`children: ReactNode`.

---

# Що важливо запам'ятати

| Що                                | Для чого                              |
| ---------------------------------- | -------------------------------------- |
| `string`, `number`, `boolean`      | Примітивні типи                        |
| `object`                           | Об'єкт (краще описувати явно)          |
| `User[]`                           | Масив об'єктів                         |
| `A \| B`                           | Один із кількох типів                  |
| `A & B`                            | Об'єднання типів                       |
| `[string, number]`                 | Кортеж                                 |
| `any`                              | Вимикає перевірку типів                |
| `unknown`                          | Безпечна альтернатива `any`            |
| `enum`                             | Набір констант                         |
| `type`                             | Універсальний аліас типів              |
| `interface`                        | Опис структури об'єктів                |
| `void`                             | Функція нічого не повертає             |
| `never`                            | Функція ніколи не завершується         |
| `Partial<T>` / `Pick<T,K>` тощо    | Готові Utility Types                   |
| `Type['key']`                      | Тип конкретного поля з іншого типу     |

---

# Встановлення TypeScript

## 1. Встановлюємо TypeScript

Якщо проєкт створений на **React + Vite**, встановлюємо необхідні пакети:

```bash
npm install -D typescript @types/react @types/react-dom
```

Створюємо файл конфігурації:

```bash
npx tsc --init
```

## 2. Налаштовуємо tsconfig.json

Замінюємо вміст файлу на такий:

```jsonc
{
  "compilerOptions": {
    "target": "ESNext", // Використовувати можливості сучасного JavaScript.

    "module": "ESNext", // Використовувати ES Modules.
    "moduleResolution": "Bundler", // Шукати модулі так само, як Vite.

    "jsx": "react-jsx", // Новий JSX-трансформер React (не потрібно імпортувати React у кожному файлі).
    "jsxImportSource": "@emotion/react", // Каже TS брати типи JSX з Emotion, а не з React — потрібно для коректної типізації styled-компонентів і css-пропа.

    "strict": true, // Строга перевірка типів.

    "baseUrl": ".", // Корінь проєкту для абсолютних імпортів.

    "paths": {
      "@/*": ["src/*"], // Аліас @ → src.
    },

    "allowJs": true, // Дозволити використовувати .js та .jsx.
    "checkJs": false, // Не перевіряти JS-файли на помилки типів.

    "resolveJsonModule": true, // Дозволити імпорт JSON.

    "isolatedModules": true, // Перевіряти кожен файл окремо (вимога Vite).

    "noEmit": true, // Не компілювати JS. Це робить Vite.

    "skipLibCheck": true, // Не перевіряти типи бібліотек.
  },

  "include": ["src"], // Перевіряти лише папку src.
}
```

## 3. Видаляємо jsconfig.json

Після появи `tsconfig.json` файл `jsconfig.json` більше не потрібен.

Чому? Тому що налаштування

```jsonc
"allowJs": true,
"checkJs": false
```

дозволяють TypeScript одночасно працювати з `.js`, `.jsx`, `.ts` і `.tsx`.
Тому проєкт можна переносити поступово, а не переписувати все одразу.

## 4. Підключаємо ambient-типи для Vite

TypeScript за замовчуванням нічого не знає про те, як імпортувати `.css`,
`.svg` та інші не-JS/TS файли. Щоб це виправити, всередині `src/` створюємо
файл `vite-env.d.ts`:

```ts
/// <reference types="vite/client" />
```

Цей рядок підключає ambient-типи із самого пакета `vite`, які оголошують
модулі `*.css`, `*.svg`, `import.meta.env` тощо.

**Якщо після створення файлу помилка не зникла — перевір по кроках:**

1. Файл лежить саме всередині `src/` (там, куди дивиться `"include": ["src"]`
   в `tsconfig.json`), а не в корені проєкту.
2. Перезапусти TS-сервер у редакторі: `Cmd+Shift+P` →
   `TypeScript: Restart TS Server` — VSCode іноді кешує стан мовної служби.
3. Гарантований fallback, якщо нічого не допомогло — оголосити модуль явно:
   ```ts
   // src/types/css.d.ts
   declare module '*.css';
   ```

## 5. Перейменовуємо перші файли

Спочатку достатньо перейменувати:

```text
main.jsx → main.tsx
App.jsx  → App.tsx
```

Після цього TypeScript уже починає працювати в проєкті.

## 6. Виправляємо main.tsx

Було (JavaScript):

```jsx
createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

Стало (TypeScript):

```tsx
const root = document.getElementById('root');

if (!root) {
  throw new Error('Root element not found');
}

createRoot(root).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

### Чому?

Тому що `document.getElementById()` може повернути `HTMLElement | null`.
TypeScript не дасть викликати `createRoot(null)` і змушує явно обробити цей
випадок — або перевіркою (як вище), або приведенням типу через `as`:

```tsx
const root = document.getElementById('root') as HTMLElement;

createRoot(root).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

## 7. Поступово переносимо проєкт

Рекомендований порядок:

```text
✅ main.tsx
        ↓
✅ App.tsx
        ↓
✅ store.ts
        ↓
✅ Redux slices
        ↓
✅ hooks
        ↓
✅ pages
        ↓
✅ components
        ↓
✅ utils
```

Не потрібно перейменовувати все одразу.

## 8. Під час міграції

Проєкт може виглядати так, і це абсолютно нормально:

```text
src/
│
├── App.tsx
├── main.tsx
│
├── redux/
│   ├── store.ts
│   ├── authSlice.js
│   └── filter.js
│
├── components/
│   ├── Button.tsx
│   ├── Header.jsx
│   └── Loader.tsx
```

## 9. Після повної міграції

Коли в проєкті більше не залишиться файлів `.js` та `.jsx`, можна:

- видалити `"allowJs": true`;
- видалити `"checkJs": false`;
- видалити всі інші згадки JS у конфігурації.

Після цього проєкт стане повністю TypeScript-проєктом, і `strict: true`
перевірятиме абсолютно весь код.

---

# Порядок перейменування файлів на практиці

Не потрібно одразу міняти всі файли. Наприклад:

**Сьогодні:**

```text
main.jsx → main.tsx
App.jsx  → App.tsx
```

**Завтра:**

```text
Header.jsx → Header.tsx
```

**Післязавтра:**

```text
Button.jsx → Button.tsx
```

І так далі, файл за файлом, у своєму темпі.

## Styled Components

# Налаштування типізованої теми для Emotion (TypeScript)

Інструкція для проєктів на React + TypeScript + Emotion (`@emotion/styled`,
`@emotion/react`). Актуально й для `styled-components` — відмінності
позначені окремо.

Проблема, яку вирішуємо: за замовчуванням параметр `theme` у
`styled.div\`...\`` типізований порожнім службовим інтерфейсом бібліотеки
(`Theme` у emotion / `DefaultTheme` у styled-components). Тому TS не бачить
твої кастомні поля (`colors`, `spacing` тощо), навіть якщо об'єкт теми в тебе
правильний і все працює в рантаймі.

Рішення — **declaration merging** (злиття оголошень): ми «доповнюємо»
службовий інтерфейс бібліотеки власними полями.

---

## Крок 1. Виносимо тип теми в окремий файл і типізуємо сам об'єкт теми

Заводимо окремий файл з інтерфейсом (це єдине джерело правди для форми теми
— використовується і в об'єкті теми, і в module augmentation на кроці 2):

```ts
// src/theme/theme.types.ts

export interface Theme {
  colors: {
    background: { [key: string]: string };
    text: { [key: string]: string };
    accent: { [key: string]: string };
    decorations: { [key: string]: string };
    border: { [key: string]: string };
  };
  shadows: { [key: string]: string };
  radii: { [key: string]: string };
  spacing: { [key: string]: string };
  fonts: { [key: string]: string };
}
```

Застосовуємо цей тип до самого об'єкта теми:

```ts
// src/theme/theme.ts
import { Theme } from './theme.types';

export const theme: Theme = Object.freeze({
  colors: {
    background: { main: '#fff' },
    text: { primary: '#4f46e5' },
    accent: { blue: '#667eea' },
    decorations: { star: '#FFE066' },
    border: { light: '#E0D9F0' },
  },
  shadows: { card: '0px 4px 12px rgba(0,0,0,0.08)' },
  radii: { small: '6px' },
  spacing: { sm: '8px' },
  fonts: { main: "'Inter', sans-serif" },
});
```

> Навіщо окремий файл `theme.types.ts`, а не інтерфейс прямо в `theme.ts`?
> Щоб на кроці 2 імпортувати той самий тип, а не дублювати його структуру
> вручну ще раз у `emotion.d.ts`.

---

## Крок 2. Розширюємо службовий тип бібліотеки (module augmentation)

Створюємо файл `emotion.d.ts` (можна й `.ts` — але `.d.ts` — загальноприйнята
угода для файлів, які лише оголошують типи й не містять runtime-коду).

```ts
// src/emotion.d.ts
import '@emotion/react';
import { Theme as AppTheme } from './theme/theme.types';

declare module '@emotion/react' {
  export interface Theme extends AppTheme {}
}
```

**Для `styled-components` замість цього:**

```ts
// src/styled.d.ts
import 'styled-components';
import { Theme as AppTheme } from './theme/theme.types';

declare module 'styled-components' {
  export interface DefaultTheme extends AppTheme {}
}
```

### Важливо перевірити

- [ ] Файл лежить всередині `src` (або іншої директорії, вказаної в секції
      `include` `tsconfig.json`).
- [ ] На початку файлу є `import '@emotion/react'` (або
      `'styled-components'`) — без нього файл не вважається модулем, і
      augmentation не спрацює.
- [ ] Використовується саме `interface`, а не `type` — лише інтерфейси
      підтримують declaration merging.

---

## Крок 3. Перейменовуємо файли стилів `.js`/`.jsx` → `.ts`

Файли на кшталт `Footer.styled.js` → `Footer.styled.ts`.

Якщо файл зі styled-компонентами містить **лише** виклики `styled.xxx` без
JSX-розмітки всередині самого файлу — достатньо `.ts`. Розширення `.tsx`
потрібне лише там, де у файлі є безпосередньо JSX-синтаксис (`<div>...</div>`
поза шаблонними рядками styled).

Після перейменування:

```ts
// src/components/Footer/Footer.styled.ts
import styled from '@emotion/styled';

export const FooterWrapper = styled.footer`
  padding: 0 ${({ theme }) => theme.spacing.lg};
  border-radius: ${({ theme }) => theme.radii.medium};
`;
```

Тепер `theme` у колбеку автоматично має тип із кроку 2 — автодоповнення та
перевірка типів працюють без помилки
`Property 'spacing' does not exist on type 'Theme'`.

---

## Як перевірити, що все підключилося

1. Наведи курсор на `theme` всередині `({ theme }) => theme...` у будь-якому
   styled-файлі.
2. VSCode має показати твій повний тип (`colors`, `spacing`, `radii`, ...), а
   не порожній `{}` / `DefaultTheme`.
3. Спробуй набрати `theme.` всередині шаблонного рядка — має з'явитися
   автодоповнення за твоїми реальними полями.

---

## Полiморфний `as` prop у styled-компонентів

Часта помилка під час спроби відрендерити styled-компонент як інший елемент
— наприклад, `<Link>` з `react-router-dom` замість звичайного `<a>`:

```tsx
const Logo = styled.a`
  /* ... */
`;

<Logo as={Link} to="/" aria-label="Logo of the project">
```

TypeScript лається приблизно так:

> Свойство "to" не существует в типе "... & AnchorHTMLAttributes<...>"

**Чому так відбувається.** Тип пропсів styled-компонента фіксується в момент
його створення — `styled.a` знає лише про
`AnchorHTMLAttributes<HTMLAnchorElement>` (і `theme`). Проп `as` змінює
елемент у рантаймі, але статична типізація `styled.a` про це нічого не знає й
не підтягує автоматично пропси компонента, який передано в `as` (у цьому
випадку — `LinkProps` з `react-router-dom`, звідки й береться `to`).
Поліморфізм у рантаймі є, а в типах — ні, якщо не підключити його окремо.

**Рішення — стилізувати сам компонент, а не елемент.** Замість `styled.a` з
подальшим `as={Link}` компонент обгортається напряму:

```tsx
import { Link } from 'react-router-dom';
import styled from '@emotion/styled';

const Logo = styled(Link)`
  /* ті самі стилі */
`;
```

```tsx
<Logo to="/" aria-label="Logo of the project">
```

Коли `styled()` обгортає не рядок-тег (`'a'`), а сам React-компонент
(`Link`), він забирає тип пропсів саме в нього — тобто `Logo` автоматично
знає про `to`, `replace`, `state` і всі інші пропси `Link`, без `as` і без
ручних дженериків. Це і є стандартний підхід: `as` у styled-components/
Emotion зручний у рантаймі, але для типобезпеки надійніше заздалегідь
вирішити, яким компонентом рендериться styled-обгортка, і обгорнути саме
його.

---

## Підсумкова структура файлів

```
src/
├─ theme/
│  ├─ theme.types.ts   ← інтерфейс Theme (єдине джерело правди)
│  └─ theme.ts         ← сам об'єкт теми, типізований через Theme
├─ emotion.d.ts         ← declaration merging (module augmentation)
└─ components/
   └─ Footer/
      └─ Footer.styled.ts
```

## Звичайні файли

```text
theme.js     → theme.ts
constants.js → constants.ts
utils.js     → utils.ts
```

## Компоненти

Просте правило:

- `.ts` — файл **не містить** JSX;
- `.tsx` — файл **містить** JSX.

```text
Component.jsx → Component.tsx   (повертає JSX)
helper.js     → helper.ts       (JSX немає, лише функції)
```

## Що станеться після перейменування?

TypeScript одразу покаже помилки — але лише в цьому конкретному файлі, не в
усьому проєкті. Наприклад:

```tsx
const Button = ({ title }) => {};
```

TypeScript скаже:

> Parameter 'title' implicitly has an 'any' type.

Це означає, що потрібно просто додати тип:

```tsx
type ButtonProps = {
  title: string;
};

const Button = ({ title }: ButtonProps) => {
  return <button>{title}</button>;
};
```

І так — файл за файлом, ти поступово додаєш типи в міру того, як TypeScript
їх запитує.

# Redux Toolkit: типізація slice, store, селекторів

## Типізація slice

Slice складається з трьох речей, які потрібно типізувати: форма стану,
`action.payload` у кожному редьюсері та (за замовчуванням) сам
`initialState`.

```ts
interface FilterState {
  value: string;
}

const initialState: FilterState = { value: '' };

const filterSlice = createSlice({
  name: 'filter',
  initialState,
  reducers: {
    setFilter: (state, action: PayloadAction<string>) => {
      state.value = action.payload;
    },
  },
});
```

`PayloadAction<T>` — дженерик з Redux Toolkit, де `T` — тип того, що лежить у
`action.payload`. Без нього `action` мав би тип `AnyAction`, і
`action.payload` був би `any` — тобто можна було б присвоїти що завгодно
куди завгодно без жодної помилки компілятора.

Якщо стан вкладений — сутність (наприклад, користувача) зручніше виносити в
окремий `interface`, а не описувати інлайном. Тоді її можна перевикористати
в інших місцях — наприклад, у типі, що повертає селектор:

```ts
interface AuthUser {
  login: string;
  password: string;
}

interface AuthState {
  isLoggedIn: boolean;
  user: AuthUser | null;
}
```

Тип і `interface`/`type`, який його описує, прийнято називати в `PascalCase`
(`AuthState`, `FilterState`) — так їх одразу видно серед звичайних змінних і
функцій.

---

## `RootState` і `AppDispatch`

Це два ключові типи для будь-якого Redux + TypeScript проєкту. `RootState` —
тип усього стану застосунку (потрібен селекторам і `useSelector`).
`AppDispatch` — тип функції `dispatch` конкретно цього store (потрібен для
типізованого `useDispatch`, особливо коли в middleware додані thunk-и чи
RTK Query).

Обидва типи не пишуться вручну — вони виводяться автоматично із самого store
через `ReturnType` і `typeof`:

```ts
import { configureStore } from '@reduxjs/toolkit';
import { contactsApi } from '@/redux/services/contactsApi';
import authReducer from './authSlice';
import filterReducer from './filterSlice';

const store = configureStore({
  reducer: {
    [contactsApi.reducerPath]: contactsApi.reducer,
    filter: filterReducer,
    auth: authReducer,
  },
  middleware: getDefaultMiddleware =>
    getDefaultMiddleware().concat(contactsApi.middleware),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

export default store;
```

Чому саме так:

- `store.getState` — це функція, яка повертає об'єкт стану.
  `ReturnType<typeof store.getState>` читається як «тип того, що повертає ця
  функція» — тобто точна форма всього state, автоматично зібрана з усіх
  редьюсерів одразу. Якщо завтра додасться новий slice, `RootState`
  оновиться сам, без ручного дописування.
- `typeof store.dispatch` бере тип самої функції `dispatch` цього
  конкретного store — з усіма можливостями, які додають підключені
  middleware (у тому числі RTK Query).
- Щоб `typeof store...` взагалі спрацював, store має бути окремою змінною, а
  не результатом, що одразу вилітає в `export default` — `typeof`
  посилається на змінну за іменем, посилатися нема на що, якщо імені нема.
  Далі `RootState` імпортується туди, де потрібно знати форму state — у
  першу чергу в селектори:

```ts
// selectors.ts
import type { RootState } from './store';

export const selectUser = (state: RootState) => state.auth.user;
export const selectIsLoggedIn = (state: RootState) => state.auth.isLoggedIn;
export const selectFilter = (state: RootState) => state.filter.value;
```

Використано `import type`, а не звичайний `import` — тому що з `store.ts`
потрібен лише тип, а не щось, що реально виконується в рантаймі. `import
type` гарантовано видаляється на етапі збірки: у фінальному JS-файлі такого
імпорту взагалі не буде. Це корисно у двох сенсах: по-перше, чітко видно з
коду, що залежність суто «типова»; по-друге, це знімає будь-які побоювання
щодо циклічних імпортів між `store.ts` і файлами, які імпортують з нього
`RootState`, — імпорт, що стирається під час збірки, фізично не може
зациклитися в рантаймі.

З типізованими селекторами `state` всередині селектора отримує
автодоповнення та захист від помилок у назвах полів, а все, що бере значення
через `useSelector(selectUser)`, автоматично отримує правильний тип без
ручних анотацій.

---

## Типізовані хуки `useAppDispatch` / `useAppSelector`

`useDispatch` і `useSelector` з `react-redux` за замовчуванням нічого не
знають про конкретний store конкретного проєкту — `dispatch` без дженерика
не в курсі про thunk-и та middleware, а `state` всередині `useSelector` без
дженерика — `any`.

Стандартний підхід — один раз завести власні типізовані версії цих хуків і
далі по всьому проєкту використовувати лише їх:

```ts
// src/redux/hooks.ts
import { useDispatch, useSelector } from 'react-redux';
import type { TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

```tsx
import { useAppDispatch, useAppSelector } from '@/redux/hooks';
import { selectUser } from '@/redux/selectors';

const dispatch = useAppDispatch();
const user = useAppSelector(selectUser); // одразу правильний тип, без generics вручну
```

Плюс такого підходу: типізація робиться один раз у `hooks.ts`, а не
повторюється дженериками в кожному компоненті, який звертається до store.

---

# RTK Query: типізація API

## `builder.query<ResultType, ArgType>` і `builder.mutation<ResultType, ArgType>`

Кожен ендпоінт у `createApi` — це дженерик із двома параметрами: що запит
**повертає** і що він **приймає як аргумент**. Вказувати їх потрібно завжди
явно — якщо пропустити, TypeScript підставить `unknown`, і всередині `query`
аргумент не матиме взагалі жодних відомих полів, а результат запиту не
можна буде використати напряму без додаткових перевірок.

`builder.mutation<A, B>` — це як функція з двома «слотами» для типів, і
порядок фіксований:

Перший параметр (`A`) — що поверне сервер після виконання запиту (те, що
опиниться в `data` після успішної відповіді). Другий параметр (`B`) — що ти
передаєш у саму мутацію, коли її викликаєш у компоненті.

```ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import { Contact } from '@/interfaces';

type ContactsList = Contact[];
type NewContact = Omit<Contact, 'id' | 'isFavorite'>;

export const contactsApi = createApi({
  reducerPath: 'contactsApi',
  tagTypes: ['Contact'],
  baseQuery: fetchBaseQuery({
    baseUrl: 'https://65e95c314bb72f0a9c513d32.mockapi.io',
  }),
  endpoints: builder => ({
    getContacts: builder.query<ContactsList, void>({
      query: () => `/contacts`,
      providesTags: ['Contact'],
    }),

    addContact: builder.mutation<Contact, NewContact>({
      query: newContact => ({
        url: `/contacts`,
        method: 'POST',
        body: newContact,
      }),
      invalidatesTags: ['Contact'],
    }),

    deleteContact: builder.mutation<{ id: string }, string>({
      query: id => ({
        url: `/contacts/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: ['Contact'],
    }),

    toggleFavorite: builder.mutation<Contact, Contact>({
      query: contact => {
        const { id, isFavorite } = contact;
        return {
          url: `/contacts/${id}`,
          method: 'PUT',
          body: { ...contact, isFavorite: !isFavorite },
        };
      },
      invalidatesTags: ['Contact'],
    }),
  }),
});

export const {
  useGetContactsQuery,
  useAddContactMutation,
  useDeleteContactMutation,
  useToggleFavoriteMutation,
} = contactsApi;
```

Кілька моментів, чому це виглядає саме так:

- Коли запиту не потрібен аргумент (`getContacts`), другим параметром
  ставиться `void`, а не `{}`. `{}` у TypeScript означає «майже будь-яке
  значення, крім `null`/`undefined`» — це не «порожній об'єкт» і точно не
  «аргументів немає». `void` — це явно «аргумент не потрібен», і викликати
  `useGetContactsQuery()` з якимось сміттям усередині вже не вийде.
- Тип аргументу мутації — це те, що реально приходить у `query`. Для
  видалення приходить просто `id` (рядок), тому другим параметром стоїть
  `string`, а не весь об'єкт контакту.
- Для `addContact` аргумент — не `Contact`, а
  `Omit<Contact, 'id' | 'isFavorite'>`. `Omit<T, Keys>` — Utility Type, який
  бере тип `T` і прибирає з нього перелічені поля. `id` та `isFavorite`
  обчислюються на клієнті (`crypto.randomUUID()`, `false` за замовчуванням)
  і не приходять із форми — отже, вимагати їх в аргументі було б неправильно.
  Усе, що повертають згенеровані хуки, вже типізовано на основі цих
  дженериків і не потребує ручної анотації:

```ts
const { data, isLoading, isError } = useGetContactsQuery();
// data: ContactsList | undefined
```

---

## Тегування за id, а не лише за типом

Це вже не про TypeScript, а про сам RTK Query — але раз мова про те, «як на
проді», варто знати. Один спільний тег на весь список
(`providesTags: ['Contact']`, `invalidatesTags: ['Contact']`) означає, що
після **будь-якої** мутації весь список запитується заново повністю.

Точніший варіант — тегувати кожен елемент окремо за `id`, тоді після мутації
перезапитується лише змінений запис, а не весь список:

```ts
getContacts: builder.query<ContactsList, void>({
  query: () => `/contacts`,
  providesTags: result =>
    result
      ? [
          ...result.map(({ id }) => ({ type: 'Contact' as const, id })),
          { type: 'Contact' as const, id: 'LIST' },
        ]
      : [{ type: 'Contact' as const, id: 'LIST' }],
}),

toggleFavorite: builder.mutation<Contact, Contact>({
  query: contact => ({ /* ... */ }),
  invalidatesTags: (result, error, contact) => [
    { type: 'Contact', id: contact.id },
  ],
}),
```

Для невеликого списку різниці не видно, але при зростанні кількості даних це
економить зайві запити до сервера.
