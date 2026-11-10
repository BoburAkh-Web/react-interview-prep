# Section 10: Thinking in React: Components, Composition, and Reusability

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Projects in this section: usePopcorn (movie app), reusable Star Rating, Text Expander.

---

## 1. [Junior] UI'ni component'larga bo'lishda nimalarga qaraymiz?

Bir nechta mezon bor:

- **Mantiqiy bo'linish**: kontent va layout alohida ma'noli bo'laklarga bo'linadimi
- **Qayta ishlatish (reusability)**: shu bo'lak boshqa joyda ham kerak bo'ladimi
- **Mas'uliyat va murakkablik**: component juda ko'p narsa qilyaptimi
- **Shaxsiy kod yozish uslubi**: jamoa kelishuvi

Juda **kichik** component'lar ko'payib ketsa, kodda adashish va ortiqcha props uzatish paydo bo'ladi. Juda **katta** component'larni esa saqlash, qayta ishlatish va tushunish qiyin. Amaliy qoida: nomi aniq bo'lmagan yoki "va" bilan ta'riflanadigan component bo'linishi kerak. Odatda kattaroq boshlab, kerak bo'lganda ajratiladi.

## 2. [Junior] Component kategoriyalari qanday?

- **Stateless / presentational**: faqat props qabul qilib JSX qaytaradi, state'i yo'q (masalan, `Logo`, `Movie`, `NumResults`)
- **Stateful**: o'z state'iga ega (masalan, `Search`)
- **Structural**: ilova tuzilishini quradi: sahifalar, layout'lar, ekranlar (masalan, `App`, `NavBar`, `Main`, `Box`)

Bu qat'iy qoida emas, lekin component'ni qaysi turga kiritishni bilish uni to'g'ri joyga qo'yishga yordam beradi.

## 3. [Junior] Prop drilling nima va nega muammo?

Ma'lumotni faqat chuqur joylashgan component ishlatadi, lekin u oraliq component'lar orqali bir necha qatlam props sifatida uzatiladi. Oraliq component'lar ma'lumotni o'zlari ishlatmaydi. Natijada kod bir-biriga bog'lanib qoladi va o'zgartirish qiyinlashadi.

## 4. [Strong junior] Composition bilan prop drilling'ni qanday hal qilamiz?

Chuqur component'ni parent'ning o'zida yaratib, oraliq component'ga **`children`** orqali beramiz. Oraliq component ma'lumotni bilmaydi.

```jsx
// Avval: NavBar movies ni faqat NumResults uchun uzatardi
<NavBar movies={movies} />

// Keyin: NavBar faqat o'rab turadi
<NavBar>
  <Search />
  <NumResults movies={movies} />
</NavBar>
```

`movies` endi to'g'ridan-to'g'ri ishlatiladigan joyga beriladi, `NavBar` esa ma'lumotga bog'liq emas va qayta ishlatiladigan layout bo'lib qoladi.

## 5. [Strong junior] Component composition nima? Nega React'da inheritance emas, composition ishlatiladi?

Composition: katta component'ni kichik, bir-biriga mos component'larni **birlashtirib** yig'ish (`children` yoki boshqa props orqali). React jamoasi inheritance (merosxo'rlik) o'rniga composition'ni tavsiya qiladi, chunki u moslashuvchan: istalgan ichki mazmunni o'rab turuvchi component'ga bera olasiz, class zanjirlari esa qattiq bog'liq bo'lib qoladi.

## 6. [Junior] Qayta ishlatiladigan `Box` component'i qanday yoziladi?

```jsx
function Box({ children }) {
  const [isOpen, setIsOpen] = useState(true);

  return (
    <div className="box">
      <button className="btn-toggle" onClick={() => setIsOpen((open) => !open)}>
        {isOpen ? "-" : "+"}
      </button>
      {isOpen && children}
    </div>
  );
}

<Box><MovieList movies={movies} /></Box>
<Box><WatchedSummary watched={watched} /></Box>
```

Ochish/yopish logikasi bir joyda, ichidagi mazmun esa har joyda har xil. Ikki "quti" (ro'yxat va ko'rilganlar) bitta component'dan foydalanadi.

## 7. [Strong junior] `children` o'rniga elementni oddiy prop sifatida berish (`element` prop) qachon kerak?

```jsx
<Box element={<MovieList movies={movies} />} />

function Box({ element }) {
  const [isOpen, setIsOpen] = useState(true);
  return <div className="box">...{isOpen && element}</div>;
}
```

Natija `children` bilan bir xil. `children` bitta asosiy mazmun uchun qulay va odatiy hisoblanadi. Agar component'da **bir nechta joy** bo'lsa (masalan, `header`, `footer`, `sidebar`), har birini alohida nomli prop sifatida berish aniqroq. Bu usul React Router'da ham uchraydi (`element={<Page />}`).

## 8. [Strong junior] Qayta ishlatiladigan Star Rating component'i qanday ishlaydi? Nega ikkita state kerak?

Ikkita state:

```jsx
const [rating, setRating] = useState(0);        // tanlangan (bosilgan) baho
const [tempRating, setTempRating] = useState(0); // sichqoncha ustidagi vaqtinchalik baho
```

Hover paytida foydalanuvchi bahoni **ko'rib** turadi, lekin hali tanlamagan. Agar bitta state bo'lsa, hover bosilgan bahoni buzib yuborardi. Yulduz to'la ko'rinishi:

```jsx
full={tempRating ? tempRating >= i + 1 : rating >= i + 1}
```

Ya'ni hover bor bo'lsa, `tempRating` ga, bo'lmasa `rating` ga qaraydi.

## 9. [Strong junior] Hover event'larida `onMouseEnter` va `onMouseOver` farqi nima?

`onMouseOver` ichki (child) elementlarga o'tganda ham qayta ishga tushadi, chunki event **bubbling** qiladi. `onMouseEnter` esa faqat element chegarasiga kirganda bir marta ishlaydi va bubble qilmaydi. Yulduz kabi ichki elementi bor (masalan, SVG) tugmalarda `onMouseEnter` / `onMouseLeave` aniqroq va keraksiz chaqiruvlarsiz ishlaydi.

## 10. [Strong junior] "Props as a component API" nimani anglatadi? Qayta ishlatiladigan component'ning props'ini qanday loyihalaymiz?

Component'ning props'i uning **ommaviy interfeysi (API)**. Yaxshi API:

- **Oqilona default qiymatlar**: `maxRating = 5`, `color = "#fcc419"`, `size = 48`
- Hardcode qilmaslik: rang, o'lcham, xabarlar props orqali sozlanadi
- `className` va `style` ni qabul qilish: tashqaridan moslashtirish uchun
- Natijani parent'ga **callback** bilan yetkazish: `onSetRating(rating)`
- Boshlang'ich qiymat: `defaultRating`

Props ko'paytirishda ehtiyot bo'lish kerak: ko'p props (10+) component'ni tushunish va ishlatishni qiyinlashtiradi.

## 11. [Junior] PropTypes nima va nega kerak? Cheklovlari qanday?

PropTypes props'lar turini **runtime'da (development'da)** tekshiradi va noto'g'ri tur berilsa, konsolda ogohlantirish chiqaradi.

```jsx
StarRating.propTypes = {
  maxRating: PropTypes.number,
  color: PropTypes.string,
  onSetRating: PropTypes.func,
};
```

Cheklovlari: tekshiruv faqat kod ishga tushganda va faqat konsol ogohlantirishi beradi, kompilyatsiya paytida xatoni ushlamaydi. Zamonaviy loyihalarda uning o'rniga **TypeScript** ishlatiladi. Bundan tashqari, React 19 da component'lar uchun propTypes tekshiruvi ishlamaydi, shuning uchun yangi loyihalarda TypeScript tavsiya etiladi.

## 12. [Strong junior] Text Expander'da qaysi ma'lumot state, qaysi biri derived?

Faqat bitta state kerak: `isExpanded`. Ko'rsatiladigan matn **derived**:

```jsx
const [isExpanded, setIsExpanded] = useState(expanded);

const displayText = isExpanded
  ? children
  : children.split(" ").slice(0, collapsedNumWords).join(" ") + "...";
```

Props orqali sozlash mumkin bo'lgan qismlar: `collapsedNumWords`, `expandButtonText`, `collapseButtonText`, `buttonColor`, `className`. Bu "props as API" tamoyilining amaliy misoli.

## 13. [Middle] Qayta ishlatiladigan component'ni ortiqcha murakkablashtirmasdan qanday loyihalaysiz? Controlled va uncontrolled variant nima?

Props'ni **kerak bo'lgandagina** qo'shish kerak. Har holat uchun alohida prop qo'shilaversa, component tushunib bo'lmas "konfiguratsiya"ga aylanadi. Bunday paytda component'ni bo'lib tashlash yoki composition ishlatish yaxshiroq.

Star Rating misolida ikki variant:

- **Uncontrolled**: component bahoni o'z ichidagi state'da saqlaydi, `defaultRating` bilan boshlanadi, parent'ga faqat `onSetRating` orqali xabar beradi
- **Controlled**: baho parent'ning state'ida turadi, component `rating` va `onChange` ni props sifatida oladi (parent to'liq nazoratda)

Oddiy holat uchun uncontrolled qulay, parent bahoni boshqa joyda ham ishlatishi yoki tashqaridan o'zgartirishi kerak bo'lsa, controlled variant tanlanadi.

## 14. [Middle] Composition qachon yetarli emas? Qachon Context yoki global state kerak?

Composition oraliq component'lar faqat **layout** vazifasini bajarganda yaxshi ishlaydi: ma'lumotni yuqoridagi component yaratib, pastga `children` orqali beradi. Lekin ma'lumot ilovaning **uzoq-uzoq joylarida** (bir-biriga bog'liq bo'lmagan shoxlarda) kerak bo'lsa, yoki juda ko'p joyda ishlatilsa (foydalanuvchi, tema, til), composition kodni chalkashtirib yuboradi. Shunda Context API yoki global state kutubxonasi tanlanadi. Tartib: avval composition, yetmasa Context, murakkab bo'lsa maxsus kutubxona.
