# Section 7: Thinking In React: State Management

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Projects in this section: "Far Away" travel list (davomi), Accordion, Tip Calculator.

---

## 1. [Junior] "Thinking in React" nima? Ilova qurishning qadamlari qanday?

Bu React ilovasini rejalashtirish usuli:

1. UI'ni **component'larga bo'lib**, component daraxtini chizish
2. Avval **statik versiyani** yozish (state'siz, faqat props bilan)
3. **State haqida o'ylash**: qanday ma'lumot kerak, qaysi biri o'zgaradi, qayerda turishi kerak
4. **Ma'lumot oqimini** (data flow) belgilash: props pastga, funksiyalar yuqoriga

Kod yozishdan oldin shu bosqichlardan o'tish keyinchalik qayta yozishdan saqlaydi.

## 2. [Junior] Ma'lumotni state'ga qo'yish kerakmi yoki yo'qmi, qanday hal qilamiz?

- Ma'lumot **o'zgaradimi**? Yo'q bo'lsa, oddiy o'zgaruvchi yoki props yetarli
- U **mavjud state/props'dan hisoblab olinadimi**? Ha bo'lsa, state qilmaymiz (derived state)
- Yuqoridagilar yo'q bo'lsa va ma'lumot UI'ga ta'sir qilsa, **state'ga** qo'yamiz

Faqat renderga ta'sir qilmaydigan o'zgaruvchi uchun (masalan, timer id) state emas, `useRef` ishlatiladi.

## 3. [Junior] Local state va global state farqi nima?

- **Local state**: bitta component (yoki uning bir-ikki child'i) ishlatadi
- **Global state**: ilovaning ko'p joyidagi component'lar ishlatadi (masalan, foydalanuvchi, tema, savat)

Doim local'dan boshlanadi. Faqat ko'p joyda kerak bo'lib qolsa, global'ga (Context yoki kutubxona) ko'tariladi.

## 4. [Strong junior] "Lifting state up" nima va qachon kerak?

Ikki **qo'shni (sibling)** component bir xil state'ga muhtoj bo'lsa, o'sha state ularning **eng yaqin umumiy parent'iga** ko'tariladi va props orqali pastga beriladi.

```jsx
// App ichida: ikkala child ham items ni ishlatadi
const [items, setItems] = useState([]);

<Form onAddItem={handleAddItem} />
<PackingList items={items} />
```

Siblinglar bir-biriga to'g'ridan-to'g'ri ma'lumot bera olmaydi, shuning uchun state parent'da turadi.

## 5. [Strong junior] Child parent'ga qanday ma'lumot yuboradi? ("Child-to-parent communication")

Props faqat pastga oqadi. Child parent'ning state'ini o'zgartirishi kerak bo'lsa, parent **funksiyani props sifatida** beradi, child esa uni kerak payti chaqiradi ("props pastga, event'lar yuqoriga").

```jsx
// Parent
function handleDeleteItem(id) {
  setItems((items) => items.filter((item) => item.id !== id));
}
<PackingList items={items} onDeleteItem={handleDeleteItem} />

// Child
<button onClick={() => onDeleteItem(item.id)}>❌</button>
```

## 6. [Strong junior] Ro'yxatdan elementni qanday o'chiramiz va yangilaymiz? (Immutable)

State'ni mutatsiya qilmaymiz, yangi massiv yasaymiz:

```jsx
// o'chirish
setItems((items) => items.filter((item) => item.id !== id));

// bitta maydonni yangilash (masalan, packed)
setItems((items) =>
  items.map((item) => item.id === id ? { ...item, packed: !item.packed } : item)
);
```

Element `id` orqali topiladi, indeks bilan emas (ro'yxat o'zgarsa indeks adashtiradi).

## 7. [Junior] Derived state nima? Nega alohida `useState` qilmaymiz?

Mavjud state yoki props'dan **har renderda hisoblab olinadigan** qiymat:

```jsx
const numItems = items.length;
const numPacked = items.filter((item) => item.packed).length;
const percentage = Math.round((numPacked / numItems) * 100);
```

Agar uni alohida state qilsak, ikkita ma'lumotni bir-biriga mos saqlash kerak bo'ladi, bu xato va ortiqcha render keltiradi. Hisoblangan qiymat esa doim joriy state bilan mos.

## 8. [Strong junior] Saralash (sorting) qanday qilinadi? State nimani saqlaydi?

State'da faqat **tanlov** (masalan, `sortBy = "input" | "description" | "packed"`) saqlanadi, saralangan ro'yxatning o'zi esa **derived** bo'ladi:

```jsx
let sortedItems;
if (sortBy === "input") sortedItems = items;
if (sortBy === "description")
  sortedItems = items.slice().sort((a, b) => a.description.localeCompare(b.description));
```

Muhim: `.sort()` massivni **o'zgartiradi** (mutate), shuning uchun avval `.slice()` yoki `[...items]` bilan nusxa olinadi, aks holda state buziladi. (Yangi muhitlarda `toSorted()` ham bor.)

## 9. [Junior] Component'larni alohida fayllarga nega ko'chiramiz? `export default` va named export farqi?

Bir faylda bitta component bo'lsa, kod o'qilishi, topilishi va qayta ishlatilishi osonlashadi. 

- **Default export**: faylda bitta asosiy narsa, import paytida istalgan nom berish mumkin (`import Logo from "./Logo"`)
- **Named export**: nomi aniq, bitta faylda bir nechta chiqarish mumkin (`import { Logo } from "./Logo"`), tahrirlashda xatoga kam yo'l qo'yadi

## 10. [Junior] `children` prop nima?

Component'ning ochuvchi va yopuvchi tegi **orasiga** yozilgan hamma narsa unga `children` nomli prop sifatida keladi.

```jsx
function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

<Button onClick={handleNext}>Next →</Button>
```

Shunda bitta Button turli matn va ikonkalar bilan qayta ishlatiladi.

## 11. [Strong junior] Component composition nima? `children` bilan nima foyda?

Katta component'ni kichik, o'zaro birlashadigan bo'laklardan yig'ish. `children` yordamida component **ichiga nima joylashishini bilmasdan** layout/shakl yasaydi (masalan, Accordion'ning content qismi). Bu component'ni universal qiladi va ko'p props berishning o'rniga JSX'ni to'g'ridan-to'g'ri o'tkazish imkonini beradi.

## 12. [Strong junior] Accordion'da bir vaqtda faqat bitta element ochiq bo'lishi kerak. State qayerda turishi kerak?

Agar har bir `AccordionItem` o'zining `isOpen` state'iga ega bo'lsa, u boshqalarni yopa olmaydi. Shuning uchun state **parent'ga ko'tariladi**: parent `curOpen` (ochiq elementning id'si) ni saqlaydi, har bir item `isOpen = curOpen === num` ni props orqali oladi va bosilganda parent'ning funksiyasini chaqiradi.

```jsx
const [curOpen, setCurOpen] = useState(null);

<AccordionItem
  isOpen={curOpen === i}
  onOpen={setCurOpen}
  num={i}
/>
```

Bu "lifting state up" ning amaliy misoli.

## 13. [Strong junior] Tip Calculator'da qaysi ma'lumot state, qaysi biri derived?

State: `bill`, `myTip`, `friendTip` (foydalanuvchi kiritadi). Derived: `tip` va `total`, ular state'dan hisoblanadi:

```jsx
const tip = bill * ((myTip + friendTip) / 2 / 100);
const total = bill + tip;
```

Eslatma: input'dan kelgan qiymat har doim **string**, shuning uchun hisoblashdan oldin `Number(e.target.value)` bilan songa aylantiriladi, aks holda `"10" + "5"` = `"105"` bo'lib qoladi.

## 14. [Middle] Prop drilling nima va uni qanday kamaytiramiz?

Ma'lumotni faqat pastdagi component ishlatadi, lekin u oraliq component'lar orqali bir necha qatlam props sifatida uzatiladi (oraliqlar uni o'zlari ishlatmaydi). Kod bog'liq va o'zgartirish qiyin bo'ladi. Yechimlar:

- **Component composition / `children`**: tayyor JSX'ni pastga bevosita berish, oraliq component'lar ma'lumotni bilmaydi
- **Context API** (keyingi section'larda)
- **Global state kutubxonasi** (Redux, Zustand va h.k.)

Birinchi qadam sifatida odatda composition, keyin kerak bo'lsa Context tanlanadi.

## 15. [Middle] State'ni qayerga qo'yish kerak? Doim yuqoriga ko'tarish yaxshimi?

Yo'q. Qoida: state'ni uni ishlatadigan component'larga **iloji boricha yaqin** saqlash (state colocation), faqat ikki yoki undan ko'p component'ga kerak bo'lsa **eng yaqin umumiy parent'ga** ko'tarish. Keraksiz baland joyga qo'yilgan state har o'zgarganda butun katta daraxtni qayta render qiladi va kodni tushunishni qiyinlashtiradi.
