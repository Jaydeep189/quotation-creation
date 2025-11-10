# OpinionFlow - Product Review Platform Documentation

## Overview

This repository contains the technical documentation and specifications for **OpinionFlow**, a product review aggregation and monetization platform optimized for the Indian market.

---

## 📁 Documentation Structure

### Essential Documents

1. **[DASHBOARD_SITEMAP.md](opinionflow/DASHBOARD_SITEMAP.md)** ⭐ *PRIMARY DOCUMENT*
   - Comprehensive dashboard navigation
   - Detailed user actions per page
   - Complete feature specifications
   - Role-based access details

2. **[DATABASE_SCHEMA.md](opinionflow/DATABASE_SCHEMA.md)**
   - JSON-based relational schema
   - Visual table relationships
   - Dashboard-focused queries
   - Indian language support structure

3. **[AD_ENGINE.md](opinionflow/AD_ENGINE.md)**
   - Custom ad engine specification
   - Basic targeting capabilities
   - Implementation effort estimates
   - Performance targets

4. **[DELIVERABLES_SUMMARY.md](DELIVERABLES_SUMMARY.md)**
   - Project scope summary
   - Technology stack
   - Development effort breakdown

---

## 🎯 Project Scope

### Core Components

- **Dashboards:** 3 primary dashboards (60+ pages total)
  - Super Admin Dashboard
  - Content Admin Dashboard
  - Business User Dashboard

- **Ad Engine:** Custom advertising system
  - Basic targeting (geography, device, category, time)
  - CPM, CPC, and flat-rate pricing models
  - Real-time tracking and analytics

- **Database:** PostgreSQL with JSONB
  - Flexible schema design
  - Optimized for dashboard queries
  - Indian language translation support

---

## 🛠️ Technology Stack

### Frontend
- **Public Website:** Astro.js (static site generation)
- **Dashboards:** SvelteKit (reactive framework)
- **Styling:** Tailwind CSS
- **Charts:** Recharts / Chart.js

### Backend
- **API Server:** Golang (high performance)
- **Database:** PostgreSQL 15+
- **Cache:** Redis 7+
- **Search:** Elasticsearch 8+ OR PostgreSQL full-text

### Infrastructure
- **Cloud:** AWS / Google Cloud / Vercel
- **CDN:** Cloudflare
- **Storage:** AWS S3 / Cloudflare R2
- **CI/CD:** GitHub Actions

---

## 🌐 Multilingual Support

### Indian Languages Focus

The platform is designed with **Indian languages** as the primary translation target:

- Hindi (हिंदी)
- Bengali (বাংলা)
- Telugu (తెలుగు)
- Tamil (தமிழ்)
- Marathi (मराठी)
- Gujarati (ગુજરાતી)
- Kannada (ಕನ್ನಡ)
- Malayalam (മലയാളം)

### Translation Approach

The platform uses **Sarvam.ai Text Translation API** as the primary translation provider, offering superior translation quality for Indian languages compared to traditional services like DeepL (which is optimized for European languages).

**Why Sarvam.ai?**
- Specialized in Indian native languages with deep understanding of cultural context
- Built specifically for the Indian market with better linguistic accuracy
- Supports all major Indian languages with high-quality real-time translation
- More suitable than European-focused translation services

### Implementation Strategy

1. **Primary Translation Provider: Sarvam.ai API**
   - Real-time API-based translation using Sarvam.ai Text Translation API (https://www.sarvam.ai/text)
   - Automatic translation of content (products, blogs, offers, UI strings)
   - Intelligent caching mechanism to reduce API calls and improve performance
   - Rate limiting and quota management for cost optimization

2. **Translation Caching:**
   - Translated content cached in database for instant retrieval
   - Cache invalidation on source content updates
   - Reduces API costs while maintaining freshness

3. **Manual Override Capability:**
   - Critical content can be manually reviewed and overridden
   - Quality assurance process for important translations
   - Professional translators can refine AI translations when needed

### Translation Architecture

The database schema includes a dedicated `translations` table that supports:
- Entity type (product, blog, offer, etc.)
- Entity ID
- Field name (title, description, etc.)
- Target language
- Cached translation content
- Translation method (api_cached, manual_override)
- Source API metadata (provider, version, timestamp)
- Quality verification flags

**Technical Configuration:**
- API key management for Sarvam.ai
- Rate limiting and request throttling
- Automatic retry logic with exponential backoff
- Fallback to cached translations on API failures
- Analytics for translation usage and costs

---

## 💰 Effort Estimation

### Dashboard Development
- **Super Admin:** 2.5 weeks
- **Content Admin:** 1.5 weeks
- **Business User:** 2.0 weeks
- **Total:** 6 weeks

### Ad Engine Development
- **Core Engine:** 4.5 weeks
- **Dashboard Integration:** 1.5 weeks
- **Testing:** 1 week
- **Total:** 7 weeks (can run in parallel)

### Database Setup
- **Schema Implementation:** 1 week
- **Data Migration Tools:** 1 week
- **Total:** 2 weeks

### **Total Project Duration:** ~10-12 weeks (with parallel development)

---

## 📊 Ad Engine Scope

### Basic-Level Features (MVP)
✅ Geographic targeting (country, city)  
✅ Device type targeting (mobile, tablet, desktop)  
✅ Category/Product targeting  
✅ Time-based scheduling (day, hour)  
✅ CPM, CPC, Flat-rate pricing  
✅ Budget and pacing management  
✅ Real-time impression and click tracking  
✅ Basic analytics dashboard  

### Deferred Features (Future Phases)
❌ Advanced demographic targeting  
❌ Behavioral retargeting  
❌ Custom audiences  
❌ A/B testing framework  
❌ Conversion tracking  
❌ Advanced fraud detection  

---

## 🚀 Getting Started

### For Developers
1. Review **DASHBOARD_SITEMAP.md** for feature requirements
2. Review **DATABASE_SCHEMA.md** for data structure
3. Review **AD_ENGINE.md** for ad system specifications
4. Set up development environment based on technology stack

### For Stakeholders
1. Review **DELIVERABLES_SUMMARY.md** for project overview
2. Review **DASHBOARD_SITEMAP.md** for detailed functionality
3. Provide feedback on scope and priorities

---

## 📝 Document Status

| Document | Version | Status | Last Updated |
|----------|---------|--------|--------------|
| DASHBOARD_SITEMAP.md | 2.0 | ✅ Complete | Nov 10, 2024 |
| DATABASE_SCHEMA.md | 2.0 | ✅ Complete | Nov 10, 2024 |
| AD_ENGINE.md | 1.0 | ✅ Complete | Nov 10, 2024 |
| DELIVERABLES_SUMMARY.md | 2.0 | 🔄 Updated | Nov 10, 2024 |

---

## 🔗 Additional Resources

- [Technology Stack Rationale](#technology-stack)
- [Indian Language Translation Strategy](#multilingual-support)
- [Ad Engine Performance Targets](opinionflow/AD_ENGINE.md#key-performance-targets)
- [Database Query Examples](opinionflow/DATABASE_SCHEMA.md#dashboard-specific-queries)

---

## 📞 Contact

For questions or clarifications about the specifications:
- Technical queries → Review technical documents first
- Scope questions → Refer to DASHBOARD_SITEMAP.md
- Database questions → Refer to DATABASE_SCHEMA.md

---

**Project:** OpinionFlow Platform  
**Repository:** Jaydeep189/quotation-creation  
**Documentation Version:** 2.0  
**Last Updated:** November 10, 2024
