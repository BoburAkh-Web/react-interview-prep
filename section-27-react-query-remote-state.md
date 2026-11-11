# Section 27: React Query: Managing Remote State

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: The Wild Oasis (3-qism: kabinalar CRUD, React Hook Form, rasm yuklash, custom hook'lar, sozlamalar).

> **Versiya eslatmasi:** kurs **React Query v4** bilan yozilgan, ish joylarida esa **v5** ishlatiladi. Asosiy farq: v5 da `isLoading` o'rniga **`isPending`**, `cacheTime` o'rniga **`gcTime`**, `useQuery` dagi `onSuccess`/`onError` olib tashlangan (`useMutation` da qoladi). Quyidagi misollar **v5** ko'rinishida.

---

## 1. [Junior] React Query nima? Qanday muammoni hal qiladi?

React Query (TanStack Query) bu **server (remote) state**ni boshqarish kutubxonasi. Server state client state'dan farq qiladi: u **boshqa joyda** (serverda) saqlanadi, boshqalar uni o'zgartirishi mumkin va bizdagi nusxa **eskirib qolishi** mumkin.

React Query avtomatik beradi:

- **Keshlash**: bir xil ma'lumot qayta so'ralmaydi, darhol keshdan ko'rsatiladi
- **Loading va error holatlari**
- **Fonda qayta yuklash** (oyna fokus olganda, internet qaytganda)
- **Eskirgan ma'lumotni yangilash** (stale time)
- Mutatsiyadan keyin **keshni bekor qilish** (invalidation)
- Prefetching, pagination, retry

`useEffect` + `fetch` bilan bularning hammasini qo'lda yozish kerak edi (Section 12, 24-savol).

## 2. [Junior] React Query qanday sozlanadi?

```jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: { staleTime: 60 * 1000 },   // 1 daqiqa "yangi" hisoblanadi
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <ReactQueryDevtools initialIsOpen={false} />
      ...
    </QueryClientProvider>
  );
}
```

`QueryClient` keshni saqlaydi va butun ilovaga Provider orqali beriladi. DevTools keshdagi so'rovlarni, ularning holatini va ma'lumotini ko'rsatadi.

## 3. [Junior] `useQuery` bilan ma'lumot qanday olinadi? Query key nima?

```jsx
const { isPending, data: cabins, error } = useQuery({
  queryKey: ["cabins"],
  queryFn: getCabins,
});
```

- **`queryKey`**: keshdagi ma'lumotning **noyob identifikatori** (massiv). Bir xil key'li so'rovlar bitta kesh yozuvini ulashadi. Parametr bo'lsa key'ga qo'shiladi: `["booking", bookingId]`, `["bookings", filter, page]`, shunda parametr o'zgarganda yangi so'rov ketadi
- **`queryFn`**: Promise qaytaradigan funksiya (bizda `services/` dagi API funksiyasi). Xato bo'lsa **throw** qilishi kerak
- Qaytgan holatlar: `isPending` (birinchi yuklanish, ma'lumot yo'q), `isFetching` (fonda yangilanish ham), `error`, `data`

## 4. [Strong junior] `staleTime` va `gcTime` (eski nomi `cacheTime`) farqi nima?

- **`staleTime`**: ma'lumot qancha vaqt **yangi (fresh)** hisoblanadi. Shu vaqt ichida qayta so'rov **yuborilmaydi**. Default: `0`, ya'ni ma'lumot darhol "stale" bo'ladi va keyingi mount yoki oyna fokusida fonda qayta yuklanadi
- **`gcTime`**: query **ishlatilmay qolgandan** (hech qaysi component uni o'qimasa) keyin keshdan qancha vaqtda o'chiriladi. Default: 5 daqiqa

Ya'ni stale ma'lumot ham keshdan **darhol ko'rsatiladi**, yangilanishi fonda ketadi (stale-while-revalidate), shuning uchun foydalanuvchi yuklanishni kam ko'radi.

## 5. [Strong junior] `useMutation` va keshni bekor qilish (invalidation) qanday ishlaydi? (Kabinani o'chirish)

```jsx
const queryClient = useQueryClient();

const { isPending: isDeleting, mutate: deleteCabin } = useMutation({
  mutationFn: deleteCabinApi,
  onSuccess: () => {
    toast.success("Cabin successfully deleted");
    queryClient.invalidateQueries({ queryKey: ["cabins"] });
  },
  onError: (err) => toast.error(err.message),
});

<button onClick={() => deleteCabin(cabinId)} disabled={isDeleting}>Delete</button>
```

- **`useMutation`** ma'lumotni **o'zgartiradigan** so'rovlar (create, update, delete) uchun. U avtomatik ishga tushmaydi, `mutate()` chaqirilganda ishlaydi
- **`invalidateQueries`** `["cabins"]` keshini **eskirgan** deb belgilaydi, natijada ro'yxat **avtomatik qayta yuklanadi** va UI yangilanadi. Ro'yxatni qo'lda yangilash shart emas
- `onSuccess`, `onError`, `onSettled` callback'lari natijaga qarab ishlaydi

## 6. [Junior] Toast bildirishnomalari nima uchun? Qanday ishlatiladi?

Foydalanuvchiga harakat natijasini (muvaffaqiyat yoki xato) qisqa xabar bilan bildirish. Kursda **`react-hot-toast`**:

```jsx
<Toaster position="top-center" toastOptions={{ success: { duration: 3000 } }} />

toast.success("Cabin successfully created");
toast.error(err.message);
```

`<Toaster />` bir marta (App'da) render qilinadi, `toast()` funksiyasi esa istalgan joydan, shu jumladan mutation callback'laridan chaqiriladi.

## 7. [Junior] React Hook Form nima? Nega u qulay?

Formalarni boshqarish kutubxonasi. Input'lar **uncontrolled** (har harf uchun re-render yo'q), validatsiya, xatolar va submit bitta hook'da.

```jsx
const { register, handleSubmit, reset, getValues, formState } = useForm();
const { errors } = formState;

function onSubmit(data) { createCabin(data); }
function onError(errors) { console.log(errors); }

<form onSubmit={handleSubmit(onSubmit, onError)}>
  <input id="name" {...register("name", { required: "This field is required" })} />
  {errors?.name?.message && <Error>{errors.name.message}</Error>}
</form>
```

- **`register`** input'ni formaga ulaydi (`name`, `ref`, `onChange`, `onBlur` beradi)
- **`handleSubmit`** validatsiyadan o'tsa `onSubmit` ni, aks holda `onError` ni chaqiradi
- **`formState.errors`**: xatolar obyekti
- **`reset()`**: formani tozalaydi (masalan, muvaffaqiyatli yaratishdan keyin)

`useState` + `onChange` bilan har bir maydonni qo'lda yozishdan ancha qisqa va performance yaxshiroq.

## 8. [Strong junior] React Hook Form'da validatsiya qanday yoziladi? Maydonlar bir-biriga bog'liq bo'lsa-chi?

```jsx
{...register("maxCapacity", {
  required: "This field is required",
  min: { value: 1, message: "Capacity should be at least 1" },
})}

{...register("discount", {
  required: "This field is required",
  validate: (value) =>
    Number(value) <= Number(getValues().regularPrice) ||
    "Discount should be less than regular price",
})}
```

- Standart qoidalar: `required`, `min`, `max`, `minLength`, `pattern`
- **`validate`** maxsus funksiya: `true` qaytarsa o'tadi, string qaytarsa shu xato xabari ko'rsatiladi
- **`getValues()`** boshqa maydonlarning qiymatini o'qiydi (chegirma narxdan kichik bo'lishi kerak)
- Input qiymatlari **string**, shuning uchun solishtirishda `Number()` kerak (yoki `valueAsNumber: true`)

Katta loyihalarda validatsiya ko'pincha **Zod** sxemasi bilan (`zodResolver`) yoziladi: bitta sxema ham validatsiya, ham TypeScript tipi bo'ladi.

## 9. [Strong junior] Rasmni Supabase Storage'ga qanday yuklaymiz?

```js
export async function createEditCabin(newCabin, id) {
  const hasImagePath = newCabin.image?.startsWith?.(supabaseUrl);

  const imageName = `${Math.random()}-${newCabin.image.name}`.replaceAll("/", "");
  const imagePath = hasImagePath
    ? newCabin.image
    : `${supabaseUrl}/storage/v1/object/public/cabin-images/${imageName}`;

  // 1. Kabinani bazaga yozish (rasm URL'i bilan)
  const { data, error } = await supabase.from("cabins").insert([{ ...newCabin, image: imagePath }]).select().single();
  if (error) throw new Error("Cabin could not be created");

  // 2. Rasm allaqachon yuklangan bo'lsa, qayta yuklamaymiz
  if (hasImagePath) return data;

  // 3. Rasmni storage'ga yuklash
  const { error: storageError } = await supabase.storage.from("cabin-images").upload(imageName, newCabin.image);

  // 4. Rasm yuklanmasa, yaratilgan kabinani o'chirish (izchillik uchun)
  if (storageError) {
    await supabase.from("cabins").delete().eq("id", data.id);
    throw new Error("Cabin image could not be uploaded and the cabin was not created");
  }
  return data;
}
```

Asosiy nuqtalar:

- Fayl input'idan `FileList` keladi, birinchi faylni olamiz: `data.image[0]`
- **Noyob nom** (tasodifiy prefiks), `/` belgilarini olib tashlash (aks holda papka yaratiladi)
- Bazada faqat **URL**, faylning o'zi storage'da
- Rasm yuklanmasa, bazadagi yozuvni **qaytarib o'chirish**: bu ikki xizmat orasida izchillikni saqlashning oddiy yo'li (to'liq tranzaksiya emas)

## 10. [Strong junior] Bitta formani ham yaratish, ham tahrirlash uchun qanday ishlatamiz?

```jsx
function CreateCabinForm({ cabinToEdit = {} }) {
  const { id: editId, ...editValues } = cabinToEdit;
  const isEditSession = Boolean(editId);

  const { register, handleSubmit, reset } = useForm({
    defaultValues: isEditSession ? editValues : {},
  });

  function onSubmit(data) {
    const image = typeof data.image === "string" ? data.image : data.image[0];
    if (isEditSession)
      editCabin({ newCabinData: { ...data, image }, id: editId }, { onSuccess: () => reset() });
    else
      createCabin({ ...data, image }, { onSuccess: () => reset() });
  }
  ...
  <input type="file" {...register("image", { required: isEditSession ? false : "This field is required" })} />
}
```

- `cabinToEdit` bor bo'lsa, **edit rejimi**; forma `defaultValues` bilan to'ldiriladi
- Tahrirlashda rasm **majburiy emas**: yangi rasm tanlanmasa, eski URL (string) saqlanadi
- API funksiyasi `id` bor-yo'qligiga qarab `insert` yoki `update` qiladi
- `mutate` ga ikkinchi argument sifatida **`{ onSuccess }`** berib, component darajasidagi ishni (formani tozalash, modalni yopish) bajarish mumkin
- `mutate` faqat **bitta argument** qabul qiladi, shuning uchun bir nechta qiymat obyektga yig'iladi (`{ newCabinData, id }`)

## 11. [Strong junior] React Query mantiqini custom hook'larga nega ajratamiz?

```js
// features/cabins/useCabins.js
export function useCabins() {
  const { isPending, data: cabins, error } = useQuery({ queryKey: ["cabins"], queryFn: getCabins });
  return { isPending, cabins, error };
}

// features/cabins/useDeleteCabin.js
export function useDeleteCabin() {
  const queryClient = useQueryClient();
  const { isPending: isDeleting, mutate: deleteCabin } = useMutation({
    mutationFn: deleteCabinApi,
    onSuccess: () => {
      toast.success("Cabin successfully deleted");
      queryClient.invalidateQueries({ queryKey: ["cabins"] });
    },
    onError: (err) => toast.error(err.message),
  });
  return { isDeleting, deleteCabin };
}
```

Foydalari:

- Component faqat **nima kerak**ligini biladi (`useCabins()`), query key, API va toast'lar yashirin
- Bir xil query **bir xil key** bilan ko'p joyda ishlatiladi (key'dagi typo xavfi yo'qoladi)
- O'zgartirish **bir joyda** (masalan, toast matni yoki invalidation)
- Component qisqa va o'qilishi oson

## 12. [Junior] Kabinani nusxalash (duplicate) qanday qilindi?

Yangi mantiq yozilmadi, mavjud **`createCabin`** mutatsiyasi qayta ishlatildi:

```js
function handleDuplicate() {
  createCabin({
    name: `Copy of ${name}`,
    maxCapacity, regularPrice, discount, image, description,
  });
}
```

Rasm URL'i allaqachon mavjud (string) bo'lgani uchun API uni qayta yuklamaydi. Custom hook'lar tufayli bu bir necha qator bo'ldi: qayta ishlatishning amaliy foydasi.

## 13. [Strong junior] Sozlamalar (settings) qanday olinadi va yangilanadi? Nega `onBlur`?

Settings jadvalida **bitta qator** bor, `useSettings` uni oladi (`.single()`), `useUpdateSetting` esa yangilaydi.

```jsx
function handleUpdate(e, field) {
  const { value } = e.target;
  if (!value) return;
  updateSetting({ [field]: value });   // computed property name
}

<Input type="number" defaultValue={minBookingLength} disabled={isUpdating}
       onBlur={(e) => handleUpdate(e, "minBookingLength")} />
```

- Alohida "Saqlash" tugmasi yo'q: input fokusni yo'qotganda (**`onBlur`**) avtomatik saqlanadi. Har harfda emas, chunki har bir bosish yangi so'rov yuborardi
- `{ [field]: value }` bilan bitta funksiya **har qanday maydonni** yangilaydi
- Input'lar **uncontrolled** (`defaultValue`), qiymat React Query keshidan keladi
- Yaxshilash: qiymat avvalgisi bilan bir xil bo'lsa, so'rov yubormaslik

## 14. [Middle] React Query client state'ni ham boshqara oladimi? Redux bilan birga ishlatish kerakmi?

React Query **faqat server state** uchun. Client state (modal ochiqmi, dark mode, forma qoralamasi, tanlangan filtr) uchun `useState`, Context, URL yoki Zustand ishlatiladi. Wild Oasis'da server state React Query'da, UI state esa Context (dark mode) va URL'da (filtr, saralash, sahifa).

Ko'p ilovalarda React Query paydo bo'lgach **Redux kerak bo'lmay qoladi**, chunki Redux'dagi state'ning katta qismi aslida server ma'lumoti edi. Redux bilan birga ishlatish mumkin, lekin bir ma'lumotni **ikki joyda** (Redux va React Query keshida) saqlamaslik kerak.

## 15. [Middle] Optimistik yangilash (optimistic update) nima? Qachon kerak?

Odatda UI **server tasdiqlagandan keyin** yangilanadi (pessimistik). Optimistik yangilashda UI **darhol** o'zgaradi, server javobi kelguncha kutilmaydi, xato bo'lsa esa oldingi holatga qaytariladi:

```js
useMutation({
  mutationFn: deleteCabinApi,
  onMutate: async (id) => {
    await queryClient.cancelQueries({ queryKey: ["cabins"] });
    const previous = queryClient.getQueryData(["cabins"]);
    queryClient.setQueryData(["cabins"], (old) => old.filter((c) => c.id !== id));
    return { previous };
  },
  onError: (err, id, context) => queryClient.setQueryData(["cabins"], context.previous),
  onSettled: () => queryClient.invalidateQueries({ queryKey: ["cabins"] }),
});
```

Foyda: ilova juda tez seziladi (like, todo belgilash, o'chirish). Kamchilik: murakkabroq kod va xato bo'lsa UI "orqaga sakraydi". Muhim va qaytarib bo'lmaydigan amallarda (to'lov) pessimistik yondashuv xavfsizroq.

## 16. [Middle] `invalidateQueries` va `setQueryData` farqi nima?

- **`invalidateQueries`**: keshni eskirgan deb belgilaydi va **serverdan qayta yuklaydi**. Oddiy va ishonchli, lekin qo'shimcha so'rov
- **`setQueryData`**: keshni **qo'lda, darhol** yangilaydi (masalan, mutation javobi bilan), qo'shimcha so'rov yo'q. Tezroq, lekin ma'lumot serverdagi bilan mos kelishini o'zingiz ta'minlaysiz

Ko'pincha `invalidateQueries` yetarli. Javobda yangilangan obyekt to'liq qaytsa yoki optimistik yangilash kerak bo'lsa, `setQueryData` ishlatiladi.
