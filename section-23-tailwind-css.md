# Section 23: [Optional] Tailwind CSS Crash Course: Styling the App

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: Fast React Pizza Co. (2-qism: stillash).

> Kurs **Tailwind v3** bilan yozilgan (`tailwind.config.js`, PostCSS). Hozirgi **v4** da sozlash boshqacha (14-savol). Intervyuda versiyani aytib o'tish foydali.

---

## 1. [Junior] Tailwind CSS nima? "Utility-first" degani nima?

Tailwind bu **utility-first** CSS framework: oldindan tayyorlangan, har biri **bitta vazifa** bajaradigan kichik class'lar (`p-4`, `text-center`, `bg-yellow-400`, `flex`) to'plami. Stil alohida CSS faylida emas, to'g'ridan-to'g'ri JSX'dagi `className` ichida yoziladi:

```jsx
<button className="rounded-full bg-yellow-400 px-4 py-3 font-semibold uppercase text-stone-800">
  Order now
</button>
```

Bootstrap'dan farqi: Bootstrap **tayyor component'lar** beradi (`btn btn-primary`), Tailwind esa **qurilish bloklari**ni beradi va dizayn to'liq sizning qo'lingizda.

## 2. [Junior] Tailwind'ning afzallik va kamchiliklari qanday?

**Afzalliklar:**

- Class nomini o'ylab topish shart emas
- Fayllar orasida sakrash yo'q: markup va stil bir joyda
- **Dizayn tizimi** tayyor: ranglar, o'lchamlar, shriftlar izchil
- Ishlatilmagan class'lar build'ga tushmaydi, natijada CSS fayli kichik
- Responsive va holatlar (hover, focus) juda qisqa yoziladi

**Kamchiliklar:**

- Markup "iflos" va uzun ko'rinadi
- Class nomlarini o'rganish kerak
- Takrorlanadigan stillarni qayta ishlatish uchun yechim kerak (`@apply` yoki component)
- CSS'ni chuqur bilmasdan foydalanish xavfi bor: Tailwind CSS o'rnini bosmaydi, uning ustiga qurilgan

## 3. [Junior] Tailwind'da ranglar va matn qanday stillanadi?

Ranglar `xususiyat-rang-to'qlik` shaklida: `bg-stone-100`, `text-yellow-500`, `border-red-300`. To'qlik 50 dan 950 gacha.

Matn: o'lcham `text-sm`, `text-xl`, `text-3xl`; qalinlik `font-semibold`, `font-bold`; registr `uppercase`; harf oralig'i `tracking-widest`; satr balandligi `leading-relaxed`; tekislash `text-center`.

## 4. [Junior] Box model: spacing, border va display qanday yoziladi?

- **Padding/margin**: `p-4`, `px-6` (gorizontal), `py-2` (vertikal), `mt-8`, `mx-auto`
- Tailwind shkalasi: `1` = `0.25rem` (odatda 4px), ya'ni `p-4` = `1rem`
- Elementlar oralig'i: `space-y-4` (child'lar orasiga vertikal masofa), `gap-4` (flex/grid'da)
- **Border**: `border`, `border-b`, `border-stone-200`, `rounded-full`, `rounded-md`
- **Display**: `block`, `inline-block`, `hidden`, `flex`, `grid`
- `divide-y` ro'yxat elementlari orasiga chiziq qo'yadi

## 5. [Strong junior] Responsive dizayn qanday yoziladi? "Mobile-first" nima?

Tailwind **mobile-first**: prefikssiz class'lar **barcha ekranlarda** (avvalo kichik ekranda) ishlaydi, prefiksli class'lar esa **shu kenglikdan boshlab** ustidan yoziladi.

```jsx
<h1 className="text-xl sm:text-3xl md:text-5xl">Fast React Pizza</h1>
<input className="w-28 sm:w-64 sm:focus:w-72" />
```

Standart breakpoint'lar: `sm` (640px), `md` (768px), `lg` (1024px), `xl` (1280px), `2xl` (1536px). Ular `min-width` media query, ya'ni `sm:` "640px va undan katta" degani. Tuzoq: `sm:` ni "kichik ekran uchun" deb o'ylash noto'g'ri.

## 6. [Junior] Flexbox va Grid Tailwind'da qanday yoziladi?

```jsx
<header className="flex items-center justify-between px-4 py-3">...</header>

<div className="grid h-screen grid-rows-[auto_1fr_auto]">
  <Header />
  <main className="overflow-scroll">...</main>
  <CartOverview />
</div>
```

- Flex: `flex`, `flex-col`, `items-center`, `justify-between`, `gap-4`, `grow`
- Grid: `grid`, `grid-cols-3`, `grid-rows-[auto_1fr_auto]`, `col-span-2`
- Kvadrat qavs ichida **ixtiyoriy qiymat** (arbitrary value): `grid-rows-[auto_1fr_auto]`, `w-[73px]`. Lekin ularni ko'p ishlatish dizayn tizimining ma'nosini yo'qotadi

## 7. [Strong junior] Holatlar (hover, focus, disabled) va animatsiyalar qanday qo'shiladi?

Holat prefikslari bilan:

```jsx
<button className="bg-yellow-400 transition-colors duration-300
                   hover:bg-yellow-300
                   focus:outline-none focus:ring focus:ring-yellow-300 focus:ring-offset-2
                   disabled:cursor-not-allowed">
```

- `hover:`, `focus:`, `active:`, `disabled:`, `placeholder:`
- Prefikslarni birlashtirish mumkin: `sm:focus:w-72`
- `transition-all duration-300` silliq o'tish beradi
- Accessibility: `focus:outline-none` qilsangiz, o'rniga albatta `focus:ring` kabi ko'rinadigan focus belgisini qoldiring (klaviatura foydalanuvchilari uchun)

## 8. [Strong junior] Takrorlanadigan stillarni qanday qayta ishlatamiz? `@apply` va component farqi?

**1. `@apply`**: CSS faylida bir nechta utility'ni bitta class'ga yig'ish:

```css
@layer components {
  .input {
    @apply rounded-full border border-stone-200 px-4 py-2 text-sm
           focus:outline-none focus:ring focus:ring-yellow-400 md:px-6 md:py-3;
  }
}
```

**2. React component** (tavsiya etiladi):

```jsx
function Button({ children, disabled, to, type = "primary" }) {
  const base = "inline-block rounded-full bg-yellow-400 font-semibold uppercase ...";
  const styles = {
    primary: base + " px-4 py-3 md:px-6 md:py-4",
    small:   base + " px-4 py-2 text-xs md:px-5 md:py-2.5",
    secondary: "inline-block rounded-full border-2 border-stone-300 ...",
  };

  if (to) return <Link to={to} className={styles[type]}>{children}</Link>;
  return <button disabled={disabled} className={styles[type]}>{children}</button>;
}
```

`@apply` ni ko'p ishlatish Tailwind'ni oddiy CSS'ga qaytaradi (yana class nomlari o'ylab topiladi). React'da qayta ishlatishning asosiy yo'li: **component**. `@apply` faqat component qilib bo'lmaydigan joylarda (masalan, oddiy `<input>` uchun) qulay.

## 9. [Strong junior] Tailwind'da class nomini dinamik yasash mumkinmi? (`bg-${color}-500`)

**Yo'q.** Tailwind build paytida fayllarni **matn sifatida** skanerlab, unda **to'liq yozilgan** class nomlarini topadi va faqat ularning CSS'ini yaratadi. Dinamik yasalgan nom (`` `bg-${color}-500` ``) kodda butun holida yo'q, shuning uchun uning CSS'i yaratilmaydi va stil ishlamaydi.

```jsx
// Ishlamaydi
<div className={`bg-${color}-500`} />

// Ishlaydi: to'liq class nomlari kodda bor
const colors = { red: "bg-red-500", green: "bg-green-500" };
<div className={colors[color]} />
```

Bu intervyuda tez-tez beriladigan amaliy savol.

## 10. [Junior] Absolute positioning, z-index va boshqa joylashuv class'lari

`relative`, `absolute`, `fixed`, `sticky`; joylashuv `top-0`, `left-1/2`, `inset-0`; qatlam `z-10`, `z-50`; `-translate-x-1/2` markazlash uchun. Masalan, loader'ni butun ekran ustiga qo'yish: `absolute inset-0 flex items-center justify-center bg-slate-200/20 backdrop-blur-sm`. `bg-slate-200/20` dagi `/20` shaffoflikni bildiradi.

## 11. [Strong junior] Tailwind'ni qanday sozlaymiz? (Shrift, ranglar)

Tailwind v3 da `tailwind.config.js` orqali:

```js
export default {
  content: ["./index.html", "./src/**/*.{js,jsx}"],   // qaysi fayllar skanerlanadi
  theme: {
    fontFamily: { sans: "Roboto Mono, monospace" },     // standartni ALMASHTIRADI
    extend: {
      height: { screen: "100dvh" },                     // standartga QO'SHADI/ustidan yozadi
    },
  },
};
```

- `theme` ichida to'g'ridan-to'g'ri yozilgan qiymat Tailwind'ning standart qiymatlarini **butunlay almashtiradi**
- `theme.extend` ichidagisi esa standartlarni **saqlab**, yangisini qo'shadi
- `content` noto'g'ri bo'lsa, class'lar build'ga tushmaydi (ko'p uchraydigan xato)
- Shriftni o'zi `index.html` da Google Fonts orqali ulanadi

## 12. [Junior] Class'larni tartibli saqlash uchun qanday vositalar bor?

- **Prettier plugin** (`prettier-plugin-tailwindcss`): class'larni rasmiy tavsiya etilgan tartibda avtomatik saralaydi
- **Tailwind CSS IntelliSense** (VS Code kengaytmasi): avtomatik to'ldirish, rang ko'rsatish, xato topish
- Shartli class'lar uchun **`clsx`** yoki to'qnashuvlarni hal qiluvchi **`tailwind-merge`**

## 13. [Middle] Tailwind'ni qachon tanlaymiz, qachon CSS Modules yoki styled-components?

| | Tailwind | CSS Modules | styled-components |
|---|---|---|---|
| Stil qayerda | JSX `className` ichida | Alohida `.module.css` | JS ichida (CSS-in-JS) |
| Tezlik (yozish) | Juda tez | O'rtacha | O'rtacha |
| Runtime xarajat | Yo'q (faqat CSS) | Yo'q | Bor (runtime'da CSS yaratadi) |
| Dinamik stil | Class'larni almashtirish | Class'larni almashtirish | Props asosida juda qulay |
| Dizayn tizimi | Ichida tayyor | O'zingiz yozasiz | Theme provider orqali |

Tanlov jamoa kelishuviga bog'liq. Hozir ko'p yangi loyihalarda Tailwind (shadcn/ui bilan birga) ommabop. styled-components esa runtime xarajati va React Server Components bilan muammolari sababli kamroq tanlanmoqda (kursda Wild Oasis loyihasida ishlatiladi).

## 14. [Middle] Tailwind v3 va v4 farqi nima?

- **v4** da sozlash asosan **CSS ichida**: `@import "tailwindcss";` va `@theme { --font-sans: ...; --color-brand: ...; }`. `tailwind.config.js` majburiy emas
- Vite uchun alohida plagin: `@tailwindcss/vite` (PostCSS sozlash kerak emas)
- `content` ni qo'lda yozish shart emas, fayllar avtomatik aniqlanadi
- Tezroq build dvigateli, zamonaviy CSS imkoniyatlari (cascade layers, CSS o'zgaruvchilari)

Utility class'larning asosiy mantiqi o'zgarmagan, shuning uchun kursdagi bilim to'liq foydali.
