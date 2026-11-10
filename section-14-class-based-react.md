# Section 14: [Optional] React Before Hooks: Class-Based React

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: Classy Weather (ob-havo ilovasi, class component'lar bilan).

> Bu section **optional** va yangi loyihalarda class component'lar deyarli yozilmaydi. Lekin eski kod bazalarida uchraydi va intervyuda "class va function component farqi", "lifecycle metodlar" kabi savollar beriladi.

---

## 1. [Junior] Class component qanday yoziladi?

```jsx
import { Component } from "react";

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  render() {
    return <p>{this.state.count}</p>;
  }
}
```

- `extends Component` orqali React imkoniyatlari olinadi
- `constructor` ichida **`super(props)`** majburiy (aks holda `this` ishlamaydi)
- State `this.state` obyektida, props `this.props` orqali olinadi
- Nima ko'rsatilishi **`render()`** metodida qaytariladi

## 2. [Junior] Class component'da state qanday yangilanadi? `useState` dan farqi nima?

State faqat **`this.setState`** orqali yangilanadi. `this.state.count = 5` deb to'g'ridan-to'g'ri yozib bo'lmaydi (React o'zgarishni sezmaydi, xuddi function component'dagi kabi).

```jsx
this.setState({ count: 5 });                                  // oddiy
this.setState((curState) => ({ count: curState.count + 1 }));  // hozirgi state asosida (callback)
```

Muhim farq: `setState` obyektni **birlashtiradi (shallow merge)**: faqat berilgan maydonlar yangilanadi, qolganlari saqlanadi. `useState` setter esa qiymatni **to'liq almashtiradi** (obyekt state bo'lsa, qolgan maydonlarni qo'lda spread bilan saqlash kerak).

## 3. [Strong junior] Class'da event handler ichida `this` nega `undefined` bo'ladi? Qanday tuzatamiz?

```jsx
class Counter extends Component {
  handleClick() {
    this.setState(...);   // xato: this === undefined
  }
  render() {
    return <button onClick={this.handleClick}>+</button>;
  }
}
```

Handler tugmaga berilganda u **class'dan ajralib qoladi** va chaqirilganda `this` ma'lumoti yo'qoladi. Tuzatish usullari:

1. **Constructor'da bind**: `this.handleClick = this.handleClick.bind(this);`
2. **Arrow function** (JSX'da): `onClick={() => this.handleClick()}`
3. **Class field + arrow function** (eng qulay, 4-savol)

Function component'larda `this` umuman yo'q, shuning uchun bu muammo ham yo'q.

## 4. [Junior] Class fields nima? Boilerplate'ni qanday kamaytiradi?

Class ichida `constructor` va `bind`siz to'g'ridan-to'g'ri maydon e'lon qilish:

```jsx
class Counter extends Component {
  state = { count: 0 };                        // constructor va super kerak emas

  handleClick = () => {                        // arrow function: this avtomatik to'g'ri
    this.setState((s) => ({ count: s.count + 1 }));
  };

  render() {
    return <button onClick={this.handleClick}>{this.state.count}</button>;
  }
}
```

Arrow function class'ning `this` ini avtomatik "eslab qoladi", shuning uchun `bind` shart emas.

## 5. [Strong junior] Class component'ning lifecycle metodlari qanday?

- **`componentDidMount()`**: component birinchi marta ekranga chiqqandan keyin bir marta ishlaydi (odatda ma'lumot yuklash)
- **`componentDidUpdate(prevProps, prevState)`**: har yangilanishdan keyin ishlaydi (birinchi renderdan tashqari)
- **`componentWillUnmount()`**: component olib tashlanishidan oldin (cleanup)

```jsx
componentDidMount() {
  this.fetchWeather();
}

componentDidUpdate(prevProps, prevState) {
  if (this.state.location !== prevState.location) {   // shart MAJBURIY
    this.fetchWeather();
  }
}

componentWillUnmount() {
  // listener, timer'larni tozalash
}
```

`componentDidUpdate` ichida **doim eski va yangi qiymatni solishtirish** kerak. Aks holda har yangilanish yana `setState` chaqirsa, **cheksiz tsikl** bo'ladi.

## 6. [Strong junior] `useEffect` lifecycle metodlariga qanday mos keladi? Qaysi biri yaxshi?

| Class | Function component |
|---|---|
| `componentDidMount` | `useEffect(fn, [])` |
| `componentDidUpdate` (solishtirish bilan) | `useEffect(fn, [dep])` |
| `componentWillUnmount` | `useEffect` ning cleanup funksiyasi |

Lekin bu mutlaqo bir xil emas. `useEffect` "lifecycle" emas, **sinxronlash** haqida: bitta effect mount, update va unmount holatlarini birga boshqaradi. Class'da bir mantiq uch metodga **bo'linib ketadi**, aloqasiz mantiqlar esa bitta metodda **aralashib qoladi**. Hook'larda bir maqsadga oid kod bir joyda to'planadi.

## 7. [Strong junior] Nega zamonaviy React'da function component + hook'lar class'larning o'rnini bosdi?

| | Class | Function + hooks |
|---|---|---|
| Kod hajmi | Ko'p boilerplate (`constructor`, `super`, `bind`) | Kam |
| `this` | Chalkashlik manbai | Yo'q |
| Mantiqni qayta ishlatish | HOC va render props (murakkab) | **Custom hook**lar (oddiy) |
| Bog'liq kod | Lifecycle metodlari orasida bo'linib ketadi | Bir effect'da to'planadi |
| O'rganish va o'qish | Qiyinroq | Osonroq |

React jamoasi yangi funksiyalarni (masalan, concurrent imkoniyatlar) hook'lar atrofida rivojlantiradi. Class'lar hamon ishlaydi, lekin yangi kodda tavsiya etilmaydi.

## 8. [Middle] Class component'larda mantiq qanday qayta ishlatilgan? Nega bu qiyin edi?

Ikki yo'l bilan: **Higher-Order Component (HOC)**, ya'ni component'ni boshqa component bilan o'rab beradigan funksiya, va **render props**, ya'ni funksiyani props sifatida berish. Muammolari: component'lar ichma-ich o'rala berib "wrapper hell" hosil qiladi, props nomlari to'qnashadi, kodni o'qish va debug qilish qiyinlashadi. **Custom hook**lar buni oddiy funksiya chaqiruviga aylantirdi.

## 9. [Middle] Qaysi holatda class component hali ham kerak?

**Error Boundary**lar. Hozircha React'da xatolarni ushlaydigan (`componentDidCatch` / `static getDerivedStateFromError`) maxsus hook yo'q, shuning uchun Error Boundary class component bilan yoziladi. Amalda ko'pchilik tayyor kutubxonadan (masalan, `react-error-boundary`) foydalanadi, u ichida class'ni yashiradi. Bundan tashqari, class component'lar eski loyihalarni qo'llab-quvvatlashda uchraydi.

## 10. [Junior] Class component'da child-to-parent aloqa qanday amalga oshadi?

Function component'dagidek: parent funksiyani **props** sifatida beradi, child uni `this.props` orqali chaqiradi.

```jsx
// Parent
<Input location={location} onChangeLocation={handleChangeLocation} />

// Child (class)
<input onChange={(e) => this.props.onChangeLocation(e.target.value)} />
```

Props bu yerda ham faqat pastga oqadi va child uni o'zgartirmaydi.
