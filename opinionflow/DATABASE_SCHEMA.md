# OpinionFlow - Database Schema

## Overview

This document defines the relational database schema for OpinionFlow, optimized for dashboard requirements. The schema uses **PostgreSQL** as the primary database with JSONB fields for flexible data storage.

---

## Database Technology

- **Primary Database:** PostgreSQL 15+
- **Cache Layer:** Redis 7+
- **Full-text Search:** Elasticsearch 8+ OR PostgreSQL full-text search

---

## Table Relationships Graph

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CORE ENTITIES                               │
└─────────────────────────────────────────────────────────────────────┘

            ┌─────────────┐
            │    USERS    │◄───────────────┐
            │  (Central)  │                │
            └──────┬──────┘                │
                   │                       │
        ┌──────────┼──────────┬───────────┼────────────┐
        │          │          │           │            │
        ▼          ▼          ▼           ▼            ▼
  ┌─────────┐  ┌────────┐  ┌──────┐  ┌────────┐  ┌──────────┐
  │ REVIEWS │  │ BLOGS  │  │OFFERS│  │CAMPAIGNS│ │INFLUENCER│
  └────┬────┘  └───┬────┘  └──┬───┘  └───┬────┘  └────┬─────┘
       │           │          │          │            │
       │      ┌────┴──────────┴──────────┴─────┐      │
       │      │                                 │      │
       ▼      ▼                                 ▼      ▼
  ┌─────────────┐                      ┌────────────────┐
  │  PRODUCTS   │                      │  AD_CREATIVES  │
  │  (Central)  │                      │                │
  └──────┬──────┘                      └────────────────┘
         │
         │
  ┌──────┴───────┐
  │              │
  ▼              ▼
┌──────────┐  ┌────────────┐
│CATEGORIES│  │  VENDORS   │
└──────────┘  └────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    SUPPORTING ENTITIES                              │
└─────────────────────────────────────────────────────────────────────┘

    USERS  ──────►  INVOICES  ──────►  PAYMENTS
      │
      └──────────►  SUPPORT_TICKETS  ──────►  MESSAGES
      │
      └──────────►  SESSIONS


    CAMPAIGNS  ────►  AD_IMPRESSIONS  ────►  AD_CLICKS
                             │
                             └──────────────►  AD_ANALYTICS


    PRODUCTS  ─────►  TRANSLATIONS
    BLOGS     ─────►  TRANSLATIONS
    OFFERS    ─────►  TRANSLATIONS

```

---

## JSON-Based Schema Definitions

### 1. User Management

#### users
```json
{
  "table": "users",
  "description": "Core user accounts for all roles",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "email": {"type": "VARCHAR(255)", "unique": true, "required": true},
    "phone": {"type": "VARCHAR(20)", "unique": true},
    "password_hash": {"type": "VARCHAR(255)", "required": true},
    "first_name": {"type": "VARCHAR(100)"},
    "last_name": {"type": "VARCHAR(100)"},
    "role": {
      "type": "ENUM",
      "values": ["end_user", "admin", "super_admin", "content_admin", "sme", "business", "influencer", "marketing", "sales", "finance", "ict"],
      "required": true
    },
    "status": {
      "type": "ENUM",
      "values": ["active", "inactive", "suspended", "pending"],
      "default": "pending"
    },
    "email_verified": {"type": "BOOLEAN", "default": false},
    "phone_verified": {"type": "BOOLEAN", "default": false},
    "preferred_language": {"type": "VARCHAR(10)", "default": "en"},
    "location": {
      "type": "JSONB",
      "structure": {
        "country": "VARCHAR(2)",
        "city": "VARCHAR(100)",
        "lat": "DECIMAL(10,8)",
        "lng": "DECIMAL(11,8)"
      }
    },
    "avatar_url": {"type": "TEXT"},
    "kyc_verified": {"type": "BOOLEAN", "default": false},
    "kyc_documents": {"type": "JSONB"},
    "metadata": {
      "type": "JSONB",
      "description": "Additional flexible user data"
    },
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "last_login_at": {"type": "TIMESTAMP"},
    "deleted_at": {"type": "TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["email"], "type": "btree"},
    {"columns": ["role"], "type": "btree"},
    {"columns": ["status"], "type": "btree"},
    {"columns": ["location"], "type": "gin"}
  ],
  "relationships": {
    "has_many": ["sessions", "reviews", "blogs", "campaigns", "offers", "tickets"],
    "has_one": ["preferences", "influencer_profile", "business_profile"]
  }
}
```

#### user_sessions
```json
{
  "table": "user_sessions",
  "description": "Active user sessions for authentication",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "user_id": {"type": "UUID", "references": "users.id", "on_delete": "CASCADE"},
    "token_hash": {"type": "VARCHAR(255)", "required": true},
    "device_info": {
      "type": "JSONB",
      "structure": {
        "device_type": "VARCHAR(50)",
        "browser": "VARCHAR(100)",
        "os": "VARCHAR(100)",
        "ip_address": "INET",
        "user_agent": "TEXT"
      }
    },
    "expires_at": {"type": "TIMESTAMP", "required": true},
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["user_id"], "type": "btree"},
    {"columns": ["token_hash"], "type": "btree"},
    {"columns": ["expires_at"], "type": "btree"}
  ],
  "relationships": {
    "belongs_to": "users"
  }
}
```

---

### 2. Product Management

#### products
```json
{
  "table": "products",
  "description": "Product catalog for reviews and content",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "name": {"type": "VARCHAR(500)", "required": true},
    "slug": {"type": "VARCHAR(600)", "unique": true, "required": true},
    "category_id": {"type": "UUID", "references": "categories.id"},
    "brand": {"type": "VARCHAR(200)"},
    "sku": {"type": "VARCHAR(100)", "unique": true},
    "description": {"type": "TEXT"},
    "short_description": {"type": "VARCHAR(500)"},
    "specifications": {
      "type": "JSONB",
      "description": "Product specs as key-value pairs"
    },
    "pricing": {
      "type": "JSONB",
      "structure": {
        "mrp": "DECIMAL(12,2)",
        "selling_price": "DECIMAL(12,2)",
        "currency": "VARCHAR(3)"
      }
    },
    "images": {
      "type": "JSONB",
      "description": "Array of image URLs"
    },
    "status": {
      "type": "ENUM",
      "values": ["draft", "pending", "approved", "rejected", "inactive"],
      "default": "draft"
    },
    "seo_metadata": {
      "type": "JSONB",
      "structure": {
        "meta_title": "VARCHAR(200)",
        "meta_description": "VARCHAR(500)",
        "keywords": "TEXT[]",
        "og_image": "TEXT"
      }
    },
    "rating_aggregate": {
      "type": "JSONB",
      "structure": {
        "average_rating": "DECIMAL(3,2)",
        "total_reviews": "INTEGER",
        "rating_distribution": "JSONB"
      }
    },
    "created_by": {"type": "UUID", "references": "users.id"},
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "published_at": {"type": "TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["slug"], "type": "btree"},
    {"columns": ["category_id"], "type": "btree"},
    {"columns": ["status"], "type": "btree"},
    {"columns": ["brand"], "type": "btree"},
    {"columns": ["name"], "type": "gin", "using": "gin_trgm_ops"}
  ],
  "relationships": {
    "belongs_to": ["categories", "users"],
    "has_many": ["reviews", "variants", "translations"]
  }
}
```

#### categories
```json
{
  "table": "categories",
  "description": "Product categories hierarchy",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "name": {"type": "VARCHAR(200)", "required": true},
    "slug": {"type": "VARCHAR(250)", "unique": true, "required": true},
    "parent_id": {"type": "UUID", "references": "categories.id"},
    "description": {"type": "TEXT"},
    "icon_url": {"type": "TEXT"},
    "image_url": {"type": "TEXT"},
    "display_order": {"type": "INTEGER", "default": 0},
    "status": {
      "type": "ENUM",
      "values": ["active", "inactive"],
      "default": "active"
    },
    "seo_metadata": {"type": "JSONB"},
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["slug"], "type": "btree"},
    {"columns": ["parent_id"], "type": "btree"},
    {"columns": ["status", "display_order"], "type": "btree"}
  ],
  "relationships": {
    "belongs_to": "categories (self-reference for parent)",
    "has_many": ["products", "subcategories (self-reference)"]
  }
}
```

---

### 3. Review System

#### reviews
```json
{
  "table": "reviews",
  "description": "Product reviews and ratings",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "product_id": {"type": "UUID", "references": "products.id", "required": true},
    "user_id": {"type": "UUID", "references": "users.id"},
    "reviewer_name": {"type": "VARCHAR(200)"},
    "rating": {
      "type": "JSONB",
      "structure": {
        "overall": "DECIMAL(2,1)",
        "quality": "DECIMAL(2,1)",
        "value": "DECIMAL(2,1)",
        "features": "DECIMAL(2,1)",
        "design": "DECIMAL(2,1)"
      },
      "required": true
    },
    "title": {"type": "VARCHAR(300)"},
    "content": {"type": "TEXT", "required": true},
    "pros": {"type": "JSONB", "description": "Array of pros"},
    "cons": {"type": "JSONB", "description": "Array of cons"},
    "verified_purchase": {"type": "BOOLEAN", "default": false},
    "ai_summary": {"type": "TEXT"},
    "quality_score": {"type": "DECIMAL(3,2)"},
    "status": {
      "type": "ENUM",
      "values": ["pending", "approved", "rejected", "flagged"],
      "default": "pending"
    },
    "moderation_notes": {"type": "TEXT"},
    "helpful_count": {"type": "INTEGER", "default": 0},
    "unhelpful_count": {"type": "INTEGER", "default": 0},
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "published_at": {"type": "TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["product_id"], "type": "btree"},
    {"columns": ["user_id"], "type": "btree"},
    {"columns": ["status"], "type": "btree"},
    {"columns": ["created_at"], "type": "btree"},
    {"columns": ["content"], "type": "gin", "using": "gin_trgm_ops"}
  ],
  "relationships": {
    "belongs_to": ["products", "users"],
    "has_many": ["votes"]
  }
}
```

---

### 4. Advertising System

#### ad_campaigns
```json
{
  "table": "ad_campaigns",
  "description": "Advertising campaigns",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "business_user_id": {"type": "UUID", "references": "users.id", "required": true},
    "name": {"type": "VARCHAR(300)", "required": true},
    "description": {"type": "TEXT"},
    "campaign_type": {
      "type": "ENUM",
      "values": ["banner", "sponsored_content", "video", "native"],
      "required": true
    },
    "status": {
      "type": "ENUM",
      "values": ["draft", "pending_approval", "active", "paused", "completed", "rejected"],
      "default": "draft"
    },
    "targeting": {
      "type": "JSONB",
      "structure": {
        "countries": "TEXT[]",
        "cities": "TEXT[]",
        "age_range": {"min": "INTEGER", "max": "INTEGER"},
        "gender": "VARCHAR(20)",
        "interests": "TEXT[]",
        "product_categories": "TEXT[]",
        "page_types": "TEXT[]",
        "devices": "TEXT[]",
        "days_of_week": "INTEGER[]",
        "hours_of_day": "INTEGER[]"
      }
    },
    "budget": {
      "type": "JSONB",
      "structure": {
        "total": "DECIMAL(12,2)",
        "daily": "DECIMAL(12,2)",
        "currency": "VARCHAR(3)"
      },
      "required": true
    },
    "pricing_model": {
      "type": "ENUM",
      "values": ["CPM", "CPC", "FLAT"],
      "required": true
    },
    "pricing_rate": {"type": "DECIMAL(10,2)"},
    "impressions_quota": {"type": "INTEGER"},
    "clicks_quota": {"type": "INTEGER"},
    "start_date": {"type": "DATE", "required": true},
    "end_date": {"type": "DATE", "required": true},
    "priority": {"type": "INTEGER", "default": 5},
    "stats": {
      "type": "JSONB",
      "structure": {
        "impressions": "INTEGER",
        "clicks": "INTEGER",
        "spent": "DECIMAL(12,2)",
        "conversions": "INTEGER"
      },
      "default": "{'impressions': 0, 'clicks': 0, 'spent': 0, 'conversions': 0}"
    },
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "approved_at": {"type": "TIMESTAMP"},
    "approved_by": {"type": "UUID", "references": "users.id"}
  },
  "indexes": [
    {"columns": ["business_user_id"], "type": "btree"},
    {"columns": ["status", "start_date", "end_date"], "type": "btree"},
    {"columns": ["targeting"], "type": "gin"}
  ],
  "relationships": {
    "belongs_to": "users (business_user_id)",
    "has_many": ["ad_creatives", "ad_impressions", "ad_clicks"]
  }
}
```

#### ad_impressions
```json
{
  "table": "ad_impressions",
  "description": "Track ad impressions for billing and analytics",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "campaign_id": {"type": "UUID", "references": "ad_campaigns.id", "required": true},
    "creative_id": {"type": "UUID", "references": "ad_creatives.id"},
    "user_id": {"type": "UUID", "references": "users.id"},
    "session_id": {"type": "VARCHAR(100)"},
    "page_url": {"type": "TEXT"},
    "page_type": {"type": "VARCHAR(100)"},
    "device_type": {"type": "VARCHAR(50)"},
    "location": {
      "type": "JSONB",
      "structure": {
        "country": "VARCHAR(2)",
        "city": "VARCHAR(100)"
      }
    },
    "viewable": {"type": "BOOLEAN", "default": false},
    "view_duration": {"type": "INTEGER"},
    "impression_time": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP", "required": true}
  },
  "indexes": [
    {"columns": ["campaign_id", "impression_time"], "type": "btree"},
    {"columns": ["session_id", "creative_id"], "type": "btree"},
    {"columns": ["impression_time"], "type": "brin"}
  ],
  "relationships": {
    "belongs_to": ["ad_campaigns", "ad_creatives", "users"]
  }
}
```

#### ad_clicks
```json
{
  "table": "ad_clicks",
  "description": "Track ad clicks for billing and analytics",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "campaign_id": {"type": "UUID", "references": "ad_campaigns.id", "required": true},
    "creative_id": {"type": "UUID", "references": "ad_creatives.id"},
    "impression_id": {"type": "UUID", "references": "ad_impressions.id"},
    "user_id": {"type": "UUID", "references": "users.id"},
    "session_id": {"type": "VARCHAR(100)"},
    "ip_address": {"type": "INET"},
    "device_type": {"type": "VARCHAR(50)"},
    "click_time": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP", "required": true},
    "cost": {"type": "DECIMAL(10,4)"}
  },
  "indexes": [
    {"columns": ["campaign_id", "click_time"], "type": "btree"},
    {"columns": ["impression_id"], "type": "btree"}
  ],
  "relationships": {
    "belongs_to": ["ad_campaigns", "ad_creatives", "ad_impressions", "users"]
  }
}
```

---

### 5. Offers Management

#### offers
```json
{
  "table": "offers",
  "description": "Promotional offers and deals",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "business_user_id": {"type": "UUID", "references": "users.id", "required": true},
    "title": {"type": "VARCHAR(300)", "required": true},
    "slug": {"type": "VARCHAR(350)", "unique": true},
    "description": {"type": "TEXT"},
    "offer_type": {
      "type": "ENUM",
      "values": ["discount", "cashback", "bundle", "freebie"],
      "required": true
    },
    "offer_value": {
      "type": "JSONB",
      "structure": {
        "type": "ENUM(percentage, flat)",
        "amount": "DECIMAL(10,2)",
        "currency": "VARCHAR(3)"
      }
    },
    "offer_code": {"type": "VARCHAR(50)", "unique": true},
    "terms": {"type": "TEXT"},
    "images": {"type": "JSONB"},
    "products": {
      "type": "JSONB",
      "description": "Array of product IDs"
    },
    "validity": {
      "type": "JSONB",
      "structure": {
        "start_date": "DATE",
        "end_date": "DATE"
      },
      "required": true
    },
    "redemption": {
      "type": "JSONB",
      "structure": {
        "limit_total": "INTEGER",
        "limit_per_user": "INTEGER",
        "min_purchase": "DECIMAL(10,2)",
        "count_total": "INTEGER",
        "count_unique_users": "INTEGER"
      }
    },
    "status": {
      "type": "ENUM",
      "values": ["draft", "pending_approval", "active", "expired", "rejected"],
      "default": "draft"
    },
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["business_user_id"], "type": "btree"},
    {"columns": ["slug"], "type": "btree"},
    {"columns": ["offer_code"], "type": "btree"},
    {"columns": ["status"], "type": "btree"}
  ],
  "relationships": {
    "belongs_to": "users (business_user_id)",
    "has_many": "offer_redemptions"
  }
}
```

---

### 6. Blog Management

#### blog_posts
```json
{
  "table": "blog_posts",
  "description": "Blog content management",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "author_id": {"type": "UUID", "references": "users.id", "required": true},
    "title": {"type": "VARCHAR(300)", "required": true},
    "slug": {"type": "VARCHAR(350)", "unique": true, "required": true},
    "excerpt": {"type": "VARCHAR(500)"},
    "content": {"type": "TEXT", "required": true},
    "featured_image": {"type": "TEXT"},
    "category": {"type": "VARCHAR(100)"},
    "tags": {"type": "JSONB"},
    "status": {
      "type": "ENUM",
      "values": ["draft", "pending", "published", "archived"],
      "default": "draft"
    },
    "seo_metadata": {"type": "JSONB"},
    "reading_time": {"type": "INTEGER"},
    "view_count": {"type": "INTEGER", "default": 0},
    "featured": {"type": "BOOLEAN", "default": false},
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "published_at": {"type": "TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["author_id"], "type": "btree"},
    {"columns": ["slug"], "type": "btree"},
    {"columns": ["status", "published_at"], "type": "btree"},
    {"columns": ["category"], "type": "btree"}
  ],
  "relationships": {
    "belongs_to": "users (author_id)",
    "has_many": ["comments", "translations"]
  }
}
```

---

### 7. Financial Management

#### invoices
```json
{
  "table": "invoices",
  "description": "Billing invoices for business users",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "invoice_number": {"type": "VARCHAR(50)", "unique": true, "required": true},
    "business_user_id": {"type": "UUID", "references": "users.id", "required": true},
    "invoice_type": {
      "type": "ENUM",
      "values": ["campaign", "subscription", "vendor_listing", "other"],
      "required": true
    },
    "line_items": {
      "type": "JSONB",
      "description": "Array of invoice line items",
      "structure": {
        "description": "VARCHAR(500)",
        "quantity": "INTEGER",
        "unit_price": "DECIMAL(10,2)",
        "amount": "DECIMAL(10,2)"
      }
    },
    "subtotal": {"type": "DECIMAL(12,2)", "required": true},
    "tax": {
      "type": "JSONB",
      "structure": {
        "rate": "DECIMAL(5,2)",
        "amount": "DECIMAL(12,2)"
      }
    },
    "total": {"type": "DECIMAL(12,2)", "required": true},
    "currency": {"type": "VARCHAR(3)", "default": "INR"},
    "status": {
      "type": "ENUM",
      "values": ["draft", "sent", "paid", "overdue", "cancelled"],
      "default": "draft"
    },
    "due_date": {"type": "DATE"},
    "paid_at": {"type": "TIMESTAMP"},
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["invoice_number"], "type": "btree"},
    {"columns": ["business_user_id"], "type": "btree"},
    {"columns": ["status"], "type": "btree"},
    {"columns": ["due_date"], "type": "btree"}
  ],
  "relationships": {
    "belongs_to": "users (business_user_id)",
    "has_many": "payments"
  }
}
```

#### payments
```json
{
  "table": "payments",
  "description": "Payment transactions",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "invoice_id": {"type": "UUID", "references": "invoices.id"},
    "user_id": {"type": "UUID", "references": "users.id", "required": true},
    "payment_method": {
      "type": "ENUM",
      "values": ["card", "bank_transfer", "upi", "wallet", "other"],
      "required": true
    },
    "amount": {"type": "DECIMAL(12,2)", "required": true},
    "currency": {"type": "VARCHAR(3)", "default": "INR"},
    "status": {
      "type": "ENUM",
      "values": ["pending", "processing", "completed", "failed", "refunded"],
      "default": "pending"
    },
    "transaction_id": {"type": "VARCHAR(200)"},
    "gateway": {"type": "VARCHAR(100)"},
    "gateway_response": {"type": "JSONB"},
    "payment_metadata": {"type": "JSONB"},
    "paid_at": {"type": "TIMESTAMP"},
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["invoice_id"], "type": "btree"},
    {"columns": ["user_id"], "type": "btree"},
    {"columns": ["transaction_id"], "type": "btree"},
    {"columns": ["status"], "type": "btree"}
  ],
  "relationships": {
    "belongs_to": ["invoices", "users"]
  }
}
```

---

### 8. Support System

#### support_tickets
```json
{
  "table": "support_tickets",
  "description": "Customer support tickets",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "ticket_number": {"type": "VARCHAR(50)", "unique": true, "required": true},
    "user_id": {"type": "UUID", "references": "users.id"},
    "subject": {"type": "VARCHAR(300)", "required": true},
    "description": {"type": "TEXT", "required": true},
    "category": {
      "type": "ENUM",
      "values": ["technical", "billing", "content", "account", "other"],
      "required": true
    },
    "priority": {
      "type": "ENUM",
      "values": ["low", "medium", "high", "urgent"],
      "default": "medium"
    },
    "status": {
      "type": "ENUM",
      "values": ["open", "in_progress", "waiting_user", "resolved", "closed"],
      "default": "open"
    },
    "assigned_to": {"type": "UUID", "references": "users.id"},
    "attachments": {"type": "JSONB"},
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "resolved_at": {"type": "TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["ticket_number"], "type": "btree"},
    {"columns": ["user_id"], "type": "btree"},
    {"columns": ["status"], "type": "btree"},
    {"columns": ["assigned_to"], "type": "btree"}
  ],
  "relationships": {
    "belongs_to": ["users (user_id)", "users (assigned_to)"],
    "has_many": "support_messages"
  }
}
```

---

### 9. Translation System

#### translations
```json
{
  "table": "translations",
  "description": "Multilingual content translations (Indian languages focus)",
  "columns": {
    "id": {"type": "UUID", "primary": true},
    "entity_type": {
      "type": "ENUM",
      "values": ["product", "category", "blog", "offer", "ui_string"],
      "required": true
    },
    "entity_id": {"type": "UUID", "required": true},
    "field_name": {"type": "VARCHAR(100)", "required": true},
    "language": {
      "type": "VARCHAR(10)",
      "required": true,
      "description": "hi, bn, te, ta, mr, gu, kn, ml, etc."
    },
    "translation": {"type": "TEXT", "required": true},
    "translation_method": {
      "type": "ENUM",
      "values": ["manual", "ai", "community"],
      "default": "manual"
    },
    "quality_score": {"type": "DECIMAL(3,2)"},
    "translator_id": {"type": "UUID", "references": "users.id"},
    "created_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"},
    "updated_at": {"type": "TIMESTAMP", "default": "CURRENT_TIMESTAMP"}
  },
  "indexes": [
    {"columns": ["entity_type", "entity_id", "language", "field_name"], "type": "btree", "unique": true},
    {"columns": ["language"], "type": "btree"}
  ],
  "relationships": {
    "belongs_to": "users (translator_id)"
  }
}
```

---

## Key Features

### JSONB Flexibility
All schema definitions use JSONB fields for:
- Flexible metadata storage
- Complex nested data structures
- Easy schema evolution
- Efficient querying with GIN indexes

### Relationship Types
- **belongs_to:** Foreign key relationship (many-to-one)
- **has_many:** Reverse foreign key relationship (one-to-many)
- **has_one:** One-to-one relationship

### Index Strategy
- **btree:** Standard B-tree indexes for equality and range queries
- **gin:** Generalized Inverted Index for JSONB and full-text search
- **brin:** Block Range Index for time-series data (impressions, clicks)
- **gin_trgm_ops:** Trigram indexes for fuzzy text search

---

## Dashboard-Specific Queries

### Super Admin Dashboard

```sql
-- User Statistics
SELECT 
  role,
  status,
  COUNT(*) as user_count
FROM users
GROUP BY role, status;

-- Revenue Metrics
SELECT 
  DATE(created_at) as date,
  SUM(total) as daily_revenue
FROM invoices
WHERE status = 'paid'
GROUP BY DATE(created_at)
ORDER BY date DESC;

-- Campaign Performance
SELECT 
  status,
  COUNT(*) as campaign_count,
  SUM((stats->>'impressions')::int) as total_impressions,
  SUM((stats->>'clicks')::int) as total_clicks,
  SUM((stats->>'spent')::decimal) as total_spent
FROM ad_campaigns
GROUP BY status;
```

### Business User Dashboard

```sql
-- My Campaign Stats
SELECT 
  id,
  name,
  status,
  (stats->>'impressions')::int as impressions,
  (stats->>'clicks')::int as clicks,
  (stats->>'spent')::decimal as spent,
  (budget->>'total')::decimal as budget
FROM ad_campaigns
WHERE business_user_id = :user_id
ORDER BY created_at DESC;

-- Campaign Performance Over Time
SELECT 
  DATE(impression_time) as date,
  COUNT(*) as impressions,
  COUNT(DISTINCT session_id) as unique_views
FROM ad_impressions
WHERE campaign_id = :campaign_id
GROUP BY DATE(impression_time)
ORDER BY date;
```

### Content Admin Dashboard

```sql
-- Product Status Summary
SELECT 
  status,
  COUNT(*) as product_count
FROM products
GROUP BY status;

-- Review Queue
SELECT 
  r.id,
  r.product_id,
  p.name as product_name,
  r.reviewer_name,
  r.rating->>'overall' as rating,
  r.created_at
FROM reviews r
JOIN products p ON r.product_id = p.id
WHERE r.status = 'pending'
ORDER BY r.created_at ASC;
```

---

## Performance Considerations

### Partitioning Strategy
```sql
-- Partition ad_impressions by month for better performance
CREATE TABLE ad_impressions_2024_11 PARTITION OF ad_impressions
FOR VALUES FROM ('2024-11-01') TO ('2024-12-01');

-- Partition ad_clicks similarly
CREATE TABLE ad_clicks_2024_11 PARTITION OF ad_clicks
FOR VALUES FROM ('2024-11-01') TO ('2024-12-01');
```

### Materialized Views for Analytics
```sql
-- Daily campaign analytics
CREATE MATERIALIZED VIEW campaign_daily_stats AS
SELECT 
  campaign_id,
  DATE(impression_time) as date,
  COUNT(*) as impressions,
  COUNT(DISTINCT user_id) as unique_users,
  AVG(view_duration) as avg_view_duration
FROM ad_impressions
GROUP BY campaign_id, DATE(impression_time);

-- Refresh strategy: Once per hour
CREATE INDEX ON campaign_daily_stats (campaign_id, date);
```

---

## Backup & Maintenance

### Backup Strategy
- **Full backup:** Daily at 2 AM
- **Incremental backup:** Every 6 hours
- **WAL archiving:** Continuous
- **Retention:** 30 days

### Maintenance Tasks
- **Vacuum:** Weekly on large tables
- **Analyze:** Daily during off-peak hours
- **Index rebuild:** Monthly for heavily updated tables
- **Partition management:** Automated monthly rotation

---

**Document Version:** 2.0  
**Last Updated:** November 10, 2024  
**Status:** Complete - Ready for Implementation
**Focus:** Dashboard Requirements & Indian Language Support
