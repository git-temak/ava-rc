# Goals App - Technical Documentation

## Table of Contents
1. [Technology Stack](#technology-stack)
2. [System Architecture](#system-architecture)
3. [Database Schema](#database-schema)
4. [API Architecture](#api-architecture)
5. [Authentication & Security](#authentication--security)
6. [RevenueCat Implementation](#revenuecat-implementation)
7. [Deployment & Infrastructure](#deployment--infrastructure)
8. [Future Enhancements](#future-enhancements)

---

## Technology Stack

### Frontend - Mobile Application

**Flutter (Dart)**
- **Version**: Flutter 3.0+, Dart 3.0+
- **UI Framework**: Material Design 3
- **State Management**: Provider pattern
- **Key Dependencies**:
  - `provider` (^6.1.1) - State management
  - `http` (^1.1.2) - HTTP client for API calls
  - `shared_preferences` (^2.2.2) - Local data persistence
  - `go_router` (^12.1.3) - Navigation and routing
  - `google_fonts` (^6.1.0) - Typography
  - `webview_flutter` (^4.4.2) - In-app web content display
  - `intl` (^0.18.1) - Internationalization and date formatting

**Why Flutter:**
- Cross-platform development (iOS & Android from single codebase)
- Native performance and smooth animations
- Rich widget ecosystem for beautiful UIs
- Strong developer community and documentation
- Cost-effective for MVP and scaling

### Backend - API Server

**Node.js with Express.js**
- **Runtime**: Node.js 18+ LTS
- **Framework**: Express.js 4.x
- **Key Dependencies**:
  - `express` (^4.18.2) - Web framework
  - `mongoose` (^7.0.3) - MongoDB ODM
  - `bcryptjs` (^2.4.3) - Password hashing
  - `jsonwebtoken` (^9.0.0) - JWT authentication
  - `cors` (^2.8.5) - Cross-origin resource sharing
  - `dotenv` (^16.0.3) - Environment variable management

**Why Node.js:**
- JavaScript full-stack for consistency
- Excellent for I/O-intensive operations
- Large ecosystem (npm)
- Easy horizontal scaling
- Fast development iteration

### Database

**MongoDB Atlas**
- **Type**: NoSQL document database (cloud-hosted)
- **Version**: MongoDB 6.0+
- **Hosting**: MongoDB Atlas (managed service)

**Why MongoDB:**
- Flexible schema for evolving features
- Excellent for user-generated content
- Horizontal scalability
- Rich query capabilities
- Cloud-native with Atlas

### Additional Services

**RevenueCat**
- In-app purchase and subscription management
- Cross-platform receipt validation
- Subscription analytics and insights

**Future Integrations:**
- **Firebase Cloud Messaging** - Push notifications
- **Sentry** - Error tracking and monitoring
- **Google Analytics / Mixpanel** - User analytics
- **OpenAI API** - AI-powered action generation

---

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Flutter Mobile App                    │
│  ┌──────────┐  ┌───────────┐  ┌─────────────────────┐  │
│  │   Auth   │  │   Dreams  │  │   Daily Actions     │  │
│  │  Provider│  │  Provider │  │      Provider       │  │
│  └──────────┘  └───────────┘  └─────────────────────┘  │
│         │              │                   │             │
│         └──────────────┴───────────────────┘             │
│                        │                                 │
│                 ┌──────▼──────┐                          │
│                 │  API Service │                         │
│                 └──────┬──────┘                          │
└────────────────────────┼──────────────────────────────────┘
                         │ HTTP/REST
                         │
        ┌────────────────▼────────────────┐
        │     Node.js/Express Backend     │
        │                                  │
        │  ┌──────────┐  ┌─────────────┐ │
        │  │   Auth   │  │   Dreams    │ │
        │  │  Routes  │  │   Routes    │ │
        │  └──────────┘  └─────────────┘ │
        │  ┌──────────┐  ┌─────────────┐ │
        │  │  Actions │  │    Users    │ │
        │  │  Routes  │  │   Routes    │ │
        │  └──────────┘  └─────────────┘ │
        │         │              │        │
        │         └──────┬───────┘        │
        └────────────────┼────────────────┘
                         │
                ┌────────▼─────────┐
                │  MongoDB Atlas   │
                │                  │
                │  ┌────────────┐  │
                │  │   Users    │  │
                │  ├────────────┤  │
                │  │   Dreams   │  │
                │  ├────────────┤  │
                │  │   Goals    │  │
                │  ├────────────┤  │
                │  │  Actions   │  │
                │  └────────────┘  │
                └──────────────────┘

        ┌─────────────────────┐
        │     RevenueCat      │
        │  (Subscription Mgmt)│
        └─────────────────────┘
```

### Architecture Patterns

**Client-Server Model**
- Flutter app communicates with REST API
- Stateless API design for scalability
- JWT-based authentication

**MVC on Backend**
- **Models**: Mongoose schemas for data structure
- **Controllers**: Business logic and data processing
- **Routes**: HTTP endpoint definitions

**Provider Pattern on Frontend**
- Centralized state management
- Reactive UI updates
- Separation of business logic from UI

---

## Database Schema

### Users Collection

```javascript
{
  _id: ObjectId,
  name: String,           // User's full name
  email: String,          // Unique email (used for login)
  password: String,       // Bcrypt hashed password
  createdAt: Date,        // Account creation timestamp
  hasCompletedOnboarding: Boolean,  // Onboarding status
  settings: {
    notificationsEnabled: Boolean,
    dailyReminderTime: String,
    timezone: String
  },
  subscription: {
    tier: String,         // 'free', 'premium', 'premium_plus'
    status: String,       // 'active', 'cancelled', 'expired'
    expiresAt: Date,
    revenueCatId: String  // RevenueCat customer ID
  }
}
```

### Dreams Collection

```javascript
{
  _id: ObjectId,
  userId: ObjectId,       // Reference to Users collection
  title: String,          // Dream title (e.g., "Start my own business")
  description: String,    // Detailed description
  category: String,       // 'career', 'health', 'relationships', 'personal', 'financial'
  categoryEmoji: String,  // Visual identifier (e.g., "💼")
  status: String,         // 'active', 'completed', 'paused', 'archived'
  createdAt: Date,
  updatedAt: Date,
  targetDate: Date,       // Optional target completion date
  milestones: [{
    title: String,
    completed: Boolean,
    completedAt: Date
  }]
}
```

### Goals Collection

```javascript
{
  _id: ObjectId,
  dreamId: ObjectId,      // Reference to Dreams collection
  userId: ObjectId,       // Reference to Users collection
  title: String,          // Goal title (e.g., "Create business plan")
  description: String,
  status: String,         // 'pending', 'in_progress', 'completed'
  priority: Number,       // 1-5, determines ordering
  createdAt: Date,
  updatedAt: Date,
  completedAt: Date
}
```

### Actions Collection

```javascript
{
  _id: ObjectId,
  goalId: ObjectId,       // Reference to Goals collection
  dreamId: ObjectId,      // Reference to Dreams collection
  userId: ObjectId,       // Reference to Users collection
  title: String,          // Action title (e.g., "Research competitors")
  description: String,    // Detailed instructions
  estimatedMinutes: Number, // Time estimate (15-60)
  status: String,         // 'pending', 'completed', 'skipped'
  scheduledDate: Date,    // Date action is scheduled for
  completedAt: Date,
  createdAt: Date,
  category: String,       // Inherited from dream
  categoryEmoji: String,
  isDaily: Boolean        // If true, shown as daily action
}
```

### Prompts Collection

```javascript
{
  _id: ObjectId,
  text: String,           // Motivational prompt text
  category: String,       // 'motivation', 'reflection', 'gratitude'
  tags: [String],         // For targeted prompts
  active: Boolean,
  createdAt: Date
}
```

---

## API Architecture

### Base URL
- **Development**: `http://localhost:3000/api`
- **Production**: `https://api.goalsapp.com/api` (planned)

### Authentication Endpoints

**POST** `/auth/register`
- Creates new user account
- Request: `{ name, email, password }`
- Response: `{ success, data: { user, token } }`

**POST** `/auth/login`
- Authenticates user and returns JWT
- Request: `{ email, password }`
- Response: `{ success, data: { user, token } }`

**GET** `/auth/me`
- Returns current authenticated user
- Headers: `Authorization: Bearer <token>`
- Response: `{ success, data: { user } }`

### Dreams Endpoints

**GET** `/dreams`
- Returns all dreams for authenticated user
- Headers: `Authorization: Bearer <token>`
- Response: `{ success, data: [dreams] }`

**POST** `/dreams`
- Creates new dream
- Request: `{ title, description, category, targetDate }`
- Response: `{ success, data: { dream } }`

**PUT** `/dreams/:id`
- Updates existing dream
- Request: `{ title, description, status, ... }`
- Response: `{ success, data: { dream } }`

**DELETE** `/dreams/:id`
- Soft deletes dream (sets status to 'archived')
- Response: `{ success, message }`

### Goals Endpoints

**GET** `/goals`
- Returns all goals for user's dreams
- Query params: `?dreamId=<id>` (optional)
- Response: `{ success, data: [goals] }`

**POST** `/goals`
- Creates new goal linked to a dream
- Request: `{ dreamId, title, description, priority }`
- Response: `{ success, data: { goal } }`

### Actions Endpoints

**GET** `/actions/daily`
- Returns today's recommended action
- Algorithm selects based on priority, dream balance, completion history
- Response: `{ success, data: { action } }`

**POST** `/actions/:id/complete`
- Marks action as completed
- Updates: `completedAt` timestamp, `status` to 'completed'
- Response: `{ success, data: { action } }`

**GET** `/actions/stats`
- Returns user statistics
- Response: `{ success, data: { completed, streak, thisWeek, thisMonth } }`

### Prompts Endpoints

**GET** `/prompts/daily`
- Returns daily motivational prompt
- Response: `{ success, data: { prompt } }`

---

## Authentication & Security

### JWT Authentication

**Token Structure:**
```javascript
{
  userId: ObjectId,
  email: String,
  iat: Number,    // Issued at
  exp: Number     // Expiration (7 days)
}
```

**Implementation:**
- Tokens issued on login/registration
- Stored in Flutter app via `SharedPreferences`
- Sent with each API request in `Authorization` header
- Backend middleware validates token on protected routes

### Password Security

- Passwords hashed using **bcrypt** (salt rounds: 10)
- Never stored or transmitted in plain text
- Password complexity requirements enforced on registration

### API Security Measures

1. **CORS**: Configured to allow only mobile app origins
2. **Rate Limiting**: Prevent brute force attacks (planned)
3. **Input Validation**: Sanitize all user inputs
4. **HTTPS Only**: All production traffic over TLS
5. **Environment Variables**: Sensitive keys stored in `.env`, never committed

### Data Privacy

- User data isolated by `userId` in all queries
- No cross-user data access possible
- MongoDB Atlas encryption at rest
- HTTPS encryption in transit
- GDPR-compliant data deletion on account removal

---

## RevenueCat Implementation

### Overview

RevenueCat handles all in-app purchase and subscription logic, providing:
- Cross-platform receipt validation (iOS, Android)
- Subscription state management
- Analytics and revenue tracking
- Webhook notifications for subscription events

### Architecture Integration

```
Flutter App ──────► RevenueCat SDK ──────► App Store / Google Play
     │                      │
     │                      │ (Webhook)
     │                      ▼
     └────────────────► Backend API
                            │
                            ▼
                       MongoDB (Update user.subscription)
```

### Implementation Details

#### Flutter Integration

**Dependencies:**
```yaml
purchases_flutter: ^6.0.0
```

**Initialization:**
```dart
import 'package:purchases_flutter/purchases_flutter.dart';

await Purchases.configure(
  PurchasesConfiguration('your_revenuecat_api_key')
);
```

**Checking Subscription Status:**
```dart
CustomerInfo customerInfo = await Purchases.getCustomerInfo();
bool isPremium = customerInfo.entitlements.active.containsKey('pro');
```

**Making a Purchase:**
```dart
try {
  CustomerInfo customerInfo = await Purchases.purchasePackage(package);
  if (customerInfo.entitlements.active.containsKey('pro')) {
    // Unlock premium features
    updateLocalSubscriptionState('pro');
    syncWithBackend();
  }
} catch (e) {
  // Handle error
}
```

**Restoring Purchases:**
```dart
try {
  CustomerInfo customerInfo = await Purchases.restorePurchases();
  // Update UI based on restored entitlements
} catch (e) {
  // Handle error
}
```

#### Backend Integration

**Webhook Endpoint:**
```javascript
POST /webhooks/revenuecat
```

**Webhook Handler:**
```javascript
app.post('/webhooks/revenuecat', async (req, res) => {
  const event = req.body;

  // Verify webhook signature (security)
  if (!verifyRevenueCatWebhook(req)) {
    return res.status(401).send('Unauthorized');
  }

  // Handle subscription events
  switch (event.type) {
    case 'INITIAL_PURCHASE':
      await handleNewSubscription(event);
      break;
    case 'RENEWAL':
      await handleRenewal(event);
      break;
    case 'CANCELLATION':
      await handleCancellation(event);
      break;
    case 'EXPIRATION':
      await handleExpiration(event);
      break;
  }

  res.status(200).send('OK');
});

async function handleNewSubscription(event) {
  const { app_user_id, product_id, purchased_at_ms } = event;

  await User.updateOne(
    { _id: app_user_id },
    {
      'subscription.tier': productIdToTier(product_id),
      'subscription.status': 'active',
      'subscription.expiresAt': new Date(purchased_at_ms + 30*24*60*60*1000),
      'subscription.revenueCatId': event.id
    }
  );
}
```

### Product Configuration

**RevenueCat Dashboard Setup:**

**Entitlements:**
- `pro` - Dream Achiever features

**Products:**
- **iOS (App Store Connect):**
  - `goals_pro_monthly` - $9.99/month
  - `goals_pro_yearly` - $79.99/year

- **Android (Google Play Console):**
  - `goals_pro_monthly` - $9.99/month
  - `goals_pro_yearly` - $79.99/year

### Feature Gating

**Implementation Pattern:**

```dart
class SubscriptionService {
  static bool isPro = false;
  static const int FREE_DREAM_LIMIT = 3;

  static Future<void> checkSubscription() async {
    CustomerInfo customerInfo = await Purchases.getCustomerInfo();
    isPro = customerInfo.entitlements.active.containsKey('pro');
  }

  static bool canCreateDream(int currentDreamCount) {
    if (isPro) return true;
    return currentDreamCount < FREE_DREAM_LIMIT;
  }

  static bool hasPro() {
    return isPro;
  }
}
```

**Usage in UI:**

```dart
if (!SubscriptionService.canCreateDream(dreamCount)) {
  showDialog(
    context: context,
    builder: (context) => PremiumUpgradeDialog(),
  );
  return;
}
```

### Analytics & Metrics

RevenueCat Dashboard provides:
- Monthly Recurring Revenue (MRR)
- Active subscriptions by tier
- Churn rate
- Trial conversion rate
- Revenue by product
- Subscriber lifetime value

---

## Deployment & Infrastructure

### Current Setup (MVP)

**Backend:**
- Hosted on local development server
- MongoDB Atlas cloud database

**Frontend:**
- Flutter development builds
- Testing on iOS Simulator / Android Emulator

### Production Deployment Plan

**Backend:**
- **Platform**: Heroku / Railway / DigitalOcean
- **Environment**: Node.js 18+ LTS
- **Process Manager**: PM2 for process management
- **Monitoring**: Sentry for error tracking

**Database:**
- **MongoDB Atlas** (M10 cluster for production)
- Automated daily backups
- Read replicas for scaling

**Mobile App:**
- **iOS**: Distributed via TestFlight → App Store
- **Android**: Distributed via Internal Testing → Google Play Store

**CI/CD Pipeline:**
- GitHub Actions for automated testing
- Automated builds on code push
- Environment-based configurations (dev/staging/prod)

### Environment Variables

**Backend `.env`:**
```
NODE_ENV=production
PORT=3000
MONGODB_URI=mongodb+srv://...
JWT_SECRET=<secure-random-string>
REVENUECAT_WEBHOOK_SECRET=<revenuecat-secret>
REVENUECAT_API_KEY=<revenuecat-api-key>
```

**Flutter Environment:**
```dart
const String API_BASE_URL = String.fromEnvironment(
  'API_URL',
  defaultValue: 'http://localhost:3000/api'
);
```

---

## Future Enhancements

### Technical Roadmap

**Phase 1: Core Stability**
- Comprehensive error handling and logging
- Offline mode with local caching
- Push notifications via Firebase
- Performance optimization

**Phase 2: AI Integration**
- OpenAI GPT-4 for intelligent action generation
- Personalized recommendations based on user patterns
- Natural language dream input

**Phase 3: Social Features**
- User profiles and achievements
- Community challenges
- Dream sharing and collaboration
- In-app messaging

**Phase 4: Advanced Analytics**
- Detailed progress insights
- Predictive goal completion
- Habit pattern analysis
- Custom reporting

### Scalability Considerations

**Database Optimization:**
- Implement indexing on frequently queried fields
- Sharding for horizontal scaling
- Caching layer (Redis) for frequently accessed data

**API Performance:**
- CDN for static assets
- API response caching
- Horizontal scaling with load balancing
- Database connection pooling

**Mobile App:**
- Lazy loading of data
- Image optimization and caching
- Background sync for offline changes
- Progressive feature rollout via feature flags

---

## Development & Testing

### Local Development Setup

**Prerequisites:**
- Node.js 18+
- Flutter 3.0+
- MongoDB Atlas account

**Backend:**
```bash
cd backend
npm install
cp .env.example .env
# Configure .env with your MongoDB URI and secrets
npm run dev
```

**Flutter App:**
```bash
cd flutter_app
flutter pub get
flutter run
```

### Testing Strategy

**Backend:**
- Unit tests for business logic (Jest)
- Integration tests for API endpoints
- Database migration tests

**Frontend:**
- Widget tests for UI components
- Integration tests for user flows
- Mock API responses for isolated testing

### Code Quality

- **Linting**: ESLint (backend), Dart analyzer (frontend)
- **Formatting**: Prettier (backend), dartfmt (frontend)
- **Type Safety**: TypeScript migration planned for backend
- **Code Reviews**: Required for all pull requests

---

## Appendix

### API Response Format

All API responses follow this structure:

**Success:**
```json
{
  "success": true,
  "data": { ... },
  "message": "Optional success message"
}
```

**Error:**
```json
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE",
  "statusCode": 400
}
```

### Error Codes

- `AUTH_INVALID_CREDENTIALS` - Login failed
- `AUTH_TOKEN_EXPIRED` - JWT expired
- `RESOURCE_NOT_FOUND` - Requested resource doesn't exist
- `VALIDATION_ERROR` - Invalid input data
- `PERMISSION_DENIED` - User lacks permission
- `SUBSCRIPTION_REQUIRED` - Feature requires premium


---
