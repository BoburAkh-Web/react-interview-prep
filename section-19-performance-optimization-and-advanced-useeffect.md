# Section 19: Performance Optimization and Advanced useEffect

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Projects in this section: The Atomic Blog (optimizatsiya mashqlari), WorldWise (`useCallback`, code splitting), Workout Timer (challenge).

> WorldWise'dagi haqiqiy optimizatsiyalar (`getCity` uchun `useCallback`, `lazy` + `Suspense`) `projects/worldwise-interview.md` faylining 3 va 7-bo'limlarida. Bu fayl **nazariya** uchun.

---

## 1. [Junior] React ilovasini tezlashtirishning qanday yo'nalishlari bor?

Uch asosiy yo'nalish:

1. **Keraksiz (wasted) render'larni oldini olish**
2. **Ilova tezligi va javob berishini yaxshilash** (og'ir hisob-kitoblarni qayta bajarmaslik)
3. **Bundle hajmini kamaytirish** (kamroq kod yuklash)

Har biri uchun alohida vositalar bor: `memo`, `children` hiylasi (1); `useMemo`, `useCallback` (2); code splitting va `lazy` (3).

## 2. [Junior] Wasted render nima? Qachon muammo bo'ladi?

**Wasted render**: component qayta render bo'ldi, lekin natija **avvalgisi bilan bir xil** va DOM o'zgarmadi. Bunday render'larning o'zi odatda **muammo emas**, React ularni tez bajaradi. Faqat render **juda tez-tez** bo'lsa yoki component **juda og'ir** bo'lsa (katta ro'yxat, murakkab hisob-kitob), foydalanuvchi sustlikni sezadi. Shu holatdagina optimizatsiya qilinadi.

Eslatma (Section 11 dan): component render bo'lishining sabablari: state o'zgarishi, Context o'zgarishi va **parent'ning render bo'lishi**, ya'ni parent render bo'lsa, props o'zgarmagan bo'lsa ham child ham render bo'ladi.

## 3. [Junior] React DevTools Profiler nima uchun kerak?

Profiler render'larni **o'lchaydi**: yozib olish (record) boshlanadi, foydalanuvchi ilova bilan ishlaydi, to'xtatilgach har bir commit uchun qaysi component **qancha vaqt** render bo'lgani va **nima sababdan** (state, props yoki parent) ko'rinadi. Flame chart'da og'ir component'lar ko'zga tashlanadi. Optimizatsiyadan oldin muammoni shu bilan **topish** kerak, taxmin qilib emas.

## 4. [Strong junior] `children` hiylasi nima? Nega u render'ni kamaytiradi?

Og'ir component state o'zgaradigan component bilan **bir joyda yaratilsa**, har state o'zgarishda u ham render bo'ladi. Uni `children` sifatida **tashqaridan** bersak, render bo'lmaydi.

```jsx
// Sekin: count o'zgarsa, SlowComponent ham qayta render bo'ladi
function Test() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>
      <SlowComponent />
    </div>
  );
}

// Tez: SlowComponent tashqarida yaratiladi va children sifatida keladi
function Counter({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>
      {children}
    </div>
  );
}
function Test() {
  return <Counter><SlowComponent /></Counter>;
}
```

Sabab: `<SlowComponent />` elementi `Test` render bo'lganda yaratiladi. `Counter`'ning state'i o'zgarganda `Test` qayta render bo'lmaydi, shuning uchun `children` prop'i **o'sha-o'sha element havolasi**da qoladi va React uni qayta render qilmaydi. Bu composition'ning performance foydasi.

## 5. [Strong junior] `memo` nima va qanday ishlaydi?

`memo(Component)` component'ni o'raydi va uni **faqat props o'zgargandagina** qayta render qiladi (parent render bo'lganda props'ni oldingisi bilan solishtiradi).

```jsx
const Archive = memo(function Archive({ show, onAddPost }) {
  // og'ir component
});
```

Muhim cheklovlar:

- Solishtirish **yuzaki (shallow)**: obyekt, massiv va funksiyalar **havola** bo'yicha tekshiriladi
- `memo` faqat **props o'zgarishidan** himoya qiladi. Component'ning **o'z state'i** yoki o'qiyotgan **Context** i o'zgarsa, u baribir render bo'ladi
- Solishtirishning o'zi ham xarajat, shuning uchun arzon component'larga foyda bermaydi

## 6. [Strong junior] `memo` ba'zan nega ishlamaydi?

Props sifatida **har renderda yangi yaratiladigan** obyekt yoki funksiya berilsa, shallow solishtirish "o'zgargan" deb hisoblaydi va `memo` foydasiz bo'lib qoladi.

```jsx
<Archive
  archiveOptions={{ show: false, title: "Post archive" }}   // har renderda yangi obyekt
  onAddPost={handleAddPost}                                   // har renderda yangi funksiya
/>
```

Yechim: obyektni `useMemo` bilan, funksiyani `useCallback` bilan **memoizatsiya qilish**, shunda havolalar barqaror bo'ladi.

## 7. [Strong junior] `useMemo` va `useCallback` farqi nima?

- **`useMemo(() => qiymat, [deps])`**: hisoblangan **qiymatni** (obyekt, massiv, hisob-kitob natijasi) eslab qoladi
- **`useCallback(fn, [deps])`**: **funksiyani** eslab qoladi (havolasini saqlaydi)

```jsx
const archiveOptions = useMemo(() => ({ show: false, title: `Post archive in addition to ${posts.length} main posts` }), [posts.length]);

const handleAddPost = useCallback(function handleAddPost(post) {
  setPosts((posts) => [post, ...posts]);
}, []);
```

`useCallback(fn, deps)` bu `useMemo(() => fn, deps)` ning qisqa yozuvi. Dependency array o'zgarmaguncha, ikkalasi ham oldingi qiymat/funksiyani qaytaradi.

## 8. [Strong junior] `useMemo` va `useCallback` qachon ishlatiladi? Uch asosiy holat

1. **`memo` bilan birga**: child'ga beriladigan obyekt/funksiyani barqaror qilish (wasted render'larni oldini olish)
2. **Og'ir hisob-kitob natijasini** har renderda qayta hisoblamaslik
3. **Boshqa hook'ning dependency array'ida** ishlatiladigan obyekt/funksiyani barqaror qilish (effect keraksiz qayta ishlamasligi uchun)

Misol (3): WorldWise'da `getCity` `City` dagi `useEffect` ning dependency'si, shuning uchun `useCallback` ga o'ralgan. Aks holda u har renderda yangi funksiya bo'lib, effect va fetch tsikli (loop) hosil qilardi.

## 9. [Strong junior] `useMemo` va `useCallback` har joyda ishlatish yaxshimi?

Yo'q. Ularning ham narxi bor: xotira, dependency'larni solishtirish va kodni murakkablashtirish. Foyda faqat yuqoridagi uch holatda bo'ladi. Faqat `useCallback` ga o'rab qo'yish, agar funksiya `memo` qilingan child'ga yoki dependency array'ga berilmasa, hech qanday foyda bermaydi. Qoida: avval o'lchang, keyin optimizatsiya qiling.

## 10. [Middle] Context re-render'larini qanday optimallaymiz? Qachon optimallash kerak?

Muammo: Provider qayta render bo'lganda `value` yangi obyekt bo'ladi va **barcha** consumer'lar render bo'ladi. `memo` buni to'xtatmaydi (Context o'zgarishi props emas).

Yechim:

```jsx
const value = useMemo(() => ({
  posts: searchedPosts,
  onAddPost: handleAddPost,
  onClearPosts: handleClearPosts,
  searchQuery,
  setSearchQuery,
}), [searchedPosts, searchQuery]);   // funksiyalar useCallback bilan barqaror
```

Yoki Context'ni ikkiga bo'lish (masalan, tez o'zgaradigan va kam o'zgaradigan qismlar).

Context'ni optimallash **faqat uchta shart birga** bajarilganda mantiqli: (1) Context'dagi state **tez-tez o'zgaradi**, (2) Context'ni **ko'p component** o'qiydi, (3) ilova **sezilarli sekin**. Aks holda ortiqcha murakkablik.

## 11. [Strong junior] Code splitting va bundle hajmini kamaytirish qanday qilinadi?

**Bundle** (build natijasi) katta bo'lsa, ilova sekin yuklanadi. **Code splitting** kodni bir necha **chunk**ga bo'ladi va har birini faqat kerak bo'lganda yuklaydi.

```jsx
const Homepage = lazy(() => import("./pages/Homepage"));
const AppLayout = lazy(() => import("./pages/AppLayout"));

<Suspense fallback={<SpinnerFullPage />}>
  <Routes>...</Routes>
</Suspense>
```

- Dinamik `import()` bundler'ga (Vite) shu modulni alohida faylga ajratishni bildiradi
- `lazy` component'ni birinchi marta render bo'lganda yuklaydi, `Suspense` yuklanish davomida `fallback` ko'rsatadi
- `lazy` faqat **default export** bilan ishlaydi
- Odatda **route bo'yicha** bo'linadi (eng sodda va samarali)

Bundle tarkibini ko'rish uchun build natijasini yoki vizualizator plaginlarini (masalan, `rollup-plugin-visualizer`) ishlatish mumkin.

## 12. [Junior] "Erta optimizatsiya qilmang" (Don't optimize prematurely) nima degani?

Optimizatsiya kodni **murakkablashtiradi** (qo'shimcha hook'lar, dependency'lar, xato ehtimoli), ko'p holatda esa sezilarli foyda bermaydi. Tartib:

1. Avval ilovani **to'g'ri va sodda** yozing
2. Sustlik **sezilsa** yoki Profiler muammoni ko'rsatsa, **o'lchang**
3. Aniq muammoli joyni **nishonga olib** optimallang

Qo'shimcha: zamonaviy React ekotizimida **React Compiler** memoizatsiyani avtomatik qila oladi. Shuning uchun qo'lda `memo`/`useMemo`/`useCallback` kam yoziladigan bo'lib bormoqda. Lekin mexanikani tushunish intervyuda hamon muhim.

## 13. [Strong junior] `useEffect` qoidalari va best practice'lar qanday?

- Effect ichida ishlatilgan **har bir reaktiv qiymat** (state, props, ular asosida hosil bo'lgan o'zgaruvchi va funksiya) dependency array'da bo'lishi kerak
- **Obyekt, massiv va funksiyalar** component ichida har renderda yangi yaratiladi, shuning uchun ularni dependency qilsak, effect **har render'da** ishlaydi. Yechimlar: ularni effect **ichiga** ko'chirish, component'dan **tashqariga** chiqarish (reaktiv qiymatga tayanmasa), **memoizatsiya** qilish (`useMemo`/`useCallback`) yoki **primitiv qiymatlarni** (`id`, `length`) dependency qilish
- Dependency'ni "xalaqit beryapti" deb **o'chirib qo'ymang** (ESLint ogohlantirishi odatda haqiqiy xatoni bildiradi)
- Effect faqat **sinxronlash** uchun (API, DOM, timer). Hisoblanadigan qiymat va foydalanuvchi harakatiga javob uchun effect kerak emas
- Subscription, timer va listener uchun **cleanup** yozing

## 14. [Strong junior] State'ni boshqa state'ga asoslab yangilashda dependency'dan qanday qutulamiz?

Yangilash funksiyasi (updater) ishlatsak, effect yoki callback ichida state qiymatining o'ziga bog'lanish kerak bo'lmaydi:

```jsx
// Yomon: duration dependency, har soniyada interval qayta yaratiladi
useEffect(() => {
  const id = setInterval(() => setDuration(duration - 1), 1000);
  return () => clearInterval(id);
}, [duration]);

// Yaxshi: updater funksiya, duration dependency kerak emas
useEffect(() => {
  const id = setInterval(() => setDuration((d) => d - 1), 1000);
  return () => clearInterval(id);
}, []);
```

Bir necha bog'liq state bo'lsa, ularni `useReducer` ga olish yoki hisoblanadigan qiymatni **derived state** sifatida render ichida topish ham yo'llardan biri.

## 15. [Strong junior] Effect ichida yordamchi (helper) funksiyani ishlatishda qanday variantlar bor?

Effect ichida chaqiriladigan funksiya reaktiv qiymatlarga tayansa, u ham dependency hisoblanadi. Variantlar:

1. Funksiyani **effect ichida** e'lon qilish (agar boshqa joyda kerak bo'lmasa)
2. Funksiyani component'dan **tashqariga** chiqarish (agar state/props'ga tayanmasa)
3. Funksiyani **`useCallback`** ga o'rab, dependency array'ga qo'shish
4. Kerakli qiymatlarni funksiyaga **argument sifatida** berish, shunda u component'ning reaktiv qiymatlariga tayanmaydi

Eng soddasi odatda 1 yoki 2.

## 16. [Strong junior] Effect'dagi closure nima? "Stale closure" xatosi qanday ko'rinadi?

Effect funksiyasi va uning cleanup'i **yaratilgan renderdagi** o'zgaruvchilarni "eslab qoladi" (JavaScript closure). Dependency array to'liq bo'lmasa, effect **eski (stale)** qiymat bilan ishlab qoladi:

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  const id = setInterval(() => {
    console.log(count);        // doim 0 chiqadi, chunki effect faqat birinchi renderdagi count'ni ko'radi
  }, 1000);
  return () => clearInterval(id);
}, []);                        // count dependency'da yo'q
```

Yechimlar: to'g'ri dependency'lar (`[count]`), state yangilashda **updater funksiya** (`setCount((c) => c + 1)`), yoki eng so'nggi qiymatni **`useRef`** da saqlash. Shuning uchun ESLint'ning `exhaustive-deps` qoidasiga amal qilish muhim.

## 17. [Strong junior] Workout Timer misolida qanday performance muammolari paydo bo'lishi mumkin va qanday tuzatiladi?

Tipik belgilar va yechimlar:

- **Effect'lar keraksiz qayta ishlaydi**: sabab dependency'dagi har renderda yangi yaratiladigan obyekt/funksiya. Yechim: `useMemo`/`useCallback` yoki ularni effect ichiga/tashqariga ko'chirish
- **Interval qayta-qayta yaratiladi**: sabab effect state qiymatiga bog'liq. Yechim: updater funksiya (`setX((x) => x - 1)`)
- **Og'ir hisob-kitob har renderda qaytadan bajariladi**: yechim `useMemo`
- **Child'lar keraksiz render bo'ladi**: yechim `memo` va barqaror props

Birinchi qadam doim **Profiler** bilan muammoni topish. Kursdagi aniq yechim tafsilotlari boshqacha bo'lishi mumkin, tamoyillar esa shu.
