# Rezyume texnologiyalari, 2-qism: Zod, React Hook Form, state vositalari, Axios va Next.js

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

> Har bir savol tuzilmasi 1-qismdagidek: **Oddiy qilib** → **Kod** → **Intervyuda shunday ayting** → **Qo'shimcha savol**.
>
> React Query, Redux Toolkit va Context bo'yicha kurs darajasidagi savollar `react-course/section-18`, `section-20`, `section-24`, `section-27` fayllarida. Bu yerda ular **rezyume nuqtai nazaridan**: ularni qanday tanlaysiz va birga qanday ishlatasiz.

---

## Lug'at: forma va state kalit so'zlari

| So'z | Ma'nosi |
|---|---|
| **Schema** | Ma'lumot qanday ko'rinishda bo'lishi kerakligining tavsifi (qoidalar to'plami) |
| **Parse** | Ma'lumotni sxema bo'yicha tekshirish va kerak bo'lsa o'zgartirib qaytarish |
| **Coerce** | Qiymatni majburan boshqa turga o'girish (`"5"` → `5`) |
| **Refine** | Sxemaga o'zingizning qo'shimcha tekshiruv qoidangizni qo'shish |
| **Resolver** | React Hook Form'ni tashqi validatsiya kutubxonasi (Zod) bilan bog'lovchi |
| **Controlled / uncontrolled input** | Qiymatni React state boshqaradi / qiymatni DOM o'zi saqlaydi |
| **Server state** | Serverda yashaydigan, bizda faqat nusxasi bo'lgan ma'lumot (menyu, buyurtmalar) |
| **Client state** | Faqat brauzerda yashaydigan ma'lumot (modal ochiqmi, tema, savat) |
| **Selector** | Store'dan faqat kerakli qismni oladigan funksiya |
| **Boilerplate** | Har safar takrorlanadigan "majburiy" kod |
| **Middleware** | So'rov yoki action bilan asosiy ish orasida ishlaydigan oraliq qatlam |
| **Interceptor** | Axios'da har bir so'rov yoki javobni avtomatik ushlab, o'zgartiruvchi funksiya |

---

## C. Zod

### C1. [Junior] Zod nima va nima uchun kerak?

**Oddiy qilib:** Zod bu ma'lumot uchun **"qoidalar varag'i"** va tekshiruvchi. Siz sxemada "email bo'lishi kerak, parol kamida 8 belgi" deb yozasiz, Zod esa haqiqiy ma'lumotni shu qoidalarga solishtiradi. Eng katta afzalligi: shu sxemadan **TypeScript turi avtomatik** chiqadi, ya'ni qoida va tur bitta joyda.

```ts
import { z } from "zod";

const UserSchema = z.object({
  name: z.string().min(2, "Ism kamida 2 harf"),
  email: z.string().email("Email noto'g'ri"),
  age: z.number().int().min(7).max(14),
  role: z.enum(["student", "teacher"]),
});

type User = z.infer<typeof UserSchema>;
// { name: string; email: string; age: number; role: "student" | "teacher" }
```

**Intervyuda shunday ayting:** "Zod schema validation kutubxonasi. Men uni formalarni va tashqaridan keladigan ma'lumotlarni tekshirish uchun ishlataman. Afzalligi: `z.infer` bilan TypeScript turi sxemadan chiqadi, qoida va turni alohida yozib, ularni sinxron saqlashga hojat yo'q."

---

### C2. [Junior] `parse` va `safeParse` farqi nima?

**Oddiy qilib:** `parse` ma'lumot noto'g'ri bo'lsa **xato tashlaydi** (try/catch kerak). `safeParse` xato tashlamaydi, natijani **obyekt** ko'rinishida qaytaradi: muvaffaqiyatmi yoki yo'qmi.

```ts
// parse: noto'g'ri bo'lsa throw
const user = UserSchema.parse(data);

// safeParse: xato tashlamaydi
const result = UserSchema.safeParse(data);
if (result.success) {
  console.log(result.data);           // tozalangan, tiplangan ma'lumot
} else {
  console.log(result.error.issues);   // har bir xato: qaysi maydon, qanday xabar
}
```

**Intervyuda shunday ayting:** "API javobini tekshirishda, xato butun so'rovni to'xtatishi kerak bo'lsa, `parse` ishlataman. Foydalanuvchiga xatolarni ko'rsatish kerak bo'lsa yoki xatoni o'zim boshqarmoqchi bo'lsam, `safeParse`."

---

### C3. [Junior] `optional`, `nullable` va `default` farqi?

```ts
z.object({
  bio: z.string().optional(),          // string | undefined (maydon bo'lmasligi mumkin)
  avatar: z.string().nullable(),       // string | null (maydon bor, lekin null bo'lishi mumkin)
  discount: z.number().default(0),     // berilmasa, 0 qo'yiladi
  nickname: z.string().nullish(),      // string | null | undefined
});
```

**Oddiy qilib:** `optional` "bu maydon umuman bo'lmasligi mumkin", `nullable` "maydon bor, lekin bo'sh (`null`) bo'lishi mumkin", `default` "berilmasa, shu qiymatni qo'y". Backend ko'pincha bo'sh qiymatni `null` qaytaradi, shuning uchun bu farq muhim.

---

### C4. [Strong junior] `coerce`, `transform` va `refine` nima?

**Oddiy qilib:**

- **`coerce`**: kiruvchi qiymatni majburan turga o'giradi. Forma input'lari doim **string** qaytaradi, `coerce` uni songa aylantiradi
- **`transform`**: tekshiruvdan keyin qiymatni **o'zgartiradi** (tozalash, formatlash)
- **`refine`**: o'zingizning **qo'shimcha qoidangiz**

```ts
const schema = z.object({
  price: z.coerce.number().positive(),     // "120" → 120

  phone: z
    .string()
    .transform((v) => v.replace(/\D/g, ""))                       // "+998 (90) 123-45-67" → "998901234567"
    .refine((v) => /^998\d{9}$/.test(v), "Telefon raqami noto'g'ri"),

  name: z.string().trim().min(1, "Majburiy maydon"),             // bo'shliqlarni olib tashlash
});
```

**Intervyuda shunday ayting:** "Gross'da telefon raqami uchun aynan shunday qilganman: mask'ni olib tashlab, toza raqamlarni `refine` bilan tekshirganman. Forma input'lari string qaytargani uchun sonli maydonlarda `coerce` ishlataman."

---

### C5. [Strong junior] Bir nechta maydonga bog'liq validatsiya qanday yoziladi?

**Oddiy qilib:** Ba'zi qoidalar bitta maydonga emas, **ikki maydon orasidagi** munosabatga bog'liq: "parol va tasdiqlash bir xil bo'lsin", "chegirma narxdan kichik bo'lsin", "tugash sanasi boshlanishdan keyin bo'lsin". Bunday tekshiruv butun obyekt darajasida yoziladi.

```ts
const schema = z
  .object({
    password: z.string().min(8),
    confirm: z.string(),
    regularPrice: z.coerce.number(),
    discount: z.coerce.number(),
  })
  .refine((d) => d.password === d.confirm, {
    message: "Parollar mos emas",
    path: ["confirm"],              // xato qaysi maydon ostida ko'rinsin
  })
  .refine((d) => d.discount <= d.regularPrice, {
    message: "Chegirma narxdan katta bo'lmasin",
    path: ["discount"],
  });
```

---

### C6. [Middle] Bitta Zod sxemasini frontend va backend'da ishlatish mumkinmi?

Ha, agar backend ham JavaScript/TypeScript (Node.js) bo'lsa. Sxema umumiy paketda (monorepo) yoki alohida faylda turadi, frontend forma uchun, backend esa kelgan so'rovni tekshirish uchun ishlatadi. Natija: **qoidalar ikki joyda farq qilib qolmaydi**. Lekin frontend validatsiyasi faqat qulaylik uchun: backend tekshiruvi **har doim majburiy**, chunki so'rovni brauzerni chetlab o'tib ham yuborish mumkin.

---

## D. React Hook Form

### D1. [Junior] React Hook Form nima? Nega `useState` bilan forma yozish o'rniga?

**Oddiy qilib:** `useState` bilan har bir maydon uchun state, `onChange`, xato state'i, submit mantiqi qo'lda yoziladi va har harf yozilganda butun forma **qayta render** bo'ladi. React Hook Form (RHF) bularning hammasini bitta hook'da beradi va input'larni **uncontrolled** qoldiradi: har harfda re-render yo'q.

```tsx
import { useForm } from "react-hook-form";

type FormValues = { name: string; email: string };

function SignupForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormValues>();

  async function onSubmit(data: FormValues) {
    await createUser(data);
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("name", { required: "Ism majburiy" })} />
      {errors.name && <p>{errors.name.message}</p>}

      <input type="email" {...register("email", { required: "Email majburiy" })} />
      {errors.email && <p>{errors.email.message}</p>}

      <button disabled={isSubmitting}>Yuborish</button>
    </form>
  );
}
```

- **`register("name")`** input'ni formaga ulaydi: `name`, `ref`, `onChange`, `onBlur` ni beradi (shuning uchun spread qilinadi)
- **`handleSubmit(onSubmit)`** avval validatsiya qiladi, o'tsagina `onSubmit` ni chaqiradi
- **`formState.errors`** xatolar, **`isSubmitting`** yuborish jarayoni (async `onSubmit` tugaguncha `true`)

**Intervyuda shunday ayting:** "React Hook Form formalarni kam kod bilan boshqaradi: validatsiya, xatolar va submit bitta hook'da. Input'lar uncontrolled bo'lgani uchun har harfda butun forma qayta render bo'lmaydi, bu katta formalarda sezilarli farq. Men uni Zod bilan birga ishlataman."

---

### D2. [Strong junior] React Hook Form'ni Zod bilan qanday birlashtirasiz?

**Oddiy qilib:** Qoidalarni har bir `register` ichida yozish o'rniga, bitta Zod sxemasi beriladi. **Resolver** RHF va Zod orasidagi "tarjimon".

```tsx
import { zodResolver } from "@hookform/resolvers/zod";

const schema = z.object({
  email: z.string().email("Email noto'g'ri"),
  password: z.string().min(8, "Kamida 8 ta belgi"),
});
type LoginValues = z.infer<typeof schema>;

const { register, handleSubmit, formState: { errors } } = useForm<LoginValues>({
  resolver: zodResolver(schema),
  defaultValues: { email: "", password: "" },
});

<input {...register("email")} />
{errors.email?.message}
```

Foydalari: qoidalar **bitta joyda**, forma turi sxemadan chiqadi, sxemani testlash va qayta ishlatish oson.

---

### D3. [Junior] `defaultValues`, `reset`, `watch` va `setValue` nima uchun?

```tsx
const { register, reset, watch, setValue } = useForm<CabinForm>({
  defaultValues: isEdit ? cabinToEdit : { name: "", maxCapacity: 1 },   // tahrirlash formasi
});

const capacity = watch("maxCapacity");     // qiymatni kuzatish (o'zgarganda re-render)
setValue("discount", 0);                   // qiymatni kod orqali o'rnatish
reset();                                   // formani boshlang'ich holatga qaytarish
reset(newData);                            // yangi ma'lumot bilan to'ldirish (API'dan kelgach)
```

**Oddiy qilib:** `defaultValues` boshlang'ich qiymatlar (tahrirlash formasida eski ma'lumot), `reset` formani tozalash yoki qayta to'ldirish, `watch` bir maydon qiymatini jonli kuzatish (masalan, narxni darhol hisoblash uchun), `setValue` maydonga koddan qiymat yozish.

**Qo'shimcha savol:** "API'dan ma'lumot kelgach forma nega to'lmayapti?" → `defaultValues` faqat **birinchi** renderda o'qiladi. Ma'lumot keyin kelsa, `reset(data)` chaqiriladi.

---

### D4. [Strong junior] Controlled UI component'lar (Select, DatePicker, MUI) bilan qanday ishlaysiz?

**Oddiy qilib:** `register` oddiy HTML input'lar uchun (`ref` orqali). Lekin tayyor UI kutubxona component'lari ko'pincha `ref` qabul qilmaydi, ular `value` va `onChange` bilan ishlaydi. Ular uchun **`Controller`** ishlatiladi.

```tsx
import { Controller } from "react-hook-form";

<Controller
  name="startDate"
  control={control}
  rules={{ required: "Sana majburiy" }}
  render={({ field, fieldState }) => (
    <>
      <DatePicker selected={field.value} onChange={field.onChange} onBlur={field.onBlur} />
      {fieldState.error && <p>{fieldState.error.message}</p>}
    </>
  )}
/>
```

---

### D5. [Strong junior] Validatsiya qachon ishga tushadi? `mode` nima?

| `mode` | Qachon tekshiradi |
|---|---|
| `onSubmit` (default) | Faqat yuborishda |
| `onBlur` | Maydondan chiqqanda |
| `onChange` | Har o'zgarishda (ko'proq render) |
| `onTouched` | Birinchi blur'dan keyin, so'ng har o'zgarishda |
| `all` | Blur va change ikkalasida |

**Intervyuda shunday ayting:** "Foydalanuvchini yozayotganda xato bilan bezovta qilmaslik uchun ko'pincha `onTouched` yoki `onBlur` tanlayman: maydondan chiqqanda tekshiriladi, keyin tuzatayotganda jonli yangilanadi."

---

### D6. [Middle] Server qaytargan xatoni (masalan, "Bu email band") formada qanday ko'rsatasiz?

**Oddiy qilib:** Ba'zi xatolarni faqat server biladi. Ular `setError` bilan aniq maydonga biriktiriladi, foydalanuvchi uni oddiy validatsiya xatosi kabi ko'radi.

```tsx
const { setError } = useForm<SignupValues>();

async function onSubmit(data: SignupValues) {
  try {
    await signup(data);
  } catch (err) {
    if (err.code === "EMAIL_TAKEN") {
      setError("email", { message: "Bu email allaqachon ro'yxatdan o'tgan" });
    } else {
      setError("root", { message: "Server xatosi, keyinroq urinib ko'ring" });   // umumiy xato
    }
  }
}

{errors.root && <p>{errors.root.message}</p>}
```

---

### D7. [Middle] Dinamik maydonlar (qo'shiladigan va o'chiriladigan qatorlar) qanday qilinadi?

**Oddiy qilib:** Masalan, test savoliga variantlar qo'shish yoki buyurtmaga mahsulot qatorlari qo'shish. Buning uchun **`useFieldArray`**.

```tsx
const { control, register } = useForm<QuizForm>({ defaultValues: { options: [{ text: "" }] } });
const { fields, append, remove } = useFieldArray({ control, name: "options" });

{fields.map((field, index) => (
  <div key={field.id}>                                    {/* index emas, field.id */}
    <input {...register(`options.${index}.text`)} />
    <button type="button" onClick={() => remove(index)}>O'chirish</button>
  </div>
))}
<button type="button" onClick={() => append({ text: "" })}>Variant qo'shish</button>
```

Muhim: `key` uchun RHF beradigan `field.id` ishlatiladi, indeks emas (qatorlar o'chirilganda state aralashib ketmasligi uchun). Forma ichidagi qo'shimcha tugmalarga `type="button"` beriladi, aks holda ular formani yuboradi.

---

## E. State boshqaruvi: vositalarni tanlash

### E1. [Junior] Rezyumeda Context API, Redux, Redux Toolkit, Zustand va TanStack Query bor. Ularning har biri nima uchun?

**Oddiy qilib:** Avval state'ni **turlarga** ajratish kerak, keyin har turga mos vosita tanlanadi:

```
                      Qayerda saqlanadi?
                             │
        ┌────────────────────┼─────────────────────┐
        │                    │                     │
   SERVER STATE         CLIENT STATE          URL STATE
 (menyu, buyurtmalar,  (modal, tema, savat,  (filtr, sahifa,
  foydalanuvchilar)     o'yin xonasi)         tanlangan ID)
        │                    │                     │
  TanStack Query       ┌─────┴──────┐         useSearchParams
                       │            │
                  Lokal (bitta   Global (ko'p
                  component)     component)
                       │            │
                   useState     Context / Zustand / Redux Toolkit
```

| Vosita | Eng mos joy |
|---|---|
| `useState` / `useReducer` | Bitta component yoki kichik daraxt ichidagi state |
| **Context API** | Kam o'zgaradigan global qiymat: tema, til, auth foydalanuvchisi |
| **Zustand** | O'rta hajmdagi global client state, kam kod |
| **Redux Toolkit** | Katta ilova, ko'p domen, katta jamoa, qat'iy tuzilma |
| **TanStack Query** | Server state: kesh, loading, error, qayta yuklash |

**Intervyuda shunday ayting:** "Avval state turini aniqlayman. Server ma'lumoti TanStack Query'da. Faqat bitta joyda kerak bo'lsa `useState`. Global client state kam bo'lsa va kam o'zgarsa Context, ko'proq va tez o'zgaradigan bo'lsa Zustand, katta jamoaviy ilovada Redux Toolkit. Rezyumedagi ro'yxat turli loyihalarimdagi tajribani ko'rsatadi, bitta loyihada hammasini birga ishlatmayman."

---

### E2. [Strong junior] Context API'ning cheklovi nima? Uni qanday optimallaysiz?

**Oddiy qilib:** Context bu "radio to'lqini": Provider qiymatni efirga uzatadi, `useContext` bilan "tinglayotgan" har bir component qiymat o'zgarganda **qayta render** bo'ladi, qiymatning faqat kichik qismini ishlatsa ham. Shuning uchun tez-tez o'zgaradigan katta state uchun Context yomon tanlov.

```tsx
// Muammo: value har renderda yangi obyekt, barcha consumer'lar qayta render
<AuthContext.Provider value={{ user, login, logout }}>

// Yechim 1: memoizatsiya
const value = useMemo(() => ({ user, login, logout }), [user, login, logout]);

// Yechim 2: kontekstni bo'lish (state alohida, funksiyalar alohida)
<AuthStateContext.Provider value={user}>
  <AuthActionsContext.Provider value={actions}>
```

**Intervyuda shunday ayting:** "Context state boshqaruvi emas, ma'lumotni yetkazish mexanizmi. Uning kamchiligi: qiymat o'zgarsa, barcha consumer'lar render bo'ladi, selector yo'q. Shuning uchun men uni tema, til, auth kabi kam o'zgaradigan narsalar uchun ishlataman. Tez o'zgaradigan state uchun Zustand."

---

### E3. [Strong junior] Zustand qanday ishlaydi? Redux'dan nimasi bilan farq qiladi?

**Oddiy qilib:** Zustand bu kichik global "omborxona": bitta funksiya bilan store yaratasiz, Provider kerak emas, istalgan component hook orqali undan **faqat kerakli qismini** oladi va faqat o'sha qism o'zgarganda render bo'ladi.

```ts
import { create } from "zustand";

type CartState = {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: number) => void;
  clear: () => void;
};

export const useCartStore = create<CartState>((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) => set((state) => ({ items: state.items.filter((i) => i.id !== id) })),
  clear: () => set({ items: [] }),
}));
```

```tsx
// Selector bilan: faqat items soni o'zgarganda render bo'ladi
const count = useCartStore((state) => state.items.length);
const addItem = useCartStore((state) => state.addItem);

// React tashqarisida (masalan, API funksiyasida yoki socket handler'da)
useCartStore.getState().clear();
```

| | Zustand | Redux Toolkit |
|---|---|---|
| Provider | Kerak emas | Kerak |
| Kod hajmi | Juda kam | Ko'proq (slice, store, Provider) |
| Tuzilma | Erkin | Qat'iy (slice, action, reducer) |
| DevTools | Middleware bilan | Ichida tayyor |
| Kim uchun | Kichik va o'rta ilova | Katta ilova va jamoa |

**Qo'shimcha savol:** "Zustand'da state'ni localStorage'ga qanday saqlaysiz?" → `persist` middleware: `create(persist((set) => ({...}), { name: "cart" }))`.

---

### E4. [Strong junior] Zustand selector'ida obyekt qaytarsa nima bo'ladi?

**Oddiy qilib:** Selector har safar **yangi obyekt** qaytarsa (`{ a, b }`), Zustand uni "o'zgardi" deb hisoblaydi va component **har store yangilanishida** render bo'ladi. Yechimlar: har bir qiymatni alohida selector bilan olish yoki **`useShallow`** bilan yuzaki solishtirish.

```ts
// Yomon: har yangilanishda render
const { items, total } = useCartStore((s) => ({ items: s.items, total: s.total }));

// Yaxshi
import { useShallow } from "zustand/react/shallow";
const { items, total } = useCartStore(useShallow((s) => ({ items: s.items, total: s.total })));
```

---

### E5. [Junior] Klassik Redux va Redux Toolkit farqi nima? Nega rezyumeda ikkalasi ham bor?

**Oddiy qilib:** Klassik Redux'da hamma narsa qo'lda: action type konstantalari, action creator'lar, `switch` li reducer, immutable yangilash spread bilan, store va middleware sozlash. **Redux Toolkit (RTK)** buning hammasini qisqartiradi: `createSlice` reducer va action'larni birga yaratadi, ichidagi **Immer** "mutatsiya qilgandek" yozishga ruxsat beradi, `configureStore` DevTools va thunk'ni o'zi ulaydi.

```ts
// Klassik Redux
const ADD_ITEM = "cart/addItem";
function cartReducer(state = initialState, action) {
  switch (action.type) {
    case ADD_ITEM: return { ...state, items: [...state.items, action.payload] };
    default: return state;
  }
}

// Redux Toolkit
const cartSlice = createSlice({
  name: "cart",
  initialState,
  reducers: {
    addItem(state, action) { state.items.push(action.payload); },   // Immer
  },
});
```

**Intervyuda shunday ayting:** "Klassik Redux'ni tushunaman, chunki eski kod bazalarida uchraydi. Yangi kodda faqat Redux Toolkit ishlataman: rasmiy tavsiya etilgan usul va boilerplate ancha kam."

---

### E6. [Middle] Server ma'lumotini Redux yoki Zustand'da saqlash nega yomon fikr?

**Oddiy qilib:** Server ma'lumoti **boshqa joyda** yashaydi va vaqt o'tishi bilan **eskiradi**. Uni Redux'da saqlasangiz, quyidagilarni hammasini qo'lda yozasiz: loading va error holatlari, keshlash, qachon qayta yuklash, oyna fokusida yangilash, ikki component bir xil so'rovni ikki marta yubormasligi, mutatsiyadan keyin yangilash. TanStack Query bularning hammasini o'zi qiladi.

**Intervyuda shunday ayting:** "Server state va client state turli muammolar. TanStack Query paydo bo'lgach, ko'p loyihalarda Redux'dagi state'ning katta qismi aslida server ma'lumoti ekani ma'lum bo'ldi. Men server ma'lumotini TanStack Query'da, oz miqdordagi UI state'ni Zustand yoki Context'da saqlayman va bitta ma'lumotni ikki joyda saqlamayman."

---

### E7. [Middle] Bir loyihada TanStack Query, Zustand va React Hook Form qanday bo'linadi? (EasyCoddy misolida)

| Ma'lumot | Vosita | Sabab |
|---|---|---|
| Foydalanuvchi profili, darslar, natijalar | TanStack Query | Serverdan keladi, kesh va yangilash kerak |
| Joriy o'yin xonasi, o'yinchilar holati (socket) | Zustand | Tez o'zgaradi, ko'p component o'qiydi |
| Login, ro'yxatdan o'tish, sozlamalar formasi | React Hook Form + Zod | Forma state va validatsiya |
| Modal ochiqmi, tanlangan tab | `useState` | Faqat bitta component |
| Filtr, sahifa raqami | URL (`useSearchParams`) | Havolani ulashish va Back tugmasi ishlashi uchun |

`(o'z loyihangizdagi haqiqiy taqsimotni ayting)`

---

## F. Axios

### F1. [Junior] Axios va `fetch` farqi nima?

**Oddiy qilib:** Ikkalasi ham HTTP so'rov yuboradi. `fetch` brauzerga o'rnatilgan, Axios esa uning ustidagi qulayliklar to'plami.

| | `fetch` | Axios |
|---|---|---|
| O'rnatish | Kerak emas | `npm install axios` |
| JSON | `await res.json()` qo'lda | `res.data` da tayyor |
| 404/500 xatolar | Reject **qilmaydi**, `res.ok` ni tekshirish kerak | Avtomatik reject |
| Interceptor | Yo'q | Bor |
| Umumiy sozlama (baseURL, header) | Qo'lda | `axios.create()` |
| Timeout | `AbortSignal.timeout()` | `timeout` opsiyasi |

```ts
// fetch
const res = await fetch(`${API_URL}/cabins`);
if (!res.ok) throw new Error("Xato");
const cabins = await res.json();

// Axios
const { data: cabins } = await api.get("/cabins");
```

---

### F2. [Strong junior] Axios instance va interceptor nima? Qanday ishlatasiz?

**Oddiy qilib:** **Instance** bu umumiy sozlamali Axios nusxasi (manzil, header'lar). **Interceptor** esa "nazorat posti": har bir so'rov ketishidan **oldin** yoki har bir javob kelgandan **keyin** avtomatik ishlaydi. Tokenni har so'rovga qo'lda qo'shish o'rniga, bir marta interceptor'da yoziladi.

```ts
// api.ts
export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
});

// So'rov interceptor'i: har so'rovga token qo'shish
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Javob interceptor'i: 401 bo'lsa login'ga yo'naltirish
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem("token");
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);
```

**Intervyuda shunday ayting:** "Bitta Axios instance yarataman: baseURL va timeout bilan. Request interceptor har so'rovga tokenni qo'shadi, response interceptor esa 401'da tokenni tozalab login'ga yo'naltiradi. Shunda bu mantiq har bir API funksiyasida takrorlanmaydi."

---

### F3. [Middle] Access token muddati tugaganda refresh token bilan qanday yangilaysiz?

**Oddiy qilib:** Access token qisqa muddatli. U tugaganda server `401` qaytaradi. Interceptor shu `401` ni ushlaydi, **refresh token** bilan yangi access token oladi va **asl so'rovni qayta yuboradi**. Foydalanuvchi buni sezmaydi.

```ts
api.interceptors.response.use(
  (res) => res,
  async (error) => {
    const original = error.config;
    if (error.response?.status === 401 && !original._retry) {
      original._retry = true;                       // cheksiz takrorlanmasligi uchun
      const { data } = await axios.post("/auth/refresh", {}, { withCredentials: true });
      setAccessToken(data.accessToken);
      original.headers.Authorization = `Bearer ${data.accessToken}`;
      return api(original);                         // asl so'rovni qayta yuborish
    }
    return Promise.reject(error);
  }
);
```

Nozik joy: bir vaqtda bir nechta so'rov `401` olsa, ularning har biri alohida refresh qilmasligi uchun, birinchi refresh **navbat** qilinadi va qolganlar uni kutadi.

---

## G. Next.js

### Lug'at: Next.js kalit so'zlari

| So'z | Ma'nosi |
|---|---|
| **CSR** (Client-Side Rendering) | Sahifa brauzerda JavaScript bilan chiziladi (oddiy Vite SPA) |
| **SSR** (Server-Side Rendering) | Har so'rovda HTML serverda tayyorlanadi |
| **SSG** (Static Site Generation) | HTML build paytida bir marta tayyorlanadi |
| **ISR** (Incremental Static Regeneration) | Statik sahifa ma'lum vaqtdan keyin fonda yangilanadi |
| **Hydration** | Serverdan kelgan tayyor HTML'ga brauzerda JavaScript "jon kiritishi" (event'lar ulanadi) |
| **Server Component** | Faqat serverda ishlaydigan component, JavaScript'i brauzerga bormaydi |
| **Client Component** | Brauzerda ishlaydigan interaktiv component (`"use client"`) |
| **Streaming** | Sahifani qismlarga bo'lib, tayyor bo'lgan qismini darhol yuborish |
| **Server Action** | Serverda bajariladigan funksiya, formadan to'g'ridan-to'g'ri chaqiriladi |
| **Route handler** | Next.js ichidagi API endpoint (`route.ts`) |
| **Revalidation** | Keshdagi ma'lumotni yangilash |

---

### G1. [Junior] Next.js nima? Oddiy React (Vite) ilovasidan farqi?

**Oddiy qilib:** React faqat UI kutubxonasi. Vite + React ilovasida server bo'sh HTML beradi va butun sahifa brauzerda JavaScript bilan chiziladi. **Next.js** React ustidagi **framework**: sahifani **serverda** tayyorlab bera oladi, routing'ni fayllar orqali qiladi, rasm va shriftlarni optimallaydi, kichik backend (API) yozishga ham imkon beradi.

```
Vite SPA:     Brauzer → bo'sh HTML → JS yuklanadi → sahifa chiziladi → ma'lumot so'raladi
Next.js SSR:  Brauzer → TAYYOR HTML (ma'lumot bilan) → JS yuklanadi → hydration
```

Natija: foydalanuvchi kontentni **tezroq** ko'radi va qidiruv tizimlari sahifani **yaxshiroq indekslaydi** (SEO).

**Intervyuda shunday ayting:** "Next.js React framework'i: fayl asosidagi routing, server-side rendering va statik generatsiya, Server Components, API route'lar va rasm optimallash beradi. SEO va birinchi yuklanish tezligi muhim bo'lgan saytlarda uni tanlayman."

---

### G2. [Junior] App Router'da routing qanday ishlaydi? Maxsus fayllar qaysilar?

**Oddiy qilib:** `app/` papkasidagi **papkalar URL** bo'ladi, papka ichidagi `page.tsx` esa shu URL'dagi sahifa.

```
app/
├── layout.tsx              → barcha sahifalar uchun umumiy qobiq (html, body, header)
├── page.tsx                → /
├── loading.tsx             → yuklanish holati
├── error.tsx               → xato holati
├── not-found.tsx           → 404
├── cabins/
│   ├── page.tsx            → /cabins
│   └── [cabinId]/
│       └── page.tsx        → /cabins/12   (dinamik)
├── (auth)/                 → route group: URL'ga qo'shilmaydi, faqat tartib uchun
│   └── login/page.tsx      → /login
└── api/
    └── cabins/route.ts     → /api/cabins  (API endpoint)
```

| Fayl | Vazifa |
|---|---|
| `page.tsx` | Sahifaning o'zi |
| `layout.tsx` | Umumiy qobiq. Navigatsiyada **saqlanib qoladi**, qayta render bo'lmaydi |
| `loading.tsx` | Sahifa yuklanguncha avtomatik ko'rsatiladi (ichida `Suspense`) |
| `error.tsx` | Xato chegarasi. **Client component** bo'lishi shart, `reset` funksiyasini oladi |
| `not-found.tsx` | `notFound()` chaqirilganda yoki sahifa topilmasa |
| `route.ts` | API endpoint: `export async function GET() {}` |

---

### G3. [Junior] Dinamik route'dagi parametrni qanday olasiz?

```tsx
// app/cabins/[cabinId]/page.tsx
export default async function CabinPage({ params }: { params: Promise<{ cabinId: string }> }) {
  const { cabinId } = await params;          // yangi versiyalarda params Promise
  const cabin = await getCabin(cabinId);
  if (!cabin) notFound();                    // 404 sahifasini ko'rsatish
  return <h1>{cabin.name}</h1>;
}
```

Next.js 15 dan boshlab `params` va `searchParams` **Promise** bo'ldi, shuning uchun `await` bilan olinadi. Eski versiyalarda (14 va undan oldin) ular oddiy obyekt edi. Client component ichida esa `useParams()` hook'i ishlatiladi.

---

### G4. [Strong junior] Server Component va Client Component farqi nima?

**Oddiy qilib:** App Router'da har bir component **default holda Server Component**: u faqat serverda ishlaydi, ma'lumotni bazadan yoki API'dan to'g'ridan-to'g'ri oladi va brauzerga faqat **tayyor natija** (HTML) boradi, uning JavaScript kodi bormaydi. Agar component'ga interaktivlik kerak bo'lsa (state, click, effect), faylning boshiga `"use client"` yoziladi.

```tsx
// Server Component (default): async bo'lishi mumkin
export default async function CabinList() {
  const cabins = await getCabins();          // to'g'ridan-to'g'ri, useEffect'siz
  return cabins.map((c) => <CabinCard key={c.id} cabin={c} />);
}

// Client Component
"use client";
import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
}
```

| | Server Component | Client Component |
|---|---|---|
| Qayerda ishlaydi | Serverda | Serverda (birinchi HTML) **va** brauzerda |
| `useState`, `useEffect`, `onClick` | Yo'q | Ha |
| Bazaga, maxfiy kalitga murojaat | Ha (xavfsiz) | Yo'q |
| JavaScript brauzerga boradimi | Yo'q | Ha |
| `async` bo'la oladimi | Ha | Yo'q |

**Intervyuda shunday ayting:** "Imkon qadar Server Component ishlataman: brauzerga kam JavaScript boradi, ma'lumot serverda olinadi va maxfiy kalitlar xavfsiz. Interaktivlik kerak bo'lgan kichik qismlarni, masalan tugma, forma yoki filtrni, `"use client"` bilan ajrataman."

**Qo'shimcha savol:** "Client component ham serverda render bo'ladimi?" → Ha, birinchi HTML uchun serverda ham render bo'ladi, keyin brauzerda hydration qilinadi. "Client" so'zi "faqat brauzerda" degani emas, "brauzerda ham interaktiv ishlaydi" degani.

---

### G5. [Strong junior] Server va Client Component'larni qanday birga ishlatasiz? Qanday cheklovlar bor?

**Oddiy qilib:** Server component client component'ni **import qilib** ishlata oladi va unga props beradi. Lekin props **serializatsiya qilinadigan** bo'lishi kerak (son, string, oddiy obyekt, massiv): **funksiya bera olmaydi**, chunki u serverdan brauzerga "jo'natiladi". Client component esa server component'ni import qila olmaydi, lekin uni **`children`** sifatida qabul qila oladi.

```tsx
// Server component
import Filter from "./Filter";        // client component
import CabinList from "./CabinList";  // server component

export default function Page() {
  return (
    <Modal>                           {/* client component */}
      <CabinList />                   {/* server component children sifatida: ruxsat */}
    </Modal>
  );
}
```

Qoida: `"use client"` chegarasini daraxtning **iloji boricha pastiga** qo'ying. Butun sahifani client qilsangiz, Server Component'ning foydasi yo'qoladi.

---

### G6. [Strong junior] Hydration nima? "Hydration error" nega chiqadi?

**Oddiy qilib:** Server tayyor HTML yuboradi, foydalanuvchi uni darhol ko'radi, lekin tugmalar hali ishlamaydi. Keyin JavaScript yuklanib, React o'sha HTML'ga **"jon kiritadi"**: event'larni ulaydi. Bu **hydration**. Xato shunda chiqadi: React brauzerda render qilgan natija serverdagi HTML bilan **mos kelmasa**.

Tipik sabablar:

```tsx
<p>{new Date().toLocaleTimeString()}</p>     // server va brauzerda vaqt har xil
<p>{Math.random()}</p>                       // har safar har xil
{typeof window !== "undefined" && <Banner />} // serverda yo'q, brauzerda bor
{localStorage.getItem("theme")}              // serverda localStorage yo'q
```

Yechim: brauzerga xos qiymatlarni **`useEffect` ichida** o'qish (u faqat brauzerda, hydration'dan keyin ishlaydi), yoki component'ni faqat client'da render qilish.

```tsx
const [time, setTime] = useState<string | null>(null);
useEffect(() => setTime(new Date().toLocaleTimeString()), []);
```

---

### G7. [Strong junior] SSR, SSG va ISR farqi nima? Qachon qaysi biri?

**Oddiy qilib:** Restoran o'xshatishi: **SSG** oldindan pishirib qo'yilgan taom (darhol beriladi, lekin hamma uchun bir xil), **SSR** buyurtma bo'yicha pishiriladigan taom (har kimga yangi, lekin kutish kerak), **ISR** oldindan pishirilgan, lekin har soatda yangilanib turadigan taom.

| | Qachon HTML yaratiladi | Tezlik | Ma'lumot yangiligi | Misol |
|---|---|---|---|---|
| **SSG** | Build paytida | Eng tez | Build'dagi holat | Landing, blog, hujjat |
| **ISR** | Build'da + har N sekundda fonda | Tez | Deyarli yangi | Mahsulotlar katalogi, kabinalar ro'yxati |
| **SSR** | Har so'rovda | Sekinroq | Har doim yangi | Shaxsiy kabinet, savat, qidiruv natijalari |
| **CSR** | Brauzerda | Birinchi ko'rinish sekin | Har doim yangi | Login orqasidagi dashboard |

```tsx
// ISR: sahifa har 1 soatda yangilanadi
export const revalidate = 3600;

// Dinamik sahifalarni build paytida yaratish (SSG)
export async function generateStaticParams() {
  const cabins = await getCabins();
  return cabins.map((c) => ({ cabinId: String(c.id) }));
}
```

Next.js'da keshlash qoidalari versiyadan versiyaga o'zgargan: masalan, Next.js 15 da `fetch` so'rovlari default holda **kesh qilinmaydigan** bo'ldi, yangi versiyalarda esa keshlash aniq (opt-in) usullarga o'tmoqda. Intervyuda "loyihadagi versiya hujjatiga qarayman" deyish to'g'ri.

---

### G8. [Strong junior] Next.js'da ma'lumot qanday olinadi va yangilanadi?

**Oddiy qilib:** Server Component'da ma'lumot to'g'ridan-to'g'ri `await` bilan olinadi, `useEffect` va loading state kerak emas. Bir-biriga bog'liq bo'lmagan so'rovlar **parallel** yuboriladi. Ma'lumot o'zgargach (masalan, Server Action'dan keyin), keshdagi sahifa **`revalidatePath`** yoki **`revalidateTag`** bilan yangilanadi.

```tsx
export default async function Dashboard() {
  const [bookings, stats] = await Promise.all([getBookings(), getStats()]);   // parallel
  return <>...</>;
}

// Sekin qismni alohida yuklash (streaming)
<Suspense fallback={<Spinner />}>
  <SlowChart />
</Suspense>
```

Client component'da interaktiv ma'lumot (masalan, filtrlash, cheksiz scroll) uchun hamon **TanStack Query** ishlatish mumkin.

---

### G9. [Strong junior] Server Actions nima? Qanday ishlatiladi?

**Oddiy qilib:** Server Action bu **serverda bajariladigan funksiya**, uni formaga yoki tugmaga to'g'ridan-to'g'ri ulash mumkin. Alohida API endpoint yozish, `fetch` qilish va JSON bilan ishlash shart emas: Next.js buni o'zi qiladi.

```tsx
// actions.ts
"use server";

export async function updateProfile(formData: FormData) {
  const session = await auth();
  if (!session) throw new Error("Login qiling");          // HAR DOIM tekshirish!

  const name = formData.get("name") as string;
  await db.user.update({ where: { id: session.user.id }, data: { name } });
  revalidatePath("/account");                              // sahifani yangilash
}
```

```tsx
// Server component ichida forma
<form action={updateProfile}>
  <input name="name" />
  <SubmitButton />
</form>

// Client component: yuborish holati
"use client";
import { useFormStatus } from "react-dom";
function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? "Saqlanmoqda..." : "Saqlash"}</button>;
}
```

Muhim xavfsizlik nuqtasi: Server Action aslida **ochiq HTTP endpoint**. Uni istalgan kishi chaqira oladi, shuning uchun ichida **foydalanuvchini va huquqni tekshirish** va ma'lumotni (Zod bilan) validatsiya qilish majburiy.

**Intervyuda shunday ayting:** "Server Actions ma'lumot o'zgartirishni API endpoint yozmasdan, forma orqali to'g'ridan-to'g'ri serverda bajarishga imkon beradi. React Router'dagi action'larga o'xshaydi, lekin serverda ishlaydi. Ichida doim auth va validatsiyani tekshiraman, chunki bu ochiq endpoint."

---

### G10. [Junior] `next/link`, `next/image` va `next/font` nima beradi?

- **`<Link>`**: sahifani qayta yuklamasdan o'tish va ko'rinib turgan havolalarni **oldindan yuklash** (prefetch), shuning uchun o'tish juda tez
- **`<Image>`**: rasmni avtomatik zamonaviy formatga (`webp`/`avif`) va ekran o'lchamiga moslash, default holda **lazy loading**, `width`/`height` majburiy bo'lgani uchun sahifa **sakramaydi** (CLS). Muhim (birinchi ekrandagi) rasmlar uchun `priority`
- **`next/font`**: shriftni build paytida yuklab, o'z serveringizdan beradi, matn "sakrashi"ni oldini oladi

```tsx
import Image from "next/image";
<Image src="/hero.webp" alt="The Wild Oasis" width={1200} height={600} priority />
```

---

### G11. [Junior] Next.js'da SEO uchun metadata qanday yoziladi?

```tsx
// Statik
export const metadata = {
  title: "The Wild Oasis",
  description: "Luxury cabins in the Italian Dolomites",
};

// Dinamik (sahifaga qarab)
export async function generateMetadata({ params }) {
  const { cabinId } = await params;
  const cabin = await getCabin(cabinId);
  return { title: `Cabin ${cabin.name}` };
}
```

Qo'shimcha: `sitemap.ts`, `robots.ts`, Open Graph rasmlari, semantik HTML.

---

### G12. [Strong junior] Environment variable'lar Next.js'da qanday ishlaydi? Qaysilari xavfsiz?

**Oddiy qilib:** Next.js'da **ikki xil** o'zgaruvchi bor:

- **`NEXT_PUBLIC_`** bilan boshlanadiganlar brauzerga **yuboriladi**: istalgan kishi ko'radi. Faqat ochiq qiymatlar uchun (API manzili, Supabase publishable kaliti)
- Prefikssizlar **faqat serverda** mavjud: maxfiy kalitlar (AI API kaliti, Supabase secret kaliti, ma'lumotlar bazasi paroli)

```ts
process.env.NEXT_PUBLIC_SUPABASE_URL   // brauzerda ham bor
process.env.OPENAI_API_KEY             // faqat Server Component, Server Action, route.ts'da
```

Bu Vite'dagi `VITE_` qoidasiga o'xshaydi. Afzallik: Next.js'da maxfiy kalitni ishlatadigan kod **serverda** qolishi mumkin, shuning uchun AI integratsiyasi (Happy Quiz kabi) xavfsiz qilinadi.

---

### G13. [Strong junior] Next.js'da autentifikatsiya va himoyalangan sahifalar qanday qilinadi?

**Oddiy qilib:** Ikki qatlam: **so'rov darajasida** (middleware, yangi versiyalarda `proxy` deb ataladi) foydalanuvchini sahifaga yetib bormasdan login'ga yo'naltirish, va **ma'lumot darajasida** har bir Server Component, Server Action va route handler ichida sessiyani qayta tekshirish. Kutubxona sifatida ko'pincha **Auth.js (NextAuth)**.

```ts
// middleware.ts (yoki yangi versiyalarda proxy.ts)
export function middleware(request: NextRequest) {
  const token = request.cookies.get("session");
  if (!token) return NextResponse.redirect(new URL("/login", request.url));
}
export const config = { matcher: ["/account/:path*"] };
```

**Intervyuda shunday ayting:** "Middleware tez yo'naltirish uchun, lekin men faqat unga tayanmayman: ma'lumot qaytaradigan yoki o'zgartiradigan har bir joyda sessiya va huquqni qayta tekshiraman. Sessiyani `httpOnly` cookie'da saqlayman."

---

### G14. [Junior] Client'da navigatsiya va URL bilan ishlash uchun qaysi hook'lar bor?

```tsx
"use client";
import { useRouter, usePathname, useSearchParams } from "next/navigation";

const router = useRouter();
router.push("/cabins");            // dasturiy o'tish
router.refresh();                  // server ma'lumotini yangilash

const pathname = usePathname();    // "/cabins/12": faol menyu elementini belgilash uchun

const searchParams = useSearchParams();
const filter = searchParams.get("capacity") ?? "all";
```

Filtrni URL'ga yozish (`?capacity=small`) uni server component ham o'qiy olishiga imkon beradi (`searchParams` prop orqali), havola ulashilsa ham filtr saqlanadi. Muhim: `next/router` (Pages Router) emas, **`next/navigation`** dan import qilinadi.

---

### G15. [Middle] Qachon Next.js, qachon Vite SPA tanlaysiz?

| Vaziyat | Tanlov | Sabab |
|---|---|---|
| Ochiq sayt, SEO muhim (e-commerce, blog, landing) | **Next.js** | SSR/SSG, metadata |
| Birinchi yuklanish tezligi muhim | **Next.js** | Tayyor HTML, kam JavaScript |
| Server mantiq, maxfiy kalitlar kerak (AI, to'lov) | **Next.js** | Server Actions, route handlers |
| Login orqasidagi dashboard yoki admin panel | **Vite SPA** yetarli | SEO kerak emas, oddiyroq |
| Statik hosting, alohida backend bor | **Vite SPA** | Oddiy deploy |
| Real-time o'yin, juda interaktiv ilova | **Vite SPA** (yoki Next.js'da asosan client) | Server rendering foydasi kam |

**Intervyuda shunday ayting:** "Masalan, Bogizilol'ning mijozlar qismi menyuni qidiruvda topish uchun Next.js'da yaxshiroq bo'lardi, admin panel esa Vite SPA'da ham to'liq yetarli. Tanlov SEO, birinchi yuklanish va server mantiqi kerakligiga bog'liq."

---

### G16. [Middle] Pages Router va App Router farqi nima?

| | Pages Router (`pages/`) | App Router (`app/`) |
|---|---|---|
| Ma'lumot olish | `getServerSideProps`, `getStaticProps` | `async` Server Component'da `await` |
| Default component turi | Client | **Server** |
| Layout | `_app.tsx`, qo'lda | `layout.tsx`, ichma-ich |
| Yuklanish va xato holatlari | Qo'lda | `loading.tsx`, `error.tsx` |
| Mutatsiya | API route'lar | Server Actions |
| Router hook'i | `next/router` | `next/navigation` |

Yangi loyihalar **App Router**da qilinadi. Pages Router mavjud eski loyihalarda uchraydi, shuning uchun uning asosiy tushunchalarini bilish foydali.
