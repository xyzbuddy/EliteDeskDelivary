# EliteDesk Implementation Guide

**Step-by-step development roadmap for the EliteDesk Telegram Delivery Bot**

---

## Phase 1: Project Setup & Infrastructure

### Step 1.1: Initialize Project Structure
- [ ] Create Python project directory
- [ ] Set up Git repository
- [ ] Create virtual environment (`python -m venv venv`)
- [ ] Create `requirements.txt` with dependencies:
  - aiogram (Telegram bot framework)
  - python-dotenv (environment variables)
  - sqlalchemy (ORM)
  - psycopg2-binary (PostgreSQL driver)
  - uuid (built-in)
  - python-dateutil (date handling)

### Step 1.2: Database Setup
- [ ] Install PostgreSQL locally or use cloud service (Supabase)
- [ ] Create database named `elitedesk_db`
- [ ] Note connection credentials (host, user, password, port)
- [ ] Create `.env` file with:
  ```
  BOT_TOKEN=your_telegram_bot_token
  ADMIN_ID=your_telegram_id
  DATABASE_URL=postgresql://user:password@localhost:5432/elitedesk_db
  ADMIN_LOG_CHANNEL_ID=your_channel_id
  MANDATORY_CHANNEL_ID=your_channel_id
  ```

### Step 1.3: Project Directory Structure
```
elitedesk/
├── bot/
│   ├── __init__.py
│   ├── main.py
│   ├── handlers/
│   │   ├── __init__.py
│   │   ├── start.py
│   │   ├── main_menu.py
│   │   ├── redeem.py
│   │   ├── products.py
│   │   ├── profile.py
│   │   ├── support.py
│   │   └── admin.py
│   ├── keyboards/
│   │   ├── __init__.py
│   │   ├── inline.py
│   │   └── reply.py
│   └── utils/
│       ├── __init__.py
│       ├── decorators.py
│       └── helpers.py
├── database/
│   ├── __init__.py
│   ├── models.py
│   └── db.py
├── config.py
├── requirements.txt
├── .env
└── .gitignore
```

---

## Phase 2: Database Schema & Models

### Step 2.1: Create Database Models
Create `database/models.py`:

**User Model:**
- id (Primary Key)
- telegram_id (Unique)
- username
- join_date (DateTime)
- claim_count (Integer, default=0)

**Product Model:**
- id (Primary Key)
- uuid (String, unique)
- name (String)
- description (Text)
- instructions (Text)
- emoji (String)
- delivery_limit (Integer)
- status (Enum: ACTIVE, LOW_STOCK, OUT_OF_STOCK, HIDDEN, MAINTENANCE, DISABLED, PAUSED)
- created_at (DateTime)

**Stock Model:**
- id (Primary Key)
- uuid (String, unique)
- product_id (Foreign Key → Product)
- content_json (JSON - stores account data)
- usage_count (Integer, default=0)
- max_usage (Integer)
- admin_note (Text, nullable)
- created_at (DateTime)
- archived_at (DateTime, nullable)

**RedeemCode Model:**
- id (Primary Key)
- uuid (String, unique)
- code (String, unique)
- product_id (Foreign Key → Product)
- is_used (Boolean, default=False)
- expires_at (DateTime, nullable)
- bound_user (Integer, nullable - telegram_id)
- created_at (DateTime)

**Claim Model:**
- id (Primary Key)
- uuid (String, unique)
- user_id (Foreign Key → User)
- product_id (Foreign Key → Product)
- stock_id (Foreign Key → Stock)
- redeem_code (String)
- claimed_at (DateTime)

### Step 2.2: Initialize SQLAlchemy
Create `database/db.py`:
- [ ] Configure PostgreSQL connection
- [ ] Set up session factory
- [ ] Create tables using `Base.metadata.create_all()`
- [ ] Implement session context manager

### Step 2.3: Test Database Connection
- [ ] Run test script to verify all tables created
- [ ] Confirm PostgreSQL data is persisting

---

## Phase 3: Core Bot Infrastructure

### Step 3.1: Bot Initialization
Create `bot/main.py`:
- [ ] Initialize Dispatcher and Bot
- [ ] Load environment variables
- [ ] Set up logging
- [ ] Register all handlers
- [ ] Configure polling

### Step 3.2: Configuration Module
Create `config.py`:
- [ ] Load all environment variables
- [ ] Define constants (timeouts, limits, etc.)
- [ ] Store API keys
- [ ] Configure database connection string

### Step 3.3: Helper Utilities
Create `bot/utils/helpers.py`:
- [ ] UUID generation function
- [ ] Timestamp utilities
- [ ] JSON serialization helpers
- [ ] Emoji parsing functions
- [ ] Message formatting helpers

### Step 3.4: Decorators & Middleware
Create `bot/utils/decorators.py`:
- [ ] Admin-only decorator
- [ ] User verification decorator
- [ ] Rate limiting decorator
- [ ] Database session decorator

---

## Phase 4: Authentication & Verification Flow

### Step 4.1: Implement Start Handler
Create `bot/handlers/start.py`:
- [ ] `/start` command handler
- [ ] Check if user exists in database
- [ ] Create user record if new
- [ ] Redirect to channel verification flow

### Step 4.2: Mandatory Channel Join Verification
Create verification flow:
- [ ] Display join button with channel link
- [ ] "I Joined" button callback
- [ ] Bot checks user membership in channel
- [ ] If verified: proceed to main menu
- [ ] If not verified: show error and retry

### Step 4.3: Session Management
- [ ] Track user state (awaiting channel join, in main menu, redeeming, etc.)
- [ ] Implement user context storage (Redis optional for performance)

---

## Phase 5: Main Menu & Core User Flows

### Step 5.1: Main Menu Handler
Create `bot/handlers/main_menu.py`:
- [ ] Display main menu with buttons:
  - 🏠 Home
  - 🎟 Redeem Code
  - 📦 Products
  - 👤 Profile
  - 📞 Support
  - 💰 My Wallet (placeholder for future)
- [ ] Handle menu button callbacks

### Step 5.2: Redeem Code Handler
Create `bot/handlers/redeem.py`:

**Redeem Flow:**
- [ ] Prompt user to enter redeem code
- [ ] Validate code format (6-8 characters)
- [ ] Check database for code validity
- [ ] Verify code not used
- [ ] Verify code not expired
- [ ] Display delivery preview screen
- [ ] User confirms redemption
- [ ] Implement queue lock (PostgreSQL row lock)
- [ ] Fetch next available stock from FIFO queue
- [ ] Increment stock usage_count
- [ ] Create Claim record
- [ ] Mark RedeemCode as is_used=True
- [ ] Generate delivery message with watermark
- [ ] Send delivery message
- [ ] Schedule auto-delete (15 minutes)
- [ ] Log claim to admin channel

### Step 5.3: Products Handler
Create `bot/handlers/products.py`:
- [ ] List all ACTIVE products
- [ ] Display product with emoji, stock count
- [ ] Show product details on click
- [ ] Filter out OUT_OF_STOCK, HIDDEN, PAUSED products

### Step 5.4: Profile Handler
Create `bot/handlers/profile.py`:
- [ ] Display user Telegram ID
- [ ] Show username
- [ ] Show bot join date
- [ ] Show total claims count

### Step 5.5: Support Handler
Create `bot/handlers/support.py`:
- [ ] Display support username
- [ ] Show estimated reply time
- [ ] Show replacement policy
- [ ] Provide support contact button

---

## Phase 6: Admin Panel Implementation

### Step 6.1: Admin Authentication
- [ ] Restrict admin commands to ADMIN_ID
- [ ] Create admin command prefix or admin-only menu

### Step 6.2: Admin Panel Main Menu
Create `bot/handlers/admin.py`:
- [ ] Display admin menu:
  - 📦 Manage Products
  - 📥 Add Stock
  - 🎟 Generate Codes
  - 📢 Broadcast
  - 📊 Logs
  - ⚙️ Settings
  - 🛠 Queue Repair
  - 📁 Archive Manager
  - 📌 Global Announcement
  - ⏸️ Emergency Pause

### Step 6.3: Product Management
- [ ] Create new product (input: name, description, instructions, warranty, delivery_limit, emoji)
- [ ] Edit existing product
- [ ] List all products with status
- [ ] Update product visibility status

### Step 6.4: Stock Management
- [ ] Add single stock entry
- [ ] Bulk upload TXT file parsing
- [ ] Bulk upload CSV file parsing
- [ ] Export stock as file
- [ ] Add admin notes to stock
- [ ] View stock usage status

### Step 6.5: Redeem Code Generation
- [ ] Generate random 6-8 character codes
- [ ] Link codes to products
- [ ] Set optional code expiration
- [ ] Set optional user binding (for specific user)
- [ ] Display generated codes

### Step 6.6: Broadcast System
- [ ] Text broadcast to all users
- [ ] Image broadcast with optional text
- [ ] Add inline buttons to broadcast
- [ ] Confirm before sending

### Step 6.7: Admin Logs
- [ ] View all claims in admin log channel
- [ ] View failed redeem attempts
- [ ] View low stock alerts
- [ ] View admin actions
- [ ] Log format: timestamp, user, product, code, usage

### Step 6.8: Settings Panel
- [ ] Configure support username
- [ ] Configure support reply time estimate
- [ ] Configure redeem rate limits
- [ ] Set auto-delete timeout
- [ ] Configure replacement policy text

### Step 6.9: Queue Repair Tool
- [ ] Reset usage count for stock
- [ ] Move queue position
- [ ] Fix corrupted stock entries
- [ ] Restore archived stock
- [ ] Reassign delivery slots

### Step 6.10: Archive Manager
- [ ] View archived stock
- [ ] Restore archived stock
- [ ] Delete permanently (with confirmation)
- [ ] Search archived records

### Step 6.11: Global Announcement Pin
- [ ] Set global pinned notice
- [ ] Display at menu top
- [ ] Edit announcement anytime
- [ ] Clear announcement

### Step 6.12: Emergency Pause
- [ ] Instantly pause any product
- [ ] Hide product without deleting stock
- [ ] Preserve stock safely
- [ ] Resume product anytime

---

## Phase 7: Advanced Features

### Step 7.1: Rate Limiting
- [ ] Implement max 3 redeem attempts per minute per user
- [ ] Track invalid attempts
- [ ] Display warning message when limit exceeded
- [ ] Reset counter after time window

### Step 7.2: Low Stock Alerts
- [ ] Check stock level after each claim
- [ ] If stock <= 1, send alert to admin log channel
- [ ] Include product name and remaining count

### Step 7.3: Auto-Delete System
- [ ] Schedule message deletion 15 minutes after delivery
- [ ] Use aiogram scheduled tasks
- [ ] Handle deletion failures gracefully

### Step 7.4: Delivery Watermark
- [ ] Add "Delivered To:" section with username and telegram_id
- [ ] Include in all delivery messages
- [ ] Format consistently

### Step 7.5: Product Visibility States
Implement 7 status modes:
- [ ] ACTIVE - visible and redeemable
- [ ] LOW_STOCK - visible with warning
- [ ] OUT_OF_STOCK - visible but disabled
- [ ] HIDDEN - not visible to users
- [ ] MAINTENANCE - not visible, temporarily paused
- [ ] DISABLED - permanently unavailable
- [ ] PAUSED - temporarily unavailable (can resume)

### Step 7.6: Concurrent Delivery Protection
- [ ] Implement PostgreSQL row-level locking
- [ ] Use SQLAlchemy `with_for_update()` on stock queries
- [ ] Ensure FIFO queue integrity during simultaneous claims
- [ ] Handle lock timeout gracefully

### Step 7.7: Manual Claim Revoke System
- [ ] Admin can revoke specific claim
- [ ] Remove claim record from database
- [ ] Decrement stock usage_count
- [ ] Return slot to queue
- [ ] Mark user eligible for replacement

### Step 7.8: Emoji System Implementation
- [ ] Emoji Library Mode: support predefined emoji shortcuts
- [ ] Manual Emoji Mode: accept custom Telegram emoji from admin
- [ ] Load emoji packs from configured sources
- [ ] Parse and display emojis correctly in messages

---

## Phase 8: Logging & Monitoring

### Step 8.1: Admin Log Channel
- [ ] Configure dedicated Telegram channel for logs
- [ ] Log new claims with all details
- [ ] Log low stock alerts
- [ ] Log failed redeem attempts
- [ ] Log admin actions (product creation, stock edits, etc.)
- [ ] Maintain formatted, searchable logs

### Step 8.2: Error Handling
- [ ] Graceful error messages to users
- [ ] Detailed error logging for debugging
- [ ] Admin notifications for critical errors
- [ ] Database connection error recovery

### Step 8.3: Activity Monitoring
- [ ] Track active users
- [ ] Monitor redeem success rate
- [ ] Track stock depletion rate
- [ ] Monitor bot uptime

---

## Phase 9: Security Implementation

### Step 9.1: Data Encryption
- [ ] Encrypt sensitive stock content in database (passwords, emails)
- [ ] Use environment variables for encryption keys
- [ ] Implement encrypt/decrypt functions

### Step 9.2: Queue Integrity
- [ ] PostgreSQL transaction locking for FIFO delivery
- [ ] Prevent race conditions during concurrent claims
- [ ] Validate queue state before each delivery

### Step 9.3: Admin Audit Trail
- [ ] Log all admin actions with timestamp
- [ ] Track who modified what and when
- [ ] Store immutable audit logs

### Step 9.4: UUID Tracking
- [ ] Every entity gets unique UUID
- [ ] UUIDs remain internal and immutable
- [ ] Use UUIDs for secure tracking and debugging
- [ ] Format: `prd_` (products), `stk_` (stock), `clm_` (claims), `code_` (codes)

### Step 9.5: Rate Limiting
- [ ] Implement per-user redeem attempt limits
- [ ] Implement per-user spam limits
- [ ] Track failed attempts
- [ ] Block/warn users exceeding limits

### Step 9.6: Input Validation
- [ ] Validate all user inputs
- [ ] Sanitize database queries (use parameterized queries)
- [ ] Validate file uploads (TXT, CSV)
- [ ] Limit message lengths

---

## Phase 10: Deployment & DevOps

### Step 10.1: Production Database
- [ ] Set up PostgreSQL on production server or Supabase
- [ ] Configure backups and recovery
- [ ] Set up database monitoring
- [ ] Implement connection pooling

### Step 10.2: Bot Hosting
- [ ] Choose hosting provider (VPS, PaaS)
- [ ] Deploy bot with proper logging
- [ ] Set up uptime monitoring
- [ ] Configure auto-restart on failure

### Step 10.3: Environment Configuration
- [ ] Separate .env files for dev and production
- [ ] Secure credential management
- [ ] Implement secrets management

### Step 10.4: Database Backups
- [ ] Automated daily backups
- [ ] Test backup restoration
- [ ] Keep backup retention policy

---

## Phase 11: Optional Web Dashboard

### Step 11.1: Backend Setup
- [ ] Create FastAPI application
- [ ] Connect to same PostgreSQL database
- [ ] Implement authentication (admin login)
- [ ] Set up CORS for frontend

### Step 11.2: Dashboard Features
- [ ] Product management interface
- [ ] Stock management interface
- [ ] Claim analytics and statistics
- [ ] Admin logs viewer
- [ ] User management

### Step 11.3: Frontend
- [ ] Create React/Vue frontend
- [ ] Build product management UI
- [ ] Build analytics dashboard
- [ ] Implement responsive design

### Step 11.4: Deployment
- [ ] Deploy FastAPI backend (Railway, Render)
- [ ] Deploy frontend (Vercel, Netlify)
- [ ] Configure domain and SSL
- [ ] Set up monitoring

---

## Phase 12: Testing & QA

### Step 12.1: Unit Tests
- [ ] Test database models
- [ ] Test helper functions
- [ ] Test UUID generation
- [ ] Test validation functions

### Step 12.2: Integration Tests
- [ ] Test redeem flow end-to-end
- [ ] Test concurrent deliveries
- [ ] Test admin panel operations
- [ ] Test broadcast functionality

### Step 12.3: Manual Testing
- [ ] Test all user flows
- [ ] Test all admin operations
- [ ] Test edge cases (empty stock, expired codes, etc.)
- [ ] Test error scenarios

### Step 12.4: Load Testing
- [ ] Test with multiple concurrent users
- [ ] Monitor database performance
- [ ] Optimize queries if needed

---

## Phase 13: Future Expansion (Placeholder)

### Step 13.1: Payment Integration
- [ ] Integrate payment gateway (Stripe, etc.)
- [ ] Enable paid redeem codes
- [ ] Track revenue

### Step 13.2: Reseller System
- [ ] Support reseller accounts
- [ ] Commission tracking
- [ ] Reseller dashboard

### Step 13.3: Crypto Payments
- [ ] Accept cryptocurrency
- [ ] Implement blockchain integration

### Step 13.4: Subscriptions
- [ ] Support recurring subscriptions
- [ ] Auto-renewal system
- [ ] Subscription management UI

### Step 13.5: API Access
- [ ] Build REST/GraphQL API
- [ ] API authentication and rate limiting
- [ ] API documentation

### Step 13.6: Advanced Analytics
- [ ] User behavior analytics
- [ ] Sales trends
- [ ] Product performance metrics
- [ ] Custom reports

---

## Checklist Summary

### Critical Path (MVP)
- [ ] Phase 1: Project Setup
- [ ] Phase 2: Database Schema
- [ ] Phase 3: Bot Infrastructure
- [ ] Phase 4: Authentication
- [ ] Phase 5: Core User Flows
- [ ] Phase 7.6: Concurrent Delivery Protection
- [ ] Phase 8: Logging
- [ ] Phase 9: Security
- [ ] Phase 10: Deployment

### Post-MVP (Enhancement)
- [ ] Phase 6: Admin Panel (full)
- [ ] Phase 7: Advanced Features
- [ ] Phase 11: Web Dashboard
- [ ] Phase 12: Testing
- [ ] Phase 13: Future Expansion

---

## Key Development Tips

1. **Start Simple**: Build MVP first (core redeem flow), add features incrementally
2. **Test Database**: Verify all queries before moving forward
3. **Queue Lock**: Implement row-level locking early to prevent bugs
4. **Logging**: Add comprehensive logging from the beginning
5. **Environment Variables**: Never hardcode credentials
6. **Error Handling**: Handle failures gracefully at every step
7. **User Messages**: Keep messages clear and helpful
8. **Admin Tools**: Build admin features with safety in mind
9. **Backup Data**: Never lose user data or stock
10. **Monitor Production**: Track bot health continuously

---

## Timeline Estimate

- **Phase 1-4**: 1-2 weeks (setup and auth)
- **Phase 5**: 1-2 weeks (core flows)
- **Phase 6**: 2-3 weeks (admin panel)
- **Phase 7-8**: 1-2 weeks (advanced features & logging)
- **Phase 9-10**: 1 week (security & deployment)
- **Phase 11-13**: 2-4 weeks (dashboard & future features)

**Total for MVP (Phases 1-10): 6-10 weeks**

---

## Next Steps

1. ✅ Review this implementation guide
2. Create GitHub issues for each phase
3. Set up development environment (Python, PostgreSQL)
4. Begin Phase 1: Project Setup
5. Build and test incrementally
6. Deploy to production when Phase 10 is complete
