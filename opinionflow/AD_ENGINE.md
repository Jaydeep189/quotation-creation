# OpinionFlow - Ad Engine Specification

## Overview

This document provides a comprehensive specification for the custom advertising engine that will power OpinionFlow's monetization strategy. The ad engine is designed to be lightweight, efficient, and scalable, with basic targeting capabilities suitable for initial launch.

---

## 1. Architecture

### 1.1 System Flow

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

---

## 2. Core Components

### 2.1 Ad Server API

**Technology Stack:** Golang + Redis + PostgreSQL

**Endpoints:**
```
GET  /api/v1/ads/serve          - Get ad for placement
POST /api/v1/ads/impression     - Track impression
POST /api/v1/ads/click          - Track click & redirect
GET  /api/v1/ads/analytics      - Get campaign analytics
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

---

### 2.2 Basic Targeting Engine

**Supported Targeting Criteria:**

```go
type TargetingCriteria struct {
    // Geographic
    Countries []string         `json:"countries,omitempty"`
    Cities    []string         `json:"cities,omitempty"`
    
    // Demographics
    AgeRange  *AgeRange        `json:"age_range,omitempty"`
    Gender    string           `json:"gender,omitempty"` // "male", "female", "other", "all"
    
    // Behavioral
    Interests         []string `json:"interests,omitempty"`
    ProductCategories []string `json:"product_categories,omitempty"`
    
    // Contextual
    PageTypes []string         `json:"page_types,omitempty"`
    Keywords  []string         `json:"keywords,omitempty"`
    
    // Technical
    Devices  []string          `json:"devices,omitempty"` // "mobile", "tablet", "desktop"
    
    // Temporal
    DaysOfWeek []int            `json:"days_of_week,omitempty"` // 0-6 (Sunday-Saturday)
    HoursOfDay []int            `json:"hours_of_day,omitempty"` // 0-23
}

type AgeRange struct {
    Min int `json:"min"`
    Max int `json:"max"`
}
```

**Basic Matching Algorithm:**
```go
func MatchCampaigns(request AdRequest, campaigns []Campaign) []ScoredCampaign {
    matched := []ScoredCampaign{}
    
    for _, campaign := range campaigns {
        score := 0
        
        // Geographic match (highest priority)
        if matchesGeography(request, campaign.Targeting) {
            score += 10
        }
        
        // Category match
        if matchesCategory(request, campaign.Targeting) {
            score += 8
        }
        
        // Device match
        if matchesDevice(request, campaign.Targeting) {
            score += 5
        }
        
        // Time-based match
        if matchesSchedule(request, campaign.Targeting) {
            score += 3
        }
        
        if score >= MINIMUM_MATCH_SCORE {
            matched = append(matched, ScoredCampaign{
                Campaign:       campaign,
                RelevanceScore: score,
            })
        }
    }
    
    // Sort by relevance score (descending)
    sort.Slice(matched, func(i, j int) bool {
        return matched[i].RelevanceScore > matched[j].RelevanceScore
    })
    
    return matched
}
```

---

### 2.3 Budget & Pacing System

**Budget Check:**
```go
func CheckBudget(campaign Campaign) (bool, string) {
    // Total budget check
    if campaign.Spent >= campaign.Budget {
        return false, "Budget exhausted"
    }
    
    // Daily budget check
    todaySpent := GetDailySpend(campaign.ID, time.Now())
    if campaign.DailyBudget > 0 && todaySpent >= campaign.DailyBudget {
        return false, "Daily budget exhausted"
    }
    
    // Impression quota check
    if campaign.ImpressionsQuota > 0 && campaign.ImpressionsServed >= campaign.ImpressionsQuota {
        return false, "Impression quota reached"
    }
    
    // Click quota check
    if campaign.ClicksQuota > 0 && campaign.ClicksServed >= campaign.ClicksQuota {
        return false, "Click quota reached"
    }
    
    return true, "OK"
}
```

**Simple Pacing Algorithm:**
```go
func CalculatePacingScore(campaign Campaign) float64 {
    // Ensures even distribution of impressions/budget over campaign duration
    totalDays := campaign.EndDate.Sub(campaign.StartDate).Hours() / 24
    elapsedDays := time.Now().Sub(campaign.StartDate).Hours() / 24
    
    expectedProgress := elapsedDays / totalDays
    actualProgress := campaign.Spent / campaign.Budget
    
    // If underpacing (spending too slowly), increase priority
    if actualProgress < expectedProgress-0.1 {
        return 1.5
    }
    // If overpacing (spending too fast), decrease priority
    if actualProgress > expectedProgress+0.1 {
        return 0.5
    }
    
    return 1.0
}
```

---

### 2.4 Auction & Selection

**eCPM Calculation:**
```go
func CalculateECPM(campaign Campaign, relevanceScore int) float64 {
    var baseECPM float64
    
    switch campaign.PricingModel {
    case "CPM":
        baseECPM = campaign.CPMRate
    case "CPC":
        // Estimate CTR based on historical data
        estimatedCTR := GetEstimatedCTR(campaign)
        baseECPM = campaign.CPCRate * estimatedCTR * 1000
    case "FLAT":
        dailyImpressions := float64(campaign.ImpressionsQuota) / float64(campaign.DurationDays)
        baseECPM = (campaign.FlatFee / float64(campaign.DurationDays)) / dailyImpressions * 1000
    }
    
    // Apply relevance multiplier
    ecpm := baseECPM * (float64(relevanceScore) / 10.0)
    
    // Apply pacing multiplier
    pacingScore := CalculatePacingScore(campaign)
    ecpm *= pacingScore
    
    // Apply priority multiplier
    ecpm *= (1.0 + float64(campaign.Priority)*0.1)
    
    return ecpm
}
```

**Weighted Random Selection:**
```go
func SelectAd(eligibleCampaigns []ScoredCampaign) *Campaign {
    if len(eligibleCampaigns) == 0 {
        return nil
    }
    
    // Calculate weights based on eCPM
    weights := make([]float64, len(eligibleCampaigns))
    totalWeight := 0.0
    
    for i, sc := range eligibleCampaigns {
        // Square eCPM for more aggressive weighting
        weight := math.Pow(sc.ECPM, 2)
        weights[i] = weight
        totalWeight += weight
    }
    
    // Random selection based on weights
    randVal := rand.Float64() * totalWeight
    cumulative := 0.0
    
    for i, weight := range weights {
        cumulative += weight
        if randVal <= cumulative {
            return &eligibleCampaigns[i].Campaign
        }
    }
    
    // Fallback to first campaign
    return &eligibleCampaigns[0].Campaign
}
```

---

### 2.5 Tracking & Attribution

**Impression Tracking:**
```go
func TrackImpression(ctx context.Context, data ImpressionData) error {
    // Validate impression
    if !ValidateImpression(data) {
        return errors.New("invalid impression data")
    }
    
    // Deduplicate (prevent double counting)
    cacheKey := fmt.Sprintf("imp:%s:%s", data.CreativeID, data.SessionID)
    exists, _ := redis.Exists(ctx, cacheKey).Result()
    if exists {
        return nil // Already tracked
    }
    
    redis.SetEx(ctx, cacheKey, "1", time.Hour) // 1 hour TTL
    
    // Log to database
    _, err := db.Exec(`
        INSERT INTO ad_impressions (
            campaign_id, creative_id, user_id, session_id, page_url,
            device_type, location_country, location_city, viewable, 
            view_duration, impression_time
        ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11)
    `, data.CampaignID, data.CreativeID, data.UserID, data.SessionID,
       data.PageURL, data.DeviceType, data.Location.Country, 
       data.Location.City, data.Viewable, data.ViewDuration, time.Now())
    
    if err != nil {
        return err
    }
    
    // Update campaign stats in Redis
    redis.HIncrBy(ctx, fmt.Sprintf("campaign:%s:stats", data.CampaignID), "impressions", 1)
    
    return nil
}
```

**Click Tracking:**
```go
func TrackClick(ctx context.Context, data ClickData) (string, error) {
    // Validate click
    if !ValidateClick(data) {
        return "", errors.New("invalid click data")
    }
    
    // Verify impression exists (click fraud prevention)
    var impressionID string
    err := db.QueryRow(`
        SELECT id FROM ad_impressions 
        WHERE creative_id = $1 AND session_id = $2 
        AND impression_time >= $3
    `, data.CreativeID, data.SessionID, time.Now().Add(-time.Hour)).Scan(&impressionID)
    
    if err != nil {
        log.Warn("Click without impression", "creative_id", data.CreativeID)
        return "", errors.New("no matching impression found")
    }
    
    // Log click
    _, err = db.Exec(`
        INSERT INTO ad_clicks (
            campaign_id, creative_id, impression_id, user_id,
            session_id, click_time, ip_address, device_type
        ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8)
    `, data.CampaignID, data.CreativeID, impressionID, data.UserID,
       data.SessionID, time.Now(), data.IPAddress, data.DeviceType)
    
    if err != nil {
        return "", err
    }
    
    // Update campaign stats
    redis.HIncrBy(ctx, fmt.Sprintf("campaign:%s:stats", data.CampaignID), "clicks", 1)
    
    // Calculate and update cost
    cost := CalculateClickCost(data.CampaignID)
    redis.HIncrByFloat(ctx, fmt.Sprintf("campaign:%s:stats", data.CampaignID), "spent", cost)
    
    // Get redirect URL
    redirectURL := GetCampaignRedirectURL(data.CampaignID, data.CreativeID)
    
    return redirectURL, nil
}
```

---

## 3. Performance Optimization

### 3.1 Caching Strategy

```go
// Cache eligible campaigns per targeting criteria
func GetEligibleCampaigns(ctx context.Context, criteria TargetingCriteria) ([]Campaign, error) {
    cacheKey := fmt.Sprintf("ads:eligible:%s", HashCriteria(criteria))
    
    // Try cache first
    cached, err := redis.Get(ctx, cacheKey).Result()
    if err == nil {
        var campaigns []Campaign
        json.Unmarshal([]byte(cached), &campaigns)
        return campaigns, nil
    }
    
    // Query database
    campaigns, err := db.QueryEligibleCampaigns(criteria)
    if err != nil {
        return nil, err
    }
    
    // Cache for 5 minutes
    data, _ := json.Marshal(campaigns)
    redis.SetEx(ctx, cacheKey, string(data), 5*time.Minute)
    
    return campaigns, nil
}
```

### 3.2 Database Indexing

```sql
-- Critical indexes for ad serving
CREATE INDEX idx_campaigns_active ON ad_campaigns(status, start_date, end_date) 
WHERE status = 'active';

CREATE INDEX idx_campaigns_targeting ON ad_campaigns 
USING GIN (targeting jsonb_path_ops);

CREATE INDEX idx_impressions_session ON ad_impressions(session_id, creative_id, impression_time);

CREATE INDEX idx_clicks_campaign ON ad_clicks(campaign_id, click_time);
```

### 3.3 Load Handling

- Redis for distributed rate limiting
- Circuit breakers for external dependencies
- Async queue processing for analytics aggregation
- Horizontal scaling capability for ad server instances

---

## 4. Effort Estimation

### 4.1 Development Phases

| Phase | Component | Effort | Complexity |
|-------|-----------|--------|------------|
| Phase 1 | Ad Server API Setup | 3 days | Medium |
| Phase 2 | Basic Targeting Engine | 4 days | Medium |
| Phase 3 | Budget & Pacing System | 3 days | Medium |
| Phase 4 | Selection Algorithm | 2 days | Low |
| Phase 5 | Tracking & Analytics | 4 days | Medium-High |
| Phase 6 | Dashboard Integration | 3 days | Medium |
| Phase 7 | Testing & Optimization | 3 days | Medium |
| Phase 8 | Documentation | 1 day | Low |

**Total Estimated Effort: 23 days (approximately 4.5 weeks)**

### 4.2 Team Requirements

- 1 Backend Developer (Golang) - 4 weeks
- 1 Frontend Developer (Dashboard UI) - 1.5 weeks  
- 1 QA Engineer - 1 week

### 4.3 Cost Breakdown

| Resource | Duration | Rate (₹/day) | Cost (₹) |
|----------|----------|--------------|----------|
| Backend Developer | 20 days | 6,000 | 1,20,000 |
| Frontend Developer | 7 days | 5,000 | 35,000 |
| QA Engineer | 5 days | 4,000 | 20,000 |
| **Total** | | | **₹1,75,000** |

---

## 5. Basic-Level Features (MVP)

### 5.1 Targeting Capabilities
- ✅ Geographic targeting (country, city)
- ✅ Device type targeting (mobile, tablet, desktop)
- ✅ Category/Product targeting
- ✅ Time-based scheduling (day, hour)
- ❌ Advanced demographic targeting (deferred)
- ❌ Behavioral retargeting (deferred)
- ❌ Custom audiences (deferred)

### 5.2 Pricing Models
- ✅ CPM (Cost Per Mille/Thousand Impressions)
- ✅ CPC (Cost Per Click)
- ✅ Flat Fee
- ❌ CPA (Cost Per Action) - deferred

### 5.3 Campaign Management
- ✅ Budget setting (total & daily)
- ✅ Impression/click quotas
- ✅ Simple pacing algorithm
- ✅ Campaign scheduling
- ❌ Advanced optimization (deferred)
- ❌ A/B testing (deferred)

### 5.4 Analytics & Reporting
- ✅ Impressions & clicks tracking
- ✅ Spend tracking
- ✅ Basic CTR calculation
- ✅ Real-time stats dashboard
- ❌ Advanced attribution (deferred)
- ❌ Conversion tracking (deferred)

---

## 6. Future Enhancements (Post-MVP)

### Phase 2 Additions:
- Advanced demographic targeting
- Behavioral retargeting
- Lookalike audiences
- Frequency capping
- View-through attribution
- A/B testing framework
- ML-based optimization

### Phase 3 Additions:
- Programmatic bidding
- Header bidding integration
- Video ad support
- Native ad formats
- Advanced fraud detection
- Real-time bidding (RTB)

---

## 7. Technology Stack Summary

| Component | Technology | Rationale |
|-----------|------------|-----------|
| Ad Server | Golang | High performance, low latency |
| Database | PostgreSQL | ACID compliance, JSONB support |
| Cache | Redis | Fast in-memory data store |
| Message Queue | Redis Pub/Sub | Simple, reliable messaging |
| Frontend | SvelteKit | Reactive, efficient dashboard |
| Monitoring | Prometheus + Grafana | Industry standard metrics |

---

## 8. Key Performance Targets

| Metric | Target | Notes |
|--------|--------|-------|
| Ad Serve Latency | < 50ms | P95 |
| Impression Tracking | < 100ms | P95 |
| Click Redirect | < 200ms | P95 |
| Uptime | 99.5% | Excluding maintenance |
| Concurrent Requests | 1000/sec | Initial capacity |
| Data Accuracy | 99.9% | Tracking precision |

---

## 9. Security & Fraud Prevention

### 9.1 Basic Measures
- Impression deduplication (1-hour window)
- Click-to-impression validation
- Rate limiting per IP/session
- Geographic validation
- User agent validation

### 9.2 Advanced Measures (Future)
- ML-based fraud detection
- Bot detection
- Click farm identification
- Viewability tracking
- Invalid traffic filtering

---

## 10. Conclusion

This ad engine specification provides a **solid foundation** for OpinionFlow's monetization strategy. The basic-level implementation focuses on:

1. **Core functionality** - Serving, targeting, and tracking ads
2. **Performance** - Low latency and high throughput
3. **Scalability** - Horizontal scaling capability
4. **Cost-effectiveness** - Approximately ₹1.75L for MVP

The design allows for **incremental enhancement** with advanced features added in future phases based on business needs and revenue growth.

---

**Document Version:** 1.0  
**Last Updated:** November 10, 2024  
**Status:** Ready for Implementation
