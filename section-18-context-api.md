# Section 18: Advanced State Management: The Context API

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Projects in this section: The Atomic Blog (prop drilling mashqi), WorldWise (`CitiesContext`, xarita, fake auth), React Quiz'ni Context'ga ko'chirish.

> WorldWise'ning haqiqiy kodi bo'yicha savollar (arxitektura, `CitiesContext`, `AuthContext`, `ProtectedRoute`, Leaflet) `projects/worldwise-interview.md` faylida. Bu fayl **nazariya** va umumiy pattern'lar uchun.

---

## 1. [Junior] Context API nima va qanday muammoni hal qiladi?

Context ma'lumotni **props orqali har bir qatlamdan o'tkazmasdan**, daraxtning istalgan chuqurligidagi component'ga to'g'ridan-to'g'ri yetkazish mexanizmi.

Hal qiladigan muammo: **prop drilling**, ya'ni ma'lumotni faqat pastdagi component ishlatadi, lekin u oraliqdagi barcha component'lar orqali props sifatida uzatiladi (Atomic Blog'da `posts`, `onAddPost`, `searchQuery` kabilar shunday uzatilgan).

Uch qism: **context yaratish** (`createContext`), **Provider** (qiymatni beradi) va **consumer** (`useContext` bilan o'qiydi).

## 2. [Junior] Context qanday yaratiladi, beriladi va o'qiladi?

```jsx
import { createContext, useContext } from "react";

const PostContext = createContext();                  // 1. yaratish

function App() {
  const [posts, setPosts] = useState([]);
  return (
    <PostContext.Provider value={{ posts, setPosts }}>   {/* 2. berish */}
      <Header />
      <Main />
    </PostContext.Provider>
  );
}

function Results() {
  const { posts } = useContext(PostContext);          // 3. o'qish
  return <p>{posts.length} posts</p>;
}
```

`Provider` ichidagi **istalgan** child (qanchalik chuqur bo'lmasin) `useContext` bilan qiymatni oladi. Eslatma: React 19 da `<PostContext.Provider>` o'rniga to'g'ridan-to'g'ri `<PostContext value={...}>` yozish mumkin.

## 3. [Junior] Provider'ning `value` si o'zgarsa nima bo'ladi?

`value` o'zgarganda (havola bo'yicha) shu context'ni `useContext` bilan o'qiyotgan **barcha component'lar qayta render** bo'ladi, ular qiymatning faqat bir qismini ishlatsa ham. Context'ni o'qimaydigan component'lar bundan ta'sirlanmaydi. Shuning uchun `value` ichida nima turishi va u qancha tez o'zgarishi performance uchun muhim (12-savolga qarang).

## 4. [Junior] `createContext(defaultValue)` dagi default qiymat nima uchun?

Default qiymat faqat component **hech qanday Provider ichida bo'lmaganda** ishlatiladi. Amalda ko'pincha uni berilmaydi (`createContext()` → `undefined`) va custom hook ichida tekshiriladi (5-savol), chunki Provider'siz ishlatish odatda xato.

## 5. [Strong junior] Custom provider va custom hook pattern nima? Nega kerak?

Uch narsani birlashtiramiz: **context**, **state'ni saqlaydigan Provider component** va **`use...` hook**.

```jsx
const PostContext = createContext();

function PostProvider({ children }) {
  const [posts, setPosts] = useState(() => createInitialPosts());
  const [searchQuery, setSearchQuery] = useState("");

  const searchedPosts = searchQuery.length > 0
    ? posts.filter((p) => `${p.title} ${p.body}`.toLowerCase().includes(searchQuery.toLowerCase()))
    : posts;                                           // derived state

  function handleAddPost(post) { setPosts((posts) => [post, ...posts]); }
  function handleClearPosts()  { setPosts([]); }

  return (
    <PostContext.Provider
      value={{ posts: searchedPosts, onAddPost: handleAddPost,
               onClearPosts: handleClearPosts, searchQuery, setSearchQuery }}
    >
      {children}
    </PostContext.Provider>
  );
}

function usePosts() {
  const context = useContext(PostContext);
  if (context === undefined)
    throw new Error("PostContext was used outside of the PostProvider");
  return context;
}
```

Foydasi:

- **App toza qoladi**: state va mantiq Provider'da, `App` faqat `<PostProvider>` ni render qiladi
- **Inkapsulyatsiya**: component'lar `PostContext` ni emas, faqat `usePosts()` ni import qiladi
- **Xavfsizlik**: Provider'siz ishlatilsa, tushunarli xato chiqadi (`undefined` bilan boshqa joyda yiqilish o'rniga)

## 6. [Strong junior] Context o'zi "state management" emasmi? Uning vazifasi nima?

Yo'q. Context state'ni **saqlamaydi** va boshqarmaydi, u faqat **yetkazish (transport)** mexanizmi. State haqiqatda `useState` yoki `useReducer` da turadi (odatda Provider ichida), Context esa uni daraxt bo'ylab hamma uchun ochib beradi. Shuning uchun "Context + `useReducer`" juftligi tez-tez qo'llaniladi.

## 7. [Strong junior] State'ni global qilish (Context) kerakmi yoki yo'qmi, qanday hal qilamiz?

("Thinking in React: Advanced State Management") Savollar ketma-ketligi:

1. State bitta component uchun kerakmi? → **local state** (`useState`)
2. Bir nechta yaqin component'ga kerakmi? → **lift state up** yoki **composition** (`children`)
3. Ko'p va uzoq component'larga kerakmi? → **Context** (yoki global state kutubxonasi)
4. Bu **server ma'lumoti** (API'dan keladi)mi? → **React Query** kabi vosita yaxshiroq

Context ham, URL ham "global" bo'la oladi: tanlangan element `id`si yoki koordinata kabilar URL'da, UI/auth kabilar Context'da saqlanadi.

## 8. [Strong junior] Context + `useReducer` kombinatsiyasi qanday tuziladi? (CitiesContext)

Provider ichida `useReducer` state'ni boshqaradi, Provider **state va yordamchi funksiyalarni** context orqali beradi:

```jsx
function CitiesProvider({ children }) {
  const [{ cities, isLoading, currentCity, error }, dispatch] =
    useReducer(reducer, initialState);

  useEffect(function () {
    async function fetchCities() {
      dispatch({ type: "loading" });
      try {
        const res = await fetch(`${BASE_URL}/cities`);
        const data = await res.json();
        dispatch({ type: "cities/loaded", payload: data });
      } catch {
        dispatch({ type: "rejected", payload: "There was an error loading cities..." });
      }
    }
    fetchCities();
  }, []);

  // getCity, createCity, deleteCity: async funksiyalar shu yerda,
  // natijaga qarab dispatch qiladi

  return (
    <CitiesContext.Provider
      value={{ cities, isLoading, currentCity, error, getCity, createCity, deleteCity }}
    >
      {children}
    </CitiesContext.Provider>
  );
}
```

Muhim: **API chaqiruvlari (side effect) reducer ichida emas**, provider'dagi funksiyalarda. Reducer toza (pure) bo'lib, faqat action'ni qabul qilib yangi state qaytaradi.

## 9. [Strong junior] Fake authentication qanday ishlaydi? Haqiqiy auth'dan farqi nima?

`AuthContext` `user` va `isAuth` state'ini hamda `login` va `logout` funksiyalarini beradi. Login'da kiritilgan email va parol kod ichidagi **soxta foydalanuvchi** bilan solishtiriladi, mos kelsa, `user` state'ga yoziladi.

Haqiqiy auth'dan farqi:

- Tekshiruv **serverda** bo'lishi kerak (client'dagi parol bundle'ga tushadi va ko'rinib qoladi)
- Server **token** (JWT) yoki **httpOnly cookie** qaytaradi
- Sessiya **saqlanadi** (sahifa yangilanganda logout bo'lmaydi)
- Token muddati, qayta yangilash va xatolar boshqariladi

Fake auth'ning foydasi: Context'ning tashqi API'si (`user`, `isAuth`, `login`, `logout`) haqiqiy auth'ga o'tganda **o'zgarmaydi**, faqat `login` ichidagi mantiq almashadi.

## 10. [Strong junior] `ProtectedRoute` (himoyalangan route) qanday tuziladi?

```jsx
function ProtectedRoute({ children }) {
  const { isAuth } = useAuth();
  return isAuth ? children : <Navigate to="/login" replace />;
}

<Route path="app" element={<ProtectedRoute><AppLayout /></ProtectedRoute>}>
```

Wrapper component `isAuth` ni tekshiradi: login qilingan bo'lsa, `children` ni ko'rsatadi, aks holda login sahifasiga yo'naltiradi (`replace` bilan, Back tugmasi tuzoqqa tushmasligi uchun).

**Muhim ogohlantirish:** bu faqat **UX himoyasi**, xavfsizlik emas. Haqiqiy himoya serverda: API har bir so'rovda foydalanuvchini tekshirishi kerak, chunki client kodini foydalanuvchi istalgancha o'zgartira oladi.

## 11. [Strong junior] Uchinchi tomon imperativ kutubxonani (Leaflet) React bilan qanday birlashtiramiz?

Leaflet o'zi imperativ (`map.setView(...)`, `marker.addTo(map)`). `react-leaflet` uni **deklarativ component'lar** ko'rinishida o'raydi (`MapContainer`, `Marker`, `Popup`). Qoidalar:

- **React state'i haqiqat manbai** bo'lib qoladi (shaharlar Context'da, xarita markazi state yoki URL'da)
- Imperativ amallar (`setView`) `useMap()` orqali ichki kichik component'larda, **effect'da** bajariladi
- Hodisalar (`click`) `useMapEvents` bilan ushlanadi va React state/URL'ga yoziladi
- Xarita markazi `MapContainer`'ning `center` prop'i bilan **faqat birinchi mount**da beriladi, keyingi o'zgarishlar uchun `useMap().setView` kerak

## 12. [Middle] Context performance muammosi nimada? Qanday hal qilinadi?

`value` obyekti Provider har render bo'lganda **yangi** bo'ladi, shuning uchun uni o'qiyotgan barcha component'lar qayta render bo'ladi. Yo'llar:

- `value` ni **`useMemo`** ga o'rash, funksiyalarni **`useCallback`** ga o'rash
- Context'ni **ikkiga bo'lish**: tez-tez o'zgaradigan va kam o'zgaradigan qismlar alohida (masalan, state va dispatch)
- **Composition**: Provider `children` qabul qilganda, uning state'i o'zgarsa ham `children` qayta render bo'lmaydi (element havolasi o'sha-o'sha)
- Juda tez o'zgaradigan state uchun Context o'rniga selector'li kutubxona (Zustand, Redux Toolkit)

Batafsil Section 19 da (Optimizing Context re-renders).

## 13. [Middle] Context qachon yaxshi tanlov emas?

- **Tez-tez o'zgaradigan** ma'lumot (sichqoncha pozitsiyasi, har tugma bosilishi): har o'zgarishda barcha consumer'lar render bo'ladi
- **Server ma'lumoti** (keshlash, qayta yuklash, loading/error kerak): React Query kabi vosita aniqroq
- **Bitta-ikkita component** orasidagi oddiy ma'lumot: props yoki composition yetarli
- **Juda ko'p va murakkab** global state: maxsus kutubxona (Redux Toolkit, Zustand) kuchliroq (selector, devtools, middleware)

## 14. [Middle] Context vs Redux/Zustand: asosiy farq nima?

| | Context + `useReducer` | Redux Toolkit / Zustand |
|---|---|---|
| Qo'shimcha kutubxona | Yo'q (React ichida) | Bor |
| Re-render nazorati | Barcha consumer'lar render bo'ladi | **Selector** bilan faqat kerakli qism |
| DevTools, middleware | Yo'q | Bor |
| Kod hajmi | Kichik loyihada qisqa | Ko'proq boilerplate (Redux), kam (Zustand) |
| Tavsiya | Kichik va o'rta, kam o'zgaradigan global state | Katta ilova, tez-tez o'zgaradigan state |

## 15. [Middle] React Quiz'ni Context'ga ko'chirish nimani o'zgartiradi? Kamchiligi bormi?

`App` dagi `useReducer` va hisoblangan qiymatlar (`numQuestions`, `maxPossiblePoints`) `QuizProvider` ga ko'chadi, component'lar esa `useQuiz()` bilan o'qiydi. Natija: `state` va `dispatch` ni har bir component'ga props bilan uzatish yo'qoladi, `App` qisqaradi.

Kamchilik: component'lar endi **Context'ga bog'liq**, ya'ni Provider'siz ishlamaydi va boshqa loyihada qayta ishlatish qiyinlashadi (props orqali ishlaydigan "toza" component'lar universal edi). Shuning uchun Context'ga **hammasini** ko'chirmaslik, faqat chindan ko'p joyda kerak bo'lganini ko'chirish ma'qul.
