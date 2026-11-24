TypeScript intervyu savollari: Sapphire loyihasi asosida
Nov 24, 2026 · @Jigar
Loyihada nimalarni o'rgandik
Sapphire loyihasida hozirgacha 20 dan ortiq TypeScript tushunchasini real kodda ishlatdik. Har bir savolga javob shu kodlardan olingan misol bilan berilgan, shuning uchun intervyuda "buni o'z loyihamda qildim" deb aytsa bo'ladi.
Tushuncha
Loyihadagi joyi
type, obyekt tipi, export
types/ papkasi: Truck, HeroData, NavLink
? ixtiyoriy maydon
Button va Container props'idagi className?
Union | va literal tiplar
pricingType: "perMile" | "percent", Button variant, Theme
Intersection &
IconProps = LucideProps & { name: string }
Massiv Truck[]
TrucksData.trucks, NavLink[]
Tuple va as const
useLocalStorageState qaytaradigan [value, setValue]
Funksiya tipi, void
toggleTheme: () => void, setActiveValue: (value: string) => void
Qaytish tipi : boolean
useScrolled(offset = 0): boolean
Type inference
useState(false), offset = 0
Generics <T>
useLocalStorageState<T>, createContext<...>, useState<T>
Record<K, V>
heroImages, icons xaritalari
| null va type narrowing
useTheme, useTabsContext
as (type assertion)
data/index.ts: trucksJson as TrucksData
! (non-null assertion)
main.tsx: getElementById("root")!
ReactNode, props tiplash
children: ReactNode
Rest props ...props
Icon komponenti
import type, verbatimModuleSyntax
barcha tip importlari
JSON'ni tiplash
resolveJsonModule, data/index.ts
1. Asosiy tiplar va type alias
1.1. TypeScript nima va nega kerak?
TypeScript bu JavaScript ustiga qo'shilgan tiplar tizimi. U kodni ishga tushirishdan oldin xatolarni topadi. Brauzer TypeScript'ni bilmaydi: build paytida barcha tiplar o'chiriladi va oddiy JavaScript qoladi.
const price: number = "10"; // ❌ xato: matn songa mos emas
JavaScript'da bu xato faqat sayt ishlaganda chiqardi, TypeScript esa uni yozayotgan paytda ko'rsatadi.
1.2. Asosiy primitive tiplar qaysilar?
• string: matn, masalan "Sprinter Van"
• number: son (butun ham, kasr ham), masalan 0.8
• boolean: true yoki false
• null: "qiymat yo'q" (ataylab)
• undefined: "qiymat berilmagan"
JavaScript'da butun son va kasr son uchun alohida tip yo'q, ikkalasi ham number.
1.3. type nima?
type obyekt (yoki boshqa qiymat) qanday shaklda bo'lishini tasvirlaydi va unga nom beradi. Bunga type alias deyiladi.
type Truck = {
  id: number;
  name: string;
  from: number;
};

const van: Truck = { id: 1, name: "Sprinter Van", from: 0.8 }; // ✅
const bad: Truck = { id: "1", name: "Van", from: 0.8 };       // ❌ id son bo'lishi kerak
Tip nomi doim katta harf bilan yoziladi (PascalCase): Truck, HeroData. Kichik harf (navigation) odatda o'zgaruvchi nomi bo'ladi.
1.4. Nega tip nomi birlikda (Truck), ko'plikda (Trucks) emas?
Tip bitta narsani tasvirlaydi. Ro'yxat kerak bo'lsa, [] qo'shiladi: Truck[]. Agar tipni Trucks desak, Trucks[] "ko'pliklar ro'yxati" bo'lib, chalkash o'qiladi.
1.5. Obyekt ichidagi obyekt qanday tiplanadi?
Ikki usul bor. Ichida yozish:
type HeroData = {
  title: {
    start: string;
    highlight: string;
    end: string;
  };
};
Yoki alohida tipga ajratish (o'qish osonroq, qayta ishlatsa bo'ladi):
type HeroTitle = { start: string; highlight: string; end: string };
type HeroData = { title: HeroTitle };
Alohida tip export qilinsa, nomi aniq bo'lishi kerak: Title emas, HeroTitle. Aks holda boshqa fayldagi Title bilan aralashadi.
1.6. ? (ixtiyoriy maydon) nima?
? "bu maydon berilmasa ham bo'ladi" degani. Uning tipi avtomatik ravishda string | undefined bo'ladi.
type ButtonProps = {
  children: ReactNode;  // majburiy
  className?: string;   // ixtiyoriy
};

<Button>Ok</Button>                      // ✅ className yo'q, xato emas
? nomdan keyin, ikki nuqtadan oldin yoziladi: className?: string. className: ?string noto'g'ri.
1.7. Ixtiyoriy prop'ga standart qiymat qanday beriladi?
Props'ni olayotgan joyda = bilan:
function Button({ variant = "primary", className = "" }: ButtonProps) {}
Bu bo'lmasa, className undefined bo'lib qoladi va klass matniga "undefined" so'zi yozilib ketadi.
1.8. Type inference nima?
TypeScript tipni qiymatdan o'zi aniqlaydi. Har joyda tip yozish shart emas.
const count = 5;                       // number
const [isScrolled] = useState(false);  // boolean
function useScrolled(offset = 0) {}    // offset: number
Qoida: tip aniq ko'rinib tursa, yozma. Faqat TypeScript noto'g'ri yoki juda keng tip chiqaradigan joyda yoz (masalan, useLocalStorageState<Theme>, 6-bo'limga qara).
1.9. type va interface farqi nima?
Ikkalasi ham obyekt shaklini tasvirlaydi va ko'p holatda bir xil ishlaydi.
type Truck = { id: number };
interface Truck { id: number }
Farqlari:
• type union ("a" | "b"), tuple va boshqa har qanday tipni ham ifodalay oladi, interface faqat obyekt shaklini.
• interface ni extends bilan kengaytirish mumkin, type da buning o'rniga & ishlatiladi.
• Bir xil nomli ikki interface avtomatik birlashadi (declaration merging), type da bu xato.
Bu loyihada hammasi type bilan yozilgan, chunki union'lar ko'p va bitta uslubda bo'lishi qulay. Intervyuda "jamoa qaysi birini tanlagan bo'lsa, shunga amal qilaman" deyish yaxshi javob.
1.10. export nima uchun kerak?
export qilinmagan tip faqat o'sha fayl ichida ko'rinadi. Boshqa faylda ishlatish uchun export type ... deb yoziladi va kerakli joyda import qilinadi. Faqat ichki ishlatiladigan tiplar (masalan, ThemeContextValue) ataylab export qilinmaydi.
2. Union | va literal tiplar
2.1. Union type nima?
Union | belgisi bilan yoziladi va "yo u, yo bu" degani. Qiymat sanab o'tilgan variantlardan biri bo'lishi kerak.
let id: string | number;
id = "ftl"; // ✅
id = 5;     // ✅
id = true;  // ❌ boolean ro'yxatda yo'q
2.2. Literal tip nima?
Oddiy string har qanday matnni qabul qiladi. Literal tip esa faqat bitta aniq qiymatni qabul qiladi.
let a: string = "dark";   // har qanday matn
let b: "dark" = "dark";   // faqat "dark"
b = "light";              // ❌
O'zi alohida kam ishlatiladi. Uning kuchi union bilan birga ko'rinadi.
2.3. Union va literalni birga qayerda ishlatding?
Uchta joyda:
pricingType: "perMile" | "percent";       // trucks.ts
variant?: "primary" | "accent" | "ghost";  // Button.tsx
type Theme = "dark" | "light";             // ThemeContext.tsx
Bu "faqat shu qiymatlardan biri" degani. variant="green" deb yozsang, kod ishga tushmasdan qizil chiziq chiqadi.
2.4. Nega pricingType: string emas, union yozding?
Eski saytda Semi Truck narxi "10%" degan matn edi. Kod uni hisoblay olmadi va kalkulyator bo'sh chiqdi. Union qo'shib, ikki narsaga erishdik:
• kod mashina narxi dollar/milya yoki foiz ekanini aniq biladi;
• "perMle" kabi yozuv xatosini TypeScript darhol ushlaydi.
string bo'lsa, "banana" ham qabul qilinardi va xato faqat ishlatayotganda chiqardi.
2.5. Union obyekt kalitlarini qanday himoya qiladi?
const variantClasses = {
  primary: "bg-brand-blue",
  accent: "bg-brand-red",
};

variantClasses[variant]; // variant: "primary" | "accent" | "ghost"
Union'ga "ghost" qo'shib, obyektga qo'shmasang, TypeScript xato beradi: "ghost kaliti obyektda yo'q". Ya'ni yangi variantning stilini unutib qo'yishning iloji yo'q.
2.6. Nima uchun JSON import qilinganda union xato berdi?
TypeScript JSON fayldan tipni ehtiyotkorlik bilan chiqaradi. "pricingType": "perMile" ni ko'rib, uni string deb hisoblaydi, "perMile" literal deb emas. Chunki JSON'da ertaga boshqa qiymat bo'lmasligiga kafolat yo'q.
string esa "perMile" | "percent" dan kengroq, shuning uchun mos kelmaydi:
Type 'string' is not assignable to type '"perMile" | "percent"'.
Yechim as (7-bo'limga qara). Bu savol yaxshi, chunki u "keng tip torroq tipga avtomatik o'tmaydi" degan qoidani ko'rsatadi.
2.7. TypeScript xato xabarini qanday o'qiysan?
Uzun xatolarni pastdan yuqoriga o'qiyman. Eng pastki qatorda haqiqiy sabab yozilgan, yuqoridagilar esa unga qanday yetib kelinganini ko'rsatadi (obyekt → trucks → bitta mashina → pricingType).
3. Intersection &
3.1. Intersection type nima?
& ikki (yoki ko'proq) tipni birlashtiradi. Natijaviy tipda ikkala tipning hamma maydonlari bo'ladi.
type HasId = { id: number };
type HasName = { name: string };

type Truck = HasId & HasName;
// { id: number; name: string }

const t: Truck = { id: 1, name: "Van" }; // ✅ ikkalasi ham shart
3.2. Loyihada qayerda ishlatding?
Icon komponentida:
type IconProps = LucideProps & {
  name: string;
};
LucideProps lucide ikonkasi qabul qiladigan hamma prop'lar (className, size, strokeWidth...). Unga o'zimizning name prop'imizni qo'shdik. Natijada Icon ham name ni, ham lucide'ning barcha prop'larini qabul qiladi:
<Icon name="truck" className="size-6" strokeWidth={1.5} />
Bu usulning foydasi: lucide'ning 10 ta prop'ini qo'lda qayta yozish kerak emas.
3.3. | va & farqi nima?
Eng ko'p so'raladigan savollardan biri. Oddiy qoida:

A | B (union)
A & B (intersection)
Ma'nosi
yo A, yo B
ham A, ham B
Obyektlarda
ikkalasidan bittasining shakli yetadi
ikkalasining hamma maydonlari shart
Loyihadagi misol
"dark" | "light"
LucideProps & { name: string }
Tuzoq: nomlar teskari tuyuladi. & "kesishma" deb nomlanadi, lekin obyektda maydonlar ko'payadi. Sababi: A & B qiymati bir vaqtda ham A ning, ham B ning talablariga javob berishi kerak, shuning uchun unda hammasi bo'lishi shart.
3.4. & mos kelmaydigan tiplarda nima bo'ladi?
type X = string & number; // never
Hech bir qiymat bir vaqtda ham matn, ham son bo'la olmaydi. Natija never, ya'ni "bunday qiymat mavjud emas". & amalda obyekt tiplarini birlashtirish uchun ishlatiladi.
3.5. & va interface extends farqi?
Ikkalasi ham tipni kengaytiradi:
type IconProps = LucideProps & { name: string };
interface IconProps extends LucideProps { name: string }
Natija deyarli bir xil. type bilan ishlasang &, interface bilan ishlasang extends ishlatiladi.
4. Massivlar, tuple va as const
4.1. Massiv tipi qanday yoziladi?
Element tipidan keyin [] qo'yiladi:
const names: string[] = ["FTL", "LTL"];
const trucks: Truck[] = [van, semi];
Truck[] "Truck lar ro'yxati" degani: har bir element Truck shaklida bo'lishi shart. Boshqa yozuv ham bor, Array<Truck>, ma'nosi bir xil.
4.2. JSON faylning o'zi massiv bo'lsa, qanday tiplading?
navigation.json to'g'ridan-to'g'ri [ ... ] bilan boshlanadi. Shuning uchun alohida "butun fayl" tipi yozmadim, faqat bitta element tipini yozdim va [] qo'shdim:
export type NavLink = { label: string; href: string };

export const navigationData = navigationJson as NavLink[];
Boshqa JSON'lar obyekt bo'lgani uchun ularga butun fayl tipi berilgan (TrucksData, HeroData).
4.3. .map() ichida elementga tip yozish kerakmi?
Yo'q. Massiv tipi ma'lum bo'lsa, TypeScript har bir elementning tipini o'zi biladi:
navigationData.map((link) => link.label); // link: NavLink
link.lable deb xato yozsang, darhol qizil chiziq chiqadi. Bu type inference'ning yana bir misoli.
4.4. Tuple nima?
Tuple bu uzunligi va har bir o'rnidagi tipi aniq bo'lgan massiv.
const pair: [string, number] = ["Van", 0.8];
// 0-o'rin doim string, 1-o'rin doim number
Oddiy massivdan farqi:

Oddiy massiv (string | number)[]
Tuple [string, number]
Uzunligi
istalgan
aniq 2 ta
arr[0] tipi
string | number
string
useState ham tuple qaytaradi: [qiymat, setter].
4.5. as const nima qiladi?
TypeScript'ga "bu qiymat o'zgarmaydi, uni eng aniq tip bilan tushun" deydi.
const a = ["dark", 5];           // (string | number)[]
const b = ["dark", 5] as const;  // readonly ["dark", 5]
Ikkinchi holatda TypeScript uzunligini, tartibini va hatto aniq qiymatlarini biladi. readonly esa uni o'zgartirib bo'lmasligini bildiradi (b.push(...) xato).
4.6. Loyihada as const ni qayerda va nega ishlatding?
useLocalStorageState hookining oxirida:
return [value, setValue] as const;
as const bo'lmasa, TypeScript buni "aralash massiv" deb tushunadi: har bir element yo qiymat, yo funksiya. Natijada:
const [theme, setTheme] = useLocalStorageState("theme", "dark");
setTheme("light"); // ❌ setTheme funksiya bo'lmasligi ham mumkin
as const bilan u tuple bo'ladi: 1-element qiymat, 2-element setter. useState ham aynan shunday ishlaydi.
4.7. Custom hook ikki narsa qaytarsa, massiv yaxshimi yoki obyekt?
• Massiv (tuple): ishlatganda nomni o'zing tanlaysan: const [theme, setTheme] = .... useState uslubiga o'xshaydi. Lekin as const kerak.
• Obyekt: nomlar qat'iy (const { value, setValue } = ...), lekin as const kerak emas va 3 tadan ko'p qiymatda o'qish osonroq.
Ikki qiymat uchun massiv, ko'p qiymat uchun obyekt odatiy tanlov.
5. Funksiya tiplari
5.1. Funksiya parametrlariga tip qanday beriladi?
Har parametr nomidan keyin : va tip:
function getHeroImage(name: string) {}
function add(a: number, b: number) {}
Standart qiymat berilsa, tip yozish shart emas, TypeScript uni qiymatdan chiqaradi: offset = 0 → number.
5.2. Funksiya qaytaradigan tip qanday yoziladi?
Qavslardan keyin, : bilan:
function useScrolled(offset = 0): boolean {
  return window.scrollY > offset;
}

function getHeroImage(name: string): string { ... }
Endi funksiya boolean dan boshqa narsa qaytarsa, TypeScript xato beradi. Qaytish tipini ham TypeScript o'zi chiqara oladi. Lekin uni yozib qo'ysang, funksiyaning "va'dasi" aniq bo'ladi va ichida xato qaytarilsa, darhol ko'rinadi.
5.3. void nima?
"Funksiya hech narsa qaytarmaydi" degani. U faqat ish bajaradi.
function toggleTheme(): void {
  setTheme(...); // faqat state'ni o'zgartiradi, natija bermaydi
}
void va undefined deyarli bir xil ko'rinadi, lekin void "qaytgan qiymat bilan qiziqmaymiz" degan ma'noni beradi.
5.4. Obyekt ichidagi funksiyaning tipi qanday yoziladi?
Strelka shaklida: (parametrlar) => qaytish tipi.
type ThemeContextValue = {
  theme: Theme;
  toggleTheme: () => void;
};

type TabsContextValue = {
  activeValue: string;
  setActiveValue: (value: string) => void;
};
O'qilishi:
• () => void: argument olmaydi, hech narsa qaytarmaydi.
• (value: string) => void: bitta string oladi, hech narsa qaytarmaydi.
• (a: number) => number: son oladi, son qaytaradi.
Bu yerdagi => arrow funksiya emas, tip yozuvi. Ichidagi value nomi faqat hujjat uchun, chaqirayotganda istalgan nom berilsa bo'ladi.
5.5. setActiveValue uchun (value: string) => void yozding, lekin useState setteri callback ham qabul qiladi. Nega xato bermadi?
useState setterining haqiqiy tipi kengroq: u qiymatni ham, (prev) => ... callback'ni ham qabul qiladi. TypeScript "ko'proq narsa qila oladigan funksiyani kamroq talab qilinadigan joyga bersa bo'ladi" deb hisoblaydi. Bizga faqat string bilan chaqirish kerak, setter esa buni bajara oladi, shuning uchun mos keladi.
5.6. onClick={toggleTheme} va onClick={() => setActiveValue(value)} farqi nima?
• onClick={toggleTheme}: funksiyaning o'zi uzatiladi, bosilganda chaqiriladi. Argument kerak emas, shuning uchun qavssiz.
• onClick={() => setActiveValue(value)}: argument (value) uzatish kerak, shuning uchun arrow funksiya bilan o'raymiz.
• onClick={setActiveValue(value)}: ❌ xato. Funksiya render paytida darhol chaqiriladi va natijasi (undefined) uzatiladi. State o'zgarishi esa yangi render chaqiradi, ya'ni cheksiz render bo'ladi.
Shu qoida addEventListener("scroll", handleScroll) da ham bor: qavssiz yoziladi.
6. Generics
6.1. Generic nima? Sodda qilib tushuntir.
Generic bu tip uchun parametr. Oddiy funksiya qiymat parametri oladi, generic esa tip parametri oladi. Tip chaqirilgan paytda aniqlanadi.
function identity<T>(value: T): T {
  return value;
}

identity("salom"); // T = string, natija string
identity(42);      // T = number, natija number
<T> "bu yerda qandaydir tip bo'ladi, uni keyin bilamiz" degani. T shunchaki nom (Type), <Item> yoki <Value> ham bo'lishi mumkin.
6.2. Nega any emas, generic?
function a(value: any): any { return value; }
function b<T>(value: T): T { return value; }

a("salom").toFixed(); // ✅ TypeScript jim, lekin ishlaganda yiqiladi
b("salom").toFixed(); // ❌ TypeScript darhol ushlaydi: string'da toFixed yo'q
any tipni yo'qotadi, generic esa uni saqlab qoladi. Kirgan tip qaysi bo'lsa, chiqqani ham o'sha.
6.3. Loyihada qanday generic yozding?
useLocalStorageState hooki. U har qanday qiymatni ("dark", son, obyekt) localStorage da saqlay olishi kerak edi:
function useLocalStorageState<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(initialValue);
  // ...
  return [value, setValue] as const;
}
initialValue qaysi tipda bo'lsa, value ham shu tipda bo'ladi. Har tip uchun alohida hook yozish shart emas.
6.4. Generic tipni qachon qo'lda yozish kerak?
TypeScript tipni o'zi chiqaradi, lekin ba'zan juda keng chiqaradi:
useLocalStorageState("theme", "dark");          // T = string, setTheme("banana") ham o'tadi
useLocalStorageState<Theme>("theme", "dark");   // T = "dark" | "light"
Birinchi holatda TypeScript "dark" ni oddiy string deb oladi. Ikkinchisida tipni <Theme> bilan aniq aytdik, endi faqat ikki qiymat qabul qilinadi.
6.5. React'ning qaysi funksiyalari generic?
• useState<T>: useState<Theme>("dark")
• createContext<T>: createContext<ThemeContextValue | null>(null)
• useRef<T>, useReducer ham generic.
Ko'pincha useState ga tip yozish shart emas (useState(false) → boolean). Lekin boshlang'ich qiymat tipni to'liq ko'rsatmasa, yozish kerak. Masalan, useState(null) faqat null tipini oladi, shuning uchun useState<User | null>(null) deb yoziladi.
6.6. Record<K, V> nima?
Tayyor generic tip: "kalitlari K, qiymatlari V bo'lgan obyekt".
const heroImages: Record<string, string> = {
  "hero-night-highway.webp": heroNight,
};

const icons: Record<string, LucideIcon> = {
  truck: Truck,
  globe: Globe,
};
Birinchisida qiymat rasm yo'li (matn), ikkinchisida ikonka komponenti.
6.7. Record<string, ...> ning xavfli tomoni nima?
U "har qanday kalit uchun qiymat bor" deb va'da beradi. Shuning uchun:
heroImages["xato-nom.webp"]; // TypeScript: string. Aslida: undefined
TypeScript xato bermaydi, lekin amalda undefined keladi. Shuning uchun loyihada tekshiruvni bitta funksiyaga yig'dim:
export function getHeroImage(name: string): string {
  const src = heroImages[name];
  if (!src) throw new Error(`Hero image not found: ${name}`);
  return src;
}
Qo'shimcha himoya: tsconfig'da noUncheckedIndexedAccess yoqilsa, TypeScript bunday natijani string | undefined deb beradi va tekshirishga majbur qiladi.
6.8. Record kalitiga union bersa nima bo'ladi?
const labels: Record<"dark" | "light", string> = {
  dark: "Tun",
  light: "Kun",
};
Endi aynan shu ikki kalit majburiy. Bittasini unutsang yoki ortiqcha kalit qo'shsang, xato chiqadi. Bu Record<string, ...> dan ancha xavfsizroq.
7. null, narrowing, as, ! va any
7.1. null va undefined farqi nima?
• undefined: qiymat berilmagan. Masalan, ixtiyoriy prop yozilmasa yoki obyektda yo'q kalit o'qilsa.
• null: "qiymat yo'q" deb ataylab qo'yilgan. Masalan, createContext(null) yoki localStorage.getItem hech narsa topmasa.
const stored = localStorage.getItem("theme"); // string | null
7.2. Nega createContext<ThemeContextValue | null>(null) deb yozding?
Context'ning boshlang'ich qiymati faqat Provider tashqarisida ishlatiladi. Bizda theme va toggleTheme Provider ichida yaratiladi, shuning uchun tashqarida to'g'ri qiymat yo'q. Shu sabab null berdik.
Tip esa haqiqatni aks ettirishi kerak: qiymat null bo'lishi mumkin, demak tipda | null bo'ladi. Faqat <ThemeContextValue> yozilsa, TypeScript (null) ni rad etadi.
7.3. Type narrowing nima?
TypeScript kod mantiqiga qarab tipni toraytiradi. if tekshiruvidan keyin u qiymat qaysi tipda ekanini aniqroq biladi.
function useTheme() {
  const context = useContext(ThemeContext); // ThemeContextValue | null

  if (context === null) {
    throw new Error("useTheme must be used inside ThemeProvider");
  }

  return context; // ThemeContextValue (null yo'q)
}
throw funksiyani to'xtatadi. TypeScript xulosa qiladi: "if dan pastga o'tilgan bo'lsa, context null emas". Hech qanday as yozilmadi, tekshiruv haqiqatan bajariladi.
7.4. Narrowing yana qanday usullar bilan qilinadi?
if (typeof id === "string") { /* id: string */ }
if (!src) throw ...;              // src endi undefined emas
if (truck.pricingType === "perMile") { /* faqat perMile varianti */ }
Oxirgisi kalkulyatorda ishlatiladi: union maydonini tekshirib, har variantni alohida hisoblaymiz.
7.5. Nega null tekshiruvini useTheme ichiga olding?
Bu bo'lmasa, Context'ni ishlatgan har bir komponentda null ni tekshirish kerak bo'lardi. useTheme buni bitta joyda bir marta qiladi. Bonus: kimdir komponentni Provider'siz ishlatsa, chalkash "cannot read properties of null" o'rniga aniq xabar chiqadi. useTabsContext ham xuddi shunday.
7.6. as (type assertion) nima?
TypeScript'ga "bu qiymatning tipini men bilaman, shunday qabul qil" deyish:
export const trucksData = trucksJson as TrucksData;
const parsed = JSON.parse(stored) as T;
Muhim: as tekshirmaydi, faqat ishonadi. JSON'da "pricingType": "banana" yozilsa ham, as dan keyin xato chiqmaydi.
7.7. as ni qachon ishlatish to'g'ri, qachon yomon?
• To'g'ri: ma'lumot manbasini o'zing nazorat qilsang (o'zimiz yozgan JSON) yoki TypeScript bila olmaydigan narsa bo'lsa (JSON.parse natijasi any).
• Yomon: xatoni "jim qildirish" uchun. Bu xatoni tuzatmaydi, faqat yashiradi.
Loyihada as faqat data/index.ts da, bitta joyda turadi. Komponentlar tayyor tiplangan ma'lumot oladi. Tashqi API'dan kelgan ma'lumot uchun as o'rniga Zod kabi runtime tekshiruv ishlatiladi (forma bosqichida ko'ramiz).
7.8. ! (non-null assertion) nima?
Qiymatdan keyin qo'yiladi va "bu null yoki undefined emas" degani:
createRoot(document.getElementById("root")!)
getElementById element topmasa null qaytaradi, tipi HTMLElement | null. ! bu null ni olib tashlaydi. as kabi u ham tekshirmaydi. Bu yerda xavfsiz, chunki index.html da <div id="root"> aniq bor.
7.9. any nima va nega undan qochish kerak?
any "istalgan narsa" degani va TypeScript tekshiruvini butunlay o'chiradi:
const x: any = "salom";
x.toFixed(); // TypeScript jim, ishlaganda yiqiladi
JSON.parse any qaytaradi, shuning uchun uni darhol as T bilan tiplab qo'ydik. Noma'lum tip uchun xavfsizroq variant unknown: u ham hamma narsani qabul qiladi, lekin ishlatishdan oldin narrowing talab qiladi.
7.10. as, ! va narrowing farqi bir jumlada?
Narrowing tekshiradi va shundan keyin tipni toraytiradi. as va ! esa tekshirmaydi, faqat dasturchiga ishonadi. Imkon bo'lsa narrowing, faqat aniq bilgan joyda as/!.
8. React + TypeScript
8.1. React komponent props'i qanday tiplanadi?
Alohida type yoziladi va destructuring'dan keyin : bilan beriladi:
type ContainerProps = {
  children: ReactNode;
  className?: string;
};

function Container({ children, className = "" }: ContainerProps) { ... }
Endi <Container titel="x"> deb noto'g'ri prop yozsang yoki majburiy prop'ni unutsang, TypeScript darhol ogohlantiradi.
8.2. ReactNode nima?
children ichida keladigan har qanday narsaning tipi: matn, son, JSX, bir nechta element, null. children uchun eng ko'p ishlatiladigan tip shu.
<Button>Request a quote</Button>                  // matn
<Button>Contact Us <ArrowRight /></Button>        // matn + komponent
Ikkinchi misol children: ReactNode tufayli ishlaydi.
8.3. Ikki xil HTML element qaytaradigan komponentni qanday qilding?
Button ichida href ga qarab:
if (href) {
  return <a href={href} className={classes}>{children}</a>;
}
return <button type="button" className={classes}>{children}</button>;
href?: string ixtiyoriy. Berilmasa undefined bo'ladi, if o'tkazib yuboriladi va <button> qaytadi. Bu early return. Eski saytdagi <a> ichida <button> xatosi shu bilan hal bo'ldi.
8.4. Rest props (...props) nima va qayerda ishlatding?
Icon komponentida:
function Icon({ name, ...props }: IconProps) {
  const IconComponent = icons[name];
  return <IconComponent {...props} />;
}
name alohida olinadi, qolgan hamma prop'lar (className, strokeWidth...) props obyektiga yig'iladi va haqiqiy ikonkaga to'liq uzatiladi. Tipi IconProps dan name olib tashlangan qism bo'ladi, TypeScript buni o'zi hisoblaydi.
8.5. Nega IconComponent katta harf bilan?
JSX'da kichik harfli teg (<div>) HTML elementi deb, katta harfli (<IconComponent>) esa React komponenti deb tushuniladi. O'zgaruvchida saqlangan komponentni chizish uchun nomi albatta katta harf bilan boshlanishi kerak.
8.6. LucideIcon va LucideProps nima?
lucide-react kutubxonasi bergan tayyor tiplar:
• LucideIcon: ikonka komponentining tipi (icons xaritasidagi qiymatlar uchun).
• LucideProps: ikonka qabul qiladigan prop'lar tipi (IconProps uchun).
Yaxshi kutubxonalar o'z tiplarini o'zi beradi, ularni qo'lda yozish shart emas.
8.7. Context'ni TypeScript bilan qanday tiplading? Qadamlarini ayt.
1.  Qiymat tipini yozish: type ThemeContextValue = { theme: Theme; toggleTheme: () => void }.
2.  Context yaratish: createContext<ThemeContextValue | null>(null).
3.  Provider'da value={{ theme, toggleTheme }} berish. Obyekt tipga mos bo'lmasa (masalan, toggleTheme unutilsa), xato chiqadi.
4.  Custom hook (useTheme) ichida null ni tekshirib, toza tipni qaytarish.
5.  Context'ning o'zini export qilmaslik, faqat Provider va hookni chiqarish.
8.8. Compound component nima va unda Context nega kerak?
Komponent kichik qismlarga bo'linadi va ular birga ishlaydi:
<Tabs defaultValue="ftl">
  <Tabs.List>
    <Tabs.Trigger value="ftl">FTL</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Panel value="ftl">...</Tabs.Panel>
</Tabs>
Tabs o'z bolalariga prop bera olmaydi, chunki ularni Services bo'limi yozadi va ular har xil chuqurlikda turadi. Shuning uchun faol tab Context orqali uzatiladi: Tabs uni efirga chiqaradi, Trigger va Panel o'qiydi. Tabs.List = TabsList yozilganda TypeScript funksiya tipini o'zi kengaytiradi, chunki u shu faylda e'lon qilingan.
8.9. import type nima va nega kerak?
import type { TrucksData } from "../types/trucks";
import { createContext, type ReactNode } from "react";
Tiplar build paytida o'chiriladi. type so'zi kompilyatorga "bu import faqat tip, uni JavaScript'dan butunlay olib tashla" deydi.
• Hammasi tip bo'lsa: import type { ... }.
• Aralash bo'lsa: faqat tip oldiga type.
8.10. verbatimModuleSyntax nima?
tsconfig sozlamasi. U tip importlarida type so'zini majburiy qiladi. Sababi: Vite (esbuild) har faylni alohida, boshqa fayllarga qaramay kompilyatsiya qiladi. U qaysi import tip ekanini o'zi bila olmaydi, shuning uchun buni kodda aniq yozish kerak.
8.11. JSON faylni qanday tipladingiz?
1.  tsconfig'da resolveJsonModule: true (JSON import qilishga ruxsat).
2.  Har JSON uchun types/ da tip yozish.
3.  data/index.ts da hammasini bir marta as bilan tiplab, export qilish.
4.  Komponentlar JSON'ni emas, import { trucksData } from "../data" ni oladi.
Shunda as bitta faylda turadi va hech bir komponent tipsiz ma'lumot bilan ishlamaydi.
8.12. Rasm importi (import logo from "./logo.png") qanday ishlaydi?
Vite rasmni build paytida ko'chiradi, nomiga hash qo'shadi va logo ga yakuniy URL'ni (string) beradi. TypeScript bu importni Vite shablonidagi tiplar (vite/client) orqali taniydi. JSON'da faqat fayl nomi saqlangani uchun nom va import qilingan yo'l Record<string, string> xarita orqali bog'landi.
9. Tez takrorlash
Intervyudan oldin shu jadvalni bir marta ko'zdan kechirish yetadi.
Belgi / tushuncha
Ma'nosi
Misol
: string
o'zgaruvchi yoki parametr tipi
name: string
?
ixtiyoriy maydon
className?: string
|
yo u, yo bu
"dark" | "light"
&
ham u, ham bu
LucideProps & { name: string }
[]
ro'yxat
Truck[]
[A, B]
tuple (aniq uzunlik va tartib)
[string, number]
as const
o'zgarmas, eng aniq tip
return [value, setValue] as const
(): T
funksiya qaytaradigan tip
useScrolled(): boolean
() => void
funksiya tipi, hech narsa qaytarmaydi
toggleTheme: () => void
<T>
generic, tip parametri
useLocalStorageState<T>
Record<K, V>
kalit → qiymat obyekti
Record<string, LucideIcon>
| null + if
narrowing bilan xavfsiz tekshiruv
useTheme
as
tipni qo'lda aytish, tekshirmaydi
trucksJson as TrucksData
!
"null emas", tekshirmaydi
getElementById("root")!
any
tekshiruv o'chadi, qochish kerak
JSON.parse natijasi
unknown
noma'lum, ishlatishdan oldin tekshirish shart
xavfsiz any o'rnini bosuvchi
never
bunday qiymat bo'lmaydi
string & number
ReactNode
children ichidagi har qanday narsa
children: ReactNode
import type
faqat tip importi, JS'dan o'chadi
import type { HeroData }
Eng ko'p so'raladigan 5 savol: type va interface farqi (1.9), | va & farqi (3.3), generic nima va nega any emas (6.1–6.2), as va narrowing farqi (7.10), Context'ni tiplash (8.7).
