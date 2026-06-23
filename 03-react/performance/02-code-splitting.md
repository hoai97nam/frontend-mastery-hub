# 📦 Code Splitting & Lazy Loading

> Giảm bundle size, tăng tốc load ban đầu bằng cách **chỉ tải code khi cần**.

---

## Mục Lục

1. [Tại Sao Cần Code Splitting?](#1-tại-sao-cần-code-splitting)
2. [React.lazy & Suspense](#2-reactlazy--suspense)
3. [Route-based Splitting](#3-route-based-splitting)
4. [Component-level Splitting](#4-component-level-splitting)
5. [Bundle Analysis](#5-bundle-analysis)
6. [Preloading & Prefetching](#6-preloading--prefetching)
7. [Dynamic Import Patterns](#7-dynamic-import-patterns)
8. [Images & Assets](#8-images--assets)

---

## 1. Tại Sao Cần Code Splitting?

### Vấn Đề — Monolithic Bundle

```
Không code split:
├── main.js (2.5MB) ← Toàn bộ app tải cùng lúc
    ├── HomePage code
    ├── AdminDashboard code     ← User thường không cần
    ├── ReportingModule code    ← Chỉ admin dùng
    ├── ChartLibrary (800KB)    ← Chỉ dùng ở 1 trang
    └── PDFExporter (400KB)     ← Chỉ dùng khi export
```

```
Sau code split:
├── main.js (300KB)   ← Load ngay khi vào app
├── admin.js (400KB)  ← Load khi navigate vào /admin
├── reports.js (500KB)← Load khi cần báo cáo
└── charts.js (800KB) ← Load khi render chart
```

### Impact Thực Tế

| Metric | Không Split | Có Split | Cải thiện |
|--------|-------------|----------|-----------|
| Initial JS | 2.5MB | 300KB | -88% |
| Time to Interactive | 8.2s | 2.1s | -74% |
| Lighthouse Score | 42 | 87 | +45 điểm |

---

## 2. `React.lazy` & `Suspense`

### Cú Pháp Cơ Bản

```jsx
import { lazy, Suspense } from 'react';

// ❌ Import thông thường — vào bundle ngay lập tức
import HeavyComponent from './HeavyComponent';

// ✅ Lazy import — tải khi cần render lần đầu
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### Suspense Fallback Tốt vs Xấu

```jsx
// ❌ Fallback đơn giản gây layout shift
<Suspense fallback={<div>Loading...</div>}>

// ✅ Skeleton UI — giữ layout, tránh shift
<Suspense fallback={<ProductCardSkeleton />}>
  <ProductCard />
</Suspense>

// ✅ Skeleton tự tạo
function ProductCardSkeleton() {
  return (
    <div className="skeleton-card">
      <div className="skeleton skeleton-image" />
      <div className="skeleton skeleton-title" />
      <div className="skeleton skeleton-price" />
    </div>
  );
}
```

```css
/* Skeleton animation */
.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: 4px;
}

@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}
```

### Đặt Suspense Boundary Đúng Chỗ

```jsx
// ❌ Suspense quá cao → toàn bộ trang bị fallback
function App() {
  return (
    <Suspense fallback={<FullPageSpinner />}> {/* Bọc cả app */}
      <Header />
      <LazyDashboard />
      <Footer />
    </Suspense>
  );
}

// ✅ Suspense bọc đúng component cần lazy load
function App() {
  return (
    <>
      <Header />  {/* Không bị ảnh hưởng */}
      <Suspense fallback={<DashboardSkeleton />}>
        <LazyDashboard /> {/* Chỉ section này fallback */}
      </Suspense>
      <Footer />  {/* Không bị ảnh hưởng */}
    </>
  );
}
```

### Error Handling với ErrorBoundary

```jsx
import { Component, lazy, Suspense } from 'react';

class LazyErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <p>Không tải được module.</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Thử lại
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

const LazyChart = lazy(() => import('./Chart'));

function Dashboard() {
  return (
    <LazyErrorBoundary>
      <Suspense fallback={<ChartSkeleton />}>
        <LazyChart />
      </Suspense>
    </LazyErrorBoundary>
  );
}
```

---

## 3. Route-based Splitting

### Với React Router v6

```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazy load từng trang
const HomePage    = lazy(() => import('./pages/HomePage'));
const ProductPage = lazy(() => import('./pages/ProductPage'));
const AdminPage   = lazy(() => import('./pages/AdminPage'));
const CheckoutPage = lazy(() => import('./pages/CheckoutPage'));

// Loading component có thể tái sử dụng
function PageLoader() {
  return (
    <div style={{ display: 'flex', justifyContent: 'center', padding: '80px' }}>
      <Spinner size={40} />
    </div>
  );
}

function AppRouter() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/"         element={<HomePage />} />
        <Route path="/products/:id" element={<ProductPage />} />
        <Route path="/checkout" element={<CheckoutPage />} />
        {/* Admin routes — chỉ load khi vào /admin */}
        <Route path="/admin/*"  element={<AdminPage />} />
      </Routes>
    </Suspense>
  );
}
```

### Với Next.js App Router

```jsx
// ✅ Mặc định: Server Components được code-split tự động
// ✅ Chỉ cần lazy load Client Components nặng

import dynamic from 'next/dynamic';

// Lazy load với SSR tắt (component dùng browser APIs)
const Chart = dynamic(() => import('./Chart'), {
  ssr: false,
  loading: () => <ChartSkeleton />,
});

// Lazy load với SSR bật (component có thể render server)
const RichEditor = dynamic(() => import('./RichEditor'), {
  loading: () => <EditorSkeleton />,
});

export default function Page() {
  return (
    <div>
      <Chart />      {/* Load client-side only */}
      <RichEditor /> {/* SSR + hydration */}
    </div>
  );
}
```

---

## 4. Component-level Splitting

### Lazy Load Khi User Cần (Modal, Tooltip, v.v.)

```jsx
const HeavyModal = lazy(() => import('./HeavyModal'));

function App() {
  const [showModal, setShowModal] = useState(false);

  return (
    <>
      <button onClick={() => setShowModal(true)}>
        Mở Modal
      </button>

      {/* Modal chỉ load khi showModal = true */}
      {showModal && (
        <Suspense fallback={<ModalSkeleton />}>
          <HeavyModal onClose={() => setShowModal(false)} />
        </Suspense>
      )}
    </>
  );
}
```

### Lazy Load Dựa Trên Feature Flag / User Role

```jsx
const AdminPanel   = lazy(() => import('./AdminPanel'));
const RegularPanel = lazy(() => import('./RegularPanel'));

function Dashboard({ userRole }) {
  const Panel = userRole === 'admin' ? AdminPanel : RegularPanel;

  return (
    <Suspense fallback={<PanelSkeleton />}>
      <Panel />
    </Suspense>
  );
}
```

### Conditional Lazy Import (Pattern Nâng Cao)

```jsx
// Chỉ load PDF library khi user click export
async function handleExport(data) {
  // Dynamic import — chỉ tải khi cần
  const { exportToPDF } = await import('./utils/pdfExporter');
  await exportToPDF(data);
}

function ReportPage({ data }) {
  return (
    <div>
      <ReportView data={data} />
      <button onClick={() => handleExport(data)}>
        Export PDF
      </button>
    </div>
  );
}
```

---

## 5. Bundle Analysis

### Vite — Visualize Bundle

```bash
# Cài plugin
npm install -D rollup-plugin-visualizer

# vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

export default {
  plugins: [
    visualizer({
      open: true,       // Tự mở browser
      gzipSize: true,   // Hiển thị gzip size
      brotliSize: true, // Hiển thị brotli size
      filename: 'stats.html',
    }),
  ],
};

# Run build để tạo report
npm run build
```

### Webpack Bundle Analyzer

```bash
# CRA
npx source-map-explorer 'build/static/js/*.js'

# Webpack
npm install -D webpack-bundle-analyzer
# webpack.config.js thêm plugin
```

### Đọc Bundle Report

```
Tìm:
□ Thư viện lớn không cần thiết (moment.js 67KB → thay bằng date-fns 13KB)
□ Duplicate code (lodash được import nhiều cách)
□ Module bị bundle nhầm vào main chunk
□ Dependencies không tree-shakeable
```

### Tree Shaking — Import Đúng Cách

```jsx
// ❌ Import toàn bộ lodash (70KB+)
import _ from 'lodash';
const debounced = _.debounce(fn, 300);

// ✅ Import chỉ hàm cần dùng (vài KB)
import debounce from 'lodash/debounce';

// ✅ Hoặc dùng lodash-es (ES modules, tree-shakeable)
import { debounce } from 'lodash-es';

// ❌ Import toàn bộ MUI icons (huge!)
import { Delete, Edit, Add } from '@mui/icons-material';

// ✅ Import từng icon riêng
import Delete from '@mui/icons-material/Delete';
import Edit   from '@mui/icons-material/Edit';
```

---

## 6. Preloading & Prefetching

### Preload — Load Ngay Khi Có Thể

```jsx
// Preload component khi user hover vào link
const AdminPage = lazy(() => import('./AdminPage'));

function NavLink({ to, label }) {
  // Preload khi hover — gần như chắc chắn user sẽ click
  const handleMouseEnter = () => {
    import('./AdminPage'); // Trigger download, không await
  };

  return (
    <Link to={to} onMouseEnter={handleMouseEnter}>
      {label}
    </Link>
  );
}
```

### Preload theo Route Transition

```jsx
import { useNavigate } from 'react-router-dom';

function ProductCard({ product }) {
  const navigate = useNavigate();

  // Preload detail page khi hover vào card
  const handleMouseEnter = useCallback(() => {
    import('./pages/ProductDetail'); // Warm up
  }, []);

  return (
    <div
      onMouseEnter={handleMouseEnter}
      onClick={() => navigate(`/products/${product.id}`)}
    >
      {product.name}
    </div>
  );
}
```

### `<link rel="preload">` trong HTML

```html
<!-- Preload critical chunks -->
<link rel="preload" href="/static/js/main.chunk.js" as="script">
<link rel="preload" href="/static/css/main.chunk.css" as="style">

<!-- Prefetch likely-needed chunks -->
<link rel="prefetch" href="/static/js/admin.chunk.js">
```

---

## 7. Dynamic Import Patterns

### Named Export với Lazy

```jsx
// ❌ React.lazy chỉ hỗ trợ default export
const { UserModal } = lazy(() => import('./Modals')); // Không hoạt động

// ✅ Wrap để tạo default export giả
const UserModal = lazy(() =>
  import('./Modals').then(module => ({ default: module.UserModal }))
);
```

### Lazy Load Thư Viện Nặng

```jsx
// ✅ Chỉ load monaco-editor khi user mở code editor
function CodeEditor({ code, onChange }) {
  const [Editor, setEditor] = useState(null);

  useEffect(() => {
    import('@monaco-editor/react').then(mod => {
      setEditor(() => mod.default);
    });
  }, []);

  if (!Editor) return <SimpleTextarea code={code} onChange={onChange} />;
  return <Editor value={code} onChange={onChange} language="javascript" />;
}
```

---

## 8. Images & Assets

### Lazy Load Images

```jsx
// Native lazy loading (modern browsers)
function ProductImage({ src, alt }) {
  return (
    <img
      src={src}
      alt={alt}
      loading="lazy"        // Browser tự handle
      decoding="async"      // Không block main thread
      width={400}           // Tránh layout shift
      height={300}
    />
  );
}
```

### Intersection Observer — Custom Lazy Image

```jsx
function LazyImage({ src, alt, placeholder }) {
  const imgRef = useRef(null);
  const [loaded, setLoaded] = useState(false);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          imgRef.current.src = src;
          observer.disconnect();
        }
      },
      { rootMargin: '200px' } // Bắt đầu load trước 200px
    );

    if (imgRef.current) observer.observe(imgRef.current);
    return () => observer.disconnect();
  }, [src]);

  return (
    <img
      ref={imgRef}
      src={placeholder}        // Placeholder ban đầu
      alt={alt}
      onLoad={() => setLoaded(true)}
      style={{ filter: loaded ? 'none' : 'blur(10px)', transition: 'filter 0.3s' }}
    />
  );
}
```

### Next.js Image Optimization

```jsx
import Image from 'next/image';

// ✅ Tự động: lazy load, WebP conversion, responsive sizes, blur placeholder
function ProductCard({ product }) {
  return (
    <div>
      <Image
        src={product.imageUrl}
        alt={product.name}
        width={400}
        height={300}
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,..."
        priority={product.isFeatured} // Load ngay nếu above-the-fold
      />
    </div>
  );
}
```

---

## 📎 Tổng Kết

```
Initial load chậm?
  □ Route-based code splitting (React.lazy + Suspense)
  □ Bundle analysis → tìm thư viện nặng
  □ Tree shaking đúng cách
  □ Image lazy loading

Specific feature nặng?
  □ Component-level lazy loading
  □ Dynamic import cho thư viện (PDF, chart, editor)
  □ Preload khi hover để giảm perceived latency
```

---

🔙 [Rendering Optimization](./01-rendering-optimization.md) | 🔙 [README](./README.md) | ➡️ [State & Data Flow Performance](./03-state-data-perf.md)
