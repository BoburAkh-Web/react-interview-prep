# Intervyu savollari 2-qism: Zustand, RHF, Zod va TypeScript

## Bu qismda nimalar bor

Bu hujjat 1-qismning davomi: Truck Types kalkulyatori, Contact formasi, Footer va suzuvchi tugmalar bosqichlarida ishlatilgan mavzular. Har bir javob loyihadagi haqiqiy koddan olingan misol bilan berilgan.

| Mavzu | Loyihadagi joyi |
| --- | --- |
| Discriminated union | `QuoteResult`: `{ kind: "price" } \| { kind: "onRequest" }` |
| `typeof` (tip ichida) | `z.infer<typeof contactSchema>` |
| Event tipi | `ChangeEvent<HTMLInputElement>` (`MilesInput`) |
| `Promise<void>` | `sendContactEmail` |
| `.d.ts`, declaration merging | `vite-env.d.ts` dagi `ImportMetaEnv` |
| `readonly` | `.env` o'zgaruvchilari |
| Zustand | `useQuoteStore`, selector, `set`, `useSelectedTruck` |
| Zod | `contactSchema`, `z.infer` |
| React Hook Form | `useForm`, `register`, `handleSubmit`, `errors`, `reset` |
| Yuborish holatlari | `FormStatus = "idle" \| "success" \| "error"` |
| Servis qatlami, `.env` | `services/emailService.ts`, EmailJS |
| Sof funksiyalar | `formatCurrency`, `calculateQuote`, `formatQuoteSummary` |
| `key` bilan reset | mashina rasmi animatsiyasi |
| Motion | `AnimatePresence`, `variants`, `MotionConfig` |
| Tailwind v4 | `@theme`, `aria-invalid:`, `group-hover:` |

## 1. TypeScript (yangi mavzular)

### 1.1. Discriminated union nima?

Bu obyekt shakllarining union'i, va har bir shaklda **bir xil nomli, lekin har xil literal qiymatli** maydon bo'ladi. Shu maydon qaysi variant ekanini bildiradi.

```ts
type QuoteResult =
  | { kind: "price"; min: number; max: number }
  | { kind: "onRequest" };
```

`kind` ajratib turuvchi maydon (discriminant). Uning nomi istalgancha bo'lishi mumkin (`type`, `status`), muhimi u har variantda bo'lishi va literal qiymatga ega bo'lishi.

### 1.2. Discriminated union nima beradi?

TypeScript variantga xos maydonga faqat `kind` tekshirilgandan keyin ruxsat beradi:

```ts
result.min; // ❌ "onRequest" variantida min yo'q

if (result.kind === "price") {
  result.min; // ✅
}
```

Ya'ni "narx yo'q" holatini unutib, ekranga `undefined` chiqarish **imkonsiz**. Eski saytda Semi Truck tanlanganda bo'sh bloklar chiqardi, yangi kodda bu xato tip darajasida yopilgan.

### 1.3. Nega `{ min?: number; max?: number }` emas?

Ixtiyoriy maydonlar bilan "min bor, max yo'q" kabi mantiqsiz holat ham tip bo'yicha ruxsat etilgan bo'lib qoladi. Discriminated union'da faqat **ma'noli** holatlar mavjud: yo ikkala narx bor, yo umuman yo'q.

### 1.4. Discriminated union yana qayerda ishlatiladi?

- API javoblari: `{ status: "success"; data: User } | { status: "error"; message: string }`
- Redux action'lari: `{ type: "add"; item } | { type: "remove"; id }`
- Forma va so'rov holatlari.

### 1.5. Tip ichida `typeof` nima qiladi?

JavaScript'da `typeof 5` matn qaytaradi (`"number"`). TypeScript'da esa **tip yoziladigan joyda** `typeof` o'zgaruvchining TypeScript tipini oladi:

```ts
const truck = { id: 1, name: "Van" };
type T = typeof truck; // { id: number; name: string }
```

Loyihada: `z.infer<typeof contactSchema>`. `contactSchema` qiymat, `z.infer` ga esa tip kerak, `typeof` qiymatdan tipni oladi.

### 1.6. React event'iga tip qanday beriladi?

Handler **alohida funksiya** bo'lsa, event tipini yozish kerak:

```ts
function handleChange(event: ChangeEvent<HTMLInputElement>) {
  event.target.valueAsNumber;
}
```

- `ChangeEvent`: hodisa turi (React'dan, `import type`).
- `<HTMLInputElement>`: hodisa qaysi elementda bo'lgani. Shu tufayli `event.target` ning maydonlari ma'lum.

Handler **inline** bo'lsa (`onChange={(e) => ...}`), tip yozish shart emas: TypeScript uni `onChange` dan o'zi chiqaradi. Boshqa tez-tez uchraydiganlari: `FormEvent<HTMLFormElement>`, `MouseEvent<HTMLButtonElement>`, `KeyboardEvent`.

### 1.7. `event.target.value` va `valueAsNumber` farqi?

`value` **har doim `string`**, hatto `type="number"` inputda ham. `valueAsNumber` qiymatni darhol son qilib beradi, input bo'sh bo'lsa esa `NaN` qaytaradi. Shuning uchun `Number.isNaN(value) ? 0 : value` tekshiruvi kerak.

### 1.8. `async` funksiyaning qaytish tipi nima bo'ladi?

Har doim `Promise<...>`:

```ts
async function sendContactEmail(values: ContactFormValues): Promise<void> {}
```

- `Promise<void>`: qiymat qaytmaydi, faqat tugashini kutamiz.
- `Promise<User[]>`: foydalanuvchilar ro'yxati bilan tugaydi.

Xato bo'lsa, Promise reject bo'ladi va `await` qilgan joydagi `try / catch` ga yetib boradi.

### 1.9. `.d.ts` fayl nima?

Faqat tiplar yoziladigan fayl (declaration file). Unda ishlaydigan kod yo'q, build'da brauzerga hech narsa tushmaydi. Loyihada `vite-env.d.ts`: `.env` o'zgaruvchilarining tiplari shu yerda.

### 1.10. Declaration merging nima? Nega `vite-env.d.ts` da `interface` ishlatilgan?

Bir xil nomli ikki `interface` TypeScript'da avtomatik **birlashadi**:

```ts
interface ImportMetaEnv {
  readonly VITE_EMAILJS_SERVICE_ID: string;
}
```

Vite'da `ImportMetaEnv` allaqachon bor. Biz shu nom bilan yana yozsak, maydonlarimiz Vite'nikiga qo'shiladi. `type` bilan bir xil nomni ikki marta yozib bo'lmaydi, shuning uchun bu yerda faqat `interface` ishlaydi. Bu `type` va `interface` farqining amaldagi misoli.

### 1.11. `readonly` nima? `const` dan farqi?

`readonly` obyekt **maydoniga** qiymat yozishni taqiqlaydi. `const` esa **o'zgaruvchini** qayta belgilashni taqiqlaydi, lekin ichini emas:

```ts
const van = { id: 1 };
van.id = 2;      // ✅ const ichini himoya qilmaydi

type T = { readonly id: number };
t.id = 2;        // ❌ readonly himoya qiladi
```

Ikki cheklov: `readonly` faqat TypeScript'da ishlaydi (brauzerda iz qolmaydi, runtime uchun `Object.freeze`) va faqat bir qavat chuqurlikda (shallow).

### 1.12. `.find()` natijasining tipi nima?

`T | undefined`, chunki element topilmasligi mumkin. Shuning uchun natijani ishlatishdan oldin tekshirish kerak:

```ts
const truck = trucks.find((t) => t.id === id); // Truck | undefined
if (!truck) throw new Error("Truck not found");
return truck; // Truck
```

### 1.13. Telefon raqami nega `string`, `number` emas?

Unda `+`, probel, qavs bo'ladi, boshidagi `0` songa aylanganda yo'qoladi va u bilan hisob-kitob qilinmaydi. Qoida: **ustida matematik amal bajariladigan bo'lsa `number`, aks holda `string`.** Pochta indeksi, karta raqami va ID kodlar ham `string`.

## 2. Zustand

### 2.1. Zustand nima?

Global state uchun kichik kutubxona. Uni **komponentlardan tashqarida turadigan `useState`** deb tasavvur qilsa bo'ladi: state alohida faylda yashaydi va istalgan komponent uni o'qiy oladi.

### 2.2. Nega Zustand ishlatding? Loyihada qanday muammoni hal qildi?

Kalkulyatordagi tanlangan mashina va milya ikki joyda kerak edi: Truck Types bo'limida (hisoblash) va Contact formasida (xatga hisob-kitobni qo'shish). Bu ikki bo'lim sahifada bir-biridan uzoqda. `useState` bilan state'ni `App` ga ko'tarib, prop orqali tushirish kerak bo'lardi. Zustand'da ikkalasi ham store'dan to'g'ridan-to'g'ri o'qiydi.

### 2.3. Zustand, Context va Redux farqi?

| | Context | Redux Toolkit | Zustand |
| --- | --- | --- | --- |
| Provider kerakmi | ha | ha | yo'q |
| Kod hajmi | kam | ko'p (slice, store, dispatch) | juda kam |
| Re-render nazorati | qiymat o'zgarsa, hamma iste'molchi | selector bilan | selector bilan |
| Qachon | kam o'zgaradigan narsa (theme) | katta, murakkab ilova | o'rta va kichik global state |

Loyihada ikkalasi ham bor: theme va Tabs uchun Context, kalkulyator uchun Zustand. Ular bir-birini almashtirmaydi.

### 2.4. Store qanday yaratiladi?

```ts
type QuoteState = {
  truckId: number;
  miles: number;
  setMiles: (miles: number) => void;
};

export const useQuoteStore = create<QuoteState>()((set) => ({
  truckId: 1,
  miles: 100,
  setMiles: (miles) => set({ miles }),
}));
```

Store bu oddiy obyekt: unda ham ma'lumot (state), ham uni o'zgartiradigan funksiyalar (action'lar) bor. `create` **hook** qaytaradi, shuning uchun nomi `use` bilan boshlanadi.

### 2.5. Nega `create<QuoteState>()(...)`, ya'ni ikki juft qavs?

TypeScript bilan Zustand'ning rasmiy shakli. Birinchi chaqiruvga tip beriladi, u funksiya qaytaradi, ikkinchisiga store beriladi. Bu `persist` kabi middleware qo'shilganda tiplar to'g'ri chiqishi uchun kerak. Qoida: TS bilan har doim `create<Tip>()(...)`.

### 2.6. `set` qanday ishlaydi? `useState` setteridan farqi?

`set` yangi qiymatni eskisi bilan **birlashtiradi** (merge):

```ts
set({ miles: 250 }); // faqat miles o'zgaradi, truckId va funksiyalar joyida qoladi
```

`useState` bilan obyekt saqlaganda esa `{ ...prev, miles: 250 }` deb qo'lda yozish kerak. Merge faqat **bir qavat** chuqurlikda ishlaydi.

### 2.7. Yangi state eskisiga bog'liq bo'lsa-chi?

`set` ga funksiya beriladi, u eng so'nggi state'ni oladi:

```ts
selectNextTruck: () =>
  set((state) => ({ truckId: shiftTruckId(state.truckId, 1) })),
```

Bu `useState` dagi `setOpen((open) => !open)` bilan bir xil g'oya.

### 2.8. Selector nima va nega muhim?

```tsx
const miles = useQuoteStore((state) => state.miles);
```

`(state) => state.miles` selector: "store'dan menga faqat shu qismni ber". Komponent faqat **tanlagan qismi** o'zgarganda qayta render bo'ladi. `truckId` o'zgarsa, faqat `miles` ni o'qiydigan komponent render bo'lmaydi.

### 2.9. Zustand'dagi eng ko'p uchraydigan xatolar?

1. Selectorsiz chaqirish: `const { miles } = useQuoteStore()`. Butun store'ga obuna bo'ladi va istalgan o'zgarishda render bo'ladi.
2. Selectordan yangi obyekt qaytarish: `useQuoteStore((s) => ({ a: s.a, b: s.b }))`. Har safar yangi obyekt yaratiladi, Zustand uni "o'zgardi" deb hisoblaydi va ortiqcha render bo'ladi (5-versiyada hatto cheksiz render xatosi). Yechim: har qiymatga alohida selector yoki `useShallow`.

### 2.10. Store'da butun mashina obyektini emas, `truckId` ni saqladingiz. Nega?

Obyektni ham saqlasak, ikki joyda bir xil ma'lumot bo'lardi va ular bir-biridan ajralib qolishi mumkin edi. Faqat `id` saqlanadi, obyekt esa `useSelectedTruck` custom hookida `find` bilan topiladi (derived state).

### 2.11. Action'larni store'ga yozish yaxshimi yoki komponentga?

Store'ga. "Keyingi mashina" mantig'i (indeks, aylanish) store'da bo'lgani uchun komponent faqat `selectNextTruck()` ni chaqiradi. Mantiq bir joyda, uni bir nechta komponent qayta ishlatadi.

## 3. Zod

### 3.1. Zod nima va nega kerak?

Ma'lumotni **sayt ishlayotgan paytda** (runtime) tekshiradigan kutubxona. TypeScript faqat kod yozilayotganda tekshiradi. Forma ma'lumoti esa foydalanuvchidan keladi: kimdir email o'rniga `abc` yozsa, TypeScript buni bila olmaydi. Zod shu bo'shliqni to'ldiradi.

### 3.2. Sxema qanday yoziladi?

```ts
export const contactSchema = z.object({
  phone: z.string().trim().min(7, "Please enter a valid phone number"),
  email: z.email("Please enter a valid email address"),
  message: z.string().trim().max(1000, "Message is too long"),
  includeQuote: z.boolean(),
});
```

- `z.object`: obyekt va uning maydonlari.
- `.trim()`: atrofdagi probellarni olib tashlaydi, shunda `"   "` bo'sh hisoblanadi.
- `.min` va `.max`: chegaralar, ikkinchi argument xato xabari.
- `.min` bo'lmasa, maydon ixtiyoriy (bu yerda `message`).
- Zod 4'da email: `z.email()`. Eski yozuv: `z.string().email()`.

### 3.3. `z.infer` nima?

Sxemadan TypeScript tipini **avtomatik** chiqaradi:

```ts
export type ContactFormValues = z.infer<typeof contactSchema>;
// { phone: string; email: string; message: string; includeQuote: boolean }
```

### 3.4. `z.infer` ning foydasi nima?

**Bitta haqiqat manbai.** Tip qo'lda yozilsa, sxema va tip ikki joyda bo'lardi va bir-biridan ajralib qolishi mumkin edi. Loyihada sxemaga `includeQuote` qo'shilganda tip o'zi o'zgardi va TypeScript `defaultValues` da shu maydon yo'qligini darhol ko'rsatdi.

### 3.5. `z.infer` tipni qayerdan biladi?

Zod TypeScript'da yozilgan. Har bir funksiyasi qaytaradigan sxemaning tipi ichida "qanday ma'lumotni tekshiradi" degan ma'lumot generic sifatida turadi (`z.string()` taxminan `ZodSchema<string>`). `z.object` ularni bitta obyekt tipiga yig'adi, `z.infer` esa shu yashirin parametrni chiqarib oladi. Hammasi kod yozilayotganda bo'ladi, brauzerga hech narsa qo'shilmaydi.

### 3.6. Zod va `as` farqi?

| | `as` | Zod |
| --- | --- | --- |
| Tekshiradimi | yo'q, faqat ishonadi | ha, haqiqatan tekshiradi |
| Qachon ishlaydi | kod yozilayotganda | sayt ishlayotganda |
| Qachon ishlatiladi | o'zimiz nazorat qiladigan ma'lumot (o'z JSON'imiz) | tashqi ma'lumot: forma, API javobi |

### 3.7. Zod'ni yana qayerda ishlatsa bo'ladi?

API javobini tekshirish (`schema.parse(data)`), `.env` o'zgaruvchilarini tekshirish, URL parametrlari va `localStorage` dan o'qilgan ma'lumot. Umumiy qoida: **dasturdan tashqaridan kelgan har qanday ma'lumot**.

### 3.8. `parse` va `safeParse` farqi?

- `parse`: ma'lumot noto'g'ri bo'lsa, **xato otadi**.
- `safeParse`: xato otmaydi, natija qaytaradi: `{ success: true, data } | { success: false, error }`. Bu ham discriminated union.

Formada bularni o'zimiz chaqirmadik, chunki `zodResolver` ularni React Hook Form ichida chaqiradi.

## 4. React Hook Form

### 4.1. React Hook Form nima va nega ishlatding?

Formalar uchun kutubxona: maydonlar qiymati, validatsiya, xatolar va yuborish holatini o'zi boshqaradi. Asosiy afzalligi: har harf yozilganda butun forma **qayta render bo'lmaydi**. Qo'lda yozilgan formada har bir maydon uchun `useState` va `onChange` kerak bo'lardi.

### 4.2. Controlled va uncontrolled input farqi?

| | Controlled | Uncontrolled |
| --- | --- | --- |
| Qiymat qayerda | React state'da (`value={...}`) | DOM'da, `ref` orqali o'qiladi |
| Har harf yozilganda | komponent qayta render bo'ladi | render bo'lmaydi |
| Loyihadagi misol | Miles input (narx darhol yangilanishi kerak) | Contact formasi (ma'lumot faqat yuborishda kerak) |

RHF asosan uncontrolled usulda ishlaydi. Ikkalasi ham to'g'ri, tanlov vazifaga bog'liq.

### 4.3. `useForm` qanday sozlanadi?

```tsx
const { register, handleSubmit, reset, formState: { errors, isSubmitting } } =
  useForm<ContactFormValues>({
    resolver: zodResolver(contactSchema),
    defaultValues: { phone: "", email: "", message: "", includeQuote: false },
  });
```

- `<ContactFormValues>`: generic, forma maydonlari tipi (Zod'dan).
- `resolver: zodResolver(...)`: validatsiyani Zod sxemasiga topshiradi.
- `defaultValues`: boshlang'ich qiymatlar, tipga mos bo'lishi shart.

### 4.4. `register` nima qaytaradi?

Obyekt: `{ name, onChange, onBlur, ref }`. `{...register("phone")}` uni inputga atribut sifatida yoyadi. TypeScript faqat mavjud maydon nomlarini qabul qiladi: `register("phon")` xato. Checkbox uchun RHF matn emas, `true`/`false` beradi.

### 4.5. `handleSubmit` nima qiladi?

Tugma bosilganda uch ish:

1. `preventDefault`: sahifa yangilanmaydi.
2. Maydonlarni Zod sxemasidan o'tkazadi.
3. **Faqat hammasi to'g'ri bo'lsa** `onSubmit(data)` ni chaqiradi.

Shuning uchun `onSubmit` ichida qayta tekshirish shart emas: u yerga faqat to'g'ri ma'lumot keladi.

### 4.6. Xatolar qanday ko'rsatiladi?

`formState.errors` da faqat xatosi bor maydonlar bo'ladi:

```tsx
{errors.email && <p id="email-error">{errors.email.message}</p>}
```

`errors.email` tipi `FieldError | undefined`, `&&` narrowing qiladi. Xabar matni sxemadan keladi. Inputga `aria-invalid` va `aria-describedby` qo'shiladi.

### 4.7. Validatsiya qachon ishlaydi?

Standart holatda: birinchi marta **yuborilganda**, undan keyin har o'zgarishda. Shuning uchun foydalanuvchi hali yozayotganda xato chiqmaydi, lekin xatoni tuzatayotganda xabar darhol yo'qoladi. Buni `mode` (`onBlur`, `onChange`, `onTouched`) bilan o'zgartirsa bo'ladi.

### 4.8. `isSubmitting` qayerdan keladi?

`onSubmit` `async` bo'lsa, RHF uning Promise'i tugaguncha `isSubmitting` ni `true` qilib turadi. Shuning uchun "yuborilmoqda" holati uchun alohida `useState` kerak emas. U tugmani o'chirish va "Sending..." yozuvi uchun ishlatiladi.

### 4.9. `reset` ni qayerda chaqirish kerak?

Faqat **muvaffaqiyatli** yuborishdan keyin:

```tsx
try {
  await sendContactEmail(data, quoteSummary);
  reset();
  setStatus("success");
} catch {
  setStatus("error");
}
```

Xato bo'lsa, `await` xato otadi va kod `catch` ga sakraydi, `reset` ishlamaydi. Foydalanuvchi yozganlari saqlanib qoladi.

### 4.10. `noValidate` nima uchun?

`type="email"` inputda brauzer o'z tekshiruvini qiladi va standart pufakcha chiqaradi. `noValidate` uni o'chiradi, shunda tekshiruvni faqat Zod qiladi va xabarlar bizning dizaynda chiqadi.

## 5. Formani yuborish: holatlar, servis va `.env`

### 5.1. Forma holatlarini qanday boshqarding?

Union bilan:

```ts
type FormStatus = "idle" | "success" | "error";
const [status, setStatus] = useState<FormStatus>("idle");
```

"Yuborilmoqda" holati RHF'ning `isSubmitting` idan olinadi. `useState<FormStatus>` da generic aniq yozilishi shart, aks holda TypeScript `string` chiqaradi va `setStatus("succes")` kabi xatolarni ushlamaydi.

### 5.2. Nega ikkita boolean (`isSuccess`, `isError`) emas?

Ikki boolean bilan ikkalasi bir vaqtda `true` bo'lib qolishi mumkin, ya'ni mantiqsiz holat. Union'da forma faqat **bitta** holatda bo'la oladi. Qoida: **mumkin bo'lmagan holatlarni tip darajasida imkonsiz qil.**

### 5.3. Servis qatlami nima va nega kerak?

Tashqi xizmat bilan ishlash (bu yerda EmailJS) alohida faylga chiqarilgan: `services/emailService.ts`. Komponent faqat `sendContactEmail(data)` ni chaqiradi va "qanday yuboriladi" haqida bilmaydi. Foydalari:

- EmailJS boshqa xizmatga almashsa, faqat shu fayl o'zgaradi;
- shablon nomlariga tarjima (`phone` → `user_number`) bitta joyda;
- komponent sodda qoladi.

### 5.4. Xato qayerda ushlanadi?

Servis xatoni ushlamaydi: `emailjs.send` reject bo'lsa, xato `await` orqali komponentdagi `try / catch` ga yetib boradi. Har qatlam o'z ishini qiladi: servis yuboradi, komponent natijani ko'rsatadi. Eski saytda xato faqat `console.log` qilinardi va foydalanuvchi forma ishlamaganini bilmasdi.

### 5.5. Nega avval soxta yuborish bilan ishlading?

`await new Promise((resolve) => setTimeout(resolve, 1500))` bilan holatlar (yuborilmoqda, muvaffaqiyat, xato) EmailJS'siz sinab olindi. Keyin bitta qator haqiqiy chaqiruvga almashtirildi, qolgan kod o'zgarmadi. Tashqi xizmatga bog'liq qismni oxirida ulash xatolarni ajratishni osonlashtiradi.

### 5.6. `.env` nima uchun kerak? `VITE_` prefiksi nima?

Sozlamalar va kalitlar kodda emas, `.env` faylda turadi va `.gitignore` orqali GitHub'ga chiqmaydi. Vite faqat `VITE_` bilan boshlangan o'zgaruvchilarni brauzer kodiga beradi, qolganlari `undefined` bo'ladi. Bu server sirlari tasodifan saytga tushib qolishidan himoya. `.env` o'zgarsa, dev serverni qayta ishga tushirish kerak.

### 5.7. `.env` dagi kalit maxfiy bo'lib qoladimi?

Yo'q. Brauzerga yuborilgan har qanday qiymat (`VITE_` o'zgaruvchilari) DevTools'da ko'rinadi. `.env` faqat kalitni koddan va git tarixidan olib tashlaydi. EmailJS public key ochiq bo'lishi uchun mo'ljallangan, himoya uning panelida domen cheklovi bilan qilinadi. Haqiqiy sirlar (API secret, parol) faqat serverda saqlanadi.

### 5.8. Loyihada EmailJS'da qanday muammo chiqdi va uni qanday aniqlading?

Network'da so'rov payload'i to'g'ri ketgan edi, javob esa: `Gmail_API: Invalid grant`. Bu kod xatosi emas: EmailJS'ning mijoz Gmail akkauntiga ulanishi yaroqsiz bo'lib qolgan. Yechim akkaunt egasi tomonida (Gmail'ni qayta ulash). Muhim xulosa: eski sayt ham xat yubormayotgan ekan va bu xato ko'rsatilmagani uchun sezilmagan. Debug tartibi: **payload → response → xato qaysi tomonda.**

## 6. React patternlari

### 6.1. Derived state nima? Misol keltir.

Boshqa state'dan hisoblash mumkin bo'lgan qiymat. Uni alohida state qilib saqlamaslik kerak, har renderda hisoblanadi. Loyihadagi misollar:

- `quote = calculateQuote(truck, miles)`: narx state emas;
- `isSolid = isScrolled || isMenuOpen`: Header foni;
- `miles === preset`: tez tanlash tugmasining faolligi;
- `useSelectedTruck`: store'da faqat `truckId`, obyekt `find` bilan topiladi.

Sababi: ikki joyda saqlangan bir xil ma'lumot bir-biridan **ajralib qolishi** mumkin.

### 6.2. `useMemo` qachon kerak, qachon kerak emas?

Faqat **og'ir** hisob-kitobda (minglab elementni filtrlash yoki saralash) yoki referens barqarorligi muhim bo'lganda. Loyihada `useMemo` ishlatilmadi: `find` 4 ta elementda, `calculateQuote` esa ikki ko'paytmadan iborat. `useMemo` ning o'zi ham xotira va solishtirish talab qiladi, shuning uchun uni har joyga qo'yish kodni sekinlashtirishi mumkin.

### 6.3. `key` bilan komponentni qanday "reset" qilasan?

React elementni `key` orqali taniydi. `key` o'zgarsa, eski element o'chiriladi va yangisi noldan yaratiladi:

```tsx
<motion.img key={truck.id} ... />
```

Mashina almashganda `key` o'zgaradi, shuning uchun eski rasm chiqib ketish, yangisi kirish animatsiyasini oladi. Formani tozalash uchun ham shu usul ishlatiladi.

### 6.4. Sof (pure) funksiya nima? Loyihada qaysilari bor?

Bir xil kirish uchun doim bir xil natija beradi va tashqi narsani o'zgartirmaydi. Loyihada: `formatCurrency`, `formatRate`, `calculateQuote`, `formatQuoteSummary`. Ular `utils/` da, React va store'dan mustaqil. Foydasi: oson tushuniladi, oson testlanadi va istalgan joyda qayta ishlatiladi.

### 6.5. Kasr sonlar bilan qanday muammo chiqdi?

`1.1 * 100` JavaScript'da `110.00000000000001` beradi, chunki kasrlar ikkilik sanoqda aniq saqlanmaydi. Yechim: sentgacha yaxlitlash.

```ts
Math.round(value * 100) / 100;
```

Pul bilan ishlaganda bu muhim, chunki qiymat xatga ham ketadi. Katta tizimlarda pul ko'pincha sentlarda, butun son sifatida saqlanadi.

### 6.6. `Intl.NumberFormat` nima uchun ishlatding?

Sonni pul ko'rinishiga keltirish uchun (`$1,100.00`). Brauzerda tayyor bor, kutubxona kerak emas. Formatter komponentdan tashqarida bir marta yaratiladi, chunki uni yaratish qimmatroq. Bu bilan eski saytdagi "$16.00 - $21" formati xatosi tuzatildi.

### 6.7. Bitta custom hookni qayta ishlatishga misol?

`useScrolled(offset)`: Header'da `useScrolled(20)` fon uchun, suzuvchi tugmalarda `useScrolled(600)` "tepaga" tugmasi uchun. Hookni parametr bilan yozish uni turli vazifalarga moslaydi.

### 6.8. Mantiqni qayerga joylashni qanday tanlaysan?

- Formatlash va hisob-kitob → `utils/` (sof funksiya).
- Bir nechta bo'lim bo'lishadigan state → Zustand store.
- Bitta komponent daraxtidagi umumiy state → Context (Tabs, theme).
- Faqat bitta komponentga kerak → `useState`.
- Tashqi xizmat → `services/`.

## 7. Motion animatsiyalari

### 7.1. Motion nima va qanday ishlaydi?

React uchun animatsiya kutubxonasi (avvalgi nomi Framer Motion). Oddiy teg o'rniga `motion.div`, `motion.h1` va hokazo ishlatiladi, ular brauzerda oddiy teg bo'lib chiqadi:

```tsx
<motion.h1
  initial={{ opacity: 0, y: 30 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.7, ease: "easeOut" }}
/>
```

`initial` qayerdan boshlashni, `animate` qayerga borishni, `transition` qanday borishni belgilaydi.

### 7.2. Variants nima?

Animatsiya holatlariga nom berish. Ota element `initial="hidden" animate="visible"` deydi, bolalar faqat `variants` oladi va holat ularga avtomatik o'tadi. `staggerChildren` bolalarni navbat bilan chiqaradi (Hero: sarlavha, matn, tugmalar). `Variants` tipi obyekt shaklini tekshiradi.

### 7.3. `AnimatePresence` nima uchun kerak?

React o'chirilgan elementni darhol olib tashlaydi, `exit` animatsiyasi ulgurmaydi. `AnimatePresence` o'chayotgan elementni `exit` tugaguncha ushlab turadi. Loyihada: mashina rasmi va "tepaga" tugmasi.

### 7.4. Services tablarida `AnimatePresence` nega ishlamadi?

`AnimatePresence` faqat o'zining **bevosita** bolasining yo'qolishini ko'radi. `Tabs.Panel` faol bo'lmaganda ichkarida `null` qaytaradi, ya'ni o'chayotgan narsa `AnimatePresence` dan bir qavat chuqurroqda. Shuning uchun u yerda faqat kirish animatsiyasi qilindi. Mashina rasmida esa `motion.img` to'g'ridan-to'g'ri `AnimatePresence` ichida, shuning uchun ishlaydi.

### 7.5. `mode="wait"` nima?

Avval eski element to'liq chiqib ketadi, keyin yangisi kiradi. Usiz ikkalasi bir vaqtda ustma-ust ko'rinadi.

### 7.6. Nega `width` emas, `scaleX` animatsiya qilinadi?

`transform` (`scale`, `x`, `y`) va `opacity` brauzer uchun eng arzon: ular layout'ni qayta hisoblamaydi va GPU'da chiziladi. `width`, `height` va `top` esa har kadrda layout'ni qayta hisoblaydi va animatsiya qotishi mumkin. Hero'dagi chiziq shuning uchun `scaleX` va `origin-left` bilan.

### 7.7. `reducedMotion="user"` nima?

Operatsion tizimda "kamroq harakat" sozlamasi yoqilgan foydalanuvchilar uchun harakat animatsiyalarini o'chiradi (bosh aylanishiga moyil odamlar uchun muhim). `App` da `MotionConfig` bilan butun sayt uchun bitta qatorda hal qilindi.

## 8. Tailwind v4 va accessibility

### 8.1. Tailwind v4 ning v3 dan asosiy farqi?

`tailwind.config.js` yo'q. Ranglar va shriftlar CSS faylda `@theme` bilan yoziladi:

```css
@theme {
  --color-brand-blue: #3e4095;
}
```

`--color-` bilan boshlangan o'zgaruvchidan Tailwind o'zi klass yasaydi: `bg-brand-blue`, `text-brand-blue`, `bg-brand-blue/50`.

### 8.2. Dark mode qanday qilingan?

`.dark` klassi `<html>` da turadi, `@custom-variant dark` Tailwind'ga shuni bildiradi. Ranglar semantik: `bg-page`, `text-fg` kabi klasslar CSS o'zgaruvchilariga bog'langan, ular `.dark` bor-yo'qligiga qarab o'zgaradi. Komponentda har joyda `dark:` yozish shart emas. Tanlov `localStorage` da saqlanadi va `index.html` dagi kichik skript sahifa chizilishidan oldin uni qo'llaydi (flash bo'lmaydi).

### 8.3. Nega `bg-${color}` kabi dinamik klasslar ishlamaydi?

Tailwind kodni **matn** sifatida o'qib, faqat to'liq yozilgan klasslarni topadi. `bg-${color}` dan qaysi klass kerakligini bila olmaydi va uni CSS'ga qo'shmaydi. Yechim: to'liq klasslarni obyektda saqlash:

```ts
const variantClasses = {
  primary: "bg-brand-blue",
  accent: "bg-brand-red",
};
```

### 8.4. `group-hover:`, `aria-invalid:`, `data-[state=active]:` nima?

- `group-hover:`: ota elementga `group` qo'yiladi, bola esa **otaning** hover'iga javob beradi (footer'dagi strelka).
- `aria-invalid:`: element `aria-invalid="true"` bo'lsa, stil qo'llanadi. Bitta atribut ham accessibility, ham ko'rinish uchun ishlaydi.
- `data-[state=active]:`: `data-state="active"` bo'lsa, stil qo'llanadi. `Tabs` faqat holatni belgilaydi, rangni ishlatgan joy hal qiladi (Radix va shadcn uslubi).

### 8.5. Loyihada qanday accessibility ishlari qilingan?

- Bosiladigan hamma narsa `<button>` yoki `<a>` (eski saytda `<img onClick>` bor edi).
- Ikonkali tugmalarda `aria-label`.
- Burger menyuda `aria-expanded` va `aria-controls`.
- Tablarda `role="tablist"`, `role="tab"`, `aria-selected`.
- Formada `sr-only` label'lar, `aria-invalid`, `aria-describedby`.
- Natija xabarlarida `role="status"` va `aria-live`.
- Rasmlarda mazmunli `alt`.
- `reducedMotion` va `focus-visible` ramkalari.

### 8.6. Placeholder nega label o'rnini bosmaydi?

Yozishni boshlaganda placeholder yo'qoladi, ekran o'quvchilar uni har doim o'qimaydi va kontrasti odatda past. Dizaynda label ko'rinmasa ham, u `sr-only` bilan saqlanadi: ko'zdan yashirin, ekran o'quvchi uchun mavjud.

### 8.7. Nega `<a>` ichiga `<button>` qo'yib bo'lmaydi?

HTML qoidasiga zid: interaktiv element ichida boshqa interaktiv element bo'lmasligi kerak. Klaviatura va ekran o'quvchida bitta narsa ikki marta fokuslanadi. Yechim: `Button` komponenti `href` bo'lsa `<a>`, aks holda `<button>` qaytaradi (early return).

## 9. Tez takrorlash

| Tushuncha | Bir jumlada | Loyihadagi misol |
| --- | --- | --- |
| Discriminated union | umumiy literal maydonli shakllar union'i | `QuoteResult` |
| `typeof` (tipda) | qiymatdan tip oladi | `typeof contactSchema` |
| `ChangeEvent<T>` | alohida handler uchun event tipi | `MilesInput` |
| `Promise<void>` | async, qiymat qaytarmaydi | `sendContactEmail` |
| Declaration merging | bir xil nomli `interface` lar birlashadi | `ImportMetaEnv` |
| `readonly` | maydonni faqat o'qiladigan qiladi | `.env` tiplari |
| Zustand store | Provider'siz global state | `useQuoteStore` |
| Selector | store'dan faqat kerakli qismni olish | `(s) => s.miles` |
| `set` callback | yangi state eskisidan hisoblansa | `selectNextTruck` |
| Zod sxema | runtime validatsiya | `contactSchema` |
| `z.infer` | sxemadan avtomatik tip | `ContactFormValues` |
| `register` | maydonni RHF'ga ulaydi | `{...register("email")}` |
| `handleSubmit` | tekshiradi, keyin `onSubmit` | `ContactForm` |
| `isSubmitting` | async yuborish davomida `true` | "Sending..." |
| Status union | bir vaqtda faqat bitta holat | `FormStatus` |
| Servis qatlami | tashqi xizmat alohida faylda | `emailService.ts` |
| `VITE_` | faqat shular brauzerga beriladi | `.env` |
| Derived state | hisoblanadigan narsani saqlama | `quote`, `isSolid` |
| `key` reset | `key` o'zgarsa, element qayta yaratiladi | mashina rasmi |
| `AnimatePresence` | yo'qolish animatsiyasini kutadi | "tepaga" tugmasi |
| `@theme` | Tailwind v4 sozlamasi CSS'da | brend ranglari |

**Eng ko'p so'raladigan 5 savol:** controlled va uncontrolled (4.2), Zustand, Context va Redux farqi (2.3), selector va re-render (2.8), Zod va `as` farqi (3.6), discriminated union (1.1–1.2).
