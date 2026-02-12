# Design Document: DhanGyan - Gamified Financial Literacy Platform

## 1. System Overview

DhanGyan is a gamified financial literacy platform that transforms passive learning into an interactive, personalized journey. The platform leverages AI, gamification mechanics, and community features to make financial education engaging and habit-forming for 10 million+ users across India.

### 1.1 Design Principles

- **Engagement First**: Every feature should drive user engagement and retention
- **Personalization**: Tailor content and guidance to individual user contexts
- **Accessibility**: Design for low-literacy, low-bandwidth, and diverse linguistic backgrounds
- **Habit Formation**: Build features that encourage daily interaction and long-term behavior change
- **Community-Driven**: Leverage peer learning and social motivation
- **Mobile-First**: Optimize for smartphone users with limited resources

### 1.2 Key Components

1. **Gyan Assistant**: AI-powered conversational assistant for instant financial guidance
2. **Gamification Engine**: Points, badges, streaks, challenges, and rewards system
3. **Learning Content System**: Modular, interactive financial literacy content
4. **Community Guilds**: Peer learning through leaderboards and team challenges
5. **Habit Building System**: Daily challenges and behavior tracking
6. **Analytics Engine**: User progress tracking and impact measurement

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Web App    │  │  Mobile App  │  │   PWA        │      │
│  │  (React)     │  │(React Native)│  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     API Gateway Layer                        │
│              (Load Balancer, Rate Limiting)                  │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Application Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   User       │  │  Learning    │  │ Gamification │      │
│  │   Service    │  │  Service     │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Community   │  │   Habit      │  │  Analytics   │      │
│  │   Service    │  │   Service    │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      AI/ML Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │Gyan Assistant│  │ Personalization│ │  Content    │      │
│  │   (LLM)      │  │    Engine    │  │ Recommender │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  PostgreSQL  │  │    Redis     │  │   MongoDB    │      │
│  │  (User Data) │  │   (Cache)    │  │  (Content)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Technology Stack

**Frontend**
- Web: React 18+ with Next.js 14
- Mobile: React Native with Expo
- State Management: Redux Toolkit / Zustand
- UI Components: Tailwind CSS, shadcn/ui
- Animations: Framer Motion

**Backend**
- API Framework: Node.js with NestJS / Express
- Authentication: JWT, OAuth 2.0
- Real-time: Socket.io for live features
- Task Queue: Bull (Redis-based)

**AI/ML**
- LLM: OpenAI GPT-4 / Google Gemini Pro
- Vector Database: Pinecone / ChromaDB
- NLP: Langchain for orchestration
- Translation: Google Cloud Translation API / Bhashini API

**Database**
- Primary: PostgreSQL (user data, transactions)
- Cache: Redis (sessions, leaderboards)
- Content: MongoDB (learning materials)
- Search: Elasticsearch (content discovery)

**Infrastructure**
- Cloud: AWS / Google Cloud Platform
- CDN: CloudFlare
- Storage: S3 / Cloud Storage
- Monitoring: DataDog / New Relic
- Analytics: Mixpanel / Amplitude

**DevOps**
- Containerization: Docker
- Orchestration: Kubernetes
- CI/CD: GitHub Actions / GitLab CI
- Infrastructure as Code: Terraform

---

## 3. Component Design

### 3.1 Gyan Assistant (AI-Powered Q&A)

**Purpose**: Provide instant, personalized financial guidance through conversational AI.

**Architecture**:
```
User Query → NLP Processing → Context Retrieval → LLM Generation → Response
                                    ↓
                            User Profile + History
                                    ↓
                            Knowledge Base (RAG)
```

**Key Features**:
- Natural language understanding in multiple Indian languages
- Context-aware conversations with memory
- Personalized responses based on user demographics
- Retrieval-Augmented Generation (RAG) for accurate information
- Voice input/output support
- Fallback to human support for complex queries

**Data Model**:
```typescript
interface GyanConversation {
  id: string;
  userId: string;
  messages: Message[];
  context: ConversationContext;
  language: string;
  createdAt: Date;
  updatedAt: Date;
}

interface Message {
  role: 'user' | 'assistant';
  content: string;
  timestamp: Date;
  metadata?: {
    confidence: number;
    sources?: string[];
  };
}

interface ConversationContext {
  userProfile: UserProfile;
  recentTopics: string[];
  learningProgress: LearningProgress;
}
```

**API Endpoints**:
- `POST /api/gyan/chat` - Send message to Gyan Assistant
- `GET /api/gyan/history/:userId` - Retrieve conversation history
- `POST /api/gyan/voice` - Voice input processing
- `DELETE /api/gyan/conversation/:id` - Clear conversation

**Implementation Details**:
- Use Langchain for LLM orchestration
- Implement RAG with vector embeddings of financial content
- Cache common queries in Redis for faster responses
- Rate limiting: 50 requests per user per hour
- Response time target: < 3 seconds

---

### 3.2 Gamification Engine

**Purpose**: Drive engagement through game mechanics (points, badges, streaks, challenges).

**Architecture**:
```
User Action → Event Trigger → Rules Engine → Reward Calculation → Update State
                                    ↓
                            Achievement System
                                    ↓
                            Leaderboard Update
```

**Key Features**:
- Points system for all learning activities
- Badge/achievement unlocking
- Daily streak tracking
- Level progression (1-100)
- Challenge system (daily, weekly, special events)
- Reward redemption (virtual and real rewards)

**Data Model**:
```typescript
interface UserGameState {
  userId: string;
  points: number;
  level: number;
  currentStreak: number;
  longestStreak: number;
  badges: Badge[];
  achievements: Achievement[];
  lastActivityDate: Date;
}

interface Badge {
  id: string;
  name: string;
  description: string;
  iconUrl: string;
  unlockedAt: Date;
  rarity: 'common' | 'rare' | 'epic' | 'legendary';
}

interface Challenge {
  id: string;
  type: 'daily' | 'weekly' | 'special';
  title: string;
  description: string;
  requirements: ChallengeRequirement[];
  reward: Reward;
  expiresAt: Date;
  status: 'active' | 'completed' | 'expired';
}

interface Reward {
  points: number;
  badges?: string[];
  unlocks?: string[];
  realWorldReward?: {
    type: string;
    value: number;
  };
}
```

**Point System**:
- Complete quiz: 10-50 points (based on difficulty)
- Daily login: 5 points
- Complete module: 100 points
- Maintain 7-day streak: 50 bonus points
- Help community member: 20 points
- Share achievement: 10 points

**Level Progression**:
- Level 1-10: 100 points per level
- Level 11-30: 200 points per level
- Level 31-50: 500 points per level
- Level 51-100: 1000 points per level

**API Endpoints**:
- `GET /api/gamification/state/:userId` - Get user game state
- `POST /api/gamification/action` - Record user action for points
- `GET /api/gamification/challenges/:userId` - Get active challenges
- `POST /api/gamification/challenge/:id/complete` - Mark challenge complete
- `GET /api/gamification/leaderboard` - Get leaderboard data
- `POST /api/gamification/reward/redeem` - Redeem rewards

---

### 3.3 Learning Content System

**Purpose**: Deliver modular, interactive financial literacy content.

**Content Structure**:
```
Learning Path
  └── Modules (e.g., "Budgeting Basics")
       └── Lessons (e.g., "Creating Your First Budget")
            └── Content Blocks (video, text, interactive, quiz)
```

**Data Model**:
```typescript
interface LearningPath {
  id: string;
  title: string;
  description: string;
  targetAudience: UserType[];
  modules: Module[];
  estimatedDuration: number; // minutes
  difficulty: 'beginner' | 'intermediate' | 'advanced';
}

interface Module {
  id: string;
  title: string;
  description: string;
  lessons: Lesson[];
  prerequisites?: string[];
  order: number;
}

interface Lesson {
  id: string;
  title: string;
  contentBlocks: ContentBlock[];
  quiz?: Quiz;
  estimatedDuration: number;
  order: number;
}

interface ContentBlock {
  id: string;
  type: 'text' | 'video' | 'image' | 'interactive' | 'case-study';
  content: any;
  order: number;
}

interface Quiz {
  id: string;
  questions: Question[];
  passingScore: number;
  timeLimit?: number;
}

interface Question {
  id: string;
  text: string;
  type: 'multiple-choice' | 'true-false' | 'scenario';
  options: string[];
  correctAnswer: number | number[];
  explanation: string;
  points: number;
}
```

**Content Delivery**:
- Progressive loading for low bandwidth
- Offline caching of completed modules
- Adaptive content based on user performance
- Multi-language support with translation API

**API Endpoints**:
- `GET /api/learning/paths` - List all learning paths
- `GET /api/learning/path/:id` - Get specific learning path
- `GET /api/learning/module/:id` - Get module content
- `GET /api/learning/lesson/:id` - Get lesson content
- `POST /api/learning/progress` - Update user progress
- `POST /api/learning/quiz/submit` - Submit quiz answers
- `GET /api/learning/recommendations/:userId` - Get personalized recommendations

---

### 3.4 Community Guilds

**Purpose**: Enable peer learning through guilds, leaderboards, and team challenges.

**Data Model**:
```typescript
interface Guild {
  id: string;
  name: string;
  description: string;
  avatarUrl: string;
  createdBy: string;
  members: GuildMember[];
  totalPoints: number;
  rank: number;
  createdAt: Date;
  settings: GuildSettings;
}

interface GuildMember {
  userId: string;
  role: 'leader' | 'member';
  joinedAt: Date;
  contributionPoints: number;
}

interface GuildSettings {
  isPublic: boolean;
  maxMembers: number;
  requireApproval: boolean;
}

interface Leaderboard {
  type: 'individual' | 'guild' | 'global';
  period: 'daily' | 'weekly' | 'monthly' | 'all-time';
  entries: LeaderboardEntry[];
  lastUpdated: Date;
}

interface LeaderboardEntry {
  rank: number;
  userId?: string;
  guildId?: string;
  name: string;
  points: number;
  level: number;
  avatarUrl?: string;
}
```

**Features**:
- Create/join guilds (max 50 members per guild)
- Guild leaderboards with real-time updates
- Team challenges and competitions
- Guild chat and discussion boards
- Achievement sharing within guilds
- Guild vs Guild competitions

**API Endpoints**:
- `POST /api/guilds` - Create new guild
- `GET /api/guilds` - List all guilds
- `POST /api/guilds/:id/join` - Join guild
- `DELETE /api/guilds/:id/leave` - Leave guild
- `GET /api/guilds/:id/members` - Get guild members
- `GET /api/leaderboard/:type/:period` - Get leaderboard
- `POST /api/guilds/:id/challenge` - Create guild challenge

**Leaderboard Caching**:
- Use Redis sorted sets for real-time leaderboards
- Update every 5 minutes for global leaderboards
- Real-time updates for guild leaderboards
- Cache top 100 entries for quick access

---

### 3.5 Habit Building System

**Purpose**: Create lasting financial behavior change through daily challenges and tracking.

**Data Model**:
```typescript
interface DailyChallenge {
  id: string;
  userId: string;
  date: Date;
  challenge: Challenge;
  status: 'pending' | 'completed' | 'skipped';
  completedAt?: Date;
}

interface HabitTracker {
  userId: string;
  habits: Habit[];
  currentStreak: number;
  longestStreak: number;
  totalDaysActive: number;
}

interface Habit {
  id: string;
  name: string;
  description: string;
  frequency: 'daily' | 'weekly';
  category: string;
  completionHistory: CompletionRecord[];
  createdAt: Date;
}

interface CompletionRecord {
  date: Date;
  completed: boolean;
  notes?: string;
}

interface BehaviorMetric {
  userId: string;
  metricType: string; // e.g., 'savings_rate', 'debt_reduction'
  value: number;
  date: Date;
  trend: 'improving' | 'stable' | 'declining';
}
```

**Challenge Generation**:
- AI-powered personalized daily challenges
- Difficulty adapts to user level
- Variety across categories (savings, budgeting, learning, etc.)
- Time-bound challenges (24 hours for daily)

**Habit Categories**:
- Savings habits (e.g., "Save ₹50 today")
- Budgeting habits (e.g., "Track all expenses today")
- Learning habits (e.g., "Complete one lesson")
- Awareness habits (e.g., "Review your spending")

**API Endpoints**:
- `GET /api/habits/:userId` - Get user habits
- `POST /api/habits` - Create new habit
- `POST /api/habits/:id/complete` - Mark habit complete
- `GET /api/habits/challenges/daily` - Get today's challenges
- `GET /api/habits/streak/:userId` - Get streak information
- `GET /api/habits/metrics/:userId` - Get behavior metrics

**Notification System**:
- Daily reminder at user-preferred time
- Streak milestone notifications
- Challenge expiry warnings
- Encouragement messages for consistency

---

## 4. User Experience Design

### 4.1 User Onboarding Flow

**Step 1: Welcome & Value Proposition**
- Show key benefits: "Learn by doing, not by reading"
- Highlight gamification elements
- Display social proof (user count, success stories)

**Step 2: Profile Creation**
- Collect: Name, mobile number, age, occupation
- Select user type: Farmer / Woman / Student / Young Adult / Other
- Language preference selection
- Optional: Financial goals

**Step 3: Personalization**
- Quick assessment quiz (5 questions)
- Determine current financial literacy level
- Identify priority learning areas

**Step 4: Tutorial**
- Interactive walkthrough of key features
- First challenge completion
- Gyan Assistant introduction
- Guild recommendation

**Step 5: First Win**
- Award welcome badge
- Grant starting points
- Show personalized learning path
- Encourage first daily challenge

### 4.2 Core User Flows

**Learning Flow**:
1. User selects learning path or gets recommendation
2. Starts module with video/text content
3. Interacts with content blocks
4. Completes quiz at end of lesson
5. Receives immediate feedback and points
6. Unlocks next lesson
7. Tracks progress on dashboard

**Challenge Flow**:
1. User receives daily challenge notification
2. Opens app and views challenge details
3. Completes challenge activity
4. Submits proof/answer
5. System verifies completion
6. Awards points and updates streak
7. Shows next challenge preview

**Gyan Assistant Flow**:
1. User asks financial question (text/voice)
2. System processes and retrieves context
3. AI generates personalized response
4. User receives answer with sources
5. Option to ask follow-up questions
6. Option to save conversation
7. Recommendation for related learning content

**Guild Flow**:
1. User browses available guilds
2. Joins guild or creates new one
3. Views guild leaderboard
4. Participates in guild challenges
5. Shares achievements with guild
6. Competes in guild vs guild events

### 4.3 Mobile App Screens

**Home Screen**:
- Daily challenge card (prominent)
- Current streak display
- Quick access to Gyan Assistant
- Learning progress summary
- Guild leaderboard widget
- Notification center

**Learning Screen**:
- Learning paths grid
- Continue learning section
- Recommended modules
- Progress tracker
- Search functionality

**Profile Screen**:
- User avatar and level
- Points and badges display
- Streak calendar
- Achievement showcase
- Settings access

**Guild Screen**:
- Guild information
- Member list
- Leaderboard
- Guild chat
- Team challenges

**Gyan Assistant Screen**:
- Chat interface
- Voice input button
- Conversation history
- Quick action buttons (common questions)
- Related content suggestions

### 4.4 Accessibility Features

**Language Support**:
- Hindi, English, Tamil, Telugu, Bengali, Marathi, Gujarati, Kannada, Malayalam, Punjabi
- Dynamic language switching
- Regional content customization

**Low Literacy Support**:
- Voice navigation throughout app
- Visual icons and minimal text
- Audio versions of all content
- Video-first learning materials

**Low Bandwidth Optimization**:
- Progressive image loading
- Compressed video streaming
- Offline mode for downloaded content
- Text-only mode option

**Visual Accessibility**:
- High contrast mode
- Adjustable font sizes
- Screen reader compatibility
- Color-blind friendly palette

---

## 5. Data Models

### 5.1 User Model

```typescript
interface User {
  id: string;
  phoneNumber: string;
  email?: string;
  profile: UserProfile;
  preferences: UserPreferences;
  gameState: UserGameState;
  learningProgress: LearningProgress;
  createdAt: Date;
  lastActiveAt: Date;
}

interface UserProfile {
  name: string;
  age?: number;
  gender?: string;
  occupation: string;
  userType: 'farmer' | 'woman' | 'student' | 'young-adult' | 'other';
  location?: {
    state: string;
    district?: string;
  };
  financialGoals?: string[];
  literacyLevel?: 'beginner' | 'intermediate' | 'advanced';
}

interface UserPreferences {
  language: string;
  notificationsEnabled: boolean;
  notificationTime?: string;
  privacySettings: {
    showInLeaderboard: boolean;
    allowGuildInvites: boolean;
  };
}

interface LearningProgress {
  completedModules: string[];
  completedLessons: string[];
  currentPath?: string;
  quizScores: QuizScore[];
  totalTimeSpent: number; // minutes
  lastLessonDate?: Date;
}

interface QuizScore {
  quizId: string;
  score: number;
  maxScore: number;
  attempts: number;
  completedAt: Date;
}
```

### 5.2 Analytics Events

```typescript
interface AnalyticsEvent {
  eventId: string;
  userId: string;
  eventType: string;
  eventData: any;
  timestamp: Date;
  sessionId: string;
  platform: 'web' | 'android' | 'ios';
}

// Key Events to Track:
// - user_signup
// - user_login
// - lesson_started
// - lesson_completed
// - quiz_attempted
// - quiz_passed
// - challenge_completed
// - streak_milestone
// - badge_unlocked
// - gyan_query
// - guild_joined
// - content_shared
// - reward_redeemed
```

---

## 6. API Design

### 6.1 Authentication

```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout
POST /api/auth/refresh-token
POST /api/auth/verify-otp
```

### 6.2 User Management

```
GET /api/users/:id
PUT /api/users/:id
DELETE /api/users/:id
GET /api/users/:id/profile
PUT /api/users/:id/preferences
```

### 6.3 Learning

```
GET /api/learning/paths
GET /api/learning/paths/:id
GET /api/learning/modules/:id
GET /api/learning/lessons/:id
POST /api/learning/progress
POST /api/learning/quiz/submit
GET /api/learning/recommendations/:userId
```

### 6.4 Gamification

```
GET /api/gamification/state/:userId
POST /api/gamification/action
GET /api/gamification/challenges
POST /api/gamification/challenges/:id/complete
GET /api/gamification/leaderboard
POST /api/gamification/rewards/redeem
```

### 6.5 Community

```
GET /api/guilds
POST /api/guilds
GET /api/guilds/:id
POST /api/guilds/:id/join
DELETE /api/guilds/:id/leave
GET /api/guilds/:id/members
GET /api/leaderboard/:type/:period
```

### 6.6 Gyan Assistant

```
POST /api/gyan/chat
GET /api/gyan/history/:userId
POST /api/gyan/voice
DELETE /api/gyan/conversation/:id
```

---

## 7. Security Design

### 7.1 Authentication & Authorization

- JWT-based authentication
- Refresh token rotation
- OTP verification for sensitive actions
- Role-based access control (RBAC)
- Rate limiting per user and IP

### 7.2 Data Protection

- Encryption at rest (AES-256)
- Encryption in transit (TLS 1.3)
- PII data anonymization in analytics
- Regular security audits
- GDPR compliance

### 7.3 API Security

- API key authentication for services
- Request signing for sensitive operations
- Input validation and sanitization
- SQL injection prevention
- XSS protection
- CSRF tokens

---

## 8. Performance Optimization

### 8.1 Caching Strategy

**Redis Caching**:
- User sessions (TTL: 24 hours)
- Leaderboards (TTL: 5 minutes)
- Common Gyan queries (TTL: 1 hour)
- User game state (TTL: 10 minutes)

**CDN Caching**:
- Static assets (images, videos)
- Learning content
- User avatars

### 8.2 Database Optimization

- Indexing on frequently queried fields
- Query optimization and explain analysis
- Connection pooling
- Read replicas for analytics
- Partitioning for large tables

### 8.3 Frontend Optimization

- Code splitting and lazy loading
- Image optimization (WebP, lazy loading)
- Service worker for offline support
- Prefetching for predicted user actions

---

## 9. Monitoring & Analytics

### 9.1 Application Monitoring

- Error tracking (Sentry)
- Performance monitoring (New Relic)
- Uptime monitoring (Pingdom)
- Log aggregation (ELK stack)

### 9.2 User Analytics

- User behavior tracking (Mixpanel)
- Funnel analysis
- Cohort analysis
- A/B testing framework
- Heatmaps and session recordings

### 9.3 Business Metrics

- DAU/MAU ratio
- Retention rates (D1, D7, D30)
- Engagement metrics (session duration, actions per session)
- Learning completion rates
- Behavior change indicators

---

## 10. Deployment Architecture

### 10.1 Infrastructure

```
┌─────────────────────────────────────────────────────────────┐
│                     CloudFlare CDN                           │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Load Balancer (AWS ALB)                    │
└─────────────────────────────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                ▼                       ▼
┌──────────────────────┐    ┌──────────────────────┐
│   Web Server Cluster │    │   API Server Cluster │
│   (Auto-scaling)     │    │   (Auto-scaling)     │
└──────────────────────┘    └──────────────────────┘
                                        │
                ┌───────────────────────┼───────────────────────┐
                ▼                       ▼                       ▼
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│   PostgreSQL RDS     │    │   Redis Cluster      │    │   MongoDB Atlas      │
│   (Multi-AZ)         │    │   (ElastiCache)      │    │                      │
└──────────────────────┘    └──────────────────────┘    └──────────────────────┘
```

### 10.2 Environments

- **Development**: Single instance, shared database
- **Staging**: Production-like, separate database
- **Production**: Multi-region, auto-scaling, high availability

### 10.3 CI/CD Pipeline

1. Code commit to GitHub
2. Automated tests run
3. Build Docker images
4. Push to container registry
5. Deploy to staging
6. Run integration tests
7. Manual approval for production
8. Blue-green deployment to production
9. Health checks and rollback if needed

---

## 11. Scalability Considerations

### 11.1 Horizontal Scaling

- Stateless application servers
- Database read replicas
- Microservices architecture for independent scaling
- Message queue for async processing

### 11.2 Database Scaling

- Sharding by user ID for user data
- Separate databases for different domains
- Caching layer to reduce database load
- Archive old data to cold storage

### 11.3 Content Delivery

- Multi-region CDN
- Edge caching for static content
- Video streaming optimization
- Progressive loading for mobile

---

## 12. Testing Strategy

### 12.1 Unit Testing

- Test coverage target: 80%
- Jest for JavaScript/TypeScript
- Mock external dependencies
- Test business logic thoroughly

### 12.2 Integration Testing

- API endpoint testing
- Database integration tests
- Third-party service integration tests
- End-to-end user flows

### 12.3 Performance Testing

- Load testing (10,000+ concurrent users)
- Stress testing
- Spike testing
- Endurance testing

### 12.4 User Acceptance Testing

- Beta testing with target user groups
- Usability testing
- Accessibility testing
- Localization testing

---

## 13. Correctness Properties

### 13.1 Gamification Properties

**Property 1: Point Consistency**
- For any user action that awards points, the user's total points must increase by exactly the awarded amount
- Validation: `newPoints = oldPoints + awardedPoints`

**Property 2: Streak Integrity**
- A user's streak count must increment by 1 if they complete a daily challenge on consecutive days
- A user's streak must reset to 0 if they miss a day
- Validation: Streak calculation based on consecutive activity dates

**Property 3: Level Progression**
- A user's level must increase when their points reach the threshold for the next level
- A user cannot skip levels
- Validation: `level = f(totalPoints)` where f is monotonically increasing

**Property 4: Badge Uniqueness**
- A user cannot earn the same badge twice
- Badge unlock conditions must be met before awarding
- Validation: Check badge requirements before awarding

### 13.2 Learning Properties

**Property 5: Progress Monotonicity**
- A user's learning progress (completed lessons) can only increase, never decrease
- Validation: `newCompletedLessons ⊇ oldCompletedLessons`

**Property 6: Quiz Score Validity**
- Quiz scores must be between 0 and maximum possible score
- Validation: `0 ≤ score ≤ maxScore`

**Property 7: Prerequisite Enforcement**
- A user cannot access a lesson if prerequisites are not completed
- Validation: Check all prerequisites before allowing access

### 13.3 Community Properties

**Property 8: Guild Member Limit**
- A guild cannot exceed its maximum member limit
- Validation: `guildMembers.length ≤ maxMembers`

**Property 9: Leaderboard Consistency**
- Leaderboard rankings must be sorted by points in descending order
- Validation: `leaderboard[i].points ≥ leaderboard[i+1].points`

**Property 10: Guild Points Aggregation**
- A guild's total points must equal the sum of all member contribution points
- Validation: `guildPoints = Σ(memberPoints)`

### 13.4 Data Integrity Properties

**Property 11: User Uniqueness**
- No two users can have the same phone number or email
- Validation: Unique constraints on user identifiers

**Property 12: Referential Integrity**
- All foreign key references must point to existing records
- Validation: Database foreign key constraints

**Property 13: Timestamp Ordering**
- For any entity, `createdAt ≤ updatedAt`
- Validation: Timestamp comparison on updates

---

## 14. Future Enhancements

### Phase 2 (6-12 months)
- AI-powered financial planning tools
- Integration with banking APIs for real-time tracking
- Virtual financial advisor avatars
- Marketplace for financial products

### Phase 3 (12-18 months)
- AR/VR immersive learning experiences
- Corporate partnerships for employee financial wellness
- Government integration for subsidy awareness
- International expansion

---

## Document Control

**Version:** 1.0  
**Last Updated:** 2026-02-13  
**Status:** Draft  
**Owner:** DhanGyan Engineering Team  
**Reviewers:** Product, Design, Security teams
