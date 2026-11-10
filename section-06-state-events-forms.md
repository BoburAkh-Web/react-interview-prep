# Section 6: State, Events, and Forms: Interactive Components

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**. This section has no Middle-level questions.

Projects in this section: Steps component, Date Counter, "Far Away" travel list.

---

## 1. [Junior] Event'ni React'da qanday ulaymiz? `onClick={handle}` va `onClick={handle()}` farqi nima?

Event nomlari camelCase yoziladi (`onClick`, `onChange`) va ularga **funksiya** beriladi.

```jsx
<button onClick={handleNext}>Next</button>     // to'g'ri: bosilganda chaqiriladi
<button onClick={handleNext()}>Next</button>   // xato: render paytidayoq chaqirilib ketadi
```

Argument kerak bo'lsa, o'rab beriladi: `onClick={() => handleNext(2)}`.

## 2. [Junior] State nima? Oddiy o'zgaruvchidan farqi nima?

State bu component'ning "xotirasi". Oddiy o'zgaruvchi (`let step = 1`) har renderda qaytadan boshlanadi va o'zgarsa ham React UI'ni yangilamaydi. State esa renderlar orasida saqlanadi va o'zgarganda React component'ni **qayta render** qiladi.

## 3. [Junior] `useState` qanday ishlatiladi?

```jsx
const [step, setStep] = useState(1);
```

U ikki narsa qaytaradi: hozirgi qiymat (`step`) va uni o'zgartiruvchi funksiya (`setStep`). Boshlang'ich qiymat (`1`) faqat **birinchi renderda** ishlatiladi, keyingi renderlarda React saqlangan qiymatni beradi.

## 4. [Junior] Nega state'ni qo'lda o'zgartirib bo'lmaydi (`step = 2` yoki `step++`)?

React bunday o'zgarishni **sezmaydi**, shuning uchun component qayta render bo'lmaydi va ekran eskicha qoladi. Hatto `const` bo'lgani uchun xato ham beradi. State faqat `setStep(...)` orqali o'zgartiriladi, chunki React shu chaqiruv orqali yangi render kerakligini biladi.

## 5. [Strong junior] `setStep(2)` dan keyin darrov `console.log(step)` yozsak, nima chiqadi?

Eski qiymat. `setStep` state'ni darhol o'zgartirmaydi, u **yangi renderni rejalashtiradi**. Joriy renderdagi `step` o'zgarmaydi (u shu render uchun "surat" kabi qotib qolgan), yangi qiymat faqat keyingi renderda ko'rinadi.

## 6. [Strong junior] Component funksiya har renderda qayta ishlaydi, state qanday yo'qolmaydi?

State component funksiya ichida emas, **React ichida** saqlanadi. React har `useState` chaqiruvini **chaqirilish tartibi** bo'yicha eslab qoladi. Shuning uchun hook'larni shart yoki sikl ichida chaqirib bo'lmaydi (tartib buzilsa, React qaysi state qaysi hook'ga tegishli ekanini adashtiradi).

## 7. [Junior] Bitta component'da bir nechta `useState` ishlatsa bo'ladimi?

Ha. Har bir mustaqil ma'lumot uchun alohida `useState` yoziladi:

```jsx
const [step, setStep] = useState(1);
const [isOpen, setIsOpen] = useState(true);
```

Biri o'zgarsa, ikkinchisining qiymati saqlanib qoladi, lekin component baribir bitta marta qayta render bo'ladi.

## 8. [Junior] State o'zgarganda React nima qiladi?

State o'zgarganda React shu component'ni (va uning child'larini) **qayta render** qiladi: component funksiyasini yangi state bilan qaytadan chaqiradi, yangi JSX'ni oldingisi bilan solishtiradi va DOM'da faqat o'zgargan joyni yangilaydi.

## 9. [Junior] React Developer Tools nima uchun kerak?

Brauzer kengaytmasi. **Components** tabida component daraxtini, har bir component'ning props va state qiymatlarini ko'rish (hatto o'zgartirib sinab ko'rish) mumkin. **Profiler** tabi esa qaysi component qancha vaqt render bo'lganini ko'rsatadi. Xatoni qidirishda `console.log` dan tezroq.

## 10. [Strong junior] State'ni hozirgi state asosida yangilashda nega callback ishlatiladi?

```jsx
setStep((s) => s + 1);   // to'g'ri: eng so'nggi state beriladi
setStep(step + 1);       // joriy renderdagi "qotib qolgan" step'ga tayanadi
```

Callback shaklida React o'zi eng so'nggi state'ni beradi. Shuning uchun state eski qiymat bo'lib qolishi mumkin bo'lgan joylarda (bir nechta ketma-ket yangilash, setTimeout, async kod) callback xavfsiz.

## 11. [Strong junior] Bitta handler ichida `setCount(count + 1)` ni uch marta chaqirsak, count qanchaga oshadi?

Faqat **1 ga**. Uchala chaqiruv ham joriy renderdagi bir xil `count` ni ko'radi (masalan, 0), hammasi `setCount(1)` bo'lib qoladi. Callback shaklida (`setCount(c => c + 1)`) yozsak, 3 ga oshadi. React 18 da bunday ketma-ket yangilashlar **batching** orqali bitta render'ga birlashtiriladi.

## 12. [Junior] Qaysi ma'lumotni state'ga qo'yish kerak, qaysini yo'q? ("Derived state" nima?)

State'ga faqat **vaqt o'tishi bilan o'zgaradigan va UI'ga ta'sir qiladigan** ma'lumot qo'yiladi. Boshqa state yoki props'dan **hisoblab olish mumkin bo'lgan** narsani alohida state qilmaslik kerak:

```jsx
const [items, setItems] = useState([]);
const total = items.length;   // state emas, har renderda hisoblanadi
```

Aks holda ikki joyda bir xil ma'lumotni sinxron saqlashga to'g'ri keladi va xato chiqishi mumkin. Date Counter'dagi sana ham shunday: u `count` dan hisoblab olinadi.

## 13. [Strong junior] Massiv yoki obyekt state'ini nega mutatsiya qilmasdan, yangi nusxa bilan yangilaymiz?

React state o'zgarganini **havola (reference) bo'yicha** solishtiradi. `items.push(x)` massivning o'zini o'zgartiradi, havola esa o'sha-o'sha qoladi, shuning uchun React o'zgarishni sezmaydi.

```jsx
setItems((items) => [...items, newItem]);                        // qo'shish
setItems((items) => items.filter((i) => i.id !== id));           // o'chirish
setItems((items) => items.map((i) => i.id === id ? { ...i, packed: !i.packed } : i)); // o'zgartirish
```

## 14. [Strong junior] Vanilla JS va React yondashuvi qanday farq qiladi?

Vanilla JS'da (imperative) siz DOM'ni qo'lda qidirib, o'zgartirib borasiz. React'da (declarative) siz UI **state'ning funksiyasi** ekanini yozasiz: `UI = f(state)`. State o'zgaradi, React UI'ni o'zi yangilaydi. Shuning uchun ma'lumot va ekran hech qachon bir-biriga to'g'ri kelmay qolmaydi.

## 15. [Junior] Form submit'da `event.preventDefault()` nega yoziladi?

Brauzer formani yuborganda sahifani **qayta yuklab** yuboradi, bu esa React state'ini yo'qotadi. `e.preventDefault()` shu standart xatti-harakatni to'xtatadi va formani o'zimiz JavaScript bilan boshqaramiz.

```jsx
function handleSubmit(e) {
  e.preventDefault();
  if (!description) return;   // bo'sh bo'lsa qo'shmaymiz
  // ...
}
```

## 16. [Junior] Controlled element (controlled input) nima?

Input'ning qiymatini DOM emas, **React state** boshqaradi:

```jsx
const [description, setDescription] = useState("");

<input value={description} onChange={(e) => setDescription(e.target.value)} />
```

Shunda React state "yagona haqiqat manbai" bo'ladi: input'dagi qiymatni istalgan joyda o'qish, tozalash yoki tekshirish oson (masalan, submit'dan keyin `setDescription("")`).

## 17. [Strong junior] Input'ga `value` berib, `onChange` bermasak nima bo'ladi?

Input faqat o'qish uchun bo'lib qoladi: foydalanuvchi yozolmaydi, chunki qiymatni React boshqaradi, lekin uni yangilaydigan hech narsa yo'q. React konsolda ogohlantirish ham chiqaradi.

## 18. [Junior] State va props farqi nima?

- **Props**: tashqaridan (parent'dan) keladi, component ichida **o'zgartirib bo'lmaydi**
- **State**: component'ning o'zida, **o'zi boshqaradi** va setter orqali o'zgartiradi

Ikkalasi o'zgarganda ham component qayta render bo'ladi. Parent'ning state'i child uchun props bo'lib boradi.
