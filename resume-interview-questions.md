# Rezyume bo'yicha texnik intervyu savollari

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

> Bu savollar rezyumedagi **har bir bandga** tayanadi: intervyuer rezyumeni o'qib, aynan shu joylardan so'raydi. Loyiha savollarida `(o'z tafsilotingiz)` deb belgilangan joylarga **haqiqiy** misolingizni qo'ying: intervyuer "qanday?" deb chuqurroq so'raydi, umumiy javob tez ochilib qoladi.
>
> React, Redux, React Query, Router va pattern'lar bo'yicha batafsil savollar `react-course/` papkasidagi fayllarda. Bu yerda ular takrorlanmaydi.

---

## 1. Umumiy va tajriba haqida

### 1.1 [Junior] O'zingizni tanishtiring (1–2 daqiqa)

Tuzilma: **kimsiz → nima qilyapsiz → eng kuchli 1–2 natija → nima qidiryapsiz**.

> "Men Bobur, frontend dasturchiman, asosiy stack'im React va TypeScript. Hozir Newton IT Academy'da bolalar uchun EasyCoddy ta'lim platformasining frontend'ini noldan qayta quryapman: React, TypeScript, Tailwind, TanStack Query va Socket.io bilan real-time o'yin xonalari. Undan oldin Gross Insurance'da Nuxt 3 va TypeScript bilan gross.uz ustida katta jamoada ishladim, GitLab merge request va code review jarayonidan o'tdim. U yerda OSAGO kalkulyatoridagi takroriy API so'rovlarini topib tuzatganman. Freelance'da Bogizilol restorani uchun 4 ta rolli admin panel qurdim. Endi React/TypeScript/Next.js bo'yicha kuchli jamoada mahsulot ustida ishlashni xohlayman."

### 1.2 [Junior] "2+ yil tajriba" qanday hisoblanadi?

Intervyuer albatta tekshiradi, javob **aniq va halol** bo'lsin: qaysi davrda nima qilganingizni ayting (Coddycamp: 2025-dan mentor va dasturchi; Gross: 2026, 4 oylik shartnoma; Newton: 2026-dan; oradagi freelance loyihalar). "Kommersiya tajribasi taxminan X oy, freelance va mentorlik bilan jami 2 yildan ortiq" kabi bo'lib aytish ishonchli eshitiladi.

### 1.3 [Junior] Nega Gross Insurance'dan ketdingiz?

> "U yerda 4 oylik shartnoma asosida ishladim, shartnoma muddati tugadi. Bu davrda katta kod bazasida ishlash, code review va sprint jarayonini o'rgandim. Hozir React ekotizimiga to'liq o'tib, shu yo'nalishda o'sishni xohlayman."

Salbiy gap aytmang, faktni va o'rganganingizni ayting.

### 1.4 [Strong junior] Eng qiyin bug va uni qanday hal qilganingiz haqida gapiring

**STAR** tuzilmasida (Situation, Task, Action, Result). Eng yaxshi misol: OSAGO kalkulyatori (3.3-savol).

> "Vaziyat: kalkulyatorda bitta o'zgarishda serverga bir nechta bir xil so'rov ketardi. Vazifa: sababini topish va keraksiz so'rovlarni olib tashlash. Harakat: DevTools Network tabida takrorlanishni ko'rdim, keyin kodda bitta forma o'zgarishi bir nechta reaktiv mutatsiyaga bo'linib, har biri watcher'ni alohida ishga tushirayotganini topdim. Yangilanishlarni bitta mutatsiyaga birlashtirdim (`Object.assign`), so'rovni debounce qildim va majburiy maydonlar tekshiruvidagi xatoni tuzatdim. Natija: har o'zgarishda faqat bitta so'rov ketadigan bo'ldi."

### 1.5 [Junior] Figma dizaynini kodga qanday o'girasiz?

1. Dizaynni **component'larga bo'laman** (takrorlanadigan qismlarni topaman)
2. **Design tokens**ni olaman: ranglar, shriftlar, bo'shliqlar, radius (CSS variables yoki Tailwind config)
3. Avval **mobile** ko'rinish, keyin katta ekranlar (mobile-first)
4. Figma'dagi o'lchamlarni Inspect/Dev Mode orqali olaman, lekin qat'iy piksel emas, **moslashuvchan** birliklar (`rem`, `%`, `clamp`)
5. Holatlar: hover, focus, disabled, loading, empty, error, ular ko'pincha dizaynda yo'q, dizayner bilan aniqlashtiraman

---

## 2. Newton IT Academy: EasyCoddy

### 2.1 [Junior] Nega frontend noldan qayta qurilmoqda?

`(o'z tafsilotingiz)`. Odatiy sabablar: eski kodda TypeScript yo'q, struktura kengaytirishga yaramaydi, performance yoki dizayn yangilanishi. Javobda **sabab → qaror → natija** bo'lsin va "eski kod yomon edi" deb emas, "yangi talablarga mos kelmasdi" deb ayting.

### 2.2 [Strong junior] Socket.io'ni React bilan qanday ulaysiz? Qanday xatolar bo'ladi?

```tsx
// socket.ts: ilova bo'ylab BITTA ulanish
import { io } from "socket.io-client";
export const socket = io(import.meta.env.VITE_SOCKET_URL, { autoConnect: false });

// Component yoki custom hook
useEffect(() => {
  socket.connect();
  function onPlayerMoved(data: PlayerMove) { /* state yangilash */ }
  socket.on("player:moved", onPlayerMoved);

  return () => {
    socket.off("player:moved", onPlayerMoved);   // listener'ni AYNAN o'sha funksiya bilan olib tashlash
    socket.disconnect();
  };
}, []);
```

Tipik xatolar:

- **Har renderda yangi ulanish** yaratish (socket component ichida `io()` chaqirilsa)
- `off` qilinmagan listener'lar: handler **ikki-uch marta** ishlaydi (Strict Mode'da ayniqsa ko'rinadi)
- Listener ichida **eski (stale) state** (closure): updater funksiya (`setX(prev => ...)`) yoki `useRef` kerak
- Qayta ulanish (reconnect) va ulanish holatini UI'da ko'rsatmaslik

### 2.3 [Strong junior] Typing Race kabi real-time o'yinda state qayerda turishi kerak?

**Server haqiqat manbai** (authoritative) bo'lishi kerak: kim qancha yozdi, kim birinchi tugatdi, vaqt, natijalarni server hisoblaydi. Client faqat **hodisalarni** yuboradi (masalan, progress) va server yuborgan holatni ko'rsatadi. Aks holda foydalanuvchi client kodini o'zgartirib "yutib" qo'yishi mumkin.

Performance uchun: har bir tugma bosishni emas, progressni **throttle** qilib (masalan, har 100–200 ms) yuborish; lokal input esa darhol ko'rinadi.

### 2.4 [Middle] WebSocket, polling va Server-Sent Events: qachon qaysi biri?

| | Polling | SSE | WebSocket |
|---|---|---|---|
| Yo'nalish | Client so'raydi | Server → client | Ikki tomonlama |
| Kechikish | Interval'ga bog'liq | Past | Past |
| Murakkablik | Juda oddiy | Oddiy (HTTP) | Yuqoriroq (ulanish, reconnect) |
| Misol | Buyurtma holati har 30 sek | Bildirishnomalar, live feed | O'yin xonasi, chat |

O'yin xonalari ikki tomonlama va past kechikish talab qiladi, shuning uchun WebSocket (Socket.io) to'g'ri tanlov. Holatni kamdan-kam yangilash yetarli bo'lsa, polling (React Query `refetchInterval`) ancha sodda.

### 2.5 [Strong junior] TanStack Query, Zustand va React Hook Form'ni qanday bo'lib ishlatasiz?

- **TanStack Query**: **server state** (foydalanuvchi profili, darslar, natijalar): kesh, loading, error, invalidation
- **Zustand**: **client state** (joriy o'yin xonasi, UI holati, socket'dan keladigan tez o'zgaruvchi holat)
- **React Hook Form** (+ Zod): **forma state** (login, ro'yxatdan o'tish, sozlamalar)

Qoida: bitta ma'lumotni ikki joyda saqlamaslik. Masalan, server ma'lumotini Zustand'ga ko'chirmaslik.

### 2.6 [Middle] Socket'dan kelgan yangilanishni React Query keshi bilan qanday moslaysiz?

Ikki yo'l:

```ts
socket.on("leaderboard:updated", (data) => {
  queryClient.setQueryData(["leaderboard", roomId], data);           // to'g'ridan-to'g'ri yangilash
  // yoki
  queryClient.invalidateQueries({ queryKey: ["leaderboard", roomId] }); // qayta yuklash
});
```

Event to'liq yangi ma'lumot bersa, `setQueryData` (qo'shimcha so'rov yo'q); faqat "nimadir o'zgardi" signali bo'lsa, `invalidateQueries`.

### 2.7 [Strong junior] Zustand store qanday yoziladi va Redux'dan farqi nima?

```ts
import { create } from "zustand";

type RoomState = { roomId: string | null; players: Player[]; joinRoom: (id: string) => void };

export const useRoomStore = create<RoomState>((set) => ({
  roomId: null,
  players: [],
  joinRoom: (id) => set({ roomId: id }),
}));

// Faqat kerakli qismni olish: boshqa qismlar o'zgarsa re-render bo'lmaydi
const roomId = useRoomStore((state) => state.roomId);
```

Redux'dan farqi: Provider kerak emas, action/reducer boilerplate yo'q, store bitta funksiya, selector bilan ishlaydi. Redux Toolkit esa kattaroq jamoalarda tuzilma, DevTools, middleware va qat'iy qoidalar beradi.

### 2.8 [Strong junior] Backend bilan API kelishuvini (contract) qanday boshqarasiz?

- Endpoint, so'rov va javob shakli oldindan **kelishiladi** (Swagger/OpenAPI, yoki umumiy hujjat)
- Frontend'da javob **tiplari** (TypeScript) yoziladi
- Muhim joylarda javob **Zod** bilan tekshiriladi: backend o'zgarsa, xato darhol ko'rinadi
- Backend tayyor bo'lmaganda **mock** ma'lumot bilan ishlash (MSW yoki oddiy mock funksiya)

### 2.9 [Middle] 3D Labyrinth kabi o'yinda React render'ini qanday nazorat qilasiz?

`(qaysi texnologiya ishlatilganini ayting: three.js, react-three-fiber, canvas)`. Umumiy qoida: **har kadrda** (60 fps) o'zgaradigan qiymatlarni (pozitsiya, kamera) **React state'ida saqlamaslik**, aks holda har kadrda butun daraxt qayta render bo'ladi. Ular `useRef` yoki o'yin dvigatelining o'z ichida, `requestAnimationFrame` (yoki `useFrame`) bilan yangilanadi. React state'da faqat kam o'zgaradigan narsalar: o'yin holati (start, finish), ball, natija.

### 2.10 [Junior] Bolalar uchun UI qilishda nimalarga e'tibor berasiz?

Katta bosiladigan maydonlar, sodda matn va ikonkalar, aniq qayta aloqa (rang, animatsiya, ovoz), xatolarni do'stona ko'rsatish, kontrast va o'qilishi oson shriftlar, planshet va sensorli ekranlarda ishlashi.

---

## 3. Gross Insurance (Nuxt 3, Vue 3, TypeScript)

### 3.1 [Junior] Vue va React'ning asosiy farqlari qanday?

| | Vue 3 | React |
|---|---|---|
| Reaktivlik | Proxy asosida: qiymat o'zgarsa, unga bog'liq joy **avtomatik** yangilanadi | State o'zgarsa, component **qayta render** bo'ladi |
| Shablon | HTML shablon (`v-if`, `v-for`) | JSX |
| Hisoblangan qiymat | `computed` (avtomatik kesh) | Render ichida hisoblash yoki `useMemo` |
| Kuzatish | `watch`, `watchEffect` | `useEffect` |
| Global state | Pinia | Redux, Zustand, Context |
| Framework | Nuxt | Next.js |

Vue'da to'g'ridan-to'g'ri mutatsiya (`state.count++`) normal, React'da esa state immutable yangilanadi.

### 3.2 [Strong junior] Nuxt 3 va Next.js farqi va o'xshashligi nima?

O'xshash: ikkalasi ham **SSR/SSG**, **fayl asosidagi routing**, layout'lar, server API (Nuxt server routes, Next route handlers) beradi. Farq: Nuxt'da ma'lumot olish `useFetch`/`useAsyncData` bilan (server va client'da), Next.js App Router'da esa **Server Components** asosiy model: component serverda ishlaydi va ma'lumotni to'g'ridan-to'g'ri oladi, interaktiv qismlar `"use client"` bilan ajratiladi. Nuxt tajribasi SSR, hydration va SEO tushunchalarini tushunishga yordam beradi.

### 3.3 [Strong junior] OSAGO kalkulyatoridagi takroriy so'rovlar muammosini batafsil tushuntiring

**Sabab:** bitta foydalanuvchi o'zgarishi reaktiv obyektga **bir necha alohida mutatsiya** sifatida yozilardi. Har bir mutatsiya watcher'ni ishga tushirib, alohida API so'rov yuborardi. Bundan tashqari, majburiy maydonlar (`requiredFields`) tekshiruvi to'g'ri ishlamagan, so'rov hali to'liq bo'lmagan ma'lumot bilan ham ketardi.

**Yechim:**

1. Yangilanishlarni **bitta mutatsiyaga** birlashtirish (`Object.assign` yoki spread bilan bir vaqtda)
2. So'rovni **debounce** qilish (`lodash-es`): ketma-ket o'zgarishlarda faqat oxirgisidan keyin so'rov ketadi
3. `requiredFields` mantiqini tuzatish: maydonlar to'liq bo'lmaguncha so'rov yuborilmaydi

**Natija:** har bir o'zgarishga bitta so'rov.

### 3.4 [Junior] Debounce va throttle farqi nima? Debounce'ni o'zingiz yozib bering

- **Debounce**: hodisalar to'xtagandan **X ms keyin bir marta** bajaradi (qidiruv input'i, kalkulyator)
- **Throttle**: har **X ms da ko'pi bilan bir marta** bajaradi (scroll, resize, o'yin progressi)

```js
function debounce(fn, delay) {
  let timerId;
  return function (...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

### 3.5 [Middle] Debounce takroriy so'rovlar muammosini to'liq hal qiladimi?

Yo'q. Debounce so'rovlar **sonini** kamaytiradi, lekin **race condition**ni hal qilmaydi: ikki so'rov ketib, eskisining javobi kechroq kelsa, eski natija ekranda qoladi. Qo'shimcha kerak:

- **AbortController** bilan oldingi so'rovni bekor qilish
- Yoki javobni faqat **eng oxirgi** so'rovga tegishli bo'lsa qabul qilish (so'rov ID'si bilan)
- React'da TanStack Query buni query key orqali o'zi boshqaradi

### 3.6 [Strong junior] Telefon raqami maskasi va Zod validatsiyasi qanday qilingan?

- **Mask** faqat **UX** uchun: foydalanuvchi `+998 (90) 123-45-67` ko'rinishida ko'radi
- Validatsiyadan oldin qiymat **toza raqamlarga** keltiriladi (`value.replace(/\D/g, "")`)
- **Zod** sxema format va uzunlikni tekshiradi:

```ts
const phoneSchema = z
  .string()
  .transform((v) => v.replace(/\D/g, ""))
  .refine((v) => /^998\d{9}$/.test(v), { message: "Telefon raqami noto'g'ri" });
```

Muhim: frontend validatsiyasi qulaylik uchun, **backend ham** baribir tekshirishi shart.

### 3.7 [Junior] GitLab'da merge request jarayoni qanday edi? Code review'da nimalarga qaraysiz?

Jarayon: task → yangi branch (`feature/...`) → commit'lar → merge request → CI tekshiruvi → reviewer izohlari → tuzatish → merge.

Review'da qarayman: kod vazifani to'g'ri bajaradimi, edge case'lar (bo'sh, xato, loading), nomlar va o'qilishi, takrorlangan kod, keraksiz re-render yoki so'rovlar, `console.log` qolib ketmaganmi, tiplar to'g'rimi.

### 3.8 [Junior] Git'da `merge` va `rebase` farqi nima? Konflikt qanday hal qilinadi?

- **Merge**: ikki branch'ni birlashtiruvchi yangi commit yaratadi, tarix saqlanadi
- **Rebase**: commit'laringizni boshqa branch'ning ustiga ko'chiradi, tarix chiziqli bo'ladi. **Umumiy (shared) branch'da rebase qilinmaydi**, chunki boshqalarning tarixi buziladi

Konflikt: `git status` bilan fayllarni topaman, `<<<<<<<` belgilari orasidagi ikki variantni kod mazmuniga qarab birlashtiraman, keyin ilovani ishga tushirib tekshiraman. Tushunmagan qismni kod muallifi bilan aniqlashtiraman.

### 3.9 [Strong junior] `#gross .btn` kabi CSS specificity muammosi nima edi? Specificity qanday hisoblanadi?

Specificity tartibi: **inline style** > **ID** > **class, atribut, pseudo-class** > **element**. `#gross .btn` (1 ID + 1 class) oddiy `.btn-primary` (1 class) dan kuchli, shuning uchun yangi stillar ishlamay qolardi.

Yechimlar: selektorni kuchaytirish o'rniga **global ID selektorlarini kamaytirish**, CSS Modules yoki scoped styles, `@layer` bilan tartibni boshqarish. `!important` oxirgi chora.

### 3.10 [Junior] Pinia'da global state'ga qanday narsa chiqarilishi kerak?

Bir nechta, bir-biridan uzoq component'larga kerak bo'lgan holat. Masalan, SOS modal oynasining ochiq/yopiq holatini navigatsiya va boshqa sahifalar ochishi kerak edi, shuning uchun u `useMainStore` ga ko'chirildi. Faqat bitta component ishlatadigan holat esa lokal qoladi.

---

## 4. Coddycamp: dasturchi va mentor

### 4.1 [Junior] 1000+ topshiriqni tekshirishda eng ko'p qaysi xatolarni ko'rgansiz?

State'ni to'g'ridan-to'g'ri o'zgartirish, `key` sifatida indeks, `useEffect` dependency'larini noto'g'ri yozish, hamma narsani bitta component'ga yozish, semantik HTML va accessibility'ga e'tibor bermaslik, responsive'ni oxiriga qoldirish. Bu savolga javob sizning **code review** tajribangizni ko'rsatadi.

### 4.2 [Junior] Mentorlik dasturchi sifatida sizga nima berdi?

Murakkab narsani sodda tushuntirish, boshqalarning kodini tez o'qish va xatoni topish, aniq va konstruktiv izoh yozish. Bu jamoada code review va hamkasblar bilan ishlashda bevosita foydali.

---

## 5. Bogizilol (restoran CRM)

### 5.1 [Junior] Loyiha haqida gapirib bering

> "Bogizilol: restoran uchun mijozlar sayti va admin panel. Frontend: React 18, TypeScript, Vite, Tailwind va shadcn/ui; backend: Node.js, Express, MongoDB. To'rtta rol bor: admin, oshpaz, kuryer va mijoz, har birining o'z interfeysi. Mijoz menyudan savatga qo'shib buyurtma beradi, oshpaz oshxona panelida buyurtmalarni ko'radi va holatini yangilaydi, admin kuryerni biriktiradi, kuryer o'z buyurtmalarini marshrut ko'rinishida oladi. Uch tilda (o'zbek, rus, ingliz). Server ma'lumoti TanStack Query bilan, savat Context va localStorage bilan."

### 5.2 [Strong junior] Rolga asoslangan kirish (RBAC) frontend'da qanday qilingan?

Login'dan keyin foydalanuvchi ma'lumoti (`GET /users/:id`) olinadi va rol token bilan birga saqlanadi. `ProtectedRoute` ikki narsani tekshiradi: token bormi va muddati o'tmaganmi, foydalanuvchi roli shu route uchun ruxsat etilganmi:

```tsx
function ProtectedRoute({ allowedRoles, children }: Props) {
  const { token, role } = useAuth();
  if (!token || isExpired(token)) return <Navigate to="/login" replace />;
  if (!allowedRoles.includes(role)) return <Navigate to="/" replace />;
  return children;
}
```

Har rol uchun alohida layout va sahifalar (`/admin`, `/kitchen`, `/courier`).

### 5.3 [Middle] Bu RBAC xavfsizmi? Rolni localStorage'da saqlash muammo emasmi?

Frontend'dagi tekshiruv **faqat UX**: foydalanuvchi localStorage'dagi rolni o'zgartirib boshqa panelni ochishi mumkin. **Haqiqiy himoya backend'da**: har bir endpoint JWT'ni tekshirishi va foydalanuvchining **rolini serverda** aniqlab, ruxsat berishi kerak. Yaxshiroq variant: rolni JWT payload'iga qo'shish (server imzolaydi, o'zgartirib bo'lmaydi), frontend esa uni faqat UI'ni moslash uchun o'qiydi.

Intervyuda buni o'zingiz aytsangiz, xavfsizlikni tushunishingizni ko'rsatadi.

### 5.4 [Strong junior] JWT nima? Qanday tuzilgan?

Uch qism, nuqta bilan ajratilgan: **header.payload.signature**. Header va payload **base64url** kodlangan, **shifrlanmagan**, ya'ni istalgan kishi o'qiy oladi, shuning uchun payload'ga parol yoki maxfiy ma'lumot qo'yilmaydi. Signature server kaliti bilan imzolanadi: payload o'zgartirilsa, imzo mos kelmaydi va server rad etadi.

Frontend'da muddatni tekshirish: payload'dagi `exp` (sekundlarda) `Date.now() / 1000` bilan solishtiriladi, muddati o'tgan bo'lsa token o'chiriladi va login'ga yo'naltiriladi.

### 5.5 [Middle] Tokenni qayerda saqlash kerak: localStorage yoki cookie?

| | localStorage | httpOnly cookie |
|---|---|---|
| JavaScript o'qiy oladimi | Ha | Yo'q |
| XSS xavfi | Yuqori (skript tokenni o'g'irlaydi) | Past |
| CSRF xavfi | Yo'q | Bor (`SameSite`, CSRF token bilan himoyalanadi) |
| Sozlash | Oson | Backend bilan kelishish kerak |

Xavfsizroq variant: **httpOnly, Secure, SameSite** cookie. localStorage ishlatilsa, XSS'dan himoya (kiruvchi ma'lumotni tozalash, `dangerouslySetInnerHTML` dan qochish) muhim.

### 5.6 [Middle] Admin tokenining 90 kunlik muddati to'g'rimi?

Yo'q. Token o'g'irlansa, 90 kun davomida admin huquqi bilan ishlatiladi. To'g'ri yondashuv: **qisqa muddatli access token** (15 daqiqa – bir necha soat) va **refresh token** (httpOnly cookie'da). Access token muddati o'tganda frontend `401` olib, refresh orqali yangisini oladi (Axios interceptor bilan qulay, 8.4-savol). Admin uchun muddat yanada qisqa bo'lishi kerak.

### 5.7 [Strong junior] Loyihada state qanday taqsimlangan?

- **TanStack Query**: menyu, buyurtmalar, statistika (server state, CRUD va invalidation)
- **Context + localStorage**: savat (sahifa yangilansa yo'qolmasligi uchun)
- **Zustand** `(qayerda ishlatganingizni aniq ayting)`
- **Lokal state**: forma, modal, filtr

### 5.8 [Strong junior] Oshxona panelidagi avtomatik yangilanish qanday ishlaydi? WebSocket'dan farqi?

React Query'ning `refetchInterval` (masalan, har N soniyada) **polling** bilan:

```ts
useQuery({ queryKey: ["orders", "kitchen"], queryFn: getOrders, refetchInterval: 15000 });
```

Afzallik: juda sodda, backend o'zgarishi kerak emas. Kamchilik: kechikish interval'ga teng va yangilik bo'lmasa ham so'rov ketadi. Real-time muhim bo'lsa (yangi buyurtma darhol chiqishi), **WebSocket** yoki SSE yaxshiroq.

### 5.9 [Strong junior] Mijoz buyurtmalarini kuzatish uchun ID'lar nega localStorage'da?

Backend mijozga faqat **ID bo'yicha bitta buyurtma** berardi, "mening buyurtmalarim" endpoint'i yo'q edi. Shuning uchun yaratilgan buyurtma ID'lari localStorage'ga saqlanib, har biri alohida so'raladi. Bu **vaqtinchalik yechim**: boshqa qurilmada yoki localStorage tozalansa, buyurtmalar ko'rinmaydi. To'g'risi: backend'da `GET /orders/me` (token bo'yicha) endpoint'i.

### 5.10 [Junior] Ko'p tillilik (i18n) qanday qilingan?

`(qaysi kutubxona yoki usul ishlatganingizni ayting, masalan react-i18next)`. Umumiy tamoyil: matnlar kodda emas, til fayllarida (`uz.json`, `ru.json`, `en.json`), component'larda kalit orqali olinadi (`t("menu.title")`). Tanlangan til localStorage'da saqlanadi. Menyu nomlari kabi **ma'lumotlar bazasidagi** kontent ham ko'p tilli bo'lishi kerak (backend har til uchun maydon beradi).

### 5.11 [Junior] Framer Motion'da qanday animatsiyalar ishlatgansiz?

`motion.div` bilan kirish/chiqish animatsiyalari, ro'yxatlar uchun `AnimatePresence` (element olib tashlanganda chiqish animatsiyasi), `layout` prop bilan joy almashish animatsiyasi. Accessibility uchun `prefers-reduced-motion` ni hurmat qilish (`useReducedMotion`). Faqat `transform` va `opacity` animatsiya qilinadi, chunki ular layout'ni qayta hisoblamaydi.

### 5.12 [Strong junior] Loyiha qanday deploy qilingan? Qanday muammolar bo'lgan?

Frontend Vercel va Netlify'da, backend Render'da, domen `bogizilol.uz`. E'tibor beriladigan joylar:

- **SPA fallback** (barcha yo'llar `index.html` ga), aks holda ichki sahifani yangilaganda 404
- API manzili **environment variable** orqali (`VITE_API_URL`)
- Bepul Render serverining **cold start**i: birinchi so'rov sekin, foydalanuvchiga loading ko'rsatish
- Rasm fayl nomlaridagi katta-kichik harf (Linux serverda farq qiladi)

---

## 6. Boshqa loyihalar

### 6.1 [Junior] Sapphire Transportation'dagi yetkazish narxi kalkulyatori qanday ishlaydi?

Input qiymatlari state'da (controlled), narx esa **derived**: alohida state emas, render ichida hisoblanadi. Input'dan kelgan qiymatlar **songa** aylantiriladi, noto'g'ri qiymatlar tekshiriladi, natija `Intl.NumberFormat` bilan valyuta formatida ko'rsatiladi. `(formula va maydonlarni o'zingiz ayting)`

### 6.2 [Strong junior] EZ Trans Group'dagi "live dispatch status" qanday yangilanadi?

`(haqiqatda qanday qilinganini ayting)`. Agar polling bo'lsa, buni ochiq aytish kerak: "jonli" so'zi WebSocket'ni anglatadi deb o'ylashlari mumkin. Variantlar va farqlari 2.4-savolda.

### 6.3 [Middle] Happy Quiz'dagi AI integratsiyasi: API kalitini qayerda saqlagansiz?

To'g'ri javob: **frontend'da emas**. Brauzerdagi har qanday kalitni (hatto `.env` dagi `VITE_` o'zgaruvchisini) foydalanuvchi ko'ra oladi va o'g'irlay oladi. AI so'rovlari **backend yoki serverless function** (Supabase Edge Function, Vercel/Netlify function) orqali yuboriladi: kalit serverda, frontend faqat o'z endpoint'ingizga murojaat qiladi. Qo'shimcha: rate limit, foydalanuvchi autentifikatsiyasi, javobni streaming bilan ko'rsatish.

Agar kalit hozir frontend'da bo'lsa, intervyudan oldin tuzating va kalitni yangilang (rotate).

### 6.4 [Junior] Spartans AI'dagi progress tracking ma'lumotlari qayerda saqlanadi?

`(localStorage, Supabase yoki Firebase: haqiqatini ayting)`. localStorage bo'lsa, cheklovi: faqat bitta qurilma va brauzerda. Ko'p qurilmada ishlashi uchun backend va autentifikatsiya kerak.

### 6.5 [Strong junior] CBCT Optimizator'da dynamic programming nima uchun ishlatilgan?

Bu **uskunani almashtirish muammosi** (equipment replacement problem): har yili uskunani **saqlab qolish** (ta'mirlash va ekspluatatsiya xarajati yoshi bilan oshadi) yoki **yangisiga almashtirish** (sotib olish narxi minus qoldiq qiymat) kerak. Maqsad: rejalashtirilgan davr uchun **umumiy xarajatni minimallashtirish**.

DP formulasi (soddalashtirilgan):

```
f(t, a) = min(
  saqlash:    xarajat(a) + f(t + 1, a + 1),
  almashtirish: narx − qoldiq(a) + xarajat(0) + f(t + 1, 1)
)
```

bu yerda `t` yil, `a` uskunaning yoshi. Nega DP: **qayta takrorlanadigan kichik masalalar** (bir xil `(t, a)` ko'p marta uchraydi) va **optimal substruktura**. Brute force eksponensial (`2^T`), DP esa `O(T × A)`.

### 6.6 [Junior] Portfolio saytingiz Lovable'da. Uni o'zingiz yozdingizmi?

Halol javob bering: AI vositasi bilan yaratilgan bo'lsa, shuni ayting va **nimani o'zingiz o'zgartirganingizni, qanday qarorlarni o'zingiz qabul qilganingizni** tushuntiring. AI vositalaridan foydalanish bugun normal, lekin intervyuer kodni tushunishingizni tekshiradi.

---

## 7. Texnologiyalar: JavaScript

### 7.1 [Junior] `var`, `let`, `const` farqi nima?

`var`: funksiya doirasi, hoisting'da `undefined`, qayta e'lon qilish mumkin. `let`/`const`: blok doirasi, e'londan oldin ishlatib bo'lmaydi (temporal dead zone). `const` qayta o'zlashtirib bo'lmaydi, lekin obyektning **ichini** o'zgartirish mumkin.

### 7.2 [Junior] Closure nima? Misol keltiring

Funksiya o'zi yaratilgan muhitdagi o'zgaruvchilarni, o'sha muhit tugagandan keyin ham "eslab qoladi".

```js
function createCounter() {
  let count = 0;
  return () => ++count;
}
const counter = createCounter();
counter(); // 1
counter(); // 2
```

React'da: event handler va effect'lar render'dagi qiymatlarni closure orqali ushlaydi, shuning uchun "stale closure" muammosi bo'ladi.

### 7.3 [Strong junior] Event loop qanday ishlaydi? Bu kod nima chiqaradi?

```js
console.log(1);
setTimeout(() => console.log(2), 0);
Promise.resolve().then(() => console.log(3));
console.log(4);
// 1, 4, 3, 2
```

Avval **sinxron** kod (call stack), keyin **microtask**lar (Promise, `queueMicrotask`), keyin **macrotask**lar (`setTimeout`, event'lar). Shuning uchun Promise `setTimeout(0)` dan oldin ishlaydi.

### 7.4 [Junior] `==` va `===` farqi?

`==` turlarni avtomatik o'zgartirib solishtiradi (`"5" == 5` → `true`), `===` turni ham solishtiradi. Har doim `===` ishlatiladi.

### 7.5 [Junior] Promise va async/await: xatolar qanday ushlanadi? `Promise.all` va `Promise.allSettled` farqi?

`async/await` Promise ustidagi qulay sintaksis, xato `try/catch` bilan ushlanadi. `Promise.all` bitta so'rov xato bersa **butunlay reject** bo'ladi; `Promise.allSettled` esa hammasini kutadi va har birining natijasini (muvaffaqiyat yoki xato) qaytaradi. Bir-biriga bog'liq bo'lmagan so'rovlarni **parallel** yuborish uchun ikkalasi ham ketma-ket `await` dan tezroq.

### 7.6 [Junior] Spread operator chuqur nusxa oladimi?

Yo'q, **yuzaki (shallow)** nusxa: ichki obyekt va massivlar havola bo'yicha qoladi. Chuqur nusxa uchun `structuredClone(obj)`. React'da ichki obyektni yangilashda har bir qatlamni spread qilish kerak (yoki Immer).

### 7.7 [Junior] `this` qanday aniqlanadi? Arrow function'da qanday?

Oddiy funksiyada `this` **qanday chaqirilganiga** bog'liq (obyekt metodi sifatida → o'sha obyekt; oddiy chaqiruv → `undefined` strict mode'da). Arrow function'ning o'z `this` i yo'q, u **tashqi muhitnikini** oladi. Shuning uchun class component'larda handler'lar arrow function bilan yozilardi.

### 7.8 [Junior] `map`, `filter`, `reduce` farqi? Qaysilari massivni o'zgartiradi?

`map` har elementni o'zgartirib **yangi massiv**, `filter` shartga mos elementlarning **yangi massivi**, `reduce` bitta **qiymatga** yig'adi. Ular asl massivni o'zgartirmaydi. O'zgartiradiganlar: `push`, `pop`, `splice`, `sort`, `reverse` (React state'da ehtiyot bo'lish kerak; yangilari: `toSorted`, `toReversed`).

---

## 8. Texnologiyalar: TypeScript, Next.js va kutubxonalar

### 8.1 [Junior] TypeScript'da `interface` va `type` farqi?

Ikkalasi ham obyekt shaklini tasvirlaydi. `type` union (`"admin" | "user"`), tuple va murakkab tiplar uchun ham ishlatiladi; `interface` kengaytiriladi (`extends`) va bir xil nom bilan **birlashadi** (declaration merging). Amalda: obyekt va props uchun ikkalasi ham to'g'ri, jamoa kelishuviga qarab tanlanadi.

### 8.2 [Junior] React'da props, state va event'lar qanday tiplanadi?

```tsx
type ButtonProps = { variant?: "primary" | "danger"; children: React.ReactNode; onClick?: () => void };

function Button({ variant = "primary", children, onClick }: ButtonProps) { ... }

const [user, setUser] = useState<User | null>(null);

function handleChange(e: React.ChangeEvent<HTMLInputElement>) { setQuery(e.target.value); }
```

### 8.3 [Strong junior] `any` va `unknown` farqi? Generic nima? Utility type'lardan qaysilarini bilasiz?

- `any` tekshiruvni **o'chiradi**; `unknown` esa ishlatishdan oldin turini **tekshirishga majbur** qiladi (xavfsizroq)
- **Generic**: tur parametri, funksiya yoki component har xil turlar bilan ishlaydi: `function first<T>(arr: T[]): T | undefined`
- Utility type'lar: `Partial<T>`, `Required<T>`, `Pick<T, K>`, `Omit<T, K>`, `Record<K, V>`, `ReturnType<F>`

```ts
type UpdateUser = Partial<Omit<User, "id">>;   // id'siz, hamma maydon ixtiyoriy
```

### 8.4 [Strong junior] Axios va `fetch` farqi? Interceptor'lar nima uchun?

| | fetch | Axios |
|---|---|---|
| JSON | `res.json()` qo'lda | Avtomatik |
| HTTP xato (404/500) | Reject **qilmaydi**, `res.ok` tekshiriladi | Reject qiladi |
| Interceptor | Yo'q | Bor |
| Timeout | AbortController bilan | `timeout` opsiyasi |

Interceptor: har bir so'rovga **tokenni avtomatik qo'shish**, `401` javobda tokenni yangilash (refresh) yoki login'ga yo'naltirish, xatolarni markazlashgan qayta ishlash.

### 8.5 [Junior] Next.js nima va oddiy React (Vite) ilovasidan farqi?

Next.js React ustidagi **framework**: fayl asosidagi routing, **server-side rendering** (SSR), statik generatsiya (SSG), server component'lar, API (route handlers), rasm va shrift optimizatsiyasi. Vite + React SPA'da hamma narsa brauzerda render bo'ladi (CSR), routing va ma'lumot olish uchun kutubxonalar o'zimiz tanlaymiz.

### 8.6 [Strong junior] Server Components va Client Components farqi nima?

- **Server Component** (App Router'da default): serverda render bo'ladi, JavaScript'i brauzerga yuborilmaydi, ma'lumotni to'g'ridan-to'g'ri (bazadan yoki API'dan) oladi. Hook'lar (`useState`, `useEffect`) va event handler'lar ishlatib bo'lmaydi
- **Client Component** (`"use client"`): brauzerda interaktiv ishlaydi, state va event'lar uchun

Qoida: imkon qadar server component, interaktivlik kerak bo'lgan **kichik qismlar** client component.

### 8.7 [Strong junior] SSR, SSG va ISR farqi?

- **SSR**: har so'rovda serverda HTML yaratiladi (shaxsiy, tez o'zgaradigan ma'lumot)
- **SSG**: build paytida bir marta yaratiladi (blog, landing): eng tez
- **ISR**: statik sahifa ma'lum vaqtdan keyin fonda **qayta yaratiladi** (`revalidate`)

Next.js'da keshlash qoidalari versiyalar orasida o'zgargan (masalan, Next.js 15 da `fetch` default holda kesh qilinmaydi), shuning uchun ishlatilayotgan versiya hujjatini tekshirish muhim.

### 8.8 [Middle] Qachon Next.js kerak, qachon Vite SPA yetarli?

- **Next.js**: SEO muhim (ochiq sahifalar, blog, e-commerce), birinchi yuklanish tezligi muhim, server mantig'i kerak
- **Vite SPA**: login orqasidagi dashboard va ichki ilovalar (SEO kerak emas), oddiyroq deploy (statik hosting)

Masalan, Bogizilol'ning mijozlar qismi SEO uchun Next.js'da yaxshiroq bo'lardi, admin panel esa SPA'da ham yetarli.

### 8.9 [Junior] Redux Toolkit va Zustand: qachon qaysi biri?

Zustand: kichik va o'rta ilovalar, kam boilerplate. Redux Toolkit: katta jamoa, murakkab va ko'p domenli state, kuchli DevTools va qat'iy tuzilma kerak bo'lganda. Ikkalasi ham **client state** uchun, server ma'lumoti uchun TanStack Query.

### 8.10 [Junior] Vite va Webpack farqi?

Webpack development'da ham butun ilovani **bundle** qiladi, shuning uchun katta loyihada sekin. Vite development'da brauzerning **native ES modullaridan** foydalanadi, faqat so'ralgan faylni o'giradi, shuning uchun juda tez ishga tushadi va HMR tez. Production build uchun Vite Rollup (yangi versiyalarda Rolldown) ishlatadi. Webpack'ni eski yoki maxsus sozlangan loyihalarda uchratasiz.

### 8.11 [Junior] SASS/SCSS va CSS variables farqi?

SASS **build paytida** CSS'ga aylanadi: o'zgaruvchilar, nesting, mixin'lar, funksiyalar. **CSS variables** esa **brauzerda** ishlaydi va runtime'da o'zgartirish mumkin (dark mode, tema). Hozirgi CSS'da nesting va o'zgaruvchilar bor, shuning uchun SASS'ga ehtiyoj kamaygan.

### 8.12 [Junior] Material UI va DaisyUI (Tailwind) farqi?

MUI: tayyor, to'liq funksional React component'lar (Material dizayn), tez boshlash, lekin o'ziga xos dizayn qilish qiyinroq va bundle og'irroq. DaisyUI: Tailwind ustidagi class'lar to'plami (`btn`, `card`), yengil, lekin JavaScript mantiqi (modal boshqaruvi va h.k.) o'zingizda. shadcn/ui esa component kodini loyihangizga ko'chiradi va to'liq nazorat beradi.

### 8.13 [Junior] Supabase va Firebase farqi?

Supabase: **PostgreSQL** (relyatsion, SQL, jadvallar orasidagi bog'lanish), RLS, ochiq kodli. Firebase (Firestore): **NoSQL** hujjatlar, real-time kuchli, Google ekotizimi. Bog'lanishlar va murakkab so'rovlar ko'p bo'lsa Supabase, oddiy real-time ma'lumot va mobil ilovalar uchun Firebase qulay.

### 8.14 [Junior] ESLint va Prettier farqi?

ESLint kod **sifatini** tekshiradi (ishlatilmagan o'zgaruvchi, hook qoidalari, potentsial xatolar). Prettier faqat **formatlash** (bo'shliq, qo'shtirnoq, qator uzunligi). Ikkalasi birga ishlatiladi, formatlash qoidalari bo'yicha to'qnashmasligi uchun `eslint-config-prettier`.

---

## 9. Tushunchalar (Concepts)

### 9.1 [Junior] REST API nima? HTTP metodlar va status kodlarini ayting

Resurslar URL orqali, amallar HTTP metodlari orqali: `GET` (o'qish), `POST` (yaratish), `PUT` (to'liq almashtirish), `PATCH` (qisman yangilash), `DELETE`.

Status kodlar: `200` OK, `201` Created, `204` No Content, `400` Bad Request, `401` Unauthorized (login qilinmagan), `403` Forbidden (huquq yo'q), `404` Not Found, `409` Conflict, `422` validatsiya xatosi, `500` server xatosi.

### 9.2 [Strong junior] `401` va `403` farqi nima? Frontend ularga qanday javob beradi?

`401`: foydalanuvchi **aniqlanmagan** (token yo'q yoki muddati o'tgan) → token yangilash yoki login'ga yo'naltirish. `403`: foydalanuvchi aniqlangan, lekin **huquqi yo'q** (masalan, kuryer admin endpoint'iga) → "Ruxsat yo'q" xabari, login'ga yo'naltirish shart emas.

### 9.3 [Strong junior] CORS nima va uni frontend'da "tuzatsa" bo'ladimi?

Brauzer xavfsizlik qoidasi: sahifa bir domendan boshqa domenga so'rov yuborsa, server javobida ruxsat (`Access-Control-Allow-Origin`) bo'lishi kerak. CORS **server tomonida** sozlanadi, frontend uni "o'chira olmaydi". Development'da Vite **proxy** bilan muammoni chetlab o'tish mumkin, lekin production'da backend to'g'ri sozlanishi kerak.

### 9.4 [Junior] Responsive va mobile-first nima?

Responsive: sahifa har qanday ekranga moslashadi. Mobile-first: stillar avval **kichik ekran** uchun yoziladi, kattalari `min-width` media query bilan qo'shiladi. Vositalar: Flexbox/Grid, `rem` va `%`, `clamp()`, moslashuvchan rasmlar (`srcset`, `object-fit`).

### 9.5 [Junior] Cross-browser moslikni qanday ta'minlaysiz?

Can I Use bilan imkoniyatni tekshirish, Autoprefixer (vendor prefikslar), kerak bo'lsa polyfill, real brauzerlarda (Chrome, Safari, Firefox, mobil Safari) sinab ko'rish. Safari ko'pincha eng ko'p muammo chiqaradigan brauzer (masalan, `100vh` va mobil toolbar, sana input'lari).

### 9.6 [Junior] "Pixel-perfect" va responsive bir-biriga zid emasmi?

Qisman. Pixel-perfect asosiy breakpoint'larda (dizayn berilgan ekranlarda) Figma'ga aniq mos kelishni anglatadi, oraliq o'lchamlarda esa layout **mantiqiy** moslashishi kerak. Dizayner bilan oraliq holatlarni oldindan kelishish muhim.

### 9.7 [Strong junior] Web performance qanday o'lchanadi? Core Web Vitals nima?

- **LCP** (Largest Contentful Paint): eng katta element qancha vaqtda ko'rindi (yaxshi: 2.5 s dan kam)
- **INP** (Interaction to Next Paint): bosish yoki yozishga javob tezligi (yaxshi: 200 ms dan kam)
- **CLS** (Cumulative Layout Shift): sahifa "sakrashi" (yaxshi: 0.1 dan kam)

O'lchash: Lighthouse, Chrome DevTools Performance, PageSpeed Insights. Yaxshilash: rasmlarni optimallash (`webp`, o'lcham, `loading="lazy"`), rasmga `width/height` berish (CLS), code splitting, kerakli shriftlarni oldindan yuklash, keraksiz JavaScript'ni kamaytirish.

### 9.8 [Junior] Lazy loading va code splitting farqi?

**Code splitting**: kodni bir nechta chunk'larga bo'lish (dinamik `import()`). **Lazy loading**: resursni faqat **kerak bo'lganda** yuklash: `React.lazy` + `Suspense` (component'lar), `loading="lazy"` (rasmlar). Odatda route bo'yicha bo'linadi.

### 9.9 [Strong junior] SPA'da SEO nega muammo? Qanday yechimlar bor?

Oddiy React SPA'da server bo'sh `<div id="root">` qaytaradi, kontent JavaScript bilan chiziladi. Qidiruv robotlari buni yomonroq indekslaydi va ijtimoiy tarmoqlarda havola preview'lari (Open Graph) ishlamaydi. Yechimlar: **SSR/SSG** (Next.js), `<title>` va `<meta>` teglari (sahifa bo'yicha), semantik HTML, `sitemap.xml`, `robots.txt`, tezkor yuklanish. Login orqasidagi dashboard'lar uchun SEO muhim emas.

### 9.10 [Strong junior] Frontend xavfsizligi: XSS nima va React undan qanday himoya qiladi?

**XSS**: saytga zararli skript kiritilib, boshqa foydalanuvchilar brauzerida ishlashi (masalan, token o'g'irlash). React `{}` ichidagi matnni **avtomatik escape** qiladi, shuning uchun oddiy holatda xavfsiz. Xavf joylari: `dangerouslySetInnerHTML` (faqat DOMPurify bilan tozalangan HTML), `href` ga foydalanuvchi kiritgan `javascript:` URL, uchinchi tomon skriptlari.

### 9.11 [Junior] Reusable component'larni qanday loyihalaysiz?

Bitta vazifa, aniq props API (variant, o'lcham), oqilona default qiymatlar, `children` va composition, `className` qabul qilish, bog'liq qismlar uchun compound component. Batafsil: `react-course/section-10` va `section-28` fayllari.

---

## 10. Xulq-atvor (behavioral) savollari: texnik suhbat oxirida

### 10.1 Muddatga ulgurmasangiz nima qilasiz?

Muammoni **erta** aytaman (oxirgi kun emas), nima tayyor va nima qolganini, sababni va yangi taxminiy muddatni beraman. Kerak bo'lsa, vazifani bo'lib, eng muhim qismini birinchi topshirishni taklif qilaman.

### 10.2 Code review'da izohga qo'shilmasangiz nima qilasiz?

Avval reviewer'ning sababini tushunishga harakat qilaman, keyin o'z variantimni **dalil bilan** (performance, o'qilishi, hujjat) tushuntiraman. Kelisha olmasak, jamoa standartiga yoki tajribaliroq hamkasbning qaroriga amal qilaman. Maqsad: kod, shaxs emas.

### 10.3 Yangi texnologiyani qanday o'rganasiz?

Rasmiy hujjat, kichik amaliy loyiha, keyin real vazifada qo'llash. Misol: Vue/Nuxt'ni Gross'da ish jarayonida o'rgandim, React Query va Supabase'ni kurs va o'z loyihalarimda qo'lladim.

### 10.4 AI vositalaridan (Copilot, Claude, ChatGPT) qanday foydalanasiz?

Kod yozishni tezlashtirish, xatoni tushuntirish, test yoki hujjat yozish uchun. Lekin AI yozgan kodni **tushunmasdan qabul qilmayman**: o'qib, tekshirib, loyiha standartiga moslayman. Intervyuer bu yerda mas'uliyatli foydalanishni eshitmoqchi.

### 10.5 Bizga nima uchun qo'shilmoqchisiz? Sizda savollar bormi?

Kompaniya mahsuloti haqida oldindan o'qing va **aniq** sabab ayting. Oxirida albatta 2–3 savol bering: jamoa tuzilishi, code review jarayoni, texnologiyalar (TypeScript, test), birinchi 3 oyda nimalar kutiladi.
