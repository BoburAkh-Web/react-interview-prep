# Section 11: How React Works Behind the Scenes

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Bu section interview'da eng ko'p so'raladigan "React ichida nima sodir bo'ladi?" savollarining manbai. Avval **katta rasm**, keyin savol-javoblar.

---

## Katta rasm: bitta renderning hayoti

```
 TRIGGER                 RENDER PHASE                 COMMIT PHASE            BROWSER
 (sabab)           (React hisob-kitob qiladi)      (React DOM'ni yozadi)       (chizadi)
┌──────────┐      ┌────────────────────────┐      ┌───────────────────┐     ┌─────────┐
│ 1. Birinchi│     │ Component funksiyalari │      │ Farqlar DOM'ga    │     │ Ekran   │
│    render  │ ──► │ chaqiriladi → yangi    │ ──►  │ yoziladi (qo'shish│ ──► │ yangi-  │
│ 2. State   │     │ virtual DOM yasaladi → │      │ / o'chirish /     │     │ lanadi  │
│    o'zgardi│     │ eskisi bilan solish-   │      │ yangilash)        │     │         │
└──────────┘      │ tiriladi (diffing)     │      └───────────────────┘     └─────────┘
                  └────────────────────────┘
                  Hali DOM'ga TEGILMAYDI!         DOM faqat shu yerda o'zgaradi
```

**Esda saqlash uchun o'xshatish (restoran):**

- **Trigger** = mijoz buyurtma berdi
- **Render phase** = oshpaz **qog'ozda** yangi menyu/buyurtmani eskisi bilan solishtiradi (hali hech narsa pishirilmagan, faqat reja)
- **Commit phase** = ofitsiant **faqat o'zgargan taomlarni** stolga qo'yadi yoki olib ketadi

"Render" so'zi shu yerda chalkash: React'da **render = component funksiyalarini chaqirish** (qog'ozdagi reja). Ekranga chizish emas.

---

## 1. [Junior] Component, instance va element farqi nima?

- **Component**: UI bo'lagining **tavsifi**, ya'ni oddiy funksiya (reja, qolip).
- **Instance** (nusxa): component'dan foydalanganimizda React xotirada yaratadigan **"tirik" nusxa**. Uning o'z state'i, props'i va hayot tsikli bor.
- **Element**: component chaqirilganda **qaytadigan natija** (JSX): React'ga "ekranda nima bo'lishi kerak" deb aytadigan oddiy **obyekt**.

```jsx
function Tab() { ... }            // component (qolip)

<Tab />                           // bu yerda instance yaratiladi
<Tab />                           // bu BOSHQA instance (o'z state'i bilan)
```

O'xshatish: component = uyning **chizmasi**, instance = chizma bo'yicha qurilgan **haqiqiy uy** (har uyning o'z jihozlari bor), element = uyning qanday ko'rinishini tavsiflovchi **rasm**.

Muhim natija: bir xil component'ni ikki marta ishlatsak, ularning state'lari **alohida** bo'ladi (bittasi o'zgarsa, ikkinchisiga ta'sir qilmaydi).

## 2. [Strong junior] JSX nimaga aylanadi? React elementi DOM elementidan nimasi bilan farq qiladi?

JSX `React.createElement` chaqiruvi bo'lib, natijasi **React element**, ya'ni oddiy JavaScript obyekt:

```jsx
<Tab name="A" />
// taxminan shunday obyekt:
// { type: Tab, props: { name: "A" }, key: null, ... }
```

Bu obyekt **hech narsa chizmaydi**, faqat tavsif. React uni oladi, component funksiyasini chaqiradi va oxir-oqibat **haqiqiy DOM elementi**ga (`<div>`, `<button>`) aylantiradi. Ya'ni: component → (chaqirilsa) → React element(lar) → (commit paytida) → DOM element(lar).

## 3. [Junior] React renderi qanday bosqichlardan o'tadi?

1. **Trigger**: render uchun sabab paydo bo'ladi
2. **Render phase**: React component funksiyalarini chaqiradi va nima o'zgarishi kerakligini hisoblaydi
3. **Commit phase**: React hisoblangan o'zgarishlarni haqiqiy DOM'ga yozadi
4. **Brauzer chizadi** (paint)

## 4. [Junior] Render qachon boshlanadi? (Trigger'lar)

Faqat ikki holatda:

- **Birinchi render** (`createRoot(...).render(<App />)`)
- **State o'zgarishi** (`setState` chaqirilganda), shuningdek Context qiymati o'zgarganda

Muhim: **props o'zgarishi o'zi render'ni boshlamaydi.** Child'ning props'i o'zgargan bo'lsa, buning sababi parent'ning state'i o'zgargani va parent **qayta render bo'lgani**dir.

Yana bir muhim qoida: component qayta render bo'lsa, uning **barcha child'lari ham** qayta render bo'ladi (props o'zgarmagan bo'lsa ham).

```
App (state o'zgardi) ──► qayta render
 ├── Header   ──► qayta render
 └── List     ──► qayta render
      └── Item ──► qayta render
```

Bu odatda tez ishlaydi va **normal holat**; muammo faqat component juda og'ir bo'lganda (optimizatsiya keyingi section'larda).

## 5. [Strong junior] "Render" va "DOM yangilanishi" bir narsami?

**Yo'q.** Component render bo'lishi (funksiya chaqirilishi) ekranda hech narsa o'zgarmaganini ham anglatishi mumkin. Agar yangi va eski natija bir xil bo'lsa, commit paytida DOM'ga **hech narsa yozilmaydi**.

> Render = hisob-kitob (tez, faqat JavaScript). Commit = DOM'ni o'zgartirish (qimmat). React shuning uchun avval solishtiradi, keyin faqat farqni yozadi.

## 6. [Strong junior] Render phase'da aniq nima sodir bo'ladi?

1. React render kerak bo'lgan component funksiyalarini **chaqiradi**
2. Natijada **yangi virtual DOM** (React elementlari daraxti) hosil bo'ladi
3. Yangi virtual DOM **oldingisi bilan solishtiriladi** (reconciliation / diffing)
4. Natijada "DOM'da nimalarni o'zgartirish kerak" ro'yxati (effects list) tayyorlanadi

Bu bosqichda DOM'ga **hech narsa tegilmaydi**. Ichkarida React buning uchun **Fiber** daraxtidan foydalanadi: har bir component instance va DOM element uchun bitta "fiber" (ish birligi) bor, unda state, props, bajarilishi kerak bo'lgan ishlar saqlanadi. Fiber'lar har renderda qaytadan yaratilmaydi, ular yangilanib turadi. Shu tuzilma tufayli React render ishini **to'xtatib, keyin davom ettira oladi** (concurrent funksiyalar buning ustiga qurilgan).

## 7. [Strong junior] Virtual DOM nima? U haqiqiy DOM'dan tezmi?

Virtual DOM bu React elementlaridan tuzilgan **daraxt** (oddiy JavaScript obyektlari), haqiqiy DOM'ning yengil nusxasi. Har renderda yangisi hosil bo'ladi va eskisi bilan solishtiriladi.

"Virtual DOM tezroq" degani aniq emas. Aslida u **tezlik uchun emas, qulaylik uchun**: biz UI'ni "state'ning funksiyasi" sifatida deklarativ yozamiz, React esa DOM'da **minimal zarur o'zgarishni** o'zi topib bajaradi, qo'lda `document.querySelector` yozish shart emas. DOM operatsiyalari qimmat bo'lgani uchun ularni kamaytirish foyda beradi, lekin virtual DOM'ning o'zi ham hisob-kitob talab qiladi.

## 8. [Strong junior] Commit phase'da nima sodir bo'ladi?

React render phase'da topilgan farqlarni **haqiqiy DOM'ga yozadi**: yangi elementlarni qo'shadi, keraksizlarini o'chiradi, o'zgarganlarini yangilaydi.

- Bu ishni **ReactDOM** (renderer) bajaradi, hisob-kitobni esa React o'zi (reconciler). Shuning uchun React boshqa muhitlarda ham ishlaydi (React Native, boshqa renderer'lar).
- Commit phase **sinxron** va to'xtatib bo'lmaydi (yarimta yangilangan UI ko'rinib qolmasligi uchun).
- Tartib: DOM yoziladi → (layout effect'lar) → brauzer ekranni chizadi → odatda shundan keyin `useEffect` ishlaydi.

## 9. [Strong junior] Diffing (reconciliation) qanday ishlaydi? Asosiy qoidalar qanday?

React eski va yangi daraxtni solishtirishda ikkita taxminga tayanadi:

1. **Turi har xil bo'lgan ikki element** turli daraxt hosil qiladi
2. **Barqaror `key`** ga ega element renderlar orasida bir xil element deb tan olinadi

Daraxtdagi **bir xil joyda (position)** quyidagicha:

| Holat | Natija |
|---|---|
| Element turi **boshqa** (`<div>` → `<section>`, yoki `<A />` → `<B />`) | Eskisi **butunlay yo'q qilinadi** (state'i va child'lari bilan), yangisi noldan yaratiladi |
| Element turi **bir xil** | DOM element **saqlanadi**, faqat o'zgargan atribut/props yangilanadi, ichki child'lar rekursiv solishtiriladi. Component bo'lsa, **state saqlanadi** |

## 10. [Strong junior] Mashhur tuzoq: ikki xil shartli component'da state saqlanib qoladimi?

```jsx
{isFirst ? <Counter name="A" /> : <Counter name="B" />}
```

Ikkala holatda ham **bir xil joyda bir xil tur** (`Counter`) turibdi. Shuning uchun React uni **bir xil instance** deb hisoblaydi va state'ni **saqlab qoladi**, faqat `name` prop yangilanadi. Foydalanuvchi "boshqa" component ko'ryapti deb o'ylaydi, lekin eski hisoblagich qiymati qolib ketadi. Agar turlar boshqa bo'lsa (`<CounterA />` va `<CounterB />`), state yo'qoladi.

Tuzatish kerak bo'lsa, `key` beriladi (12-savol).

## 11. [Strong junior] Ro'yxatda `key` nega kerak? Indeksni key qilsak nima bo'ladi?

React ro'yxat elementlarini **joyi** bo'yicha emas, `key` bo'yicha taniydi. Barqaror `key` bo'lsa, element ro'yxat ichida joyini almashtirsa ham React uni **bir xil element** deb biladi: DOM element va state'ni saqlaydi, faqat joyini ko'chiradi.

Indeks `key` bo'lsa: element qo'shilsa, o'chirilsa yoki tartib o'zgarsa, indekslar boshqa elementlarga ko'chadi, React esa noto'g'ri elementga eski state'ni (masalan, input ichidagi matn, belgilangan checkbox) biriktirib qo'yadi. Shuning uchun `key` ma'lumotning **barqaror `id`**si bo'lishi kerak.

## 12. [Strong junior] `key` bilan state'ni qanday qayta tiklaymiz (reset)?

`key` o'zgarganda React uni **boshqa element** deb hisoblaydi: eskisini yo'q qiladi (state bilan birga), yangisini noldan yaratadi. Shu orqali state'ni qasddan reset qilish mumkin:

```jsx
<FormSplitBill key={selectedFriend.id} selectedFriend={selectedFriend} />
```

Boshqa do'st tanlansa, formadagi eski summalar o'chib ketadi. (Section 8 dagi Eat-'N-Split loyihasi misoli.) Bu `useEffect` ichida state'ni qo'lda tozalashdan ancha sodda va ishonchli.

## 13. [Strong junior] Pure component nima? Render logikasi qanday qoidalarga bo'ysunishi kerak?

Render logikasi (component funksiyasining tanasi, event handler va effect'lardan **tashqari** qismi) **toza (pure)** bo'lishi kerak: bir xil props va state berilsa, **doim bir xil JSX** qaytarishi kerak.

Render paytida **qilish mumkin emas**:

- API so'rovi, timer, event listener o'rnatish
- DOM'ni qo'lda o'zgartirish
- Tashqi o'zgaruvchi yoki props obyektini **mutatsiya** qilish
- `setState` chaqirish (cheksiz render tsikliga olib keladi)
- Tasodifiy qiymat yoki `Date.now()` ni natijaga ta'sir qiladigan holda ishlatish

Sabab: React render'ni istalgan paytda to'xtatishi, qayta ishga tushirishi yoki ikki marta chaqirishi mumkin. Toza bo'lmagan kod bunda xato beradi. Strict Mode (development'da) component'ni ataylab ikki marta render qilib, bunday xatolarni ko'rsatib beradi.

## 14. [Strong junior] Side effect'larni qayerda yozamiz?

**Render logikasidan tashqarida**:

- **Event handler'larda**: foydalanuvchi harakatiga javoban (tugma bosish, forma yuborish). Aksariyat side effect'lar shu yerda bo'lishi kerak
- **`useEffect` ichida**: component ekranga chiqqandan keyin yoki ma'lum qiymat o'zgarganda bajarilishi kerak bo'lgan ishlar uchun (masalan, ma'lumot yuklash)

## 15. [Strong junior] State update batching nima va nega kerak?

Bir event handler ichida bir necha marta `setState` chaqirilsa, React ularni **bitta render'ga birlashtiradi** (batching).

```jsx
function handleClick() {
  setA(1);
  setB(2);
  setC(3);
}   // 3 ta emas, faqat 1 ta re-render
```

Sabab: har `setState` uchun alohida render qilish **keraksiz ish** va noto'g'ri oraliq holatni ekranda ko'rsatishi mumkin (masalan, `A` yangilangan, `B` hali eski). Batching ish samaradorligini oshiradi va UI'ni izchil saqlaydi.

## 16. [Strong junior] `setState` dan keyin state nega darrov yangilanmaydi?

State yangilanishi **asinxron**: `setState` yangi render'ni **rejalashtiradi**, joriy render'dagi o'zgaruvchi esa o'zgarmaydi (u shu render uchun "qotib qolgan surat").

```jsx
const [likes, setLikes] = useState(0);

function handleClick() {
  setLikes(likes + 1);
  console.log(likes);   // ESKI qiymat (masalan, 0)
}
```

Yangi qiymat faqat **keyingi renderda** ko'rinadi. Shu sababdan eng so'nggi state'ga tayanish kerak bo'lsa, **callback** ishlatiladi:

```jsx
function handleTripleInc() {
  setLikes(likes + 1);
  setLikes(likes + 1);
  setLikes(likes + 1);      // natija: faqat +1 (uchalasi bir xil eski `likes` ni ko'radi)
}

function handleTripleIncCorrect() {
  setLikes((l) => l + 1);
  setLikes((l) => l + 1);
  setLikes((l) => l + 1);   // natija: +3
}
```

Yangi qiymat bilan darrov ish qilmoqchi bo'lsangiz, uni alohida o'zgaruvchiga hisoblab olib, ham `setState` ga, ham keyingi qatorga bering.

## 17. [Middle] React 18 dagi automatic batching nima? `flushSync` qachon kerak?

React 17 va undan oldin batching faqat **React event handler'lari** ichida ishlardi. `setTimeout`, `Promise` yoki native event'lar ichida har `setState` alohida render keltirardi. **React 18** da (`createRoot` ishlatilganda) batching **hamma joyda** avtomatik ishlaydi: timeout, promise, native handler'larda ham.

Kamdan-kam holatda yangilanishni **darrov** DOM'ga yozish kerak bo'lsa (masalan, state o'zgargach darrov DOM o'lchash), `flushSync` bilan batching'dan chiqish mumkin. Bu odatda kerak bo'lmaydi va unumdorlikka salbiy ta'sir qilishi mumkin.

## 18. [Strong junior] React'da event'lar qanday ishlaydi? Event delegation va synthetic event nima?

**Event delegation:** React har bir elementga alohida listener qo'ymaydi. Listener'ni **bitta joyda, root container'da** (React daraxti render qilinadigan `#root` elementida) o'rnatadi. Brauzer event'i yuqoriga ko'tarilib (**bubbling**) root'ga yetganda, React qaysi element bosilganini aniqlaydi va unga tegishli handler'ni chaqiradi. Bu ko'p element bo'lganda samarali.

**Synthetic event:** handler'ga kelayotgan `e` oddiy brauzer event'i emas, React yaratgan **o'rovchi (wrapper)** obyekt (`SyntheticEvent`). U brauzerlar orasidagi farqlarni tekislaydi va native event bilan deyarli bir xil interfeysga ega (`e.preventDefault()`, `e.stopPropagation()`, `e.target`). Asl event kerak bo'lsa, `e.nativeEvent` orqali olinadi.

## 19. [Middle] React event'lari native event'lardan qanday farq qiladi?

- Ko'pchilik React event'lari **bubble qiladi**, shu jumladan `onFocus`, `onBlur` (React ichida `focusin`/`focusout` ishlatiladi). `onScroll` bundan mustasno: u bubble qilmaydi (React 17 dan)
- `onChange` input'da **har bir belgi kiritilganda** ishlaydi (native `change` esa input fokusni yo'qotganda ishlardi)
- Event nomlari camelCase (`onClick`), handler'ga **funksiya** beriladi, string emas
- Bubbling'ni to'xtatish uchun `e.stopPropagation()`, brauzerning standart harakatini bekor qilish uchun `e.preventDefault()` ishlatiladi. Ular **ikki xil narsa**: biri event'ni yuqoriga uzatmaydi, ikkinchisi standart harakatni (masalan, form submit) to'xtatadi

## 20. [Junior] React kutubxonami yoki framework? Ekotizim nima?

React rasmiy ravishda **view kutubxonasi**: faqat UI qismi uchun mas'ul (component'lar, state, render). Routing, ma'lumot yuklash, global state, forma va stil kabi narsalar uning ichida **yo'q**, ularni ekotizimdan o'zimiz tanlaymiz:

| Vazifa | Misollar |
|---|---|
| Routing | React Router |
| Server ma'lumoti | React Query (TanStack Query) |
| Global state | Redux Toolkit, Zustand, Context |
| Forma | React Hook Form |
| Stil | Tailwind CSS, styled-components |
| Framework (React ustiga qurilgan) | Next.js, Remix |

**Framework** (masalan, Angular) hammasini birga, tayyor qilib beradi. **Kutubxona**da erkinlik ko'p, lekin to'g'ri kombinatsiyani o'zimiz tanlashimiz va ularni bir-biriga ulashimiz kerak. Next.js kabi "React framework"lar aynan shu bo'shliqni to'ldiradi.

## 21. [Middle] Parent qayta render bo'lganda child nega ham render bo'ladi? Buni qanday cheklaymiz?

Chunki render **funksiyani chaqirish** va child'lar parent'ning JSX'ida yoziladi: parent funksiyasi qayta ishlasa, uning ichidagi `<Child />` elementlari ham qaytadan yaratiladi va React ularni ham render qiladi. Props o'zgarganmi yoki yo'qmi, tekshirilmaydi.

Cheklash usullari (keyingi section'larda batafsil):

- `memo` bilan component'ni o'rash (props o'zgarmasa, render'ni o'tkazib yuboradi)
- State'ni **kerakli joyga yaqinroq** tushirish (kamroq component'ga ta'sir qiladi)
- **Composition** (`children`): `children` parent ichida yaratilgan bo'lsa, o'sha state o'zgarganda qayta yaratilmaydi

Lekin avval **o'lchash** kerak: ko'p holatda qo'shimcha render sezilmaydi va erta optimizatsiya kodni murakkablashtiradi.

## 22. [Middle] Component keraksiz qayta render bo'layotganini qanday aniqlaysiz?

- **React DevTools → Profiler**: yozib olib, qaysi component qancha vaqt va nima sababdan render bo'lganini ko'rish
- DevTools sozlamasida **"Highlight updates when components render"** ni yoqish: render bo'lgan component'lar ekranda belgilanadi
- Vaqtincha `console.log` bilan render sonini kuzatish

Asosiy fikr: ko'p render o'zi xato emas. Foydalanuvchi **sustlikni sezganda** va Profiler aniq muammoli joyni ko'rsatganda optimizatsiya qilinadi.

---

## Qisqa xulosa jadvali (Cheat sheet)

| Tushuncha | Bir jumlada |
|---|---|
| Component | UI bo'lagining tavsifi (funksiya) |
| Instance | Component'ning xotiradagi "tirik" nusxasi (o'z state'i bor) |
| Element | Component qaytargan obyekt (JSX natijasi), faqat tavsif |
| Render | Component funksiyalarini chaqirish (DOM o'zgarmaydi) |
| Commit | Topilgan farqlarni DOM'ga yozish |
| Trigger | Birinchi render yoki state o'zgarishi |
| Diffing | Eski va yangi daraxtni bir xil joylarda solishtirish |
| Key | Elementni renderlar orasida taniydigan barqaror belgi; o'zgarsa, state reset bo'ladi |
| Pure render | Bir xil props/state → bir xil JSX; render ichida side effect yo'q |
| Batching | Bir nechta `setState` → bitta render |
| Event delegation | Listener root'da, handler'ni React o'zi topadi |

## Eng ko'p uchraydigan 5 ta chalkashlik

1. **"Render = ekran yangilandi."** Yo'q: render hisob-kitob, ekran faqat commit'da o'zgaradi.
2. **"Props o'zgarsa component render bo'ladi."** Aslida parent render bo'lgani uchun render bo'ladi.
3. **"setState holatni darrov o'zgartiradi."** Yo'q: yangi qiymat keyingi renderda ko'rinadi.
4. **"Bir xil component har safar yangi bo'ladi."** Yo'q: bir xil joyda bir xil tur bo'lsa, state saqlanadi (`key` bilan boshqarish mumkin).
5. **"Virtual DOM tez bo'lgani uchun React tez."** Asosiy foyda: deklarativ yozish va DOM'ga minimal o'zgartirish yozish.
