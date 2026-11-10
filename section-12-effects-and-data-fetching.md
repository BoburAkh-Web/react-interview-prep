# Section 12: Effects and Data Fetching

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Projects in this section: usePopcorn (movie search app, davomi), Currency Converter.

---

## 1. [Junior] Component lifecycle nima? `useEffect` bilan qanday bog'liq?

Component'ning hayoti uch bosqichdan iborat:

1. **Mount**: component birinchi marta ekranga chiqadi (instance yaratiladi)
2. **Update (re-render)**: state yoki props o'zgargani uchun qayta render bo'ladi (bir necha marta bo'lishi mumkin)
3. **Unmount**: component ekrandan olib tashlanadi (instance yo'q qilinadi)

`useEffect` yordamida shu bosqichlarning ma'lum paytlarida kod bajarishimiz mumkin: masalan, mount bo'lganda ma'lumot yuklash.

## 2. [Junior] Nega ma'lumotni render ichida to'g'ridan-to'g'ri `fetch` qilib bo'lmaydi?

```jsx
function App() {
  const [movies, setMovies] = useState([]);

  fetch(`...&s=interstellar`)
    .then((res) => res.json())
    .then((data) => setMovies(data.Search));   // XATO!

  return ...
}
```

`fetch` bu **side effect** va render logikasi toza (pure) bo'lishi kerak. Bundan tashqari, bu kod **cheksiz tsikl** hosil qiladi:

```
render → fetch → setMovies → state o'zgardi → qayta render → fetch → setMovies → ...
```

Har render yangi so'rov yuboradi, so'rov natijasi esa yana render'ni boshlaydi. Yechim: fetch'ni `useEffect` ichiga olish.

## 3. [Junior] `useEffect` nima va qanday yoziladi? Side effect nima?

**Side effect**: component'ning tashqi dunyo bilan o'zaro ta'siri, ya'ni render'ning "yon natijasi" (API so'rovi, timer, DOM'ni o'zgartirish, event listener, `localStorage` bilan ishlash). Bunday kodni render ichiga emas, event handler yoki `useEffect` ichiga yozamiz.

```jsx
useEffect(function () {
  // side effect shu yerda
}, [dependency1, dependency2]);
```

Birinchi argument: effect funksiyasi, ikkinchisi: dependency array (qachon qayta ishlashini belgilaydi).

## 4. [Strong junior] Effect aynan qachon ishlaydi?

Effect funksiyasi **render tugagach va ekran chizilgandan keyin** ishlaydi (render paytida emas), shuning uchun u UI'ni kutdirmaydi. Keyin qayta ishlashi dependency array'ga bog'liq (8-savol).

## 5. [Strong junior] Nega `useEffect` ga to'g'ridan-to'g'ri `async` funksiya bera olmaymiz?

`async` funksiya doim **Promise** qaytaradi. `useEffect` esa effect funksiyasidan faqat **cleanup funksiya yoki hech narsa** (`undefined`) kutadi. Shuning uchun ichida alohida async funksiya e'lon qilib, uni chaqiramiz:

```jsx
useEffect(function () {
  async function fetchMovies() {
    const res = await fetch(`...&s=${query}`);
    const data = await res.json();
    setMovies(data.Search);
  }
  fetchMovies();
}, [query]);
```

## 6. [Junior] Loading state qanday qo'shiladi?

Ma'lumot kelguncha foydalanuvchiga "Loading..." ko'rsatish uchun alohida state:

```jsx
const [isLoading, setIsLoading] = useState(false);

async function fetchMovies() {
  setIsLoading(true);
  const res = await fetch(...);
  const data = await res.json();
  setMovies(data.Search);
  setIsLoading(false);
}

{isLoading ? <Loader /> : <MovieList movies={movies} />}
```

Muvaffaqiyatli bo'lsa ham, xato bo'lsa ham loading to'xtashi uchun `setIsLoading(false)` ni `finally` blokiga qo'yish tavsiya etiladi (7-savol).

## 7. [Strong junior] Fetch xatolarini qanday to'g'ri qayta ishlaymiz?

Muhim: `fetch` faqat **tarmoq xatosi** (internet yo'q) bo'lganda reject bo'ladi. `404` yoki `500` kabi HTTP xatolarida **reject qilmaydi**, shuning uchun `res.ok` ni qo'lda tekshiramiz:

```jsx
const [error, setError] = useState("");

async function fetchMovies() {
  try {
    setIsLoading(true);
    setError("");
    const res = await fetch(...);
    if (!res.ok) throw new Error("Something went wrong with fetching movies");

    const data = await res.json();
    // ba'zi API'lar (masalan, OMDb) xatoni 200 status ichida qaytaradi:
    if (data.Response === "False") throw new Error("Movie not found");

    setMovies(data.Search);
  } catch (err) {
    setError(err.message);
  } finally {
    setIsLoading(false);
  }
}

{isLoading && <Loader />}
{!isLoading && !error && <MovieList movies={movies} />}
{error && <ErrorMessage message={error} />}
```

Uch holat (loading, success, error) alohida boshqariladi va faqat bittasi ko'rsatiladi.

## 8. [Junior] Dependency array nima? Uch variantining farqi?

| Yozilishi | Effect qachon ishlaydi |
|---|---|
| `useEffect(fn, [])` | Faqat **mount** bo'lganda (bir marta) |
| `useEffect(fn, [a, b])` | Mount'da va `a` yoki `b` **o'zgarganda** |
| `useEffect(fn)` (array yo'q) | **Har renderdan keyin** (odatda kerak emas) |

## 9. [Strong junior] Dependency array'ga nimalarni yozish kerak? Nega?

Effect ichida ishlatilgan har bir **reaktiv qiymat**ni (state, props va ulardan hosil bo'lgan o'zgaruvchilar) yozish kerak. Aks holda effect eski (**stale**) qiymat bilan ishlab qolishi mumkin: funksiya yaratilgan paytdagi qiymatni "eslab" qoladi.

ESLint'ning `react-hooks/exhaustive-deps` qoidasi buni tekshiradi. Uni o'chirib qo'ymaslik kerak, chunki ogohlantirish ko'pincha haqiqiy xatoga ishora qiladi. Dependency array bu "effect qachon ishlasin" tugmasi emas, "effect qaysi qiymatlarga bog'liq" deb ta'riflash to'g'riroq.

## 10. [Strong junior] Qidiruv so'zi (`query`) o'zgarganda natijalar qanday yangilanadi? ("Synchronizing queries with movie data")

`query` state'ini effect'ning dependency array'iga qo'shamiz:

```jsx
useEffect(function () {
  if (query.length < 3) {
    setMovies([]);
    setError("");
    return;
  }
  fetchMovies();
}, [query]);
```

`query` har o'zgarganda effect qayta ishlaydi va yangi qidiruv natijasi yuklanadi. Effect shu yerda **component'ni tashqi tizim (API) bilan sinxronlab turadi**. Qisqa so'zlar uchun (`< 3` belgi) keraksiz so'rov yubormaslik uchun erta chiqamiz.

## 11. [Strong junior] Ma'lumotni event handler'da yuklash kerakmi yoki effect'dami?

Qoida: **foydalanuvchi harakatiga javob** bo'lsa, event handler; **component ekranda ko'rinishi uchun ma'lumot kerak bo'lsa** (yoki qiymat o'zgarganda sinxronlash kerak bo'lsa), effect.

- Tugma bosilganda forma yuborish: event handler
- Sahifa ochilganda yoki `query`/`selectedId` o'zgarganda ma'lumot yuklash: effect

React hujjatlari ham "effect'lar event'larga javob berish uchun emas" deb ogohlantiradi.

## 12. [Strong junior] Film tanlash (selecting a movie) qanday qilinadi?

`selectedId` state'ini parent'da saqlaymiz va bosilganda uni almashtiramiz (takror bosilsa yopiladi):

```jsx
const [selectedId, setSelectedId] = useState(null);

function handleSelectMovie(id) {
  setSelectedId((selectedId) => (id === selectedId ? null : id));
}
function handleCloseMovie() {
  setSelectedId(null);
}

{selectedId ? (
  <MovieDetails selectedId={selectedId} onCloseMovie={handleCloseMovie} />
) : (
  <>...</>
)}
```

Funksiyalar props orqali child'ga beriladi (child-to-parent aloqa).

## 13. [Strong junior] Tanlangan film tafsilotlarini yuklash uchun effect qayerda va qanday yoziladi?

`MovieDetails` component'ining o'zida, dependency array'iga `selectedId` ni qo'yib:

```jsx
function MovieDetails({ selectedId, onCloseMovie }) {
  const [movie, setMovie] = useState({});
  const [isLoading, setIsLoading] = useState(false);

  useEffect(function () {
    async function getMovieDetails() {
      setIsLoading(true);
      const res = await fetch(`...&i=${selectedId}`);
      const data = await res.json();
      setMovie(data);
      setIsLoading(false);
    }
    getMovieDetails();
  }, [selectedId]);
  ...
}
```

Boshqa film tanlanganda `selectedId` o'zgaradi va effect yangi film ma'lumotini yuklaydi. Ma'lumotni uni ishlatadigan component'ga yaqin yuklash mantiqni ixcham qiladi.

## 14. [Strong junior] Ko'rilgan filmni (watched movie) ro'yxatga qanday qo'shamiz?

`watched` state'i `App` da turadi (ikkala tomon, ya'ni tafsilotlar va ro'yxat unga muhtoj), qo'shish funksiyasi props orqali `MovieDetails` ga beriladi:

```jsx
// App
function handleAddWatched(movie) {
  setWatched((watched) => [...watched, movie]);
}

// MovieDetails
function handleAdd() {
  const newWatchedMovie = {
    imdbID: selectedId,
    title,
    year,
    poster,
    imdbRating: Number(imdbRating),
    runtime: Number(runtime.split(" ").at(0)),   // "148 min" → 148
    userRating,
  };
  onAddWatched(newWatchedMovie);
  onCloseMovie();
}
```

Muhim nuqtalar:

- API ma'lumotni **string** qaytaradi (`"148 min"`, `"8.6"`), saqlashdan oldin **songa aylantiramiz**, aks holda keyin hisoblashlar (o'rtacha, jami) xato chiqadi
- Film allaqachon ko'rilganmi, **derived** qilib aniqlanadi (alohida state emas): `watched.map((m) => m.imdbID).includes(selectedId)`. Shunga qarab baholash formasi o'rniga mavjud bahoni ko'rsatamiz
- State **immutable** yangilanadi (`[...watched, movie]`)

## 15. [Junior] Sahifa sarlavhasini (`document.title`) nega effect ichida o'zgartiramiz?

`document.title` ni o'zgartirish **DOM bilan to'g'ridan-to'g'ri ishlash**, ya'ni side effect. Render ichida bunday qilib bo'lmaydi.

```jsx
useEffect(function () {
  if (!title) return;
  document.title = `Movie | ${title}`;
}, [title]);
```

`title` o'zgarganda sarlavha ham yangilanadi. `title` hali yuklanmagan bo'lsa, `undefined` ko'rsatmaslik uchun erta chiqamiz.

## 16. [Junior] Cleanup funksiyasi nima va qachon ishlaydi?

Effect'dan **qaytariladigan funksiya**. U ikki holatda ishlaydi:

1. Effect **qayta ishlashidan oldin** (dependency o'zgarganda, oldingi effect'ni tozalash uchun)
2. Component **unmount** bo'lganda

```jsx
useEffect(function () {
  // sozlash (setup)
  return function () {
    // tozalash (cleanup)
  };
}, [dep]);
```

Nima uchun kerak: effect'ning "ortida qoldirgan izini" yig'ishtirish uchun: event listener'ni olib tashlash, timer'ni to'xtatish, so'rovni bekor qilish, sarlavhani qaytarish.

## 17. [Strong junior] Effect va cleanup qanday tartibda ishlaydi?

```
Mount:       effect (setup)
Dependency o'zgardi:  cleanup (eski render'niki) → effect (yangi render'niki)
Unmount:     cleanup
```

Ya'ni cleanup har doim **keyingi effect'dan oldin** ishlaydi. Shuning uchun effect'ni "sinxronlashni boshlash" va cleanup'ni "to'xtatish" juftligi sifatida o'ylash qulay.

## 18. [Strong junior] Sarlavhani tozalashda (`document.title = "usePopcorn"`) cleanup qanday qiymatlarni eslab qoladi? (Closure)

```jsx
useEffect(function () {
  if (!title) return;
  document.title = `Movie | ${title}`;

  return function () {
    document.title = "usePopcorn";   // component yo'qolganda asl sarlavhaga qaytaramiz
  };
}, [title]);
```

Cleanup component allaqachon yo'q bo'lgandan keyin ham ishlaydi, lekin u o'zi yaratilgan paytdagi o'zgaruvchilarni (`title` kabi) **closure** orqali eslab qoladi. JavaScript funksiyasi o'zi yaratilgan muhitdagi o'zgaruvchilarga ulangan holda qoladi. Shuning uchun cleanup eski render'ning qiymatlari bilan ishlaydi.

## 19. [Strong junior] Race condition nima? Fetch'ni cleanup bilan qanday tozalaymiz?

Foydalanuvchi tez yozsa (`a`, `ab`, `abc`), bir necha so'rov **bir vaqtda** yuboriladi. Javoblar tartibsiz kelishi mumkin: eski so'rovning javobi kechroq kelib, yangisining natijasini **ustidan yozib yuboradi** va ekranda noto'g'ri natija qoladi. Bu **race condition**.

Yechim: `AbortController` bilan eski so'rovni cleanup'da bekor qilish:

```jsx
useEffect(function () {
  const controller = new AbortController();

  async function fetchMovies() {
    try {
      setIsLoading(true);
      setError("");
      const res = await fetch(`...&s=${query}`, { signal: controller.signal });
      if (!res.ok) throw new Error("Something went wrong");
      const data = await res.json();
      if (data.Response === "False") throw new Error("Movie not found");
      setMovies(data.Search);
    } catch (err) {
      if (err.name !== "AbortError") setError(err.message);
    } finally {
      setIsLoading(false);
    }
  }

  fetchMovies();

  return function () {
    controller.abort();   // eski so'rovni bekor qiladi
  };
}, [query]);
```

Muhim detal: `abort()` so'rovni `AbortError` bilan reject qiladi. Bu **haqiqiy xato emas**, shuning uchun `catch` da uni foydalanuvchiga ko'rsatmaymiz. Bonus: bekor qilingan so'rov tarmoq trafigini ham tejaydi.

## 20. [Strong junior] Klaviatura tugmasini (masalan, Escape) tinglash effect'i qanday yoziladi? Nega cleanup shart?

```jsx
useEffect(function () {
  function callback(e) {
    if (e.code === "Escape") onCloseMovie();
  }

  document.addEventListener("keydown", callback);

  return function () {
    document.removeEventListener("keydown", callback);
  };
}, [onCloseMovie]);
```

- Nega effect ichida: hodisani `document` ga qo'lda ulash **DOM bilan ishlash**, ya'ni side effect
- Nega cleanup shart: tozalanmasa, `MovieDetails` har ochilganda **yangi listener** qo'shilib, eskilari qolib ketadi. Ular to'planib, handler bir necha marta ishlaydi va xotira ham sarflanadi (memory leak)
- `removeEventListener` ga **xuddi o'sha funksiya** (reference) berilishi kerak, shuning uchun handler nomli funksiya (`callback`) sifatida e'lon qilinadi
- `onCloseMovie` effect ichida ishlatilgani uchun u dependency array'da bo'lishi kerak

## 21. [Strong junior] Currency Converter'da effect qanday tuziladi?

Bitta effect uchta qiymatga bog'liq: `[amount, from, to]`. Ulardan biri o'zgarsa, kurs qayta yuklanadi (sinxronlash):

```jsx
useEffect(function () {
  if (from === to) return setConverted(amount);   // bir xil valyuta: so'rov kerak emas

  async function convert() {
    setIsLoading(true);
    const res = await fetch(`...?amount=${amount}&from=${from}&to=${to}`);
    const data = await res.json();
    setConverted(data.rates[to]);
    setIsLoading(false);
  }
  convert();
}, [amount, from, to]);
```

Qo'shimcha amaliyotlar: so'rov ketayotganda input va select'larni `disabled={isLoading}` qilish (foydalanuvchi ketma-ket o'zgartirib yubormasligi uchun), natijani ko'rsatish, va kerak bo'lsa cleanup bilan eski so'rovni bekor qilish.

## 22. [Middle] Nega bitta component'da bir nechta `useEffect` yozish yaxshi?

Har bir effect **bitta ishga** mas'ul bo'lishi kerak: biri ma'lumot yuklaydi, biri sarlavhani yangilaydi, biri klaviatura listener'ini boshqaradi. Foydalari:

- Har biriga **o'z dependency array'i** va o'z cleanup'i to'g'ri yoziladi
- Kod o'qilishi va o'zgartirilishi osonlashadi (mas'uliyat ajratilgan)
- Bir effect ichida aralash mantiq bo'lsa, ortiqcha qayta ishlashlar paydo bo'ladi (bir qiymat o'zgarganda keraksiz ishlar ham bajariladi)

Mantiq takrorlansa va ko'p joyda kerak bo'lsa, uni **custom hook**ga ajratish mumkin (keyingi section'larda).

## 23. [Middle] Qachon effect **kerak emas**? ("You might not need an effect")

Effect faqat **tashqi tizim bilan sinxronlash** uchun (API, DOM, timer, listener). Quyidagi hollarda effect ishlatish kerak emas:

- **Mavjud state/props'dan hisoblab olinadigan qiymat** (derived state): render ichida hisoblanadi, effect va qo'shimcha state kerak emas
- **Foydalanuvchi harakatiga javob**: event handler ichida bajariladi
- **Props o'zgarganda state'ni reset qilish**: `key` ishlatiladi
- State'ni boshqa state'ga ko'chirish uchun effect yozish: ortiqcha render va xato manbai

Qoida: avval "bu kodni render ichida hisoblab bo'ladimi yoki event handler'ga qo'yib bo'ladimi?" deb so'rang. Effect oxirgi chora.

## 24. [Middle] `useEffect` bilan `fetch` qilishning kamchiliklari qanday? Zamonaviy alternativa nima?

`useEffect` + `fetch` ishlaydi, lekin quyidagilar o'zingiz boshqarishingizga qoladi:

- Har safar **loading, error, data** uchun boilerplate (ko'p takroriy kod)
- **Keshlash yo'q**: bir xil so'rov har safar qaytadan yuboriladi
- **Race condition** va bekor qilishni qo'lda yozish kerak (19-savol)
- So'rovlar bir-birini kutib qolishi (waterfall), avtomatik qayta yuklash va yangilash yo'q

Shuning uchun real loyihalarda server ma'lumoti uchun odatda **React Query (TanStack Query)** yoki **RTK Query** ishlatiladi. Ular keshlash, loading/error holatlari, qayta yuklash va yangilashni o'zi boshqaradi. `useEffect` bilan fetch qilish mexanikani tushunish uchun foydali, lekin ishlab chiqarishda kamdan-kam qo'lda yoziladi.

## 25. [Middle] Development'da effect nega ikki marta ishlaydi?

React 18 Strict Mode'da **development** paytida component'ni ataylab "mount → unmount → qayta mount" qiladi, shuning uchun effect ikki marta ishlaydi. Maqsad: **cleanup funksiyasi yozilmagan yoki noto'g'ri yozilgan** effect'larni topish (masalan, ochiq qolgan listener yoki bekor qilinmagan so'rov). Production'da bunday ikki marta ishlash **yo'q**.

Demak, konsolda ikkita so'rovni ko'rsangiz, bu darhol xato degani emas. Lekin effect'ingiz qayta ishga tushganda xavfsiz bo'lishi, ya'ni to'g'ri cleanup'i bo'lishi kerak ekanini eslatadi.
