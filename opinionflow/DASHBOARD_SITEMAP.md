# OpinionFlow - Comprehensive Dashboard Sitemap

This document provides a detailed sitemap of all dashboards in the OpinionFlow platform, organized by user role with specific user actions documented for each page.

---

## Table of Contents
1. [Super Admin Dashboard](#1-super-admin-dashboard)
2. [Content Admin Dashboard](#2-content-admin-dashboard)
3. [Business User Dashboard](#3-business-user-dashboard)
4. [Dashboard Scope Summary](#dashboard-scope-summary)

---

## 1. Super Admin Dashboard

### 1.1 Overview (`/admin/overview`)
**Purpose:** System health and key metrics at a glance

**User Actions:**
- View system health indicators (uptime, response time, error rates)
- Monitor revenue metrics (daily, weekly, monthly)
- Track user statistics (active users, new registrations, churn rate)
- Review recent activities across the platform
- Access quick links to critical sections
- Filter metrics by date range
- Export dashboard data as PDF/CSV
- Set up custom alerts for metrics

### 1.2 User Management (`/admin/users`)
**Purpose:** Manage all platform users across roles

**User Actions:**
- **User List Page:**
  - View paginated list of all users
  - Filter by role, status, registration date, location
  - Search users by name, email, phone
  - Bulk select users for actions
  - Bulk activate/deactivate/suspend users
  - Export user list to CSV/Excel
  - View user details in quick preview
  
- **User Details Page:**
  - View complete user profile
  - Edit user information (name, email, phone, role)
  - Change user role
  - Activate/deactivate/suspend user account
  - Reset user password
  - View user activity history
  - View user's content (reviews, blogs, campaigns)
  - Add internal notes about user
  - Send email/notification to user
  - View user's financial transactions
  
- **Role Management:**
  - View all role types and permissions
  - Create custom roles (future feature)
  - Assign/modify permissions per role
  - View users per role
  
- **Activity Logs:**
  - View all user activities (login, logout, actions)
  - Filter by user, action type, date range
  - Search logs by keywords
  - Export logs for auditing

### 1.3 Content Management (`/admin/content`)
**Purpose:** Oversee and moderate all platform content

**User Actions:**
- **Products Section:**
  - View all products in the system
  - Filter by category, status, date added
  - Search products by name, SKU
  - View product details
  - Approve/reject pending products
  - Edit product information
  - Deactivate/reactivate products
  - Flag products for review
  
- **Reviews Section:**
  - View all product reviews
  - Filter by status (pending, approved, rejected, flagged)
  - Search reviews by content, author, product
  - Moderate reviews (approve/reject/flag)
  - Edit review content (with audit trail)
  - Ban users for policy violations
  - Respond to reviews as admin
  
- **Blog Posts:**
  - View all blog posts across categories
  - Filter by status, author, category, date
  - Search blog posts
  - Approve/reject pending posts
  - Edit post content
  - Feature/unfeature posts
  - Set post visibility
  
- **Moderation Queue:**
  - View content flagged by users or AI
  - Review flagged items with context
  - Take moderation actions (approve/remove/ban)
  - Set moderation rules
  - View moderation history

### 1.4 Campaign Management (`/admin/campaigns`)
**Purpose:** Oversee all advertising campaigns

**User Actions:**
- **All Campaigns View:**
  - View list of all campaigns (any status)
  - Filter by status, date range, business user, budget
  - Search campaigns by name, ID
  - Quick view campaign details
  - Export campaign list
  
- **Pending Approvals:**
  - Review campaigns awaiting approval
  - View campaign details and targeting
  - Approve campaigns
  - Reject campaigns with reason
  - Request modifications
  - Send feedback to campaign owner
  
- **Active Campaigns:**
  - Monitor running campaigns
  - View real-time performance metrics
  - Pause/resume campaigns
  - View ad impressions and clicks
  - Check budget utilization
  - View targeting effectiveness
  
- **Campaign Analytics:**
  - View overall campaign performance
  - Analyze revenue by campaign type
  - Track CTR, CPC, CPM metrics
  - Generate performance reports
  - Export analytics data

### 1.5 Offers Management (`/admin/offers`)
**Purpose:** Manage promotional offers and deals

**User Actions:**
- View all offers (active, expired, pending)
- Filter by type, status, business user, date
- Search offers by title, code
- Approve/reject pending offers
- Edit offer details
- Extend offer duration
- Deactivate fraudulent offers
- View offer performance (views, clicks, redemptions)
- Export offer data

### 1.6 Vendor Management (`/admin/vendors`)
**Purpose:** Manage vendor listings and partnerships

**User Actions:**
- View all vendor listings
- Filter by status, category, location
- Search vendors by name, product type
- Approve/reject vendor applications
- Verify vendor credentials
- Edit vendor information
- Suspend/ban vendors
- View vendor analytics (views, leads)
- Monitor vendor performance
- Export vendor data

### 1.7 Finance (`/admin/finance`)
**Purpose:** Financial oversight and reporting

**User Actions:**
- **Revenue Dashboard:**
  - View total revenue (daily, monthly, annual)
  - Track revenue by source (campaigns, affiliates, subscriptions)
  - Monitor payment processing
  - View profit margins
  - Access financial graphs and charts
  
- **Invoices:**
  - View all invoices generated
  - Filter by status, date, client
  - Generate new invoices
  - Mark invoices as paid
  - Send invoice reminders
  - Export invoice data
  - Void/cancel invoices
  
- **Payments:**
  - View all payment transactions
  - Filter by status, method, date
  - Process refunds
  - Investigate payment failures
  - Reconcile payments
  - Export payment reports
  
- **Reports:**
  - Generate financial reports (P&L, balance sheet)
  - Schedule automatic reports
  - Export to accounting software
  - View tax reports
  - Analyze payment trends

### 1.8 System Settings (`/admin/system`)
**Purpose:** Configure platform-wide settings

**User Actions:**
- **General Settings:**
  - Update platform name, logo, branding
  - Configure email/SMS settings
  - Set timezone and locale
  - Configure CDN settings
  - Manage integrations
  
- **Pricing Rules:**
  - Set campaign pricing (CPM, CPC rates)
  - Configure commission rates
  - Set platform fees
  - Create pricing tiers
  - Schedule price changes
  
- **Email Templates:**
  - View all email templates
  - Edit template content
  - Preview templates
  - Test email delivery
  - Configure email triggers
  
- **Audit Logs:**
  - View all system changes
  - Filter by user, action, date
  - Search audit logs
  - Export for compliance
  
- **API Keys:**
  - Generate API keys
  - View active API keys
  - Revoke keys
  - Set API rate limits
  - Monitor API usage

### 1.9 Reports (`/admin/reports`)
**Purpose:** Generate comprehensive platform reports

**User Actions:**
- **User Reports:**
  - Generate user growth reports
  - Analyze user engagement
  - Track user retention
  - Export user metrics
  
- **Revenue Reports:**
  - Generate revenue reports by period
  - Analyze revenue sources
  - Compare period-over-period
  - Forecast revenue
  
- **Content Reports:**
  - Track content creation trends
  - Analyze content performance
  - Identify top content
  - Monitor content quality
  
- **System Reports:**
  - Generate uptime reports
  - Analyze system performance
  - Track error rates
  - Monitor resource usage

---

## 2. Content Admin Dashboard

### 2.1 Overview (`/content-admin/overview`)
**Purpose:** Content metrics and performance

**User Actions:**
- View content creation statistics
- Monitor content approval pipeline
- Track content performance metrics
- View trending products/reviews
- Access quick content creation links

### 2.2 Product Management (`/content-admin/products`)
**Purpose:** Manage product catalog

**User Actions:**
- **Product List:**
  - View all products with pagination
  - Filter by category, brand, status, price range
  - Search products by name, SKU, description
  - Sort by date added, popularity, price
  - Quick edit product status
  - Bulk actions (activate, deactivate, delete)
  - Export product list
  
- **Add Product:**
  - Fill product information form
  - Upload product images (multiple)
  - Set product category and subcategory
  - Add product specifications
  - Set SEO metadata
  - Add pricing information
  - Set product visibility
  - Save as draft or publish
  - Preview product page
  
- **Edit Product:**
  - Modify all product fields
  - Update product images
  - Change category/subcategory
  - Update specifications
  - Edit SEO metadata
  - View edit history
  - Revert to previous version
  
- **Categories:**
  - View category tree
  - Add new categories
  - Edit category details
  - Reorder categories
  - Delete unused categories
  - Set category images
  - Configure category SEO
  
- **Bulk Import:**
  - Download import template
  - Upload CSV/Excel file
  - Map fields to database
  - Validate import data
  - Preview import
  - Execute bulk import
  - View import results/errors
  - Export failed records

### 2.3 Review Management (`/content-admin/reviews`)
**Purpose:** Manage product reviews

**User Actions:**
- **Review Queue:**
  - View all reviews awaiting processing
  - Filter by product, reviewer, date
  - Search reviews by content
  - Assign reviews to SME reviewers
  - Prioritize reviews
  - Bulk assign reviews
  
- **Pending Reviews:**
  - Review AI-generated summaries
  - Read full review content
  - Check review quality scores
  - Approve reviews
  - Reject with reason
  - Request revisions
  - Flag for further review
  
- **Published Reviews:**
  - View all live reviews
  - Edit published reviews
  - Unpublish reviews
  - Feature reviews on homepage
  - Pin reviews to product pages
  - Respond to user comments
  
- **Flagged Reviews:**
  - Review flagged content
  - Investigate policy violations
  - Take moderation action
  - Ban repeat offenders
  - Update flagging rules

### 2.4 SEO Tools (`/content-admin/seo`)
**Purpose:** Optimize content for search engines

**User Actions:**
- **SEO Dashboard:**
  - View overall SEO health score
  - Monitor keyword rankings
  - Track organic traffic
  - View search console data
  - Check indexing status
  
- **Meta Tags Manager:**
  - View all pages and meta tags
  - Edit title tags
  - Edit meta descriptions
  - Add Open Graph tags
  - Configure canonical URLs
  - Set robots directives
  - Bulk update meta tags
  
- **Sitemap Generator:**
  - Generate XML sitemap
  - Configure sitemap settings
  - Submit to search engines
  - View sitemap status
  - Schedule automatic updates
  
- **Keywords Tracker:**
  - Add keywords to track
  - Monitor keyword positions
  - Analyze keyword difficulty
  - View keyword trends
  - Get keyword suggestions
  - Track competitor rankings

### 2.5 Publishing (`/content-admin/publishing`)
**Purpose:** Manage content publishing workflow

**User Actions:**
- **Publishing Calendar:**
  - View content schedule (calendar view)
  - Add new scheduled content
  - Drag-and-drop to reschedule
  - Filter by content type
  - View publish history
  - Set recurring publications
  
- **Scheduled Posts:**
  - View all scheduled content
  - Edit scheduled items
  - Change publish time
  - Cancel scheduled publication
  - Publish immediately
  - View scheduling conflicts
  
- **Draft Queue:**
  - View all drafts
  - Resume editing drafts
  - Delete old drafts
  - Convert drafts to scheduled
  - Share drafts for review
  - Bulk publish drafts

### 2.6 Analytics (`/content-admin/analytics`)
**Purpose:** Analyze content performance

**User Actions:**
- **Content Performance:**
  - View page views by content
  - Track engagement metrics
  - Analyze time on page
  - Monitor bounce rates
  - View conversion rates
  
- **Top Products:**
  - View most viewed products
  - Track top converting products
  - Analyze product page performance
  - Identify underperforming products
  
- **Engagement Metrics:**
  - Track user interactions
  - Monitor social shares
  - Analyze comments/reviews
  - View heatmaps
  - Track scroll depth

---

## 3. Business User Dashboard

### 3.1 Overview (`/business/overview`)
**Purpose:** Business performance at a glance

**User Actions:**
- View campaign summary (active, paused, completed)
- Monitor revenue summary (daily, monthly spend)
- View quick action buttons (create campaign, create offer)
- Read platform notifications
- Check budget utilization
- View performance highlights
- Access analytics shortcuts

### 3.2 Campaign Management (`/business/campaigns`)
**Purpose:** Create and manage advertising campaigns

**User Actions:**
- **My Campaigns:**
  - View all campaigns (list/grid view)
  - Filter by status, date, budget
  - Search campaigns
  - Quick view metrics (impressions, clicks, spend)
  - Pause/resume campaigns
  - Duplicate campaigns
  - Archive completed campaigns
  - Share campaign reports
  
- **Create Campaign:**
  - Select campaign type (banner, sponsored content, video)
  - Set campaign name and description
  - Choose targeting options:
    - Geographic (country, cities)
    - Demographic (age, gender - if available)
    - Category/Product
    - Device type
    - Time scheduling
  - Upload creative assets (images, videos)
  - Write ad copy (headline, description, CTA)
  - Set budget (total, daily)
  - Choose pricing model (CPM, CPC, flat fee)
  - Set campaign duration
  - Preview ad placements
  - Save as draft
  - Submit for approval
  
- **Edit Campaign:**
  - Modify campaign details
  - Update targeting parameters
  - Replace creative assets
  - Adjust budget (if allowed)
  - Extend campaign duration
  - View edit restrictions (for active campaigns)
  - Save changes
  - Resubmit for approval if needed
  
- **Campaign Analytics:**
  - View detailed performance metrics
  - Track impressions over time
  - Monitor click-through rates
  - Analyze cost per click/impression
  - View geographic distribution
  - Check device breakdown
  - View hourly/daily performance
  - Compare against benchmarks
  - Export analytics report
  - Download performance graphs
  
- **Budget Management:**
  - View total budget vs. spent
  - Monitor daily spend rate
  - Set spend limits
  - Top up campaign budget
  - View budget forecasts
  - Get budget alerts
  - Adjust pacing settings

### 3.3 Offers Management (`/business/offers`)
**Purpose:** Create promotional offers and deals

**User Actions:**
- **My Offers:**
  - View all offers (active, upcoming, expired)
  - Filter by status, type, date
  - Search offers
  - Quick edit offer status
  - Duplicate offers
  - Archive old offers
  
- **Create Offer:**
  - Set offer title and description
  - Choose offer type (discount, cashback, bundle)
  - Set offer value (percentage/flat discount)
  - Add terms and conditions
  - Set validity period
  - Upload offer images
  - Add participating products
  - Set redemption limits
  - Generate offer code
  - Set minimum purchase amount
  - Preview offer page
  - Submit for approval
  
- **Edit Offer:**
  - Modify offer details
  - Extend validity
  - Update terms
  - Change offer value (if allowed)
  - Update images
  - Add/remove products
  - Save changes
  
- **Offer Analytics:**
  - View offer impressions
  - Track clicks and redemptions
  - Calculate ROI
  - View user demographics
  - Monitor redemption patterns
  - Export offer report

### 3.4 Vendor Listings (`/business/vendors`)
**Purpose:** Manage vendor directory listings

**User Actions:**
- **My Listings:**
  - View all vendor listings
  - Filter by status, category
  - Edit listings
  - View listing analytics
  - Renew expired listings
  
- **Create Listing:**
  - Fill vendor information
  - Add business details
  - Upload business documents
  - Set business hours
  - Add location/map
  - Upload photos
  - Add contact information
  - Select categories
  - Submit for verification
  
- **Edit Listing:**
  - Update vendor information
  - Change photos
  - Update business hours
  - Modify contact details
  - Save changes
  
- **Listing Analytics:**
  - View listing views
  - Track inquiries/leads
  - Monitor phone clicks
  - Check direction requests
  - View visitor demographics

### 3.5 Billing & Payments (`/business/billing`)
**Purpose:** Manage financial transactions

**User Actions:**
- **Invoices:**
  - View all invoices
  - Download invoice PDFs
  - View invoice details
  - Check payment status
  - Request invoice corrections
  
- **Payments:**
  - View payment history
  - Make new payment
  - Set up auto-pay
  - View upcoming payments
  - Download payment receipts
  
- **Payment Methods:**
  - Add credit/debit card
  - Add bank account
  - Set default payment method
  - Remove payment methods
  - Verify payment methods
  
- **Transaction History:**
  - View all transactions
  - Filter by date, type, status
  - Search transactions
  - Export transaction history
  - Dispute transactions

### 3.6 Settings (`/business/settings`)
**Purpose:** Manage account settings

**User Actions:**
- **Profile:**
  - Edit personal information
  - Update email/phone
  - Change password
  - Upload profile photo
  - Set timezone
  
- **Company Info:**
  - Update company name
  - Edit business address
  - Update tax information
  - Upload business documents
  - Verify business
  
- **Notification Preferences:**
  - Enable/disable email notifications
  - Enable/disable SMS alerts
  - Choose notification types
  - Set notification frequency
  - Configure alert thresholds
  
- **API Integration:**
  - Generate API key
  - View API documentation
  - Test API endpoints
  - Monitor API usage
  - Revoke API access

---

## Dashboard Scope Summary

### Total Pages by Dashboard

| Dashboard | Pages | Key Functions |
|-----------|-------|---------------|
| Super Admin | 25+ | System oversight, user management, approvals |
| Content Admin | 15+ | Content creation, SEO, publishing |
| Business User | 20+ | Campaigns, offers, billing |
| **Total Core Dashboards** | **60+** | **Complete platform management** |

### Common Features Across All Dashboards

- **Responsive Design:** Mobile, tablet, desktop optimized
- **Dark/Light Mode:** User preference toggle
- **Advanced Filters:** Multi-criteria filtering on all lists
- **Search Functionality:** Real-time search on all data tables
- **Export Options:** CSV, Excel, PDF export capabilities
- **Bulk Actions:** Select multiple items for batch operations
- **Real-time Updates:** Live data refresh without page reload
- **Role-based Access:** Permissions enforced at page/action level
- **Audit Trail:** All actions logged for compliance
- **Help System:** Contextual help and tooltips
- **Keyboard Shortcuts:** Power user navigation
- **Customizable Views:** Save custom filters and layouts

### Performance Targets

- Page load time: < 2 seconds
- Data table render: < 500ms
- Search response: < 200ms
- API calls: < 100ms
- Real-time updates: < 1 second delay

---

**Document Version:** 2.0  
**Last Updated:** November 10, 2024  
**Status:** Complete - Ready for Development
