# 🗃️ State & Data Flow Performance

> Hiệu năng của React phụ thuộc nhiều vào **cách bạn tổ chức và truyền state**. Đây là nguồn gốc phổ biến nhất của re-render không cần thiết.

---

## Mục Lục

1. [State Shape — Cấu Trúc State Đúng Cách](#1-state-shape--cấu-trúc-state-đúng-cách)
2. [State Colocation — Đặt State Đúng Chỗ](#2-state-colocation--đặt-state-đúng-chỗ)
3. [Context Performance](#3-context-performance)
4. [Selector Pattern](#4-selector-pattern)
5. [Zustand — Global State Không Re-render](#5-zustand--global-state-không-re-render)
6. [Normalization — State Chuẩn Hóa](#6-normalization--state-chuẩn-hóa)
7. [Derived State — Tính Toán Từ State](#7-derived-state--tính-toán-từ-state)

---

## 1. State Shape — Cấu Trúc State Đúng Cách

### ❌ State Không Hiệu Quả

```jsx
// ❌ State lồng sâu → khó update, dễ gây re-render toàn bộ
const [state, setState] = useState({
  user: {
    profile: {
      name: 'Nguyen Van A',
      address: {
        city: 'HCM',
        district: 'Q1',
      }
    },
    settings: {
      theme: 'dark',
      notifications: true,
    }
  },
  cart: [...],
  products: [...],
});

// Khi update một field nhỏ → phải spread toàn bộ object
setState(prev => ({
  ...prev,
  user: {
    ...prev.user,
    profile: {
      ...prev.user.profile,
      address: {
        ...prev.user.profile.address,
        city: 'Hanoi', // Chỉ đổi city này thôi!
      }
    }
  }
}));
```

### ✅ State Phẳng (Flat State)

```jsx
// ✅ Tách thành nhiều state độc lập
const [user, setUser] = useState({ name: '', email: '' });
const [address, setAddress] = useState({ city: '', district: '' });
const [settings, setSettings] = useState({ theme: 'dark', notifications: true });
const [cart, setCart] = useState([]);

// Update đơn giản, chỉ re-render component cần thiết
setAddress(prev => ({ ...prev, city: 'Hanoi' }));
```

### Immer — Mutate Immutably

```jsx
import { useImmer } from 'use-immer';

function ProfileEditor() {
  const [state, updateState] = useImmer({
    user: {
      profile: { name: '', address: { city: '' } },
      settings: { theme: 'dark' },
    }
  });

  // ✅ Viết code như đang mutate trực tiếp, nhưng thực ra immutable
  const handleCityChange = (city) => {
    updateState(draft => {
      draft.user.profile.address.city = city; // Gọn và rõ ràng!
    });
  };

  return (
    <input
      value={state.user.profile.address.city}
      onChange={e => handleCityChange(e.target.value)}
    />
  );
}
```

---

## 2. State Colocation — Đặt State Đúng Chỗ

### Nguyên Tắc: State Càng Thấp Càng Tốt

```
           App
          /   \
        Page  Sidebar
       /    \
    Header  Content        ← State của Content nên đặt ở Content
              |
           ItemList        ← Không phải ở App hay Page
```

### ❌ State Quá Cao (Over-lifting)

```jsx
// ❌ Modal open state đặt ở App → mọi thứ re-render
function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [user, setUser] = useState(null);

  return (
    <>
      <Header />       {/* Re-render khi isModalOpen thay đổi */}
      <Sidebar />      {/* Re-render khi isModalOpen thay đổi */}
      <MainContent
        isModalOpen={isModalOpen}
        setIsModalOpen={setIsModalOpen}
      />
      <Footer />       {/* Re-render khi isModalOpen thay đổi */}
    </>
  );
}
```

### ✅ State Colocated

```jsx
// ✅ Modal state sống trong component cần nó
function MainContent() {
  const [isModalOpen, setIsModalOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsModalOpen(true)}>Mở Modal</button>
      {isModalOpen && <Modal onClose={() => setIsModalOpen(false)} />}
    </div>
  );
}

// App hoàn toàn sạch
function App() {
  return (
    <>
      <Header />
      <Sidebar />
      <MainContent /> {/* Chỉ MainContent re-render khi modal toggle */}
      <Footer />
    </>
  );
}
```

### "Children as Props" Pattern — Tránh Re-render

```jsx
// ❌ Vấn đề: count thay đổi → cả App re-render, ExpensiveTree cũng re-render
function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <ExpensiveTree /> {/* Không dùng count nhưng vẫn re-render */}
    </div>
  );
}

// ✅ Giải pháp: Extract state vào component riêng, nhận children
function Counter({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      {children} {/* ExpensiveTree được truyền từ ngoài */}
    </div>
  );
}

function App() {
  return (
    <Counter>
      <ExpensiveTree /> {/* Không re-render khi count thay đổi! */}
    </Counter>
  );
}
```

> **Tại sao hoạt động?** `<ExpensiveTree />` được tạo bởi `App` (không re-render), không phải `Counter`. React không re-create element nếu reference không đổi.

---

## 3. Context Performance

### Vấn Đề — Context Re-render All Consumers

```jsx
// ❌ Một value object → tất cả consumers re-render khi BẤT KỲ field nào thay đổi
const AppContext = createContext(null);

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('dark');
  const [language, setLanguage] = useState('vi');

  // Mỗi render → object mới → tất cả consumers re-render
  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme, language, setLanguage }}>
      {children}
    </AppContext.Provider>
  );
}

// Component chỉ cần theme nhưng vẫn re-render khi user thay đổi!
function ThemeToggle() {
  const { theme, setTheme } = useContext(AppContext);
  return <button onClick={() => setTheme('light')}>{theme}</button>;
}
```

### Fix 1 — Tách Context Theo Domain

```jsx
// ✅ Mỗi context chịu trách nhiệm một domain
const UserContext   = createContext(null);
const ThemeContext  = createContext(null);
const LanguageContext = createContext(null);

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('dark');
  const [language, setLanguage] = useState('vi');

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <LanguageContext.Provider value={{ language, setLanguage }}>
          {children}
        </LanguageContext.Provider>
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// ThemeToggle chỉ subscribe ThemeContext → không re-render khi user thay đổi
function ThemeToggle() {
  const { theme, setTheme } = useContext(ThemeContext);
  return <button onClick={() => setTheme('light')}>{theme}</button>;
}
```

### Fix 2 — Tách Data và Dispatch Context

```jsx
// ✅ Pattern phổ biến: tách state và dispatch để tránh re-render khi chỉ cần dispatch
const CartStateContext    = createContext(null);
const CartDispatchContext = createContext(null);

function CartProvider({ children }) {
  const [cart, dispatch] = useReducer(cartReducer, []);

  return (
    <CartStateContext.Provider value={cart}>
      <CartDispatchContext.Provider value={dispatch}>
        {children}
      </CartDispatchContext.Provider>
    </CartStateContext.Provider>
  );
}

// Component chỉ dispatch (không cần cart state) → không re-render khi cart thay đổi
function AddToCartButton({ productId }) {
  const dispatch = useContext(CartDispatchContext);
  return (
    <button onClick={() => dispatch({ type: 'ADD', productId })}>
      Thêm vào giỏ
    </button>
  );
}

// Component chỉ đọc cart (không dispatch)
function CartCount() {
  const cart = useContext(CartStateContext);
  return <span>{cart.length}</span>;
}
```

### Fix 3 — Memoize Context Value

```jsx
function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  // ✅ Chỉ tạo object mới khi user thực sự thay đổi
  const value = useMemo(() => ({
    user,
    login: async (creds) => {
      const data = await auth.login(creds);
      setUser(data);
    },
    logout: () => setUser(null),
  }), [user]);

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

---

## 4. Selector Pattern

### Vấn Đề — Subscribe Toàn Bộ State

```jsx
// ❌ Component re-render khi bất kỳ phần nào của store thay đổi
function UserAvatar() {
  const store = useContext(StoreContext);
  // Chỉ cần avatar nhưng subscribe toàn bộ store
  return <img src={store.user.avatar} />;
}
```

### Implement Selector với `useSyncExternalStore`

```jsx
import { useSyncExternalStore } from 'react';

// Simple store implementation
function createStore(initialState) {
  let state = initialState;
  const listeners = new Set();

  return {
    getState: () => state,
    setState: (updater) => {
      state = typeof updater === 'function' ? updater(state) : updater;
      listeners.forEach(listener => listener());
    },
    subscribe: (listener) => {
      listeners.add(listener);
      return () => listeners.delete(listener);
    },
  };
}

const store = createStore({ user: { name: 'An', avatar: '/img/an.jpg' }, theme: 'dark' });

// Selector hook — chỉ re-render khi selected value thay đổi
function useSelector(selector) {
  return useSyncExternalStore(
    store.subscribe,
    () => selector(store.getState()),
  );
}

// ✅ Chỉ re-render khi user.avatar thay đổi
function UserAvatar() {
  const avatar = useSelector(state => state.user.avatar);
  return <img src={avatar} />;
}

// ✅ Chỉ re-render khi theme thay đổi
function ThemeIndicator() {
  const theme = useSelector(state => state.theme);
  return <span>{theme}</span>;
}
```

---

## 5. Zustand — Global State Không Re-render

### Tại Sao Zustand Hiệu Năng Tốt Hơn Context

```
Context:  Thay đổi bất kỳ field nào → re-render TẤT CẢ consumers
Zustand:  Chỉ re-render component subscribe ĐÚNG field thay đổi
```

### Setup & Selector

```jsx
import { create } from 'zustand';

const useStore = create((set, get) => ({
  // State
  user: null,
  cart: [],
  theme: 'dark',

  // Actions
  setUser: (user) => set({ user }),
  setTheme: (theme) => set({ theme }),

  addToCart: (item) => set(state => ({
    cart: [...state.cart, item]
  })),

  removeFromCart: (id) => set(state => ({
    cart: state.cart.filter(item => item.id !== id)
  })),

  // Computed (gọi trong component)
  getCartTotal: () =>
    get().cart.reduce((sum, item) => sum + item.price * item.qty, 0),
}));

// ✅ Chỉ re-render khi user thay đổi (không re-render khi cart thay đổi)
function UserAvatar() {
  const user = useStore(state => state.user);
  return user ? <img src={user.avatar} /> : null;
}

// ✅ Chỉ re-render khi cart.length thay đổi
function CartBadge() {
  const count = useStore(state => state.cart.length);
  return <span>{count}</span>;
}

// ✅ Subscribe nhiều field — dùng useShallow để so sánh nông
import { useShallow } from 'zustand/react/shallow';

function CartSummary() {
  const { cart, addToCart, removeFromCart } = useStore(
    useShallow(state => ({
      cart: state.cart,
      addToCart: state.addToCart,
      removeFromCart: state.removeFromCart,
    }))
  );
  return <div>...</div>;
}
```

### Zustand Middleware — Persist & Devtools

```jsx
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

const useCartStore = create(
  devtools(
    persist(
      immer((set) => ({
        items: [],
        addItem: (item) => set(state => {
          // ✅ Immer cho phép mutate trực tiếp
          const existing = state.items.find(i => i.id === item.id);
          if (existing) {
            existing.qty += 1;
          } else {
            state.items.push({ ...item, qty: 1 });
          }
        }),
        removeItem: (id) => set(state => {
          state.items = state.items.filter(i => i.id !== id);
        }),
        clearCart: () => set({ items: [] }),
      })),
      {
        name: 'cart-storage', // localStorage key
        partialize: (state) => ({ items: state.items }), // Chỉ persist items
      }
    ),
    { name: 'CartStore' } // Redux DevTools label
  )
);
```

---

## 6. Normalization — State Chuẩn Hóa

### ❌ State Lồng Nhau — Khó Update

```jsx
// ❌ Products lồng trong categories → update 1 product phải duyệt toàn bộ
const state = {
  categories: [
    {
      id: 1,
      name: 'Electronics',
      products: [
        { id: 101, name: 'Phone', price: 999, stock: 50 },
        { id: 102, name: 'Laptop', price: 1499, stock: 20 },
      ]
    },
    {
      id: 2,
      name: 'Clothing',
      products: [
        { id: 201, name: 'T-Shirt', price: 29, stock: 100 },
      ]
    }
  ]
};

// Để update stock của product 102 → khó và chậm
```

### ✅ Normalized State — Flat & Fast

```jsx
// ✅ Cấu trúc giống database — lookup O(1)
const state = {
  categories: {
    byId: {
      1: { id: 1, name: 'Electronics', productIds: [101, 102] },
      2: { id: 2, name: 'Clothing', productIds: [201] },
    },
    allIds: [1, 2],
  },
  products: {
    byId: {
      101: { id: 101, name: 'Phone', price: 999, stock: 50, categoryId: 1 },
      102: { id: 102, name: 'Laptop', price: 1499, stock: 20, categoryId: 1 },
      201: { id: 201, name: 'T-Shirt', price: 29, stock: 100, categoryId: 2 },
    },
    allIds: [101, 102, 201],
  },
};

// Update stock → O(1), chỉ re-render component dùng product 102
const updateStock = (productId, newStock) => ({
  ...state,
  products: {
    ...state.products,
    byId: {
      ...state.products.byId,
      [productId]: {
        ...state.products.byId[productId],
        stock: newStock,
      }
    }
  }
});
```

### Với `normalizr` Library

```jsx
import { normalize, schema } from 'normalizr';

const product = new schema.Entity('products');
const category = new schema.Entity('categories', {
  products: [product]
});

const apiResponse = {
  categories: [
    { id: 1, name: 'Electronics', products: [{ id: 101, name: 'Phone' }] }
  ]
};

const normalizedData = normalize(apiResponse, { categories: [category] });
// → { entities: { products: {...}, categories: {...} }, result: {...} }
```

---

## 7. Derived State — Tính Toán Từ State

### ❌ Lưu Derived State Vào State

```jsx
// ❌ Antipattern: lưu cả total vào state
const [cart, setCart] = useState([]);
const [total, setTotal] = useState(0); // Derived từ cart!

const addItem = (item) => {
  const newCart = [...cart, item];
  setCart(newCart);
  setTotal(newCart.reduce((sum, i) => sum + i.price, 0)); // Dễ sai sync
};
```

### ✅ Tính Derived State Trong Render (+ useMemo nếu Cần)

```jsx
// ✅ Chỉ lưu source of truth
const [cart, setCart] = useState([]);

// Computed trong render — đúng nhất
const total = cart.reduce((sum, item) => sum + item.price * item.qty, 0);
const itemCount = cart.reduce((sum, item) => sum + item.qty, 0);
const hasItems = cart.length > 0;

// Hoặc useMemo nếu tính toán nặng
const expensiveTotal = useMemo(() =>
  cart.reduce((sum, item) => sum + applyDiscount(item), 0),
  [cart]
);
```

### Khi Nào Derived State Nên Lưu?

```
Không cần lưu (tính trong render):
  ✓ Tính toán đơn giản (sum, count, filter)
  ✓ Derive từ 1 source

Nên useMemo:
  ✓ Tính toán nặng (sort 10k items, complex math)
  ✓ Kết quả dùng trong nhiều chỗ trong render

Nên lưu vào state (hiếm):
  ✓ Giá trị có thể thay đổi độc lập với source
  ✓ Giá trị đến từ async operation
```

---

## 📎 Tổng Kết

```
State quá nhiều re-render?
  □ Colocate state — đưa xuống component gần nhất
  □ "Children as props" pattern
  □ Tách Context theo domain
  □ Tách State và Dispatch Context

Global state gây lag?
  □ Dùng Zustand với selector thay vì Context
  □ Normalize state (flat structure)
  □ Chỉ subscribe field cần thiết

Derived state sai?
  □ Không lưu derived state vào state
  □ Tính trong render, hoặc useMemo nếu nặng
```

---

🔙 [Code Splitting](./02-code-splitting.md) | 🔙 [README](./README.md) | ➡️ [Network & Data Fetching](./04-network-data.md)
