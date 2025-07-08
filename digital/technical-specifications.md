# 💻 Technical Specifications

> *Technology in Service of Transformation*

## Architecture Overview

### System Design Philosophy
- **Human-centered**: Technology serves consciousness, not ego
- **Scalable**: Built for growth while maintaining intimacy
- **Accessible**: Inclusive design for all users
- **Sustainable**: Efficient, environmentally conscious code

### Technology Stack

#### Frontend
- **Framework**: Next.js 14+ (React 18+)
- **Styling**: Tailwind CSS 3.x with custom Catalyst theme
- **State Management**: Zustand for global state, React Query for server state
- **TypeScript**: 5.x for type safety
- **Testing**: Jest + React Testing Library

#### Backend
- **Runtime**: Node.js 20 LTS
- **Framework**: Express.js 4.x
- **API**: GraphQL (Apollo Server) + REST endpoints
- **Database**: PostgreSQL 15+ with Prisma ORM
- **Cache**: Redis 7.x
- **Authentication**: Auth0

#### Infrastructure
- **Hosting**: Vercel (frontend), Railway (backend)
- **CDN**: Cloudflare for global performance
- **Storage**: AWS S3 for media
- **Email**: SendGrid for transactional emails
- **Analytics**: Mixpanel for events, Google Analytics for basics

---

## Database Schema

### Core Entities

#### Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  username VARCHAR(50) UNIQUE,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  avatar_url TEXT,
  bio TEXT,
  location VARCHAR(255),
  consciousness_level INTEGER DEFAULT 1,
  chain_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Products Table
```sql
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sku VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  philosophy TEXT,
  story TEXT,
  price_cents INTEGER NOT NULL,
  category VARCHAR(100),
  collection VARCHAR(100),
  is_active BOOLEAN DEFAULT true,
  has_reversed_text BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Orders Table
```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  order_number VARCHAR(50) UNIQUE NOT NULL,
  status VARCHAR(50) DEFAULT 'pending',
  total_cents INTEGER NOT NULL,
  gift_message TEXT,
  tracking_code VARCHAR(100) UNIQUE,
  shipped_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Gift_Chains Table
```sql
CREATE TABLE gift_chains (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  originator_order_id UUID REFERENCES orders(id),
  current_holder_id UUID REFERENCES users(id),
  product_id UUID REFERENCES products(id),
  chain_position INTEGER DEFAULT 1,
  tracking_code VARCHAR(100) UNIQUE,
  story TEXT,
  location_given VARCHAR(255),
  given_at TIMESTAMP,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### Community_Posts Table
```sql
CREATE TABLE community_posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  type VARCHAR(50), -- 'transformation', 'mirror_moment', 'chain_story'
  title VARCHAR(255),
  content TEXT NOT NULL,
  image_urls TEXT[],
  is_featured BOOLEAN DEFAULT false,
  like_count INTEGER DEFAULT 0,
  comment_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## API Specifications

### GraphQL Schema

#### User Operations
```graphql
type User {
  id: ID!
  email: String!
  username: String
  firstName: String
  lastName: String
  avatarUrl: String
  bio: String
  location: String
  consciousnessLevel: Int!
  chainCount: Int!
  orders: [Order!]!
  communityPosts: [CommunityPost!]!
  giftChains: [GiftChain!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Query {
  me: User
  user(id: ID!): User
  users(limit: Int = 10, offset: Int = 0): [User!]!
}

type Mutation {
  updateProfile(input: UpdateProfileInput!): User!
  deleteAccount: Boolean!
}
```

#### Product Operations
```graphql
type Product {
  id: ID!
  sku: String!
  name: String!
  description: String
  philosophy: String
  story: String
  priceCents: Int!
  category: String
  collection: String
  isActive: Boolean!
  hasReversedText: Boolean!
  variants: [ProductVariant!]!
  images: [ProductImage!]!
  reviews: [Review!]!
  createdAt: DateTime!
}

type Query {
  products(
    category: String
    collection: String
    isActive: Boolean = true
    limit: Int = 20
    offset: Int = 0
  ): [Product!]!
  product(id: ID, sku: String): Product
}
```

#### Order & Gift Chain Operations
```graphql
type Order {
  id: ID!
  orderNumber: String!
  user: User!
  items: [OrderItem!]!
  status: OrderStatus!
  totalCents: Int!
  giftMessage: String
  trackingCode: String
  giftChain: GiftChain
  shippedAt: DateTime
  createdAt: DateTime!
}

type GiftChain {
  id: ID!
  originatorOrder: Order!
  currentHolder: User
  product: Product!
  chainPosition: Int!
  trackingCode: String!
  story: String
  locationGiven: String
  givenAt: DateTime
  isActive: Boolean!
  history: [GiftChainEvent!]!
}

type Mutation {
  createOrder(input: CreateOrderInput!): Order!
  continueGiftChain(input: ContinueChainInput!): GiftChain!
  addChainStory(chainId: ID!, story: String!): GiftChain!
}
```

### REST Endpoints

#### Payment Processing
```typescript
// POST /api/payments/create-intent
interface CreatePaymentIntentRequest {
  productIds: string[];
  quantities: number[];
  giftMessage?: string;
  shippingAddress: Address;
}

interface CreatePaymentIntentResponse {
  clientSecret: string;
  orderId: string;
  amount: number;
}
```

#### Chain Tracking
```typescript
// GET /api/chains/track/:trackingCode
interface ChainTrackingResponse {
  chain: {
    id: string;
    product: Product;
    currentPosition: number;
    totalPositions: number;
    lastLocation?: string;
    lastGivenAt?: string;
    story?: string;
  };
  publicHistory: ChainEvent[];
}
```

---

## Frontend Specifications

### Component Architecture

#### Design System Components
```typescript
// Button Component with Catalyst styling
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'ghost' | 'mirror';
  size: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
  leftIcon?: ReactNode;
  rightIcon?: ReactNode;
  children: ReactNode;
}

// Mirror Text Component for reversed text
interface MirrorTextProps {
  text: string;
  size: 'sm' | 'md' | 'lg' | 'xl';
  weight: 'normal' | 'medium' | 'bold';
  className?: string;
}
```

#### Page Components
```typescript
// Product Detail Page
interface ProductPageProps {
  product: Product;
  variants: ProductVariant[];
  reviews: Review[];
  relatedProducts: Product[];
}

// Community Page
interface CommunityPageProps {
  posts: CommunityPost[];
  featuredStories: Story[];
  userStats: UserStats;
}
```

### State Management

#### Global State (Zustand)
```typescript
interface AppState {
  // User state
  user: User | null;
  isAuthenticated: boolean;
  
  // Cart state
  cart: CartItem[];
  cartTotal: number;
  
  // UI state
  isLoading: boolean;
  notifications: Notification[];
  
  // Actions
  setUser: (user: User | null) => void;
  addToCart: (item: CartItem) => void;
  removeFromCart: (itemId: string) => void;
  clearCart: () => void;
  addNotification: (notification: Notification) => void;
}
```

#### Server State (React Query)
```typescript
// Custom hooks for data fetching
const useProducts = (filters?: ProductFilters) => {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => fetchProducts(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

const useGiftChain = (trackingCode: string) => {
  return useQuery({
    queryKey: ['giftChain', trackingCode],
    queryFn: () => fetchGiftChain(trackingCode),
    enabled: !!trackingCode,
  });
};
```

---

## Performance Specifications

### Core Web Vitals Targets
- **Largest Contentful Paint (LCP)**: < 2.5s
- **First Input Delay (FID)**: < 100ms
- **Cumulative Layout Shift (CLS)**: < 0.1
- **Time to Interactive (TTI)**: < 3.5s

### Optimization Strategies

#### Image Optimization
```typescript
// Next.js Image component configuration
const CatalystImage: FC<ImageProps> = ({
  src,
  alt,
  priority = false,
  ...props
}) => (
  <Image
    src={src}
    alt={alt}
    priority={priority}
    quality={85}
    placeholder="blur"
    blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/..."
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
    {...props}
  />
);
```

#### Code Splitting
```typescript
// Route-based code splitting
const ProductPage = lazy(() => import('./pages/ProductPage'));
const CommunityPage = lazy(() => import('./pages/CommunityPage'));
const ChainTracker = lazy(() => import('./pages/ChainTracker'));

// Component-based code splitting
const VideoPlayer = lazy(() => import('./components/VideoPlayer'));
```

#### Caching Strategy
```typescript
// API response caching
const cacheConfig = {
  // Static content: 1 year
  static: 'public, max-age=31536000, immutable',
  
  // Product data: 1 hour
  products: 'public, max-age=3600, s-maxage=7200',
  
  // User data: no cache
  user: 'private, no-cache, no-store, must-revalidate',
  
  // Community content: 5 minutes
  community: 'public, max-age=300, s-maxage=600'
};
```

---

## Security Specifications

### Authentication & Authorization
```typescript
// JWT token structure
interface CatalystJWT {
  sub: string; // user ID
  email: string;
  role: 'user' | 'admin' | 'moderator';
  consciousnessLevel: number;
  iat: number;
  exp: number;
}

// Route protection middleware
const requireAuth = (minimumLevel?: number) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const token = extractToken(req);
    const decoded = verifyToken(token);
    
    if (!decoded) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    if (minimumLevel && decoded.consciousnessLevel < minimumLevel) {
      return res.status(403).json({ error: 'Insufficient consciousness level' });
    }
    
    req.user = decoded;
    next();
  };
};
```

### Data Protection
```typescript
// Personal data handling
interface DataProtectionConfig {
  // PII encryption
  encryptedFields: ['email', 'firstName', 'lastName', 'location'];
  
  // Data retention
  retentionPeriods: {
    userProfiles: '7 years',
    orderHistory: '7 years',
    communityPosts: 'indefinite', // user controlled
    analytics: '26 months'
  };
  
  // GDPR compliance
  userRights: {
    dataExport: true,
    dataCorrection: true,
    dataDeletion: true,
    processingOptOut: true
  };
}
```

### Input Validation
```typescript
// API input schemas using Zod
const createOrderSchema = z.object({
  productIds: z.array(z.string().uuid()),
  quantities: z.array(z.number().positive().int()),
  shippingAddress: z.object({
    street1: z.string().min(1).max(100),
    street2: z.string().max(100).optional(),
    city: z.string().min(1).max(50),
    state: z.string().min(2).max(3),
    zipCode: z.string().regex(/^\d{5}(-\d{4})?$/),
    country: z.string().length(2)
  }),
  giftMessage: z.string().max(500).optional()
});
```

---

## Testing Specifications

### Testing Strategy
```typescript
// Unit tests for utility functions
describe('mirrorText utility', () => {
  it('should reverse text for mirror reading', () => {
    expect(mirrorText('You are the Catalyst')).toBe('tsylatac eht era uoY');
  });
  
  it('should handle empty strings', () => {
    expect(mirrorText('')).toBe('');
  });
});

// Integration tests for API endpoints
describe('POST /api/orders', () => {
  it('should create order with gift chain', async () => {
    const response = await request(app)
      .post('/api/orders')
      .send(validOrderData)
      .expect(201);
      
    expect(response.body.giftChain).toBeDefined();
    expect(response.body.giftChain.trackingCode).toMatch(/^[A-Z0-9]{8}$/);
  });
});

// End-to-end tests for critical user flows
describe('Purchase Flow', () => {
  it('should complete full Get/Give purchase', () => {
    cy.visit('/products/cat-001');
    cy.get('[data-testid="add-to-cart"]').click();
    cy.get('[data-testid="checkout"]').click();
    cy.get('[data-testid="gift-message"]').type('Found you for a reason');
    cy.get('[data-testid="complete-order"]').click();
    cy.url().should('include', '/order-confirmation');
    cy.contains('Chain Reaction Started').should('be.visible');
  });
});
```

### Performance Testing
```typescript
// Load testing configuration
const loadTestConfig = {
  scenarios: {
    // Normal traffic
    baseline: {
      executor: 'constant-vus',
      vus: 10,
      duration: '5m'
    },
    
    // Product launch traffic
    spike: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '2m', target: 50 },
        { duration: '5m', target: 50 },
        { duration: '2m', target: 100 },
        { duration: '5m', target: 100 },
        { duration: '2m', target: 0 }
      ]
    }
  },
  
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.01']
  }
};
```

---

## Deployment & DevOps

### CI/CD Pipeline
```yaml
# .github/workflows/deploy.yml
name: Deploy Catalyst
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm run test:unit
      - run: npm run test:integration
      
      - name: E2E Tests
        run: npm run test:e2e
        env:
          CYPRESS_baseUrl: ${{ secrets.PREVIEW_URL }}
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to Production
        run: |
          curl -X POST "${{ secrets.VERCEL_DEPLOY_HOOK }}"
```

### Monitoring & Observability
```typescript
// Error tracking setup
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  
  // Custom tags for Catalyst-specific tracking
  initialScope: {
    tags: {
      component: 'catalyst-frontend',
      version: process.env.APP_VERSION
    }
  },
  
  // Performance monitoring
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  
  // Custom error filtering
  beforeSend(event) {
    // Don't send user input validation errors
    if (event.exception?.values?.[0]?.type === 'ValidationError') {
      return null;
    }
    return event;
  }
});
```

---

*"Technology should be invisible, empowering the human experience without drawing attention to itself. Every technical choice serves consciousness and connection."*

---

*Technical Specifications Last Updated: July 2025*
*Next Review: Monthly with technology updates*