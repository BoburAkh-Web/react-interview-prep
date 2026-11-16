# Rezyume texnologiyalari, 1-qism: JavaScript va TypeScript

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

> Bu qism `resume-technologies-questions.md` ning **batafsil** versiyasi. Har bir savol shu tuzilmada:
>
> - **Oddiy qilib**: tushuncha noldan, o'xshatish bilan
> - **Kod**: izohli misol
> - **Intervyuda shunday ayting**: 2–4 jumlali tayyor javob
> - **Qo'shimcha savol**: intervyuer odatda keyin nima so'raydi
>
> Avval **lug'at**ni o'qing: tushunarsiz kalit so'zlar shu yerda.

---

## Lug'at: JavaScript kalit so'zlari

| So'z | Ma'nosi |
|---|---|
| **Scope** | O'zgaruvchi "ko'rinadigan" hudud: funksiya ichi, `{}` blok ichi yoki global |
| **Hoisting** | E'lonlar kod bajarilishidan oldin "tepaga ko'tarilgandek" ishlashi |
| **TDZ** (temporal dead zone) | `let`/`const` e'lon qilingunga qadar ularga murojaat qilib bo'lmaydigan oraliq |
| **Closure** | Funksiya o'zi yaratilgan joydagi o'zgaruvchilarni "eslab qolishi" |
| **Primitive** | Oddiy qiymat: `string`, `number`, `boolean`, `null`, `undefined`, `bigint`, `symbol` |
| **Reference type** | Obyekt, massiv, funksiya: o'zgaruvchida qiymat emas, xotiradagi **manzil** saqlanadi |
| **Shallow / deep copy** | Yuzaki nusxa (faqat birinchi qavat) / chuqur nusxa (barcha qavatlar) |
| **Mutate** | Mavjud obyekt yoki massivni joyida o'zgartirish |
| **Immutability** | O'zgartirmasdan, yangi nusxa yaratish tamoyili |
| **Side effect** | Funksiya tashqi dunyoni o'zgartirishi (API, DOM, localStorage, tashqi o'zgaruvchi) |
| **Pure function** | Bir xil kirishga doim bir xil natija, side effect yo'q |
| **Promise** | Kelajakda tayyor bo'ladigan natija uchun "va'da" obyekti |
| **Call stack** | Hozir bajarilayotgan funksiyalar navbati (stek) |
| **Event loop** | Asinxron vazifalarni stek bo'shaganda navbat bilan bajaruvchi mexanizm |
| **Microtask / macrotask** | Asinxron navbatlar: Promise'lar (micro) va `setTimeout`, event'lar (macro) |
| **Prototype** | Obyekt meros oladigan "ota" obyekt |
| **Bubbling / capturing** | DOM event'ining yuqoriga ko'tarilishi / pastga tushishi |
| **Event delegation** | Ko'p element o'rniga bitta ota-elementga listener qo'yish |
| **ESM / CommonJS** | `import/export` va `require/module.exports` modul tizimlari |
| **Tree shaking** | Ishlatilmagan kodni bundle'dan olib tashlash |

---

## A. JavaScript

### A1. [Junior] `var`, `let` va `const` farqi nima? Scope nima?

**Oddiy qilib:** Scope bu o'zgaruvchi "yashaydigan xona". `var` butun **funksiya** xonasida yashaydi, `let` va `const` esa faqat o'zi turgan **`{}` blok** ichida. `const` ga qayta qiymat berib bo'lmaydi, lekin u obyekt bo'lsa, ichini o'zgartirish mumkin (chunki o'zgaruvchi obyektning **manzili**ni saqlaydi, manzil o'zgarmaydi).

```js
function test() {
  if (true) {
    var a = 1;
    let b = 2;
  }
  console.log(a); // 1  (var blokdan "chiqib ketadi")
  console.log(b); // ReferenceError (let faqat blok ichida)
}

const user = { name: "Ali" };
user.name = "Vali";   // mumkin: obyektning ichi o'zgardi
user = {};            // TypeError: const'ga qayta qiymat berib bo'lmaydi
```

Klassik tuzoq, sikl va `setTimeout`:

```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3, 3, 3
for (let j = 0; j < 3; j++) setTimeout(() => console.log(j)); // 0, 1, 2
```

`var` bitta umumiy `i` yaratadi, `let` esa har aylanishda **yangi** `j`.

**Intervyuda shunday ayting:** "`var` funksiya doirasida, `let` va `const` blok doirasida ishlaydi. `var` ni qayta e'lon qilish mumkin va u hoisting'da `undefined` bo'ladi. Men default `const`, qiymat o'zgarishi kerak bo'lsa `let` ishlataman, `var` ishlatmayman."

**Qo'shimcha savol:** "`const` massivga `push` qilsa bo'ladimi?" → Ha, massivning manzili o'zgarmaydi, faqat ichi.

---

### A2. [Junior] Hoisting nima?

**Oddiy qilib:** JavaScript kodni bajarishdan oldin uni bir marta "ko'rib chiqadi" va barcha e'lonlarni xotiraga yozib qo'yadi. Shuning uchun ba'zi narsalarni e'lon qilinmasdan **oldin** ishlatish mumkin bo'ladi, go'yo ular tepaga ko'tarilgandek.

```js
sayHi();                 // ishlaydi: function declaration to'liq ko'tariladi
function sayHi() { console.log("Hi"); }

console.log(x);          // undefined: var ko'tariladi, lekin qiymatsiz
var x = 5;

console.log(y);          // ReferenceError: let TDZ'da
let y = 5;

greet();                 // TypeError: greet hali undefined (var + funksiya ifodasi)
var greet = () => console.log("Hello");
```

**Intervyuda shunday ayting:** "Hoisting'da e'lonlar bajarilishdan oldin xotiraga olinadi. Function declaration to'liq ko'tariladi, `var` `undefined` bilan, `let` va `const` esa ko'tariladi, lekin e'longacha temporal dead zone'da bo'ladi va murojaat qilinsa xato beradi."

---

### A3. [Junior] Closure nima? Amalda qayerda ishlatiladi?

**Oddiy qilib:** Funksiya tug'ilgan joyining "xotirasini" o'zi bilan olib yuradi. Tashqi funksiya ishini tugatsa ham, ichki funksiya uning o'zgaruvchilariga murojaat qila oladi. Xuddi uydan chiqib ketgan odam cho'ntagida uy kalitini olib yurgandek.

```js
function createCounter() {
  let count = 0;                 // tashqaridan ko'rinmaydi (private)
  return function () {
    count++;
    return count;
  };
}

const counter = createCounter();
counter(); // 1
counter(); // 2  (count "eslab qolingan")
```

Amalda: **debounce** (timer ID'sini eslab qoladi), **private o'zgaruvchilar**, event handler'lar, React'dagi barcha hook'lar.

React'dagi **stale closure** muammosi:

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  const id = setInterval(() => console.log(count), 1000); // doim 0
  return () => clearInterval(id);
}, []);  // effect faqat birinchi renderdagi count'ni "eslab qolgan"
```

**Intervyuda shunday ayting:** "Closure bu funksiyaning o'zi yaratilgan muhitdagi o'zgaruvchilarga, o'sha muhit tugagandan keyin ham murojaat qila olishi. Men uni debounce yozishda ishlatganman. React'da event handler va effect'lar render'dagi qiymatlarni closure orqali ushlaydi, shuning uchun dependency array to'liq bo'lmasa eski qiymat bilan ishlab qoladi."

---

### A4. [Strong junior] `this` qanday aniqlanadi? Arrow function'da qanday?

**Oddiy qilib:** `this` bu "funksiyani kim chaqirdi" degan savolning javobi. U funksiya **qayerda yozilganiga** emas, **qanday chaqirilganiga** bog'liq. Arrow function esa bundan mustasno: uning o'z `this` i yo'q, u tashqaridagi `this` ni oladi.

```js
const user = {
  name: "Ali",
  regular() { console.log(this.name); },   // this = user (user.regular() deb chaqirilsa)
  arrow: () => console.log(this?.name),    // this = tashqi muhit (user emas!)
};

user.regular();             // "Ali"
const fn = user.regular;
fn();                       // undefined: endi "chaqiruvchi" yo'q

const bound = user.regular.bind(user);
bound();                    // "Ali": bind this'ni qotirib qo'yadi
```

`call` va `apply` funksiyani darhol, berilgan `this` bilan chaqiradi (`fn.call(obj, a, b)`, `fn.apply(obj, [a, b])`). `bind` esa yangi funksiya qaytaradi.

**Intervyuda shunday ayting:** "`this` chaqiruv usuliga bog'liq: obyekt metodi sifatida chaqirilsa obyekt, oddiy chaqiruvda strict mode'da `undefined`. Arrow function'ning o'z `this` i yo'q, tashqi muhitnikini oladi. Function component'larda `this` ishlatilmaydi, class component'larda esa handler'lar shu sababli arrow function bilan yozilardi."

---

### A5. [Junior] Primitive va reference turlar farqi nima? Obyektlarni `===` bilan solishtirish qanday ishlaydi?

**Oddiy qilib:** Primitive qiymat o'zgaruvchiga **nusxa** bo'lib yoziladi. Obyekt esa xotirada bir joyda turadi, o'zgaruvchida faqat uning **manzili** (havolasi) bo'ladi. Obyektni boshqa o'zgaruvchiga bersangiz, manzil nusxalanadi, ya'ni ikkala o'zgaruvchi **bitta** obyektga qaraydi.

```js
let a = 5;
let b = a;
b = 10;
console.log(a); // 5 (alohida nusxa)

const obj1 = { x: 1 };
const obj2 = obj1;
obj2.x = 99;
console.log(obj1.x); // 99 (bitta obyekt!)

{ x: 1 } === { x: 1 };  // false: ikki xil obyekt, manzillar har xil
obj1 === obj2;          // true: bir xil manzil
```

React shu sababli state'ni **havola bo'yicha** solishtiradi: obyektni joyida o'zgartirsangiz, manzil o'sha-o'sha qoladi va React o'zgarishni sezmaydi.

**Qo'shimcha savol (tuzoq):** `typeof null` nima qaytaradi? → `"object"`. Bu JavaScript'ning tarixiy xatosi, `null` aslida primitive.

---

### A6. [Junior] Shallow copy va deep copy farqi nima?

**Oddiy qilib:** Shallow (yuzaki) nusxa faqat **birinchi qavatni** nusxalaydi, ichidagi obyektlar esa hali ham **umumiy** qoladi. Deep (chuqur) nusxa barcha qavatlarni alohida nusxalaydi.

```js
const user = { name: "Ali", address: { city: "Toshkent" } };

const shallow = { ...user };
shallow.address.city = "Samarqand";
console.log(user.address.city);  // "Samarqand": ichki obyekt umumiy!

const deep = structuredClone(user);
deep.address.city = "Buxoro";
console.log(user.address.city);  // "Samarqand": endi ta'sir qilmadi
```

React'da ichki obyektni yangilashda har bir qavatni spread qilish kerak:

```js
setUser((u) => ({ ...u, address: { ...u.address, city: "Buxoro" } }));
```

**Intervyuda shunday ayting:** "Spread va `Object.assign` shallow copy qiladi, ichki obyektlar havola bo'yicha qoladi. Chuqur nusxa uchun `structuredClone` ishlataman. React state'ida ichki maydonni yangilashda har bir qavatni spread qilaman yoki Immer ishlataman."

**Qo'shimcha savol:** "`JSON.parse(JSON.stringify(obj))` bilan deep copy qilsa bo'ladimi?" → Ishlaydi, lekin `Date`, `undefined`, funksiya, `Map`/`Set` yo'qoladi yoki buziladi. `structuredClone` yaxshiroq.

---

### A7. [Junior] Destructuring, rest va spread nima?

**Oddiy qilib:** Destructuring obyekt yoki massivdan qiymatlarni "qutidan olib chiqib" alohida o'zgaruvchilarga solish. Spread (`...`) narsalarni **yoyadi**, rest (`...`) esa qolganlarini **yig'adi**. Belgi bir xil, joyiga qarab ma'nosi o'zgaradi.

```js
// Obyekt destructuring: default qiymat va qayta nomlash
const { name, age = 18, role: userRole } = user;

// Massiv destructuring: useState aynan shunday
const [count, setCount] = useState(0);

// Rest: qolganlarini yig'ish
const { password, ...safeUser } = user;   // password'siz obyekt
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }

// Spread: yoyish
const merged = { ...defaults, ...options };    // o'ngdagi ustun keladi
const all = [...arr1, ...arr2];
```

**Intervyuda shunday ayting:** "Destructuring obyekt va massivdan qiymatlarni qisqa olish uchun, React'da props va `useState` da doim ishlatiladi. Spread obyekt yoki massivni yoyib, nusxa yoki birlashtirish uchun; rest esa qolgan elementlarni yig'adi, masalan obyektdan bitta maydonni chiqarib tashlash uchun."

---

### A8. [Junior] `||`, `??` va `?.` farqi nima?

**Oddiy qilib:**

- `||` "chap tomon **yolg'on (falsy)** bo'lsa, o'ngini ol". Falsy: `false`, `0`, `""`, `null`, `undefined`, `NaN`
- `??` "chap tomon faqat **`null` yoki `undefined`** bo'lsa, o'ngini ol"
- `?.` "bu narsa bo'lmasa, xato berma, `undefined` qaytar"

```js
const count = 0;
const a = count || 10;   // 10: 0 falsy bo'lgani uchun yo'qoldi (XATO bo'lishi mumkin)
const b = count ?? 10;   // 0: to'g'ri

user?.address?.city;      // address yo'q bo'lsa: undefined (xato emas)
user.address.city;        // address yo'q bo'lsa: TypeError
callback?.();             // funksiya bo'lsa chaqir
```

**Intervyuda shunday ayting:** "`??` faqat `null` va `undefined` ni tekshiradi, `||` esa barcha falsy qiymatlarni. Son yoki bo'sh string to'g'ri qiymat bo'lishi mumkin bo'lgan joyda `??` ishlataman. API'dan kelgan ichma-ich ma'lumot uchun optional chaining."

---

### A9. [Junior] Massiv metodlari: qaysilari massivni o'zgartiradi?

**Oddiy qilib:** Ba'zi metodlar **yangi massiv** qaytaradi (asl massiv tegilmaydi), ba'zilari esa **asl massivni** o'zgartiradi. React state'ida faqat birinchi turdagilar xavfsiz.

| O'zgartirmaydi (React'da xavfsiz) | O'zgartiradi (state'da ehtiyot!) |
|---|---|
| `map`, `filter`, `reduce`, `find`, `some`, `every`, `slice`, `concat`, `toSorted`, `toReversed` | `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse` |

```js
const prices = [10, 25, 5];

prices.map((p) => p * 2);                 // [20, 50, 10]   har birini o'zgartirish
prices.filter((p) => p > 8);              // [10, 25]       shartga moslarini olish
prices.reduce((sum, p) => sum + p, 0);    // 40             bitta qiymatga yig'ish
prices.find((p) => p > 8);                // 10             birinchi mosini topish
prices.some((p) => p > 20);               // true           kamida bittasi mosmi
prices.every((p) => p > 0);               // true           hammasi mosmi

prices.sort();                  // asl massiv o'zgardi!
const sorted = prices.toSorted((a, b) => a - b);   // yangi massiv
```

**Qo'shimcha savol (tuzoq):** `[10, 1, 5].sort()` nima qaytaradi? → `[1, 10, 5]`: default `sort` raqamlarni **string** sifatida solishtiradi. Son uchun `(a, b) => a - b` kerak.

---

### A10. [Junior] `Set` va `Map` nima uchun kerak?

**Oddiy qilib:** `Set` takrorlanmaydigan qiymatlar to'plami ("faqat noyoblar ro'yxati"). `Map` kalit-qiymat juftligi, lekin oddiy obyektdan farqli, kalit **istalgan tur** bo'la oladi.

```js
// Set: takrorlarni olib tashlash va tez tekshirish
const unique = [...new Set(["uz", "ru", "uz"])];   // ["uz", "ru"]
const selected = new Set([1, 2, 3]);
selected.has(2);   // true, juda tez

// Map: kalit sifatida obyekt ham bo'ladi
const cache = new Map();
cache.set(userObj, "data");
cache.get(userObj);
cache.size;
```

Tezlik: katta massivda `arr.includes(x)` har safar boshidan oxirigacha qaraydi (**O(n)**), `set.has(x)` esa deyarli bir zumda (**O(1)**).

**Intervyuda shunday ayting:** "Set takrorlanmas qiymatlar va tez `has` tekshiruvi uchun, masalan takrorlarni olib tashlash yoki tanlangan ID'lar to'plami. Map kalit istalgan tur bo'lishi va tartib saqlanishi kerak bo'lganda."

---

### A11. [Junior] Promise nima? async/await qanday ishlaydi?

**Oddiy qilib:** Promise bu restorandagi **buyurtma cheki**: taom hali tayyor emas, lekin "tayyor bo'lganda olasiz" degan va'da. Uchta holati bor: **pending** (kutilmoqda), **fulfilled** (tayyor), **rejected** (xato). `async/await` esa Promise'larni oddiy, ketma-ket kod kabi yozish usuli.

```js
// .then zanjiri
fetch("/api/menu")
  .then((res) => res.json())
  .then((data) => console.log(data))
  .catch((err) => console.error(err));

// async/await: xuddi shu narsa
async function loadMenu() {
  try {
    const res = await fetch("/api/menu");
    if (!res.ok) throw new Error(`HTTP ${res.status}`);   // fetch 404'da reject qilmaydi!
    const data = await res.json();
    return data;
  } catch (err) {
    console.error(err);
    throw err;            // chaqiruvchi ham bilishi uchun
  } finally {
    setIsLoading(false);  // muvaffaqiyat ham, xato ham bo'lsa ishlaydi
  }
}
```

`async` funksiya **doim Promise** qaytaradi. `await` faqat `async` funksiya ichida (yoki modulning yuqori darajasida) ishlaydi.

**Intervyuda shunday ayting:** "Promise asinxron natijani ifodalaydi: pending, fulfilled yoki rejected. async/await uning ustidagi sintaksis, xatolar `try/catch` bilan ushlanadi. `fetch` HTTP xatolarida reject qilmaydi, shuning uchun `res.ok` ni doim tekshiraman."

---

### A12. [Strong junior] `Promise.all`, `allSettled`, `race` va `any` farqi?

**Oddiy qilib:** Bir nechta so'rovni **bir vaqtda** yuborib, natijalarni turlicha kutish usullari.

| Metod | Qachon tugaydi | Xato bo'lsa |
|---|---|---|
| `Promise.all` | Hammasi muvaffaqiyatli bo'lganda | **Birinchi** xatoda butunlay reject |
| `Promise.allSettled` | Hammasi tugaganda (xato bo'lsa ham) | Har birining holatini qaytaradi |
| `Promise.race` | **Birinchi** tugaganida (muvaffaqiyat yoki xato) | Birinchi xato bo'lsa reject |
| `Promise.any` | **Birinchi muvaffaqiyatli** tugaganida | Hammasi xato bo'lsagina reject |

```js
// Ketma-ket (sekin): 1 sek + 1 sek = 2 sek
const user = await getUser();
const orders = await getOrders();

// Parallel (tez): ~1 sek
const [user, orders] = await Promise.all([getUser(), getOrders()]);

// Biri xato bo'lsa ham qolganini ko'rsatish
const results = await Promise.allSettled([getStats(), getChart()]);
results.forEach((r) => r.status === "fulfilled" ? show(r.value) : log(r.reason));
```

**Intervyuda shunday ayting:** "Bir-biriga bog'liq bo'lmagan so'rovlarni ketma-ket `await` qilmay, `Promise.all` bilan parallel yuboraman, bu tezroq. Ulardan biri xato bersa ham qolganini ko'rsatish kerak bo'lsa, `allSettled`."

---

### A13. [Strong junior] Event loop qanday ishlaydi?

**Oddiy qilib:** JavaScript **bir oqimli**: bir vaqtda faqat bitta ish qiladi. Uzoq ishlar (timer, tarmoq so'rovi) brauzerga topshiriladi, tayyor bo'lgach ularning callback'lari **navbatga** turadi. Event loop esa navbatchi: **stek bo'shaganini** kutadi va navbatdagi vazifani stekka qo'yadi.

```
        ┌──────────────┐
        │  Call stack  │  ← hozir bajarilayotgan kod
        └──────┬───────┘
               │ bo'shasa
   ┌───────────▼────────────┐
   │ 1. Microtask navbati   │  ← Promise.then, await, queueMicrotask  (HAMMASI bajariladi)
   └───────────┬────────────┘
               │ keyin
   ┌───────────▼────────────┐
   │ 2. Render (kerak bo'lsa)│
   └───────────┬────────────┘
               │
   ┌───────────▼────────────┐
   │ 3. Macrotask navbati   │  ← setTimeout, setInterval, click event (BITTA olinadi)
   └────────────────────────┘
```

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Natija: 1, 4, 3, 2
```

Sabab: `1` va `4` sinxron. Keyin stek bo'shaydi: avval **microtask** (`3`), keyin **macrotask** (`2`). `setTimeout(fn, 0)` "darhol" degani emas, "eng tez navbatda" degani.

**Intervyuda shunday ayting:** "JavaScript bir oqimli. Sinxron kod call stack'da bajariladi, asinxron callback'lar navbatga tushadi. Stek bo'shaganda event loop avval barcha microtask'larni (Promise), keyin bitta macrotask'ni (setTimeout, event) bajaradi. Shuning uchun Promise `setTimeout(0)` dan oldin ishlaydi."

**Qo'shimcha savol:** "Uzoq sinxron sikl sahifani nega qotiradi?" → Stek band, event loop render va click'larni bajara olmaydi.

---

### A14. [Strong junior] Prototype va prototypal inheritance nima?

**Oddiy qilib:** Har bir obyektning yashirin "ota"si bor: **prototip**. Obyektda xossa topilmasa, JavaScript otasidan, keyin bobosidan qidiradi (**prototip zanjiri**). Massivda `map` metodi yozilmagan, lekin u `Array.prototype` da turadi, shuning uchun hamma massivlar uni ishlata oladi.

```js
const arr = [1, 2, 3];
arr.map;                                          // arr'ning o'zida yo'q
Object.getPrototypeOf(arr) === Array.prototype;   // true: shu yerdan olinadi

const animal = { eat() { return "eating"; } };
const dog = Object.create(animal);   // dog'ning prototipi animal
dog.eat();                           // "eating": zanjir bo'ylab topildi
```

**Intervyuda shunday ayting:** "JavaScript'da meros prototiplar orqali: xossa obyektda topilmasa, prototip zanjiri bo'ylab qidiriladi. ES6 `class` va `extends` aynan shu mexanizm ustidagi qulay sintaksis."

---

### A15. [Junior] ES6 `class` qanday yoziladi?

```js
class User {
  #password;                         // haqiqiy private maydon (#)
  constructor(name, password) {
    this.name = name;
    this.#password = password;
  }
  greet() { return `Salom, ${this.name}`; }     // prototipga qo'shiladi
  static create(name) { return new User(name, ""); }   // class'ning o'zida
}

class Admin extends User {
  constructor(name) {
    super(name, "");                 // ota constructor'ini chaqirish majburiy
    this.role = "admin";
  }
}
```

**Intervyuda shunday ayting:** "Class prototiplar ustidagi sintaksis: `constructor`, metodlar, `static` metodlar, `extends` va `super`. `#` bilan haqiqiy private maydon yasaladi. React'da hozir function component va hook'lar ishlatiladi, class'larni eski kodda va Error Boundary'da uchrataman."

---

### A16. [Junior] ES Modules va CommonJS farqi? Default va named export?

**Oddiy qilib:** Modul tizimi kodni fayllarga bo'lib, ular orasida narsa almashish usuli. Zamonaviy usul **ESM** (`import/export`), eski Node.js usuli **CommonJS** (`require`).

```js
// ESM
export const API_URL = "...";            // named export (bir faylda ko'p)
export default function Button() {}     // default export (bittagina)

import Button from "./Button";          // default: istalgan nom bilan
import { API_URL } from "./config";     // named: aynan shu nom bilan
import { API_URL as URL } from "./config";

// CommonJS
const express = require("express");
module.exports = { start };
```

ESM **statik**: import'lar faylning boshida va aniq, shuning uchun build vositasi ishlatilmagan kodni topib olib tashlay oladi (**tree shaking**).

**Intervyuda shunday ayting:** "Frontend'da ESM ishlataman. Named export'ni afzal ko'raman, chunki nom aniq va avtomatik import va refactoring qulayroq. React.lazy uchun esa default export kerak."

---

### A17. [Strong junior] Event bubbling, capturing va delegation nima? `stopPropagation` va `preventDefault` farqi?

**Oddiy qilib:** Tugmani bossangiz, event avval `window` dan pastga, tugmagacha tushadi (**capturing**), keyin tugmadan yuqoriga, `window` gacha ko'tariladi (**bubbling**). Odatda listener'lar bubbling bosqichida ishlaydi.

```
window → document → body → ul → li → button   (capturing, pastga)
window ← document ← body ← ul ← li ← button   (bubbling, yuqoriga)
```

**Event delegation:** 100 ta `li` ga alohida listener o'rniga, bitta `ul` ga qo'yish. Bubbling tufayli `li` dagi click `ul` ga yetib keladi.

```js
list.addEventListener("click", (e) => {
  const item = e.target.closest("li");   // aynan qaysi li bosildi
  if (!item) return;
  select(item.dataset.id);
});
```

- **`e.stopPropagation()`**: event'ni **yuqoriga uzatmaydi** (ota-elementlar bilmaydi)
- **`e.preventDefault()`**: brauzerning **standart harakatini** bekor qiladi (forma yuborish, havolaga o'tish)

Ular **mustaqil**: biri ikkinchisini o'z ichiga olmaydi.

**Intervyuda shunday ayting:** "Event'lar pastga tushadi (capturing) va yuqoriga ko'tariladi (bubbling). Delegation bubbling'dan foydalanib, bitta ota-elementda ko'p child'ning event'ini ushlaydi. React'ning o'zi ham shunday ishlaydi. `stopPropagation` tarqalishni to'xtatadi, `preventDefault` esa standart harakatni."

---

### A18. [Junior] `localStorage`, `sessionStorage` va cookie farqi?

**Oddiy qilib:** Uchalasi brauzerda ma'lumot saqlash joyi, lekin muddati va vazifasi har xil.

| | localStorage | sessionStorage | Cookie |
|---|---|---|---|
| Qancha saqlanadi | O'chirilmaguncha | Tab yopilguncha | Belgilangan muddatgacha |
| Hajm | ~5 MB | ~5 MB | ~4 KB |
| Serverga yuboriladimi | Yo'q | Yo'q | **Har so'rovda avtomatik** |
| JS o'qiy oladimi | Ha | Ha | Ha, `httpOnly` bo'lmasa |

```js
localStorage.setItem("cart", JSON.stringify(cart));   // faqat string saqlanadi
const cart = JSON.parse(localStorage.getItem("cart") ?? "[]");
```

Muhim: brauzerning private rejimida localStorage xato berishi mumkin, shuning uchun `try/catch` ichida ishlatish ma'qul.

**Intervyuda shunday ayting:** "UI sozlamalari va savat kabi narsalarni localStorage'da saqlayman. Auth token uchun eng xavfsizi `httpOnly` cookie, chunki JavaScript uni o'qiy olmaydi va XSS hujumida o'g'irlanmaydi."

---

### A19. [Strong junior] `fetch` so'rovini qanday bekor qilasiz? Nega bu kerak?

**Oddiy qilib:** Foydalanuvchi tez yozsa yoki sahifadan chiqib ketsa, eski so'rovlar keraksiz bo'lib qoladi. Agar eski so'rov yangisidan **kechroq** qaytsa, ekranda noto'g'ri natija qoladi (**race condition**). `AbortController` bu so'rovlarni "o'chirish tugmasi".

```js
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/search?q=${query}`, { signal: controller.signal })
    .then((res) => res.json())
    .then(setResults)
    .catch((err) => {
      if (err.name !== "AbortError") setError(err.message);   // bekor qilish xato emas
    });

  return () => controller.abort();   // query o'zgarsa yoki component yo'qolsa
}, [query]);

// Timeout
fetch(url, { signal: AbortSignal.timeout(5000) });
```

**Intervyuda shunday ayting:** "AbortController bilan eski so'rovni effect cleanup'ida bekor qilaman, shunda race condition bo'lmaydi va keraksiz trafik ketmaydi. Real loyihalarda TanStack Query buni query key orqali o'zi qiladi."

---

### A20. [Strong junior] Pure function, side effect va immutability nima? React bilan qanday bog'liq?

**Oddiy qilib:** **Pure function** kalkulyator kabi: bir xil son kiritsangiz, doim bir xil javob beradi va atrofdagi hech narsani o'zgartirmaydi. **Side effect** funksiyaning tashqi dunyoga ta'siri. **Immutability** eski ma'lumotni buzmay, yangisini yaratish.

```js
// Pure
const add = (a, b) => a + b;

// Impure: tashqi o'zgaruvchini o'zgartiradi
let total = 0;
const addToTotal = (x) => { total += x; };

// Mutate (yomon, React state'ida)
cart.push(item);

// Immutable (yaxshi)
const newCart = [...cart, item];
```

React'da: component render'i va reducer'lar **pure** bo'lishi kerak, side effect'lar event handler va `useEffect` da, state esa **immutable** yangilanadi.

**Intervyuda shunday ayting:** "React render'ni bir necha marta chaqirishi mumkin (Strict Mode ikki marta chaqiradi), shuning uchun render pure bo'lishi kerak. State'ni joyida o'zgartirmayman, chunki React havola bo'yicha solishtiradi va o'zgarishni sezmaydi."

---

### A21. [Junior] Debounce va throttle nima? Debounce'ni yozib bering

**Oddiy qilib:**

- **Debounce** lift eshigi kabi: odamlar kirib turgan ekan, eshik yopilmaydi; oxirgi odamdan **X soniya o'tgach** yopiladi. Qidiruv input'i uchun: yozish to'xtagach bitta so'rov
- **Throttle** svetofor kabi: qancha mashina kelmasin, **har X soniyada bir marta** o'tkazadi. Scroll, resize, o'yindagi progress uchun

```js
function debounce(fn, delay) {
  let timerId;                         // closure orqali eslab qolinadi
  return function (...args) {
    clearTimeout(timerId);             // oldingi rejani bekor qilish
    timerId = setTimeout(() => fn.apply(this, args), delay);
  };
}

function throttle(fn, interval) {
  let last = 0;
  return function (...args) {
    const now = Date.now();
    if (now - last >= interval) {
      last = now;
      fn.apply(this, args);
    }
  };
}

const search = debounce((q) => fetchResults(q), 400);
input.addEventListener("input", (e) => search(e.target.value));
```

**Intervyuda shunday ayting:** "Debounce hodisalar to'xtagandan keyin bir marta bajaradi, throttle esa ma'lum interval'da ko'pi bilan bir marta. Gross'da kalkulyatordagi takroriy so'rovlarni debounce bilan kamaytirganman."

---

## Lug'at: TypeScript kalit so'zlari

| So'z | Ma'nosi |
|---|---|
| **Type annotation** | Turni qo'lda yozish: `let age: number` |
| **Type inference** | TypeScript turni o'zi taxmin qilishi: `let age = 5` → `number` |
| **Compile time / runtime** | Kod yozish va build paytida / brauzerda ishlab turgan paytda |
| **Union** | "Yoki shu, yoki bu" turi: `string \| number` |
| **Literal type** | Aniq qiymat turi: `"admin"`, `42` |
| **Narrowing** | Tekshiruv orqali union'ni aniq turga toraytirish |
| **Type guard** | Narrowing qiladigan tekshiruv yoki funksiya |
| **Discriminated union** | Umumiy belgi maydoni (`status`, `type`) bor union |
| **Generic** | Tur parametri: funksiya yoki component turli turlar bilan ishlaydi |
| **Utility type** | Mavjud turdan yangisini yasovchi tayyor tur: `Partial`, `Pick`, `Omit` |
| **Type assertion** | `as` bilan TypeScript'ga "men bilaman" deyish |
| **Strict mode** | Qattiqroq tekshiruvlar to'plami (`tsconfig` dagi `strict`) |
| **Schema validation** | Ma'lumotni runtime'da qoidaga mosligini tekshirish (Zod) |

---

## B. TypeScript

### B1. [Junior] TypeScript nima va nima uchun kerak?

**Oddiy qilib:** TypeScript bu JavaScript + **turlar**. Kod brauzerga borishidan oldin, siz yozayotgan paytda xatolarni topadi: noto'g'ri tur, mavjud bo'lmagan xossa, unutilgan argument. Build paytida TypeScript **oddiy JavaScript'ga** aylanadi, brauzer turlarni ko'rmaydi.

```ts
function getTotal(price: number, qty: number) {
  return price * qty;
}
getTotal("10", 2);   // xato: kod yozayotganda ko'rinadi, brauzerda emas

type User = { name: string; age: number };
const user: User = { name: "Ali" };   // xato: age yo'q
user.adress;                          // xato: typo, "address" emas
```

Foydalari: xatolar **erta** topiladi, avtomatik to'ldirish (autocomplete), xavfsiz refactoring (nomni o'zgartirsangiz, hamma joy ko'rsatiladi), kod o'zi hujjat vazifasini bajaradi.

**Intervyuda shunday ayting:** "TypeScript JavaScript'ga statik turlar qo'shadi. Xatolarni brauzerda emas, kod yozayotganda topadi, refactoring xavfsiz bo'ladi va jamoada kodni tushunish osonlashadi. Muhimi, turlar faqat compile time'da bor, runtime'da yo'qoladi."

---

### B2. [Junior] Asosiy turlar va type inference

```ts
let name: string = "Ali";
let age = 20;                        // inference: number (yozish shart emas)
let isAdmin: boolean = false;
let tags: string[] = ["react", "ts"];
let point: [number, number] = [41, 69];   // tuple: aniq uzunlik va tartib
let id: string | number;             // union

function greet(name: string): string {    // parametr va qaytish turi
  return `Salom, ${name}`;
}

let maybe: string | undefined;       // ixtiyoriy qiymat
function log(msg: string): void {}   // hech narsa qaytarmaydi
```

**Intervyuda shunday ayting:** "TypeScript turni ko'p joyda o'zi aniqlaydi (inference), shuning uchun men turni asosan funksiya parametrlari, props va API javoblari kabi **chegaralarda** yozaman. Qolgan joyda inference'ga ishonaman."

---

### B3. [Junior] `interface` va `type` farqi nima?

**Oddiy qilib:** Ikkalasi ham obyekt shaklini tasvirlaydi va ko'p holatda bir-birining o'rnini bosadi. Farqlar kichik:

```ts
interface User { id: number; name: string }
interface Admin extends User { permissions: string[] }   // kengaytirish

type Role = "admin" | "user";                 // union: faqat type bilan
type Point = [number, number];                // tuple: faqat type bilan
type AdminUser = User & { role: Role };       // intersection (&)

// Declaration merging: faqat interface
interface Window { myGlobal: string }        // mavjud Window turiga qo'shiladi
```

| | `interface` | `type` |
|---|---|---|
| Obyekt shakli | Ha | Ha |
| Union, tuple, primitive alias | Yo'q | Ha |
| Kengaytirish | `extends` | `&` |
| Bir xil nom bilan birlashish | Ha | Yo'q (xato) |

**Intervyuda shunday ayting:** "Obyekt va props uchun ikkalasi ham ishlaydi, jamoa kelishuviga amal qilaman. Union, tuple yoki murakkab tur kerak bo'lsa `type`, kutubxona turini kengaytirish kerak bo'lsa `interface`."

---

### B4. [Junior] Union, literal type va narrowing nima?

**Oddiy qilib:** Union "yoki u, yoki bu" degani. Literal type esa aniq qiymat: `"admin"` turi faqat `"admin"` string'ini qabul qiladi. Narrowing union'ni tekshiruv orqali **toraytirish**: tekshiruvdan keyin TypeScript qaysi tur ekanini biladi.

```ts
type Role = "admin" | "chef" | "courier" | "user";   // literal union
let role: Role = "manager";   // xato: bunday rol yo'q

function format(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase();   // bu yerda value: string
  }
  return value.toFixed(2);        // bu yerda value: number
}

function getLength(x: string[] | null) {
  if (!x) return 0;               // null holati chiqarib tashlandi
  return x.length;                // bu yerda x: string[]
}
```

Narrowing usullari: `typeof`, `instanceof`, `in` (`"email" in user`), `===` bilan solishtirish, truthiness tekshiruvi.

**Intervyuda shunday ayting:** "Rollar, statuslar va variantlar uchun literal union ishlataman, shunda noto'g'ri qiymat yozib bo'lmaydi va autocomplete ishlaydi. Union bilan ishlaganda `typeof`, `in` yoki maydon tekshiruvi orqali turni toraytiraman."

---

### B5. [Strong junior] Custom type guard nima?

**Oddiy qilib:** O'zingiz yozgan tekshiruv funksiyasi, u TypeScript'ga "bu `true` qaytarsa, qiymat shu turda" deb aytadi. Qaytish turida `x is Tur` yoziladi.

```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

function isCat(animal: Cat | Dog): animal is Cat {
  return "meow" in animal;
}

function speak(animal: Cat | Dog) {
  if (isCat(animal)) animal.meow();   // TypeScript endi Cat ekanini biladi
  else animal.bark();
}

// Amaliy: massivdan null'larni olib tashlash
const ids = [1, null, 2].filter((x): x is number => x !== null);   // number[]
```

---

### B6. [Strong junior] Discriminated union nima? React'da qayerda foydali?

**Oddiy qilib:** Union'ning har bir variantida **umumiy "belgi" maydoni** bor (`status`, `type`). TypeScript shu belgiga qarab qolgan maydonlarni aniq biladi. Natijada "imkonsiz holatlar" tur darajasida yo'q bo'ladi.

```ts
// Yomon: bir vaqtda isLoading va error ikkalasi bo'lishi mumkin
type BadState = { isLoading: boolean; error?: string; data?: User[] };

// Yaxshi: har holat aniq
type State =
  | { status: "loading" }
  | { status: "error"; error: string }
  | { status: "success"; data: User[] };

function render(state: State) {
  switch (state.status) {
    case "loading": return "Yuklanmoqda...";
    case "error":   return state.error;          // faqat shu yerda error bor
    case "success": return state.data.length;    // faqat shu yerda data bor
  }
}

// Reducer action'lari
type Action =
  | { type: "deposit"; payload: number }
  | { type: "requestLoan"; payload: { amount: number; purpose: string } }
  | { type: "closeAccount" };
```

**Intervyuda shunday ayting:** "Discriminated union'da har variantning umumiy belgi maydoni bor, TypeScript unga qarab turni toraytiradi. Men uni loading/error/success holatlari va reducer action'larini tiplashda ishlataman: noto'g'ri kombinatsiya umuman yozib bo'lmaydi."

---

### B7. [Strong junior] `any`, `unknown` va `never` farqi?

**Oddiy qilib:**

- **`any`**: "tekshirma". TypeScript'ni butunlay o'chiradi, xatolar yashirinadi
- **`unknown`**: "nima ekanini bilmayman, avval tekshir". Xavfsiz `any`
- **`never`**: "bu hech qachon bo'lmaydi". Masalan, doim xato tashlaydigan funksiya yoki barcha holatlar ko'rib chiqilgandan keyingi qism

```ts
let a: any = "hello";
a.foo.bar();               // TypeScript jim, runtime'da xato

let u: unknown = JSON.parse(text);
u.name;                    // xato: avval tekshiring
if (typeof u === "object" && u !== null && "name" in u) { /* endi xavfsiz */ }

// never: hamma holat ko'rib chiqilganini tekshirish
function assertNever(x: never): never { throw new Error(`Unexpected: ${x}`); }

switch (state.status) {
  case "loading": ...
  case "error": ...
  case "success": ...
  default: assertNever(state);   // yangi status qo'shilsa va bu yerda unutilsa, XATO beradi
}
```

**Intervyuda shunday ayting:** "`any` dan qochaman, chunki u TypeScript foydasini yo'qotadi. Turi noma'lum tashqi ma'lumot uchun `unknown` ishlatib, keyin tekshiraman yoki Zod bilan parse qilaman. `never` ni switch'da barcha holatlar ko'rib chiqilganini kafolatlash uchun ishlataman."

---

### B8. [Strong junior] Generic nima?

**Oddiy qilib:** Generic bu turlar uchun **parametr**. Funksiyaga qiymatni argument sifatida berganingiz kabi, turni ham "argument" sifatida berasiz. Natijada bitta funksiya yoki component turli turlar bilan ishlaydi va tur ma'lumoti yo'qolmaydi.

```ts
// Generic'siz: tur yo'qoladi
function firstAny(arr: any[]): any { return arr[0]; }

// Generic bilan: tur saqlanadi
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}
const n = first([1, 2, 3]);         // number | undefined
const s = first(["a", "b"]);        // string | undefined

// Cheklov (constraint): T da id bo'lishi shart
function findById<T extends { id: number }>(items: T[], id: number): T | undefined {
  return items.find((item) => item.id === id);
}

// API funksiyasi
async function request<T>(url: string): Promise<T> {
  const res = await fetch(url);
  return res.json() as Promise<T>;
}
const cabins = await request<Cabin[]>("/api/cabins");
```

Siz allaqachon generic ishlatgansiz: `useState<User | null>(null)`, `Promise<T>`, `Array<T>`.

**Intervyuda shunday ayting:** "Generic turni parametr sifatida qabul qilib, funksiya yoki component'ni turli turlar uchun qayta ishlatishga imkon beradi va tur ma'lumotini saqlaydi. `useState<T>`, API so'rov funksiyasi va qayta ishlatiladigan List yoki Table component'larida ishlataman."

---

### B9. [Strong junior] Utility type'lardan qaysilarini bilasiz?

**Oddiy qilib:** Mavjud turdan **yangi tur yasovchi** tayyor "asboblar". Turlarni qayta yozmaslik uchun.

```ts
type User = { id: number; name: string; email: string; password: string };

type UserUpdate = Partial<User>;               // hamma maydon ixtiyoriy
type UserPreview = Pick<User, "id" | "name">;  // faqat tanlanganlar
type PublicUser = Omit<User, "password">;      // tanlangandan tashqari hammasi
type FullUser = Required<User>;                // hamma maydon majburiy
type RoleLabels = Record<Role, string>;        // { admin: string; chef: string; ... }
type ReadonlyUser = Readonly<User>;            // o'zgartirib bo'lmaydi

type Data = ReturnType<typeof getUser>;        // funksiya qaytaradigan tur
type Resolved = Awaited<Promise<User>>;        // Promise ichidagi tur: User
type NewCabin = Omit<Cabin, "id" | "created_at">;   // yaratish formasi uchun
```

**Intervyuda shunday ayting:** "Ko'p ishlatadiganlarim: `Partial` (yangilash so'rovlari), `Pick` va `Omit` (bir turdan kichikroq tur), `Record` (kalit-qiymat xaritalari). Masalan, yaratish formasi turi uchun `Omit<Cabin, "id">`, shunda Cabin turi o'zgarsa, forma turi ham avtomatik yangilanadi."

---

### B10. [Strong junior] `keyof`, `typeof` va indexed access type nima?

**Oddiy qilib:**

- `typeof` (tur kontekstida): **qiymatdan** tur olish
- `keyof`: tur kalitlarining union'i
- `Tur["kalit"]`: shu kalitdagi tur

```ts
const config = { apiUrl: "...", timeout: 5000 };
type Config = typeof config;        // { apiUrl: string; timeout: number }
type ConfigKey = keyof Config;      // "apiUrl" | "timeout"
type Timeout = Config["timeout"];   // number

// Faqat mavjud kalitni qabul qiladigan funksiya
function getValue<K extends keyof Config>(key: K): Config[K] {
  return config[key];
}
getValue("timeout");   // number
getValue("foo");       // xato: bunday kalit yo'q
```

**Intervyuda shunday ayting:** "Bu operatorlar mavjud qiymat yoki turdan yangi tur yasashga yordam beradi, turlarni takrorlamaslik uchun. Masalan, sozlamalar obyektidan tur olaman va funksiyaga faqat mavjud kalit berilishini kafolatlayman."

---

### B11. [Strong junior] `as`, non-null assertion (`!`) va `satisfies` farqi?

**Oddiy qilib:**

- **`as`** TypeScript'ga "menga ishon, bu shu tur" deyish. Tekshiruvni **chetlab o'tadi**, siz xato qilsangiz, TypeScript indamaydi
- **`!`** "bu `null` emas, ishon" deyish. Xuddi shunday xavfli
- **`satisfies`** "bu qiymat shu turga **mosligini tekshir**, lekin qiymatning aniq turini saqla"

```ts
const input = document.getElementById("name") as HTMLInputElement;   // DOM'da kerak
const el = document.getElementById("root")!;   // null bo'lmasligiga ishonchingiz komil bo'lsa

const user = {} as User;   // XAVFLI: bo'sh obyekt User emas, lekin TypeScript jim

const theme = {
  primary: "#4f46e5",
  danger: "#dc2626",
} satisfies Record<string, string>;
theme.primary;   // aniq kalit taniladi, typo bo'lsa xato beradi
```

**Intervyuda shunday ayting:** "`as` va `!` ni kamdan-kam ishlataman, faqat TypeScript bilmaydigan narsani aniq bilganimda, masalan DOM elementi turi. `as any` dan butunlay qochaman. Konfiguratsiya obyektlarida `satisfies` ishlataman: tekshiruv ham bor, aniq tur ham saqlanadi."

---

### B12. [Junior] `enum` yoki literal union: qaysi biri yaxshi? `as const` nima?

**Oddiy qilib:** `enum` nomlangan konstantalar to'plami, lekin u JavaScript'ga **qo'shimcha kod** sifatida qoladi va ba'zi g'alati xatti-harakatlari bor. Ko'p jamoalar o'rniga **literal union** va **`as const`** ishlatadi.

```ts
// enum
enum Status { Pending = "pending", Done = "done" }

// Literal union (ko'pincha afzal)
type Status = "pending" | "done";

// Ham qiymatlar ro'yxati, ham tur kerak bo'lsa
const STATUSES = ["pending", "preparing", "delivered"] as const;
type Status = (typeof STATUSES)[number];   // "pending" | "preparing" | "delivered"
STATUSES.map((s) => <option key={s}>{s}</option>);   // select uchun ro'yxat ham bor
```

`as const` qiymatni **o'zgarmas va aniq literal** qiladi: `["pending"]` `string[]` emas, `readonly ["pending"]` bo'ladi.

**Intervyuda shunday ayting:** "Ko'pincha literal union ishlataman: JavaScript'ga qo'shimcha kod qo'shmaydi va API'dan keladigan string'lar bilan to'g'ridan-to'g'ri mos. Ham ro'yxat, ham tur kerak bo'lsa, `as const` massivdan turni chiqaraman."

---

### B13. [Junior] React'da TypeScript: props, children, event va hook'lar qanday tiplanadi?

```tsx
// Props
type CabinCardProps = {
  cabin: Cabin;
  onDelete?: (id: number) => void;   // ixtiyoriy callback
  children?: React.ReactNode;        // istalgan render qilinadigan narsa
};

function CabinCard({ cabin, onDelete, children }: CabinCardProps) { ... }

// useState: boshlang'ich qiymatdan tur aniqlanmasa, qo'lda
const [user, setUser] = useState<User | null>(null);
const [count, setCount] = useState(0);        // number: o'zi aniqlaydi

// useRef: DOM element
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();

// Event'lar
function handleChange(e: React.ChangeEvent<HTMLInputElement>) { setQuery(e.target.value); }
function handleSubmit(e: React.FormEvent<HTMLFormElement>) { e.preventDefault(); }
function handleClick(e: React.MouseEvent<HTMLButtonElement>) { ... }

// Standart HTML atributlarini meros olish
type ButtonProps = React.ComponentProps<"button"> & { variant?: "primary" | "danger" };
function Button({ variant = "primary", ...rest }: ButtonProps) {
  return <button data-variant={variant} {...rest} />;   // onClick, disabled, type avtomatik
}
```

Event turini eslab qolish qiyin bo'lsa: JSX'da handler ustiga sichqonchani olib borsangiz, editor turini ko'rsatadi.

**Intervyuda shunday ayting:** "Props'ni `type` bilan, children'ni `React.ReactNode` bilan tiplayman. `useState` ga boshlang'ich qiymat yetarli bo'lmasa (`null` kabi), generic beraman. Qayta ishlatiladigan Button kabi component'larda `React.ComponentProps<"button">` dan meros olaman, shunda barcha standart atributlar avtomatik ishlaydi."

---

### B14. [Middle] API javobini faqat TypeScript bilan tiplash yetarlimi? Zod nima uchun?

**Oddiy qilib:** TypeScript turlari faqat kod yozish paytida bor, brauzerda **yo'qoladi**. Siz `res.json() as User[]` deb yozsangiz, bu shunchaki **va'da**: server boshqa shakl qaytarsa, TypeScript buni sezmaydi va xato ilovaning boshqa joyida, tushunarsiz holda chiqadi. **Zod** ma'lumotni brauzerda, haqiqatda **tekshiradi**, turni esa sxemadan avtomatik chiqaradi.

```ts
import { z } from "zod";

const CabinSchema = z.object({
  id: z.number(),
  name: z.string(),
  maxCapacity: z.number().int().positive(),
  discount: z.number().default(0),
});
type Cabin = z.infer<typeof CabinSchema>;      // turni qo'lda yozish shart emas

async function getCabins(): Promise<Cabin[]> {
  const res = await fetch("/api/cabins");
  const json: unknown = await res.json();
  return z.array(CabinSchema).parse(json);     // mos kelmasa, aniq xato tashlaydi
}

// Xato tashlamay tekshirish
const result = CabinSchema.safeParse(data);
if (!result.success) console.error(result.error.issues);
```

**Intervyuda shunday ayting:** "TypeScript turlari runtime'da yo'qoladi, shuning uchun tashqaridan keladigan ma'lumotga (API, forma, URL, localStorage) to'liq ishonib bo'lmaydi. Muhim joylarda Zod sxemasi bilan tekshiraman: bitta sxema ham validatsiya, ham TypeScript turi bo'ladi. Gross'da telefon raqamini Zod bilan tekshirganman."

---

### B15. [Strong junior] `tsconfig.json` dagi `strict` nima qiladi?

**Oddiy qilib:** `strict: true` bir nechta qattiq tekshiruvni birdaniga yoqadi. Eng muhimlari:

- **`strictNullChecks`**: `null` va `undefined` alohida tur. Ularni tekshirmasdan ishlatib bo'lmaydi (eng ko'p xatoni shu topadi)
- **`noImplicitAny`**: TypeScript turni aniqlay olmasa, jim `any` qo'ymaydi, xato beradi
- **`strictFunctionTypes`**, **`strictPropertyInitialization`** va boshqalar

```ts
// strict o'chiq: ishlaydi, lekin runtime'da xato
// strict yoqiq: "user possibly null" xatosi
function getName(user: User | null) {
  return user.name;     // xato
  return user?.name;    // to'g'ri
}
```

**Intervyuda shunday ayting:** "Yangi loyihada doim `strict: true` yoqaman. Ayniqsa `strictNullChecks` 'cannot read property of undefined' kabi eng ko'p uchraydigan xatolarni kod yozayotganda ushlaydi."
