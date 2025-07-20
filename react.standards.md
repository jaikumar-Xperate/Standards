# React + TypeScript + Vite Development Standards

*A practical guide to building maintainable React applications*

## Table of Contents
- [Project Setup](#project-setup)
- [Project Structure](#project-structure)
- [TypeScript Configuration](#typescript-configuration)
- [Component Standards](#component-standards)
- [State Management](#state-management)
- [Styling Standards](#styling-standards)
- [Testing Standards](#testing-standards)
- [Code Quality & Linting](#code-quality--linting)
- [Performance Standards](#performance-standards)
- [Security Standards](#security-standards)
- [Documentation Standards](#documentation-standards)

---

## Project Setup

Getting started with a new React project? Here's our tried-and-tested approach.

### Initial Setup
```bash
# Create a new Vite + React + TypeScript project
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install

# Add any additional tools you'll need
npm install react-router-dom
npm install -D @testing-library/react @testing-library/jest-dom vitest jsdom
```

### Essential Dependencies

These are the packages we typically include in every project:

```json
{
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "eslint": "^8.45.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.3",
    "typescript": "^5.0.2",
    "vite": "^5.0.0"
  }
}
```

---

## Project Structure

A well-organized project structure makes development smoother and onboarding easier:

```
src/
├── components/           # Reusable UI components
│   ├── ui/              # Basic building blocks (Button, Input, Modal)
│   ├── forms/           # Form-specific components
│   └── layout/          # Layout components (Header, Sidebar, Footer)
├── pages/               # Page-level components (routes)
├── hooks/               # Custom React hooks
├── contexts/            # React Context providers
├── services/            # API calls and external service integrations
├── utils/               # Pure utility functions
├── types/               # TypeScript type definitions
├── constants/           # Application constants and configuration
├── assets/              # Static files (images, icons, fonts)
├── styles/              # Global styles and theme definitions
├── __tests__/           # Test files and test utilities
├── App.tsx              # Main application component
├── main.tsx             # Application entry point
└── vite-env.d.ts        # Vite type definitions
```

### File Naming Conventions

Consistency in naming makes your codebase easier to navigate:

- **Components**: `PascalCase` → `UserProfile.tsx`
- **Custom Hooks**: `camelCase` with 'use' prefix → `useUserData.ts`
- **Utilities**: `camelCase` → `formatDate.ts`
- **Constants**: `SCREAMING_SNAKE_CASE` → `API_ENDPOINTS.ts`
- **Types**: `PascalCase` → `User.types.ts` or `types/User.ts`

---

## TypeScript Configuration

### tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": false,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"],
      "@/hooks/*": ["src/hooks/*"],
      "@/types/*": ["src/types/*"],
      "@/utils/*": ["src/utils/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### Vite Configuration
```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  // Plugins array - add React support and other build tools
  plugins: [
    react(), // Enables React JSX transformation and Fast Refresh
  ],
  
  // Module resolution configuration
  resolve: {
    // Path aliases for cleaner imports (e.g., import '@/components/Button')
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@/components': path.resolve(__dirname, './src/components'),
      '@/hooks': path.resolve(__dirname, './src/hooks'),
      '@/utils': path.resolve(__dirname, './src/utils'),
      '@/types': path.resolve(__dirname, './src/types'),
      '@/services': path.resolve(__dirname, './src/services'),
    },
  },
  
  // Development server configuration
  server: {
    port: 3000, // Development server port
    host: true, // Listen on all local IPs (useful for mobile testing)
    open: true, // Automatically open browser on server start
    
    // Proxy API calls to backend server during development
    proxy: {
      // Proxy all /api requests to backend server
      '/api': {
        target: 'http://localhost:8000', // Backend server URL
        changeOrigin: true, // Needed for virtual hosted sites
        secure: false, // Set to true if using HTTPS backend
        // rewrite: (path) => path.replace(/^\/api/, ''), // Remove /api prefix if needed
      },
      // Proxy authentication endpoints
      '/auth': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
      },
      // Proxy WebSocket connections
      '/socket.io': {
        target: 'ws://localhost:8000',
        ws: true, // Enable WebSocket proxying
        changeOrigin: true,
      },
      // Example: Proxy to external API with different domain
      '/external-api': {
        target: 'https://api.external-service.com',
        changeOrigin: true,
        secure: true, // Use HTTPS
        rewrite: (path) => path.replace(/^\/external-api/, '/v1'),
        // Add custom headers if needed
        configure: (proxy, options) => {
          proxy.on('proxyReq', (proxyReq, req, res) => {
            proxyReq.setHeader('X-Custom-Header', 'value');
          });
        },
      },
    },
  },
  
  // Build configuration for production
  build: {
    outDir: 'dist', // Output directory for production build
    sourcemap: true, // Generate source maps for debugging
    target: 'esnext', // Modern browser target for smaller bundles
    minify: 'esbuild', // Use esbuild for faster minification
    
    // Rollup-specific options
    rollupOptions: {
      output: {
        // Manual chunk splitting for better caching
        manualChunks: {
          vendor: ['react', 'react-dom'], // Vendor libraries
          router: ['react-router-dom'], // Routing library
        },
      },
    },
    
    // Build performance optimizations
    chunkSizeWarningLimit: 1000, // Warn for chunks larger than 1MB
  },
  
  // Environment variables configuration
  define: {
    // Define global constants at build time
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
  },
  
  // CSS preprocessing options
  css: {
    // CSS modules configuration
    modules: {
      localsConvention: 'camelCase', // Convert CSS class names to camelCase
    },
    // PostCSS configuration (if using Tailwind or other processors)
    postcss: {
      plugins: [
        // Add PostCSS plugins here if needed
      ],
    },
  },
})
```

## Component Standards

### Component Structure
```typescript
// ✅ Good: Functional component with TypeScript
interface UserProfileProps {
  userId: string;
  onEdit?: (user: User) => void;
  className?: string;
}

const UserProfile: React.FC<UserProfileProps> = ({ 
  userId, 
  onEdit, 
  className = '' 
}) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  // Custom hooks
  const { data: userData } = useUser(userId);

  // Effects
  useEffect(() => {
    if (userData) {
      setUser(userData);
      setLoading(false);
    }
  }, [userData]);

  // Event handlers
  const handleEdit = useCallback(() => {
    if (user && onEdit) {
      onEdit(user);
    }
  }, [user, onEdit]);

  // Early returns
  if (loading) return <LoadingSpinner />;
  if (!user) return <ErrorMessage message="User not found" />;

  // Main render
  return (
    <div className={`user-profile ${className}`}>
      <h2>{user.name}</h2>
      <button onClick={handleEdit}>Edit Profile</button>
    </div>
  );
};

export default UserProfile;
```

### Component Best Practices

Here are the key principles we follow when building React components:

1. **Always define prop interfaces** - Clear, descriptive names make your code self-documenting
2. **Stick to functional components** - They're simpler and work better with modern React patterns
3. **Destructure props** right in the function signature for cleaner code
4. **Provide sensible defaults** for optional props to avoid undefined errors
5. **Use early returns** for loading states and error conditions - keeps the main logic clean
6. **Memoize wisely** - Use `useMemo` and `useCallback` for expensive operations, not everything
7. **Keep components focused** - One responsibility per component makes testing and maintenance easier

### Re-rendering Optimization Standards

React re-renders can kill performance if not handled properly. Here's how we keep things fast:

#### Understanding When Components Re-render
```typescript
// Components re-render when:
// 1. State changes (useState, useReducer)
// 2. Props change
// 3. Parent component re-renders
// 4. Context value changes

// ❌ Bad: This will cause unnecessary re-renders
const ParentComponent = () => {
  const [count, setCount] = useState(0);
  
  // This object is recreated on every render!
  const userConfig = { theme: 'dark', lang: 'en' };
  
  return (
    <div>
      <ExpensiveChild config={userConfig} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
};

// ✅ Good: Prevent unnecessary re-renders
const ParentComponent = () => {
  const [count, setCount] = useState(0);
  
  // Move static data outside component or use useMemo
  const userConfig = useMemo(() => ({ theme: 'dark', lang: 'en' }), []);
  
  return (
    <div>
      <ExpensiveChild config={userConfig} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
};
```

#### Using React.memo for Component Memoization
```typescript
// ✅ Good: Memoize expensive components
const ExpensiveUserCard = React.memo<UserCardProps>(({ user, onEdit }) => {
  // Heavy rendering logic here
  const processedData = useMemo(() => {
    return heavyDataProcessing(user);
  }, [user]);

  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{processedData.summary}</p>
      <button onClick={() => onEdit(user)}>Edit</button>
    </div>
  );
});

// For even more control, use custom comparison
const SmartUserCard = React.memo<UserCardProps>(
  ({ user, onEdit }) => {
    // Component implementation
  },
  // Custom comparison function
  (prevProps, nextProps) => {
    // Only re-render if user ID or name changed
    return prevProps.user.id === nextProps.user.id && 
           prevProps.user.name === nextProps.user.name;
  }
);
```

#### Optimizing Callback Functions
```typescript
// ❌ Bad: Creates new function on every render
const TodoList = ({ todos }) => {
  const [filter, setFilter] = useState('all');
  
  return (
    <div>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id} 
          todo={todo} 
          onToggle={() => toggleTodo(todo.id)} // New function every render!
        />
      ))}
    </div>
  );
};

// ✅ Good: Stable callback references
const TodoList = ({ todos }) => {
  const [filter, setFilter] = useState('all');
  
  // Memoize the callback to prevent child re-renders
  const handleToggle = useCallback((todoId: string) => {
    toggleTodo(todoId);
  }, []); // Empty dependency array since toggleTodo is stable
  
  return (
    <div>
      {todos.map(todo => (
        <MemoizedTodoItem 
          key={todo.id} 
          todo={todo} 
          onToggle={handleToggle}
        />
      ))}
    </div>
  );
};

// Don't forget to memoize the child component too
const MemoizedTodoItem = React.memo<TodoItemProps>(({ todo, onToggle }) => {
  return (
    <div>
      <span>{todo.text}</span>
      <button onClick={() => onToggle(todo.id)}>Toggle</button>
    </div>
  );
});
```

#### Context Optimization
```typescript
// ❌ Bad: Context changes cause all consumers to re-render
const AppContext = createContext<AppContextType>({});

const AppProvider = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState<Notification[]>([]);
  
  // This object is recreated on every render!
  const value = {
    user, setUser,
    theme, setTheme,
    notifications, setNotifications
  };
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
};

// ✅ Good: Split contexts and memoize values
const UserContext = createContext<UserContextType>({});
const ThemeContext = createContext<ThemeContextType>({});
const NotificationContext = createContext<NotificationContextType>({});

const AppProvider = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  
  // Memoize the context value
  const userValue = useMemo(() => ({
    user,
    setUser,
  }), [user]);
  
  return (
    <UserContext.Provider value={userValue}>
      <ThemeProvider>
        <NotificationProvider>
          {children}
        </NotificationProvider>
      </ThemeProvider>
    </UserContext.Provider>
  );
};
```

#### List Rendering Optimization
```typescript
// ✅ Good: Efficient list rendering with proper keys
const UserList = ({ users, onUserSelect }) => {
  const [searchTerm, setSearchTerm] = useState('');
  
  // Memoize filtered results
  const filteredUsers = useMemo(() => {
    return users.filter(user => 
      user.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [users, searchTerm]);
  
  // Memoize the selection handler
  const handleUserSelect = useCallback((userId: string) => {
    onUserSelect(userId);
  }, [onUserSelect]);
  
  return (
    <div>
      <input 
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search users..."
      />
      {filteredUsers.map(user => (
        <UserListItem
          key={user.id} // Use stable, unique keys
          user={user}
          onSelect={handleUserSelect}
        />
      ))}
    </div>
  );
};

// Memoized list item
const UserListItem = React.memo<UserListItemProps>(({ user, onSelect }) => {
  return (
    <div className="user-item" onClick={() => onSelect(user.id)}>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name}</span>
    </div>
  );
});
```

#### Performance Debugging Tools
```typescript
// Add these in development to track re-renders
const UserProfile = ({ userId }) => {
  // Track why component re-rendered (development only)
  if (process.env.NODE_ENV === 'development') {
    const renderCount = useRef(0);
    renderCount.current++;
    console.log(`UserProfile rendered ${renderCount.current} times`);
    
    // Track prop changes
    const prevUserId = useRef(userId);
    if (prevUserId.current !== userId) {
      console.log('UserId changed:', prevUserId.current, '->', userId);
    }
    prevUserId.current = userId;
  }
  
  // Component implementation
};

// Custom hook for debugging re-renders
const useWhyDidYouUpdate = (name: string, props: Record<string, any>) => {
  const previous = useRef<Record<string, any>>();
  
  useEffect(() => {
    if (previous.current) {
      const allKeys = Object.keys({ ...previous.current, ...props });
      const changedProps: Record<string, any> = {};
      
      allKeys.forEach(key => {
        if (previous.current![key] !== props[key]) {
          changedProps[key] = {
            from: previous.current![key],
            to: props[key]
          };
        }
      });
      
      if (Object.keys(changedProps).length) {
        console.log('[why-did-you-update]', name, changedProps);
      }
    }
    
    previous.current = props;
  });
};

// Usage
const MyComponent = (props) => {
  useWhyDidYouUpdate('MyComponent', props);
  return <div>...</div>;
};
```

### Prop Types Standards
```typescript
// ✅ Good: Comprehensive prop interface
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void;
  className?: string;
  type?: 'button' | 'submit' | 'reset';
}

// ✅ Good: Optional props with defaults
const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  onClick,
  className = '',
  type = 'button',
}) => {
  // Component implementation
};
```

## State Management

Managing state effectively is crucial for maintainable React applications. Here's how we approach it:

### Local State with useState

Keep it simple when you can. `useState` is perfect for component-level state:

```typescript
// ✅ Good: Clear, typed state with sensible defaults
const UserProfile = () => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  // Avoid multiple state updates that trigger re-renders
  const updateUserProfile = useCallback(async (updates: Partial<User>) => {
    setIsLoading(true);
    setErrors({}); // Clear previous errors
    
    try {
      const updatedUser = await updateUser(user!.id, updates);
      setUser(updatedUser);
    } catch (error) {
      setErrors({ profile: 'Failed to update profile' });
    } finally {
      setIsLoading(false);
    }
  }, [user]);
  
  return (
    // Component JSX
  );
};
```

### Complex State with useReducer

When state logic gets complex, `useReducer` keeps things organized:

```typescript
// Define your state shape clearly
interface FormState {
  values: Record<string, string>;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;
}

// Action types make debugging easier
type FormAction = 
  | { type: 'SET_FIELD'; field: string; value: string }
  | { type: 'SET_ERROR'; field: string; error: string }
  | { type: 'CLEAR_ERRORS' }
  | { type: 'SET_SUBMITTING'; isSubmitting: boolean }
  | { type: 'RESET_FORM' };

const formReducer = (state: FormState, action: FormAction): FormState => {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        values: { ...state.values, [action.field]: action.value },
        // Clear error when user starts typing
        errors: { ...state.errors, [action.field]: '' }
      };
    
    case 'SET_ERROR':
      return {
        ...state,
        errors: { ...state.errors, [action.field]: action.error }
      };
    
    case 'CLEAR_ERRORS':
      return { ...state, errors: {} };
    
    case 'SET_SUBMITTING':
      return { ...state, isSubmitting: action.isSubmitting };
    
    case 'RESET_FORM':
      return {
        values: {},
        errors: {},
        touched: {},
        isSubmitting: false
      };
    
    default:
      return state;
  }
};

// Using the reducer in a component
const ContactForm = () => {
  const [formState, dispatch] = useReducer(formReducer, {
    values: {},
    errors: {},
    touched: {},
    isSubmitting: false
  });
  
  const handleFieldChange = useCallback((field: string, value: string) => {
    dispatch({ type: 'SET_FIELD', field, value });
  }, []);
  
  return (
    // Form JSX
  );
};
```

### Context API Done Right

Context is powerful but can cause performance issues if not used carefully:

```typescript
// Create focused contexts instead of one giant context
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Custom hook with proper error handling
export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

// Provider with memoized value to prevent unnecessary re-renders
export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ 
  children 
}) => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  
  const login = useCallback(async (email: string, password: string) => {
    setIsLoading(true);
    try {
      const userData = await authService.login(email, password);
      setUser(userData);
    } catch (error) {
      throw error; // Let the component handle the error
    } finally {
      setIsLoading(false);
    }
  }, []);
  
  const logout = useCallback(() => {
    setUser(null);
    authService.clearTokens();
  }, []);
  
  // Memoize the context value to prevent unnecessary re-renders
  const value = useMemo(() => ({
    user,
    login,
    logout,
    isLoading
  }), [user, login, logout, isLoading]);
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};
```

### Global State Management

For larger applications, consider these patterns:

#### Zustand (Recommended for simplicity)
```typescript
import { create } from 'zustand';

interface UserStore {
  user: User | null;
  isLoading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

export const useUserStore = create<UserStore>((set, get) => ({
  user: null,
  isLoading: false,
  
  login: async (email, password) => {
    set({ isLoading: true });
    try {
      const user = await authService.login(email, password);
      set({ user, isLoading: false });
    } catch (error) {
      set({ isLoading: false });
      throw error;
    }
  },
  
  logout: () => {
    set({ user: null });
    authService.clearTokens();
  }
}));

// Usage in components is simple
const ProfilePage = () => {
  const { user, logout } = useUserStore();
  // Component logic
};
```

## Styling Standards

### CSS Modules (Recommended)
```typescript
// UserProfile.module.css
.container {
  padding: 1rem;
  border-radius: 8px;
}

.title {
  font-size: 1.5rem;
  font-weight: 600;
}

// UserProfile.tsx
import styles from './UserProfile.module.css';

const UserProfile: React.FC = () => (
  <div className={styles.container}>
    <h2 className={styles.title}>Profile</h2>
  </div>
);
```

### Styled Components (Alternative)
```typescript
import styled from 'styled-components';

const Container = styled.div<{ variant?: 'primary' | 'secondary' }>`
  padding: 1rem;
  background-color: ${props => 
    props.variant === 'primary' ? '#007bff' : '#6c757d'
  };
`;

const UserProfile: React.FC<{ variant?: 'primary' | 'secondary' }> = ({ 
  variant 
}) => (
  <Container variant={variant}>
    {/* Content */}
  </Container>
);
```

## Testing Standards

### Testing Setup
```json
// package.json
{
  "devDependencies": {
    "@testing-library/react": "^13.0.0",
    "@testing-library/jest-dom": "^5.0.0",
    "@testing-library/user-event": "^14.0.0",
    "jsdom": "^20.0.0",
    "vitest": "^1.0.0"
  }
}
```

### Component Testing
```typescript
// __tests__/UserProfile.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { vi } from 'vitest';
import UserProfile from '../UserProfile';

describe('UserProfile', () => {
  const mockProps = {
    userId: '123',
    onEdit: vi.fn(),
  };

  it('renders user profile correctly', () => {
    render(<UserProfile {...mockProps} />);
    
    expect(screen.getByText('Profile')).toBeInTheDocument();
  });

  it('calls onEdit when edit button is clicked', async () => {
    render(<UserProfile {...mockProps} />);
    
    const editButton = screen.getByRole('button', { name: /edit/i });
    fireEvent.click(editButton);
    
    expect(mockProps.onEdit).toHaveBeenCalledTimes(1);
  });
});
```

### Custom Hook Testing
```typescript
// __tests__/useUser.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { useUser } from '../hooks/useUser';

describe('useUser', () => {
  it('fetches user data successfully', async () => {
    const { result } = renderHook(() => useUser('123'));
    
    await waitFor(() => {
      expect(result.current.data).toBeDefined();
      expect(result.current.loading).toBe(false);
    });
  });
});
```

## Code Quality & Linting

### ESLint Configuration
```json
// .eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["react-refresh"],
  "rules": {
    "react-refresh/only-export-components": [
      "warn",
      { "allowConstantExport": true }
    ],
    "@typescript-eslint/no-unused-vars": "error",
    "react/prop-types": "off",
    "react/react-in-jsx-scope": "off"
  }
}
```

### Prettier Configuration
```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

## Performance Standards

Performance isn't just about fast loading—it's about smooth user experiences. Here's how we keep our React apps snappy:

### Smart Code Splitting

Don't make users download code they don't need right away:

```typescript
// ✅ Route-based code splitting - the most impactful
import { lazy, Suspense } from 'react';

// These components are loaded only when routes are visited
const UserProfile = lazy(() => import('./pages/UserProfile'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const AdminPanel = lazy(() => 
  import('./pages/AdminPanel').then(module => ({
    default: module.AdminPanel // Handle named exports
  }))
);

// Nice loading fallback
const PageLoader = () => (
  <div className="page-loader">
    <div className="spinner" />
    <p>Loading...</p>
  </div>
);

const App: React.FC = () => (
  <Router>
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/profile" element={<UserProfile />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/admin" element={<AdminPanel />} />
      </Routes>
    </Suspense>
  </Router>
);

// ✅ Component-based splitting for heavy components
const HeavyDataTable = lazy(() => import('./components/HeavyDataTable'));

const ReportsPage = () => {
  const [showTable, setShowTable] = useState(false);
  
  return (
    <div>
      <h1>Reports</h1>
      <button onClick={() => setShowTable(true)}>
        Load Data Table
      </button>
      
      {showTable && (
        <Suspense fallback={<div>Loading table...</div>}>
          <HeavyDataTable />
        </Suspense>
      )}
    </div>
  );
};
```

### Effective Memoization Strategies

Memoization is powerful, but use it wisely—overuse can actually hurt performance:

```typescript
// ✅ Good: Memoize expensive calculations
const DataVisualization: React.FC<{ rawData: DataPoint[] }> = ({ rawData }) => {
  // This calculation only runs when rawData changes
  const processedData = useMemo(() => {
    console.log('Processing data...'); // You'll see this less often
    return rawData
      .filter(point => point.isValid)
      .map(point => ({
        ...point,
        normalizedValue: point.value / 100
      }))
      .sort((a, b) => b.normalizedValue - a.normalizedValue);
  }, [rawData]);
  
  // Stable callback reference prevents child re-renders
  const handleDataPointClick = useCallback((pointId: string) => {
    console.log('Data point clicked:', pointId);
    // Handle click logic
  }, []); // Empty deps because this function doesn't depend on anything
  
  return (
    <div className="visualization">
      {processedData.map(point => (
        <DataPoint 
          key={point.id}
          data={point}
          onClick={handleDataPointClick}
        />
      ))}
    </div>
  );
};

// Don't memoize everything - this is overkill
const SimpleCounter = () => {
  const [count, setCount] = useState(0);
  
  // ❌ Bad: No need to memoize this simple calculation
  const doubledCount = useMemo(() => count * 2, [count]);
  
  // ✅ Good: Just calculate it directly
  const tripleCount = count * 3;
  
  return <div>Count: {count}, Triple: {tripleCount}</div>;
};
```

### Virtual Scrolling for Large Lists

When you have thousands of items, don't render them all:

```typescript
// Using react-window for efficient list rendering
import { FixedSizeList as List } from 'react-window';

interface VirtualListProps {
  items: ListItem[];
  onItemClick: (item: ListItem) => void;
}

const VirtualList: React.FC<VirtualListProps> = ({ items, onItemClick }) => {
  // Each row component
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => {
    const item = items[index];
    
    return (
      <div style={style} className="list-item" onClick={() => onItemClick(item)}>
        <h3>{item.title}</h3>
        <p>{item.description}</p>
      </div>
    );
  };
  
  return (
    <List
      height={400} // Container height
      itemCount={items.length}
      itemSize={80} // Height of each item
      width="100%"
    >
      {Row}
    </List>
  );
};
```

### Image and Asset Optimization

Images often make up most of your app's size:

```typescript
// ✅ Smart image loading component
interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  lazy?: boolean;
}

const OptimizedImage: React.FC<OptimizedImageProps> = ({
  src,
  alt,
  width,
  height,
  lazy = true
}) => {
  const [isLoaded, setIsLoaded] = useState(false);
  const [hasError, setHasError] = useState(false);
  
  return (
    <div className="image-container">
      {!isLoaded && !hasError && (
        <div className="image-placeholder">Loading...</div>
      )}
      
      {hasError ? (
        <div className="image-error">Failed to load image</div>
      ) : (
        <img
          src={src}
          alt={alt}
          width={width}
          height={height}
          loading={lazy ? "lazy" : "eager"}
          onLoad={() => setIsLoaded(true)}
          onError={() => setHasError(true)}
          style={{ display: isLoaded ? 'block' : 'none' }}
        />
      )}
    </div>
  );
};

// For hero images or critical content, preload them
const HeroSection = () => {
  useEffect(() => {
    // Preload critical images
    const heroImage = new Image();
    heroImage.src = '/hero-image.jpg';
  }, []);
  
  return (
    <section className="hero">
      <OptimizedImage 
        src="/hero-image.jpg"
        alt="Hero banner"
        lazy={false} // Don't lazy load above-the-fold images
      />
    </section>
  );
};
```

### Bundle Size Monitoring

Keep an eye on what's actually shipped to users:

```typescript
// Use webpack-bundle-analyzer to visualize your bundle
// Add this to your build script in package.json:
// "analyze": "npm run build && npx vite-bundle-analyzer dist"

// Tree-shake unused code by importing only what you need
// ❌ Bad: Imports entire library
import _ from 'lodash';

// ✅ Good: Import specific functions
import { debounce, throttle } from 'lodash';

// Or even better, use smaller alternatives
import { debounce } from 'lodash-es'; // ES modules version
```

## Security Standards

Security should be baked into your development process, not added as an afterthought.

### Environment Variables & Configuration

Keep sensitive data out of your code:

```typescript
// ✅ Good: Validate environment variables at startup
interface AppConfig {
  apiUrl: string;
  apiKey: string;
  environment: 'development' | 'staging' | 'production';
}

const validateConfig = (): AppConfig => {
  const config = {
    apiUrl: import.meta.env.VITE_API_URL,
    apiKey: import.meta.env.VITE_API_KEY,
    environment: import.meta.env.VITE_ENV || 'development',
  };

  // Fail fast if required config is missing
  if (!config.apiUrl) {
    throw new Error('VITE_API_URL is required');
  }
  
  if (!config.apiKey && config.environment === 'production') {
    throw new Error('VITE_API_KEY is required in production');
  }

  return config as AppConfig;
};

// Use it in your app
export const appConfig = validateConfig();

// Create a config hook for components
export const useAppConfig = () => appConfig;
```

### XSS Prevention

Never trust user input, especially when rendering HTML:

```typescript
// ✅ Good: Sanitizing HTML content
import DOMPurify from 'dompurify';

interface SafeHTMLProps {
  htmlContent: string;
  allowedTags?: string[];
}

const SafeHTML: React.FC<SafeHTMLProps> = ({ 
  htmlContent, 
  allowedTags = ['p', 'br', 'strong', 'em'] 
}) => {
  const sanitizedHTML = useMemo(() => {
    return DOMPurify.sanitize(htmlContent, {
      ALLOWED_TAGS: allowedTags,
      ALLOWED_ATTR: ['href', 'title'], // Only allow safe attributes
    });
  }, [htmlContent, allowedTags]);
  
  return <div dangerouslySetInnerHTML={{ __html: sanitizedHTML }} />;
};

// ❌ Bad: Never do this with user input
const UnsafeComponent = ({ userContent }: { userContent: string }) => {
  return <div dangerouslySetInnerHTML={{ __html: userContent }} />; // XSS vulnerability!
};

// ✅ Good: Escape user input by default
const SafeUserContent = ({ userContent }: { userContent: string }) => {
  return <div>{userContent}</div>; // React automatically escapes this
};
```

### API Security Best Practices

```typescript
// ✅ Good: Secure API service with token management
class ApiService {
  private baseUrl: string;
  private tokenKey = 'auth-token';

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  private getAuthToken(): string | null {
    return localStorage.getItem(this.tokenKey);
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.baseUrl}${endpoint}`;
    const token = this.getAuthToken();

    const config: RequestInit = {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...(token && { Authorization: `Bearer ${token}` }),
        ...options.headers,
      },
    };

    const response = await fetch(url, config);

    // Handle common security issues
    if (response.status === 401) {
      this.clearAuthToken(); // Clear invalid token
      window.location.href = '/login';
      throw new Error('Authentication required');
    }

    if (response.status === 403) {
      throw new Error('Access forbidden');
    }

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json();
  }

  public async get<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: 'GET' });
  }

  public async post<T>(endpoint: string, data: unknown): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  public setAuthToken(token: string): void {
    localStorage.setItem(this.tokenKey, token);
  }

  public clearAuthToken(): void {
    localStorage.removeItem(this.tokenKey);
  }
}
```

### Content Security Policy (CSP)

Add security headers to your production builds:

```html
<!-- Add to your index.html for production -->
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  script-src 'self' 'unsafe-inline';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self' https://fonts.googleapis.com https://fonts.gstatic.com;
  connect-src 'self' https://api.yourdomain.com;
">
```

### Secure Form Handling

```typescript
// ✅ Good: Secure form with validation
interface LoginFormData {
  email: string;
  password: string;
}

const LoginForm = () => {
  const [formData, setFormData] = useState<LoginFormData>({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState<Partial<LoginFormData>>({});
  
  const validateForm = (data: LoginFormData): Partial<LoginFormData> => {
    const errors: Partial<LoginFormData> = {};
    
    // Validate email
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(data.email)) {
      errors.email = 'Please enter a valid email address';
    }
    
    // Validate password strength
    if (data.password.length < 8) {
      errors.password = 'Password must be at least 8 characters long';
    }
    
    return errors;
  };
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    const validationErrors = validateForm(formData);
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }
    
    try {
      // Clear form data from memory after submission
      await authService.login(formData.email, formData.password);
      setFormData({ email: '', password: '' }); // Clear sensitive data
    } catch (error) {
      setErrors({ password: 'Invalid credentials' });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields with proper validation */}
    </form>
  );
};
```

## Documentation Standards

### Component Documentation
```typescript
/**
 * UserProfile component displays user information and allows editing
 * 
 * @param userId - The unique identifier for the user
 * @param onEdit - Callback function called when edit button is clicked
 * @param className - Additional CSS classes to apply
 * 
 * @example
 * ```tsx
 * <UserProfile 
 *   userId="123" 
 *   onEdit={(user) => console.log(user)}
 *   className="my-profile"
 * />
 * ```
 */
const UserProfile: React.FC<UserProfileProps> = ({ 
  userId, 
  onEdit, 
  className 
}) => {
  // Implementation
};
```

### README Template
```markdown
# Project Name

## Getting Started

### Prerequisites
- Node.js 18+
- npm or yarn

### Installation
```bash
npm install
npm run dev
```

### Available Scripts
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build
- `npm run test` - Run tests
- `npm run lint` - Run ESLint

## Project Structure
[Describe your project structure]

## Contributing
[Contribution guidelines]
```

## Git Standards

### Commit Message Format
```
type(scope): description

Types:
- feat: New feature
- fix: Bug fix
- docs: Documentation changes
- style: Code style changes
- refactor: Code refactoring
- test: Test changes
- chore: Build process or auxiliary tool changes

Example:
feat(auth): add login functionality
fix(button): resolve disabled state styling
docs(readme): update installation instructions
```

### Branch Naming
- `feature/feature-name`
- `bugfix/bug-description`
- `hotfix/critical-fix`
- `chore/maintenance-task`

## Deployment Standards

### Build Optimization
```typescript
// vite.config.ts - Production build optimization
export default defineConfig({
  build: {
    // Output directory for production files
    outDir: 'dist',
    
    // Generate source maps for production debugging
    sourcemap: true,
    
    // Target modern browsers for smaller bundles
    target: 'esnext',
    
    // Rollup-specific build options
    rollupOptions: {
      output: {
        // Manual chunk splitting for better caching strategy
        manualChunks: {
          // Separate vendor libraries for better caching
          vendor: ['react', 'react-dom'],
          // Routing libraries in separate chunk
          router: ['react-router-dom'],
          // UI libraries in separate chunk (if using)
          ui: ['@mui/material', 'antd'],
          // Utility libraries
          utils: ['lodash', 'date-fns'],
        },
        
        // Custom chunk file naming
        chunkFileNames: 'assets/js/[name]-[hash].js',
        entryFileNames: 'assets/js/[name]-[hash].js',
        assetFileNames: 'assets/[ext]/[name]-[hash].[ext]',
      },
      
      // External dependencies (won't be bundled)
      external: [
        // Add external libraries here if needed
      ],
    },
    
    // Compression and minification settings
    minify: 'esbuild', // Fast minification with esbuild
    
    // Bundle analysis and warnings
    chunkSizeWarningLimit: 1000, // Warn for chunks > 1MB
    reportCompressedSize: false, // Disable to speed up build
    
    // Asset handling
    assetsInlineLimit: 4096, // Inline assets smaller than 4KB
  },
  
  // Preview server configuration (for production build testing)
  preview: {
    port: 4173,
    host: true,
    open: true,
  },
});
```

### Environment Configuration
```bash
# .env.example
VITE_API_URL=https://api.example.com
VITE_APP_NAME=My React App
VITE_VERSION=1.0.0
```

---

*Last updated: July 2025*