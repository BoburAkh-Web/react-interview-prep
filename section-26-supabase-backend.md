# Section 26: Supabase Crash Course: Building a Back-End!

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: The Wild Oasis (2-qism: ma'lumotlar bazasi, jadvallar, RLS, storage).

---

## 1. [Junior] Supabase nima? Frontend dasturchi uchun nega qulay?

Supabase **Backend-as-a-Service (BaaS)**: tayyor **PostgreSQL** ma'lumotlar bazasi va uning atrofida avtomatik yaratiladigan xizmatlar:

- Jadvallar asosida avtomatik **REST API** (va JavaScript client kutubxonasi `@supabase/supabase-js`)
- **Authentication** (email/parol, OAuth)
- **Storage** (rasm va fayllar)
- Realtime obunalar, Edge Functions

Frontend dasturchi backend kodi yozmasdan, haqiqiy ma'lumotlar bazasi bilan to'liq ilova qura oladi. Firebase'ning ochiq kodli, SQL asosidagi alternativasi sifatida tanilgan.

## 2. [Junior] Application state'ni qanday modellashtirdik? Qaysi jadvallar bor?

Server state'ni domenlar bo'yicha **jadvallar**ga ajratdik:

| Jadval | Nimani saqlaydi |
|---|---|
| `cabins` | Kabina: nomi, sig'imi, narxi, chegirma, tavsif, rasm |
| `guests` | Mehmon: ism, email, millati, pasport ID |
| `bookings` | Bron: sanalar, kechalar soni, narxlar, holat, kabina va mehmon |
| `settings` | Ilova sozlamalari (minimal/maksimal kechalar, nonushta narxi): **bitta qator** |

Auth foydalanuvchilari Supabase'ning o'z `auth.users` jadvalida saqlanadi.

## 3. [Strong junior] Jadvallar orasidagi bog'lanish (relationship) qanday qilinadi? Foreign key nima?

`bookings` jadvali **qaysi kabina va qaysi mehmon** ekanini bilishi kerak. Butun ma'lumotni nusxalamasdan, faqat ularning **id**sini saqlaydi:

```
bookings.cabinId  → cabins.id
bookings.guestId  → guests.id
```

Bu **foreign key**: boshqa jadvaldagi qatorni ko'rsatadigan ustun. Bog'lanish turi: **one-to-many** (bitta kabinada ko'p bron, bitta mehmonda ko'p bron).

Foydasi: ma'lumot **takrorlanmaydi** (normalizatsiya), kabina nomi o'zgarsa faqat bir joyda o'zgaradi, foreign key esa mavjud bo'lmagan kabinaga bron yaratishni oldini oladi.

So'rovda bog'langan ma'lumotni birga olish:

```js
const { data } = await supabase
  .from("bookings")
  .select("*, cabins(name), guests(fullName, email)");
```

## 4. [Strong junior] Row Level Security (RLS) nima va nega muhim?

Supabase'da API kaliti **brauzerda ko'rinib turadi** (frontend kodiga qo'yiladi). Demak, kim bo'lishidan qat'i nazar har kim API'ga so'rov yubora oladi. **RLS** har bir jadvalda **qaysi foydalanuvchi qaysi qatorlarni o'qishi, yozishi, o'chirishi mumkin**ligini belgilaydigan qoidalar (policy).

- RLS **yoqilgan** va policy **yo'q** bo'lsa: hech qanday ma'lumot qaytmaydi
- `SELECT` uchun `using (true)`: hamma o'qiy oladi (o'quv loyihasi boshida)
- `to authenticated`: faqat login qilganlar

```sql
create policy "Enable read access for all users"
on public.cabins for select
to public
using (true);
```

Real loyihada `bookings` va `guests` (shaxsiy ma'lumot) uchun **hamma o'qiy olishi** xavfli, ularni `authenticated` bilan cheklash kerak. Kursda auth qo'shilgach shu qilinadi.

## 5. [Strong junior] Supabase API kalitlari: publishable/anon va secret/service_role farqi nima?

- **Publishable** (eski nomi **anon**) kalit: brauzerda ishlatish uchun. Xavfsizlikni **RLS policy'lar** ta'minlaydi
- **Secret** (eski nomi **service_role**) kalit: **RLS'ni chetlab o'tadi**, butun bazaga to'liq kirish beradi. Faqat serverda (backend, edge function) ishlatiladi. Frontend kodiga, `.env` ga (Vite'da `VITE_` bilan) yoki GitHub'ga **hech qachon** qo'yilmaydi

Intervyuda: "Publishable kalit sir emas, xavfsizlik RLS'da" degan fikrni aytish muhim.

## 6. [Junior] Supabase'ni React ilovasiga qanday ulaymiz?

```js
// services/supabase.js
import { createClient } from "@supabase/supabase-js";

export const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseKey = import.meta.env.VITE_SUPABASE_KEY;
const supabase = createClient(supabaseUrl, supabaseKey);
export default supabase;
```

```js
// services/apiCabins.js
export async function getCabins() {
  const { data, error } = await supabase.from("cabins").select("*");
  if (error) {
    console.error(error);
    throw new Error("Cabins could not be loaded");
  }
  return data;
}
```

- Client **bitta marta** yaratiladi va ilova bo'ylab qayta ishlatiladi
- Vite'da environment o'zgaruvchilari **`VITE_`** bilan boshlanishi shart, `import.meta.env` orqali o'qiladi (`process.env` emas)
- Supabase xato bo'lsa **reject qilmaydi**, `{ data, error }` qaytaradi, shuning uchun `error` ni o'zimiz tekshirib, `throw` qilamiz (React Query xatoni shu orqali ushlaydi)
- API funksiyalari `services/` da, component'lar Supabase'ni bevosita bilmaydi

## 7. [Junior] Storage bucket nima? Rasmlar qayerda saqlanadi?

**Storage bucket**: fayllar (rasm, PDF) saqlanadigan "papka". Wild Oasis'da `cabin-images` (kabina rasmlari) va `avatars` (foydalanuvchi rasmlari) bucket'lari bor. Bucket **public** bo'lsa, fayl to'g'ridan-to'g'ri URL orqali ochiladi.

Asosiy g'oya: rasmning o'zi **ma'lumotlar bazasida emas**, storage'da saqlanadi, jadvalda esa faqat uning **URL'i** turadi. Fayl yuklash ham RLS policy'lari bilan cheklanadi.

## 8. [Middle] Faqat Supabase'ga tayanib, backend'siz ishlashning chegaralari qanday?

- **Biznes mantiq**: murakkab qoidalar (narxni server tomonda hisoblash, bir nechta jadvalni atomik o'zgartirish) uchun **Postgres funksiyalari (RPC)** yoki **Edge Functions** kerak bo'ladi
- **Xavfsizlik** to'liq RLS'ga bog'liq: policy noto'g'ri yozilsa, ma'lumot ochilib qoladi
- **Vendor lock-in**: Supabase'ga xos API va xizmatlar
- **Validatsiya** faqat frontend'da bo'lsa yetarli emas, bazada ham cheklovlar (`not null`, `check`, foreign key) bo'lishi kerak

Kichik va o'rta loyihalar, MVP va ichki ilovalar uchun juda samarali. Murakkab domenlarda odatda alohida backend qo'shiladi.

## 9. [Middle] Narx va boshqa qiymatlarni frontend'da hisoblash xavfsizmi?

Yo'q. Frontend kodini foydalanuvchi o'zgartira oladi: masalan, bron yaratishda `totalPrice` ni o'zi yuborishi mumkin. Shuning uchun muhim hisob-kitoblar (narx, chegirma, mavjudlik) **serverda** tekshirilishi yoki hisoblanishi kerak (Postgres trigger, funksiya yoki backend). Wild Oasis ichki ilova (faqat xodimlar uchun) bo'lgani uchun bu soddalashtirilgan, lekin intervyuda bu nuqtani bilish yaxshi taassurot qoldiradi.
