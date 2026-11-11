# Section 24: Adding Redux and Advanced React Router

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: Fast React Pizza Co. (3-qism: `user` va `cart` slice'lari, `createAsyncThunk`, geolokatsiya, `useFetcher`).

> Redux asoslari (store, slice, `useSelector`, `useDispatch`, Immer, thunk) Section 20 faylida. Bu fayl Redux'ni **real loyihaga qo'llash** va React Router'ning ilg'or imkoniyatlari haqida.

---

## 1. [Junior] Fast React Pizza'da qaysi state Redux'da, qaysi biri Router'da? Nega?

| State | Turi | Qayerda |
|---|---|---|
| Menyu, buyurtmalar | **Server state** (serverdan keladi, server egasi) | React Router **loader**lari |
| Foydalanuvchi ismi, manzili | **Global UI state** | Redux `user` slice |
| Savat | **Global UI state** (faqat brauzerda, buyurtma berilgunicha) | Redux `cart` slice |
| Forma input'lari | **Local state** | `useState` yoki uncontrolled forma |

Qoida: **server state** va **client (UI) state** alohida boshqariladi. Server ma'lumotini Redux'ga ko'chirish keshlash va yangilashni qo'lda yozishga majbur qiladi.

## 2. [Junior] `user` slice qanday modellanadi va o'qiladi/yangilanadi?

```js
const userSlice = createSlice({
  name: "user",
  initialState: { username: "", status: "idle", position: {}, address: "", error: "" },
  reducers: {
    updateName(state, action) {
      state.username = action.payload;
    },
  },
});
export const { updateName } = userSlice.actions;
export default userSlice.reducer;
```

```jsx
// o'qish
const username = useSelector((state) => state.user.username);

// yangilash (CreateUser formasida)
const dispatch = useDispatch();
function handleSubmit(e) {
  e.preventDefault();
  if (!username) return;
  dispatch(updateName(username));
  navigate("/menu");
}
```

Ism kiritilgach, u Header, Home va buyurtma formasida avtomatik ko'rinadi, chunki ularning hammasi store'dan o'qiydi (prop drilling yo'q).

## 3. [Strong junior] `cart` state'i qanday modellanadi? Qaysi maydonlar saqlanadi, qaysilari hisoblanadi?

```js
const initialState = {
  cart: [
    // { pizzaId: 12, name: "Mediterranean", quantity: 2, unitPrice: 16, totalPrice: 32 }
  ],
};
```

Savatda faqat **zarur ma'lumot** saqlanadi: `pizzaId` (identifikator), `name`, `quantity`, `unitPrice`, `totalPrice`. Butun pizza obyekti (rasm, ingredientlar) saqlanmaydi.

**Hisoblanadigan** (derived) qiymatlar state'da saqlanmaydi: savatdagi jami narx, jami pitsa soni, savat bo'shmi. Ular **selector**lar bilan hisoblanadi (6-savol).

## 4. [Strong junior] Cart reducer'lari qanday yoziladi? (Qo'shish, o'chirish, miqdorni o'zgartirish)

```js
reducers: {
  addItem(state, action) {
    state.cart.push(action.payload);                         // Immer: "mutatsiya" mumkin
  },
  deleteItem(state, action) {
    state.cart = state.cart.filter((item) => item.pizzaId !== action.payload);
  },
  increaseItemQuantity(state, action) {
    const item = state.cart.find((item) => item.pizzaId === action.payload);
    item.quantity++;
    item.totalPrice = item.quantity * item.unitPrice;
  },
  decreaseItemQuantity(state, action) {
    const item = state.cart.find((item) => item.pizzaId === action.payload);
    item.quantity--;
    item.totalPrice = item.quantity * item.unitPrice;
    if (item.quantity === 0) cartSlice.caseReducers.deleteItem(state, action);
  },
  clearCart(state) {
    state.cart = [];
  },
},
```

Muhim nuqtalar:

- `find` massivdagi **obyektning o'zini** (havolasini) qaytaradi, shuning uchun uni o'zgartirish draft state'ni o'zgartiradi (Immer bor joyda)
- `totalPrice` oddiy saqlangan son, `quantity` o'zgarganda **qo'lda qayta hisoblanadi**
- Miqdor 0 ga tushsa, **`caseReducers`** orqali boshqa reducer'ni qayta ishlatamiz (mantiq takrorlanmaydi)

## 5. [Junior] Pitsani savatga qo'shish tugmasi va "savatda bormi" tekshiruvi qanday ishlaydi?

```jsx
function MenuItem({ pizza }) {
  const { id, name, unitPrice, soldOut } = pizza;
  const dispatch = useDispatch();
  const currentQuantity = useSelector(getCurrentQuantityById(id));
  const isInCart = currentQuantity > 0;

  function handleAddToCart() {
    dispatch(addItem({ pizzaId: id, name, quantity: 1, unitPrice, totalPrice: unitPrice }));
  }

  return (
    <>
      {isInCart && <DeleteItem pizzaId={id} />}
      {!soldOut && !isInCart && <Button type="small" onClick={handleAddToCart}>Add to cart</Button>}
    </>
  );
}
```

"Savatda bormi" alohida state emas, savatdan **hisoblab** olinadi (derived).

## 6. [Strong junior] Redux selector'lari nima? Nega ularni slice faylida saqlaymiz?

**Selector**: store state'idan kerakli ma'lumotni olib beradigan (yoki hisoblaydigan) funksiya.

```js
// cartSlice.js
export const getCart = (state) => state.cart.cart;

export const getTotalCartQuantity = (state) =>
  state.cart.cart.reduce((sum, item) => sum + item.quantity, 0);

export const getTotalCartPrice = (state) =>
  state.cart.cart.reduce((sum, item) => sum + item.totalPrice, 0);

export const getCurrentQuantityById = (id) => (state) =>
  state.cart.cart.find((item) => item.pizzaId === id)?.quantity ?? 0;
```

```jsx
const totalCartQuantity = useSelector(getTotalCartQuantity);
```

Foydalari:

- State **tuzilishi bir joyda**: tuzilma o'zgarsa (masalan, `state.cart.cart` → `state.cart.items`), faqat selector o'zgaradi, component'lar emas
- Hisoblash mantig'i qayta ishlatiladi
- `getCurrentQuantityById` kabi **argumentli selector** funksiya qaytaradigan funksiya sifatida yoziladi

## 7. [Middle] Selector'larning performance muammosi bormi? Memoizatsiya (`createSelector`) qachon kerak?

`useSelector` har store yangilanishida selector'ni qayta chaqiradi. Ikki xavf:

- Selector **har safar yangi obyekt yoki massiv** qaytarsa (`filter`, `map`, `{...}`), `===` solishtirish doim "o'zgardi" deb hisoblaydi va component **har gal render** bo'ladi
- Selector ichida **og'ir hisob-kitob** bo'lsa, u keraksiz qayta bajariladi

Yechim: RTK ichidagi **`createSelector`** (Reselect) bilan memoizatsiya qilish. Kiruvchi qiymatlar o'zgarmasa, oldingi natija qaytariladi:

```js
import { createSelector } from "@reduxjs/toolkit";

export const getTotalCartPrice = createSelector(
  [getCart],
  (cart) => cart.reduce((sum, item) => sum + item.totalPrice, 0)
);
```

Kichik savatda bu sezilmaydi (son qaytaradigan selector'lar xavfsiz), lekin katta ro'yxatlar yoki massiv qaytaradigan selector'lar uchun muhim.

## 8. [Strong junior] Savat ma'lumotini buyurtma action'ida qanday ishlatamiz? Action ichida store'ga qanday murojaat qilamiz?

Savat component'da `useSelector` bilan olinadi va **hidden input** orqali forma action'iga yuboriladi (Section 22, 10-savol). Buyurtma berilgach savatni **tozalash** kerak, lekin action **component emas**, unda `useDispatch` ishlatib bo'lmaydi. Kursdagi yechim: store'ni **to'g'ridan-to'g'ri import** qilish:

```js
import store from "../../store";

export async function action({ request }) {
  // ...validatsiya va createOrder
  store.dispatch(clearCart());
  return redirect(`/order/${newOrder.id}`);
}
```

Ogohlantirish: bu usul **kamdan-kam** ishlatilishi kerak. Store'ni to'g'ridan-to'g'ri import qilish testlashni qiyinlashtiradi va ba'zi optimizatsiyalarni o'chiradi. Ko'p ishlatilsa, kod tuzilmasini qayta ko'rib chiqish kerak.

## 9. [Strong junior] `createAsyncThunk` bilan geolokatsiya va manzilni olish qanday yoziladi?

```js
export const fetchAddress = createAsyncThunk("user/fetchAddress", async function () {
  // 1) foydalanuvchi joylashuvi
  const positionObj = await getPosition();   // navigator.geolocation'ni Promise'ga o'raydi
  const position = { latitude: positionObj.coords.latitude, longitude: positionObj.coords.longitude };

  // 2) reverse geocoding: koordinatadan manzil
  const addressObj = await getAddress(position);
  const address = `${addressObj?.locality}, ${addressObj?.city} ${addressObj?.postcode}, ${addressObj?.countryName}`;

  // 3) qaytgan qiymat fulfilled action'ning payload'i bo'ladi
  return { position, address };
});
```

```js
extraReducers: (builder) =>
  builder
    .addCase(fetchAddress.pending, (state) => { state.status = "loading"; })
    .addCase(fetchAddress.fulfilled, (state, action) => {
      state.position = action.payload.position;
      state.address = action.payload.address;
      state.status = "idle";
    })
    .addCase(fetchAddress.rejected, (state) => {
      state.status = "error";
      state.error = "There was a problem getting your address. Make sure to fill this field!";
    }),
```

`createAsyncThunk` uchta action'ni (**pending**, **fulfilled**, **rejected**) avtomatik yaratadi, slice ularni **`extraReducers`** da ushlaydi (chunki ular `reducers` ichida yaratilmagan). Component: `dispatch(fetchAddress())`.

## 10. [Strong junior] Geolokatsiyani formaga qanday integratsiya qildik? Nimalarga e'tibor berish kerak?

- Tugma bosilganda `dispatch(fetchAddress())`. Ruxsat oynasi chiqgani uchun **foydalanuvchi harakatiga javoban** chaqiriladi, mount paytida emas
- Manzil input'i `defaultValue={address}` bilan to'ldiriladi (forma uncontrolled)
- `status === "loading"` paytida input va tugma `disabled`
- `status === "error"` bo'lsa, xato xabari ko'rsatiladi, foydalanuvchi manzilni qo'lda yozishi mumkin
- Koordinatalar ham buyurtmaga yuboriladi: `<input type="hidden" name="position" value={position.longitude && position.latitude ? `${position.latitude},${position.longitude}` : ""} />`
- Promise'ga o'ralmagan `navigator.geolocation.getCurrentPosition` callback asosida ishlaydi, `async/await` bilan ishlatish uchun uni `new Promise(...)` ga o'rash kerak

## 11. [Strong junior] `useFetcher` nima? Navigatsiyasiz ma'lumot qanday yuklanadi?

Ba'zan ma'lumot **boshqa route'ning loader'idan** kerak bo'ladi, lekin o'sha sahifaga **o'tmasdan**. Masalan, buyurtma sahifasida ingredientlarni ko'rsatish uchun menyu ma'lumoti kerak.

```jsx
const fetcher = useFetcher();

useEffect(function () {
  if (!fetcher.data && fetcher.state === "idle") fetcher.load("/menu");
}, [fetcher]);

// fetcher.data → menu loader'ining natijasi
const ingredients = fetcher?.data?.find((el) => el.id === item.pizzaId)?.ingredients ?? [];
const isLoadingIngredients = fetcher.state === "loading";
```

- `fetcher.load(url)` o'sha route'ning **loader'ini** chaqiradi, URL va sahifa o'zgarmaydi
- Natija `fetcher.data` da, holati `fetcher.state` da (`"idle"`, `"loading"`, `"submitting"`)
- `useNavigation` dan farqi: `useNavigation` **global** navigatsiya holati, `useFetcher` esa **mustaqil**, bir sahifada bir nechta bo'lishi mumkin

## 12. [Strong junior] Navigatsiyasiz ma'lumot qanday yangilanadi? (`fetcher.Form`)

Masalan, buyurtmani "priority" qilish tugmasi. Sahifadan chiqmasdan action chaqirish:

```jsx
function UpdateOrder({ order }) {
  const fetcher = useFetcher();
  return (
    <fetcher.Form method="PATCH" className="text-right">
      <Button type="primary">Make priority</Button>
    </fetcher.Form>
  );
}

export async function action({ request, params }) {
  const data = { priority: true };
  await updateOrder(params.orderId, data);
  return null;
}
```

```js
// route
{ path: "/order/:orderId", element: <Order />, loader: orderLoader, action: updateOrderAction }
```

Muhim jihat: action tugagach React Router sahifadagi loader'larni **avtomatik qayta ishga tushiradi (revalidation)**, shuning uchun buyurtma ma'lumoti yangilanadi va UI o'zgaradi. `useEffect` yoki qo'lda refetch yozish shart emas.

## 13. [Middle] `<Form>`, `fetcher.Form` va `fetcher.submit` farqi nima? Qachon qaysi biri?

| | `<Form>` | `fetcher.Form` / `fetcher.submit` |
|---|---|---|
| Navigatsiya | Bor (action'dan keyin `redirect` mumkin) | **Yo'q**, sahifada qoladi |
| Holat | Global `useNavigation()` | O'zining `fetcher.state` i |
| Bir sahifada nechta | Odatda bitta asosiy forma | Bir nechta mustaqil fetcher bo'lishi mumkin |
| Tipik ishlatish | Yangi buyurtma yaratish, login | "Like", "priority", ro'yxatdagi element holatini o'zgartirish |

`fetcher.submit(data, { method: "PATCH" })` esa forma tegisiz, kod orqali yuborish uchun.

## 14. [Middle] Fast React Pizza arxitekturasining kuchli va zaif tomonlari qanday?

**Kuchli:**

- Server state (loader/action) va UI state (Redux) aniq ajratilgan
- Feature-based tuzilma, slice'lar va selector'lar tartibli
- Formalar uncontrolled va router action'lari bilan ishlaydi, kam state va kam kod
- Navigatsiyasiz yangilash va avtomatik revalidation

**Zaif / yaxshilash mumkin:**

- Savat **sahifa yangilanganda yo'qoladi**: `localStorage` ga saqlash (persist) kerak
- Server ma'lumoti uchun **kesh yo'q**: React Query bilan birga ishlatish mumkin
- Action ichida store'ni to'g'ridan-to'g'ri import qilish (8-savol)
- TypeScript yo'q, testlar yo'q
- API manzillari va xatolarni boshqarish (`res.ok` tekshiruvi) markazlashtirilishi kerak
