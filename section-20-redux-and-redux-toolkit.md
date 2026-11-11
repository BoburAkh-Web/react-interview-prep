# Section 20: Redux and Modern Redux Toolkit (With Thunks)

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: Bank Account (`account` va `customer` slice'lari): avval "klassik" Redux, keyin Redux Toolkit (RTK).

> Section 16 dagi `useReducer` bilan solishtiring: tushunchalar (reducer, action, dispatch) bir xil, Redux esa ularni **butun ilova uchun global store**ga chiqaradi.

---

## 1. [Junior] Redux nima va qanday muammoni hal qiladi?

Redux bu **global state'ni boshqarish** uchun uchinchi tomon kutubxona. Barcha global state bitta **store**da saqlanadi, istalgan component uni o'qiy oladi va **action dispatch qilish** orqali yangilaydi. Bu `useReducer` + Context'ga juda o'xshash g'oya, lekin qo'shimcha imkoniyatlar bilan (middleware, DevTools, selector'lar).

Hal qiladigan muammo: ko'p va bir-biridan uzoq component'lar ishlatadigan, tez-tez yangilanadigan **global UI state**ni bir joyda, bashorat qilinadigan tarzda boshqarish.

## 2. [Junior] Redux ma'lumot oqimi (data flow) qanday?

```
 Event (tugma bosildi)
        │
        ▼
 dispatch(action) ──► Store ──► reducer(state, action) ──► yangi state
                                                              │
                                                              ▼
                                         useSelector bilan o'qiyotgan component'lar qayta render
```

1. Component **action** yaratadi (odatda **action creator** funksiya orqali)
2. Action **dispatch** qilinadi va **store**ga boradi
3. Store tegishli **reducer**ni chaqiradi, u **yangi state** qaytaradi
4. Shu state qismini o'qiyotgan component'lar **qayta render** bo'ladi

Bir yo'nalishli (one-way) oqim xatoni topishni osonlashtiradi: state faqat action orqali va faqat reducer'da o'zgaradi.

## 3. [Junior] Action, action creator va reducer nima?

- **Action**: nima sodir bo'lganini tasvirlovchi oddiy obyekt: `{ type: "account/deposit", payload: 500 }`. Type'larni `"domen/hodisa"` ko'rinishida nomlash odat
- **Action creator**: action obyektini qaytaradigan funksiya. Type'larni bir joyda saqlaydi va typo'larni kamaytiradi
- **Reducer**: `(state, action) => yangiState` toza funksiya

```js
function deposit(amount) {
  return { type: "account/deposit", payload: amount };
}

store.dispatch(deposit(500));
```

## 4. [Junior] Klassik Redux'da reducer va store qanday yoziladi?

```js
const initialStateAccount = { balance: 0, loan: 0, loanPurpose: "", isLoading: false };

function accountReducer(state = initialStateAccount, action) {
  switch (action.type) {
    case "account/deposit":
      return { ...state, balance: state.balance + action.payload };
    case "account/withdraw":
      return { ...state, balance: state.balance - action.payload };
    default:
      return state;            // useReducer'dan farqli: xato emas, state qaytariladi
  }
}

import { createStore, combineReducers } from "redux";
const rootReducer = combineReducers({ account: accountReducer, customer: customerReducer });
const store = createStore(rootReducer);
```

`useReducer` dan farqlar:

- `initialState` reducer'ning **default parametri** sifatida beriladi
- `default` holatda **xato tashlanmaydi**, `state` qaytariladi, chunki Redux har bir action'ni **barcha reducer'larga** yuboradi va har bir reducer o'ziga tegishli bo'lmagan action'larni e'tiborsiz qoldirishi kerak
- `combineReducers` bir nechta reducer'ni bitta **root reducer**ga birlashtiradi, natijada state `{ account: {...}, customer: {...} }` ko'rinishida bo'ladi

`createStore` hozir **eskirgan (deprecated)** deb belgilangan, yangi kodda RTK'ning `configureStore` ishlatiladi (11-savol).

## 5. [Strong junior] "State slice" va professional Redux fayl tuzilmasi qanday?

Har bir global state **domeni** (account, customer) alohida **slice**: o'z papkasi va faylida, reducer'i, initial state'i va action creator'lari bilan.

```
src/
├── store.js                       # store yaratish
└── features/
    ├── accounts/
    │   ├── accountSlice.js        # reducer + action creators
    │   └── AccountOperations.jsx  # shu domen component'lari
    └── customers/
        ├── customerSlice.js
        └── CreateCustomer.jsx
```

Foyda: mantiq **domen bo'yicha** guruhlanadi, kodni topish va kengaytirish oson. Slice fayli **reducer'ni default export**, **action creator'larni named export** qiladi.

## 6. [Junior] Redux'ni React bilan qanday ulaymiz?

`react-redux` kutubxonasi orqali:

```jsx
// main.jsx
import { Provider } from "react-redux";
import store from "./store";

<Provider store={store}>
  <App />
</Provider>
```

```jsx
// state o'qish
const customer = useSelector((store) => store.customer.fullName);

// action yuborish
const dispatch = useDispatch();
dispatch(deposit(500));
```

- **`Provider`**: store'ni butun ilovaga beradi (Context API'ga o'xshash)
- **`useSelector`**: store'dan kerakli qismni oladi va **faqat shu qism o'zgarganda** component'ni qayta render qiladi
- **`useDispatch`**: `dispatch` funksiyasini qaytaradi

## 7. [Strong junior] `useSelector` qanday ishlaydi? Uni qanday to'g'ri ishlatish kerak?

`useSelector(selectorFn)` har store yangilanganda selector'ni chaqiradi va natijani oldingisi bilan **qat'iy tenglik (`===`)** orqali solishtiradi. Natija o'zgarmagan bo'lsa, component render bo'lmaydi. Bu Context'dan katta farq: Context'da qiymat o'zgarsa, **barcha** consumer'lar render bo'ladi.

To'g'ri ishlatish:

```jsx
// Yaxshi: faqat kerakli primitiv qiymatni olish
const balance = useSelector((store) => store.account.balance);

// Yomon: har safar yangi obyekt qaytaradi → har store yangilanishida qayta render
const data = useSelector((store) => ({ balance: store.account.balance, loan: store.account.loan }));
```

Bir nechta qiymat kerak bo'lsa: bir necha `useSelector` yozish, butun slice obyektini olish (`store.account`) yoki memoizatsiyalangan selector (`createSelector`, Reselect) ishlatish.

## 8. [Strong junior] Legacy `connect` nima? Nega hook'lar uni almashtirdi?

Hook'lardan oldin component'lar store'ga **`connect` HOC** orqali ulanardi:

```jsx
function mapStateToProps(state) {
  return { balance: state.account.balance };
}
export default connect(mapStateToProps)(BalanceDisplay);
```

State va dispatch component'ga **props** sifatida kelardi. `useSelector`/`useDispatch` hook'lari kamroq kod, kamroq ichma-ich o'rash va o'qilishi osonroq. `connect` eski kod bazalarida hamon uchraydi, shuning uchun uni tanish kerak.

## 9. [Strong junior] Middleware nima? Nega async kodni reducer'ga yozmaymiz?

Reducer **toza va sinxron** bo'lishi kerak, shuning uchun ichida API chaqirib bo'lmaydi. Component'da chaqirish mumkin, lekin shunda ma'lumot olish mantig'i component'larga tarqalib ketadi.

**Middleware**: action dispatch qilingandan keyin, reducer'ga yetib borishidan **oldin** ishlaydigan funksiya. U yerda side effect'lar (API, timer, log) bajarish mumkin.

```
dispatch → Middleware (async ish, log) → Store/Reducer → yangi state
```

Eng ko'p ishlatiladigan middleware: **Redux Thunk**.

## 10. [Strong junior] Thunk nima va qanday ishlaydi? (Valyuta konvertatsiyasi misoli)

**Thunk**: action creator **obyekt o'rniga funksiya** qaytaradi. Middleware bu funksiyani `dispatch` va `getState` bilan chaqiradi, funksiya esa async ishni bajarib, oxirida oddiy action'ni dispatch qiladi.

```js
export function deposit(amount, currency) {
  if (currency === "USD") return { type: "account/deposit", payload: amount };

  return async function (dispatch, getState) {
    dispatch({ type: "account/convertingCurrency" });   // isLoading: true

    const res = await fetch(
      `https://api.frankfurter.app/latest?amount=${amount}&from=${currency}&to=USD`
    );
    const data = await res.json();
    const converted = data.rates.USD;

    dispatch({ type: "account/deposit", payload: converted });
  };
}
```

Component odatdagidek yozadi: `dispatch(deposit(amount, currency))`. Async mantiq slice ichida yashirinadi, component esa sodda qoladi.

Klassik Redux'da thunk middleware qo'lda ulanadi: `createStore(rootReducer, applyMiddleware(thunk))`. RTK'da esa u **avtomatik** ulangan.

Real loyihada `res.ok` tekshiruvi va xato uchun alohida action (`try/catch`) qo'shish kerak.

## 11. [Junior] Redux DevTools nima beradi?

Brauzer kengaytmasi. Imkoniyatlari:

- Har bir dispatch qilingan **action**ni va uning `payload`ini ko'rish
- Har action'dan keyingi **state** va farqini (diff) ko'rish
- **Time travel**: action'lar bo'ylab orqaga-oldinga yurib, ilova holatini qayta tiklash
- Action'ni qo'lda **dispatch** qilib sinash

Klassik Redux'da qo'shimcha sozlash kerak, RTK'ning `configureStore`ida esa **avtomatik** ulanadi. Bu Redux'ning Context'ga nisbatan katta afzalliklaridan biri.

## 12. [Junior] Redux Toolkit (RTK) nima? Nega hozir RTK tavsiya etiladi?

RTK bu Redux yozishning **zamonaviy, rasmiy tavsiya etilgan** usuli. Klassik Redux bilan to'liq mos, lekin:

- **Kamroq boilerplate**: action creator'lar avtomatik yaratiladi
- **Immer** ichida: reducer'da state'ni "mutatsiya qilgandek" yozish mumkin
- `configureStore` **thunk middleware** va **DevTools**ni avtomatik ulaydi
- `createAsyncThunk`, `createSelector`, **RTK Query** kabi qo'shimcha vositalar

Yangi loyihada klassik Redux yozilmaydi, faqat RTK.

## 13. [Strong junior] `configureStore` va `createSlice` qanday ishlaydi?

```js
// store.js
import { configureStore } from "@reduxjs/toolkit";
import accountReducer from "./features/accounts/accountSlice";
import customerReducer from "./features/customers/customerSlice";

const store = configureStore({
  reducer: { account: accountReducer, customer: customerReducer },
});
export default store;
```

```js
// accountSlice.js
import { createSlice } from "@reduxjs/toolkit";

const accountSlice = createSlice({
  name: "account",
  initialState,
  reducers: {
    deposit(state, action) {
      state.balance += action.payload;   // Immer tufayli "mutatsiya" mumkin
      state.isLoading = false;
    },
    withdraw(state, action) {
      state.balance -= action.payload;
    },
  },
});

export const { deposit, withdraw } = accountSlice.actions;
export default accountSlice.reducer;
```

`createSlice` uchta narsani bitta joyda yaratadi: **reducer**, **action creator'lar** va **action type'lar** (`"account/deposit"`). `configureStore` `combineReducers` ni ichida o'zi bajaradi.

## 14. [Strong junior] RTK reducer'ida state'ni to'g'ridan-to'g'ri o'zgartirish nega mumkin? Qanday tuzoqlari bor?

RTK ichidagi **Immer** reducer'ga haqiqiy state o'rniga **qoralama (draft)** beradi. Biz draft'ni o'zgartiramiz, Immer esa o'zgarishlarni kuzatib, ortida **yangi immutable state** yasaydi. Natija klassik Redux bilan bir xil: eski state buzilmaydi.

Tuzoqlar:

- Bir reducer ichida **ham draft'ni o'zgartirib, ham yangi qiymat qaytarib** bo'lmaydi: ikkalasidan birini tanlash kerak
- Arrow function'ning qisqa yozuvi beixtiyor qiymat qaytaradi: `(state) => state.count++` muammo keltiradi (uni `{}` ichiga olish kerak)
- Erta chiqish uchun oddiy `return;` yoziladi (state o'zgarmay qoladi)
- Bu faqat `createSlice` (Immer) ichida ishlaydi, oddiy `useReducer` yoki klassik Redux'da emas

## 15. [Strong junior] Action bir nechta argument qabul qilishi kerak bo'lsa, nima qilamiz? (`prepare`)

`createSlice` dagi avtomatik action creator **faqat bitta argument** qabul qiladi va uni `payload` qiladi. Bir nechta argument kerak bo'lsa (masalan, `requestLoan(amount, purpose)`), **`prepare`** funksiyasi yoziladi:

```js
requestLoan: {
  prepare(amount, purpose) {
    return { payload: { amount, purpose } };
  },
  reducer(state, action) {
    if (state.loan > 0) return;
    state.loan = action.payload.amount;
    state.loanPurpose = action.payload.purpose;
    state.balance += action.payload.amount;
  },
},
```

`prepare` argumentlarni `payload` obyektiga yig'adi. Yana bir variant: component'dan bitta obyekt yuborish: `dispatch(requestLoan({ amount, purpose }))`. `prepare` ichida ID yoki sana yaratish ham mumkin (reducer toza qolishi uchun), masalan `createdAt: new Date().toISOString()`.

## 16. [Strong junior] RTK'da thunk qanday yoziladi? `createAsyncThunk` nima?

Ikki yo'l bor:

**1. Qo'lda yozilgan thunk** (kursdagi yondashuv): `createSlice` dan tashqarida, eski usuldagi kabi funksiya qaytaradigan action creator yoziladi. Slice esa `deposit` ni boshqaradi, `export const { withdraw, ... } = accountSlice.actions;` dan `deposit` chiqarib tashlanadi va o'rniga qo'lda yozilgan `deposit` export qilinadi. Bu oddiy holatlar uchun qisqa va tushunarli.

**2. `createAsyncThunk`** (RTK'ning rasmiy yo'li): async funksiyadan avtomatik ravishda `pending`, `fulfilled` va `rejected` action'larini yaratadi, slice ularni `extraReducers` da ushlaydi:

```js
export const fetchAddress = createAsyncThunk("user/fetchAddress", async function () {
  const res = await fetch(...);
  return await res.json();    // fulfilled action'ning payload'i bo'ladi
});

extraReducers: (builder) =>
  builder
    .addCase(fetchAddress.pending,   (state) => { state.status = "loading"; })
    .addCase(fetchAddress.fulfilled, (state, action) => { state.address = action.payload; state.status = "idle"; })
    .addCase(fetchAddress.rejected,  (state, action) => { state.status = "error"; state.error = action.error.message; }),
```

`createAsyncThunk` loading/error holatlarini standart va bir xil ko'rinishda boshqarishga yordam beradi (kursda Fast React Pizza'da uchraydi).

## 17. [Middle] Bir action bir nechta slice'ga ta'sir qilishi mumkinmi?

Ha. Har dispatch qilingan action **barcha reducer'larga** boradi. Boshqa slice o'z `extraReducers` ida boshqa slice'ning action'ini ushlab, o'z state'ini yangilashi mumkin:

```js
// customerSlice
extraReducers: (builder) =>
  builder.addCase(accountClosed, (state) => { /* mijoz ma'lumotini tozalash */ }),
```

Bu Redux'ning "action hodisa, buyruq emas" g'oyasi: bitta hodisaga bir necha slice o'zicha javob beradi. Shu sababli action nomlari hodisani ifodalashi kerak (`accountClosed`, `setBalance` emas).

## 18. [Middle] Redux va Context API: qachon qaysi biri?

| | **Context API** (+ `useReducer`) | **Redux (RTK)** |
|---|---|---|
| O'rnatish | React ichida, qo'shimcha paket yo'q | Paket o'rnatish kerak |
| Boshlash | Bitta state uchun oson | Bir marta sozlansa, yangi slice qo'shish oson |
| Ko'p state | Har domen uchun alohida Context, Provider'lar ko'payib ketadi ("provider hell") | Bitta store, ko'p slice |
| Async | Qo'lda (provider'dagi funksiyalar) | Middleware, thunk, `createAsyncThunk` |
| Re-render | Qiymat o'zgarsa, barcha consumer'lar render bo'ladi, optimallash qo'lda | `useSelector` faqat kerakli qism o'zgarganda render qiladi |
| DevTools | Yo'q (faqat React DevTools) | Kuchli: action log, time travel |

Tavsiya:

- **Context**: kichik va o'rta ilovalar, **kam o'zgaradigan** global state (tema, til, foydalanuvchi), bitta-ikki domen
- **Redux**: katta ilova, **tez-tez o'zgaradigan** murakkab global UI state, ko'p domen, kuchli debug kerak bo'lganda
- **Server state** uchun ikkalasi ham emas: **React Query** yoki **RTK Query**. Bugungi kunda ko'p ilovalar umuman Redux'siz yashaydi: server state React Query'da, oz miqdordagi UI state Context yoki Zustand'da

## 19. [Middle] Redux'ning asosiy kamchiliklari nima? Zamonaviy alternativalar qaysi?

- **Murakkablik va boilerplate** (klassik Redux'da ayniqsa; RTK buni ancha kamaytirgan)
- **Kichik ilovalar uchun ortiqcha**: oddiy state uchun store, slice, Provider sozlash keraksiz
- **O'rganish egri chizig'i**: middleware, thunk, selector, normalizatsiya kabi tushunchalar
- Server ma'lumotini Redux'da saqlash keshlash va qayta yuklashni qo'lda yozishga majbur qiladi

Alternativalar: **Zustand** (minimal, hook asosidagi global state), **Jotai/Recoil** (atom asosidagi), **React Query / RTK Query** (server state). Intervyuda Redux'ni bilish hamon muhim, chunki ko'p mavjud kod bazalarida u bor.
