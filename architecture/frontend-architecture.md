# Frontend Architecture

React application structure and architecture for the Vitrin platform.

## Overview

The Vitrin frontend is built with React 18, TypeScript, and modern web technologies. The architecture follows best practices for scalability, maintainability, and performance.

## Technology Stack

### Core Technologies

- **React 18**: Modern React with concurrent features
- **TypeScript**: Type-safe development
- **Vite**: Fast build tool and dev server
- **React Router v6**: Client-side routing
- **React Query**: Data fetching and caching

### UI and Styling

- **Tailwind CSS**: Utility-first CSS framework
- **Radix UI**: Accessible component primitives
- **Lucide React**: Icon library
- **Framer Motion**: Animation library

### Development Tools

- **ESLint**: Code linting
- **Prettier**: Code formatting
- **Jest**: Testing framework
- **React Testing Library**: Component testing
- **MSW**: API mocking

## Project Structure

```
vitrin-frontend/
├── public/                 # Static assets
│   ├── favicon.ico
│   ├── manifest.json
│   └── robots.txt
├── src/
│   ├── components/         # React components
│   │   ├── ui/            # Reusable UI components
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   └── index.ts
│   │   ├── AddProduct.tsx
│   │   ├── HomeFeed.tsx
│   │   ├── LoginSignup.tsx
│   │   ├── ProductDetail.tsx
│   │   └── UserProfile.tsx
│   ├── hooks/             # Custom React hooks
│   │   ├── useAuth.ts
│   │   ├── useCategories.ts
│   │   ├── useProducts.ts
│   │   └── useLocalStorage.ts
│   ├── services/          # API services
│   │   ├── api.ts
│   │   ├── auth.ts
│   │   └── storage.ts
│   ├── types/             # TypeScript type definitions
│   │   ├── api.ts
│   │   ├── auth.ts
│   │   └── common.ts
│   ├── utils/             # Utility functions
│   │   ├── constants.ts
│   │   ├── validation.ts
│   │   └── formatting.ts
│   ├── config/            # Configuration files
│   │   ├── api.ts
│   │   └── env.ts
│   ├── styles/            # Global styles
│   │   ├── globals.css
│   │   └── components.css
│   ├── App.tsx            # Main App component
│   ├── main.tsx           # Application entry point
│   └── index.css          # Global CSS
├── package.json
├── vite.config.ts
├── tailwind.config.js
├── tsconfig.json
└── README.md
```

## Component Architecture

### Component Hierarchy

```
App
├── Router
│   ├── PublicRoutes
│   │   ├── LoginSignup
│   │   └── HomeFeed
│   └── ProtectedRoutes
│       ├── AddProduct
│       ├── ProductDetail
│       └── UserProfile
└── Layout
    ├── Header
    ├── Navigation
    └── Footer
```

### Component Types

#### 1. Page Components

```typescript
// Page components handle routing and layout
const ProductDetailPage = () => {
  const { id } = useParams<{ id: string }>();
  const { data: product, isLoading } = useProduct(id);

  if (isLoading) return <LoadingSpinner />;
  if (!product) return <NotFound />;

  return (
    <Layout>
      <ProductDetail product={product} />
    </Layout>
  );
};
```

#### 2. Feature Components

```typescript
// Feature components implement specific functionality
const ProductDetail = ({ product }: { product: Product }) => {
  const [selectedImage, setSelectedImage] = useState(0);
  const { mutate: likeProduct } = useLikeProduct();

  return (
    <div className="product-detail">
      <ProductImages 
        images={product.images}
        selected={selectedImage}
        onSelect={setSelectedImage}
      />
      <ProductInfo product={product} />
      <ProductActions 
        product={product}
        onLike={() => likeProduct(product.id)}
      />
    </div>
  );
};
```

#### 3. UI Components

```typescript
// Reusable UI components
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
}

const Button = ({ 
  variant = 'primary', 
  size = 'md', 
  children, 
  onClick, 
  disabled 
}: ButtonProps) => {
  return (
    <button
      className={cn(
        'btn',
        `btn-${variant}`,
        `btn-${size}`,
        disabled && 'btn-disabled'
      )}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};
```

## State Management

### React Query for Server State

```typescript
// API data fetching and caching
const useProducts = (filters?: ProductFilters) => {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => api.products.list(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
};

const useCreateProduct = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: api.products.create,
    onSuccess: () => {
      queryClient.invalidateQueries(['products']);
    },
  });
};
```

### Context for Global State

```typescript
// Authentication context
interface AuthContextType {
  user: User | null;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

const AuthContext = createContext<AuthContextType | null>(null);

export const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  const login = async (credentials: LoginCredentials) => {
    const response = await authService.login(credentials);
    setUser(response.user);
    localStorage.setItem('token', response.tokens.access);
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('token');
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, isLoading }}>
      {children}
    </AuthContext.Provider>
  );
};
```

### Local State with useState

```typescript
// Component-level state
const AddProduct = () => {
  const [formData, setFormData] = useState<ProductFormData>({
    name: '',
    description: '',
    price: 0,
    category: '',
    images: []
  });
  const [errors, setErrors] = useState<FormErrors>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    try {
      await createProduct(formData);
      // Handle success
    } catch (error) {
      setErrors(error.response.data.details);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
};
```

## Routing

### React Router Configuration

```typescript
// App routing
const App = () => {
  return (
    <Router>
      <AuthProvider>
        <Routes>
          <Route path="/login" element={<LoginSignup />} />
          <Route path="/register" element={<LoginSignup />} />
          <Route path="/" element={<HomeFeed />} />
          <Route path="/products/:id" element={<ProductDetailPage />} />
          <Route path="/add-product" element={
            <ProtectedRoute>
              <AddProduct />
            </ProtectedRoute>
          } />
          <Route path="/profile" element={
            <ProtectedRoute>
              <UserProfile />
            </ProtectedRoute>
          } />
        </Routes>
      </AuthProvider>
    </Router>
  );
};
```

### Protected Routes

```typescript
// Route protection
const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const { user, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) return <LoadingSpinner />;
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
};
```

## API Integration

### API Service Layer

```typescript
// API service
class ApiService {
  private baseURL: string;
  private token: string | null;

  constructor() {
    this.baseURL = import.meta.env.VITE_API_BASE_URL;
    this.token = localStorage.getItem('token');
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.baseURL}${endpoint}`;
    const config: RequestInit = {
      headers: {
        'Content-Type': 'application/json',
        ...(this.token && { Authorization: `Bearer ${this.token}` }),
        ...options.headers,
      },
      ...options,
    };

    const response = await fetch(url, config);
    
    if (!response.ok) {
      throw new Error(`API Error: ${response.status}`);
    }

    return response.json();
  }

  // Products API
  products = {
    list: (filters?: ProductFilters) => 
      this.request<ProductListResponse>('/products/', {
        method: 'GET',
        // Add query parameters
      }),
    
    get: (id: string) => 
      this.request<Product>(`/products/${id}/`),
    
    create: (data: CreateProductData) => 
      this.request<Product>('/products/', {
        method: 'POST',
        body: JSON.stringify(data),
      }),
    
    update: (id: string, data: UpdateProductData) => 
      this.request<Product>(`/products/${id}/`, {
        method: 'PUT',
        body: JSON.stringify(data),
      }),
    
    delete: (id: string) => 
      this.request<void>(`/products/${id}/`, {
        method: 'DELETE',
      }),
  };
}

export const api = new ApiService();
```

### Error Handling

```typescript
// Error boundary
class ErrorBoundary extends Component<
  { children: React.ReactNode },
  { hasError: boolean; error?: Error }
> {
  constructor(props: { children: React.ReactNode }) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Log to error reporting service
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }

    return this.props.children;
  }
}
```

## Performance Optimization

### Code Splitting

```typescript
// Lazy loading components
const AddProduct = lazy(() => import('./components/AddProduct'));
const UserProfile = lazy(() => import('./components/UserProfile'));

// Route-based code splitting
const App = () => (
  <Suspense fallback={<LoadingSpinner />}>
    <Routes>
      <Route path="/add-product" element={<AddProduct />} />
      <Route path="/profile" element={<UserProfile />} />
    </Routes>
  </Suspense>
);
```

### Image Optimization

```typescript
// Image component with optimization
const OptimizedImage = ({ 
  src, 
  alt, 
  width, 
  height 
}: ImageProps) => {
  const [isLoaded, setIsLoaded] = useState(false);
  const [hasError, setHasError] = useState(false);

  return (
    <div className="image-container">
      {!isLoaded && !hasError && <ImageSkeleton />}
      <img
        src={src}
        alt={alt}
        width={width}
        height={height}
        loading="lazy"
        onLoad={() => setIsLoaded(true)}
        onError={() => setHasError(true)}
        className={cn(
          'transition-opacity duration-300',
          isLoaded ? 'opacity-100' : 'opacity-0'
        )}
      />
      {hasError && <ImageError />}
    </div>
  );
};
```

### Memoization

```typescript
// Memoized components
const ProductCard = memo(({ product }: { product: Product }) => {
  const { mutate: likeProduct } = useLikeProduct();
  
  const handleLike = useCallback(() => {
    likeProduct(product.id);
  }, [likeProduct, product.id]);

  return (
    <div className="product-card">
      <ProductImage product={product} />
      <ProductInfo product={product} />
      <LikeButton 
        isLiked={product.isLiked}
        onClick={handleLike}
      />
    </div>
  );
});
```

## Testing

### Component Testing

```typescript
// Component test
import { render, screen, fireEvent } from '@testing-library/react';
import { ProductCard } from './ProductCard';

describe('ProductCard', () => {
  const mockProduct = {
    id: 1,
    name: 'Test Product',
    price: 99.99,
    images: [{ url: 'test.jpg', alt_text: 'Test' }],
  };

  it('renders product information', () => {
    render(<ProductCard product={mockProduct} />);
    
    expect(screen.getByText('Test Product')).toBeInTheDocument();
    expect(screen.getByText('$99.99')).toBeInTheDocument();
  });

  it('handles like button click', () => {
    const onLike = jest.fn();
    render(<ProductCard product={mockProduct} onLike={onLike} />);
    
    fireEvent.click(screen.getByRole('button', { name: /like/i }));
    expect(onLike).toHaveBeenCalledWith(mockProduct.id);
  });
});
```

### API Mocking

```typescript
// MSW setup
import { setupServer } from 'msw/node';
import { rest } from 'msw';

const server = setupServer(
  rest.get('/api/products/', (req, res, ctx) => {
    return res(
      ctx.json({
        results: [
          { id: 1, name: 'Test Product', price: 99.99 }
        ],
        count: 1
      })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Build and Deployment

### Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
          ui: ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
        },
      },
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
});
```

### Environment Configuration

```typescript
// Environment variables
interface EnvConfig {
  API_BASE_URL: string;
  GOOGLE_CLIENT_ID: string;
  APP_NAME: string;
  APP_VERSION: string;
}

export const env: EnvConfig = {
  API_BASE_URL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000/api',
  GOOGLE_CLIENT_ID: import.meta.env.VITE_GOOGLE_CLIENT_ID || '',
  APP_NAME: import.meta.env.VITE_APP_NAME || 'Vitrin',
  APP_VERSION: import.meta.env.VITE_APP_VERSION || '1.0.0',
};
```

## Accessibility

### ARIA Implementation

```typescript
// Accessible components
const Modal = ({ isOpen, onClose, children }: ModalProps) => {
  const modalRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (isOpen && modalRef.current) {
      modalRef.current.focus();
    }
  }, [isOpen]);

  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      ref={modalRef}
      tabIndex={-1}
    >
      <button
        onClick={onClose}
        aria-label="Close modal"
        className="close-button"
      >
        ×
      </button>
      {children}
    </div>
  );
};
```

### Keyboard Navigation

```typescript
// Keyboard navigation hook
const useKeyboardNavigation = (items: any[], onSelect: (item: any) => void) => {
  const [activeIndex, setActiveIndex] = useState(0);

  const handleKeyDown = useCallback((e: KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setActiveIndex(prev => (prev + 1) % items.length);
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex(prev => (prev - 1 + items.length) % items.length);
        break;
      case 'Enter':
        e.preventDefault();
        onSelect(items[activeIndex]);
        break;
    }
  }, [items, activeIndex, onSelect]);

  useEffect(() => {
    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [handleKeyDown]);

  return { activeIndex };
};
```
