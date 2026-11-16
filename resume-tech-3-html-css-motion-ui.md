# Rezyume texnologiyalari, 3-qism: HTML5, CSS3, SASS, Framer Motion va UI kutubxonalari

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

> Har bir savol tuzilmasi: **Oddiy qilib** → **Kod** → **Intervyuda shunday ayting** → **Qo'shimcha savol**.
>
> Tailwind CSS bo'yicha savollar `react-course/section-23` faylida, styled-components esa `section-25` da. Bu yerda ular faqat boshqa usullar bilan solishtirishda tilga olinadi.
>
> **Eslatma:** ko'p React dasturchilari intervyuda aynan HTML va CSS savollarida yiqiladi, chunki ularni "oddiy" deb o'ylab tayyorlanmaydi. Bu qismni jiddiy o'qing.

---

## Lug'at: HTML va CSS kalit so'zlari

| So'z | Ma'nosi |
|---|---|
| **Semantik HTML** | Mazmunni bildiradigan teglar (`nav`, `main`, `article`), shunchaki `div` emas |
| **Accessibility (a11y)** | Saytni ko'rish, eshitish yoki harakat cheklovi bor odamlar ham ishlata olishi |
| **Screen reader** | Ekrandagi matnni ovoz bilan o'qib beruvchi dastur (ko'zi ojizlar uchun) |
| **ARIA** | Elementga qo'shimcha ma'no beruvchi atributlar (`aria-label`, `aria-expanded`) |
| **Cascade** | Bir elementga bir nechta qoida tushganda qaysi biri g'olib bo'lishini belgilovchi tartib |
| **Specificity** | Selektorning "kuchi": qaysi qoida ustun kelishini aniqlaydi |
| **Box model** | Har bir element: content + padding + border + margin |
| **Stacking context** | `z-index` ishlaydigan alohida "qatlamlar guruhi" |
| **Main / cross axis** | Flexbox'da asosiy yo'nalish va unga perpendikulyar yo'nalish |
| **Breakpoint** | Layout o'zgaradigan ekran kengligi (masalan, 768px) |
| **Media query** | Ekran o'lchami yoki xususiyatiga qarab stil berish (`@media`) |
| **Pseudo-class** | Elementning holati: `:hover`, `:focus`, `:first-child` |
| **Pseudo-element** | Elementning "virtual" qismi: `::before`, `::after`, `::placeholder` |
| **Reflow / repaint** | Brauzer layout'ni qayta hisoblashi / piksellarni qayta chizishi (qimmat amallar) |
| **Preprocessor** | CSS'ga build paytida aylanadigan kengaytirilgan til (SASS) |
| **Headless UI** | Faqat mantiq va accessibility beradigan, dizaynsiz component'lar |

---

## H. HTML5

### H1. [Junior] HTML5 sahifaning asosiy tuzilmasi qanday? `meta viewport` nima uchun?

```html
<!DOCTYPE html>                  <!-- brauzerga: zamonaviy HTML5 standarti -->
<html lang="uz">                 <!-- til: screen reader va qidiruv uchun -->
  <head>
    <meta charset="UTF-8" />     <!-- o'zbek, rus harflari to'g'ri ko'rinishi uchun -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Bogizilol</title>
    <meta name="description" content="Toshkentdagi milliy taomlar restorani" />
  </head>
  <body>
    <div id="root"></div>        <!-- React shu yerga render qiladi -->
  </body>
</html>
```

**Oddiy qilib:** `meta viewport` bo'lmasa, telefon sahifani **kompyuter ekrani** kengligida (taxminan 980px) chizib, keyin kichraytirib ko'rsatadi: matnlar mayda bo'lib qoladi va media query'lar ishlamaydi. `width=device-width` "telefonning haqiqiy kengligini ishlat" degani.

**Intervyuda shunday ayting:** "`meta viewport` responsive dizayn uchun majburiy, usiz mobil brauzer sahifani kichraytirib ko'rsatadi. `lang` atributi accessibility va SEO uchun, `charset` esa harflar to'g'ri ko'rinishi uchun."

---

### H2. [Junior] Semantik HTML nima va nega muhim?

**Oddiy qilib:** Semantik teg brauzerga va screen reader'ga "bu nima" ekanini aytadi. `<div>` hech narsa demaydi, `<nav>` esa "bu navigatsiya" deydi. Ko'zi ojiz foydalanuvchi screen reader orqali "asosiy kontentga o'tish" yoki "navigatsiyalar ro'yxati" kabi buyruqlarni aynan shu teglar orqali ishlatadi.

```html
<!-- Yomon: "div soup" -->
<div class="header"><div class="nav">...</div></div>
<div class="content"><div class="post">...</div></div>

<!-- Yaxshi -->
<header>
  <nav>...</nav>
</header>
<main>
  <article>
    <h1>Sarlavha</h1>
    <section>...</section>
  </article>
  <aside>Qo'shimcha ma'lumot</aside>
</main>
<footer>...</footer>
```

| Teg | Vazifasi |
|---|---|
| `header` / `footer` | Sahifa yoki bo'lim boshi / oxiri |
| `nav` | Asosiy navigatsiya havolalari |
| `main` | Sahifaning asosiy kontenti (sahifada **bitta**) |
| `article` | Mustaqil kontent (post, kartochka, mahsulot) |
| `section` | Mavzu bo'yicha bo'lim (odatda sarlavhasi bilan) |
| `aside` | Asosiy kontentga yondosh ma'lumot (sidebar) |

**Intervyuda shunday ayting:** "Semantik teglar accessibility va SEO uchun muhim: screen reader sahifa tuzilmasini tushunadi, qidiruv tizimi esa asosiy kontentni ajratadi. Sarlavhalarni ham tartib bilan ishlataman: bitta `h1`, keyin `h2`, `h3`, darajalarni o'tkazib yubormasdan."

---

### H3. [Junior] Block va inline elementlar farqi nima?

**Oddiy qilib:** **Block** element yangi qatordan boshlanadi va butun kenglikni egallaydi (`div`, `p`, `h1`, `section`). **Inline** element matn ichida qoladi va faqat kerakli kenglikni oladi (`span`, `a`, `strong`). Inline elementga `width`, `height` va vertikal `margin` ta'sir qilmaydi. **Inline-block** ikkalasining aralashmasi: qator ichida turadi, lekin o'lcham beriladi.

```css
span { width: 200px; }          /* ishlamaydi: inline */
span { display: inline-block; width: 200px; }   /* ishlaydi */
```

---

### H4. [Junior] Nega bosiladigan narsa uchun `div onClick` emas, `button` kerak?

**Oddiy qilib:** `button` "tayyor tugma": klaviaturadagi **Tab** bilan unga o'tiladi, **Enter** va **Space** bilan bosiladi, screen reader uni "tugma" deb e'lon qiladi. `div` da bularning hech biri yo'q, hammasini qo'lda yozish kerak bo'ladi.

```jsx
// Yomon: klaviatura bilan ishlamaydi
<div onClick={handleDelete}>O'chirish</div>

// Agar baribir div kerak bo'lsa, shuncha narsa qo'shish kerak:
<div role="button" tabIndex={0} onClick={handleDelete}
     onKeyDown={(e) => (e.key === "Enter" || e.key === " ") && handleDelete()}>O'chirish</div>

// Yaxshi
<button type="button" onClick={handleDelete}>O'chirish</button>
```

Qoida: **amal bajaradigan** narsa `button`, **boshqa sahifaga olib boradigan** narsa `a` (React'da `Link`).

**Qo'shimcha savol:** "Forma ichidagi tugma nega formani yuborib yubordi?" → `button` ning default turi `type="submit"`. Yubormasligi kerak bo'lsa, `type="button"` yoziladi.

---

### H5. [Junior] Formalar: input turlari, `label` va brauzer validatsiyasi

```html
<form>
  <label for="email">Email</label>
  <input id="email" type="email" name="email" required autocomplete="email" />

  <label for="phone">Telefon</label>
  <input id="phone" type="tel" name="phone" pattern="\+998\d{9}" inputmode="tel" />

  <label for="age">Yosh</label>
  <input id="age" type="number" min="7" max="14" />

  <label>
    <input type="checkbox" name="priority" /> Tezkor yetkazish
  </label>

  <button type="submit">Yuborish</button>
</form>
```

**Oddiy qilib:**

- `label` input bilan **`for` va `id`** (React'da `htmlFor`) orqali bog'lanadi: label bosilganda input fokus oladi, screen reader maydon nomini o'qiydi
- `type="email"`, `"tel"`, `"number"` telefonda **mos klaviaturani** ochadi
- `required`, `min`, `max`, `minlength`, `pattern` bilan brauzer o'zi tekshiradi
- `autocomplete` brauzerga avto-to'ldirishda yordam beradi

**Intervyuda shunday ayting:** "Har input'ni label bilan bog'layman, mos `type` beraman, shunda telefonda to'g'ri klaviatura ochiladi. Brauzer validatsiyasi birinchi qatlam, keyin Zod bilan JavaScript validatsiyasi, eng muhimi esa backend tekshiruvi."

---

### H6. [Strong junior] Accessibility uchun qanday asosiy qoidalarga amal qilasiz?

**Oddiy qilib:** Saytni sichqonchasiz, ekranni ko'rmasdan va kichik ekranda ham ishlatsa bo'ladimi, degan savol.

| Qoida | Qanday |
|---|---|
| Rasmlarga matn | `alt="Margherita pitsa"`. Bezak rasm bo'lsa `alt=""` (screen reader o'tkazib yuboradi) |
| Input'larga nom | `label` yoki `aria-label` |
| Ikonkali tugmalar | `<button aria-label="Yopish"><XIcon /></button>`: matnsiz tugmaga nom |
| Klaviatura | Hamma narsa **Tab** bilan erishiladi, modal **Esc** bilan yopiladi |
| Focus ko'rinishi | `outline: none` qilinsa, o'rniga boshqa ko'rinadigan focus stili |
| Rang kontrasti | Matn va fon orasida yetarli kontrast (WCAG: oddiy matn uchun 4.5:1) |
| Holatlar | `aria-expanded`, `aria-selected`, `aria-current="page"`: ochiq/yopiq, tanlangan |
| Faqat rang bilan emas | Xatoni faqat qizil rang bilan emas, matn bilan ham ko'rsatish |

Tekshirish: Chrome **Lighthouse**, sahifani faqat klaviatura bilan aylanib chiqish, **axe DevTools** kengaytmasi.

**Intervyuda shunday ayting:** "Asosiy qoidalarim: semantik teglar, har input uchun label, rasmlarga alt, ikonkali tugmalarga aria-label, klaviatura bilan to'liq ishlashi va ko'rinadigan focus. Tekshirish uchun Lighthouse va klaviatura bilan sinab ko'raman. Bolalar platformasida bu ayniqsa muhim."

**Qo'shimcha savol:** "ARIA'ni ko'p ishlatish yaxshimi?" → Yo'q. Birinchi qoida: mos semantik teg bo'lsa, ARIA kerak emas. Noto'g'ri ARIA umuman yo'qligidan yomonroq.

---

### H7. [Strong junior] `<script>` dagi `defer` va `async` farqi nima?

**Oddiy qilib:** Oddiy `<script>` HTML o'qishni **to'xtatadi**: brauzer skriptni yuklab, bajarib bo'lmaguncha sahifa qotib turadi.

```
Oddiy:  HTML ━━━━ ✋ skript yuklash+bajarish ━━━━ HTML davom
async:  HTML ━━━━━━━━━━━━ ✋ bajarish ━━ HTML davom       (yuklash parallel, tayyor bo'lishi bilan bajariladi)
defer:  HTML ━━━━━━━━━━━━━━━━━━━━━━━━━━ tugadi → bajarish   (yuklash parallel, HTML tugagach, tartib bilan)
```

- **`async`**: tartib kafolatlanmaydi. Mustaqil skriptlar uchun (analytics)
- **`defer`**: HTML to'liq o'qilgach, **yozilgan tartibda** bajariladi. Sahifa bilan ishlaydigan skriptlar uchun

Vite build qilgan skriptlar `type="module"`, ular default holda `defer` kabi ishlaydi.

---

### H8. [Strong junior] Rasmlarni qanday optimallaysiz? `srcset`, `picture` va `loading="lazy"` nima?

**Oddiy qilib:** Telefonga 3000px kenglikdagi rasm yuborish bekorga trafik va vaqt. Brauzerga **bir nechta o'lchamdagi** variant beriladi, u ekranga mosini o'zi tanlaydi.

```html
<!-- Ekran kengligiga qarab mos o'lcham -->
<img
  src="pizza-800.webp"
  srcset="pizza-400.webp 400w, pizza-800.webp 800w, pizza-1600.webp 1600w"
  sizes="(max-width: 640px) 100vw, 50vw"
  alt="Margherita pitsa"
  width="800" height="600"
  loading="lazy"
/>

<!-- Ekran shakliga qarab boshqa kesim (art direction) -->
<picture>
  <source media="(max-width: 640px)" srcset="hero-mobile.webp" />
  <img src="hero-desktop.webp" alt="Restoran ichi" fetchpriority="high" />
</picture>
```

- **`width` va `height`**: brauzer rasm uchun joyni oldindan ajratadi, sahifa **sakramaydi** (CLS)
- **`loading="lazy"`**: ekranga yaqinlashgandagina yuklanadi. **Birinchi ekrandagi** asosiy rasmga qo'yilmaydi
- **`fetchpriority="high"`**: eng muhim rasmni birinchi yuklash
- Format: `webp` yoki `avif` (JPEG/PNG'dan ancha yengil)

**Intervyuda shunday ayting:** "Rasmlarni `webp` formatida, bir nechta o'lchamda beraman, `srcset` bilan brauzer mosini tanlaydi. `width`/`height` beraman, shunda layout sakramaydi. Birinchi ekrandan pastdagi rasmlar `loading="lazy"`, asosiy hero rasm esa `fetchpriority="high"`."

---

### H9. [Junior] `data-*` atributlari nima uchun?

**Oddiy qilib:** HTML elementga o'zingizning **qo'shimcha ma'lumotingizni** biriktirish usuli. JavaScript `dataset` orqali o'qiydi, CSS esa selektor sifatida ishlatadi.

```html
<li data-id="42" data-status="pending">Buyurtma #42</li>
```

```js
item.dataset.id;       // "42"
item.dataset.status;   // "pending"
```

```css
li[data-status="pending"] { color: orange; }
```

Event delegation'da qaysi element bosilganini aniqlash uchun qulay. Testlarda element topish uchun ham ishlatiladi (`data-testid`).

---

### H10. [Strong junior] Native `<dialog>` elementi nima beradi?

**Oddiy qilib:** HTML'ning o'ziga o'rnatilgan modal oyna. Ko'p narsani avtomatik qiladi: **Esc** bilan yopiladi, fokus oyna ichida qoladi, orqa fon (`::backdrop`) stillanadi, accessibility tayyor.

```jsx
const dialogRef = useRef(null);

<button onClick={() => dialogRef.current.showModal()}>Ochish</button>

<dialog ref={dialogRef} onClose={() => console.log("yopildi")}>
  <h2>Buyurtmani tasdiqlang</h2>
  <form method="dialog">              {/* bu forma yuborilganda dialog yopiladi */}
    <button>Yopish</button>
  </form>
</dialog>
```

```css
dialog::backdrop { background: rgb(0 0 0 / 0.5); backdrop-filter: blur(4px); }
```

**Intervyuda shunday ayting:** "Oddiy modal uchun native `dialog` ishlatsa bo'ladi: Esc bilan yopish, focus boshqaruvi va accessibility tayyor. Fast React Pizza'dagi pitsa tafsilotlari oynasini shunday qilganman. Murakkab holatlarda portal va compound component yoki Radix kabi kutubxona."

---

## I. CSS3

### I1. [Junior] Cascade va specificity qanday ishlaydi?

**Oddiy qilib:** Bir elementga bir nechta qoida tushsa, brauzer g'olibni shu tartibda aniqlaydi:

1. **`!important`** (eng kuchli, kamdan-kam ishlatiladi)
2. **Specificity**: selektor qanchalik "aniq" bo'lsa, shuncha kuchli
3. **Tartib**: specificity teng bo'lsa, **keyin yozilgani** g'olib

Specificity'ni uch xonali son deb tasavvur qiling: **(ID, class, teg)**.

| Selektor | Hisob | Kuch |
|---|---|---|
| `p` | (0, 0, 1) | Eng past |
| `.btn` | (0, 1, 0) | |
| `.nav .btn` | (0, 2, 0) | |
| `#gross .btn` | (1, 1, 0) | Bitta ID, istalgan class'lardan kuchli |
| `style="..."` (inline) | Eng kuchli (`!important` dan tashqari) | |

Class, atribut (`[type="text"]`) va pseudo-class (`:hover`) bir xil og'irlikda. Teg va pseudo-element (`::before`) eng past.

**Intervyuda shunday ayting:** "Gross'da `#gross .btn` kabi ID'li global selektor yangi stillarimizni bosib ketardi, chunki bitta ID istalgan sondagi class'dan kuchli. Yechim selektorni yanada kuchaytirish emas, balki specificity'ni past va tekis saqlash: asosan class'lar, ID'larni stil uchun ishlatmaslik, CSS Modules yoki Tailwind. `!important` faqat oxirgi chora."

**Qo'shimcha savol:** "`:where()` va `:is()` farqi?" → `:where()` specificity'ni **nolga** tushiradi (kutubxona stillari oson bosib ketilishi uchun), `:is()` esa ichidagi eng kuchli selektorning specificity'sini oladi.

---

### I2. [Junior] Box model nima? `box-sizing: border-box` nima qiladi?

**Oddiy qilib:** Har bir element bu quti: ichida **content**, uning atrofida **padding** (ichki bo'shliq), keyin **border** (chegara), eng tashqarida **margin** (tashqi bo'shliq).

```
┌──────────── margin ─────────────┐
│  ┌───────── border ──────────┐  │
│  │  ┌────── padding ──────┐  │  │
│  │  │      content        │  │  │
│  │  └─────────────────────┘  │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

```css
.box { width: 300px; padding: 20px; border: 2px solid; }
/* content-box (default): haqiqiy kenglik 300 + 40 + 4 = 344px */
/* border-box:            haqiqiy kenglik aynan 300px          */

*, *::before, *::after { box-sizing: border-box; }   /* deyarli har loyihada */
```

**Intervyuda shunday ayting:** "Default `content-box` da `width` faqat content'ga tegishli, padding va border qo'shilib element kattalashadi. `border-box` da `width` hammasini o'z ichiga oladi, shuning uchun o'lchamlarni hisoblash oson. Uni global qilib qo'yaman."

---

### I3. [Strong junior] Margin collapse nima?

**Oddiy qilib:** Ketma-ket ikki block elementning **vertikal** margin'lari qo'shilmaydi, **kattasi** qoladi. Masalan, yuqoridagi `margin-bottom: 30px` va pastdagi `margin-top: 20px` orasida 50px emas, **30px** bo'ladi.

```css
h2 { margin-bottom: 30px; }
p  { margin-top: 20px; }      /* orasida 30px, 50px emas */
```

Flexbox va Grid ichida margin collapse **bo'lmaydi**. Shuning uchun ko'p dasturchilar oraliqlar uchun `gap` ishlatishni afzal ko'radi.

---

### I4. [Junior] `display: none`, `visibility: hidden` va `opacity: 0` farqi?

| | Joy egallaydimi | Bosish mumkinmi | Screen reader o'qiydimi | Animatsiya qilsa bo'ladimi |
|---|---|---|---|---|
| `display: none` | Yo'q | Yo'q | Yo'q | Yo'q |
| `visibility: hidden` | **Ha** | Yo'q | Yo'q | Cheklangan |
| `opacity: 0` | **Ha** | **Ha** (ko'rinmas, lekin bosiladi!) | Ha | Ha |

React'da shartli render (`{isOpen && <Modal />}`) element'ni DOM'dan butunlay olib tashlaydi, bu `display: none` dan ham kuchliroq: component state'i ham yo'qoladi.

---

### I5. [Junior] CSS o'lchov birliklari: `px`, `em`, `rem`, `%`, `vw`/`vh`, `dvh`

**Oddiy qilib:**

| Birlik | Nimaga nisbatan | Qachon |
|---|---|---|
| `px` | Qat'iy | Border, kichik detallar |
| `rem` | **Root** (`html`) shrift o'lchami | Shrift, bo'shliq: eng ko'p tavsiya etiladi |
| `em` | **Ota-element** shrift o'lchami | Tugma ichidagi padding (shriftga mos o'ssin) |
| `%` | Ota-element o'lchami | Kenglik |
| `vw` / `vh` | Ekran kengligi / balandligi | To'liq ekranli bo'limlar |
| `dvh` | **Haqiqiy** ko'rinadigan balandlik (mobil toolbar'ni hisobga oladi) | Mobil to'liq ekran layout |

`rem` ning afzalligi: foydalanuvchi brauzerda shriftni kattalashtirsa, butun sayt **mutanosib** kattalashadi. `px` da bu ishlamaydi.

`em` ning tuzog'i: ichma-ich elementlarda **ko'payib** ketadi (`1.2em` ichida `1.2em` = 1.44 barobar).

**Qo'shimcha savol:** "Mobilda `100vh` nega muammo?" → Brauzer manzil satri ko'rinib-yo'qolganda balandlik o'zgaradi, `100vh` esa eng katta balandlikni oladi va pastki qism toolbar ostida qoladi. Yechim: `100dvh`.

---

### I6. [Junior] `position` qiymatlari farqi nima?

```css
.static   { position: static; }    /* default: oddiy oqimda, top/left ishlamaydi */
.relative { position: relative; top: 10px; }   /* o'z joyidan siljiydi, joyi saqlanadi */
.absolute { position: absolute; top: 0; right: 0; }   /* eng yaqin positioned otaga nisbatan */
.fixed    { position: fixed; bottom: 20px; right: 20px; }   /* ekranga nisbatan, scroll'da qimirlamaydi */
.sticky   { position: sticky; top: 0; }   /* scroll'da top: 0 ga yetganda yopishib qoladi */
```

**Oddiy qilib:** `absolute` element o'zining "ota"sini qidiradi: `position` i `static` **bo'lmagan** eng yaqin ota-element. Topilmasa, butun sahifaga nisbatan joylashadi. Shuning uchun odatiy usul: ota-elementga `position: relative`, child'ga `position: absolute`.

```css
.card { position: relative; }
.badge { position: absolute; top: 8px; right: 8px; }   /* kartochka burchagida */
```

**Qo'shimcha savol:** "`sticky` nega ishlamayapti?" → Ota-elementlardan birida `overflow: hidden` bor, yoki `top` berilmagan, yoki ota-element balandligi yetarli emas.

---

### I7. [Strong junior] `z-index` nega ba'zan ishlamaydi? Stacking context nima?

**Oddiy qilib:** `z-index` faqat **bitta guruh (stacking context) ichida** solishtiriladi. Ba'zi CSS xossalari elementni **yangi guruhga** aylantiradi, shunda uning ichidagi child `z-index: 9999` bo'lsa ham, guruhdan tashqariga chiqa olmaydi: ota-elementning qatlami qayerda bo'lsa, u ham o'sha yerda qoladi.

Yangi stacking context yaratadiganlar: `position` + `z-index`, `transform`, `opacity < 1`, `filter`, `will-change`, `isolation: isolate` va boshqalar.

```html
<div style="position: relative; z-index: 1">        <!-- A guruhi -->
  <div style="position: absolute; z-index: 9999">Modal</div>   <!-- A ichida qamalgan -->
</div>
<div style="position: relative; z-index: 2">Header</div>       <!-- A dan baland: modal uning ostida qoladi -->
```

Shuning uchun modal, dropdown va toast'lar **portal** orqali `document.body` ga chiqariladi: u yerda hech qanday ota-element ularni cheklamaydi.

**Intervyuda shunday ayting:** "`z-index` faqat bitta stacking context ichida ishlaydi. Ota-elementdagi `transform` yoki `opacity` yangi kontekst yaratadi va child'ni qamab qo'yadi. Modal'larni shu sababli portal bilan body'ga render qilaman."

---

### I8. [Junior] Flexbox qanday ishlaydi?

**Oddiy qilib:** Flexbox elementlarni **bir yo'nalishda** (qator yoki ustun) joylashtiradi. Ikki o'q bor: **main axis** (asosiy yo'nalish) va **cross axis** (unga perpendikulyar).

```
flex-direction: row (default)
main axis   →→→→→→→→→   justify-content boshqaradi
cross axis  ↓                align-items boshqaradi
```

```css
.navbar {
  display: flex;
  justify-content: space-between;   /* main axis: logo chapda, menyu o'ngda */
  align-items: center;              /* cross axis: vertikal markazda */
  gap: 1rem;                        /* elementlar orasidagi masofa */
  flex-wrap: wrap;                  /* sig'masa keyingi qatorga o'tsin */
}
```

| Xossa | Qiymatlar va vazifasi |
|---|---|
| `flex-direction` | `row` / `column`: yo'nalish |
| `justify-content` | `flex-start`, `center`, `space-between`, `space-around`, `space-evenly` |
| `align-items` | `stretch` (default), `center`, `flex-start`, `flex-end` |
| `flex-wrap` | `nowrap` (default) / `wrap` |
| `gap` | Elementlar orasidagi bo'shliq |

**Qo'shimcha savol:** "`flex-direction: column` qilsam, `justify-content` nima qiladi?" → Endi main axis **vertikal**, shuning uchun `justify-content` vertikal, `align-items` esa gorizontal tekislaydi.

---

### I9. [Strong junior] `flex: 1` nimani anglatadi? `flex-grow`, `flex-shrink`, `flex-basis` nima?

**Oddiy qilib:** Bu uchta xossa flex element **qancha joy olishi**ni boshqaradi:

- **`flex-basis`**: boshlang'ich o'lcham (o'sish va qisqarishdan oldin)
- **`flex-grow`**: bo'sh joy qolsa, qancha **kengayadi** (nisbat)
- **`flex-shrink`**: joy yetmasa, qancha **qisqaradi** (nisbat)

```css
.sidebar { flex: 0 0 260px; }   /* o'smaydi, qisqarmaydi, doim 260px */
.main    { flex: 1; }           /* = flex: 1 1 0%: qolgan hamma joyni oladi */

.a { flex: 1; }   /* 1 ulush */
.b { flex: 2; }   /* 2 ulush: a dan ikki baravar keng */
```

**Qo'shimcha savol (tuzoq):** "Flex ichidagi uzun matn nega elementni kengaytirib, layout'ni buzyapti?" → Flex element default holda `min-width: auto`, ya'ni kontentidan kichik bo'la olmaydi. Yechim: `min-width: 0` (keyin `text-overflow: ellipsis` ishlaydi).

---

### I10. [Junior] CSS Grid qanday ishlaydi?

**Oddiy qilib:** Grid **ikki o'lchamli**: qatorlar va ustunlarni birga boshqaradi, xuddi jadval kabi. **`fr`** birligi "bo'sh joydan bir ulush" degani.

```css
/* Dashboard layout */
.app {
  display: grid;
  grid-template-columns: 260px 1fr;        /* sidebar 260px, qolgani asosiy qism */
  grid-template-rows: auto 1fr;            /* header o'z balandligida, qolgani kontent */
  height: 100dvh;
}

/* Nomlangan hududlar: layout'ni "rasm" kabi yozish */
.page {
  display: grid;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  grid-template-columns: 240px 1fr;
}
.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }

/* Element bir nechta ustunni egallashi */
.wide { grid-column: span 2; }
```

---

### I11. [Strong junior] Media query'siz responsive kartochkalar to'ri qanday yasaladi? `auto-fit` va `auto-fill` farqi?

**Oddiy qilib:** `repeat(auto-fit, minmax(250px, 1fr))` brauzerga shuni aytadi: "Har bir ustun kamida 250px, qatorga **qancha sig'sa**, shuncha ustun qo'y, qolgan joyni teng bo'l". Natija: media query yozmasdan telefonda 1 ta, planshetda 2–3 ta, kompyuterda 4 ta kartochka.

```css
.menu {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
}
```

Farqi faqat elementlar **kam** bo'lganda ko'rinadi:

- **`auto-fill`**: bo'sh ustunlar uchun joy **saqlab qoladi** (2 ta kartochka chap tomonda, o'ngda bo'sh joy)
- **`auto-fit`**: bo'sh ustunlarni **yig'adi**, mavjud kartochkalar **cho'zilib** butun kenglikni egallaydi

---

### I12. [Junior] Flexbox yoki Grid: qachon qaysi biri?

| Flexbox | Grid |
|---|---|
| **Bir o'lchamli** (qator yoki ustun) | **Ikki o'lchamli** (qator va ustun birga) |
| Kontent o'lchamni belgilaydi | Layout o'lchamni belgilaydi |
| Navbar, tugmalar guruhi, kartochka ichi, markazlash | Sahifa layout'i, kartochkalar to'ri, dashboard, galereya |

Amalda ko'pincha birga: sahifa skeleti Grid'da, uning ichidagi kichik qismlar Flex'da.

**Elementni markazlash** (eng ko'p beriladigan savol):

```css
.parent { display: flex; justify-content: center; align-items: center; }
.parent { display: grid; place-items: center; }      /* eng qisqa */
.child  { margin-inline: auto; }                     /* faqat gorizontal, block element uchun */
```

---

### I13. [Junior] Responsive va mobile-first yondashuv qanday yoziladi?

**Oddiy qilib:** **Mobile-first**: avval **telefon** uchun asosiy stil yoziladi, keyin `min-width` media query bilan katta ekranlar uchun **qo'shimchalar**. Telefon uchun stil oddiyroq bo'lgani uchun bu tabiiyroq va telefon kamroq CSS'ni qayta yozadi.

```css
/* Telefon (asosiy) */
.menu { display: grid; grid-template-columns: 1fr; gap: 1rem; }

/* Planshet va undan katta */
@media (min-width: 768px) {
  .menu { grid-template-columns: repeat(2, 1fr); }
}

/* Kompyuter */
@media (min-width: 1024px) {
  .menu { grid-template-columns: repeat(3, 1fr); }
}
```

**Media query'siz moslashuv** vositalari:

```css
h1 { font-size: clamp(1.5rem, 4vw, 3rem); }   /* min 1.5rem, max 3rem, oraliqda ekranga qarab */
img { max-width: 100%; height: auto; }          /* rasm konteynerdan chiqmaydi */
```

**Intervyuda shunday ayting:** "Mobile-first yozaman: asosiy stil telefon uchun, `min-width` media query bilan kattaroq ekranlar. Iloji boricha media query'siz moslashuvchan usullardan foydalanaman: `clamp()` shrift uchun, `auto-fit` bilan grid, `max-width: 100%` rasmlar uchun."

---

### I14. [Middle] Container queries nima? Media query'dan farqi?

**Oddiy qilib:** Media query **ekran** kengligiga qaraydi. Container query esa component'ning **o'z konteyneri** kengligiga qaraydi. Bu qayta ishlatiladigan component'lar uchun juda muhim: bitta kartochka sidebar'da tor, asosiy qismda keng bo'lishi mumkin, ekran o'lchami esa bir xil.

```css
.card-wrapper { container-type: inline-size; }

.card { display: flex; flex-direction: column; }

@container (min-width: 400px) {
  .card { flex-direction: row; }   /* konteyner 400px'dan keng bo'lsa, gorizontal */
}
```

---

### I15. [Junior] CSS custom properties (variables) nima? Dark mode qanday qilinadi?

**Oddiy qilib:** CSS o'zgaruvchilari qiymatni bir joyda saqlab, hamma joyda ishlatish imkonini beradi. Eng muhimi: ular **brauzerda, ishlab turgan paytda** o'zgaradi, shuning uchun dark mode faqat o'zgaruvchilarni almashtirish bilan qilinadi.

```css
:root {
  --color-bg: #ffffff;
  --color-text: #1f2937;
  --color-brand: #4f46e5;
  --radius-md: 8px;
}

.dark-mode {                         /* <html class="dark-mode"> bo'lganda */
  --color-bg: #111827;
  --color-text: #f3f4f6;
}

body { background: var(--color-bg); color: var(--color-text); }
.btn { background: var(--color-brand); border-radius: var(--radius-md); }

/* Tizim sozlamasiga qarab avtomatik */
@media (prefers-color-scheme: dark) {
  :root:not(.light-mode) { --color-bg: #111827; --color-text: #f3f4f6; }
}
```

```js
document.documentElement.classList.toggle("dark-mode");   // React'da tugma orqali
```

**Intervyuda shunday ayting:** "Ranglarni CSS variables qilib saqlayman, dark mode'da faqat o'zgaruvchilar qiymatini almashtiraman, component'larga tegish shart emas. Foydalanuvchi tanlovini localStorage'da saqlayman, birinchi kirishda esa `prefers-color-scheme` ga qarayman. Wild Oasis'dagi dark mode aynan shunday qurilgan."

---

### I16. [Junior] Pseudo-class va pseudo-element farqi?

**Oddiy qilib:** **Pseudo-class** (bitta `:`) elementning **holati**: sichqoncha ustidami, fokusdami, birinchimi. **Pseudo-element** (ikkita `::`) elementning **virtual qismi**: oldiga yoki orqasiga qo'shiladigan kontent, birinchi harf, placeholder.

```css
/* Pseudo-class: holat */
.btn:hover { background: var(--color-brand-dark); }
.btn:focus-visible { outline: 2px solid var(--color-brand); }   /* faqat klaviatura fokusida */
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
li:nth-child(odd) { background: #f9fafb; }     /* toq qatorlar */
input:invalid { border-color: red; }

/* Pseudo-element: virtual qism */
.required::after { content: " *"; color: red; }
input::placeholder { color: #9ca3af; }
```

**Qo'shimcha savol:** "`:focus` va `:focus-visible` farqi?" → `:focus-visible` faqat **klaviatura** bilan fokus olinganda ishlaydi. Sichqoncha bilan bosganda fokus halqasi ko'rinmaydi, klaviatura foydalanuvchisi uchun esa ko'rinadi.

---

### I17. [Strong junior] Zamonaviy CSS imkoniyatlari: `:has()`, nesting va `@layer`

**Oddiy qilib:** Oxirgi yillarda CSS'ga oldin faqat JavaScript yoki SASS bilan qilinadigan narsalar qo'shildi.

```css
/* :has() — "ota-element selektori": ichida nimadir bo'lgan elementni tanlash */
.card:has(img) { padding-top: 0; }                    /* rasmli kartochkalar */
label:has(input:checked) { background: #fef3c7; }    /* tanlangan variant: JS'siz */
form:has(input:invalid) .submit { opacity: 0.5; }

/* Native nesting — SASS'siz */
.card {
  padding: 1rem;
  & h2 { font-size: 1.25rem; }
  &:hover { box-shadow: 0 4px 12px rgb(0 0 0 / 0.1); }
}

/* @layer — qaysi stillar guruhi ustun kelishini tartib bilan belgilash */
@layer reset, components, utilities;
@layer components { .btn { padding: 1rem; } }
@layer utilities { .p-0 { padding: 0; } }    /* keyingi qatlam: specificity'dan qat'i nazar ustun */
```

**Intervyuda shunday ayting:** "`:has()` bilan ota-elementni child holatiga qarab stillash mumkin, masalan tanlangan radio'ni o'rab turgan label'ni JS'siz belgilash. Fast React Pizza'da Tailwind v4 dagi `has-checked:` variantini shu maqsadda ishlatganman."

---

### I18. [Strong junior] Transition va animation farqi? Performance uchun qaysi xossalarni animatsiya qilish kerak?

**Oddiy qilib:** **Transition** A holatdan B holatga silliq o'tish (masalan, hover'da). **Animation** (`@keyframes`) mustaqil, ko'p bosqichli, takrorlanadigan harakat (masalan, aylanuvchi loader).

```css
.btn {
  transition: background-color 200ms ease, transform 200ms ease;
}
.btn:hover { transform: translateY(-2px); }

@keyframes spin { to { transform: rotate(360deg); } }
.loader { animation: spin 1s linear infinite; }

/* Harakatga sezgir foydalanuvchilar uchun */
@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}
```

**Performance:** brauzer sahifani uch bosqichda chizadi: **layout** (o'lcham va joylarni hisoblash) → **paint** (piksellarni chizish) → **composite** (qatlamlarni birlashtirish).

| Animatsiya qilinadigan xossa | Qaysi bosqichni qayta ishlatadi | Natija |
|---|---|---|
| `width`, `height`, `top`, `left`, `margin` | Layout + paint + composite | Sekin, "qotadi" |
| `background-color`, `color`, `box-shadow` | Paint + composite | O'rtacha |
| **`transform`**, **`opacity`** | Faqat composite | **Silliq, 60 fps** |

**Intervyuda shunday ayting:** "Animatsiyani faqat `transform` va `opacity` bilan qilaman, chunki ular layout'ni qayta hisoblamaydi va GPU'da ishlaydi. Elementni siljitish uchun `left` emas, `translate`. `prefers-reduced-motion` ni ham hurmat qilaman."

---

### I19. [Junior] Uzun matnni qanday qisqartirasiz (`...`)?

```css
/* Bir qator */
.title {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Bir nechta qator (masalan, 3 qator) */
.description {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

Flex ichida ishlamasa: element'ga `min-width: 0` (I9-savol).

---

### I20. [Junior] CSS metodologiyalari: BEM nima?

**Oddiy qilib:** Oddiy global CSS'da class nomlari to'qnashadi: ikki dasturchi ikki joyda `.title` yozadi va biri ikkinchisini buzadi. **BEM** nomlash qoidasi: `block__element--modifier`.

```css
.card { }                     /* Block: mustaqil component */
.card__title { }              /* Element: block'ning qismi */
.card__button--disabled { }   /* Modifier: holat yoki variant */
```

Hozirgi React loyihalarida bu muammo boshqacha hal qilinadi: **CSS Modules** (class nomlari avtomatik noyob), **Tailwind** (utility class'lar), **styled-components** (stil component ichida). Shuning uchun BEM kamroq kerak, lekin global CSS va SASS loyihalarida hamon uchraydi.

---

### I21. [Strong junior] Cross-browser moslikni qanday ta'minlaysiz?

**Oddiy qilib:** Har xil brauzer (Chrome, Safari, Firefox) va ularning eski versiyalari ba'zi CSS va JavaScript imkoniyatlarini qo'llamasligi mumkin.

| Vosita | Vazifasi |
|---|---|
| **caniuse.com** | Imkoniyat qaysi brauzerlarda ishlashini tekshirish |
| **Autoprefixer** | `-webkit-` kabi vendor prefikslarni avtomatik qo'shadi (Vite/PostCSS'da) |
| **`browserslist`** | Qaysi brauzerlarni qo'llash kerakligini bitta joyda belgilash |
| **`@supports`** | Brauzer qo'llasagina stil berish (progressive enhancement) |
| **Real qurilmada test** | Ayniqsa iPhone'dagi **Safari**: eng ko'p muammo shu yerda |

```css
.card { display: flex; }
@supports (display: grid) {
  .card { display: grid; }    /* qo'llaydigan brauzerlarda yaxshiroq variant */
}
```

Safari'ning tipik muammolari: `100vh` va manzil satri (`dvh` bilan hal qilinadi), sana input'lari ko'rinishi, ba'zi yangi CSS imkoniyatlarining kech qo'llanishi.

**Intervyuda shunday ayting:** "Yangi CSS imkoniyatini ishlatishdan oldin caniuse'dan tekshiraman, prefikslarni Autoprefixer qo'shadi. Ayniqsa iPhone Safari'da real qurilmada sinab ko'raman, chunki muammolarning ko'pi o'sha yerda chiqadi."

---

### I22. [Junior] Pixel-perfect nima? Responsive bilan ziddiyat yo'qmi?

**Oddiy qilib:** Pixel-perfect: dizayn berilgan ekran o'lchamlarida (masalan, 375px va 1440px) natija Figma'ga **aniq** mos kelishi. Lekin foydalanuvchilar 320px dan 2560px gacha har xil ekranda kiradi, oraliq o'lchamlarda layout **mantiqiy** moslashishi kerak, shuning uchun "har pikselda bir xil" bo'lishi mumkin emas.

Amalda: Figma'dagi ranglar, shriftlar, bo'shliqlar **design tokens** sifatida olinadi, o'lchamlar qat'iy `px` emas, moslashuvchan birliklarda (`rem`, `%`, `clamp`) yoziladi, oraliq holatlar dizayner bilan oldindan kelishiladi. Tekshirish uchun brauzer kengaytmalari (masalan, PerfectPixel) dizayn rasmini sahifa ustiga qo'yib solishtiradi.

---

## J. SASS / SCSS

### J1. [Junior] SASS nima? SCSS va SASS farqi?

**Oddiy qilib:** SASS bu CSS'ning "kuchaytirilgan versiyasi": o'zgaruvchilar, ichma-ich yozish, qayta ishlatiladigan bloklar (mixin) va funksiyalar beradi. Brauzer uni tushunmaydi, **build paytida** oddiy CSS'ga aylanadi (Vite buni `sass` paketi o'rnatilsa avtomatik qiladi).

Ikki sintaksis bor: **SCSS** (`.scss`, CSS'ga o'xshash, `{}` va `;` bilan, eng ko'p ishlatiladi) va eski **SASS** (`.sass`, qavslarsiz, chekinish bilan).

```scss
$brand: #4f46e5;                 // o'zgaruvchi
$radius: 8px;

.card {
  border-radius: $radius;
  padding: 1rem;

  &__title { font-size: 1.25rem; }        // .card__title (BEM bilan qulay)
  &:hover { box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1); }

  .btn { background: $brand; }            // .card .btn
}
```

---

### J2. [Strong junior] Mixin, `@extend` va partial'lar nima?

**Oddiy qilib:** **Mixin** bu qayta ishlatiladigan CSS bo'lagi, funksiya kabi argument ham oladi. **Partial** (`_` bilan boshlanadigan fayl) stillarni fayllarga bo'lish uchun.

```scss
// _mixins.scss
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

@mixin respond($breakpoint) {
  @if $breakpoint == tablet { @media (min-width: 768px) { @content; } }
  @if $breakpoint == desktop { @media (min-width: 1024px) { @content; } }
}

// card.scss
@use "mixins" as m;

.modal {
  @include m.flex-center;
  @include m.respond(tablet) { padding: 2rem; }
}
```

`@extend` bir selektor stillarini boshqasiga "meros qilib" beradi, lekin kutilmagan katta selektorlar hosil qilishi mumkin, shuning uchun ko'pchilik mixin'ni afzal ko'radi. Fayllarni ulashda eski `@import` o'rniga **`@use`** ishlatiladi (`@import` eskirgan).

---

### J3. [Strong junior] SASS o'zgaruvchilari va CSS custom properties farqi? Hozir SASS kerakmi?

| | SASS `$variable` | CSS `--variable` |
|---|---|---|
| Qachon ishlaydi | **Build** paytida (keyin yo'qoladi) | **Brauzerda**, runtime'da |
| JavaScript bilan o'zgartirish | Yo'q | Ha |
| Dark mode, tema | Qiyin | Juda oson |
| Media query ichida o'zgarish | Yo'q | Ha |

Zamonaviy CSS'da **o'zgaruvchilar** va **nesting** native mavjud, shuning uchun SASS'ga ehtiyoj kamaygan. U hali ham mixin, funksiya va sikllar uchun foydali, eski loyihalarda esa juda keng tarqalgan.

**Intervyuda shunday ayting:** "SASS'ni bilaman va loyihalarda ishlatganman. Lekin dark mode va tema uchun CSS variables ishlataman, chunki ular runtime'da o'zgaradi. Yangi loyihalarda esa ko'pincha Tailwind yoki CSS Modules tanlayman."

---

## K. Framer Motion

> **Eslatma:** kutubxona endi **Motion** deb nomlanadi (`npm install motion`, `import { motion } from "motion/react"`). API asosan bir xil, eski `framer-motion` paketi ham ishlaydi.

### K1. [Junior] Framer Motion'ning asosiy tushunchalari qanday?

**Oddiy qilib:** Oddiy HTML teg o'rniga `motion.` prefiksli teg yoziladi va unga "qayerdan qayerga" harakatlanishi aytiladi. Kutubxona animatsiyani o'zi hisoblaydi.

```tsx
import { motion } from "framer-motion";

<motion.div
  initial={{ opacity: 0, y: 20 }}         // boshlanish holati
  animate={{ opacity: 1, y: 0 }}          // oxirgi holat
  exit={{ opacity: 0, y: -20 }}           // olib tashlanayotganda (AnimatePresence bilan)
  transition={{ duration: 0.3, ease: "easeOut" }}
/>

<motion.button
  whileHover={{ scale: 1.05 }}            // sichqoncha ustida
  whileTap={{ scale: 0.95 }}              // bosilganda
/>
```

`transition` turlari: `tween` (davomiylik bilan, `duration`), `spring` (prujina kabi, tabiiyroq: `stiffness`, `damping`).

**Intervyuda shunday ayting:** "Framer Motion'da `motion` component'lariga `initial`, `animate` va `exit` holatlarini beraman, kutubxona o'tishni o'zi hisoblaydi. Bogizilol'da sahifa va kartochkalarning paydo bo'lishi, hover effektlari uchun ishlatganman."

---

### K2. [Strong junior] Variants va `staggerChildren` nima?

**Oddiy qilib:** **Variants** holatlarga **nom** berish (`hidden`, `visible`). Ota-element holatini o'zgartirsa, child'lar ham avtomatik o'sha nomli holatga o'tadi. **`staggerChildren`** child'larni birin-ketin, kichik kechikish bilan animatsiya qiladi (ro'yxat "to'lqin" kabi paydo bo'ladi).

```tsx
const list = {
  hidden: { opacity: 0 },
  visible: { opacity: 1, transition: { staggerChildren: 0.08 } },
};
const item = {
  hidden: { opacity: 0, y: 10 },
  visible: { opacity: 1, y: 0 },
};

<motion.ul variants={list} initial="hidden" animate="visible">
  {dishes.map((dish) => (
    <motion.li key={dish.id} variants={item}>{dish.name}</motion.li>   // holat nomi otadan keladi
  ))}
</motion.ul>
```

---

### K3. [Strong junior] `AnimatePresence` nima uchun kerak?

**Oddiy qilib:** React element'ni DOM'dan **darhol** olib tashlaydi, shuning uchun "yo'qolish" animatsiyasini ko'rsatishga vaqt qolmaydi. `AnimatePresence` element o'chirilayotganini sezadi, avval uning `exit` animatsiyasini o'ynaydi, **keyin** DOM'dan olib tashlaydi.

```tsx
import { AnimatePresence, motion } from "framer-motion";

<AnimatePresence>
  {isOpen && (
    <motion.div
      key="modal"                         // barqaror key majburiy
      initial={{ opacity: 0, scale: 0.95 }}
      animate={{ opacity: 1, scale: 1 }}
      exit={{ opacity: 0, scale: 0.95 }}
    />
  )}
</AnimatePresence>

// Savatdan mahsulot o'chirilganda ro'yxat elementi silliq yo'qoladi
<AnimatePresence>
  {cart.map((item) => (
    <motion.li key={item.id} exit={{ opacity: 0, x: -50 }} layout>...</motion.li>
  ))}
</AnimatePresence>
```

**Qo'shimcha savol:** "Sahifalar orasidagi o'tish animatsiyasi qanday qilinadi?" → `AnimatePresence` ichida route content'ini joylashtirib, `key` sifatida `location.pathname` beriladi.

---

### K4. [Strong junior] Layout animatsiyalari: `layout` va `layoutId` nima?

**Oddiy qilib:** Element **joyi yoki o'lchami** o'zgarganda (ro'yxatdagi element o'chirilib, qolganlar yuqoriga surilganda) odatda u "sakraydi". **`layout`** prop bu o'zgarishni silliq animatsiya qiladi. **`layoutId`** esa **ikki xil** element'ni "bitta element" deb biladi va biridan ikkinchisiga silliq o'tadi (masalan, tanlangan tab ostidagi chiziq yoki kichik kartochkadan katta oynaga o'tish).

```tsx
// Tanlangan tab ostidagi chiziq bir tabdan boshqasiga "suzib" o'tadi
{tabs.map((tab) => (
  <button key={tab} onClick={() => setActive(tab)}>
    {tab}
    {active === tab && <motion.div layoutId="underline" className="underline" />}
  </button>
))}
```

---

### K5. [Junior] Scroll paytida paydo bo'ladigan animatsiya qanday qilinadi?

```tsx
<motion.section
  initial={{ opacity: 0, y: 40 }}
  whileInView={{ opacity: 1, y: 0 }}        // ekranga kirganda
  viewport={{ once: true, amount: 0.3 }}    // bir marta, 30% ko'ringanda
  transition={{ duration: 0.5 }}
/>
```

Landing sahifalarda (Sapphire Transportation kabi) bo'limlar scroll paytida paydo bo'lishi uchun. Scroll progressiga bog'liq animatsiya (masalan, yuqoridagi progress chizig'i) uchun `useScroll` hook'i bor.

---

### K6. [Middle] Animatsiya qachon zarar qiladi? Performance va accessibility'ni qanday saqlaysiz?

**Oddiy qilib:** Animatsiya chiroyli, lekin noto'g'ri ishlatilsa saytni sekinlashtiradi va ba'zi odamlarga jismoniy noqulaylik (bosh aylanishi) keltiradi.

- **Faqat `transform` va `opacity`** (`x`, `y`, `scale`, `rotate`, `opacity`): Framer Motion ularni GPU'da bajaradi. `width`, `height`, `top` animatsiyasidan qochish (I18-savol)
- **`prefers-reduced-motion`** ni hurmat qilish:

```tsx
import { useReducedMotion } from "framer-motion";
const reduce = useReducedMotion();
<motion.div animate={{ x: reduce ? 0 : 100 }} />

// yoki global
<MotionConfig reducedMotion="user">...</MotionConfig>
```

- Animatsiya **qisqa** (odatda 150–400 ms) va **ma'noli** bo'lishi kerak: holat o'zgarishini ko'rsatish, e'tiborni yo'naltirish. Har bir element harakatlanadigan sayt foydalanuvchini charchatadi
- Juda ko'p element bir vaqtda animatsiya qilinmasin (uzun ro'yxatlarda)
- Bundle hajmi: oddiy hover yoki fade uchun kutubxona shart emas, CSS transition yetarli

**Intervyuda shunday ayting:** "Oddiy hover va fade'ni CSS bilan qilaman. Framer Motion'ni chiqish animatsiyasi, layout animatsiyasi va murakkab ketma-ketliklar uchun ishlataman, chunki ularni CSS'da qilish qiyin. Faqat transform va opacity'ni animatsiya qilaman va `prefers-reduced-motion` ni hurmat qilaman."

---

## L. UI kutubxonalari: Material UI va DaisyUI

### L1. [Junior] Material UI (MUI) nima? Qanday ishlatiladi?

**Oddiy qilib:** MUI bu Google'ning **Material Design** uslubidagi tayyor React component'lar to'plami: tugmalar, inputlar, jadval, modal, sana tanlagich va boshqalar. Ular ichida holat, accessibility va animatsiya tayyor.

```tsx
import { Button, TextField, Dialog } from "@mui/material";

<TextField label="Email" variant="outlined" error={!!error} helperText={error} />
<Button variant="contained" color="primary" onClick={handleSave}>Saqlash</Button>
```

**Intervyuda shunday ayting:** "MUI tez yig'iladigan admin panel va ichki tizimlar uchun qulay: tayyor, accessibility'si o'ylangan component'lar. Kamchiligi: o'ziga xos dizaynga moslash ko'proq kuch talab qiladi va bundle og'irroq."

---

### L2. [Strong junior] MUI'ni loyiha dizayniga qanday moslaysiz? `sx`, `styled` va theme farqi?

**Oddiy qilib:** Uch daraja bor, **eng kengidan** boshlash kerak:

```tsx
// 1. Theme: butun ilova uchun (ranglar, shrift, component default'lari)
const theme = createTheme({
  palette: { primary: { main: "#4f46e5" } },
  typography: { fontFamily: "Inter, sans-serif" },
  shape: { borderRadius: 8 },
  components: {
    MuiButton: { defaultProps: { disableElevation: true }, styleOverrides: { root: { textTransform: "none" } } },
  },
});
<ThemeProvider theme={theme}><App /></ThemeProvider>

// 2. styled(): qayta ishlatiladigan o'zgartirilgan component
const RoundButton = styled(Button)({ borderRadius: 999 });

// 3. sx: bitta joydagi kichik o'zgarish
<Box sx={{ p: 2, display: "flex", gap: 1, bgcolor: "background.paper" }} />
```

Qoida: har bir component'da `sx` bilan bir xil o'zgarishni takrorlash o'rniga, **theme**'da bir marta sozlash. Shunda dizayn izchil qoladi.

---

### L3. [Junior] DaisyUI nima? Tailwind'dan farqi?

**Oddiy qilib:** DaisyUI bu **Tailwind plagini**: Tailwind'ning ko'plab utility class'larini yozish o'rniga, tayyor **component class'lari** beradi (`btn`, `card`, `modal`, `badge`). Tailwind bilan birga ishlaydi: kerak bo'lsa utility class bilan ustidan yozasiz.

```html
<!-- Faqat Tailwind -->
<button class="rounded-lg bg-indigo-600 px-4 py-2 font-semibold text-white hover:bg-indigo-700">Saqlash</button>

<!-- DaisyUI -->
<button class="btn btn-primary">Saqlash</button>
<button class="btn btn-primary btn-sm rounded-full">Kichik</button>   <!-- Tailwind bilan moslash -->
```

Tayyor **temalar** (light, dark va boshqalar) `data-theme` atributi bilan almashtiriladi. Muhim farq: DaisyUI faqat **CSS class'lar**, JavaScript mantiqi yo'q. Modal ochish-yopish, dropdown holati kabilarni o'zingiz boshqarasiz (React state yoki native `<dialog>`).

O'rnatish versiyaga bog'liq: Tailwind v4 bilan CSS ichida `@plugin "daisyui";`, eski v3 da `tailwind.config.js` dagi `plugins` ro'yxatida.

---

### L4. [Strong junior] MUI, DaisyUI va shadcn/ui farqi nima?

| | MUI | DaisyUI | shadcn/ui |
|---|---|---|---|
| Nima | To'liq React component kutubxonasi | Tailwind uchun component class'lari | Component **kodini loyihangizga ko'chiradi** |
| JS mantiq (modal, select) | Ichida tayyor | Yo'q (o'zingiz) | Bor (Radix UI ustida) |
| Dizaynni moslash | Theme orqali, qiyinroq | Oson (Tailwind) | To'liq nazorat (kod sizda) |
| Accessibility | Yaxshi | Siz qanday yozsangiz | Yaxshi (Radix) |
| Bundle | Og'irroq | Juda yengil | Faqat ishlatilgani |
| Qachon | Tez admin panel, enterprise | Kichik loyiha, prototip | O'ziga xos dizaynli zamonaviy ilova |

**Intervyuda shunday ayting:** "Tez yig'iladigan ichki admin panel uchun MUI, Tailwind'li kichik loyiha uchun DaisyUI. Bogizilol'da shadcn/ui ishlatganman: kod loyihamda bo'lgani uchun dizaynni to'liq moslay olaman, accessibility esa Radix'dan keladi."

---

### L5. [Middle] UI kutubxonasini tanlashda nimalarga qaraysiz?

1. **Dizayn talablari**: o'ziga xos brend dizayni bormi yoki standart ko'rinish yetarlimi
2. **Moslash qulayligi**: theme, class yoki kod darajasida o'zgartirish mumkinmi
3. **Accessibility**: klaviatura, focus boshqaruvi, ARIA tayyormi (o'zingiz yozish qiyin va xato ko'p)
4. **Bundle hajmi** va tree shaking
5. **TypeScript** qo'llovi, hujjat sifati, hamjamiyat va yangilanib turishi
6. **Jamoa tajribasi** va mavjud stack (masalan, Tailwind bo'lsa, MUI aralashtirish noqulay)

Noyob dizayn kerak bo'lsa, ko'pincha **headless** kutubxona (Radix UI, Headless UI, React Aria) tanlanadi: ular faqat mantiq va accessibility beradi, dizaynni o'zingiz qilasiz.
