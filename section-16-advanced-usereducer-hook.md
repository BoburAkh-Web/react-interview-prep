# Section 16: The Advanced useReducer Hook

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Projects in this section: React Quiz app, Bank Account (challenge).

---

## 1. [Junior] `useReducer` nima? Qanday yoziladi?

`useReducer` bu murakkab state'ni boshqarish uchun `useState` ga alternativ hook. Barcha state o'zgarishlari **bitta reducer funksiyasida** to'planadi.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

- **`state`**: hozirgi state
- **`dispatch`**: o'zgarish so'rovini yuboradigan funksiya (`dispatch({ type: "deposit", payload: 150 })`)
- **`reducer`**: `(state, action) => yangiState` funksiyasi
- **`action`**: nima sodir bo'lganini tasvirlovchi obyekt: odatda `{ type, payload }`

Oqim: komponent `dispatch(action)` chaqiradi → React `reducer(hozirgiState, action)` ni ishga tushiradi → qaytgan qiymat yangi state bo'ladi → component qayta render bo'ladi.

State to'g'ridan-to'g'ri destructure qilinishi mumkin:

```jsx
const [{ balance, loan, isActive }, dispatch] = useReducer(reducer, initialState);
```

## 2. [Junior] `useState` va `useReducer` ni qachon tanlaymiz?

| Holat | Tanlov |
|---|---|
| 1–2 ta **mustaqil** oddiy state (son, string, boolean) | `useState` |
| **Ko'p bog'liq** state'lar bir-biriga ta'sir qiladi | `useReducer` |
| Bitta event bir nechta state'ni birga o'zgartiradi | `useReducer` |
| Keyingi state oldingisiga murakkab tarzda bog'liq | `useReducer` |
| State obyekt/massiv, ko'p turdagi yangilanishlar bor | `useReducer` |
| Kam, oddiy va lokal | `useState` (sodda va qisqa) |

`useReducer` dan foyda: barcha state o'tishlari **bir joyda**, har biri nomlangan action, mantiq component'dan ajratilgan va tushunish oson. Kamchilik: ko'proq kod (boilerplate).

## 3. [Junior] Reducer funksiyasi qanday qoidalarga bo'ysunishi kerak?

Reducer **toza (pure)** bo'lishi shart:

- Faqat `state` va `action` ga tayanadi, boshqa hech narsaga yo'q
- **Side effect yo'q**: API so'rovi, timer, `localStorage`, `Math.random()`, `console.log` (ishlab chiqarishda) bo'lmaydi
- State'ni **mutatsiya qilmaydi**: har doim **yangi obyekt** qaytaradi
- Bir xil state va action berilsa, **doim bir xil natija** qaytaradi

Sabab: React reducer'ni bir necha marta chaqirishi mumkin (Strict Mode'da ikki marta). Toza bo'lmasa, natijalar tasodifiy bo'ladi.

## 4. [Strong junior] Reducer ichida nega state'ni mutatsiya qilmaymiz? "O'zgarmadi" holatini qanday bildiramiz?

React yangi va eski state'ni **havola (reference) bo'yicha** solishtiradi. State'ni joyida o'zgartirsak, havola o'zgarmaydi va React yangilanishni sezmaydi. Shuning uchun spread bilan yangi obyekt yasaymiz:

```jsx
case "deposit":
  return { ...state, balance: state.balance + action.payload };
```

Agar action hech narsani o'zgartirmasligi kerak bo'lsa (masalan, qarz allaqachon olingan), **eski state'ni o'zini** qaytaramiz:

```jsx
case "requestLoan":
  if (state.loan > 0) return state;   // havola bir xil: React keraksiz re-render qilmaydi
```

(Redux Toolkit'da esa Immer tufayli "mutatsiya"ga o'xshash yozish mumkin, lekin oddiy `useReducer` da mumkin emas.)

## 5. [Strong junior] Action'larni qanday nomlaymiz? Nega `payload`?

Action nomi **nima sodir bo'lganini** (hodisani) ifodalaydi, "state'ni qanday o'zgartirish"ni emas: `deposit`, `requestLoan`, `dataReceived`, `newAnswer`, `nextQuestion`, `finish`, `restart`, `tick`. Bunday nomlar kodni o'qishda hodisalar ro'yxati kabi tushunarli.

`payload` action bilan birga yuboriladigan **qo'shimcha ma'lumot** (masalan, miqdor, tanlangan javob). Barcha action'larda bo'lishi shart emas: `payLoan` va `closeAccount` ga ma'lumot kerak emas.

## 6. [Strong junior] "Bog'liq state'lar" (related pieces of state) nimani anglatadi? Misol keltiring

Bir necha state bir-biriga bog'liq bo'lsa va birga yangilanishi kerak bo'lsa, ularni bitta obyektga yig'ib reducer bilan boshqaramiz. Masalan, Counter'da `count` va `step`:

```jsx
const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case "inc":     return { ...state, count: state.count + state.step };
    case "dec":     return { ...state, count: state.count - state.step };
    case "setCount": return { ...state, count: action.payload };
    case "setStep":  return { ...state, step: action.payload };
    case "reset":   return initialState;
    default: throw new Error("Unknown action");
  }
}
```

`useState` bilan bu ko'p alohida setter va handler'larga bo'linardi. Reducer'da barcha o'tishlar ko'z oldida va `reset` kabi murakkab o'zgarish bitta qatorga tushadi.

## 7. [Strong junior] Bank Account reducer'ini tushuntiring. Qanday yaxshilash mumkin?

```jsx
function reducer(state, action) {
  if (!state.isActive && action.type !== "openAccount") return state;   // guard

  switch (action.type) {
    case "openAccount":
      return { ...state, balance: 500, isActive: true };
    case "deposit":
      return { ...state, balance: state.balance + action.payload };
    case "withdraw":
      return { ...state, balance: state.balance - action.payload };
    case "requestLoan":
      if (state.loan > 0) return state;
      return { ...state, loan: action.payload, balance: state.balance + action.payload };
    case "payLoan":
      return { ...state, loan: 0, balance: state.balance - state.loan };
    case "closeAccount":
      if (state.loan > 0 || state.balance !== 0) return state;
      return initialState;
    default:
      throw new Error("Unknown action");
  }
}
```

**Nima yaxshi:**

- Boshidagi **guard clause**: "hisob ochilmagan bo'lsa, hech qaysi amal bajarilmaydi" qoidasi bir joyda
- Har bir case yangi obyekt qaytaradi, shartlar bajarilmasa **eski state**ni qaytaradi
- Hisobni yopish `initialState` ga qaytaradi

**Nimani yaxshilash mumkin:**

1. `openAccount` hisob allaqachon ochiq bo'lganda ham ishlaydi va balansni qayta 500 qiladi. UI'da tugma `disabled` bo'lgani bilan **reducer UI'ga ishonmasligi kerak**: `if (state.isActive) return state;` qo'shish kerak. Biznes qoidalari reducer'da, UI cheklovlari esa faqat qulaylik uchun.
2. `withdraw` balansdan ko'p pul yechishga ruxsat beryapti. Biznes talabiga qarab: `if (action.payload > state.balance) return state;`
3. `"Unkown"` typo (`"Unknown"`), xato xabarida action nomini ko'rsatish ham foydali: `` `Unknown action: ${action.type}` ``

## 8. [Strong junior] Nega `default` holatda xato (`throw new Error`) tashlaymiz?

Noma'lum action (masalan, type'dagi typo) kelganda **jim o'tkazib yuborish** xatoni yashiradi. `throw` esa muammoni **darhol ko'rsatadi**, ya'ni development'da tez topiladi. Alternativa: `return state` (ilova yiqilmaydi, lekin xato yashirin qoladi). Tanlov jamoa uslubiga bog'liq; ko'pincha development'da xato tashlash qulay.

## 9. [Strong junior] Quiz ilovasida `status` state'i nega bitta string, bir nechta boolean emas?

```jsx
const initialState = {
  questions: [],
  status: "loading",   // "loading" | "error" | "ready" | "active" | "finished"
  index: 0,
  answer: null,
  points: 0,
  highscore: 0,
  secondsRemaining: null,
};
```

Alohida `isLoading`, `isError`, `isFinished` booleanlari bo'lsa, **mumkin bo'lmagan kombinatsiyalar** paydo bo'ladi (masalan, bir vaqtda `isLoading` va `isFinished` ikkalasi `true`). Bitta `status` qiymati ilovaning **faqat bitta holatda** bo'lishini kafolatlaydi (state machine g'oyasi) va shartli render soddalashadi:

```jsx
{status === "loading" && <Loader />}
{status === "error" && <Error />}
{status === "ready" && <StartScreen />}
{status === "active" && <Question />}
{status === "finished" && <FinishScreen />}
```

## 10. [Strong junior] Quiz savollarini yuklashda `fetch` nega reducer ichida emas, effect'da?

Reducer **toza** bo'lishi kerak, `fetch` esa side effect. Shuning uchun so'rov `useEffect` ichida yuboriladi, natija esa reducer'ga **action sifatida** yetkaziladi:

```jsx
useEffect(function () {
  fetch("http://localhost:8000/questions")
    .then((res) => res.json())
    .then((data) => dispatch({ type: "dataReceived", payload: data }))
    .catch(() => dispatch({ type: "dataFailed" }));
}, []);
```

```jsx
case "dataReceived": return { ...state, questions: action.payload, status: "ready" };
case "dataFailed":   return { ...state, status: "error" };
```

`dispatch` funksiyasining identifikatori React tomonidan **barqaror** (renderlar orasida o'zgarmaydi), shuning uchun uni dependency array'ga qo'shish zarar qilmaydi va effect qayta ishlamaydi.

## 11. [Strong junior] Reducer ichida joriy savolga qanday kirishamiz? Qaysi qiymatlar derived?

Reducer `state` ning hammasini ko'radi, shuning uchun joriy savolni ichida topish mumkin:

```jsx
case "newAnswer": {
  const question = state.questions.at(state.index);
  return {
    ...state,
    answer: action.payload,
    points: action.payload === question.correctOption
      ? state.points + question.points
      : state.points,
  };
}
case "nextQuestion":
  return { ...state, index: state.index + 1, answer: null };
```

Derived (hisoblanadigan, state'da saqlanmaydigan) qiymatlar:

```jsx
const numQuestions = questions.length;
const maxPossiblePoints = questions.reduce((prev, cur) => prev + cur.points, 0);
```

## 12. [Strong junior] Quiz tugaganda va qayta boshlaganda reducer qanday ishlaydi?

```jsx
case "finish":
  return {
    ...state,
    status: "finished",
    highscore: state.points > state.highscore ? state.points : state.highscore,
  };

case "restart":
  return { ...initialState, questions: state.questions, status: "ready" };
```

`restart` da `initialState` ga qaytamiz, lekin **allaqachon yuklangan savollarni saqlab qolamiz** (qayta fetch qilmaslik uchun) va holatni darrov `"ready"` qilamiz.

## 13. [Strong junior] Taymerni `useEffect` va `dispatch` bilan qanday yasaymiz?

```jsx
// Timer component
useEffect(function () {
  const id = setInterval(function () {
    dispatch({ type: "tick" });
  }, 1000);

  return function () {
    clearInterval(id);   // cleanup
  };
}, [dispatch]);
```

```jsx
case "tick":
  return {
    ...state,
    secondsRemaining: state.secondsRemaining - 1,
    status: state.secondsRemaining === 0 ? "finished" : state.status,
  };
```

Foydasi: "vaqt tugaganda quizni yakunlash" mantig'i **reducer'da**, ya'ni state'ning qolgan qoidalari bilan birga turadi. Cleanup shart, aks holda interval component yo'qolgandan keyin ham ishlayveradi (Section 12 dagi cleanup mavzusi).

## 14. [Middle] Reducer'ning qanday afzalliklari bor? Test va masshtablash

- **Test qilish oson**: reducer oddiy toza funksiya, component yoki React kerak emas: `expect(reducer(state, action)).toEqual(newState)`
- **Bashorat qilinadigan**: barcha o'zgarishlar action nomlari bilan ko'rinadi (log, debug, "time travel" uchun qulay)
- **Mantiq ajratilgan**: UI va biznes qoidalari aralashmaydi
- **Context bilan birga**: `dispatch`ni Context orqali berib, prop drilling'siz global-ga yaqin state yaratish mumkin (keyingi section'lardagi mavzu)

Kamchilik: oddiy state uchun ortiqcha kod, shuning uchun hamma narsani reducer qilish shart emas.

## 15. [Middle] `useReducer` va Redux orasidagi farq nima?

Tushunchalar bir xil: **reducer, action, dispatch**. Farqlar:

| | `useReducer` | Redux (Toolkit) |
|---|---|---|
| Doirasi | Bitta component yoki Context orqali ulashilgan | Butun ilova uchun **global store** |
| Qo'shimchalar | Yo'q | Middleware, DevTools, Redux Toolkit (`createSlice`, Immer) |
| O'rnatish | React ichida | Alohida kutubxona |
| Mantiq | Oddiy, kichik | Katta ilova, ko'p slice'lar |

`useReducer` kichik va o'rta murakkablikdagi state uchun yetarli. Ilova kattalashib, ko'p joyda umumiy state kerak bo'lganda (yoki ilg'or vositalar kerak bo'lsa) Redux Toolkit yoki boshqa global state yechimi tanlanadi.
