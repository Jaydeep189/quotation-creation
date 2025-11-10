# OpinionFlow - Technical Specification Document

## 1. Executive Summary

This document provides a comprehensive technical specification for the OpinionFlow platform, covering system architecture, technology stack, custom ad engine design, multilingual implementation, security measures, and implementation strategies.

**Project Scope**: Development of all platform components EXCEPT AI/ML review analysis (handled separately)

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CDN Layer (Cloudflare)                  │
│              Static Assets, Edge Caching, DDoS              │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                  Load Balancer (AWS ALB/NLB)                │
└─────────────────────────────────────────────────────────────┘
                              ↓
        ┌────────────────────┴────────────────────┐
        ↓                                         ↓
┌──────────────────┐                    ┌──────────────────┐
│  Public Website  │                    │  API Gateway     │
│  (Astro.js SSR)  │                    │  (Node.js/Express)│
│  Edge Deployed   │                    │                  │
└──────────────────┘                    └──────────────────┘
                                                  ↓
                ┌─────────────────────────────────┴─────────────────┐
                ↓                                                   ↓
    ┌──────────────────┐                              ┌──────────────────┐
    │  Backend Services│                              │  Ad Engine       │
    │  (Node.js/Python)│                              │  (Custom Service)│
    │  - Auth          │                              └──────────────────┘
    │  - Business Logic│
    │  - APIs          │
    └──────────────────┘
                ↓
    ┌────────────────────────────────────┐
    │         Data Layer                 │
    ├────────────┬──────────┬───────────┤
    │ PostgreSQL │ MongoDB  │  Redis    │
    │ (Primary)  │ (Docs)   │ (Cache)   │
    └────────────┴──────────┴───────────┘
                ↓
    ┌────────────────────────────────────┐
    │      External Services             │
    │  - Payment Gateway                 │
    │  - Email/SMS                       │
    │  - Translation API                 │
    │  - ERP Integration                 │
    └────────────────────────────────────┘
```

### 2.2 Technology Stack

#### Frontend
- **Public Website**: Astro.js 4.x with React components
- **Dashboards**: React.js 18+ with TypeScript
- **Styling**: Tailwind CSS 3.x
- **State Management**: Zustand / React Query
- **UI Components**: shadcn/ui, Radix UI
- **Charts**: Recharts / Chart.js

#### Backend
- **API Layer**: Node.js 20+ with Express.js / Fastify
- **Business Logic**: Python 3.11+ (Django/FastAPI)
- **Authentication**: JWT + OAuth2
- **API Documentation**: OpenAPI 3.0 (Swagger)

#### Database
- **Primary DB**: PostgreSQL 15+
- **Document Store**: MongoDB 6+
- **Cache**: Redis 7+
- **Search**: Elasticsearch 8+ OR Typesense

#### Infrastructure
- **Cloud**: AWS / Google Cloud / Vercel
- **CDN**: Cloudflare
- **Storage**: AWS S3 / CloudFlare R2
- **Monitoring**: Datadog / New Relic
- **Logging**: ELK Stack / Loki
- **CI/CD**: GitHub Actions

---

## 3. Custom Ad Engine Design

### 3.1 Architecture Overview

The custom ad engine is designed to be lightweight, efficient, and scalable for initial launch, with room for growth.

```
┌──────────────────────────────────────────────────────┐
│                  Ad Request                          │
│  {page_type, location, user_context, targeting}      │
└──────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────┐
│              Ad Engine Controller                    │
│  - Request validation                                │
│  - Rate limiting                                     │
│  - User context extraction                           │
└──────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────┐
│              Targeting Engine                        │
│  - Match campaigns by:                               │
│    * Location (country, city)                        │
│    * Demographics (age, gender)                      │
│    * Category/Product                                │
│    * Device type                                     │
│    * Time/Day                                        │
└──────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────┐
│              Budget & Pacing Engine                  │
│  - Check campaign budget                             │
│  - Check daily budget                                │
│  - Verify impression/click quotas                    │
│  - Apply pacing (even distribution)                  │
└──────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────┐
│              Auction & Selection Engine              │
│  - Priority scoring                                  │
│  - eCPM calculation                                  │
│  - Randomized selection (weighted)                   │
│  - Fallback handling                                 │
└──────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────┐
│              Creative Selection                      │
│  - Format matching                                   │
│  - A/B test variant selection                        │
│  - Creative rotation                                 │
└──────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────┐
│              Response Generation                     │
│  - Build ad payload                                  │
│  - Generate tracking URLs                            │
│  - Add impression beacon                             │
│  - Return JSON response                              │
└──────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────┐
│              Tracking & Analytics                    │
│  - Log impression                                    │
│  - Update campaign stats                             │
│  - Real-time analytics pipeline                      │
└──────────────────────────────────────────────────────┘
```

### 3.2 Core Components

#### 3.2.1 Ad Server API

**Endpoints:**

```
GET  /api/v1/ads/serve
POST /api/v1/ads/impression
POST /api/v1/ads/click
GET  /api/v1/ads/analytics
```

**Ad Serve Request:**
```json
{
  "page_type": "product_page",
  "location": "sidebar_top",
  "product_id": "uuid",
  "category_id": "uuid",
  "user_context": {
    "user_id": "uuid",
    "location": {"country": "IN", "city": "Mumbai"},
    "device": "mobile",
    "language": "en"
  },
  "format": "300x250"
}
```

**Ad Serve Response:**
```json
{
  "campaign_id": "uuid",
  "creative_id": "uuid",
  "ad_data": {
    "type": "banner",
    "format": "300x250",
    "headline": "Get 50% Off!",
    "description": "Limited time offer",
    "image_url": "https://cdn.../banner.jpg",
    "cta_text": "Shop Now",
    "cta_url": "https://track.../click?id=xxx"
  },
  "tracking": {
    "impression_url": "https://track.../impression?id=xxx",
    "view_url": "https://track.../view?id=xxx"
  }
}
```

#### 3.2.2 Targeting Engine

**Targeting Criteria:**
```typescript
interface TargetingCriteria {
  // Geographic
  countries?: string[];
  cities?: string[];
  radius?: { lat: number; lng: number; km: number };
  
  // Demographics
  age_range?: { min: number; max: number };
  gender?: 'male' | 'female' | 'other' | 'all';
  
  // Behavioral
  interests?: string[];
  product_categories?: string[];
  
  // Contextual
  page_types?: string[];
  keywords?: string[];
  
  // Technical
  devices?: ('mobile' | 'tablet' | 'desktop')[];
  browsers?: string[];
  os?: string[];
  
  // Temporal
  days_of_week?: number[];
  hours_of_day?: number[];
  
  // Custom
  custom_audience_ids?: string[];
}
```

**Matching Algorithm:**
```python
def match_campaigns(request, campaigns):
    matched = []
    
    for campaign in campaigns:
        score = 0
        
        # Geographic match (high priority)
        if matches_geography(request, campaign.targeting):
            score += 10
        
        # Category match
        if matches_category(request, campaign.targeting):
            score += 8
        
        # Device match
        if matches_device(request, campaign.targeting):
            score += 5
        
        # Demographic match
        if matches_demographics(request, campaign.targeting):
            score += 7
        
        # Time-based match
        if matches_schedule(request, campaign.targeting):
            score += 3
        
        if score >= MINIMUM_MATCH_SCORE:
            matched.append({
                'campaign': campaign,
                'relevance_score': score
            })
    
    return sorted(matched, key=lambda x: x['relevance_score'], reverse=True)
```

#### 3.2.3 Budget & Pacing System

**Budget Check:**
```python
def check_budget(campaign):
    # Total budget check
    if campaign.spent >= campaign.budget:
        return False, "Budget exhausted"
    
    # Daily budget check
    today_spent = get_daily_spend(campaign.id, date.today())
    if campaign.daily_budget and today_spent >= campaign.daily_budget:
        return False, "Daily budget exhausted"
    
    # Impression quota check
    if campaign.impressions_quota:
        if campaign.impressions_served >= campaign.impressions_quota:
            return False, "Impression quota reached"
    
    # Click quota check
    if campaign.clicks_quota:
        if campaign.clicks_served >= campaign.clicks_quota:
            return False, "Click quota reached"
    
    return True, "OK"
```

**Pacing Algorithm:**
```python
def calculate_pacing_score(campaign):
    """
    Ensures even distribution of impressions/budget over campaign duration
    """
    total_days = (campaign.end_date - campaign.start_date).days
    elapsed_days = (date.today() - campaign.start_date).days
    
    expected_progress = elapsed_days / total_days
    actual_progress = campaign.spent / campaign.budget
    
    # If underpacing, increase priority
    if actual_progress < expected_progress - 0.1:
        return 1.5
    # If overpacing, decrease priority
    elif actual_progress > expected_progress + 0.1:
        return 0.5
    
    return 1.0
```

#### 3.2.4 Auction & Selection

**eCPM Calculation:**
```python
def calculate_ecpm(campaign, relevance_score):
    """
    Effective Cost Per Mille (thousand impressions)
    """
    if campaign.pricing_model == 'CPM':
        base_ecpm = campaign.cpm_rate
    elif campaign.pricing_model == 'CPC':
        # Estimate CTR based on historical data
        estimated_ctr = get_estimated_ctr(campaign)
        base_ecpm = campaign.cpc_rate * estimated_ctr * 1000
    else:  # Flat fee
        daily_impressions = campaign.impressions_quota / campaign.duration_days
        base_ecpm = (campaign.flat_fee / campaign.duration_days) / daily_impressions * 1000
    
    # Apply relevance multiplier
    ecpm = base_ecpm * (relevance_score / 10)
    
    # Apply pacing multiplier
    pacing_score = calculate_pacing_score(campaign)
    ecpm *= pacing_score
    
    # Apply priority multiplier
    ecpm *= (1 + campaign.priority * 0.1)
    
    return ecpm
```

**Selection Algorithm:**
```python
def select_ad(eligible_campaigns):
    """
    Weighted random selection based on eCPM
    """
    if not eligible_campaigns:
        return None
    
    # Calculate weights
    weights = []
    for campaign in eligible_campaigns:
        weight = campaign.ecpm ** 2  # Square for more aggressive weighting
        weights.append(weight)
    
    # Normalize weights
    total_weight = sum(weights)
    probabilities = [w / total_weight for w in weights]
    
    # Random selection
    selected = random.choices(eligible_campaigns, weights=probabilities, k=1)[0]
    
    return selected
```

#### 3.2.5 Tracking & Attribution

**Impression Tracking:**
```python
async def track_impression(impression_data):
    # Validate impression
    if not validate_impression(impression_data):
        return
    
    # Deduplicate (prevent double counting)
    cache_key = f"imp:{impression_data.creative_id}:{impression_data.session_id}"
    if await redis.exists(cache_key):
        return
    
    await redis.setex(cache_key, 3600, "1")  # 1 hour TTL
    
    # Log to database
    await db.ad_impressions.insert({
        'campaign_id': impression_data.campaign_id,
        'creative_id': impression_data.creative_id,
        'user_id': impression_data.user_id,
        'session_id': impression_data.session_id,
        'page_url': impression_data.page_url,
        'device_type': impression_data.device_type,
        'location_country': impression_data.location.country,
        'location_city': impression_data.location.city,
        'viewable': impression_data.viewable,
        'view_duration': impression_data.view_duration,
        'impression_time': datetime.now()
    })
    
    # Update campaign stats (cached)
    await redis.hincrby(f"campaign:{impression_data.campaign_id}:stats", "impressions", 1)
    
    # Queue for aggregation
    await queue.publish('analytics.impressions', impression_data)
```

**Click Tracking:**
```python
async def track_click(click_data):
    # Validate click
    if not validate_click(click_data):
        return
    
    # Verify impression exists (click fraud prevention)
    impression = await db.ad_impressions.find_one({
        'creative_id': click_data.creative_id,
        'session_id': click_data.session_id,
        'impression_time': {'$gte': datetime.now() - timedelta(hours=1)}
    })
    
    if not impression:
        logger.warning(f"Click without impression: {click_data}")
        return
    
    # Log click
    await db.ad_clicks.insert({
        'campaign_id': click_data.campaign_id,
        'creative_id': click_data.creative_id,
        'impression_id': impression.id,
        'user_id': click_data.user_id,
        'session_id': click_data.session_id,
        'click_time': datetime.now(),
        'ip_address': click_data.ip_address,
        'device_type': click_data.device_type
    })
    
    # Update campaign stats
    await redis.hincrby(f"campaign:{click_data.campaign_id}:stats", "clicks", 1)
    
    # Calculate cost
    cost = calculate_click_cost(click_data.campaign_id)
    await redis.hincrbyfloat(f"campaign:{click_data.campaign_id}:stats", "spent", cost)
    
    # Queue for aggregation
    await queue.publish('analytics.clicks', click_data)
    
    # Redirect to destination
    return get_campaign_redirect_url(click_data.campaign_id)
```

### 3.3 Ad Engine Performance Optimization

#### 3.3.1 Caching Strategy

```python
# Cache eligible campaigns per location
cache_key = f"ads:eligible:{page_type}:{location}:{targeting_hash}"
eligible_campaigns = await redis.get(cache_key)

if not eligible_campaigns:
    eligible_campaigns = await db.get_eligible_campaigns(criteria)
    await redis.setex(cache_key, 300, json.dumps(eligible_campaigns))  # 5 min TTL
```

#### 3.3.2 Database Indexing

```sql
-- Critical indexes for ad serving
CREATE INDEX idx_campaigns_active ON ad_campaigns(status, start_date, end_date) 
WHERE status = 'active';

CREATE INDEX idx_campaigns_targeting ON ad_campaigns 
USING GIN (targeting jsonb_path_ops);

CREATE INDEX idx_impressions_session ON ad_impressions(session_id, creative_id, impression_time);
```

#### 3.3.3 Load Balancing

- Use Redis for distributed rate limiting
- Implement circuit breakers for external services
- Queue-based async processing for analytics
- Horizontal scaling for ad server instances

---

## 4. Multilingual System Design

### 4.1 Architecture

```
┌─────────────────────────────────────────────────────┐
│           Content Management Layer                  │
│  (Create/Edit content in default language)          │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│           Translation Management                    │
│  - Manual translation interface                     │
│  - AI translation (DeepL/Google)                    │
│  - Professional translation workflow                │
│  - Quality review                                   │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│           Translation Storage                       │
│  Database: translations table                       │
│  Cache: Redis (hot translations)                    │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│           Content Delivery                          │
│  - API returns content in requested language        │
│  - Fallback to default if translation missing       │
│  - Client-side i18n for UI elements                 │
└─────────────────────────────────────────────────────┘
```

### 4.2 Implementation Approach

#### 4.2.1 Technology Stack

**Recommended Solution: i18next + Database Storage**

**Frontend:**
- **i18next**: Industry-standard internationalization framework
- **react-i18next**: React bindings
- **i18next-http-backend**: Load translations from API

**Backend:**
- **i18next**: For server-side rendering
- **Custom Translation Service**: API for managing translations

**Translation Services (AI):**
- **DeepL API**: High-quality neural translations (Primary)
- **Google Cloud Translation API**: Fallback option
- **LibreTranslate**: Self-hosted option (cost-effective)

#### 4.2.2 Implementation Strategy

**Step 1: Define Translatable Content Types**

```typescript
enum TranslatableEntity {
  PRODUCT = 'product',
  CATEGORY = 'category',
  BLOG_POST = 'blog_post',
  OFFER = 'offer',
  VENDOR = 'vendor',
  UI_STRINGS = 'ui_strings',
  EMAIL_TEMPLATE = 'email_template',
  METADATA = 'metadata'
}

enum TranslatableField {
  TITLE = 'title',
  DESCRIPTION = 'description',
  CONTENT = 'content',
  SHORT_DESCRIPTION = 'short_description',
  META_TITLE = 'meta_title',
  META_DESCRIPTION = 'meta_description'
}
```

**Step 2: Translation Service API**

```typescript
// Translation Service Interface
interface TranslationService {
  // Get translation
  getTranslation(
    entityType: string,
    entityId: string,
    field: string,
    language: string
  ): Promise<string | null>;
  
  // Create/Update translation
  saveTranslation(
    entityType: string,
    entityId: string,
    field: string,
    language: string,
    value: string,
    method: 'manual' | 'ai' | 'professional'
  ): Promise<void>;
  
  // Bulk translate entity
  translateEntity(
    entityType: string,
    entityId: string,
    targetLanguages: string[],
    fields?: string[]
  ): Promise<TranslationResult>;
  
  // Check translation status
  getTranslationStatus(
    entityType: string,
    entityId: string
  ): Promise<TranslationStatus>;
}
```

**Step 3: Content Retrieval with Translations**

```typescript
// API endpoint example
app.get('/api/v1/products/:id', async (req, res) => {
  const { id } = req.params;
  const language = req.headers['accept-language'] || 'en';
  
  // Get base product
  const product = await db.products.findById(id);
  
  // Get translations
  const translations = await translationService.getEntityTranslations(
    'product',
    id,
    language
  );
  
  // Merge translations
  const localizedProduct = {
    ...product,
    name: translations.name || product.name,
    description: translations.description || product.description,
    meta_title: translations.meta_title || product.meta_title,
    meta_description: translations.meta_description || product.meta_description
  };
  
  res.json(localizedProduct);
});
```

**Step 4: Frontend Integration**

```typescript
// i18next configuration
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import Backend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    supportedLngs: ['en', 'hi', 'es', 'fr', 'de', 'ja', 'zh'],
    
    backend: {
      loadPath: '/api/v1/translations/{{lng}}/{{ns}}'
    },
    
    detection: {
      order: ['querystring', 'cookie', 'localStorage', 'navigator'],
      caches: ['localStorage', 'cookie']
    },
    
    interpolation: {
      escapeValue: false
    }
  });

// Usage in components
import { useTranslation } from 'react-i18next';

function ProductCard({ product }) {
  const { t, i18n } = useTranslation();
  
  return (
    <div>
      <h2>{product.name}</h2>
      <p>{product.description}</p>
      <button>{t('product.addToCart')}</button>
    </div>
  );
}
```

**Step 5: Admin Interface for Translation Management**

```typescript
// Translation dashboard component
function TranslationManager() {
  const [entities, setEntities] = useState([]);
  const [selectedEntity, setSelectedEntity] = useState(null);
  
  const translateWithAI = async (entityId, targetLanguages) => {
    await api.post('/api/v1/admin/translations/auto-translate', {
      entity_type: 'product',
      entity_id: entityId,
      target_languages: targetLanguages,
      provider: 'deepl'
    });
  };
  
  return (
    <div>
      <EntitySelector onChange={setSelectedEntity} />
      <TranslationForm entity={selectedEntity} />
      <AITranslateButton onClick={translateWithAI} />
    </div>
  );
}
```

### 4.3 Translation Workflow

```
1. Content Creation (Default Language: English)
   ↓
2. Auto-translate to Primary Languages (AI)
   - Hindi, Spanish, French, German
   ↓
3. Quality Review (Optional)
   - Flag poor translations
   - Mark for professional translation
   ↓
4. Professional Translation (For critical content)
   - Marketing pages
   - Legal documents
   ↓
5. Publish Translations
   ↓
6. Monitor & Update
   - Track usage
   - Update based on feedback
```

### 4.4 SEO Considerations for Multilingual

```typescript
// URL Structure
// Option 1: Subdirectory (Recommended)
// en: https://opinionflow.com/products/iphone-15
// hi: https://opinionflow.com/hi/products/iphone-15
// es: https://opinionflow.com/es/products/iphone-15

// Option 2: Subdomain
// en: https://www.opinionflow.com/products/iphone-15
// hi: https://hi.opinionflow.com/products/iphone-15

// Hreflang tags
<link rel="alternate" hreflang="en" href="https://opinionflow.com/products/iphone-15" />
<link rel="alternate" hreflang="hi" href="https://opinionflow.com/hi/products/iphone-15" />
<link rel="alternate" hreflang="x-default" href="https://opinionflow.com/products/iphone-15" />

// Language switcher
function LanguageSwitcher() {
  const { i18n } = useTranslation();
  const languages = [
    { code: 'en', name: 'English' },
    { code: 'hi', name: 'हिन्दी' },
    { code: 'es', name: 'Español' }
  ];
  
  return (
    <select 
      value={i18n.language} 
      onChange={(e) => i18n.changeLanguage(e.target.value)}
    >
      {languages.map(lang => (
        <option key={lang.code} value={lang.code}>
          {lang.name}
        </option>
      ))}
    </select>
  );
}
```

### 4.5 Cost Estimation for Translation

**DeepL Pricing:**
- Free: 500,000 characters/month
- Pro: $5.49 per 500,000 characters

**Estimated Costs:**
- Average product description: 500 characters
- Average blog post: 2,000 characters
- 1,000 products × 500 chars × 5 languages = 2.5M characters = $27.45
- 500 blog posts × 2,000 chars × 5 languages = 5M characters = $54.90

**Total Initial Translation Cost: ~$100**
**Monthly ongoing cost: ~$20-50** (new content)

---

## 5. Dashboard Sitemaps

### 5.1 Super Admin Dashboard

```
/admin/dashboard
├── /admin/overview
│   ├── System Health
│   ├── Revenue Metrics
│   ├── User Statistics
│   └── Recent Activities
├── /admin/users
│   ├── User List
│   ├── Role Management
│   ├── User Details
│   └── Activity Logs
├── /admin/content
│   ├── Products
│   ├── Reviews
│   ├── Blog Posts
│   └── Moderation Queue
├── /admin/campaigns
│   ├── All Campaigns
│   ├── Pending Approvals
│   ├── Active Campaigns
│   └── Campaign Analytics
├── /admin/offers
│   ├── All Offers
│   ├── Pending Approvals
│   ├── Active Offers
│   └── Offer Analytics
├── /admin/vendors
│   ├── Vendor List
│   ├── Pending Approvals
│   ├── Active Vendors
│   └── Vendor Analytics
├── /admin/finance
│   ├── Revenue Dashboard
│   ├── Invoices
│   ├── Payments
│   └── Reports
├── /admin/system
│   ├── Settings
│   ├── Pricing Rules
│   ├── Email Templates
│   ├── Audit Logs
│   └── API Keys
└── /admin/reports
    ├── User Reports
    ├── Revenue Reports
    ├── Content Reports
    └── System Reports
```

### 5.2 Content Admin Dashboard

```
/content-admin/dashboard
├── /content-admin/overview
│   └── Content Metrics
├── /content-admin/products
│   ├── Product List
│   ├── Add Product
│   ├── Edit Product
│   ├── Categories
│   └── Bulk Import
├── /content-admin/reviews
│   ├── Review Queue
│   ├── Pending Reviews
│   ├── Published Reviews
│   └── Flagged Reviews
├── /content-admin/seo
│   ├── SEO Dashboard
│   ├── Meta Tags Manager
│   ├── Sitemap Generator
│   └── Keywords Tracker
├── /content-admin/publishing
│   ├── Publishing Calendar
│   ├── Scheduled Posts
│   └── Draft Queue
└── /content-admin/analytics
    ├── Content Performance
    ├── Top Products
    └── Engagement Metrics
```

### 5.3 Business User Dashboard (Retail)

```
/business/dashboard
├── /business/overview
│   ├── Campaign Summary
│   ├── Revenue Summary
│   ├── Quick Actions
│   └── Notifications
├── /business/campaigns
│   ├── My Campaigns
│   ├── Create Campaign
│   ├── Edit Campaign
│   ├── Campaign Analytics
│   └── Budget Management
├── /business/offers
│   ├── My Offers
│   ├── Create Offer
│   ├── Edit Offer
│   └── Offer Analytics
├── /business/vendors
│   ├── My Listings
│   ├── Create Listing
│   ├── Edit Listing
│   └── Listing Analytics
├── /business/affiliates
│   ├── Affiliate Links
│   ├── Create Link
│   └── Affiliate Reports
├── /business/sponsored-content
│   ├── My Content
│   ├── Submit Content
│   └── Content Analytics
├── /business/analytics
│   ├── Performance Dashboard
│   ├── ROI Analysis
│   ├── Audience Insights
│   └── Comparison Reports
├── /business/billing
│   ├── Invoices
│   ├── Payments
│   ├── Payment Methods
│   └── Transaction History
└── /business/settings
    ├── Profile
    ├── Company Info
    ├── Notification Preferences
    └── API Integration
```

### 5.4 SME Reviewer Dashboard

```
/sme/dashboard
├── /sme/overview
│   ├── Pending Reviews
│   ├── Completed Reviews
│   └── Performance Stats
├── /sme/review-queue
│   ├── All Drafts
│   ├── Priority Queue
│   ├── My Assignments
│   └── Filter Options
├── /sme/review-editor
│   ├── AI Summary
│   ├── Edit Form
│   ├── Pros/Cons Editor
│   ├── Quality Score
│   └── Approve/Reject
├── /sme/analytics
│   ├── My Performance
│   ├── Review Stats
│   └── Quality Metrics
└── /sme/settings
    └── Preferences
```

### 5.5 Marketing Team Dashboard

```
/marketing/dashboard
├── /marketing/overview
│   ├── Traffic Summary
│   ├── Conversion Metrics
│   ├── Campaign Performance
│   └── Growth Trends
├── /marketing/seo
│   ├── SEO Dashboard
│   ├── Keywords Rankings
│   ├── Backlinks Monitor
│   └── Competitor Analysis
├── /marketing/aeo
│   ├── Answer Engine Optimization
│   ├── Featured Snippets
│   ├── Voice Search
│   └── Schema Testing
├── /marketing/campaigns
│   ├── Marketing Campaigns
│   ├── A/B Tests
│   ├── Email Campaigns
│   └── Social Media
├── /marketing/analytics
│   ├── Traffic Sources
│   ├── User Behavior
│   ├── Conversion Funnels
│   └── Attribution
└── /marketing/reports
    ├── Monthly Reports
    ├── Quarterly Reports
    └── Custom Reports
```

### 5.6 Sales Dashboard

```
/sales/dashboard
├── /sales/overview
│   ├── Sales Pipeline
│   ├── Revenue Forecast
│   └── Target vs Actual
├── /sales/leads
│   ├── Lead List
│   ├── Add Lead
│   ├── Lead Details
│   └── Lead Scoring
├── /sales/clients
│   ├── Client List
│   ├── Client Details
│   ├── Subscription History
│   └── Upgrade Opportunities
├── /sales/deals
│   ├── Active Deals
│   ├── Deal Pipeline
│   ├── Won/Lost Deals
│   └── Deal Value Tracker
├── /sales/pricing
│   ├── Pricing Plans
│   ├── ROI Calculator
│   ├── Proposal Generator
│   └── Quote Builder
└── /sales/reports
    ├── Sales Performance
    ├── Conversion Reports
    └── Revenue Reports
```

### 5.7 Finance Dashboard

```
/finance/dashboard
├── /finance/overview
│   ├── Revenue Summary
│   ├── Outstanding Payments
│   ├── Cash Flow
│   └── Profit/Loss
├── /finance/revenue
│   ├── Revenue by Stream
│   ├── Monthly Revenue
│   ├── Revenue Forecast
│   └── Growth Analysis
├── /finance/billing
│   ├── Invoice Management
│   ├── Generate Invoice
│   ├── Pending Invoices
│   └── Paid Invoices
├── /finance/payments
│   ├── Payment Tracking
│   ├── Payment Methods
│   ├── Failed Payments
│   └── Refunds
├── /finance/disputes
│   ├── Dispute Center
│   ├── Open Disputes
│   ├── Resolved Disputes
│   └── Chargeback Management
├── /finance/reports
│   ├── P&L Statement
│   ├── Balance Sheet
│   ├── Tax Reports
│   ├── Audit Reports
│   └── Custom Reports
└── /finance/compliance
    ├── Compliance Checklist
    ├── GST Reports
    ├── TDS Reports
    └── Export Data
```

### 5.8 ICT Dashboard

```
/ict/dashboard
├── /ict/overview
│   ├── Pending Approvals
│   ├── Active Listings
│   └── Quality Scores
├── /ict/corporate-sales
│   ├── Corporate Clients
│   ├── Proposals
│   ├── Negotiations
│   └── Contracts
├── /ict/approvals
│   ├── Campaign Approvals
│   ├── Offer Approvals
│   ├── Vendor Approvals
│   └── Content Approvals
├── /ict/design
│   ├── Design Requests
│   ├── Assign to Partners
│   ├── Review Designs
│   └── Client Approval Workflow
├── /ict/moderation
│   ├── Moderation Queue
│   ├── Flagged Content
│   ├── Quality Checks
│   └── Compliance Checks
├── /ict/listing-quality
│   ├── Quality Dashboard
│   ├── Scoring System
│   ├── Improvement Suggestions
│   └── Quality Reports
└── /ict/reports
    ├── Approval Reports
    ├── Compliance Reports
    └── Quality Reports
```

### 5.9 Blog Admin Dashboard

```
/blog-admin/dashboard
├── /blog-admin/overview
│   ├── Blog Metrics
│   ├── Publishing Schedule
│   └── Top Posts
├── /blog-admin/authors
│   ├── Author List
│   ├── Add Author
│   ├── Author Permissions
│   └── Author Performance
├── /blog-admin/workflow
│   ├── Pending Review
│   ├── Scheduled Posts
│   ├── Published Posts
│   └── Draft Posts
├── /blog-admin/editorial-calendar
│   ├── Calendar View
│   ├── Schedule Post
│   └── Content Planning
├── /blog-admin/seo
│   ├── SEO Checklist
│   ├── Meta Editor
│   ├── Keyword Manager
│   └── Readability Scores
├── /blog-admin/monetization
│   ├── Affiliate Performance
│   ├── Link Management
│   └── Revenue Reports
└── /blog-admin/analytics
    ├── Traffic Analysis
    ├── Engagement Metrics
    └── Top Performing Content
```

### 5.10 Blog Editor Dashboard

```
/editor/dashboard
├── /editor/overview
│   ├── My Posts
│   ├── Drafts
│   └── Performance
├── /editor/write
│   ├── New Post
│   ├── AI Writing Assistant
│   ├── Content Scaffolding
│   └── Grammar Check
├── /editor/drafts
│   ├── All Drafts
│   ├── Edit Draft
│   ├── Preview
│   └── Submit for Review
├── /editor/templates
│   ├── Blog Templates
│   ├── How-to Template
│   ├── Listicle Template
│   └── Guide Template
├── /editor/revisions
│   ├── Revision History
│   ├── Compare Versions
│   └── Restore Version
├── /editor/media
│   ├── Media Library
│   ├── Upload Media
│   └── Media Manager
└── /editor/analytics
    ├── My Post Performance
    ├── Page Views
    ├── Engagement
    └── Shares
```

### 5.11 Influencer Dashboard

```
/influencer/dashboard
├── /influencer/overview
│   ├── Profile Summary
│   ├── Recent Activity
│   ├── Earnings
│   └── Business Leads
├── /influencer/profile
│   ├── Public Profile
│   ├── Edit Profile
│   ├── Portfolio
│   └── Social Links
├── /influencer/platforms
│   ├── Connected Platforms
│   ├── Add Platform
│   ├── Sync Status
│   └── Platform Analytics
├── /influencer/content
│   ├── My Reviews
│   ├── Submit Review
│   ├── Edit Review
│   └── Content Calendar
├── /influencer/analytics
│   ├── Performance Dashboard
│   ├── View Analytics
│   ├── Engagement Metrics
│   └── Audience Insights
├── /influencer/business-leads
│   ├── Lead Inbox
│   ├── Active Collaborations
│   ├── Completed Projects
│   └── Lead Pipeline
├── /influencer/reputation
│   ├── User Feedback
│   ├── Ratings & Reviews
│   ├── Response Management
│   └── Reputation Score
└── /influencer/earnings
    ├── Revenue Dashboard
    ├── Payment History
    ├── Payout Settings
    └── Tax Information
```

### 5.12 Support Agent Dashboard

```
/support/dashboard
├── /support/overview
│   ├── Ticket Queue
│   ├── My Tickets
│   ├── SLA Status
│   └── Performance Metrics
├── /support/tickets
│   ├── All Tickets
│   ├── Open Tickets
│   ├── Assigned to Me
│   ├── Waiting on Customer
│   └── Resolved Tickets
├── /support/ticket-detail
│   ├── Ticket Information
│   ├── Message Thread
│   ├── Quick Actions
│   ├── Escalate
│   └── Close Ticket
├── /support/knowledge-base
│   ├── FAQs
│   ├── Articles
│   ├── Canned Responses
│   └── Documentation
└── /support/reports
    ├── Resolution Time
    ├── SLA Compliance
    ├── Customer Satisfaction
    └── Agent Performance
```

### 5.13 Support Admin Dashboard

```
/support-admin/dashboard
├── /support-admin/overview
│   ├── Team Performance
│   ├── Ticket Statistics
│   └── SLA Compliance
├── /support-admin/team
│   ├── Agent List
│   ├── Add Agent
│   ├── Agent Performance
│   └── Workload Balancing
├── /support-admin/configuration
│   ├── Ticket Categories
│   ├── SLA Rules
│   ├── Escalation Rules
│   └── Auto-responses
├── /support-admin/reports
│   ├── Performance Reports
│   ├── SLA Reports
│   ├── CSAT Reports
│   └── Ticket Aging
└── /support-admin/compliance
    ├── Audit Logs
    ├── Data Retention
    ├── Export Logs
    └── GDPR/DPDP Tools
```

---

## 6. Security Implementation

### 6.1 Authentication & Authorization

```typescript
// JWT-based authentication
interface JWTPayload {
  user_id: string;
  email: string;
  role: UserRole;
  permissions: string[];
  iat: number;
  exp: number;
}

// Role-based access control
enum UserRole {
  END_USER = 'end_user',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin',
  CONTENT_ADMIN = 'content_admin',
  SME = 'sme',
  BUSINESS = 'business',
  INFLUENCER = 'influencer',
  MARKETING = 'marketing',
  SALES = 'sales',
  FINANCE = 'finance',
  ICT = 'ict',
  BLOG_ADMIN = 'blog_admin',
  BLOG_EDITOR = 'blog_editor',
  SUPPORT_AGENT = 'support_agent',
  SUPPORT_ADMIN = 'support_admin'
}

// Permission middleware
const requirePermission = (permission: string) => {
  return async (req, res, next) => {
    const user = req.user;
    
    if (!user || !user.permissions.includes(permission)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    next();
  };
};

// Usage
app.post('/api/v1/admin/products', 
  authenticate,
  requirePermission('products.create'),
  createProduct
);
```

### 6.2 Data Protection

- **Encryption at Rest**: AES-256 for sensitive data
- **Encryption in Transit**: TLS 1.3
- **Password Hashing**: bcrypt with salt
- **API Keys**: Hashed in database
- **PII Protection**: Tokenization for sensitive user data

### 6.3 Compliance

- **GDPR**: Right to access, delete, export data
- **DPDP (India)**: Data localization, consent management
- **PCI DSS**: For payment processing (use compliant gateway)
- **CCPA**: California privacy rights

---

## 7. Performance Optimization

### 7.1 Frontend Performance

- **Code Splitting**: Dynamic imports
- **Lazy Loading**: Images, components
- **CDN**: Static assets, images
- **Caching**: Service workers, browser cache
- **Image Optimization**: WebP, responsive images
- **Minification**: CSS, JS

### 7.2 Backend Performance

- **Database Indexing**: Strategic indexes
- **Query Optimization**: N+1 prevention
- **Connection Pooling**: Database connections
- **Caching**: Redis for hot data
- **Async Processing**: Queue-based jobs
- **API Rate Limiting**: Protect from abuse

### 7.3 Monitoring

- **APM**: Application performance monitoring
- **Error Tracking**: Sentry / Bugsnag
- **Logging**: Structured logging (JSON)
- **Alerting**: PagerDuty / OpsGenie
- **Metrics**: Prometheus + Grafana

---

## 8. Deployment Strategy

### 8.1 Infrastructure

```
Production Environment:
- Cloud Provider: AWS / Google Cloud
- Region: Multi-region (India, US, EU)
- Load Balancer: AWS ALB
- Web Servers: ECS / Kubernetes
- Database: RDS PostgreSQL (Multi-AZ)
- Cache: ElastiCache Redis (Cluster mode)
- Storage: S3 / CloudFlare R2
- CDN: CloudFlare

Staging Environment:
- Single region
- Smaller instance sizes
- Shared resources

Development Environment:
- Docker Compose
- Local databases
- Mock services
```

### 8.2 CI/CD Pipeline

```yaml
# GitHub Actions example
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test
      
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker image
        run: docker build -t opinionflow:${{ github.sha }} .
      - name: Push to registry
        run: docker push opinionflow:${{ github.sha }}
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: kubectl set image deployment/opinionflow opinionflow=opinionflow:${{ github.sha }}
```

---

## 9. Risk Mitigation

### 9.1 Technical Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Database performance issues | High | Proper indexing, query optimization, read replicas |
| Ad engine latency | High | Caching, async processing, fallback mechanisms |
| Translation quality | Medium | Human review for critical content, AI + professional mix |
| Third-party API failures | Medium | Circuit breakers, fallback providers, retry logic |
| Security vulnerabilities | High | Regular audits, dependency updates, penetration testing |
| Scalability bottlenecks | High | Horizontal scaling, load testing, performance monitoring |

### 9.2 Business Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Low user adoption | High | MVP launch, user feedback, iterative improvement |
| Revenue model failure | High | Multiple revenue streams, flexible pricing |
| Content quality issues | High | SME validation, moderation queue, quality scoring |
| Competition | Medium | Unique features, superior UX, fast iteration |

---

## 10. Development Phases

### Phase 1: Foundation (Weeks 1-4)
- Database setup
- Authentication system
- Basic API structure
- Admin dashboard skeleton

### Phase 2: Core Features (Weeks 5-8)
- Product management
- Review system
- Basic ad engine
- Business user dashboard

### Phase 3: Advanced Features (Weeks 9-12)
- Full ad engine
- Offer & vendor systems
- Analytics dashboards
- Multilingual support

### Phase 4: Polish & Launch (Weeks 13-16)
- All dashboard completion
- Performance optimization
- Security hardening
- Testing & bug fixes

---

This technical specification provides a comprehensive blueprint for implementing the OpinionFlow platform with scalable, secure, and maintainable architecture.
