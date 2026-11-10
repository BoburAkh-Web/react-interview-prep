# Section 8: [Optional] Practice Project: Eat-'N-Split

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: a bill-splitting app. The user keeps a list of friends, selects one, and splits a bill. The balance shows who owes whom.

Component tree:

```
App
├── FriendsList
│   └── Friend (× n)
├── FormAddFriend
├── Button
└── FormSplitBill
```

---

## 1. [Junior] Bu ilovada state qayerda turadi va nega aynan shu yerda?

`App` component'i uchta state'ni saqlaydi:

```jsx
const [showAddFriend, setShowAddFriend] = useState(false);
const [friends, setFriends] = useState(initialFriends);
const [selectedFriend, setSelectedFriend] = useState(null);
```

Sabab: do'stlar ro'yxatini `FriendsList` ham, balansni yangilaydigan `FormSplitBill` ham ishlatadi. Tanlangan do'stni esa `Friend` (ajratib ko'rsatish uchun) va `FormSplitBill` (kim bilan bo'lish uchun) bilishi kerak. Ikkala tomonga kerak bo'lgan state **eng yaqin umumiy parent'ga ko'tariladi** (lifting state up).

## 2. [Junior] Qayta ishlatiladigan `Button` component'i qanday yoziladi?

```jsx
function Button({ children, onClick }) {
  return (
    <button className="button" onClick={onClick}>
      {children}
    </button>
  );
}
```

Matn `children` orqali keladi, shuning uchun bir xil stilli tugma turli joyda ("Add friend", "Select", "Split Bill") ishlatiladi.

## 3. [Junior] Balansga qarab turli xabar chiqarish (shartli render) qanday qilinadi?

```jsx
{friend.balance < 0 && (
  <p className="red">You owe {friend.name} {Math.abs(friend.balance)}$</p>
)}
{friend.balance > 0 && (
  <p className="green">{friend.name} owes you {Math.abs(friend.balance)}$</p>
)}
{friend.balance === 0 && <p>You and {friend.name} are even</p>}
```

Uch holat bor, shuning uchun uchta `&&` ishlatiladi. `Math.abs` manfiy sonni musbat qilib ko'rsatadi, chunki xabar matnida yo'nalish (owe) allaqachon aytilgan.

## 4. [Junior] Yangi do'stni ro'yxatga qanday qo'shamiz? Child parent'ga ma'lumotni qanday uzatadi?

Parent funksiyani props orqali beradi, child (forma) uni yangi do'st bilan chaqiradi:

```jsx
// App
function handleAddFriend(friend) {
  setFriends((friends) => [...friends, friend]);
  setShowAddFriend(false);
}
<FormAddFriend onAddFriend={handleAddFriend} />

// FormAddFriend
function handleSubmit(e) {
  e.preventDefault();
  if (!name || !image) return;
  const id = crypto.randomUUID();
  onAddFriend({ id, name, image: `${image}?=${id}`, balance: 0 });
  setName("");
}
```

`id` ni handler ichida yaratgan ma'qul: component tanasida yozilsa, har renderda keraksiz yangi id yaratiladi.

## 5. [Strong junior] Nega `e.preventDefault()` validatsiya `return` idan **oldin** yozilishi kerak?

```jsx
// XATO
if (!name || !image) return;
e.preventDefault();

// TO'G'RI
e.preventDefault();
if (!name || !image) return;
```

Birinchi variantda validatsiya o'tmasa, funksiya `preventDefault` ga yetmay chiqib ketadi va brauzer sahifani qayta yuklab yuboradi (state yo'qoladi). `preventDefault` doim birinchi qatorda bo'lishi kerak.

## 6. [Strong junior] Tanlangan do'stni qanday saqlaymiz? Boshlang'ich qiymat `[]` bo'lsa nima xato chiqadi?

Bitta tanlangan obyektni saqlaymiz, boshlang'ich qiymati `null`:

```jsx
const [selectedFriend, setSelectedFriend] = useState(null);

function handleSelection(friend) {
  setSelectedFriend((selected) => (selected?.id === friend.id ? null : friend));
  setShowAddFriend(false);
}

{selectedFriend && <FormSplitBill selectedFriend={selectedFriend} />}
```

Agar boshlang'ich qiymat `[]` bo'lsa, **bo'sh massiv ham truthy** hisoblanadi, shuning uchun `selectedFriend && ...` shartida forma hech kim tanlanmagan bo'lsa ham ko'rinib qoladi ("Split a bill with undefined"). State'ning turi doim bir xil bo'lishi kerak: obyekt yoki `null`, massiv/string/obyektni aralashtirib bo'lmaydi.

## 7. [Strong junior] `<FormSplitBill key={selectedFriend.id} />` dagi `key` nima qiladi?

`key` o'zgarganda React eski component'ni olib tashlab, **yangisini noldan** yaratadi, ya'ni uning ichki state'i (`bill`, `paidByUser`) tozalanadi. Boshqa do'st tanlanganda forma ichidagi eski summalar qolib ketmasligi uchun shu usul ishlatiladi. Bu `useEffect` bilan state'ni qo'lda tozalashdan ancha sodda.

## 8. [Strong junior] `paidByFriend` nega state emas?

```jsx
const [bill, setBill] = useState("");
const [paidByUser, setPaidByUser] = useState("");
const paidByFriend = bill ? bill - paidByUser : "";
```

Do'stning ulushi `bill` va `paidByUser` dan **hisoblab olinadi** (derived state). Uni alohida state qilsak, uchta qiymatni sinxron saqlashga to'g'ri kelardi va xato ehtimoli oshardi.

## 9. [Strong junior] Foydalanuvchi o'z ulushini hisobdan ko'p kirita olmasligi uchun qanday cheklov qo'yamiz? Input'dagi `Number()` bilan qanday muammo bor?

```jsx
onChange={(e) =>
  setPaidByUser(
    Number(e.target.value) > bill ? paidByUser : Number(e.target.value)
  )
}
```

Katta qiymat kiritilsa, state eski qiymatida qoladi. Muammo: input `type="text"` bo'lsa, harf kiritilganda `Number("abc")` = `NaN` bo'ladi. Yechim: `type="number"` ishlatish yoki `NaN` ni tekshirish. Input'dan keladigan qiymat doim **string** ekanini unutmaslik kerak.

## 10. [Strong junior] Balansni yangilash mantig'i qanday? Nega `whoIsPaying === "user" ? paidByFriend : -paidByUser`?

Balans ishorasi: **musbat** bo'lsa do'st sizga qarz, **manfiy** bo'lsa siz do'stga qarz.

- Siz to'lasangiz: do'st sizga o'z ulushini qarz bo'ladi, `+paidByFriend`
- Do'st to'lasa: siz unga o'z ulushingizni qarz bo'lasiz, `-paidByUser`

```jsx
function handleSplitBill(value) {
  setFriends((friends) =>
    friends.map((friend) =>
      friend.id === selectedFriend.id
        ? { ...friend, balance: friend.balance + value }
        : friend
    )
  );
  setSelectedFriend(null);
}
```

Ro'yxat **immutable** yangilanadi: `map` yangi massiv qaytaradi, o'zgargan do'st esa spread bilan yangi obyektga aylanadi. Keyin tanlov bekor qilinadi.

## 11. [Junior] Ochiq/yopiq tugmasini almashtirishda `setShow(!show)` o'rniga nima yaxshiroq?

```jsx
<Button onClick={() => setShowAddFriend((show) => !show)}>
```

Callback shakli eng so'nggi state'ga tayanadi, shuning uchun bir nechta ketma-ket yangilashda ham to'g'ri ishlaydi (Section 6 dagi "state'ni hozirgi state asosida yangilash" qoidasi).

## 12. [Junior] Forma va ro'yxatda qanday semantik HTML va accessibility qoidalariga amal qilish kerak?

- Ro'yxat `<ul>` ichida `<li>` bilan yoziladi (`<div>` ichida emas)
- `<label>` ni input bilan bog'lash: `<label htmlFor="friend-name">` va `<input id="friend-name" />`. Shunda label'ni bosganda input fokus oladi, screen reader'lar ham tushunadi
- Tugmalarda aniq matn bo'lishi kerak

## 13. [Middle] Bu ilovani `useReducer` ga ko'chirsangiz, qanday action'lar bo'ladi? Qachon bunday qilish kerak?

Action'lar:

- `friend/added` (payload: yangi do'st)
- `friend/selected` (payload: do'st yoki `null`)
- `bill/split` (payload: qiymat)
- `addForm/toggled`

`useReducer` state bir nechta bo'lib, ular **bir-biriga bog'liq** o'zgarganda (masalan, `handleSplitBill` ham balansni, ham tanlovni yangilaydi) qulay: mantiq bitta reducer'da to'planadi va test qilish oson. Bu ilova kichik, shuning uchun `useState` yetarli, `useReducer` esa state yirikroq bo'lganda yoki Context bilan birga ishlatilganda foydali.

## 14. [Middle] Bu kodni ishlab chiqarish (production) darajasiga ko'tarish uchun nimalarni yaxshilaysiz?

- Component'larni alohida fayllarga ajratish va `Friend`/`FriendsList` ni `ul > li` ga o'tkazish
- TypeScript bilan `Friend` tipini aniqlash (`id`, `name`, `image`, `balance`)
- Pul bilan ishlashda float xatolarini (masalan, `0.1 + 0.2`) oldini olish: summalarni tiyin (butun son) ko'rinishida saqlash
- Ma'lumotni `localStorage` yoki backend'da saqlash (hozir sahifa yangilansa hammasi yo'qoladi)
- Nom va typo'larni tuzatish (`isSeleceted`, `whoIsPaing` kabi), o'qilishi osonligi uchun
- Validatsiya xabarlarini ko'rsatish (hozir noto'g'ri kiritilsa, jim qaytadi)
