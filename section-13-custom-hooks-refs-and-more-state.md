# Section 13: Custom Hooks, Refs, and More State

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: usePopcorn (davomi): `useMovies`, `useLocalStorageState`, `useKey` custom hook'lari va `useGeolocate` challenge.

---

## 1. [Junior] Hook nima? Qaysi hook'larni bilasiz?

Hook bu function component'larga React imkoniyatlariga (state, lifecycle, ref, context) **"ulanish"** imkonini beradigan maxsus funksiya. Nomi har doim `use` bilan boshlanadi.

Asosiylari: `useState`, `useEffect`, `useRef`, `useReducer`, `useContext`, `useMemo`, `useCallback`. Shuningdek, o'zimiz yozadigan **custom hook**lar ham bor.

## 2. [Junior] Hook'larning qoidalari qanday?

1. Hook'larni faqat **eng yuqori darajada** (top level) chaqirish: shart (`if`), sikl, ichki funksiya yoki `return` dan keyin **chaqirib bo'lmaydi**
2. Hook'larni faqat **React function component** yoki **custom hook** ichida chaqirish (oddiy funksiyada yoki class'da emas)

Sabab: React hook'larni **chaqirilish tartibi** bo'yicha eslab qoladi (Section 6 dagi `useState` savolini eslang). Har renderda tartib bir xil bo'lmasa, React qaysi state qaysi hook'ga tegishli ekanini adashtiradi.

## 3. [Strong junior] Qoidalar buzilishiga amaliy misollar va ularni qanday tuzatamiz?

```jsx
// XATO 1: early return'dan keyin hook
function Movie({ id }) {
  if (!id) return <p>No movie</p>;
  const [movie, setMovie] = useState(null);   // ba'zi renderlarda chaqirilmaydi!
  ...
}

// XATO 2: shart ichida hook
if (isOpen) {
  useEffect(() => { ... }, []);
}
```

Tuzatish: hook'larni **doim yuqorida** chaqirib, shartni **hook ichiga** yoki hook'lardan **keyin** qo'yish:

```jsx
function Movie({ id }) {
  const [movie, setMovie] = useState(null);

  useEffect(function () {
    if (!id) return;          // shart effect ICHIDA
    ...
  }, [id]);

  if (!id) return <p>No movie</p>;   // early return hook'lardan KEYIN
}
```

Bunday xatolarni `eslint-plugin-react-hooks` ning `rules-of-hooks` qoidasi avtomatik topadi.

## 4. [Strong junior] `useState` ning boshlang'ich qiymati haqida nimani bilish kerak?

Boshlang'ich qiymat **faqat birinchi renderda** ishlatiladi, keyingi renderlarda React saqlangan state'ni beradi. Lekin quyidagi yozuvda tuzoq bor:

```jsx
const [watched, setWatched] = useState(getStoredWatched());
```

`getStoredWatched()` **har renderda chaqiriladi** (natijasi e'tiborga olinmasa ham). Agar funksiya og'ir bo'lsa (masalan, `localStorage` o'qish va `JSON.parse`), bu ortiqcha ish. Yechim: lazy initial state (5-savol).

## 5. [Strong junior] Lazy initial state (callback bilan boshlang'ich qiymat) nima?

`useState` ga qiymat o'rniga **funksiya** bersak, React uni **faqat birinchi renderda** bir marta chaqiradi:

```jsx
const [watched, setWatched] = useState(function () {
  const stored = localStorage.getItem("watched");
  return stored ? JSON.parse(stored) : [];
});
```

Farqi:

```jsx
useState(getStored())        // har renderda getStored() chaqiriladi
useState(() => getStored())  // faqat birinchi renderda chaqiriladi (lazy)
```

Funksiya **toza (pure)** bo'lishi va **argument olmasligi** kerak. Odatda `localStorage` dan o'qish yoki og'ir hisob-kitoblar uchun ishlatiladi.

## 6. [Junior] Nega DOM elementni `document.querySelector` bilan tanlamaymiz?

```jsx
useEffect(function () {
  document.querySelector(".search").focus();   // React usuli EMAS
}, []);
```

Sabablar:

- Bu **imperativ** yondashuv, React esa deklarativ
- Class/selector bo'yicha tanlash mo'rt: nomi o'zgarsa yoki sahifada bir xil class'li bir nechta element (masalan, component'ning bir necha nusxasi) bo'lsa, noto'g'ri elementni oladi
- Element render paytida hali DOM'da bo'lmasligi mumkin

React'da DOM elementga murojaat qilishning to'g'ri yo'li: **`useRef`**.

## 7. [Junior] `useRef` nima?

`useRef` bitta o'zgaruvchan (mutable) obyekt qaytaradi: `{ current: boshlangichQiymat }`. U **renderlar orasida saqlanadi**, lekin `current` ni o'zgartirish **re-render keltirib chiqarmaydi**. Ikki asosiy vazifasi:

1. **DOM elementga havola** olish
2. Renderlar orasida **ma'lumotni saqlash** (UI'da ko'rsatilmaydigan, o'zgarganda render kerak bo'lmaydigan)

## 8. [Strong junior] Ref bilan DOM elementni qanday tanlaymiz? Nega buni effect ichida qilamiz?

```jsx
function Search({ query, setQuery }) {
  const inputEl = useRef(null);

  useEffect(function () {
    inputEl.current.focus();   // mount'da input'ga fokus
  }, []);

  return <input ref={inputEl} value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

`ref={inputEl}` DOM element yaratilgach, uni `inputEl.current` ga joylaydi. Bu **render tugagach (commit'dan keyin)** sodir bo'ladi, shuning uchun render paytida `current` hali `null`. Shu sababli DOM bilan ishlash **effect** (yoki event handler) ichida bajariladi.

## 9. [Strong junior] Ref va state farqi nima? Qachon qaysi birini ishlatamiz?

| | **State** | **Ref** |
|---|---|---|
| O'zgarganda re-render | **Ha** | **Yo'q** |
| Renderlar orasida saqlanadi | Ha | Ha |
| O'zgartirish usuli | setter funksiya (immutable) | `ref.current = ...` (mutatsiya) |
| UI'da ko'rsatiladi | Ha | Odatda yo'q |

- UI'da ko'rinishi kerak bo'lgan, o'zgarganda ekran yangilanishi kerak bo'lgan ma'lumot: **state**
- Ekranda ko'rinmaydigan, faqat "eslab qolish" kerak bo'lgan ma'lumot (timer id, hisoblagich, oldingi qiymat) va DOM havolalari: **ref**

**Muhim qoida:** `ref.current` ni **render paytida o'qimang va yozmang**, faqat **effect va event handler** ichida ishlating. Render toza (pure) bo'lishi kerak (Section 11), ref esa mutatsiya qilinadigan narsa.

## 10. [Strong junior] Ref bilan ma'lumotni renderlar orasida saqlashga misol keltiring

Foydalanuvchi bahoni tanlagunicha necha marta fikrini o'zgartirganini hisoblash:

```jsx
const countRef = useRef(0);

useEffect(function () {
  if (userRating) countRef.current = countRef.current + 1;
}, [userRating]);

// saqlashda: countRatingDecisions: countRef.current
```

Nega ref:

- Oddiy `let count = 0` bo'lsa, **har renderda qayta 0** bo'lib qoladi
- State bo'lsa, har o'zgarishda **keraksiz re-render** bo'lardi, bu qiymat esa ekranda ko'rsatilmaydi

Bu ref'ning "renderlar orasida yashaydigan, lekin render'ga ta'sir qilmaydigan o'zgaruvchi" vazifasi.

## 11. [Junior] Custom hook nima va qachon yaratiladi?

Custom hook: nomi `use` bilan boshlanadigan va **ichida bir yoki bir nechta hook ishlatadigan** funksiya. Maqsad: **UI bo'lmagan mantiqni** (state + effect va h.k.) qayta ishlatish.

- Component'lar **UI**ni qayta ishlatadi
- Custom hook'lar **mantiq**ni qayta ishlatadi

Qachon yaratamiz: bir xil mantiq bir necha component'da takrorlansa, yoki component juda uzun va murakkab bo'lib ketsa (mantiqni ajratib olish uchun). Nomi `use` bilan boshlanishi shart, chunki shunda React va ESLint uni hook deb tanib, hook qoidalarini tekshiradi.

## 12. [Strong junior] `useMovies` custom hook'i qanday yoziladi?

Component ichidagi qidiruv va ma'lumot yuklash mantig'i alohida hook'ga ko'chiriladi:

```jsx
// useMovies.js
export function useMovies(query) {
  const [movies, setMovies] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState("");

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

    if (query.length < 3) {
      setMovies([]);
      setError("");
      return;
    }
    fetchMovies();
    return function () { controller.abort(); };
  }, [query]);

  return { movies, isLoading, error };
}

// App.js
const { movies, isLoading, error } = useMovies(query);
```

`App` qisqardi va faqat "nima kerak"ni biladi, "qanday olinadi"ni emas. Hook **obyekt** qaytaryapti, chunki qiymatlar ko'p. (Konvensiya: 2-3 ta qiymat bo'lsa massiv, ko'p bo'lsa obyekt.)

## 13. [Strong junior] `useLocalStorageState` hook'i qanday yoziladi? Nega massiv qaytaradi?

`useState` ga o'xshash ishlaydigan, lekin qiymatni `localStorage` da saqlaydigan hook:

```jsx
export function useLocalStorageState(initialState, key) {
  const [value, setValue] = useState(function () {
    const storedValue = localStorage.getItem(key);
    return storedValue ? JSON.parse(storedValue) : initialState;   // lazy init
  });

  useEffect(function () {
    localStorage.setItem(key, JSON.stringify(value));
  }, [value, key]);

  return [value, setValue];
}

// ishlatilishi:
const [watched, setWatched] = useLocalStorageState([], "watched");
```

Nega massiv: chaqirilgan joyda qiymat nomlarini **o'zimiz tanlashimiz** mumkin (`[watched, setWatched]`), huddi `useState` kabi. Obyekt bo'lsa, nomlarni qayta nomlash kerak bo'lardi.

Effect `[value, key]` ga bog'liq, shuning uchun qiymat o'zgargan har qanday joyda (qo'shish ham, o'chirish ham) avtomatik saqlanadi, alohida handler yozish shart emas.

## 14. [Strong junior] `useKey` hook'i va uning foydasi nima?

Klaviatura tugmasini tinglash mantig'i (Section 12 dagi Escape misoli) bir necha joyda kerak bo'ladi (Escape: tafsilotni yopish, Enter: qidiruvga fokus). Uni bitta hook'ga yig'ib, qayta ishlatamiz:

```jsx
export function useKey(key, action) {
  useEffect(function () {
    function callback(e) {
      if (e.code.toLowerCase() === key.toLowerCase()) action();
    }
    document.addEventListener("keydown", callback);
    return function () {
      document.removeEventListener("keydown", callback);
    };
  }, [action, key]);
}

// ishlatilishi:
useKey("Escape", onCloseMovie);
```

Foyda: effect, listener qo'shish va **cleanup** bir joyda, takrorlanmaydi. `action` dependency'da bo'lgani uchun u har renderda yangi funksiya bo'lsa, effect qayta o'rnatiladi (buni `useCallback` bilan yaxshilash mumkin, keyingi section'larda).

## 15. [Strong junior] `useGeolocate` challenge: hook faqat qiymat emas, funksiya ham qaytara oladimi?

Ha. Brauzer Geolocation API'si foydalanuvchi **tugma bosganda** chaqiriladi, shuning uchun hook `getPosition` funksiyasini qaytaradi:

```jsx
function useGeolocate() {
  const [isLoading, setIsLoading] = useState(false);
  const [position, setPosition] = useState({});
  const [error, setError] = useState(null);

  function getPosition() {
    if (!navigator.geolocation)
      return setError("Your browser does not support geolocation");

    setIsLoading(true);
    navigator.geolocation.getCurrentPosition(
      (pos) => {
        setPosition({ lat: pos.coords.latitude, lng: pos.coords.longitude });
        setIsLoading(false);
      },
      (err) => {
        setError(err.message);
        setIsLoading(false);
      }
    );
  }

  return { isLoading, position, error, getPosition };
}
```

Muhim nuqtalar: foydalanuvchi harakatiga javob bo'lgani uchun bu **event handler** mantiqi (effect emas); brauzer foydalanuvchidan ruxsat so'raydi, shuning uchun xato holatini ham boshqarish kerak; `navigator.geolocation` yo'q brauzerlarni tekshirish.

## 16. [Middle] Ikki component bir xil custom hook'ni chaqirsa, ular state'ni ulashadimi?

**Yo'q.** Custom hook mantiqni ulashadi, **state'ni emas**. Har bir chaqiruv o'zining alohida state'iga ega:

```jsx
function A() { const [v] = useLocalStorageState([], "x"); }   // o'z state'i
function B() { const [v] = useLocalStorageState([], "x"); }   // boshqa, mustaqil state
```

Ikkala component'ning state'i sinxron bo'lishi kerak bo'lsa, state'ni umumiy parent'ga ko'tarish, Context yoki global state ishlatiladi.

## 17. [Middle] Yaxshi custom hook'ni qanday loyihalaymiz? Qachon ajratmaslik kerak?

Yaxshi hook:

- **Bitta aniq vazifa** bajaradi (`useMovies`, `useKey`), nomi nima qilishini ko'rsatadi
- Kerakli narsani **argument** sifatida oladi (`query`, `key`), tashqi muhitga yashirin bog'liq bo'lmaydi
- Effect'lari **cleanup**ga ega (listener, so'rov, timer)
- Qaytish shakli o'ylangan: ko'p qiymat uchun obyekt, `useState` kabi 2 ta uchun massiv

Ajratmaslik kerak: mantiq faqat bir joyda ishlatilsa va oddiy bo'lsa, erta abstraksiya kodni murakkablashtiradi. "Bir marta yozdim, ikkinchi marta takrorladim, uchinchida ajratdim" qoidasi yaxshi yo'l-yo'riq.

## 18. [Middle] Ref'ni yana qanday holatlarda ishlatamiz? Qanday xatolardan qochish kerak?

Ref ishlatiladigan tipik joylar:

- `setTimeout`/`setInterval` **id**sini saqlash (keyin tozalash uchun)
- **Oldingi qiymatni** eslab qolish
- Input'ga fokus berish, scroll qilish, o'lchamni o'qish kabi DOM operatsiyalari

```jsx
const timerRef = useRef(null);

function start() { timerRef.current = setInterval(tick, 1000); }
function stop()  { clearInterval(timerRef.current); }
```

Xatolar:

- UI'da ko'rinishi kerak bo'lgan qiymatni ref'da saqlash (ref o'zgarsa, ekran yangilanmaydi)
- `ref.current` ni **render paytida** o'qish yoki yozish (render toza bo'lishi kerak)
- Ref'ni state o'rnini bosuvchi "qulay yo'l" sifatida ishlatish: bu ma'lumot va UI'ning mosligini buzadi
