# React + TypeScript + Vite Development Standards

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

## Project Setup

### Initial Setup
```bash
# Create new Vite + React + TypeScript project
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
```

### Required Dependencies
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

## Project Structure

### Recommended Directory Structure
```
src/
├── components/           # Reusable UI components
│   ├── ui/              # Basic UI components (Button, Input, etc.)
│   ├── forms/           # Form-specific components
│   └── layout/          # Layout components (Header, Footer, etc.)
├── pages/               # Page components/views
├── hooks/               # Custom React hooks
├── contexts/            # React contexts
├── services/            # API calls and external services
├── utils/               # Utility functions
├── types/               # TypeScript type definitions
├── constants/           # Application constants
├── assets/              # Static assets (images, icons, etc.)
├── styles/              # Global styles and themes
├── __tests__/           # Test files
├── App.tsx
├── main.tsx
└── vite-env.d.ts
```

### File Naming Conventions
- **Components**: PascalCase (`UserProfile.tsx`)
- **Hooks**: camelCase with 'use' prefix (`useUserData.ts`)
- **Utilities**: camelCase (`formatDate.ts`)
- **Constants**: SCREAMING_SNAKE_CASE (`API_ENDPOINTS.ts`)
- **Types**: PascalCase (`User.types.ts` or `types/User.ts`)

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
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 3000,
    open: true,
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
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
1. **Always define prop interfaces** with clear, descriptive names
2. **Use functional components** with hooks instead of class components
3. **Destructure props** in the function signature
4. **Provide default values** for optional props
5. **Use early returns** for loading and error states
6. **Memoize expensive operations** with `useMemo` and `useCallback`
7. **Keep components small and focused** (single responsibility)

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

### Local State (useState)
```typescript
// ✅ Good: Typed state with clear initial values
const [user, setUser] = useState<User | null>(null);
const [loading, setLoading] = useState<boolean>(false);
const [errors, setErrors] = useState<Record<string, string>>({});

// ✅ Good: Complex state with useReducer
interface FormState {
  values: Record<string, string>;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;
}

const formReducer = (state: FormState, action: FormAction): FormState => {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        values: { ...state.values, [action.field]: action.value }
      };
    default:
      return state;
  }
};
```

### Context API
```typescript
// ✅ Good: Typed context with provider
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  loading: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ 
  children 
}) => {
  // Implementation
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

### Code Splitting
```typescript
// ✅ Good: Lazy loading components
const LazyUserProfile = lazy(() => import('./UserProfile'));
const LazyDashboard = lazy(() => import('./Dashboard'));

const App: React.FC = () => (
  <Router>
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/profile" element={<LazyUserProfile />} />
        <Route path="/dashboard" element={<LazyDashboard />} />
      </Routes>
    </Suspense>
  </Router>
);
```

### Memoization
```typescript
// ✅ Good: Memoizing expensive calculations
const ExpensiveComponent: React.FC<{ items: Item[] }> = ({ items }) => {
  const expensiveValue = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);

  const handleClick = useCallback((id: string) => {
    // Handle click logic
  }, []);

  return <div>{expensiveValue}</div>;
};
```

## Security Standards

### Environment Variables
```typescript
// ✅ Good: Environment variable validation
const config = {
  apiUrl: import.meta.env.VITE_API_URL || 'http://localhost:3001',
  apiKey: import.meta.env.VITE_API_KEY,
};

if (!config.apiKey) {
  throw new Error('VITE_API_KEY is required');
}
```

### XSS Prevention
```typescript
// ✅ Good: Sanitizing user input
import DOMPurify from 'dompurify';

const SafeHTML: React.FC<{ htmlContent: string }> = ({ htmlContent }) => {
  const sanitizedHTML = DOMPurify.sanitize(htmlContent);
  
  return <div dangerouslySetInnerHTML={{ __html: sanitizedHTML }} />;
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
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
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