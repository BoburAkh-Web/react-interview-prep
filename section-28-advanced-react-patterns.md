# Section 28: Advanced React Patterns

Levels: **[Junior]**, **[Strong junior]**, **[Middle]**.

Project: The Wild Oasis (4-qism: Modal, ConfirmDelete, Table, Menus) va alohida mashq misoli (ro'yxat component'i).

> Bu section intervyuda **Middle** darajasiga eng yaqin mavzular manbai: "qayta ishlatiladigan component'ni qanday loyihalaysiz?" savoliga javob shu yerda.

---

## 1. [Junior] React'da nimani va qanday qayta ishlatish mumkin?

Ikki xil narsa qayta ishlatiladi:

| Nima | Qanday |
|---|---|
| **UI** (ko'rinish) | Component'lar va props; `children` (composition) |
| **Stateful mantiq** (hook'lar bilan) | **Custom hook**lar |

Qolgan murakkab holatlar uchun **ilg'or pattern'lar**:

- **Render props**: component **nimani render qilishini** tashqaridan boshqarish
- **Compound components**: bir-biriga bog'liq component'lar to'plami bitta "super-component" sifatida ishlaydi
- **Higher-Order Component (HOC)**: asosan eski kodda uchraydi

## 2. [Strong junior] Render props pattern nima?

Component nimani chizishini o'zi bilmaydi: tashqaridan **render qiluvchi funksiya** prop sifatida beriladi, component esa uni o'z ma'lumoti bilan chaqiradi.

```jsx
function List({ title, items, render }) {
  const [isOpen, setIsOpen] = useState(true);
  const [isCollapsed, setIsCollapsed] = useState(false);
  const displayItems = isCollapsed ? items.slice(0, 3) : items;

  return (
    <div>
      <h2>{title}</h2>
      {isOpen && <ul>{displayItems.map(render)}</ul>}
    </div>
  );
}

<List title="Products" items={products}
      render={(product) => <ProductItem key={product.productName} product={product} />} />
<List title="Companies" items={companies}
      render={(company) => <CompanyItem key={company.companyName} company={company} />} />
```

`List` ochish/yopish va qisqartirish **mantiqini** saqlaydi, lekin elementni **qanday ko'rsatishni** ishlatuvchi hal qiladi. Natija: bitta `List` mahsulotlar, kompaniyalar va boshqa istalgan narsa uchun ishlaydi.

## 3. [Strong junior] Render props hozir ham kerakmi? Custom hook bilan farqi nima?

Ko'p holatda **custom hook** render props'ning o'rnini bosdi: mantiqni hook'ga chiqarib, UI'ni component o'zi yozadi. Lekin render props hamon foydali bo'ladigan holat: component **o'z ichida state va mantiqni saqlab**, faqat **bir qism UI'ni** tashqaridan qabul qilishi kerak bo'lganda (masalan, umumiy jadvalda har bir qatorni qanday chizish). Wild Oasis'dagi `Table.Body` aynan shunday ishlaydi (11-savol).

## 4. [Strong junior] Higher-Order Component (HOC) nima? Nega hozir kam ishlatiladi?

HOC: **component qabul qilib, yangi (kengaytirilgan) component qaytaradigan funksiya**. Odatda nomi `with` bilan boshlanadi.

```jsx
function withToggles(WrappedComponent) {
  return function List(props) {
    const [isOpen, setIsOpen] = useState(true);
    const [isCollapsed, setIsCollapsed] = useState(false);
    const displayItems = isCollapsed ? props.items.slice(0, 3) : props.items;

    return (
      <div>
        <WrappedComponent {...props} items={displayItems} />
        <button onClick={() => setIsCollapsed((c) => !c)}>{isCollapsed ? "Show all" : "Show less"}</button>
      </div>
    );
  };
}

const ProductListWithToggles = withToggles(ProductList);
```

Foyda: **o'zgartirib bo'lmaydigan** component'ga (masalan, uchinchi tomon kutubxonasidan) qo'shimcha imkoniyat qo'shish. Kamchiliklari: ichma-ich o'rash ("wrapper hell"), props nomlari to'qnashuvi, ma'lumot qayerdan kelayotgani noaniq. Hook'lar paydo bo'lgach HOC kam yoziladi, lekin eski kodda (Redux `connect`, React Router v5 `withRouter`) hamon uchraydi.

## 5. [Strong junior] Compound component pattern nima?

Bir-biriga bog'liq component'lar **bitta parent atrofida birga ishlaydi**: parent umumiy state'ni **Context** orqali ushlab turadi, child component'lar esa uni o'qiydi. HTML'dagi `<select>` va `<option>` g'oyasiga o'xshaydi.

```jsx
const CounterContext = createContext();

function Counter({ children }) {
  const [count, setCount] = useState(0);
  const increase = () => setCount((c) => c + 1);
  const decrease = () => setCount((c) => c - 1);
  return (
    <CounterContext.Provider value={{ count, increase, decrease }}>
      <span>{children}</span>
    </CounterContext.Provider>
  );
}

function Count()    { const { count } = useContext(CounterContext); return <span>{count}</span>; }
function Increase({ icon }) { const { increase } = useContext(CounterContext); return <button onClick={increase}>{icon}</button>; }
function Decrease({ icon }) { const { decrease } = useContext(CounterContext); return <button onClick={decrease}>{icon}</button>; }

Counter.Count = Count;
Counter.Increase = Increase;
Counter.Decrease = Decrease;

// Ishlatish: tartibni va ko'rinishni ishlatuvchi tanlaydi
<Counter>
  <Counter.Decrease icon="−" />
  <Counter.Count />
  <Counter.Increase icon="+" />
</Counter>
```

## 6. [Strong junior] Compound component'ning afzalligi nima? Oddiy props'li component'dan farqi?

Oddiy component'da har bir variant uchun yangi prop qo'shiladi: `<Counter showLabel hideIncrease decreaseIcon="−" order="reverse" ... />`. Bu **"props explosion"** (props ko'payib ketishi) va component'ni o'zgartirish qiyinlashadi.

Compound component'da:

- **Ishlatuvchi tuzilishni boshqaradi**: qaysi qism, qaysi tartibda, qanday atrofida
- **Ichki state yashirin**, lekin barcha qismlarga avtomatik ulashiladi (prop drilling yo'q)
- **Kengaytirish oson**: yangi qism (masalan, `Counter.Label`) qo'shish boshqalarga tegmaydi
- Kod o'qilganda **deklarativ** va tushunarli

Haqiqiy kutubxonalarda keng uchraydi: Radix UI, shadcn/ui (`Dialog`, `DialogTrigger`, `DialogContent`), Headless UI.

## 7. [Strong junior] React Portal nima? Modal uchun nega kerak?

**Portal** component'ni React daraxtidagi joyini saqlagan holda, DOM'ning **boshqa joyiga** (odatda `document.body`) render qiladi.

```jsx
import { createPortal } from "react-dom";

function Modal({ children, onClose }) {
  return createPortal(
    <Overlay>
      <StyledModal>
        <Button onClick={onClose}><HiXMark /></Button>
        <div>{children}</div>
      </StyledModal>
    </Overlay>,
    document.body
  );
}
```

Nega kerak: modal ota-element ichida qolsa, ota-elementning **`overflow: hidden`**, **`z-index`** yoki **`transform`** kabi CSS xususiyatlari uni kesib qo'yishi yoki ostida qoldirishi mumkin. Portal modalni DOM tepasiga chiqarib, bu muammolardan qutqaradi va component'ni har qanday joyda qayta ishlatish imkonini beradi.

## 8. [Middle] Portal ichidagi event'lar qanday tarqaladi (bubbling)?

DOM'da modal `document.body` ichida bo'lsa ham, React event'lari **React daraxti** bo'yicha tarqaladi: portal ichidagi click React daraxtidagi **parent component**larga ham yetib boradi. Props va Context ham React daraxti bo'yicha ishlaydi, shuning uchun modal ota-component'ning Context'ini o'qiy oladi. Bu ba'zan kutilmagan holatlarga olib keladi (masalan, tashqi elementdagi `onClick` modal ichidagi click'da ham ishlab ketishi), kerak bo'lsa `e.stopPropagation()` ishlatiladi.

## 9. [Strong junior] Modal'ni compound component'ga qanday aylantirdik?

Muammo: modalni ochish uchun har joyda `const [isOpen, setIsOpen] = useState(false)` va props yozish kerak edi. Endi modal **o'z ochiq/yopiq holatini o'zi saqlaydi** va bir sahifada **bir nechta oyna** bo'lishi mumkin:

```jsx
const ModalContext = createContext();

function Modal({ children }) {
  const [openName, setOpenName] = useState("");
  const close = () => setOpenName("");
  const open = setOpenName;
  return <ModalContext.Provider value={{ openName, close, open }}>{children}</ModalContext.Provider>;
}

function Open({ children, opens: opensWindowName }) {
  const { open } = useContext(ModalContext);
  return cloneElement(children, { onClick: () => open(opensWindowName) });
}

function Window({ children, name }) {
  const { openName, close } = useContext(ModalContext);
  const ref = useOutsideClick(close);
  if (name !== openName) return null;

  return createPortal(
    <Overlay>
      <StyledModal ref={ref}>
        <Button onClick={close}><HiXMark /></Button>
        <div>{cloneElement(children, { onCloseModal: close })}</div>
      </StyledModal>
    </Overlay>,
    document.body
  );
}

Modal.Open = Open;
Modal.Window = Window;
```

```jsx
<Modal>
  <Modal.Open opens="cabin-form"><Button>Add new cabin</Button></Modal.Open>
  <Modal.Window name="cabin-form"><CreateCabinForm /></Modal.Window>
</Modal>
```

`opens` va `name` orqali tugma va oyna bog'lanadi. Oyna ichidagi forma `onCloseModal` prop'ini oladi va muvaffaqiyatli saqlangach modalni o'zi yopadi.

## 10. [Middle] `cloneElement` nima? Uni ishlatishning kamchiligi bormi?

`cloneElement(element, newProps)` mavjud React elementining **nusxasini yangi props bilan** yaratadi. Modal'da u tashqaridan berilgan tugmaga `onClick`, oyna ichidagi formaga esa `onCloseModal` qo'shish uchun ishlatilgan.

Kamchiliklari: React hujjatlarida **kamdan-kam ishlatiladigan (legacy) API** sifatida belgilangan, chunki:

- Ma'lumot oqimi **noaniq** bo'ladi (props qayerdan kelgani ko'rinmaydi)
- Child bitta element bo'lishi shart, child o'z props'ini (`onClick`) **ustidan yozib yuborish** mumkin
- TypeScript bilan tiplash qiyin

Alternativalar: **render props** (`<Modal.Open>{(open) => <Button onClick={open}>...</Button>}</Modal.Open>`), Context'dan to'g'ridan-to'g'ri o'qish yoki `asChild` pattern (Radix UI).

## 11. [Strong junior] Modal tashqarisiga bosilganda yopish (click outside) qanday yasaldi? Nega capturing phase?

```js
export function useOutsideClick(handler, listenCapturing = true) {
  const ref = useRef();

  useEffect(function () {
    function handleClick(e) {
      if (ref.current && !ref.current.contains(e.target)) handler();
    }
    document.addEventListener("click", handleClick, listenCapturing);
    return () => document.removeEventListener("click", handleClick, listenCapturing);
  }, [handler, listenCapturing]);

  return ref;
}
```

- `ref` modal oynasiga beriladi, click **oynaning ichida bo'lmasa** (`!ref.current.contains(e.target)`), `handler` (yopish) chaqiriladi
- Listener effect ichida qo'shiladi va **cleanup**da olib tashlanadi
- **Capturing phase** (`true`): "Add new cabin" tugmasi bosilganda modal ochiladi, lekin o'sha click **bubbling** orqali `document` ga yetib borganda yangi ochilgan modal ichida emas (tugma modal tashqarisida), shuning uchun modal **darhol yopilib** qolardi. Capturing phase'da listener event pastga tushayotganda ishlaydi, ya'ni modal hali ochilmagan paytda, shuning uchun bu muammo yo'qoladi

## 12. [Junior] O'chirishni tasdiqlash (`ConfirmDelete`) qanday qilingan?

Modal compound component'i qayta ishlatiladi: o'chirish tugmasi `Modal.Open`, tasdiqlash oynasi esa `Modal.Window` ichida:

```jsx
<Modal.Open opens="delete"><Button>Delete</Button></Modal.Open>
<Modal.Window name="delete">
  <ConfirmDelete resourceName="cabins" disabled={isDeleting} onConfirm={() => deleteCabin(cabinId)} />
</Modal.Window>
```

`ConfirmDelete` umumiy (resurs nomi, tasdiqlash funksiyasi va `onCloseModal` props orqali), shuning uchun kabina, bron va boshqa resurslarni o'chirishda bir xil ishlatiladi. Foydalanuvchi qaytarib bo'lmaydigan amalni tasodifan bajarmasligi uchun muhim UX qoidasi.

## 13. [Strong junior] Qayta ishlatiladigan `Table` qanday qurilgan? (Compound + render props)

```jsx
const TableContext = createContext();

function Table({ columns, children }) {
  return (
    <TableContext.Provider value={{ columns }}>
      <StyledTable role="table">{children}</StyledTable>
    </TableContext.Provider>
  );
}

function Header({ children }) {
  const { columns } = useContext(TableContext);
  return <StyledHeader role="row" columns={columns} as="header">{children}</StyledHeader>;
}

function Row({ children }) {
  const { columns } = useContext(TableContext);
  return <StyledRow role="row" columns={columns}>{children}</StyledRow>;
}

function Body({ data, render }) {
  if (!data.length) return <Empty>No data to show at the moment</Empty>;
  return <StyledBody>{data.map(render)}</StyledBody>;
}

Table.Header = Header;
Table.Body = Body;
Table.Row = Row;
```

```jsx
<Table columns="0.6fr 1.8fr 2.2fr 1fr 1fr 1fr">
  <Table.Header>
    <div></div><div>Cabin</div><div>Capacity</div><div>Price</div><div>Discount</div><div></div>
  </Table.Header>
  <Table.Body data={cabins} render={(cabin) => <CabinRow cabin={cabin} key={cabin.id} />} />
</Table>
```

- **Compound**: `columns` (CSS Grid ustunlari) Context orqali Header va Row'ga avtomatik beriladi
- **Render props**: `Table.Body` ma'lumotni aylanib chiqadi, har bir qatorni qanday chizish esa `render` funksiyasida
- Jadval div va CSS Grid'da qurilgan, shuning uchun accessibility uchun **`role="table"`**, **`role="row"`** qo'shilgan
- Bo'sh ma'lumot holati bir joyda boshqariladi

## 14. [Middle] Qayta ishlatiladigan context menu (`Menus`) qanday ishlaydi? Asosiy qiyinchiliklar nima?

Har bir jadval qatorida "⋮" tugmasi bor, bosilganda Duplicate/Edit/Delete menyusi chiqadi. Talablar: **bir vaqtda faqat bitta menyu ochiq**, menyu tugma **yonida** chiqadi va **tashqariga bosilganda yopiladi**.

```jsx
const MenusContext = createContext();

function Menus({ children }) {
  const [openId, setOpenId] = useState("");
  const [position, setPosition] = useState(null);
  const close = () => setOpenId("");
  const open = setOpenId;
  return (
    <MenusContext.Provider value={{ openId, close, open, position, setPosition }}>
      {children}
    </MenusContext.Provider>
  );
}

function Toggle({ id }) {
  const { openId, close, open, setPosition } = useContext(MenusContext);
  function handleClick(e) {
    e.stopPropagation();
    const rect = e.target.closest("button").getBoundingClientRect();
    setPosition({ x: window.innerWidth - rect.width - rect.x, y: rect.y + rect.height + 8 });
    openId === "" || openId !== id ? open(id) : close();
  }
  return <StyledToggle onClick={handleClick}><HiEllipsisVertical /></StyledToggle>;
}

function List({ id, children }) {
  const { openId, position, close } = useContext(MenusContext);
  const ref = useOutsideClick(close, false);
  if (openId !== id) return null;
  return createPortal(<StyledList position={position} ref={ref}>{children}</StyledList>, document.body);
}

function Button({ children, icon, onClick }) {
  const { close } = useContext(MenusContext);
  function handleClick() { onClick?.(); close(); }
  return <li><StyledButton onClick={handleClick}>{icon}<span>{children}</span></StyledButton></li>;
}

Menus.Menu = Menu; Menus.Toggle = Toggle; Menus.List = List; Menus.Button = Button;
```

Asosiy qarorlar:

- **Bitta `openId`** butun jadval uchun Context'da, shuning uchun ikkinchi menyu ochilsa, birinchisi avtomatik yopiladi
- **Pozitsiya** tugmaning ekrandagi joyidan (`getBoundingClientRect`) hisoblanadi va menyu portal orqali `body` ga `position: fixed` bilan chiqariladi
- Toggle'da **`e.stopPropagation()`** va `useOutsideClick(close, false)` (bubbling): tugmani qayta bosish menyuni ochib-yopish sikliga tushib qolmasligi uchun
- Menyu tugmalari bosilgach menyu **o'zi yopiladi**

Kamchiliklari: sahifa scroll qilinganda pozitsiya yangilanmaydi, ekran chetida menyu sig'may qolishi mumkin, klaviatura bilan boshqarish (Escape, strelkalar, focus) qo'lda qilinmagan. Real loyihalarda shu sababli **Floating UI** yoki **Radix UI** kabi kutubxonalar ishlatiladi.

## 15. [Middle] Modal va Menus'ni birga ishlatganda qanday muammo chiqadi?

Kabina qatorida menyu ichidagi "Edit" va "Delete" tugmalari **modal ochadi**. Shuning uchun `Modal` butun qatorni o'rab turadi, `Menus.Button` esa `Modal.Open` ichiga joylashtiriladi:

```jsx
<Modal>
  <Menus.Menu>
    <Menus.Toggle id={cabinId} />
    <Menus.List id={cabinId}>
      <Menus.Button icon={<HiSquare2Stack />} onClick={handleDuplicate}>Duplicate</Menus.Button>
      <Modal.Open opens="edit"><Menus.Button icon={<HiPencil />}>Edit</Menus.Button></Modal.Open>
      <Modal.Open opens="delete"><Menus.Button icon={<HiTrash />}>Delete</Menus.Button></Modal.Open>
    </Menus.List>
    <Modal.Window name="edit"><CreateCabinForm cabinToEdit={cabin} /></Modal.Window>
    <Modal.Window name="delete"><ConfirmDelete ... /></Modal.Window>
  </Menus.Menu>
</Modal>
```

Bu yerda `Modal.Open` `cloneElement` orqali `Menus.Button` ga `onClick` beradi, `Menus.Button` esa uni chaqirib, keyin menyuni yopadi. Ikki compound component bir-birining props'iga bog'lanib qoladi: ishlaydi, lekin tushunish murakkab. Bu `cloneElement` ning kamchiliklarini (10-savol) yaxshi ko'rsatadi.

## 16. [Middle] Qachon qaysi pattern'ni tanlaysiz?

| Vaziyat | Pattern |
|---|---|
| Faqat mantiq takrorlanadi | **Custom hook** |
| Faqat UI takrorlanadi, kontent har xil | **`children`** (composition) |
| Component o'z mantig'ini saqlab, bir qism UI'ni tashqaridan oladi | **Render props** |
| Bir-biriga bog'liq bir necha qism, tuzilishi ishlatuvchiga bog'liq (Modal, Tabs, Table, Menu) | **Compound component** |
| O'zgartirib bo'lmaydigan component'ga imkoniyat qo'shish (asosan eski kod) | **HOC** |
| Ota-elementning CSS cheklovlaridan chiqish | **Portal** |

Umumiy tamoyil: eng sodda yechimdan boshlash. Pattern faqat kod takrorlanishi yoki props ko'payib ketishi aniq muammo bo'lganda qo'llaniladi.
