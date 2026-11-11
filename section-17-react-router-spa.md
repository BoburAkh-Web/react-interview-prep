# Section 17: React Router: Building Single-Page Applications (SPA)

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: WorldWise (sayohat kuzatuv ilovasi): bosh sahifa, product, pricing, login, `/app` ichida shaharlar va mamlakatlar ro'yxati, shahar tafsiloti va forma.

> Bu fayl faqat **React Router va SPA nazariyasi** uchun. WorldWise loyihasining o'zi bo'yicha (kod va arxitektura) savollar alohida faylda: `projects/worldwise.md`.

---

## 1. [Junior] SPA nima? Routing nima? Oddiy ko'p sahifali saytdan farqi?

**Ko'p sahifali sayt**: har bir havola yangi HTML sahifani serverdan yuklaydi, sahifa to'liq qayta yuklanadi.

**SPA (Single-Page Application)**: server **bitta HTML** beradi va qolganini JavaScript boshqaradi. Foydalanuvchi "sahifa"dan "sahifa"ga o'tganda brauzer sahifani qayta yuklamaydi, React faqat kerakli qismni almashtiradi. Natijada o'tish tez, ilova "native" ilovadek seziladi va state saqlanib qoladi.

**Routing**: URL ga qarab qaysi component ko'rsatilishini aniqlash. SPA'da buni brauzerning **History API**si orqali **kutubxona** (React Router) bajaradi, server so'ralmaydi.

Kamchiliklari: birinchi yuklanish og'irroq bo'lishi mumkin va SEO uchun qo'shimcha choralar kerak (SSR yoki Next.js kabi yechimlar).

## 2. [Junior] React Router'ning asosiy komponentlari qanday? Oddiy marshrutlar qanday yoziladi?

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Homepage />} />
        <Route path="product" element={<Product />} />
        <Route path="pricing" element={<Pricing />} />
        <Route path="login" element={<Login />} />
        <Route path="*" element={<PageNotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

- **`BrowserRouter`**: routing'ni yoqadi va URL'ni kuzatadi
- **`Routes`**: URL'ga mos keladigan **bitta** `Route` ni tanlaydi
- **`Route`**: `path` (URL) va `element` (ko'rsatiladigan component)
- **`path="*"`**: hech biriga mos kelmasa ishlaydi (404 sahifa)

## 3. [Junior] `<Link>` va oddiy `<a>` teg farqi nima? `<NavLink>` nima uchun kerak?

`<a href>` bosilganda brauzer **sahifani to'liq qayta yuklaydi** (SPA ma'nosi yo'qoladi, state o'chib ketadi). `<Link to="/pricing">` esa sahifani yuklamasdan faqat URL va ko'rinishni almashtiradi.

`<NavLink>` `Link` ning kengaytmasi: hozirgi URL unga mos kelsa, avtomatik **`active` class** qo'shadi. Shuning uchun navigatsiyada joriy sahifani ajratib ko'rsatish uchun ishlatiladi.

```jsx
<NavLink to="/product">Product</NavLink>
```

## 4. [Junior] React'da stillashning qanday usullari bor?

1. **Inline style**: `style={{ color: "red" }}` (oddiy, lekin pseudo-class va media query yo'q)
2. **Global CSS fayl**: oddiy, lekin class nomlari to'qnashishi mumkin
3. **CSS Modules**: class nomlari component bo'yicha ajratiladi
4. **CSS-in-JS** (styled-components, Emotion): stil JavaScript ichida, props'ga bog'liq stil
5. **Utility-first** (Tailwind CSS): tayyor kichik class'lar bilan tez yozish
6. **UI kutubxonalari** (MUI, Chakra UI, shadcn/ui): tayyor component'lar

Tanlov loyiha hajmi va jamoa kelishuviga bog'liq. Hozirgi ko'p loyihalarda Tailwind yoki CSS Modules uchraydi.

## 5. [Strong junior] CSS Modules qanday ishlaydi? Qanday muammoni hal qiladi?

Fayl nomi `*.module.css` bo'ladi. Import qilinganda class nomlari **avtomatik noyob nomga** aylantiriladi (masalan, `nav_abc123`), shuning uchun bir component'dagi `.nav` boshqa component'dagi `.nav` bilan to'qnashmaydi.

```jsx
import styles from "./PageNav.module.css";

<nav className={styles.nav}>...</nav>
```

Global class kerak bo'lsa (masalan, `NavLink` qo'shadigan `active` class), CSS ichida `:global(.active)` yoziladi, chunki oddiy class nomi hash qilingan bo'lib qoladi va `active` ga mos kelmaydi.

## 6. [Strong junior] Nested route va `index` route nima? `<Outlet />` nima qiladi?

Ba'zi sahifalar ichida **ichki sahifalar** bo'ladi (masalan, `/app` ichida `cities`, `countries`). Bunda child `Route`'lar parent `Route` ichiga yoziladi:

```jsx
<Route path="app" element={<AppLayout />}>
  <Route index element={<Navigate to="cities" replace />} />
  <Route path="cities" element={<CityList />} />
  <Route path="cities/:id" element={<City />} />
  <Route path="countries" element={<CountryList />} />
  <Route path="form" element={<Form />} />
</Route>
```

- Parent component (`AppLayout`) ichida **`<Outlet />`** child route'ning component'i ko'rsatiladigan joyni belgilaydi
- **`index` route**: parent URL'ning o'ziga (`/app`) kirilganda ko'rsatiladigan standart child, unda `path` yozilmaydi

## 7. [Strong junior] URL'da state saqlash nima uchun kerak? Qachon ishlatamiz?

URL bu **global state**: unga barcha component'lar kira oladi. URL'da state saqlashning foydalari:

- State'ni sahifalar orasida **prop drilling'siz** uzatish (masalan, tanlangan shahar `id`si)
- Foydalanuvchi sahifani **saqlab qo'ysa (bookmark) yoki ulashsa**, aynan o'sha holat tiklanadi
- Brauzerning **Back/Forward** tugmalari ishlaydi
- Sahifa yangilanganda holat yo'qolmaydi

Tipik misollar: tanlangan element `id`si, filtr, saralash, sahifa raqami, qidiruv so'zi. **Mos kelmaydi**: sezgir ma'lumot (parol, token), katta hajmdagi ma'lumot va hover kabi vaqtinchalik UI holatlari.

Ikki usul: **URL parametri** (`/cities/73930385`, resursni aniqlaydi) va **query string** (`?lat=38&lng=-9`, ixtiyoriy qo'shimcha ma'lumot).

## 8. [Strong junior] Dinamik route va `useParams` qanday ishlaydi?

Route yo'lida `:` bilan o'zgaruvchan qism belgilanadi:

```jsx
<Route path="cities/:id" element={<City />} />
```

```jsx
const { id } = useParams();   // /cities/73930385 → "73930385"
```

Bitta `City` component'i turli `id` lar uchun ishlaydi. Muhim: `id` doim **string**. Havola yasash:

```jsx
<Link to={`${id}?lat=${position.lat}&lng=${position.lng}`}>...</Link>
```

`id` o'zgarsa, shahar ma'lumotini yuklaydigan effect `[id]` dependency orqali qayta ishlaydi.

## 9. [Strong junior] Query string'ni qanday o'qiymiz va o'zgartiramiz?

```jsx
const [searchParams, setSearchParams] = useSearchParams();

const lat = searchParams.get("lat");   // "38" (string)
const lng = searchParams.get("lng");

setSearchParams({ lat: 23, lng: 50 }); // URL: ?lat=23&lng=50
```

Query string o'zgarganda route o'zgarmaydi, faqat URL'dagi parametrlar va ularni o'qiydigan component'lar yangilanadi. Bu mantiq bir necha joyda kerak bo'lsa, uni custom hook'ga ajratamiz (masalan, `useUrlPosition`), shunda `lat` va `lng` ni olish bir joyda turadi.

## 10. [Strong junior] `useNavigate` bilan dasturiy navigatsiya qanday ishlaydi?

Foydalanuvchi havolani bosmasdan, **biror harakat natijasida** (xaritani bosish, forma yuborish) boshqa sahifaga o'tkazish kerak bo'lganda:

```jsx
const navigate = useNavigate();

navigate("form");                       // yangi sahifaga
navigate(-1);                           // bir qadam orqaga (Back kabi)
navigate("/app", { replace: true });    // tarixni almashtiradi (Back bosganda qaytmaydi)
```

Forma ichidagi "Back" tugmasi submit'ni ishga tushirmasligi uchun `e.preventDefault()` yoziladi, keyin `navigate(-1)`.

## 11. [Strong junior] `<Navigate />` komponenti va `useNavigate` farqi nima? Nega `replace` ishlatamiz?

- **`useNavigate`**: event handler yoki effect ichida chaqiriladigan **funksiya** (harakatga javoban)
- **`<Navigate to="..." />`**: JSX ichida yoziladigan **komponent**, render bo'lganda darhol yo'naltiradi (deklarativ redirect, masalan `index` route'dan `cities` ga o'tkazish yoki login qilmagan foydalanuvchini bosh sahifaga qaytarish)

```jsx
<Route index element={<Navigate to="cities" replace />} />
```

**`replace`** joriy tarix yozuvini **almashtiradi**. Aks holda foydalanuvchi Back bosganda yana redirect qiluvchi sahifaga qaytib, yana yo'naltirilib, "Back tugmasi ishlamayapti" holati (tuzoq) hosil bo'ladi.

Render ichida to'g'ridan-to'g'ri `navigate()` chaqirib bo'lmaydi (side effect). Shu sababli redirect uchun `<Navigate />` yoki effect/handler ishlatiladi.

## 12. [Middle] `BrowserRouter`, `HashRouter` va deploy muammosi: nima uchun sahifa yangilanganda 404 chiqadi?

`BrowserRouter` toza URL'lar beradi (`/app/cities`) va History API'dan foydalanadi. Lekin SPA'da **faqat bitta `index.html`** bor. Foydalanuvchi `/app/cities` da sahifani yangilasa yoki to'g'ridan-to'g'ri ochsa, brauzer serverdan shu yo'lni so'raydi, server esa bunday fayl yo'qligi uchun **404** qaytaradi.

Yechim: serverni barcha yo'llarni `index.html` ga yo'naltiradigan qilib sozlash (SPA fallback). Masalan, Netlify'da `_redirects` fayli (`/* /index.html 200`), Vercel'da `rewrites`.

`HashRouter` esa URL'da `#` ishlatadi (`/#/app/cities`) va server sozlamasini talab qilmaydi, lekin URL xunukroq va SEO'ga yomon. Shuning uchun real loyihalarda `BrowserRouter` + server sozlamasi tanlanadi.

## 13. [Middle] URL state va `useState` ni qachon tanlaymiz?

| URL (query/params) | `useState` / Context |
|---|---|
| Ulashilishi yoki bookmark qilinishi kerak | Faqat shu sessiya uchun |
| Sahifa yangilanganda saqlanishi kerak | Yo'qolsa ham bo'ladi |
| Sahifalar orasida o'tadi (tanlangan id, filtr) | Bitta component yoki kichik daraxt ichida |
| Qisqa, oddiy qiymatlar (id, son, qisqa matn) | Murakkab obyekt, katta ma'lumot |
| Sezgir bo'lmagan | Sezgir bo'lishi mumkin |

Ko'p ilovalarda ikkalasi birga ishlatiladi: URL "qaysi element tanlangan"ni, `useState` esa "menyu ochiqmi" kabi vaqtinchalik UI holatini saqlaydi.

## 14. [Middle] Zamonaviy React Router'da (data router) nima o'zgargan?

Yangi versiyalarda (v6.4+) marshrutlarni `createBrowserRouter` bilan **obyekt sifatida** e'lon qilish, **loader** (sahifa ochilishidan oldin ma'lumot yuklash), **action** (forma yuborishni boshqarish) va xato sahifalari (`errorElement`) mavjud. Bu ma'lumot yuklashni component'dan marshrutga ko'chiradi va "render keyin yuklash" (waterfall) muammosini kamaytiradi. Kursda bu keyingi section'da (Redux bilan birga, "Advanced React Router") ko'riladi. Oddiy `BrowserRouter` + `Routes` yondashuvi esa hali ham keng uchraydi va kichik/o'rta loyihalar uchun yetarli.
