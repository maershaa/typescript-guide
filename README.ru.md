# 🚀 JavaScript ➔ TypeScript: Пошаговый гайд по миграции React + Vite

---

## Содержание

1. [Базовые типы](#базовые-типы-в-typescript)
2. [Сложные типы](#сложные-типы) — `object`, массивы, вложенные структуры
3. [Union и Intersection](#union-type-) — Union (`|`) один из нескольких типов, Intersection (`&`) несколько типов сразу
4. [Tuple](#tuple-кортеж) — массив фиксированной длины с известным типом каждого элемента
5. [Специальные типы: any, unknown](#специальные-типы)
6. [Enum](#enum) — именованный набор констант
7. [Типизация функций](#типизация-функций) — параметры, возврат, `void`, `never`
8. [type vs interface](#кратко-про-type-и-interface)
9. [Type Guards](#type-guards) — сужение типа внутри `if` (`typeof`, `in`, `instanceof`)
10. [Index Properties](#index-properties-индексные-сигнатуры) — объект с заранее неизвестными ключами
11. [Дженерики](#дженерики-generics) — типы-параметры для переиспользуемых функций/классов
12. [Utility Types](#utility-types) — `Partial`, `Pick`, `Omit`, `Record` и другие встроенные хелперы
13. [React: типизация компонентов](#react-типизация-компонентов) — пропсы, `useState`, `Dispatch<SetStateAction<T>>`
14. [React: типизация событий](#react-типизация-событий) — `onChange`, `onSubmit`, `onClick` и общие типы событий для формы
15. [React: children — JSX.Element vs ReactNode](#react-children--jsxelement-vs-reactnode)
16. [Установка TypeScript в проект](#установка-typescript)
17. [Практика: порядок переименования файлов](#порядок-переименования-файлов-на-практике)
    - [Как проверить, что проект полностью на TypeScript?](#как-проверить-что-проект-полностью-на-typescript)
18. [Redux Toolkit: типизация slice, store, селекторов](#redux-toolkit-типизация-slice-store-селекторов)
19. [RTK Query: типизация API](#rtk-query-типизация-api)
20. [Редко используется / устаревшее](#редко-используется--устаревшее) — работа с HTML через `as`, типизированная тема styled-компонентов

---

# Базовые типы в TypeScript

> TypeScript умеет выводить (infer) типы автоматически, поэтому примитивные типы
> часто можно не указывать явно.

## Примитивные типы

```ts
let name: string = 'Иван';
let age: number = 25;
let isOnline: boolean = true;
let emptyValue: null = null;
let notDefined: undefined = undefined;
```

---

# Сложные типы

## Object

> Для объектов лучше явно описывать их структуру, а не использовать `object` или
> `{}`. Тип `object` говорит только «это не примитив», но ничего не знает о
> полях внутри — поэтому TypeScript не сможет подсказать автодополнение или
> найти опечатку в названии свойства.

```ts
const obj: object = {};
const emptyObj: {} = {};

let user: {
  name: string;
  age?: number; // age — необязательное свойство
} = {
  name: 'Tom',
  age: 30,
};
```

---

## Array

### Массив строк

```ts
let arrString: string[] = ['a', 'b', 'c'];
```

### Массив разных типов

```ts
let mixed: (number | string)[] = [1, 'two'];
```

### Массив объектов

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

Позволяет указать, что значение может иметь **один из нескольких типов**.

```ts
let mixedType: string | number | boolean;

mixedType = 'hello'; // ✅
mixedType = 42; // ✅
mixedType = true; // ✅
```

Используется, когда заранее известно несколько допустимых вариантов, например
статус запроса: `'loading' | 'success' | 'error'`.

---

# Intersection Type (`&`)

Объединяет несколько типов в один — значение должно соответствовать **всем**
типам сразу.

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

Теперь `CEO` обязан содержать свойства обоих типов.

---

# Tuple (Кортеж)

Кортеж — это массив с **фиксированным количеством элементов**, где заранее
известен тип каждого элемента.

```ts
let tupleType: [string, boolean];

tupleType = ['hello', true]; // ✅ OK
tupleType = [true, 'hello']; // ❌ Неверный порядок типов
tupleType = ['hello', true, true]; // ❌ Лишний элемент
```

Подходит для хранения фиксированных наборов данных, например:

- координаты (`[x, y]`)
- дата (`[день, месяц, год]`)
- ответ API (`[data, error]`)

---

# Специальные типы

## `any`

Полностью отключает проверку типов — TypeScript перестаёт следить за переменной
вообще.

```ts
let value: any = 'Hello';

value = 10;
value = true;
```

Использовать рекомендуется только в крайних случаях (например, при временной
заглушке во время миграции), так как `any` убивает весь смысл TypeScript.

## `unknown`

Похож на `any`, но безопаснее: значение можно присвоить чему угодно, но
**использовать его нельзя**, пока не проверишь тип.

```ts
let value: unknown = 'Hello';

// ❌ Ошибка: Type 'unknown' is not assignable to type 'string'
let text: string = value;
```

Перед использованием необходимо сузить (narrow) тип:

```ts
if (typeof value === 'string') {
  let text: string = value; // ✅ здесь TS уже знает, что это string
}
```

**Правило:** если не уверен, какой тип придёт извне (например, ответ API) —
используй `unknown`, а не `any`.

---

# Enum

`enum` — это набор именованных констант.

### Строковый enum

```ts
enum UserStatus {
  Active = 'ACTIVE',
  Inactive = 'INACTIVE',
  Banned = 'BANNED',
}

let status: UserStatus = UserStatus.Active;
```

### Числовой enum

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

> 💡 В современных React-проектах enum часто заменяют на union-тип из строк
> (`type UserStatus = 'ACTIVE' | 'INACTIVE' | 'BANNED'`) — это проще и не
> добавляет лишнего кода в сборку.

---

# Типизация функций

## Типизация параметров

```ts
const sum = (a: number, b: number) => {
  return a + b;
};
```

## Типизация возвращаемого значения

```ts
const sum = (a: number, b: number): number => {
  return a + b;
};
```

TypeScript обычно сам выводит тип возврата, но явно указывать его полезно — так
функция не сможет случайно начать возвращать что-то другое.

## Пример с массивом объектов

```ts
type User = {
  id: number;
  name: string;
};

const getUserNames = (users: User[]): string[] => {
  return users.map((user) => user.name);
};
```

Здесь:

- `users: User[]` — функция принимает массив пользователей;
- `: string[]` — возвращает массив строк.

## `void`

`void` используется для обозначения того, что функция **ничего не возвращает**.
Обычно применяется для колбэков и обработчиков событий.

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

`never` — тип функции, которая **никогда не завершается нормально**: либо всегда
выбрасывает ошибку, либо уходит в бесконечный цикл.

```ts
// Функция, которая всегда выбрасывает ошибку
const throwError = (message: string): never => {
  throw new Error(message);
};

// Функция с бесконечным циклом
const infiniteLoop = (): never => {
  while (true) {}
};
```

Отличие от `void`: `void` — функция завершилась, просто ничего не вернула.
`never` — функция вообще не может завершиться штатно.

---

# Кратко про `type` и `interface`

Оба используются для описания структуры данных.

## Type

Чаще используется для:

- Union (`|`)
- Intersection (`&`)
- алиасов примитивов
- кортежей

```ts
type User = {
  name: string;
  age: number;
};
```

## Interface

Чаще используют для описания объектов, пропсов React-компонентов и классов.

```ts
interface User {
  name: string;
  age: number;
}
```

### Что выбрать?

Для большинства React-проектов оба варианта подходят.

Общее правило:

- `interface` — описание объектов и пропсов;
- `type` — всё остальное (Union, Intersection, Tuple и т.д.).

---

# Type Guards

Type Guards в TypeScript — это инструменты, которые помогают TypeScript понять,
с каким именно типом мы работаем внутри `if`, когда переменная описана через
Union Type.

Основные инструменты:

- `typeof` — проверка примитивного типа
- `in` — проверка наличия свойства в объекте
- `instanceof` — проверка, является ли объект экземпляром класса
- User-Defined Type Guards — собственные функции-проверки

### `typeof`

```ts
const printId = (id: string | number) => {
  if (typeof id === 'string') {
    console.log(id.toUpperCase()); // тут TS знает, что id — string
  } else {
    console.log(id.toFixed(2)); // тут TS знает, что id — number
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

Работает с классами: проверяет, был ли объект создан этим классом.

```ts
class ApiError extends Error {}

const handleError = (error: Error) => {
  if (error instanceof ApiError) {
    console.log('Это ошибка API');
  }
};
```

### User-Defined Type Guards

Свои функции-проверки, которые возвращают специальный тип `arg is Type`. Это
полезно, когда обычной проверки `typeof`/`in` недостаточно.

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

const isFish = (pet: Fish | Bird): pet is Fish => {
  return (pet as Fish).swim !== undefined;
};

const move = (pet: Fish | Bird) => {
  if (isFish(pet)) {
    pet.swim(); // TS точно знает, что это Fish
  } else {
    pet.fly();
  }
};
```

---

# Index Properties (индексные сигнатуры)

Позволяют описать объект, у которого заранее неизвестны точные названия ключей,
но известен их тип и тип значений.

```ts
type IndexType = {
  [prop: string]: string;
};

const colors: IndexType = {
  primary: '#fff',
  secondary: '#000',
  // любое количество ключей-строк со значениями-строками
};
```

---

# Дженерики (Generics)

Дженерики позволяют писать функции и классы, которые работают с разными типами,
не теряя при этом строгую типизацию.

```ts
const identity = <T>(arg: T): T => {
  return arg;
};

const output1 = identity('myString'); // T = string
const output2 = identity(100); // T = number
```

> 💡 В `.tsx` файлах после `T` ставят запятую (`<T,>`), иначе TypeScript
> перепутает дженерик с открывающим JSX-тегом.

## Несколько дженериков

```ts
const merge = <T, U>(objA: T, objB: U) => {
  return Object.assign(objA, objB);
};

const merged = merge({ name: 'Alisa' }, { age: 28 });
console.log(merged);
// { name: "Alisa", age: 28 }
```

## Дженерики с `keyof`

`keyof` берёт тип объекта и превращает его в union из названий его ключей. Это
позволяет ограничить `key` только реально существующими полями объекта.

```ts
const extractValue = <T extends object, U extends keyof T>(obj: T, key: U) => {
  return obj[key];
};

extractValue({ name: 'John' }, 'name'); // ✅ 'name' — существующий ключ
extractValue({ name: 'John' }, 'age'); // ❌ Ошибка: 'age' нет в объекте
```

## Generic Classes

Позволяют создавать классы, которые могут работать с разными типами данных,
сохраняя при этом строгую типизацию — конкретный тип задаётся при создании
экземпляра класса.

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
textStorage.addItem(1); // ❌ Error: number не подходит под string

const numberStorage = new DataStorage<number>();
numberStorage.addItem(1);
numberStorage.addItem(2);
console.log(numberStorage.getItems()); // [1, 2]
numberStorage.addItem('TEXT'); // ❌ Error: string не подходит под number
```

---

# Utility Types

Встроенные типы-помощники, которые берут существующий тип и превращают его в
новый по определённому правилу. Очень удобны для типизации API и форм.

## `Partial<T>`

Делает все свойства типа `T` **необязательными**. Отлично подходит для
частичного обновления объекта (например, метод `PATCH`).

```ts
type User = {
  id: number;
  name: string;
  email: string;
};

const updateUser = (id: number, changes: Partial<User>) => {
  // changes может содержать любую часть полей User, а может быть и пустым {}
};

updateUser(1, { name: 'New Name' }); // ✅ не нужно передавать все поля
```

## `Readonly<T>`

Делает все свойства в типе `T` доступными только для чтения. После создания
объекта их нельзя изменить.

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

Формирует новый тип только из указанных полей. Часто используется для
составления типов, например при работе с API, откуда может прийти множество
полей, а нужны не все.

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

Позволяет создать новый тип на основе типа `T`, исключив набор свойств,
указанных в `K`.

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

Позволяет описать объект, у которого ключи заранее известны, а значения имеют
один и тот же тип.

- `K` — множество возможных ключей.
- `T` — тип каждого значения.

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

Или с конкретным набором строковых ключей:

```ts
type Status = 'loading' | 'success' | 'error';

const labels: Record<Status, string> = {
  loading: 'Загрузка...',
  success: 'Готово',
  error: 'Ошибка',
};
```

Удобный частый кейс — состояние формы (`touched`, `errors` и т.д.), где для
каждого поля нужно одно и то же примитивное значение:

```typescript
export type LogInFormTouched = Record<'email' | 'password', boolean>;

const touched: LogInFormTouched = {
  email: false,
  password: false,
};
```

Это короче, чем писать `{ email: boolean; password: boolean }` вручную, и при
добавлении нового поля формы достаточно дописать его в union ключей.

## `ReturnType<T>`

Позволяет получить тип, который возвращает функция. Обязательно используется
вместе с `typeof`, потому что `ReturnType` ожидает _тип функции_, а не саму
функцию.

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

Удобно, когда тип возврата функции сложный и не хочется дублировать его вручную
в отдельном `type`.

## `Parameters<T>`

Достаёт типы параметров функции в виде кортежа. Полезно, когда нужно
переиспользовать «форму» аргументов другой функции, не переписывая её вручную —
например, чтобы обернуть функцию в свою обёртку с теми же аргументами.

```ts
type MyFunctionType = (a: string, b: number, c: boolean) => void;

type MyParametersType = Parameters<MyFunctionType>;
// [string, number, boolean]

// Практический пример: обёртка-логгер вокруг существующей функции
const originalFn = (name: string, age: number) => {
  console.log(name, age);
};

const withLogging = (...args: Parameters<typeof originalFn>) => {
  console.log('Вызов с аргументами:', args);
  originalFn(...args);
};
```

## `NonNullable<T>`

Убирает `null` и `undefined` из типа `T`. Полезен, когда нужно гарантировать,
что дальше по коду значение точно не пустое.

```ts
type MaybeString = string | null | undefined;

type DefiniteString = NonNullable<MaybeString>;
// string

const printLength = (value: DefiniteString) => {
  console.log(value.length); // можно не проверять на null — TS уже уверен
};
```

## Индексный доступ к типу поля — `Type['key']`

Не совсем «утилитный тип» в привычном смысле, но крайне полезный приём:
позволяет взять тип конкретного поля из другого типа/интерфейса, вместо того
чтобы дублировать его вручную.

```ts
interface Contact {
  id: string;
  name: string;
}

// вместо того чтобы писать id: string руками
const deleteContact = (id: Contact['id']) => {
  // ...
};
```

Если завтра `id` в `Contact` поменяется на другой тип — все места, которые
ссылаются на `Contact['id']`, подхватят изменение автоматически, без ручного
поиска и замены по всему проекту.

---

# React: типизация компонентов

## Нужно ли использовать `FC<Props>`?

Раньше типизация React-компонента через `React.FC<Props>` считалась стандартом:

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

Сегодня в комьюнити рекомендуют **не использовать `FC`**, а типизировать пропсы
напрямую в параметре функции:

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

**Почему так лучше:**

- `FC` явного профита не даёт — тип возвращаемого значения (`JSX.Element`) TS и
  так корректно выводит сам.
- В старых версиях React-типов `FC` неявно добавлял `children` в пропсы, даже
  если компонент их не принимал — источник путаницы.
- `FC` усложняет работу с дженерик-компонентами.

Это правило действует **независимо от того, есть у компонента пропсы или нет** —
даже без пропсов не нужно писать `const Component: FC = () => {...}`, достаточно
`const Component = () => {...}`.

---

## Явно указывать тип в `useState<T>` или доверять инференции?

Правило простое: **не дублируй то, что TS и так выведет из начального
значения.**

Если начальное значение однозначно определяет тип — аннотация избыточна:

```ts
const [name, setName] = useState<string>(''); // <string> здесь лишний
const [name, setName] = useState(''); // TS сам выведет string
```

Явная аннотация **обязательна**, когда реальный тип шире, чем то, что можно
вывести из начального значения:

```ts
// Без <..> TS выведет тип как просто '' (widening отключён), а не нужный union
const [gender, setGender] = useState<'male' | 'female' | ''>('');
```

Так же обязательна аннотация для `null`-начальных значений — без неё TS выведет
тип переменной как буквально `null`, и положить туда объект будет нельзя:

```ts
const [user, setUser] = useState<User | null>(null);
```

---

## Передача `setState` в дочерний компонент — `Dispatch<SetStateAction<T>>`

Когда сеттер из `useState` передаётся пропом в дочерний компонент (а не
используется прямо там, где вызван `useState`), для него нужен явный тип —
вывести его из места объявления TS уже не может. Для этого есть пара типов из
`react`: `Dispatch<T>` — тип функции-диспетчера, а `SetStateAction<T>` — тип
её аргумента (значение **или** функция-апдейтер вида `prev => next`).

```typescriptreact
import type { Dispatch, SetStateAction } from 'react';

interface IngredientsProps {
  setRecipeForm: Dispatch<SetStateAction<RecipeFormState>>;
}
```

Внутри дочернего компонента сеттер работает как обычно, в том числе с
функциональным обновлением:

```typescriptreact
setRecipeForm((prevValue) => ({
  ...prevValue,
  ingredients: prevValue.ingredients.filter(
    (ingredient) => ingredient.id !== id,
  ),
}));
```

`Dispatch<SetStateAction<RecipeFormState>>` — это ровно тот тип, который сам
TypeScript вывел бы для `setRecipeForm`, будь он объявлен через `useState`
прямо в этом компоненте. Писать его вручную нужно именно потому, что здесь
`setRecipeForm` — это проп, а не результат `useState`.

---

# React: типизация событий

| React-обработчик        | Тип события        | Чаще всего вешается на                                            |
| ----------------------- | ------------------ | ----------------------------------------------------------------- |
| `onChange`              | `ChangeEvent<T>`   | `HTMLInputElement`, `HTMLSelectElement`, `HTMLTextAreaElement`    |
| `onSubmit` / `onReset`  | `SubmitEvent<T>`\* | `HTMLFormElement`                                                 |
| `onClick`               | `MouseEvent<T>`    | `HTMLButtonElement`, `HTMLAnchorElement`, но вообще любой элемент |
| `onFocus` / `onBlur`    | `FocusEvent<T>`    | `HTMLInputElement`, `HTMLSelectElement`, `HTMLTextAreaElement`    |
| `onKeyDown` / `onKeyUp` | `KeyboardEvent<T>` | инпуты, `document`                                                |

`T` — это тип конкретного HTML-элемента, на котором висит обработчик
(например `ChangeEvent<HTMLInputElement>`).

\* Начиная с `@types/react` v19.2.10 `FormEvent`/`FormEventHandler` помечены
как `@deprecated` в пользу `SubmitEvent`/`SubmitEventHandler`. Старый тип пока
продолжает работать (просто покажет предупреждение), но в новом коде лучше
сразу брать актуальный:

```typescriptreact
import type { SubmitEvent } from 'react';

const handleFormSubmit = (e: SubmitEvent<HTMLFormElement>) => {
  e.preventDefault();
  // ...
};
```

**Общее правило выбора:**

- Обработчик — это JSX-проп (`onX={...}`) → бери тип из `React.XEvent`.
- Работаешь с `ref.current.addEventListener(...)` напрямую, минуя JSX → бери
  нативный DOM-тип, без префикса `React.`.

## Примеры

```typescriptreact
const handleFormSubmit = (e: SubmitEvent<HTMLFormElement>) => {
  e.preventDefault();
  // логика отправки формы
};

const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
  const { name, value } = e.target;
  setForm(prev => ({ ...prev, [name]: value }));
};

const handleOnBlur = (e: FocusEvent<HTMLInputElement>) => {
  const { name } = e.target;
  setTouched(prev => ({ ...prev, [name]: true }));
};
```

## Общий тип события для нескольких видов полей

Если одна и та же форма собрана из разных типов полей (`input`, `select`,
`textarea`), а обработчик всё равно один общий — удобно завести свой алиас
события один раз и переиспользовать его во всех дочерних компонентах формы,
вместо того чтобы в каждом писать union из трёх элементов заново:

```typescript
export type FieldChangeEvent = ChangeEvent<
  HTMLInputElement | HTMLSelectElement | HTMLTextAreaElement
>;

export type FieldBlurEvent = FocusEvent<
  HTMLInputElement | HTMLSelectElement | HTMLTextAreaElement
>;
```

Дальше в компонентах формы достаточно `(e: FieldChangeEvent) => { ... }` —
без повторения union-а элементов в каждом файле.

---

## `*Element` vs `*Attributes`

| Тип                       | Что описывает                                                                                                         | Где используется                                                                                |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `HTMLInputElement`        | Реальный `<input>`-элемент в DOM: свойства (`value`, `checked`, `disabled`) и методы (`focus()`, `select()`)          | В `ChangeEvent<HTMLInputElement>`, в `useRef<HTMLInputElement>(null)`                           |
| `InputHTMLAttributes<T>`  | Набор HTML-атрибутов, которые можно передать в JSX `<input>` (`placeholder`, `value`, `onChange`, `type`, `disabled`) | Свой компонент-обёртка над `<input>`, чтобы принять стандартные пропсы без ручного перечисления |
| `HTMLFormElement`         | Реальный `<form>`-элемент: методы `.reset()`, `.submit()`, свойство `.elements`                                       | `FormEvent<HTMLFormElement>` / `SubmitEvent<HTMLFormElement>`, `useRef<HTMLFormElement>(null)`  |
| `FormHTMLAttributes<T>`   | Атрибуты для JSX `<form>` (`onSubmit`, `action`, `method`, `noValidate`)                                              | Свой компонент-обёртка над `<form>`                                                             |
| `HTMLButtonElement`       | Реальный `<button>`-элемент: `.disabled`, `.form`, `.type`                                                            | `useRef<HTMLButtonElement>(null)`, события на кнопке                                            |
| `ButtonHTMLAttributes<T>` | Атрибуты для JSX `<button>` (`onClick`, `disabled`, `type`)                                                           | Свой компонент-кнопка                                                                           |
| `HTMLSelectElement`       | Реальный `<select>`-элемент: `.value`, `.selectedIndex`, `.options`                                                   | `ChangeEvent<HTMLSelectElement>`, `useRef<HTMLSelectElement>(null)`                             |
| `SelectHTMLAttributes<T>` | Атрибуты для JSX `<select>` (`onChange`, `multiple`, `value`)                                                         | Свой компонент-обёртка над `<select>`                                                           |

**Правило выбора одной фразой:** если работаешь с событием (`ChangeEvent`,
`SubmitEvent`) или `ref` — используешь `*Element`. Если пишешь свой
переиспользуемый компонент и расширяешь его пропсами родного HTML-тега
(`extends InputHTMLAttributes<...>`) — используешь `*Attributes`.

Пример переиспользуемого `<Input />`:

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

`InputHTMLAttributes<HTMLInputElement>` даёт все стандартные пропсы `<input>`
«бесплатно» — не нужно вручную писать `value?: string`, `onChange?: ...`,
`placeholder?: string` и так далее.

---

# React: `children` — `JSX.Element` vs `ReactNode`

Коротко: `children` может быть **или тем, или другим** — тип зависит от того,
что реально должно прийти в компонент.

- **`ReactNode`** — если допустимо «что угодно, что умеет рендерить React»:
  текст, число, `null`, массив элементов, условный рендер (`{cond && <div/>}`).
  Это выбор по умолчанию для универсальных обёрток (layout, Card, Modal).
- **`JSX.Element`** — если нужен ровно **один готовый React-элемент**, и важно,
  чтобы TS не пропустил строку, `null` или результат условного рендера по
  ошибке. Пример — `PrivateRoute`, где принципиально, что придёт именно
  защищаемый маршрут, а не что-то ещё:

```tsx
interface PrivateRouteProps {
  children: JSX.Element;
}

const PrivateRoute = ({ children }: PrivateRouteProps) => {
  const isLoggedIn = useSelector(selectIsLoggedIn);
  return isLoggedIn ? children : <Navigate to="/auth" replace />;
};
```

`PropsWithChildren<T>` — утилита, которая избавляет от ручного дописывания
`children: ReactNode`:

```tsx
type Props = PropsWithChildren<{ title: string }>;
// эквивалентно: { title: string; children?: ReactNode }
```

⚠️ `children` в `PropsWithChildren` **опционален**. Если он обязателен (как в
`PrivateRoute` выше) — нужен явный `children: ReactNode`, а не
`PropsWithChildren` без изменений.

---

# Что важно запомнить

| Что                               | Для чего                             |
| --------------------------------- | ------------------------------------ |
| `string`, `number`, `boolean`     | Примитивные типы                     |
| `object`                          | Объект (лучше описывать явно)        |
| `User[]`                          | Массив объектов                      |
| `A \| B`                          | Один из нескольких типов             |
| `A & B`                           | Объединение типов                    |
| `[string, number]`                | Кортеж                               |
| `any`                             | Отключает проверку типов             |
| `unknown`                         | Безопасная альтернатива `any`        |
| `enum`                            | Набор констант                       |
| `type`                            | Универсальный алиас типов            |
| `interface`                       | Описание структуры объектов          |
| `void`                            | Функция ничего не возвращает         |
| `never`                           | Функция никогда не завершается       |
| `Partial<T>` / `Pick<T,K>` и т.д. | Готовые Utility Types                |
| `Type['key']`                     | Тип конкретного поля из другого типа |

---

# Установка TypeScript

## 1. Устанавливаем TypeScript

Если проект создан на **React + Vite**, устанавливаем необходимые пакеты:

```bash
npm install -D typescript @types/react @types/react-dom
```

Создаём файл конфигурации:

```bash
npx tsc --init
```

## 2. Настраиваем tsconfig.json

Заменяем содержимое файла на следующее:

```jsonc
{
  "compilerOptions": {
    "target": "ESNext", // Использовать возможности современного JavaScript.

    "module": "ESNext", // Использовать ES Modules.
    "moduleResolution": "Bundler", // Искать модули так же, как Vite.

    "jsx": "react-jsx", // Новый JSX-трансформер React (не требует импортировать React в каждом файле).
    "jsxImportSource": "@emotion/react", // Указывает TS брать типы JSX из Emotion, а не из React — нужно для корректной типизации styled-компонентов и css-пропа.

    "strict": true, // Строгая проверка типов.

    "baseUrl": ".", // Корень проекта для абсолютных импортов.

    "paths": {
      "@/*": ["src/*"], // Алиас @ → src.
    },

    "allowJs": true, // Разрешить использовать .js и .jsx.
    "checkJs": false, // Не проверять JS-файлы на ошибки типов.

    "resolveJsonModule": true, // Разрешить импорт JSON.

    "isolatedModules": true, // Проверять каждый файл отдельно (требование Vite).

    "noEmit": true, // Не компилировать JS. Это делает Vite.

    "skipLibCheck": true, // Не проверять типы библиотек.

    "ignoreDeprecations": "6.0", // Скрыть предупреждение о будущем устаревании baseUrl.
  },

  "include": ["src"], // Проверять только папку src.
}
```

## 3. Удаляем jsconfig.json

После появления `tsconfig.json` файл `jsconfig.json` больше не нужен.

Почему? Потому что настройки

```jsonc
"allowJs": true,
"checkJs": false
```

позволяют TypeScript работать одновременно с `.js`, `.jsx`, `.ts` и `.tsx`.
Поэтому проект можно переносить постепенно, а не переписывать всё разом.

## 4. Подключаем ambient-типы для Vite

TypeScript по умолчанию ничего не знает о том, как импортировать `.css`, `.svg`
и другие не-JS/TS файлы. Чтобы это исправить, внутри `src/` создаём файл
`vite-env.d.ts`:

```ts
/// <reference types="vite/client" />
```

Эта строка подключает ambient-типы из самого пакета `vite`, которые объявляют
модули `*.css`, `*.svg`, `import.meta.env` и т.д.

**Если после создания файла ошибка не исчезла — проверь по шагам:**

1. Файл лежит именно внутри `src/` (там, куда смотрит `"include": ["src"]` в
   `tsconfig.json`), а не в корне проекта.
2. Перезапусти TS-сервер в редакторе: `Cmd+Shift+P` →
   `TypeScript: Restart TS Server` — VSCode иногда кэширует состояние языкового
   сервиса.
3. Гарантированный fallback, если ничего не помогло — объявить модуль явно:
   ```ts
   // src/types/css.d.ts
   declare module '*.css';
   ```

## 5. Переименовываем первые файлы

Сначала достаточно переименовать:

```text
main.jsx → main.tsx
App.jsx  → App.tsx
```

После этого TypeScript уже начинает работать в проекте.

## 6. Исправляем main.tsx

Было (JavaScript):

```jsx
createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
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
  </StrictMode>,
);
```

### Почему?

Потому что `document.getElementById()` может вернуть `HTMLElement | null`.
TypeScript не даст вызвать `createRoot(null)` и заставляет явно обработать этот случай.

## 7. Постепенно переносим проект

Рекомендуемый порядок:

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

Не нужно переименовывать всё сразу.

## 8. Во время миграции

Проект может выглядеть так, и это абсолютно нормально:

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

## 9. После полной миграции

Когда в проекте больше не останется файлов `.js` и `.jsx`, можно:

- удалить `"allowJs": true`;
- удалить `"checkJs": false`;
- удалить все оставшиеся упоминания JS в конфигурации.

После этого проект станет полностью TypeScript-проектом, и `strict: true` будет
проверять абсолютно весь код.

---

# Порядок переименования файлов на практике

## Обычные файлы

```text
theme.js     → theme.ts
constants.js → constants.ts
utils.js     → utils.ts
```

## Компоненты

Простое правило:

- `.ts` — файл **не содержит** JSX;
- `.tsx` — файл **содержит** JSX.

```text
Component.jsx → Component.tsx   (возвращает JSX)
helper.js     → helper.ts       (JSX нет, просто функции)
```

## Что произойдёт после переименования?

TypeScript сразу покажет ошибки — но только в этом конкретном файле, не во всём
проекте. Например:

```tsx
const Button = ({ title }) => {};
```

TypeScript скажет:

> Parameter 'title' implicitly has an 'any' type.

Это значит, что нужно просто добавить тип:

```tsx
type ButtonProps = {
  title: string;
};

const Button = ({ title }: ButtonProps) => {
  return <button>{title}</button>;
};
```

И так — файл за файлом, ты постепенно добавляешь типы по мере того, как
TypeScript их запрашивает.

## Как проверить, что проект полностью на TypeScript

Открытые вкладки в VSCode — ненадёжный способ следить за ошибками: язык-сервис
проверяет только файлы, которые сейчас открыты, поэтому закрытый файл с
ошибкой просто перестаёт мозолить глаза, а не исправляется.

**1. Прогнать компилятор по всему проекту**

```bash
npx tsc --noEmit
```

`--noEmit` — не создавать `.js`-файлы (это всё равно делает Vite), а только
проверить типы. Команда обходит **все** файлы, попадающие под `include` в
`tsconfig.json`, а не только открытые в редакторе, и выводит полный список
ошибок сразу. Если вывод пустой — проект действительно чист.

Удобно вынести в `package.json`, чтобы не набирать каждый раз:

```json
{
  "scripts": {
    "type-check": "tsc --noEmit"
  }
}
```

```bash
npm run type-check
```

**2. Убедиться, что `.js`/`.jsx`-файлов не осталось**

```bash
find src -name "*.js" -o -name "*.jsx"
```

Если команда ничего не вывела — миграция завершена: можно смело удалять
`"allowJs": true` и `"checkJs": false` из `tsconfig.json` (после этого `strict`
начнёт проверять уже весь код без исключений).

---

# Redux Toolkit: типизация slice, store, селекторов

## Типизация slice

Slice состоит из трёх вещей, которые нужно типизировать: форма состояния,
`action.payload` в каждом редьюсере и (по умолчанию) сам `initialState`.

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

`PayloadAction<T>` — дженерик из Redux Toolkit, где `T` — тип того, что лежит в
`action.payload`. Без него `action` имел бы тип `AnyAction`, и `action.payload`
был бы `any` — то есть можно было бы присвоить что угодно куда угодно без единой
ошибки компилятора.

Если состояние вложенное — сущность (например, пользователя) удобнее выносить в
отдельный `interface`, а не описывать инлайном. Тогда её можно переиспользовать
в других местах — например, в возвращаемом типе селектора:

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

Тип и `interface`/`type`, который его описывает, принято называть в `PascalCase`
(`AuthState`, `FilterState`) — так их сразу видно среди обычных переменных и
функций.

---

## `RootState` и `AppDispatch`

Это два ключевых типа для любого Redux + TypeScript проекта. `RootState` — тип
всего состояния приложения (нужен селекторам и `useSelector`). `AppDispatch` —
тип функции `dispatch` конкретно этого store (нужен для типизированного
`useDispatch`, особенно когда в middleware добавлены thunk-и или RTK Query).

Оба типа не пишутся вручную — они выводятся автоматически из самого store через
`ReturnType` и `typeof`:

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
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(contactsApi.middleware),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

export default store;
```

Почему именно так:

- `store.getState` — это функция, которая возвращает объект состояния.
  `ReturnType<typeof store.getState>` читается как «тип того, что возвращает эта
  функция» — то есть точная форма всего state, автоматически собранная из всех
  редьюсеров сразу. Если завтра добавится новый slice, `RootState` обновится
  сам, без ручного дописывания.
- `typeof store.dispatch` берёт тип самой функции `dispatch` этого конкретного
  store — со всеми возможностями, которые добавляют подключённые middleware (в
  том числе RTK Query).
- Чтобы `typeof store...` вообще сработал, store должен быть отдельной
  переменной, а не результатом, который сразу улетает в `export default` —
  `typeof` ссылается на переменную по имени, ссылаться не на что, если имени
  нет. Дальше `RootState` импортируется туда, где нужно знать форму state — в
  первую очередь в селекторы:

```ts
// selectors.ts
import type { RootState } from './store';

export const selectUser = (state: RootState) => state.auth.user;
export const selectIsLoggedIn = (state: RootState) => state.auth.isLoggedIn;
export const selectFilter = (state: RootState) => state.filter.value;
```

Использован `import type`, а не обычный `import` — потому что из `store.ts`
нужен только тип, а не что-то, что реально выполняется в рантайме. `import type`
гарантированно удаляется на этапе сборки: в финальном JS-файле такого импорта
вообще не будет. Это полезно в двух смыслах: во-первых, чётко видно по коду, что
зависимость чисто «типовая»; во-вторых, это снимает любые опасения по поводу
циклических импортов между `store.ts` и файлами, которые импортируют из него
`RootState`, — стирающийся при сборке импорт физически не может зациклиться в
рантайме.

С типизированными селекторами `state` внутри селектора получает автодополнение и
защиту от опечаток в названиях полей, а всё, что берёт значение через
`useSelector(selectUser)`, автоматически получает правильный тип без ручных
аннотаций.

---

## Типизированные хуки `useAppDispatch` / `useAppSelector`

`useDispatch` и `useSelector` из `react-redux` по умолчанию ничего не знают про
конкретный store конкретного проекта — `dispatch` без дженерика не в курсе про
thunk-и и middleware, а `state` внутри `useSelector` без дженерика — `any`.

Стандартный подход — один раз завести собственные типизированные версии этих
хуков и дальше по всему проекту использовать только их:

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
const user = useAppSelector(selectUser); // сразу правильный тип, без generics вручную
```

Плюс такого подхода: типизация делается один раз в `hooks.ts`, а не повторяется
дженериками в каждом компоненте, который обращается к store.

---

# RTK Query: типизация API

## `builder.query<ResultType, ArgType>` и `builder.mutation<ResultType, ArgType>`

Каждый эндпоинт в `createApi` — это дженерик с двумя параметрами: что запрос
**возвращает** и что он **принимает как аргумент**. Указывать их нужно всегда
явно — если пропустить, TypeScript подставит `unknown`, и внутри `query` у
аргумента не будет вообще никаких известных полей, а результат запроса нельзя
будет использовать напрямую без дополнительных проверок.

builder.mutation<A, B> — это как функция с двумя "слотами" для типов, и порядок
фиксированный:

Первый параметр (A) — что вернёт сервер после выполнения запроса (то, что
окажется в data после успешного ответа). Второй параметр (B) — что ты передаёшь
в саму мутацию, когда её вызываешь в компоненте.

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
  endpoints: (builder) => ({
    getContacts: builder.query<ContactsList, void>({
      query: () => `/contacts`,
      providesTags: ['Contact'],
    }),

    addContact: builder.mutation<Contact, NewContact>({
      query: (newContact) => ({
        url: `/contacts`,
        method: 'POST',
        body: newContact,
      }),
      invalidatesTags: ['Contact'],
    }),

    deleteContact: builder.mutation<{ id: string }, string>({
      query: (id) => ({
        url: `/contacts/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: ['Contact'],
    }),

    toggleFavorite: builder.mutation<Contact, Contact>({
      query: (contact) => {
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

Несколько моментов, почему это выглядит именно так:

- Когда запросу не нужен аргумент (`getContacts`), вторым параметром ставится
  `void`, а не `{}`. `{}` в TypeScript означает «почти любое значение, кроме
  `null`/`undefined`» — это не «пустой объект» и точно не «аргументов нет».
  `void` — это явно «аргумент не нужен», и вызвать `useGetContactsQuery()` с
  каким-то мусором внутри уже не получится.
- Тип аргумента мутации — это то, что реально приходит в `query`. Для удаления
  приходит просто `id` (строка), поэтому вторым параметром стоит `string`, а не
  весь объект контакта.
- Для `addContact` аргумент — не `Contact`, а
  `Omit<Contact, 'id' | 'isFavorite'>`. `Omit<T, Keys>` — Utility Type, который
  берёт тип `T` и убирает из него перечисленные поля. `id` и `isFavorite`
  вычисляются на клиенте (`crypto.randomUUID()`, `false` по умолчанию) и не
  приходят из формы — значит, требовать их в аргументе было бы неверно. Всё, что
  возвращают сгенерированные хуки, уже типизировано на основе этих дженериков и
  не требует ручной аннотации:

```ts
const { data, isLoading, isError } = useGetContactsQuery();
// data: ContactsList | undefined
```

---

## Тегирование по id, а не только по типу

Это уже не про TypeScript, а про сам RTK Query — но раз речь про то, «как на
проде», стоит знать. Один общий тег на весь список (`providesTags: ['Contact']`,
`invalidatesTags: ['Contact']`) означает, что после **любой** мутации весь
список запрашивается заново целиком.

Более точный вариант — тегировать каждый элемент отдельно по `id`, тогда после
мутации перезапрашивается только изменённая запись, а не весь список:

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

Для небольшого списка разницы не видно, но при росте количества данных это
экономит лишние запросы к серверу.

---

# Редко используется / устаревшее

## Работа с HTML-элементами через `as`

При работе с DOM в TypeScript часто нужно указать конкретный тип элемента —
`document.getElementById` возвращает `HTMLElement | null`, который не знает про
`.value` или другие специфичные свойства инпута.

```ts
const input = document.getElementById('inputEmail') as HTMLInputElement;
```

Есть и другой синтаксис приведения типа — через угловые скобки:

```ts
const input = <HTMLInputElement>document.getElementById('inputEmail');
```

⚠️ Этот вариант **не подходит для `.tsx` файлов** — TypeScript путает его с
JSX-разметкой. В React-проектах, где почти всё лежит в `.tsx`, всегда
используй `as`; в React-приложениях этот приём в принципе нужен редко, так как
работа с DOM чаще идёт через типизированный `useRef<HTMLInputElement>(null)`,
а не через `getElementById`.

---

## Настройка типизированной темы для Emotion / styled-components

Инструкция для проектов на React + TypeScript + Emotion (`@emotion/styled`,
`@emotion/react`). Актуально и для `styled-components` — отличия отмечены
отдельно.

Проблема, которую решаем: по умолчанию параметр `theme` в
`styled.div\`...\`` типизирован пустым служебным интерфейсом библиотеки
(`Theme`у emotion /`DefaultTheme` у styled-components). Поэтому TS не видит
твои кастомные поля (`colors`, `spacing` и т.д.), даже если объект темы у тебя
правильный и всё работает в рантайме.

Решение — **declaration merging** (слияние объявлений): мы "дополняем"
служебный интерфейс библиотеки своими полями.

### Шаг 1. Выносим тип темы в отдельный файл и типизируем сам объект темы

Заводим отдельный файл с интерфейсом (это единый источник правды для формы
темы — используется и в объекте темы, и в module augmentation на шаге 2):

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

Применяем этот тип к самому объекту темы:

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

> Зачем отдельный файл `theme.types.ts`, а не интерфейс прямо в `theme.ts`?
> Чтобы на шаге 2 импортировать один и тот же тип, а не дублировать его
> структуру вручную ещё раз в `emotion.d.ts`.

### Шаг 2. Расширяем служебный тип библиотеки (module augmentation)

Создаём файл `emotion.d.ts` (можно и `.ts` — но `.d.ts` — общепринятое
соглашение для файлов, которые только объявляют типы и не содержат
runtime-кода).

```ts
// src/emotion.d.ts
import '@emotion/react';
import type { Theme as AppTheme } from './theme/theme.types';

declare module '@emotion/react' {
  export interface Theme extends AppTheme {}
}
```

**Для `styled-components` вместо этого:**

```ts
// src/styled.d.ts
import 'styled-components';
import { Theme as AppTheme } from './theme/theme.types';

declare module 'styled-components' {
  export interface DefaultTheme extends AppTheme {}
}
```

Важно проверить:

- Файл лежит внутри `src` (или другой директории, указанной в `include`
  секции `tsconfig.json`).
- В начале файла есть `import '@emotion/react'` (или `'styled-components'`) —
  без него файл не считается модулем, и augmentation не сработает.
- Используется именно `interface`, а не `type` — только интерфейсы
  поддерживают declaration merging.

### Шаг 3. Переименовываем файлы стилей `.js`/`.jsx` → `.ts`

Файлы вида `Footer.styled.js` → `Footer.styled.ts`. Если файл со
styled-компонентами содержит **только** вызовы `styled.xxx` без JSX-разметки
внутри самого файла — достаточно `.ts`. Расширение `.tsx` нужно только там,
где в файле есть непосредственно JSX-синтаксис.

```ts
// src/components/Footer/Footer.styled.ts
import styled from '@emotion/styled';

export const FooterWrapper = styled.footer`
  padding: 0 ${({ theme }) => theme.spacing.lg};
  border-radius: ${({ theme }) => theme.radii.medium};
`;
```

Теперь `theme` в колбэке автоматически имеет тип из шага 2 — автодополнение и
проверка типов работают без ошибки
`Property 'spacing' does not exist on type 'Theme'`.

### Полиморфный `as` prop у styled-компонентов

Частая ошибка при попытке отрендерить styled-компонент как другой элемент —
например, `<Link>` из `react-router-dom` вместо обычного `<a>`:

```tsx
const Logo = styled.a`
  /* ... */
`;

<Logo as={Link} to="/" aria-label="Logo of the project">
```

TypeScript ругается примерно так:

> Свойство "to" не существует в типе "... & AnchorHTMLAttributes<...>"

**Почему так происходит.** Тип пропсов styled-компонента фиксируется в момент
его создания — `styled.a` знает только про
`AnchorHTMLAttributes<HTMLAnchorElement>` (и `theme`). Проп `as` меняет элемент
в рантайме, но статическая типизация об этом не в курсе и не подтягивает
автоматически пропсы компонента, переданного в `as` (в данном случае —
`LinkProps` из `react-router-dom`, откуда и берётся `to`).

**Решение — стилизовать сам компонент, а не элемент:**

```tsx
import { Link } from 'react-router-dom';
import styled from '@emotion/styled';

const Logo = styled(Link)`
  /* те же стили */
`;
```

```tsx
<Logo to="/" aria-label="Logo of the project">
```

Когда `styled()` оборачивает не строку тега (`'a'`), а сам React-компонент
(`Link`), он забирает тип пропсов именно у него — `Logo` автоматически знает
про `to`, `replace`, `state` и все остальные пропсы `Link`, без `as` и без
ручных дженериков.

```text
src/
├─ theme/
│  ├─ theme.types.ts   ← интерфейс Theme (единый источник правды)
│  └─ theme.ts         ← сам объект темы, типизирован через Theme
├─ emotion.d.ts         ← declaration merging (module augmentation)
└─ components/
   └─ Footer/
      └─ Footer.styled.ts
```

---
