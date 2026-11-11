# Section 25: Setting Up Our Biggest Project + Styled Components

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: The Wild Oasis (mehmonxona xodimlari uchun ichki boshqaruv paneli): 1-qism, ya'ni rejalash, styled-components, layout va navigatsiya.

> Wild Oasis loyihasining haqiqiy kodi bo'yicha savollar loyiha tugagach `projects/wild-oasis-interview.md` faylida bo'ladi. Bu fayl **nazariya** uchun.

---

## 1. [Junior] Wild Oasis qanday ilova? Uni qanday rejalashtirdik?

Wild Oasis bu mehmonxona **xodimlari** uchun ichki dashboard: kabinalar (cabins), bronlar (bookings), mehmonlarning kelishi/ketishi (check-in/check-out), sozlamalar, foydalanuvchilar va statistika.

Rejalash tartibi (Fast React Pizza bilan bir xil):

1. **Talablar** → funksiyalar ro'yxati
2. **Feature kategoriyalari**: Bookings, Cabins, Guests, Dashboard, Check in/out, Settings, Authentication
3. **Sahifalar**: `/dashboard`, `/bookings`, `/bookings/:id`, `/checkin/:id`, `/cabins`, `/users`, `/settings`, `/account`, `/login`
4. **State'ni aniqlash**: deyarli hammasi **server (remote) state** → React Query + Supabase
5. **Texnologiyalar**: React Router, styled-components, React Query, Supabase, React Hook Form, Recharts va h.k.

Bu ilova "client state"dan ko'ra **server state**ga asoslangan, shuning uchun Redux emas, React Query tanlangan.

## 2. [Junior] Styled-components nima?

CSS-in-JS kutubxonasi: stilni JavaScript ichida yozib, **stillangan React component** yaratadi.

```jsx
import styled from "styled-components";

const H1 = styled.h1`
  font-size: 3rem;
  font-weight: 600;
`;

const Button = styled.button`
  padding: 1.2rem 1.6rem;
  background-color: var(--color-brand-600);
  &:hover { background-color: var(--color-brand-700); }
`;

<H1>The Wild Oasis</H1>
```

`styled.h1` dagi backtick'lar **tagged template literal**. Natija oddiy React component, unga props berish mumkin. Class nomlari **avtomatik noyob** yaratiladi, shuning uchun to'qnashuv bo'lmaydi (CSS Modules kabi).

## 3. [Junior] Styled-components'ning afzallik va kamchiliklari qanday?

**Afzallik:**

- Stil component bilan **bir faylda** va unga bog'langan (scoped)
- **Props asosida dinamik stil** juda qulay
- Haqiqiy CSS sintaksisi: `&:hover`, media query, ichma-ich selektor
- Ishlatilmagan component'ning stili ham yo'qoladi

**Kamchilik:**

- **Runtime xarajati**: CSS brauzerda JavaScript orqali yaratiladi
- React Server Components bilan (Next.js App Router) muammoli
- JSX'da qaysi element "oddiy", qaysi biri "stillangan" ekanini farqlash qiyinlashadi
- Bugungi kunda yangi loyihalarda Tailwind va CSS Modules ko'proq tanlanmoqda

## 4. [Junior] Global stillar qanday yoziladi?

```jsx
import { createGlobalStyle } from "styled-components";

const GlobalStyles = createGlobalStyle`
  :root {
    --color-brand-600: #4f46e5;
    --color-grey-0: #fff;
    --border-radius-md: 7px;
  }
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  html { font-size: 62.5%; }
`;

// App.jsx
<>
  <GlobalStyles />
  <RouterProvider ... />
</>
```

`createGlobalStyle` bilan yaratilgan component **children qabul qilmaydi**, u faqat render qilinadi (sibling sifatida). Ranglar va o'lchamlar **CSS variables** sifatida e'lon qilinadi, keyinchalik dark mode uchun faqat shu o'zgaruvchilar almashtiriladi.

## 5. [Strong junior] Props orqali dinamik stil va `css` funksiyasi qanday ishlaydi?

```jsx
import styled, { css } from "styled-components";

const Heading = styled.h1`
  ${(props) => props.as === "h1" && css`
    font-size: 3rem;
    font-weight: 600;
  `}
  ${(props) => props.as === "h2" && css`
    font-size: 2rem;
  `}
  line-height: 1.4;
`;

<Heading as="h1">Title</Heading>
<Heading as="h2">Subtitle</Heading>
```

- Template ichida **funksiya** yozilsa, styled-components uni props bilan chaqiradi
- **`css`** helper ichki CSS blokini to'g'ri qayta ishlash va sintaksis ajratish uchun kerak
- **`as` prop**: stillar saqlangan holda render qilinadigan **HTML tegini** o'zgartiradi. Semantika to'g'ri bo'ladi (`h1` va `h2`), lekin stil bitta component'da

## 6. [Strong junior] Variant va o'lchamli qayta ishlatiladigan Button qanday yoziladi?

```jsx
const sizes = {
  small:  css`font-size: 1.2rem; padding: 0.4rem 0.8rem;`,
  medium: css`font-size: 1.4rem; padding: 1.2rem 1.6rem;`,
};
const variations = {
  primary:   css`color: var(--color-brand-50); background-color: var(--color-brand-600);`,
  danger:    css`color: var(--color-red-100); background-color: var(--color-red-700);`,
};

const Button = styled.button`
  border: none;
  border-radius: var(--border-radius-sm);
  ${(props) => sizes[props.size]}
  ${(props) => variations[props.variation]}
`;

Button.defaultProps = { variation: "primary", size: "medium" };
```

Variantlar **obyektda** saqlanadi, props orqali tanlanadi. Eslatma: `defaultProps` function component'larda **eskirgan** (React 19 da olib tashlangan), uning o'rniga destructuring'da default qiymat (`({ variation = "primary" })`) yoki styled-components'ning `.attrs()` ishlatiladi.

Muhim tafovut: oddiy React prop'lar DOM'ga uzatilib ketmasligi uchun styled-components v6 da **transient props** (`$variation`) tavsiya etiladi.

## 7. [Junior] Wild Oasis'ning layout'i qanday tuzilgan?

```jsx
const StyledAppLayout = styled.div`
  display: grid;
  grid-template-columns: 26rem 1fr;
  grid-template-rows: auto 1fr;
  height: 100vh;
`;

function AppLayout() {
  return (
    <StyledAppLayout>
      <Header />
      <Sidebar />
      <Main><Container><Outlet /></Container></Main>
    </StyledAppLayout>
  );
}
```

CSS Grid: chapda **Sidebar**, tepada **Header**, qolgan joyda sahifa (`<Outlet />`). `AppLayout` route'larda **layout route** sifatida (`path` siz) ishlatiladi, `/` esa `index` route orqali `<Navigate replace to="dashboard" />` ga yo'naltiriladi.

## 8. [Strong junior] Styled-components bilan `NavLink` qanday stillanadi?

Uchinchi tomon component'ini o'rash uchun `styled(Component)` yoziladi:

```jsx
const StyledNavLink = styled(NavLink)`
  display: flex;
  align-items: center;
  gap: 1.2rem;

  &:hover,
  &:active,
  &.active:link,
  &.active:visited {
    color: var(--color-grey-800);
    background-color: var(--color-grey-50);
  }
`;
```

`NavLink` aktiv bo'lganda qo'shadigan **`active` class**i `&.active` bilan ushlanadi. `styled(Component)` ishlashi uchun o'ralgan component **`className` prop'ini** qabul qilib, uni o'z elementiga berishi kerak (NavLink buni qiladi).

## 9. [Middle] Styled-components, CSS Modules va Tailwind'dan qaysi birini tanlaysiz?

| Mezon | Styled-components | CSS Modules | Tailwind |
|---|---|---|---|
| Dinamik stil (props bo'yicha) | Juda qulay | Class almashtirish | Class almashtirish |
| Runtime xarajat | Bor | Yo'q | Yo'q |
| Server Components | Muammoli | Mos | Mos |
| Dizayn tizimi | Theme yoki CSS variables | O'zingiz yozasiz | Ichida tayyor |
| Kod tuzilishi | Stil JS ichida | Alohida CSS fayl | JSX ichidagi class'lar |

Javob namunasi: "Mavjud loyihada jamoa ishlatayotgan yondashuvni davom ettiraman. Yangi loyihada, ayniqsa Next.js bilan, runtime xarajati yo'q Tailwind yoki CSS Modules'ni tanlardim. Styled-components props asosidagi dinamik stil ko'p bo'lgan SPA'da qulay."

## 10. [Middle] Katta loyihada dizayn tizimini qanday tashkil qilasiz?

- **Design tokens** (ranglar, shrift o'lchamlari, bo'shliqlar, radius, soya) bitta joyda: CSS variables yoki theme obyekti
- **UI primitivlari** (`Button`, `Input`, `Form`, `Row`, `Heading`, `Table`, `Modal`) `ui/` papkasida, ilova bo'ylab qayta ishlatiladi
- Variantlar (`primary`, `danger`) va o'lchamlar (`small`, `medium`) props orqali
- **Dark mode** tokenlarni almashtirish orqali (component'larni qayta yozish shart emas)

Wild Oasis aynan shu tuzilmada qurilgan: `ui/` dagi umumiy component'lar + `features/` dagi domen component'lari.
