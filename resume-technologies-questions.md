# Rezyumedagi texnologiyalar bo'yicha savol-javoblar

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

> Bu fayl rezyumedagi **"Languages & Technologies"** bo'limiga tayanadi. `resume-interview-questions.md` dagi 7–9-bo'limlar (JS asoslari, TS asoslari, Next.js asoslari, Axios, Vite/Webpack, SASS, MUI, Supabase/Firebase, ESLint, REST, SEO va h.k.) bu yerda **takrorlanmaydi**, bu fayl ularni **chuqurlashtiradi** va qolgan texnologiyalarni qamraydi.
>
> React, Redux/RTK, React Router, Context, TanStack Query, Tailwind va React Hook Form bo'yicha asosiy savollar `react-course/` papkasida. Bu yerda faqat ular orasidagi bog'lanish va rezyumega xos savollar.

| Texnologiya | Qayerda |
|---|---|
| React, hook'lar, performance | `react-course/section-05` … `section-19` |
| Context API | `section-18` |
| Redux, Redux Toolkit | `section-20`, `section-24` |
| React Router | `section-17`, `section-22` |
| TanStack Query, React Hook Form | `section-27` |
| Tailwind CSS | `section-23` |

---

## 1. JavaScript (ES6+): chuqurroq savollar

### 1.1 [Junior] Hoisting nima?

JavaScript kodni bajarishdan oldin e'lonlarni "tepaga ko'taradi". `var` `undefined` bilan ko'tariladi, **function declaration** butunligicha ko'tariladi (e'londan oldin chaqirish mumkin), `let`/`const` va `class` ham ko'tariladi, lekin e'longacha **temporal dead zone**da bo'ladi (murojaat qilinsa `ReferenceError`). Function expression va arrow function o'zgaruvchi qoidasiga bo'ysunadi.

### 1.2 [Junior] `||` va `??` farqi nima? Optional chaining qachon kerak?

`||` har qanday **falsy** qiymatda (`0`, `""`, `false`, `null`, `undefined`) o'ng tomonni oladi. `??` faqat **`null` yoki `undefined`** bo'lganda.

```js
const count = 0;
count || 10;   // 10 (0 ni yo'qotdi)
count ?? 10;   // 0  (to'g'ri)

user?.address?.city   // biror qism yo'q bo'lsa xato emas, undefined
```

### 1.3 [Junior] Destructuring va rest/spread qanday ishlatiladi?

```js
const { name, age = 18, ...rest } = user;      // obyekt, default qiymat va qolganlari
const [first, , third] = arr;                   // massivdan tanlab olish
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }  // rest parametr
const merged = { ...defaults, ...options };    // spread: obyektlarni birlashtirish
```

React'da props destructuring, `useState` qaytargan massiv va state'ni immutable yangilash shu sintaksisga tayanadi.

### 1.4 [Strong junior] Prototype va prototypal inheritance nima? `class` bilan bog'liqligi?

Har bir obyektning yashirin `[[Prototype]]` havolasi bor. Xossa obyektning o'zida topilmasa, JavaScript uni **prototip zanjiri** bo'ylab qidiradi. Masalan, `[].map` massivning o'zida emas, `Array.prototype` da turadi. ES6 `class` bu prototiplar ustidagi **qulay sintaksis**: `extends` prototip zanjirini quradi, metodlar `prototype` ga qo'shiladi.

### 1.5 [Strong junior] Event delegation nima va nega foydali?

Ko'p child elementga alohida listener qo'yish o'rniga **bitta listener ota-elementga** qo'yiladi va `e.target` orqali qaysi child bosilganini aniqlanadi. Bu **bubbling** tufayli ishlaydi. Foyda: kam xotira, dinamik qo'shilgan elementlar ham avtomatik ishlaydi. React'ning o'zi event'larni shu usulda boshqaradi (listener root'da).

```js
list.addEventListener("click", (e) => {
  const item = e.target.closest("li");
  if (item) select(item.dataset.id);
});
```

### 1.6 [Junior] `localStorage`, `sessionStorage` va cookie farqi?

| | localStorage | sessionStorage | Cookie |
|---|---|---|---|
| Hajm | ~5 MB | ~5 MB | ~4 KB |
| Muddat | O'chirilmaguncha | Tab yopilguncha | Belgilangan muddat |
| Serverga yuboriladimi | Yo'q | Yo'q | **Har so'rovda** |
| JS o'qiy oladimi | Ha | Ha | Ha (`httpOnly` bo'lmasa) |

Faqat **string** saqlanadi, obyekt uchun `JSON.stringify`/`JSON.parse`. Sezgir ma'lumot (token) uchun eng xavfsizi `httpOnly` cookie.

### 1.7 [Junior] ES Modules va CommonJS farqi?

ESM: `import`/`export`, statik (build vositasi tahlil qila oladi, **tree shaking** ishlaydi), brauzer va zamonaviy Node.js qo'llaydi. CommonJS: `require`/`module.exports`, dinamik, Node.js'ning eski formati. Default va named export farqi ESM'ga tegishli.

### 1.8 [Junior] `Set` va `Map` nima uchun?

`Set`: faqat **noyob** qiymatlar to'plami (takrorlarni olib tashlash: `[...new Set(arr)]`, tez tekshirish: `set.has(x)`). `Map`: kalit-qiymat juftligi, kalit **istalgan tur** bo'lishi mumkin (obyekt ham), tartib saqlanadi, `size` bor. Katta ro'yxatda qidiruv uchun massivdagi `includes` (O(n)) o'rniga `Set.has` (O(1)) ancha tez.

### 1.9 [Strong junior] Pure function va immutability nima? React bilan qanday bog'liq?

**Pure function**: bir xil kirishga doim bir xil natija, tashqi narsani o'zgartirmaydi (side effect yo'q). **Immutability**: ma'lumotni joyida o'zgartirmasdan, yangi nusxa yaratish. React render'i va reducer'lar pure bo'lishi kerak, state esa immutable yangilanadi, chunki React o'zgarishni **havola** bo'yicha aniqlaydi.

### 1.10 [Middle] `fetch` bilan so'rovni qanday bekor qilasiz? Timeout qanday qo'shiladi?

```js
const controller = new AbortController();
fetch(url, { signal: controller.signal });
controller.abort();                      // bekor qilish

fetch(url, { signal: AbortSignal.timeout(5000) });   // 5 sekund timeout
```

React'da effect cleanup'ida `abort()` chaqiriladi (race condition'dan himoya). Bekor qilingan so'rov `AbortError` bilan reject bo'ladi, uni foydalanuvchiga xato sifatida ko'rsatmaslik kerak.

---

## 2. TypeScript: chuqurroq savollar

### 2.1 [Junior] Type narrowing nima?

Union turidagi qiymatni tekshiruv orqali **aniq turga toraytirish**:

```ts
function format(value: string | number) {
  if (typeof value === "string") return value.toUpperCase();   // bu yerda string
  return value.toFixed(2);                                      // bu yerda number
}
```

Usullar: `typeof`, `instanceof`, `in` operatori, `===` bilan solishtirish, o'zingiz yozgan **type guard** (`function isUser(x): x is User`).

### 2.2 [Strong junior] Discriminated union nima? React'da qayerda foydali?

Har bir variantda **umumiy literal maydon** (`status`, `type`) bo'lgan union. TypeScript shu maydonga qarab qolgan maydonlarni biladi:

```ts
type State =
  | { status: "loading" }
  | { status: "error"; error: string }
  | { status: "success"; data: User[] };

function render(state: State) {
  switch (state.status) {
    case "error":   return state.error;          // faqat shu yerda error bor
    case "success": return state.data.length;
  }
}
```

Foydalari: imkonsiz holatlar (bir vaqtda `data` va `error`) tip darajasida yo'q bo'ladi. Reducer action'larini tiplashda ham ishlatiladi.

### 2.3 [Junior] `enum` yoki string literal union: qaysi biri yaxshi?

Ko'pincha **literal union** (`type Role = "admin" | "chef" | "courier" | "user"`) tavsiya etiladi: qo'shimcha JavaScript kodi yaratmaydi, API'dan keladigan string'lar bilan to'g'ridan-to'g'ri mos keladi. Qiymatlar ro'yxati ham kerak bo'lsa, `as const`:

```ts
const ROLES = ["admin", "chef", "courier", "user"] as const;
type Role = (typeof ROLES)[number];
```

### 2.4 [Strong junior] `keyof`, `typeof` va indexed access type nima?

```ts
const user = { name: "Ali", age: 20 };
type User = typeof user;             // { name: string; age: number }
type UserKey = keyof User;           // "name" | "age"
type Age = User["age"];              // number

function getField<T, K extends keyof T>(obj: T, key: K): T[K] { return obj[key]; }
```

Mavjud qiymatdan yoki turdan **yangi tur yasash**: kodni takrorlamaslik va kalit nomidagi xatolarni oldini olish uchun.

### 2.5 [Strong junior] `as` (type assertion) va `satisfies` farqi?

- **`as`** TypeScript'ga "menga ishon" deydi: tekshiruvni **chetlab o'tadi**, xato yashirinishi mumkin
- **`satisfies`** qiymat turga **mosligini tekshiradi**, lekin qiymatning aniqroq turini saqlab qoladi

```ts
const routes = { home: "/", menu: "/menu" } satisfies Record<string, string>;
routes.menu;   // aniq kalit sifatida tanilgan
```

Qoida: `as` ni kamdan-kam ishlatish, `as any` dan butunlay qochish.

### 2.6 [Strong junior] Generic React component qanday yoziladi?

```tsx
type ListProps<T> = { items: T[]; render: (item: T) => React.ReactNode };

function List<T>({ items, render }: ListProps<T>) {
  return <ul>{items.map(render)}</ul>;
}

<List items={cabins} render={(cabin) => <li key={cabin.id}>{cabin.name}</li>} />
// cabin turi avtomatik aniqlanadi
```

Qayta ishlatiladigan Table, Select, List kabi component'lar uchun.

### 2.7 [Junior] `children`, `ref` va HTML atributlarini qanday tiplaymiz?

```tsx
type ButtonProps = React.ComponentProps<"button"> & { variant?: "primary" | "danger" };

function Button({ variant = "primary", ...rest }: ButtonProps) {
  return <button data-variant={variant} {...rest} />;   // onClick, disabled, type va h.k. avtomatik
}
```

`React.ReactNode` istalgan render qilinadigan narsa (matn, element, massiv, `null`). `React.ComponentProps<"button">` barcha standart atributlarni beradi, ularni qo'lda yozish shart emas.

### 2.8 [Middle] API javoblarini tiplashda TypeScript yetarlimi? Zod nima uchun?

TypeScript tiplari faqat **kompilyatsiya paytida** mavjud, runtime'da yo'qoladi. Server boshqa shakldagi ma'lumot qaytarsa, TypeScript buni sezmaydi va xato ilovaning boshqa joyida chiqadi. **Zod** ma'lumotni **runtime'da tekshiradi** va tipni sxemadan avtomatik chiqaradi:

```ts
const UserSchema = z.object({ id: z.number(), name: z.string(), role: z.enum(["admin", "user"]) });
type User = z.infer<typeof UserSchema>;          // tip qo'lda yozilmaydi

const result = UserSchema.safeParse(json);
if (!result.success) console.error(result.error);
```

Bitta sxema = **validatsiya + tip**. Tashqi ma'lumot kiradigan joylarda (API, forma, URL, localStorage) foydali.

---

## 3. Zod va React Hook Form

### 3.1 [Junior] Zod nima va qayerda ishlatgansiz?

TypeScript uchun **schema validatsiya** kutubxonasi. Gross'da telefon raqamini tekshirishda (rezyume 3.6-savol), EasyCoddy'da formalar bilan `(o'z misolingiz)`.

### 3.2 [Strong junior] React Hook Form bilan Zod qanday birlashtiriladi?

```tsx
import { zodResolver } from "@hookform/resolvers/zod";

const schema = z.object({
  email: z.string().email("Email noto'g'ri"),
  password: z.string().min(8, "Kamida 8 ta belgi"),
});
type FormData = z.infer<typeof schema>;

const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
  resolver: zodResolver(schema),
});
```

Afzalligi: validatsiya qoidalari **bitta sxemada**, forma tipi avtomatik, xuddi shu sxemani backend (Node.js) ham ishlatishi mumkin.

### 3.3 [Strong junior] Parol tasdiqlash kabi maydonlararo validatsiya Zod'da qanday yoziladi?

```ts
const schema = z
  .object({ password: z.string().min(8), confirm: z.string() })
  .refine((d) => d.password === d.confirm, {
    message: "Parollar mos emas",
    path: ["confirm"],      // xato qaysi maydonda ko'rinishi
  });
```

### 3.4 [Middle] React Hook Form nega `useState` bilan yozilgan formadan tezroq?

Input'lar **uncontrolled** (`ref` orqali), shuning uchun har harf yozilganda butun forma qayta render **bo'lmaydi**. Faqat kerakli joy (xato xabari, `watch` qilingan maydon) yangilanadi. Katta formalarda (o'nlab maydon) bu sezilarli farq qiladi. Controlled UI kutubxona component'lari (MUI, select) uchun `Controller` ishlatiladi.

---

## 4. React ekotizimi: tanlov savollari

### 4.1 [Strong junior] Rezyumeda Redux, Redux Toolkit, Zustand va Context API bor. Ularni qanday ajratasiz?

| Vosita | Qachon |
|---|---|
| **Context API** | Kam o'zgaradigan global qiymat: tema, til, auth foydalanuvchisi |
| **Zustand** | O'rta hajmdagi client state, kam kod, selector bilan samarali |
| **Redux Toolkit** | Katta ilova, ko'p domen, katta jamoa, kuchli DevTools |
| **TanStack Query** | Server state (bu ro'yxatdagilardan alohida) |

"Redux" va "Redux Toolkit" alohida yozilgan: intervyuer klassik Redux (`createStore`, qo'lda reducer, `connect`) va RTK (`createSlice`, Immer) farqini so'rashi mumkin (`section-20` fayli).

### 4.2 [Middle] Bitta loyihada bularning hammasini ishlatish to'g'rimi?

Yo'q, odatda **bitta client state** vositasi tanlanadi. Server ma'lumoti TanStack Query'da, oz miqdordagi global UI state Zustand **yoki** Context'da. Bir loyihada Redux va Zustand birga bo'lsa, qaysi state qayerda ekanini tushunish qiyinlashadi. Rezyumedagi ro'yxat **turli loyihalardagi** tajribani ko'rsatadi, buni intervyuda aytib o'ting.

### 4.3 [Junior] React Router va Next.js routing farqi?

React Router: route'lar **kodda** e'lon qilinadi (`createBrowserRouter`), SPA ichida ishlaydi. Next.js: **fayl tizimi asosida** (`app/menu/page.tsx` → `/menu`), server bilan birga ishlaydi, layout'lar ham fayllar orqali.

---

## 5. Next.js: chuqurroq savollar

### 5.1 [Junior] App Router'da maxsus fayllar qaysilar?

| Fayl | Vazifa |
|---|---|
| `page.tsx` | Route sahifasi (URL shu bilan ochiladi) |
| `layout.tsx` | Umumiy qobiq, navigatsiyada **qayta render bo'lmaydi** |
| `loading.tsx` | Yuklanish holati (avtomatik `Suspense`) |
| `error.tsx` | Xato chegarasi (client component bo'lishi kerak) |
| `not-found.tsx` | 404 sahifa |
| `route.ts` | API endpoint (route handler) |

### 5.2 [Junior] Dinamik route'lar qanday yoziladi?

`app/cabins/[cabinId]/page.tsx` → `/cabins/12`. Parametr sahifaga `params` orqali keladi. Statik generatsiya uchun barcha qiymatlar `generateStaticParams` bilan oldindan beriladi.

### 5.3 [Strong junior] Server Actions nima?

Serverda bajariladigan funksiya, `"use server"` direktivasi bilan. Formadan to'g'ridan-to'g'ri chaqiriladi, alohida API endpoint yozish shart emas:

```tsx
async function createBooking(formData: FormData) {
  "use server";
  // bazaga yozish, validatsiya
  revalidatePath("/bookings");
}

<form action={createBooking}>...</form>
```

React Router action'lariga o'xshaydi (`section-22`), lekin serverda ishlaydi. Muhim: server action **ochiq endpoint**, ichida foydalanuvchi va huquqlarni albatta tekshirish kerak.

### 5.4 [Junior] SEO uchun Next.js nima beradi?

Server'da render qilingan HTML, `metadata` obyekti yoki `generateMetadata` (sahifa bo'yicha `title`, `description`, Open Graph), `sitemap.ts`, `robots.ts`. `next/image` va `next/font` tezlikni oshiradi, bu ham SEO'ga ta'sir qiladi.

### 5.5 [Junior] `next/image` oddiy `<img>` dan nimasi bilan yaxshi?

Rasmni avtomatik zamonaviy formatga (`webp`/`avif`) va ekran o'lchamiga moslab beradi, default holda **lazy loading**, `width`/`height` yoki `fill` majburiy bo'lgani uchun layout sakramaydi (CLS). Muhim rasmlar uchun `priority`.

### 5.6 [Strong junior] Next.js'da middleware nima uchun ishlatiladi?

So'rov sahifaga yetib borishidan **oldin** ishlaydigan kod: login qilmagan foydalanuvchini yo'naltirish, til bo'yicha redirect, header qo'shish. Yengil tekshiruvlar uchun mo'ljallangan, og'ir mantiq uchun emas. Yangi versiyalarda bu fayl nomi o'zgargan (`proxy`), loyihadagi versiya hujjatini tekshiring.

### 5.7 [Middle] Client component'ni qanday qilib server component ichida ishlatasiz va aksincha?

Server component client component'ni **import qilib** render qila oladi va unga serializatsiya qilinadigan props (son, string, oddiy obyekt) beradi, **funksiya bera olmaydi**. Client component server component'ni **import qila olmaydi**, lekin uni `children` sifatida qabul qila oladi (composition). Shu tufayli interaktiv qobiq (masalan, modal) ichida server'da render qilingan kontent bo'lishi mumkin.

---

## 6. HTML5

### 6.1 [Junior] Semantik HTML nima va nega muhim?

Mazmunni ifodalovchi teglar: `header`, `nav`, `main`, `section`, `article`, `aside`, `footer`, `button`, `form`, `label`. Foydalari: **accessibility** (screen reader tuzilmani tushunadi), **SEO**, kodni o'qish osonligi. Har narsani `div` qilish antipattern.

### 6.2 [Junior] Nega bosiladigan narsa uchun `div onClick` emas, `button`?

`button` klaviatura bilan fokus oladi, Enter/Space bilan ishlaydi va screen reader uni "tugma" deb o'qiydi. `div` da bularning hammasini qo'lda (`tabIndex`, `role`, klaviatura handler) yozish kerak. Sahifaga o'tish uchun esa `a` (`Link`).

### 6.3 [Junior] Accessibility (a11y) uchun nimalarga e'tibor berasiz?

Rasmlarga `alt`, input'ga `label` (`htmlFor`/`id`), yetarli rang kontrasti, klaviatura bilan to'liq boshqarish, ko'rinadigan focus holati, semantik teglar, kerak bo'lsa `aria-label`/`aria-expanded`. Tekshirish: Lighthouse, klaviatura bilan sahifani aylanib chiqish.

### 6.4 [Strong junior] `<script>` dagi `defer` va `async` farqi?

Ikkalasi ham skriptni HTML tahliliga **parallel** yuklaydi. `async` yuklangan zahoti bajariladi (tartib kafolatlanmaydi, analytics uchun). `defer` HTML to'liq tahlil qilingandan keyin, **tartib bilan** bajariladi. Vite build qilgan skriptlar `type="module"`, ular default holda `defer` kabi ishlaydi.

### 6.5 [Junior] HTML forma validatsiyasi qanday atributlar bilan ishlaydi?

`required`, `minlength`, `maxlength`, `min`, `max`, `pattern`, `type="email"`/`"tel"`/`"number"`. Brauzer o'zi tekshiradi, lekin bu **faqat birinchi qatlam**: JS validatsiyasi (Zod) va **backend** tekshiruvi ham kerak.

---

## 7. CSS3

### 7.1 [Junior] Box model nima? `box-sizing: border-box` nima qiladi?

Element o'lchami: **content + padding + border + margin**. Default (`content-box`) da `width` faqat content'ga tegishli, padding va border qo'shilib element kattalashadi. `border-box` da `width` padding va border'ni ham o'z ichiga oladi, shuning uchun o'lchamlarni hisoblash oson. Deyarli har loyihada global qilinadi.

### 7.2 [Junior] Flexbox va Grid farqi? Qachon qaysi biri?

**Flexbox**: **bir o'lchamli** (qator yoki ustun), elementlar o'z kontentiga qarab joylashadi: navbar, tugmalar qatori, markazlash. **Grid**: **ikki o'lchamli** (qator va ustun birga): sahifa layout'i, kartochkalar to'ri, dashboard. Ko'pincha birga ishlatiladi: sahifa Grid, ichidagi qismlar Flex.

### 7.3 [Junior] `position` qiymatlari farqi?

`static` (default), `relative` (o'z joyiga nisbatan siljiydi, absolute child'lar uchun asos), `absolute` (eng yaqin positioned ota-elementga nisbatan, oqimdan chiqadi), `fixed` (ekranga nisbatan), `sticky` (scroll'da ma'lum joyga yetganda yopishib qoladi).

### 7.4 [Strong junior] `z-index` nega ba'zan ishlamaydi? Stacking context nima?

`z-index` faqat **bir stacking context ichida** solishtiriladi. Ota-elementda `transform`, `opacity < 1`, `filter`, `position` + `z-index` kabi xossalar yangi stacking context yaratadi: ichidagi element `z-index: 9999` bo'lsa ham, ota-elementdan tashqariga chiqa olmaydi. Modal va dropdown'lar uchun shuning uchun **portal** (`document.body`) ishlatiladi.

### 7.5 [Junior] `px`, `em`, `rem`, `%`, `vh`, `dvh` farqi?

`px` qat'iy; `rem` root (`html`) shrift o'lchamiga nisbatan (foydalanuvchi sozlamasini hurmat qiladi, eng ko'p tavsiya etiladi); `em` ota-element shriftiga nisbatan (ichma-ich kattalashib ketishi mumkin); `%` ota-elementga nisbatan; `vh` ekran balandligi; `dvh` mobil brauzer toolbar'ini hisobga oladigan **dinamik** balandlik.

### 7.6 [Junior] Elementni markazga qanday qo'yasiz?

```css
.parent { display: flex; justify-content: center; align-items: center; }
/* yoki */
.parent { display: grid; place-items: center; }
```

### 7.7 [Strong junior] CSS animatsiyada performance uchun qaysi xossalar ishlatiladi?

`transform` va `opacity`: ular faqat **compositing** bosqichida ishlaydi, layout va paint'ni qayta hisoblamaydi. `width`, `height`, `top`, `left`, `margin` animatsiyasi layout'ni har kadrda qayta hisoblatadi va "qotish" (jank) keltiradi.

### 7.8 [Junior] CSS'da nomlash konvensiyasi (masalan, BEM) nima uchun?

**BEM** (`block__element--modifier`, masalan `card__title--active`): global CSS'da class to'qnashuvini oldini olish va tuzilmani nomdan tushunish uchun. CSS Modules, Tailwind va styled-components'da to'qnashuv avtomatik hal qilinadi, shuning uchun BEM'ga ehtiyoj kamayadi.

---

## 8. Framer Motion

### 8.1 [Junior] Framer Motion asosiy tushunchalari qanday?

```tsx
import { motion, AnimatePresence } from "framer-motion";   // yangi nomi: "motion/react"

<motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0 }} />
```

`initial`, `animate`, `exit` holatlari, `transition` (davomiyligi, spring), `variants` (holatlarni nomlab, child'larga tarqatish, `staggerChildren`), `whileHover`/`whileTap`.

Eslatma: kutubxona hozir **Motion** deb nomlangan (`motion` paketi), API asosan bir xil.

### 8.2 [Strong junior] `AnimatePresence` nima uchun kerak?

React element DOM'dan **darhol** olib tashlanadi, shuning uchun chiqish animatsiyasi ko'rinmaydi. `AnimatePresence` element olib tashlanayotganini kuzatib, avval `exit` animatsiyasini o'ynaydi, keyin DOM'dan o'chiradi. Har bir child'da barqaror `key` bo'lishi kerak.

### 8.3 [Middle] Animatsiya qachon zarar qiladi?

Juda ko'p yoki uzun animatsiyalar foydalanuvchini sekinlashtiradi, layout xossalarini animatsiya qilish performance'ni tushiradi, harakatga sezgir foydalanuvchilar uchun `prefers-reduced-motion` ni hurmat qilish kerak (`useReducedMotion`). Animatsiya ma'noli bo'lishi kerak: holat o'zgarishini ko'rsatish, e'tiborni yo'naltirish.

---

## 9. UI kutubxonalari (MUI, DaisyUI)

### 9.1 [Strong junior] MUI component'ini loyiha dizayniga qanday moslaysiz?

Global darajada **theme** (`createTheme`: ranglar, shrift, component default'lari), bitta joyda `sx` prop yoki `styled()`. Har joyda alohida override yozish o'rniga theme'ni sozlash: dizayn izchil qoladi.

### 9.2 [Middle] Tayyor UI kutubxona tanlashda nimalarga qaraysiz?

Dizaynni moslash qulayligi, bundle hajmi, accessibility (klaviatura, ARIA), TypeScript qo'llovi, hujjat va hamjamiyat, loyiha dizayn tizimiga mosligi. Noyob dizayn kerak bo'lsa: **headless** kutubxona (Radix UI, Headless UI) yoki shadcn/ui; tez ichki panel kerak bo'lsa: MUI.

---

## 10. Build vositalari va npm

### 10.1 [Junior] `package.json` va `package-lock.json` farqi?

`package.json`: loyiha ma'lumoti, skriptlar va **ruxsat etilgan versiya diapazonlari**. `package-lock.json`: o'rnatilgan har bir paketning (ichki bog'liqliklari bilan) **aniq versiyasi**. Lock fayl **git'ga qo'shiladi**, shunda har bir dasturchi va server aynan bir xil versiyalarni o'rnatadi.

### 10.2 [Junior] `dependencies` va `devDependencies` farqi?

`dependencies`: ilova **ishlashi** uchun kerak (react, axios). `devDependencies`: faqat **ishlab chiqish va build** uchun (vite, eslint, prettier, typescript, json-server). Frontend'da build natijasi baribir bundle bo'ladi, lekin bo'linish loyihani tushunish va server'da o'rnatish uchun muhim.

### 10.3 [Junior] Versiyadagi `^` va `~` belgilari nimani anglatadi?

Semver: `MAJOR.MINOR.PATCH`. `^1.2.3`: `2.0.0` dan pastgacha (minor va patch yangilanishlar). `~1.2.3`: `1.3.0` dan pastgacha (faqat patch). MAJOR o'zgarishi **breaking change** degani, shuning uchun avtomatik o'rnatilmaydi.

### 10.4 [Strong junior] `npm install` va `npm ci` farqi?

`npm ci` faqat **lock faylga** qarab o'rnatadi, `node_modules` ni o'chirib, toza o'rnatadi va `package.json` bilan mos kelmasa xato beradi. CI/CD va deploy uchun ishlatiladi (tez va takrorlanuvchan). `npm install` esa lock faylni yangilashi mumkin.

### 10.5 [Strong junior] Webpack'ning asosiy tushunchalari qanday?

**Entry** (boshlang'ich fayl), **output** (natija), **loaders** (fayllarni o'girish: `babel-loader`, `css-loader`), **plugins** (kengroq vazifalar: HTML yaratish, minify, env), **mode** (development/production). Code splitting dinamik `import()` bilan, **tree shaking** ESM bilan ishlaydi.

### 10.6 [Middle] Tree shaking nima va qachon ishlamaydi?

Ishlatilmagan export'larni bundle'dan olib tashlash. Ishlamaydigan holatlar: CommonJS modullar, **side effect**li modullar, butun kutubxonani import qilish (`import _ from "lodash"` o'rniga `import debounce from "lodash-es/debounce"`). Bundle'ni tekshirish: `rollup-plugin-visualizer` yoki `webpack-bundle-analyzer`.

---

## 11. Socket.io va real-time

### 11.1 [Junior] Socket.io va oddiy WebSocket farqi?

Socket.io WebSocket ustidagi kutubxona: **avtomatik qayta ulanish**, **xonalar** (rooms), nomlangan event'lar, **acknowledgement** (javobni kutish), WebSocket ishlamasa HTTP long-polling'ga o'tish. Muhim: Socket.io client oddiy WebSocket server bilan va aksincha **mos kelmaydi**, ikkala tomonda ham Socket.io bo'lishi kerak.

### 11.2 [Strong junior] Socket.io'da "room" nima? O'yin xonalarida qanday ishlatiladi?

Server socket'larni **guruhlarga** ajratadi va xabarni faqat shu guruhga yuboradi (`io.to(roomId).emit(...)`). Har bir o'yin xonasi alohida room: bir xonadagi hodisa boshqa xonadagi o'yinchilarga bormaydi. Frontend xonaga kirish uchun event yuboradi (`socket.emit("room:join", roomId)`), qolganini server boshqaradi.

### 11.3 [Middle] Ulanish uzilganda foydalanuvchi tajribasini qanday saqlaysiz?

Ulanish holatini UI'da ko'rsatish (`connect`, `disconnect`, `reconnect` event'lari), qayta ulanganda **holatni serverdan qayta olish** (uzilish paytida o'tkazib yuborilgan hodisalar uchun), foydalanuvchi harakatlarini ulanish tiklanguncha bloklash yoki navbatga qo'yish. Autentifikatsiya: token ulanish paytida (`auth` opsiyasi) beriladi, server uni tekshiradi.

---

## 12. Firebase va Supabase

### 12.1 [Strong junior] Firebase'da xavfsizlik qanday ta'minlanadi?

**Security Rules**: Firestore va Storage uchun qoidalar (Supabase'dagi RLS'ga o'xshash). Masalan, foydalanuvchi faqat o'z hujjatini o'qiy oladi (`request.auth.uid == resource.data.userId`). Firebase config kalitlari frontend'da ochiq bo'lishi normal, xavfsizlik qoidalarda.

### 12.2 [Middle] Real-time ma'lumot kerak bo'lsa, Firebase va Supabase'da qanday qilinadi?

Firebase: `onSnapshot` bilan hujjat yoki kolleksiyaga obuna bo'lish. Supabase: **Realtime** kanallari bilan jadval o'zgarishlariga obuna. React'da ikkalasi ham effect ichida obuna bo'lib, **cleanup**da obunani bekor qilish kerak. Kelgan ma'lumot React Query keshiga `setQueryData` bilan yozilishi mumkin.

---

## 13. Git, GitHub, GitLab

### 13.1 [Junior] `git stash`, `git cherry-pick`, `git revert` va `git reset` nima uchun?

- `stash`: tugatilmagan o'zgarishlarni vaqtincha saqlab qo'yish (boshqa branch'ga o'tish uchun)
- `cherry-pick`: boshqa branch'dagi **bitta commit**ni joriy branch'ga ko'chirish
- `revert`: commit'ni **bekor qiluvchi yangi commit** yaratadi (tarix saqlanadi, umumiy branch'da xavfsiz)
- `reset`: branch'ni oldingi commit'ga qaytaradi (tarixni o'zgartiradi, push qilingan umumiy branch'da xavfli)

### 13.2 [Junior] Yaxshi commit xabari qanday bo'ladi?

Qisqa, buyruq shaklida, **nima va nega** o'zgarganini aytadi. Ko'p jamoalarda **Conventional Commits**: `feat: add cart undo`, `fix: prevent duplicate OSAGO requests`, `refactor: extract useCabins hook`. Bitta commit bitta mantiqiy o'zgarish.

### 13.3 [Junior] `.gitignore` ga nimalar qo'shiladi?

`node_modules`, build natijasi (`dist`, `.next`), muhit o'zgaruvchilari fayli (`.env`, `.env.local`), log'lar, editor sozlamalari. **Sirli kalit** tasodifan push qilinsa, uni `.gitignore` ga qo'shish yetarli emas: tarixda qoladi, kalitni **yangilash** (rotate) kerak.

### 13.4 [Strong junior] Branching strategiyalari: Git Flow va trunk-based farqi?

**Git Flow**: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`, uzoq yashaydigan branch'lar, aniq release jarayoni. **Trunk-based**: hamma kichik branch'larda ishlab, tez-tez `main` ga merge qiladi, tugallanmagan funksiyalar **feature flag** bilan yashiriladi. Zamonaviy jamoalar va CI/CD'da trunk-based ko'proq.

### 13.5 [Strong junior] CI/CD nima? GitHub Actions yoki GitLab CI'da frontend uchun nimalar tekshiriladi?

**CI**: har bir push yoki merge request'da avtomatik tekshiruv. Frontend uchun: `npm ci`, `lint`, `type-check` (`tsc --noEmit`), testlar, `build`. **CD**: tekshiruvdan o'tgach avtomatik deploy (Vercel/Netlify buni o'zi qiladi). Foyda: xato `main` ga tushishidan oldin ushlanadi.

---

## 14. Figma

### 14.1 [Junior] Figma'dan dasturchi uchun nimalarni olasiz?

Dev Mode/Inspect'dan o'lcham, bo'shliq, rang, shrift, radius, soya; ikonka va rasmlarni eksport (ikonka uchun **SVG**, rasm uchun `webp`); component **variant**larini (React props'iga mos keladi). **Auto Layout** CSS Flexbox'ga to'g'ridan-to'g'ri mos keladi (yo'nalish, gap, padding, alignment).

### 14.2 [Strong junior] Dizaynda yetishmaydigan holatlar bo'lsa nima qilasiz?

Hover, focus, loading, bo'sh holat, xato, uzun matn, mobil ko'rinish ko'pincha dizaynda yo'q. Ularni **ish boshlashdan oldin** dizayner bilan aniqlashtiraman yoki mavjud dizayn tizimiga mos holda o'zim qilib, tasdiqlatib olaman.

---

## 15. Deploy: Netlify va Vercel

### 15.1 [Junior] Vite ilovasini Netlify yoki Vercel'ga qanday deploy qilasiz?

GitHub repo'ni ulash, build buyrug'i (`npm run build`) va natija papkasini (`dist`) ko'rsatish, environment variable'larni panelga qo'shish, SPA uchun barcha yo'llarni `index.html` ga yo'naltirish (Netlify `_redirects`, Vercel `vercel.json` rewrites). Har bir push avtomatik deploy bo'ladi.

### 15.2 [Strong junior] Preview deployment nima?

Har bir pull/merge request uchun **alohida vaqtinchalik URL**'da ilova versiyasi yaratiladi. Reviewer va dizayner o'zgarishni `main` ga merge qilishdan oldin jonli ko'ra oladi. Production, preview va development uchun environment variable'lar alohida beriladi.

### 15.3 [Strong junior] Frontend `.env` faylidagi kalitlar xavfsizmi?

**Yo'q.** `VITE_` (yoki Next.js'da `NEXT_PUBLIC_`) bilan boshlangan o'zgaruvchilar build paytida **bundle'ga yoziladi** va brauzerda istalgan kishi ko'ra oladi. Frontend'ga faqat ochiq bo'lishi mumkin bo'lgan qiymatlar qo'yiladi (API manzili, Supabase publishable kaliti). Maxfiy kalitlar (AI API kaliti, secret key) faqat **server** tomonda (serverless function, backend).

### 15.4 [Middle] Vercel va Netlify'dan qaysi birini tanlaysiz?

Ikkalasi ham statik saytlar va SPA uchun juda yaxshi: avtomatik deploy, preview, CDN, serverless function'lar. **Next.js** uchun Vercel eng to'liq qo'llovni beradi (Next.js'ni Vercel yaratgan). Netlify formalar, redirect qoidalari va Next.js'dan boshqa framework'lar uchun qulay. Tanlov ko'pincha jamoa odati va narxga bog'liq.
