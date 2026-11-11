# Section 22: React Router With Data Loading (v6.4+)

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: Fast React Pizza Co. (1-qism: rejalash, fayl tuzilmasi, loader va action'lar).

> Fast React Pizza loyihasining haqiqiy kodi bo'yicha savollar `projects/fast-react-pizza-interview.md` faylida. Bu fayl **nazariya** uchun. Loyiha 3 section'ga bo'lingan: 22 (Router data loading), 23 (Tailwind), 24 (Redux + advanced Router).

---

## 1. [Junior] Yangi loyihani boshlashdan oldin qanday rejalashtiramiz?

Kursdagi tartib (Section 7 dagi "Thinking in React" ning kattaroq versiyasi):

1. **Talablarni yig'ish**: ilova nima qilishi kerak (menyu, savat, buyurtma, buyurtmani kuzatish)
2. **Funksiyalarni kategoriyalarga bo'lish**: User, Menu, Cart, Order
3. **Sahifalarni aniqlash**: `/`, `/menu`, `/cart`, `/order/new`, `/order/:orderId`
4. **State turlarini aniqlash**: qaysi biri **global UI state** (user, cart), qaysi biri **server state** (menu, orders)
5. **Texnologiyalarni tanlash**: routing, styling, UI state, server state uchun vositalar

Natijada har bir qarorning sababi bor: masalan, menyu va buyurtmalar **server state** bo'lgani uchun Router loader'lari bilan yuklanadi, savat va foydalanuvchi esa **UI state** bo'lgani uchun Redux'da saqlanadi.

## 2. [Junior] "Feature-based" fayl tuzilmasi qanday? Nega u professional hisoblanadi?

```
src/
├── features/        # domen bo'yicha: user/, menu/, cart/, order/
├── ui/              # qayta ishlatiladigan umumiy UI: Button, Loader, Header, AppLayout
├── services/        # tashqi API bilan ishlash: apiRestaurant.js, apiGeocoding.js
├── utils/           # yordamchi funksiyalar: formatCurrency, formatDate
├── store.js
└── App.jsx
```

Har bir **feature** papkasida o'sha domenning component'lari, slice'i va mantig'i birga turadi. Afzalligi: loyiha kattalashganda "bu narsa qayerda?" degan savol tug'ilmaydi, bir feature ustida ishlash boshqalariga tegmaydi, kod topish oson. Kichik loyihada "turi bo'yicha" (`components/`, `pages/`) yetarli, katta loyihada feature-based ustun.

## 3. [Junior] Yangi usulda route'lar qanday e'lon qilinadi? (`createBrowserRouter`)

```jsx
import { createBrowserRouter, RouterProvider } from "react-router-dom";

const router = createBrowserRouter([
  {
    element: <AppLayout />,
    errorElement: <Error />,
    children: [
      { path: "/", element: <Home /> },
      { path: "/menu", element: <Menu />, loader: menuLoader, errorElement: <Error /> },
      { path: "/cart", element: <Cart /> },
      { path: "/order/new", element: <CreateOrder />, action: createOrderAction },
      { path: "/order/:orderId", element: <Order />, loader: orderLoader, errorElement: <Error /> },
    ],
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

Route'lar JSX emas, **obyektlar massivi** sifatida yoziladi. Shu sababli har bir route'ga **loader**, **action** va **errorElement** biriktirish mumkin, ya'ni **data API**lar faqat shu usulda ishlaydi.

## 4. [Junior] Layout route nima? `path` siz route nimani anglatadi?

Yuqoridagi birinchi obyektda `path` yo'q, faqat `element: <AppLayout />` va `children` bor. Bu **layout route**: URL'ga hech narsa qo'shmaydi, faqat child route'larni umumiy qobiq (header, footer, savat paneli) ichida `<Outlet />` orqali ko'rsatadi.

```jsx
function AppLayout() {
  return (
    <div>
      <Header />
      <main><Outlet /></main>
      <CartOverview />
    </div>
  );
}
```

## 5. [Strong junior] Loader nima? U qanday ishlaydi?

**Loader**: route ochilishidan **oldin** ma'lumot yuklaydigan funksiya. React Router uni chaqiradi, natijani component'ga `useLoaderData()` orqali beradi.

```jsx
// Menu.jsx
export async function loader() {
  const menu = await getMenu();   // services/apiRestaurant.js
  return menu;
}

function Menu() {
  const menu = useLoaderData();
  return <ul>{menu.map((pizza) => <MenuItem pizza={pizza} key={pizza.id} />)}</ul>;
}
```

Uch qadam: loader **yoziladi** (odatda component fayli ichida), route'ga **ulanadi** (`loader: menuLoader`), component'da **o'qiladi** (`useLoaderData`).

## 6. [Strong junior] "Fetch-on-render" va "render-as-you-fetch" farqi nima? Loader'ning `useEffect` dan afzalligi?

- **Fetch-on-render** (`useEffect` bilan): component **avval render bo'ladi**, keyin effect ichida so'rov ketadi. Ichma-ich component'larda bu so'rovlar **ketma-ket** (waterfall) bo'lib qoladi: parent yuklab bo'lgach child render bo'ladi, keyin child o'z so'rovini yuboradi
- **Render-as-you-fetch** (loader bilan): router URL o'zgarishi bilanoq **ma'lumot yuklashni boshlaydi**, component'larni yuklash bilan **parallel**, component esa tayyor ma'lumot bilan render bo'ladi

Qo'shimcha foydalar: loading va error holatlari router darajasida markazlashadi, component ichida `isLoading`/`error` state'larini qo'lda yozish shart emas, ma'lumot va route bir joyda turadi.

## 7. [Strong junior] Loader ishlayotganda loading indikatorini qanday ko'rsatamiz?

```jsx
import { useNavigation } from "react-router-dom";

function AppLayout() {
  const navigation = useNavigation();
  const isLoading = navigation.state === "loading";

  return (
    <div>
      {isLoading && <Loader />}
      <Header />
      <main><Outlet /></main>
    </div>
  );
}
```

`useNavigation()` **butun ilova** bo'yicha navigatsiya holatini beradi: `"idle"`, `"loading"` (loader ishlayapti) yoki `"submitting"` (action ishlayapti). Shuning uchun loader'ni AppLayout'da bir marta ushlash kifoya, har sahifaga alohida spinner yozish kerak emas.

## 8. [Strong junior] Route xatolarini qanday ushlaymiz? (`errorElement`)

Loader, action yoki render paytida xato yuz bersa, React Router eng yaqin **`errorElement`** ni ko'rsatadi. Xato obyekti `useRouteError()` bilan olinadi:

```jsx
function Error() {
  const error = useRouteError();
  const navigate = useNavigate();
  return (
    <div>
      <h1>Something went wrong 😢</h1>
      <p>{error.data || error.message}</p>
      <button onClick={() => navigate(-1)}>&larr; Go back</button>
    </div>
  );
}
```

`errorElement` ni child route'ga ham qo'yish mumkin. Shunda xato faqat **shu route ichida** (Outlet o'rnida) ko'rinadi va layout (header, savat) saqlanib qoladi. Mavjud bo'lmagan URL'ga kirilganda ham (404) shu `errorElement` ishlaydi, `error.data` da xabar bo'ladi.

## 9. [Strong junior] Loader URL parametrlarini qanday oladi?

Loader'da hook ishlatib bo'lmaydi (u component emas), shuning uchun `useParams` o'rniga router loader'ga **`params`** obyektini beradi:

```jsx
export async function loader({ params }) {
  const order = await getOrder(params.orderId);
  return order;
}
```

Shu kabi action'ga ham `{ request, params }` beriladi. Hook qoidasi bu yerda ham amal qiladi: loader va action **oddiy funksiyalar**, ichida hook chaqirib bo'lmaydi.

## 10. [Strong junior] Action nima? Forma ma'lumotini qanday yuboramiz?

**Action**: route'ga ma'lumot **yozish** (POST, PATCH, DELETE) uchun funksiya. React Router'ning `<Form>` component'i yuborilganda, router tegishli route'ning action'ini chaqiradi.

```jsx
<Form method="POST">
  <input type="text" name="customer" required />
  <input type="tel" name="phone" required />
  <input type="text" name="address" required />
  <input type="checkbox" name="priority" id="priority" />
  <input type="hidden" name="cart" value={JSON.stringify(cart)} />
  <button>Order now</button>
</Form>
```

```jsx
export async function action({ request }) {
  const formData = await request.formData();
  const data = Object.fromEntries(formData);

  const order = {
    ...data,
    cart: JSON.parse(data.cart),
    priority: data.priority === "on",
  };

  const newOrder = await createOrder(order);
  return redirect(`/order/${newOrder.id}`);
}
```

Muhim nuqtalar:

- Input'larda **`name`** atributi bo'lishi shart, ma'lumot shu nom bilan keladi
- Forma **uncontrolled**: har bir input uchun `useState` va `onChange` kerak emas
- Hamma qiymat **string** keladi: checkbox `"on"` bo'ladi, massiv/obyekt `JSON.stringify` bilan hidden input orqali yuboriladi
- Action'da `useNavigate` ishlatib bo'lmaydi, uning o'rniga **`redirect()`** qaytariladi

## 11. [Strong junior] Action'dagi validatsiya xatolarini qanday ko'rsatamiz?

Xato bo'lsa, action `redirect` o'rniga **xatolar obyektini qaytaradi**. Component uni `useActionData()` bilan oladi:

```jsx
export async function action({ request }) {
  const data = Object.fromEntries(await request.formData());
  const errors = {};
  if (!isValidPhone(data.phone))
    errors.phone = "Please give us your correct phone number.";

  if (Object.keys(errors).length > 0) return errors;

  const newOrder = await createOrder(order);
  return redirect(`/order/${newOrder.id}`);
}

function CreateOrder() {
  const formErrors = useActionData();
  return <>{formErrors?.phone && <p>{formErrors.phone}</p>}</>;
}
```

Yuborish jarayonini ko'rsatish uchun ham `useNavigation` ishlatiladi:

```jsx
const isSubmitting = useNavigation().state === "submitting";
<button disabled={isSubmitting}>{isSubmitting ? "Placing order..." : "Order now"}</button>
```

## 12. [Middle] Router loader'lari React Query'ning o'rnini bosadimi?

To'liq emas. Loader'lar **render-as-you-fetch** va route bilan bog'liq yuklashni yaxshi hal qiladi, lekin:

- **Kesh yo'q** (yoki juda cheklangan): sahifaga qaytilganda ma'lumot qayta yuklanadi
- **Fon rejimida yangilash**, eskirgan ma'lumotni qayta yuklash (stale-while-revalidate), optimistik yangilash kabi imkoniyatlar yo'q
- Bir xil ma'lumot bir nechta component'da kerak bo'lsa, ulashish qo'lda

Kichik va o'rta ilovalar uchun loader'lar yetarli. Murakkab server state uchun React Query yaxshiroq, ikkalasini **birga** ham ishlatish mumkin: loader ichida `queryClient.ensureQueryData(...)` chaqirib, keshni oldindan to'ldirish.

## 13. [Middle] React Router'ning yangi versiyalarida nima o'zgargan?

Kurs v6.4+ data API'larini ko'rsatadi. **v7** da asosiy g'oyalar (loader, action, `createBrowserRouter`) saqlangan, lekin:

- Asosiy paket **`react-router`** (`react-router-dom` hamon qayta eksport qiladi)
- `json()` yordamchisi **eskirgan**: loader va action'dan oddiy obyekt qaytariladi
- Remix bilan birlashgan "framework mode" paydo bo'lgan (fayl asosidagi routing, SSR)

Intervyuda versiyani aniq aytish va farqlarni bilish yaxshi taassurot qoldiradi.
