# OpinionFlow - Database Schema Design

## Overview
This document outlines the complete database schema for the OpinionFlow platform. The system uses PostgreSQL as the primary relational database for structured data and MongoDB for flexible, document-based storage of reviews and analytics.

---

## Database Technology Stack

- **Primary Database**: PostgreSQL 14+
- **Document Store**: MongoDB 6+
- **Cache Layer**: Redis 7+
- **Search Engine**: Elasticsearch 8+ (for full-text search)

---

## PostgreSQL Schema

### 1. User Management

#### users
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20) UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    role ENUM('end_user', 'admin', 'super_admin', 'content_admin', 'sme', 'business', 'influencer', 'marketing', 'sales', 'finance', 'ict', 'blog_admin', 'blog_editor', 'support_agent', 'support_admin') NOT NULL,
    status ENUM('active', 'inactive', 'suspended', 'pending') DEFAULT 'pending',
    email_verified BOOLEAN DEFAULT FALSE,
    phone_verified BOOLEAN DEFAULT FALSE,
    preferred_language VARCHAR(10) DEFAULT 'en',
    location_country VARCHAR(2),
    location_city VARCHAR(100),
    location_lat DECIMAL(10, 8),
    location_lng DECIMAL(11, 8),
    avatar_url TEXT,
    kyc_verified BOOLEAN DEFAULT FALSE,
    kyc_documents JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_location ON users(location_country, location_city);
```

#### user_preferences
```sql
CREATE TABLE user_preferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    theme VARCHAR(20) DEFAULT 'light',
    notifications_enabled BOOLEAN DEFAULT TRUE,
    email_notifications BOOLEAN DEFAULT TRUE,
    sms_notifications BOOLEAN DEFAULT FALSE,
    favorite_categories JSONB,
    blocked_users JSONB,
    privacy_settings JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_preferences_user_id ON user_preferences(user_id);
```

#### user_sessions
```sql
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL,
    device_type VARCHAR(50),
    browser VARCHAR(100),
    ip_address INET,
    user_agent TEXT,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_token_hash ON user_sessions(token_hash);
CREATE INDEX idx_user_sessions_expires_at ON user_sessions(expires_at);
```

---

### 2. Product Management

#### products
```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(500) NOT NULL,
    slug VARCHAR(600) UNIQUE NOT NULL,
    brand VARCHAR(200),
    category_id UUID REFERENCES categories(id),
    subcategory_id UUID,
    description TEXT,
    specifications JSONB,
    images JSONB,
    official_website TEXT,
    average_rating DECIMAL(3, 2) DEFAULT 0,
    total_reviews INTEGER DEFAULT 0,
    ccs_score DECIMAL(4, 2), -- Consumer Consensus Score
    expectation_score DECIMAL(4, 2), -- Expectation Fulfillment Score
    risk_index DECIMAL(4, 2), -- Risk Index
    confidence_indicator DECIMAL(4, 2), -- Confidence Indicator
    price_range JSONB, -- {min, max, currency}
    availability_status VARCHAR(50),
    meta_title VARCHAR(255),
    meta_description TEXT,
    meta_keywords TEXT,
    structured_data JSONB, -- Schema.org markup
    seo_score INTEGER,
    status ENUM('draft', 'published', 'archived') DEFAULT 'draft',
    view_count INTEGER DEFAULT 0,
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    published_at TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_products_slug ON products(slug);
CREATE INDEX idx_products_category_id ON products(category_id);
CREATE INDEX idx_products_brand ON products(brand);
CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_products_ccs_score ON products(ccs_score DESC);
CREATE INDEX idx_products_created_at ON products(created_at DESC);
```

#### categories
```sql
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(200) NOT NULL,
    slug VARCHAR(250) UNIQUE NOT NULL,
    parent_id UUID REFERENCES categories(id),
    description TEXT,
    icon_url TEXT,
    image_url TEXT,
    display_order INTEGER DEFAULT 0,
    meta_title VARCHAR(255),
    meta_description TEXT,
    is_featured BOOLEAN DEFAULT FALSE,
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_categories_slug ON categories(slug);
CREATE INDEX idx_categories_parent_id ON categories(parent_id);
CREATE INDEX idx_categories_status ON categories(status);
```

#### product_variants
```sql
CREATE TABLE product_variants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id UUID REFERENCES products(id) ON DELETE CASCADE,
    name VARCHAR(200),
    sku VARCHAR(100) UNIQUE,
    specifications JSONB,
    price DECIMAL(12, 2),
    currency VARCHAR(3) DEFAULT 'INR',
    availability BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_product_variants_product_id ON product_variants(product_id);
CREATE INDEX idx_product_variants_sku ON product_variants(sku);
```

---

### 3. Review System

#### reviews
```sql
CREATE TABLE reviews (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id UUID REFERENCES products(id) ON DELETE CASCADE,
    source_type ENUM('youtube', 'tiktok', 'ecommerce', 'forum', 'blog', 'user_submission') NOT NULL,
    source_id VARCHAR(255), -- External source identifier
    source_url TEXT,
    author_name VARCHAR(200),
    author_id UUID REFERENCES users(id), -- If user submission
    influencer_id UUID REFERENCES influencers(id), -- If from influencer
    title VARCHAR(500),
    content TEXT,
    rating DECIMAL(3, 2),
    sentiment VARCHAR(20), -- positive, neutral, negative
    credibility_score DECIMAL(4, 2),
    view_count INTEGER DEFAULT 0,
    helpful_count INTEGER DEFAULT 0,
    not_helpful_count INTEGER DEFAULT 0,
    media_urls JSONB, -- Images/Videos
    language VARCHAR(10) DEFAULT 'en',
    status ENUM('pending', 'approved', 'rejected', 'flagged') DEFAULT 'pending',
    reviewed_by UUID REFERENCES users(id), -- SME who reviewed
    reviewed_at TIMESTAMP,
    published_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_reviews_product_id ON reviews(product_id);
CREATE INDEX idx_reviews_source_type ON reviews(source_type);
CREATE INDEX idx_reviews_status ON reviews(status);
CREATE INDEX idx_reviews_sentiment ON reviews(sentiment);
CREATE INDEX idx_reviews_created_at ON reviews(created_at DESC);
```

#### review_analysis
```sql
CREATE TABLE review_analysis (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    review_id UUID REFERENCES reviews(id) ON DELETE CASCADE,
    overall_sentiment VARCHAR(20),
    pros JSONB, -- Array of strings
    cons JSONB, -- Array of strings
    timestamps JSONB, -- For video reviews
    keywords JSONB, -- Extracted keywords
    promised_vs_reality TEXT, -- Discrepancy analysis
    ai_confidence DECIMAL(4, 2),
    sme_validated BOOLEAN DEFAULT FALSE,
    sme_notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_review_analysis_review_id ON review_analysis(review_id);
```

#### review_votes
```sql
CREATE TABLE review_votes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    review_id UUID REFERENCES reviews(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    vote_type ENUM('helpful', 'not_helpful', 'inappropriate') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(review_id, user_id, vote_type)
);

CREATE INDEX idx_review_votes_review_id ON review_votes(review_id);
CREATE INDEX idx_review_votes_user_id ON review_votes(user_id);
```

---

### 4. Influencer Management

#### influencers
```sql
CREATE TABLE influencers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    display_name VARCHAR(200) NOT NULL,
    slug VARCHAR(250) UNIQUE NOT NULL,
    bio TEXT,
    avatar_url TEXT,
    banner_url TEXT,
    specialization JSONB, -- Array of categories
    total_followers INTEGER DEFAULT 0,
    total_reviews INTEGER DEFAULT 0,
    average_rating DECIMAL(3, 2) DEFAULT 0,
    verification_status ENUM('pending', 'verified', 'rejected') DEFAULT 'pending',
    verification_badge BOOLEAN DEFAULT FALSE,
    social_media_links JSONB,
    website_url TEXT,
    contact_email VARCHAR(255),
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_influencers_user_id ON influencers(user_id);
CREATE INDEX idx_influencers_slug ON influencers(slug);
CREATE INDEX idx_influencers_verification_status ON influencers(verification_status);
```

#### influencer_platforms
```sql
CREATE TABLE influencer_platforms (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    influencer_id UUID REFERENCES influencers(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL, -- youtube, facebook, instagram, reddit, quora, blog
    platform_username VARCHAR(200),
    platform_url TEXT,
    follower_count INTEGER DEFAULT 0,
    is_primary BOOLEAN DEFAULT FALSE,
    last_synced_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_influencer_platforms_influencer_id ON influencer_platforms(influencer_id);
CREATE INDEX idx_influencer_platforms_platform ON influencer_platforms(platform);
```

#### influencer_analytics
```sql
CREATE TABLE influencer_analytics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    influencer_id UUID REFERENCES influencers(id) ON DELETE CASCADE,
    date DATE NOT NULL,
    views INTEGER DEFAULT 0,
    engagement INTEGER DEFAULT 0,
    new_followers INTEGER DEFAULT 0,
    total_followers INTEGER DEFAULT 0,
    reviews_published INTEGER DEFAULT 0,
    average_rating DECIMAL(3, 2),
    revenue DECIMAL(12, 2) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(influencer_id, date)
);

CREATE INDEX idx_influencer_analytics_influencer_id ON influencer_analytics(influencer_id);
CREATE INDEX idx_influencer_analytics_date ON influencer_analytics(date DESC);
```

---

### 5. Advertising System

#### ad_campaigns
```sql
CREATE TABLE ad_campaigns (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_user_id UUID REFERENCES business_users(id) ON DELETE CASCADE,
    name VARCHAR(300) NOT NULL,
    description TEXT,
    campaign_type ENUM('retail', 'corporate') DEFAULT 'retail',
    status ENUM('draft', 'pending_approval', 'approved', 'active', 'paused', 'completed', 'rejected') DEFAULT 'draft',
    budget DECIMAL(12, 2),
    spent DECIMAL(12, 2) DEFAULT 0,
    currency VARCHAR(3) DEFAULT 'INR',
    pricing_model ENUM('cpm', 'cpc', 'flat_fee') NOT NULL,
    cpm_rate DECIMAL(10, 2),
    cpc_rate DECIMAL(10, 2),
    flat_fee DECIMAL(12, 2),
    daily_budget DECIMAL(12, 2),
    targeting JSONB, -- Demographics, location, category, device
    ad_locations JSONB, -- Array of locations
    start_date DATE,
    end_date DATE,
    impressions_quota INTEGER,
    impressions_served INTEGER DEFAULT 0,
    clicks_quota INTEGER,
    clicks_served INTEGER DEFAULT 0,
    priority INTEGER DEFAULT 0,
    submitted_by UUID REFERENCES users(id),
    approved_by UUID REFERENCES users(id),
    approved_at TIMESTAMP,
    rejection_reason TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_ad_campaigns_business_user_id ON ad_campaigns(business_user_id);
CREATE INDEX idx_ad_campaigns_status ON ad_campaigns(status);
CREATE INDEX idx_ad_campaigns_start_date ON ad_campaigns(start_date);
CREATE INDEX idx_ad_campaigns_end_date ON ad_campaigns(end_date);
```

#### ad_creatives
```sql
CREATE TABLE ad_creatives (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_id UUID REFERENCES ad_campaigns(id) ON DELETE CASCADE,
    name VARCHAR(300),
    type ENUM('banner', 'video', 'native', 'sponsored_content') NOT NULL,
    format VARCHAR(50), -- 300x250, 728x90, etc.
    headline VARCHAR(300),
    description TEXT,
    cta_text VARCHAR(100),
    cta_url TEXT NOT NULL,
    image_url TEXT,
    video_url TEXT,
    html_content TEXT,
    status ENUM('active', 'inactive', 'rejected') DEFAULT 'active',
    click_count INTEGER DEFAULT 0,
    impression_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_ad_creatives_campaign_id ON ad_creatives(campaign_id);
CREATE INDEX idx_ad_creatives_status ON ad_creatives(status);
```

#### ad_impressions
```sql
CREATE TABLE ad_impressions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_id UUID REFERENCES ad_campaigns(id) ON DELETE CASCADE,
    creative_id UUID REFERENCES ad_creatives(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id),
    session_id UUID,
    page_url TEXT,
    page_type VARCHAR(50),
    device_type VARCHAR(50),
    browser VARCHAR(100),
    os VARCHAR(100),
    ip_address INET,
    location_country VARCHAR(2),
    location_city VARCHAR(100),
    referrer TEXT,
    impression_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    viewable BOOLEAN DEFAULT TRUE,
    view_duration INTEGER -- seconds
);

CREATE INDEX idx_ad_impressions_campaign_id ON ad_impressions(campaign_id);
CREATE INDEX idx_ad_impressions_creative_id ON ad_impressions(creative_id);
CREATE INDEX idx_ad_impressions_impression_time ON ad_impressions(impression_time DESC);
CREATE INDEX idx_ad_impressions_location ON ad_impressions(location_country, location_city);
```

#### ad_clicks
```sql
CREATE TABLE ad_clicks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_id UUID REFERENCES ad_campaigns(id) ON DELETE CASCADE,
    creative_id UUID REFERENCES ad_creatives(id) ON DELETE CASCADE,
    impression_id UUID REFERENCES ad_impressions(id),
    user_id UUID REFERENCES users(id),
    session_id UUID,
    click_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address INET,
    device_type VARCHAR(50),
    converted BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_ad_clicks_campaign_id ON ad_clicks(campaign_id);
CREATE INDEX idx_ad_clicks_creative_id ON ad_clicks(creative_id);
CREATE INDEX idx_ad_clicks_click_time ON ad_clicks(click_time DESC);
```

#### ad_analytics_daily
```sql
CREATE TABLE ad_analytics_daily (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_id UUID REFERENCES ad_campaigns(id) ON DELETE CASCADE,
    date DATE NOT NULL,
    impressions INTEGER DEFAULT 0,
    clicks INTEGER DEFAULT 0,
    spend DECIMAL(12, 2) DEFAULT 0,
    ctr DECIMAL(5, 4), -- Click-through rate
    cpc DECIMAL(10, 2), -- Cost per click
    cpm DECIMAL(10, 2), -- Cost per mille
    conversions INTEGER DEFAULT 0,
    revenue DECIMAL(12, 2) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(campaign_id, date)
);

CREATE INDEX idx_ad_analytics_daily_campaign_id ON ad_analytics_daily(campaign_id);
CREATE INDEX idx_ad_analytics_daily_date ON ad_analytics_daily(date DESC);
```

---

### 6. Business User & Offers

#### business_users
```sql
CREATE TABLE business_users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    business_type ENUM('retail', 'corporate') NOT NULL,
    company_name VARCHAR(300) NOT NULL,
    business_registration_number VARCHAR(100),
    gst_number VARCHAR(20),
    pan_number VARCHAR(20),
    address TEXT,
    city VARCHAR(100),
    state VARCHAR(100),
    country VARCHAR(2),
    postal_code VARCHAR(20),
    contact_person VARCHAR(200),
    contact_email VARCHAR(255),
    contact_phone VARCHAR(20),
    website_url TEXT,
    industry VARCHAR(100),
    company_size VARCHAR(50),
    verification_status ENUM('pending', 'verified', 'rejected') DEFAULT 'pending',
    verification_documents JSONB,
    subscription_plan_id UUID REFERENCES subscription_plans(id),
    subscription_start_date DATE,
    subscription_end_date DATE,
    subscription_status ENUM('active', 'expired', 'cancelled') DEFAULT 'active',
    assigned_sales_rep UUID REFERENCES users(id), -- For corporate
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_business_users_user_id ON business_users(user_id);
CREATE INDEX idx_business_users_business_type ON business_users(business_type);
CREATE INDEX idx_business_users_verification_status ON business_users(verification_status);
CREATE INDEX idx_business_users_subscription_plan_id ON business_users(subscription_plan_id);
```

#### subscription_plans
```sql
CREATE TABLE subscription_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(120) UNIQUE NOT NULL,
    description TEXT,
    plan_type ENUM('basic', 'pro', 'enterprise') NOT NULL,
    price DECIMAL(12, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'INR',
    billing_cycle ENUM('monthly', 'quarterly', 'yearly') NOT NULL,
    features JSONB, -- Array of features
    ad_credits INTEGER DEFAULT 0,
    offer_slots INTEGER DEFAULT 0,
    vendor_listings INTEGER DEFAULT 0,
    priority_support BOOLEAN DEFAULT FALSE,
    analytics_access BOOLEAN DEFAULT TRUE,
    api_access BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_subscription_plans_slug ON subscription_plans(slug);
CREATE INDEX idx_subscription_plans_plan_type ON subscription_plans(plan_type);
```

#### offers
```sql
CREATE TABLE offers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_user_id UUID REFERENCES business_users(id) ON DELETE CASCADE,
    title VARCHAR(500) NOT NULL,
    slug VARCHAR(600) UNIQUE NOT NULL,
    description TEXT,
    offer_type ENUM('discount', 'coupon', 'deal', 'free_shipping', 'bundle') NOT NULL,
    discount_value DECIMAL(10, 2),
    discount_type ENUM('percentage', 'fixed') DEFAULT 'percentage',
    coupon_code VARCHAR(100),
    terms_conditions TEXT,
    product_id UUID REFERENCES products(id),
    category_id UUID REFERENCES categories(id),
    image_url TEXT,
    landing_page_url TEXT NOT NULL,
    pricing_tier ENUM('basic', 'pro', 'enterprise') NOT NULL,
    tier_price DECIMAL(12, 2),
    location_targeting JSONB,
    start_date DATE,
    end_date DATE,
    status ENUM('draft', 'pending_approval', 'active', 'expired', 'paused', 'rejected') DEFAULT 'draft',
    click_count INTEGER DEFAULT 0,
    redemption_count INTEGER DEFAULT 0,
    view_count INTEGER DEFAULT 0,
    submitted_by UUID REFERENCES users(id),
    approved_by UUID REFERENCES users(id),
    approved_at TIMESTAMP,
    rejection_reason TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_offers_business_user_id ON offers(business_user_id);
CREATE INDEX idx_offers_slug ON offers(slug);
CREATE INDEX idx_offers_status ON offers(status);
CREATE INDEX idx_offers_start_date ON offers(start_date);
CREATE INDEX idx_offers_end_date ON offers(end_date);
CREATE INDEX idx_offers_product_id ON offers(product_id);
```

#### local_vendors
```sql
CREATE TABLE local_vendors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_user_id UUID REFERENCES business_users(id) ON DELETE CASCADE,
    vendor_name VARCHAR(300) NOT NULL,
    description TEXT,
    address TEXT NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100),
    country VARCHAR(2) NOT NULL,
    postal_code VARCHAR(20),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    phone VARCHAR(20),
    email VARCHAR(255),
    website_url TEXT,
    product_categories JSONB, -- Array of category IDs
    pricing_model ENUM('cpc', 'cpl', 'flat_fee', 'subscription') NOT NULL,
    cpc_rate DECIMAL(10, 2),
    cpl_rate DECIMAL(10, 2),
    flat_fee DECIMAL(12, 2),
    subscription_plan VARCHAR(50),
    premium_listing BOOLEAN DEFAULT FALSE,
    logo_url TEXT,
    images JSONB,
    business_hours JSONB,
    rating DECIMAL(3, 2) DEFAULT 0,
    total_reviews INTEGER DEFAULT 0,
    status ENUM('draft', 'pending_approval', 'active', 'paused', 'rejected') DEFAULT 'draft',
    click_count INTEGER DEFAULT 0,
    lead_count INTEGER DEFAULT 0,
    approved_by UUID REFERENCES users(id),
    approved_at TIMESTAMP,
    rejection_reason TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_local_vendors_business_user_id ON local_vendors(business_user_id);
CREATE INDEX idx_local_vendors_city ON local_vendors(city);
CREATE INDEX idx_local_vendors_status ON local_vendors(status);
CREATE INDEX idx_local_vendors_location ON local_vendors(latitude, longitude);
```

---

### 7. Affiliate & E-commerce Links

#### affiliate_programs
```sql
CREATE TABLE affiliate_programs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(200) NOT NULL,
    platform VARCHAR(100), -- Amazon, Flipkart, etc.
    api_key TEXT,
    api_secret TEXT,
    commission_rate DECIMAL(5, 2),
    cookie_duration INTEGER, -- Days
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_affiliate_programs_platform ON affiliate_programs(platform);
```

#### affiliate_links
```sql
CREATE TABLE affiliate_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_user_id UUID REFERENCES business_users(id),
    affiliate_program_id UUID REFERENCES affiliate_programs(id),
    product_id UUID REFERENCES products(id),
    blog_post_id UUID REFERENCES blog_posts(id),
    original_url TEXT NOT NULL,
    affiliate_url TEXT NOT NULL,
    tracking_code VARCHAR(200),
    placement_type ENUM('blog', 'review', 'offer', 'product_page') NOT NULL,
    click_count INTEGER DEFAULT 0,
    conversion_count INTEGER DEFAULT 0,
    revenue DECIMAL(12, 2) DEFAULT 0,
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_affiliate_links_product_id ON affiliate_links(product_id);
CREATE INDEX idx_affiliate_links_blog_post_id ON affiliate_links(blog_post_id);
CREATE INDEX idx_affiliate_links_business_user_id ON affiliate_links(business_user_id);
```

---

### 8. Blog Management

#### blog_posts
```sql
CREATE TABLE blog_posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(500) NOT NULL,
    slug VARCHAR(600) UNIQUE NOT NULL,
    excerpt TEXT,
    content TEXT NOT NULL,
    featured_image TEXT,
    author_id UUID REFERENCES users(id),
    category_id UUID REFERENCES categories(id),
    tags JSONB, -- Array of tags
    status ENUM('draft', 'pending_review', 'published', 'archived') DEFAULT 'draft',
    seo_title VARCHAR(255),
    meta_description TEXT,
    meta_keywords TEXT,
    structured_data JSONB,
    readability_score INTEGER,
    word_count INTEGER,
    reading_time INTEGER, -- Minutes
    view_count INTEGER DEFAULT 0,
    like_count INTEGER DEFAULT 0,
    comment_count INTEGER DEFAULT 0,
    share_count INTEGER DEFAULT 0,
    related_products JSONB, -- Array of product IDs
    monetization_enabled BOOLEAN DEFAULT FALSE,
    published_at TIMESTAMP,
    scheduled_at TIMESTAMP,
    reviewed_by UUID REFERENCES users(id),
    reviewed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_blog_posts_slug ON blog_posts(slug);
CREATE INDEX idx_blog_posts_author_id ON blog_posts(author_id);
CREATE INDEX idx_blog_posts_category_id ON blog_posts(category_id);
CREATE INDEX idx_blog_posts_status ON blog_posts(status);
CREATE INDEX idx_blog_posts_published_at ON blog_posts(published_at DESC);
```

#### blog_comments
```sql
CREATE TABLE blog_comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    blog_post_id UUID REFERENCES blog_posts(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id),
    parent_comment_id UUID REFERENCES blog_comments(id),
    content TEXT NOT NULL,
    status ENUM('pending', 'approved', 'spam', 'deleted') DEFAULT 'pending',
    like_count INTEGER DEFAULT 0,
    moderated_by UUID REFERENCES users(id),
    moderated_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_blog_comments_blog_post_id ON blog_comments(blog_post_id);
CREATE INDEX idx_blog_comments_user_id ON blog_comments(user_id);
CREATE INDEX idx_blog_comments_status ON blog_comments(status);
```

---

### 9. Customer Support

#### support_tickets
```sql
CREATE TABLE support_tickets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ticket_number VARCHAR(50) UNIQUE NOT NULL,
    user_id UUID REFERENCES users(id),
    user_role VARCHAR(50), -- Role of the user creating ticket
    subject VARCHAR(500) NOT NULL,
    description TEXT NOT NULL,
    category VARCHAR(100), -- billing, technical, content, etc.
    priority ENUM('low', 'medium', 'high', 'urgent') DEFAULT 'medium',
    status ENUM('open', 'in_progress', 'waiting_on_customer', 'resolved', 'closed') DEFAULT 'open',
    assigned_to UUID REFERENCES users(id),
    assigned_team VARCHAR(50), -- support, ict, finance, etc.
    attachments JSONB,
    related_entity_type VARCHAR(50), -- campaign, offer, vendor, etc.
    related_entity_id UUID,
    sla_due_at TIMESTAMP,
    first_response_at TIMESTAMP,
    resolved_at TIMESTAMP,
    closed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_support_tickets_ticket_number ON support_tickets(ticket_number);
CREATE INDEX idx_support_tickets_user_id ON support_tickets(user_id);
CREATE INDEX idx_support_tickets_status ON support_tickets(status);
CREATE INDEX idx_support_tickets_assigned_to ON support_tickets(assigned_to);
CREATE INDEX idx_support_tickets_created_at ON support_tickets(created_at DESC);
```

#### support_ticket_messages
```sql
CREATE TABLE support_ticket_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ticket_id UUID REFERENCES support_tickets(id) ON DELETE CASCADE,
    sender_id UUID REFERENCES users(id),
    message TEXT NOT NULL,
    attachments JSONB,
    is_internal BOOLEAN DEFAULT FALSE, -- Internal notes
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_support_ticket_messages_ticket_id ON support_ticket_messages(ticket_id);
CREATE INDEX idx_support_ticket_messages_created_at ON support_ticket_messages(created_at);
```

---

### 10. Financial Management

#### invoices
```sql
CREATE TABLE invoices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invoice_number VARCHAR(50) UNIQUE NOT NULL,
    business_user_id UUID REFERENCES business_users(id),
    billing_entity ENUM('ad_campaign', 'subscription', 'offer', 'vendor_listing') NOT NULL,
    entity_id UUID NOT NULL,
    subtotal DECIMAL(12, 2) NOT NULL,
    tax DECIMAL(12, 2) DEFAULT 0,
    discount DECIMAL(12, 2) DEFAULT 0,
    total DECIMAL(12, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'INR',
    billing_period_start DATE,
    billing_period_end DATE,
    due_date DATE,
    status ENUM('draft', 'sent', 'paid', 'overdue', 'cancelled') DEFAULT 'draft',
    payment_terms TEXT,
    notes TEXT,
    line_items JSONB,
    issued_at TIMESTAMP,
    paid_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_invoices_invoice_number ON invoices(invoice_number);
CREATE INDEX idx_invoices_business_user_id ON invoices(business_user_id);
CREATE INDEX idx_invoices_status ON invoices(status);
CREATE INDEX idx_invoices_due_date ON invoices(due_date);
```

#### payments
```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invoice_id UUID REFERENCES invoices(id),
    business_user_id UUID REFERENCES business_users(id),
    payment_method ENUM('credit_card', 'debit_card', 'upi', 'bank_transfer', 'wallet') NOT NULL,
    payment_gateway VARCHAR(100),
    transaction_id VARCHAR(255),
    amount DECIMAL(12, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'INR',
    status ENUM('pending', 'processing', 'completed', 'failed', 'refunded') DEFAULT 'pending',
    gateway_response JSONB,
    notes TEXT,
    processed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_payments_invoice_id ON payments(invoice_id);
CREATE INDEX idx_payments_business_user_id ON payments(business_user_id);
CREATE INDEX idx_payments_status ON payments(status);
CREATE INDEX idx_payments_transaction_id ON payments(transaction_id);
```

#### revenue_analytics
```sql
CREATE TABLE revenue_analytics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    date DATE NOT NULL,
    revenue_stream ENUM('ads', 'offers', 'vendors', 'affiliates', 'subscriptions', 'sponsored_content') NOT NULL,
    revenue DECIMAL(12, 2) DEFAULT 0,
    transactions INTEGER DEFAULT 0,
    currency VARCHAR(3) DEFAULT 'INR',
    country VARCHAR(2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(date, revenue_stream, country)
);

CREATE INDEX idx_revenue_analytics_date ON revenue_analytics(date DESC);
CREATE INDEX idx_revenue_analytics_revenue_stream ON revenue_analytics(revenue_stream);
```

---

### 11. ERP Integration

#### erp_configurations
```sql
CREATE TABLE erp_configurations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_user_id UUID REFERENCES business_users(id),
    erp_system VARCHAR(100) NOT NULL, -- Odoo, ERPNext, Zoho, SAP
    api_endpoint TEXT NOT NULL,
    auth_type ENUM('oauth2', 'api_key', 'basic') NOT NULL,
    credentials JSONB, -- Encrypted
    entity_mappings JSONB, -- Field mappings
    sync_enabled BOOLEAN DEFAULT TRUE,
    sync_frequency VARCHAR(50), -- hourly, daily, etc.
    last_sync_at TIMESTAMP,
    status ENUM('active', 'inactive', 'error') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_erp_configurations_business_user_id ON erp_configurations(business_user_id);
```

#### erp_sync_logs
```sql
CREATE TABLE erp_sync_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    erp_configuration_id UUID REFERENCES erp_configurations(id),
    sync_type VARCHAR(100), -- offers, invoices, leads, etc.
    records_processed INTEGER DEFAULT 0,
    records_success INTEGER DEFAULT 0,
    records_failed INTEGER DEFAULT 0,
    error_details JSONB,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    status ENUM('running', 'completed', 'failed') DEFAULT 'running',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_erp_sync_logs_erp_configuration_id ON erp_sync_logs(erp_configuration_id);
CREATE INDEX idx_erp_sync_logs_started_at ON erp_sync_logs(started_at DESC);
```

---

### 12. Multilingual Support

#### translations
```sql
CREATE TABLE translations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type VARCHAR(100) NOT NULL, -- product, blog_post, category, etc.
    entity_id UUID NOT NULL,
    field_name VARCHAR(100) NOT NULL, -- title, description, content, etc.
    language VARCHAR(10) NOT NULL,
    translated_value TEXT NOT NULL,
    translation_method ENUM('manual', 'ai', 'professional') DEFAULT 'manual',
    translated_by UUID REFERENCES users(id),
    quality_score INTEGER,
    status ENUM('draft', 'published') DEFAULT 'published',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(entity_type, entity_id, field_name, language)
);

CREATE INDEX idx_translations_entity ON translations(entity_type, entity_id);
CREATE INDEX idx_translations_language ON translations(language);
```

#### supported_languages
```sql
CREATE TABLE supported_languages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(10) UNIQUE NOT NULL, -- en, hi, es, etc.
    name VARCHAR(100) NOT NULL,
    native_name VARCHAR(100) NOT NULL,
    is_rtl BOOLEAN DEFAULT FALSE,
    is_default BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    display_order INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_supported_languages_code ON supported_languages(code);
CREATE INDEX idx_supported_languages_is_active ON supported_languages(is_active);
```

---

### 13. Analytics & Tracking

#### page_views
```sql
CREATE TABLE page_views (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    session_id UUID,
    page_type VARCHAR(50), -- home, product, blog, offer, etc.
    page_id UUID,
    page_url TEXT,
    referrer TEXT,
    device_type VARCHAR(50),
    browser VARCHAR(100),
    os VARCHAR(100),
    ip_address INET,
    location_country VARCHAR(2),
    location_city VARCHAR(100),
    time_on_page INTEGER, -- Seconds
    scroll_depth INTEGER, -- Percentage
    viewed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_page_views_user_id ON page_views(user_id);
CREATE INDEX idx_page_views_page_type ON page_views(page_type, page_id);
CREATE INDEX idx_page_views_viewed_at ON page_views(viewed_at DESC);
```

#### user_actions
```sql
CREATE TABLE user_actions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    session_id UUID,
    action_type VARCHAR(100), -- click, like, share, save, comment, etc.
    entity_type VARCHAR(50),
    entity_id UUID,
    metadata JSONB,
    ip_address INET,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_actions_user_id ON user_actions(user_id);
CREATE INDEX idx_user_actions_action_type ON user_actions(action_type);
CREATE INDEX idx_user_actions_entity ON user_actions(entity_type, entity_id);
CREATE INDEX idx_user_actions_created_at ON user_actions(created_at DESC);
```

---

### 14. System Configuration

#### system_settings
```sql
CREATE TABLE system_settings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    setting_key VARCHAR(200) UNIQUE NOT NULL,
    setting_value TEXT,
    setting_type VARCHAR(50), -- string, number, boolean, json
    category VARCHAR(100),
    description TEXT,
    is_public BOOLEAN DEFAULT FALSE,
    updated_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_system_settings_setting_key ON system_settings(setting_key);
CREATE INDEX idx_system_settings_category ON system_settings(category);
```

#### audit_logs
```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL, -- create, update, delete, login, etc.
    entity_type VARCHAR(100),
    entity_id UUID,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at DESC);
```

---

## MongoDB Collections

### 1. review_content
```javascript
{
    _id: ObjectId,
    review_id: UUID, // References PostgreSQL reviews.id
    full_content: String, // Large text content
    transcript: String, // For video/audio reviews
    parsed_content: Object, // Structured content
    metadata: {
        duration: Number,
        file_size: Number,
        format: String
    },
    created_at: Date,
    updated_at: Date
}
```

### 2. analytics_events
```javascript
{
    _id: ObjectId,
    event_type: String,
    user_id: UUID,
    session_id: UUID,
    timestamp: Date,
    properties: Object, // Flexible event properties
    device: {
        type: String,
        os: String,
        browser: String
    },
    location: {
        country: String,
        city: String,
        lat: Number,
        lng: Number
    }
}
```

### 3. search_queries
```javascript
{
    _id: ObjectId,
    user_id: UUID,
    query: String,
    filters: Object,
    results_count: Number,
    clicked_result_id: UUID,
    click_position: Number,
    timestamp: Date,
    response_time: Number
}
```

### 4. user_behavior_profiles
```javascript
{
    _id: ObjectId,
    user_id: UUID,
    browsing_history: [{
        page_type: String,
        page_id: UUID,
        timestamp: Date
    }],
    interests: [String],
    purchase_intent: Object,
    engagement_score: Number,
    last_updated: Date
}
```

---

## Redis Cache Structure

### 1. Session Cache
```
Key: session:{session_id}
Value: JSON (user_id, role, expires_at, etc.)
TTL: 1 day
```

### 2. Product Cache
```
Key: product:{product_id}
Value: JSON (product details)
TTL: 1 hour
```

### 3. Ad Serving Cache
```
Key: ads:{location}:{targeting_hash}
Value: JSON (eligible campaigns)
TTL: 5 minutes
```

### 4. Rate Limiting
```
Key: ratelimit:{user_id}:{endpoint}
Value: Counter
TTL: 1 hour
```

---

## Elasticsearch Indices

### 1. products_index
```json
{
    "mappings": {
        "properties": {
            "id": {"type": "keyword"},
            "name": {"type": "text", "analyzer": "standard"},
            "description": {"type": "text"},
            "brand": {"type": "keyword"},
            "category": {"type": "keyword"},
            "rating": {"type": "float"},
            "price_range": {"type": "nested"},
            "created_at": {"type": "date"}
        }
    }
}
```

### 2. blog_posts_index
```json
{
    "mappings": {
        "properties": {
            "id": {"type": "keyword"},
            "title": {"type": "text", "analyzer": "standard"},
            "content": {"type": "text"},
            "author": {"type": "keyword"},
            "tags": {"type": "keyword"},
            "published_at": {"type": "date"}
        }
    }
}
```

### 3. reviews_index
```json
{
    "mappings": {
        "properties": {
            "id": {"type": "keyword"},
            "product_id": {"type": "keyword"},
            "content": {"type": "text"},
            "sentiment": {"type": "keyword"},
            "rating": {"type": "float"},
            "created_at": {"type": "date"}
        }
    }
}
```

---

## Database Relationships Summary

1. **Users** → Multiple dashboards/roles
2. **Business Users** → Campaigns, Offers, Vendors, Invoices
3. **Products** → Reviews, Offers, Vendors, Blog Posts
4. **Reviews** → Products, Influencers, Analysis
5. **Campaigns** → Creatives, Impressions, Clicks, Analytics
6. **Blog Posts** → Authors, Categories, Comments, Affiliates
7. **Support Tickets** → Users, Messages, Assignments
8. **Invoices** → Business Users, Payments, Entities

---

## Data Retention Policies

- **Audit Logs**: 2 years
- **Analytics Events**: 1 year (MongoDB), 90 days (Hot Storage)
- **Ad Impressions**: 90 days detailed, 2 years aggregated
- **Page Views**: 30 days detailed, 1 year aggregated
- **User Sessions**: 30 days
- **Cache**: 1 hour to 1 day depending on data type

---

## Security Considerations

1. **Encryption**: All sensitive data (passwords, API keys, payment info) encrypted at rest
2. **Access Control**: Row-level security for multi-tenant data
3. **PII Protection**: GDPR/DPDP compliant data handling
4. **Audit Trail**: All critical operations logged
5. **Data Anonymization**: User data anonymized in analytics after 90 days

---

## Backup Strategy

- **PostgreSQL**: Daily full backups, hourly incremental
- **MongoDB**: Daily snapshots
- **Redis**: Persistence enabled, daily RDB snapshots
- **Retention**: 30 days rolling backups

---

This database schema provides a comprehensive foundation for the OpinionFlow platform, supporting all functional requirements while maintaining scalability, security, and performance.
