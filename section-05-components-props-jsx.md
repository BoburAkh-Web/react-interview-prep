# Section 5: Components, Props, and JSX

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**. This section has no Middle-level questions.

---

## 1. [Junior] Component nima va nega nomi katta harf bilan boshlanadi?

Component bu UI bo'lagini qaytaradigan JavaScript funksiya. Katta harf React'ga `<Menu />` ni oddiy HTML teg emas, o'zimiz yozgan component ekanini bildiradi. Kichik harf bilan yozsa (`<menu />`), React uni HTML teg deb o'ylaydi.

## 2. [Junior] JSX nima? Brauzer uni tushunadimi?

Yo'q. JSX bu JavaScript ichida HTML'ga o'xshash yozuv. Brauzer uni tushunmaydi, uni build vositasi (Vite orqali Babel/SWC) oddiy JavaScript'ga aylantiradi:

```jsx
<h1 className="title">Salom</h1>
// aylanadi:
React.createElement("h1", { className: "title" }, "Salom")
```

Shuning uchun JSX majburiy emas, faqat qulaylik uchun.

## 3. [Junior] JSX'da nega bitta root element kerak?

Funksiya faqat bitta qiymat qaytara oladi, JSX esa `createElement` chaqiruvi. Ko'p element kerak bo'lsa, ortiqcha `<div>` o'rniga Fragment ishlatiladi: `<>...</>`.

## 4. [Junior] React Strict Mode nima qiladi? Nega component ikki marta render bo'ladi?

Strict Mode faqat **development**da ishlaydi va xatolarni erta topish uchun component'ni ataylab ikki marta render qiladi (React 18 da effect'larni ham bir marta qo'shimcha ishga tushiradi). Agar component "toza" (pure) bo'lmasa, masalan render ichida tashqi o'zgaruvchini o'zgartirsa, bu xato ikki marta ishlaganda ko'rinib qoladi. Production'da bu yo'q.

## 5. [Strong junior] "Declarative" va "imperative" yondashuv farqi nima? React qaysi biri?

Imperative: **qanday** qilishni qadamma-qadam yozasiz (`document.createElement`, `appendChild`...). Declarative: **nima** ko'rinishi kerakligini yozasiz, qolganini React hal qiladi. React declarative: state o'zgaradi, UI o'zi yangilanadi.

## 6. [Strong junior] Props nima? Nega component ichida props'ni o'zgartirib bo'lmaydi?

Props bu parent'dan child'ga beriladigan ma'lumot. U **read-only**, chunki React'da ma'lumot **bir tomonlama** (parent → child) oqadi. Child props'ni o'zgartirsa, parent bilan child'ning ma'lumoti kelishmay qoladi va xatolarni topish qiyinlashadi. O'zgaradigan narsa uchun state ishlatiladi.

## 7. [Junior] Ro'yxat render qilishda `key` nega kerak?

`key` React'ga ro'yxatdagi har bir element qaysi ekanini aytadi. Ro'yxat o'zgarganda (qo'shish, o'chirish, tartibini almashtirish) React eski va yangi elementlarni `key` orqali solishtiradi va faqat o'zgarganini yangilaydi. `key` o'zgarmas va noyob bo'lishi kerak, odatda ma'lumotning `id`si.

## 8. [Strong junior] `key` sifatida massiv indeksini ishlatsa nima bo'ladi?

Ro'yxat statik bo'lsa muammo yo'q. Lekin element qo'shilsa, o'chirilsa yoki tartibi o'zgarsa, indekslar boshqa elementlarga ko'chadi va React noto'g'ri elementga eski state'ni (masalan, input ichidagi matnni) biriktirib qo'yadi. Shuning uchun `id` ishlatiladi.

## 9. [Junior] `&&` bilan shartli render qilishdagi mashhur xato nima?

```jsx
{count && <p>Mahsulotlar bor</p>}   // count = 0 bo'lsa, ekranda "0" chiqadi
```

`0` falsy, lekin React uni raqam sifatida chizadi. Tuzatish:

```jsx
{count > 0 && <p>Mahsulotlar bor</p>}
```

## 10. [Junior] Shartli renderning 3 usuli va qachon qaysi biri?

- **`&&`**: faqat ko'rsatish yoki ko'rsatmaslik (else kerak emas)
- **Ternary** (`? :`): ikki xil variantdan birini chizish
- **Early return** (`if (...) return null`): butun component'ni chizmaslik

JSX ichida `if` yozib bo'lmaydi, chunki `{}` ichiga faqat **expression** (qiymat) qo'yiladi, `if` esa statement.

## 11. [Strong junior] Early return'ni qayerga qo'yish kerak? Hook'lardan oldin qo'ysa nima bo'ladi?

Hook'lardan **keyin**. Hook'lar har renderda bir xil tartibda chaqirilishi shart (Rules of Hooks). Early return hook'dan oldin tursa, ba'zi renderlarda hook chaqirilmay qoladi va React xato beradi.

## 12. [Junior] Props'ni destructuring qilish nima uchun kerak?

```jsx
function Card({ name, age }) { ... }   // props.name, props.age o'rniga
```

Kod qisqa va o'qilishi oson bo'ladi, qaysi props kutilayotgani funksiya imzosidayoq ko'rinadi. Default qiymat ham berish mumkin: `{ age = 18 }`.

## 13. [Junior] React Fragment nima va `<div>` dan farqi?

Fragment (`<>...</>`) DOM'ga qo'shimcha element qo'shmaydi. `<div>` esa ortiqcha tugun yaratadi, bu CSS layoutni (masalan, grid/flex, jadval) buzishi mumkin. Fragment'ga `key` kerak bo'lsa, qisqa yozuv ishlamaydi, `<React.Fragment key={id}>` yoziladi.

## 14. [Junior] React'da "Separation of Concerns" qanday tushuniladi?

Odatda HTML, CSS va JS alohida fayllarga bo'linadi (texnologiya bo'yicha). React'da esa **mas'uliyat bo'yicha** bo'linadi: bitta component o'z UI'si, logikasi va stilini birga saqlaydi. Natijada har bir bo'lak mustaqil va qayta ishlatiladigan bo'ladi.

## 15. [Junior] Shartli CSS class'ni qanday beramiz? `class` o'rniga nega `className`?

`class` JavaScript'da zahiralangan so'z (`class` kalit so'zi), shuning uchun JSX'da `className` yoziladi.

```jsx
<li className={`item ${isDone ? "done" : ""}`}>
```

## 16. [Junior] Component'ni qachon ajratish kerak?

Bitta component juda uzun bo'lib ketsa, bir xil JSX takrorlansa yoki alohida mas'uliyat paydo bo'lsa. Muhim xato: component'ni boshqa component'ning **ichida** e'lon qilmang. Har renderda u yangi funksiya bo'lib qoladi, React uni boshqa component deb o'ylab, qayta yaratadi va state yo'qoladi.
