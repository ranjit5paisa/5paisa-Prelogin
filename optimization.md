# Simple Page Speed Optimization Steps

## 1️⃣ IPO CALENDAR PAGE

**Current Load Time:** 4+ seconds | **Target:** <2 seconds

### Key Issues:
1. Massive inline JavaScript generation (300+ objects for 100 IPOs)
2. Outdated FullCalendar library from 2013 (370KB unoptimized)
3. Heavy DOM manipulation on page load
4. Full page reload for month navigation

### Steps to Fix:
1. **Replace inline JavaScript loops with JSON** - Stop generating JavaScript objects in Twig template, use single JSON serialization instead
2. **Upgrade FullCalendar to version 6.x** - Use modern CDN version instead of embedded 2013 code (reduces from 370KB to 120KB)
3. **Remove custom navigation that reloads page** - Let FullCalendar handle month navigation in-memory
4. **Create REST API endpoint** - Load only current month's IPO data instead of all data at once
5. **Use event delegation** - Replace multiple jQuery handlers with single event delegation
6. **Enable caching** - Add 1-hour cache for rendered page

---

## 2️⃣ MARKET GUIDE PAGE

**Current Load Time:** 4+ seconds | **Target:** <2 seconds

### Key Issues:
1. Drupal View rendering blocks entire page (1.5-2.5 seconds)
2. Duplicate paragraph entity loading (looping twice over same data)
3. Render-blocking JavaScript files
4. Multiple path conditionals in template (9 different URL checks)
5. No caching configured

### Steps to Fix:
1. **Use lazy builder for Drupal View** - Defer view rendering so it doesn't block initial page load
2. **Consolidate duplicate loops** - Loop through paragraph entities only once, store data for reuse
3. **Add defer attributes to JavaScript** - Prevent JavaScript from blocking page rendering
4. **Move path conditionals to preprocessing** - Move all URL checks from template to PHP preprocessing
5. **Preload critical images** - Add preload hints for above-fold images
6. **Enable render caching** - Add cache tags and 1-hour max-age

---

## 3️⃣ NEWS PAGE

**Current Load Time:** 4-5 seconds | **Target:** <2 seconds

### Key Issues:
1. N+1 query problem - Loading author data individually for each article (trending + related)
2. Duplicate rendering of related articles (desktop + mobile)
3. Inefficient string operations for social media URLs
4. No caching implementation

### Steps to Fix:
1. **Fix N+1 queries with eager loading** - Load all author terms in single query instead of one-by-one
2. **Preload related article author data** - Load all author entities upfront before template rendering
3. **Eliminate duplicate rendering** - Render related articles once, use CSS for desktop/mobile layout
4. **Move social URL building to PHP** - Generate all social media URLs in preprocessing, not template
5. **Flatten author data structure** - Avoid nested entity access that triggers additional queries
6. **Enable render caching** - Add cache metadata with proper tags and contexts

---

## 4️⃣ STOCK PAGE

**Current Load Time:** 6-10 seconds | **Target:** <2 seconds

### Key Issues:
1. **Sequential API calls** - 13 server-side APIs called one after another (5-6 seconds total)
2. **No parallelization** - APIs that don't depend on each other still wait in queue
3. **Uncacheable MarketSmith APIs** - Some critical APIs can't be cached (500-800ms each)
4. **Multiple client-side lazy loads** - 10+ additional AJAX calls after page render

### Steps to Fix:

#### Server-Side Optimizations:
1. **Call independent APIs in parallel** - Group APIs that can run simultaneously:
   - **Group 1 (parallel):** Overview, Fundamental, Technical Analysis, Tech Trend, Rapid Results, Corporate Actions (currently 2-3s sequential → 400-500ms parallel)
   - **Group 2 (after getting co_code):** Quote Details, Investment Rating, Ownership/Management, Financial Data (currently 2-3s sequential → 700-800ms parallel)

2. **Extend cache duration** - Increase 24-hour cache where appropriate (non-real-time data)

3. **Enable render caching** - Cache full page for 5-10 minutes during market hours, 1 hour after hours

4. **Optimize database queries** - Add indexes on frequently queried fields

#### Client-Side Optimizations:
5. **Defer non-critical AJAX calls** - Don't load polls, similar stocks immediately
6. **Batch related requests** - Combine shareholding pattern API calls (names + data + chart)
7. **Lazy load below-fold content** - Load forecast and financial tables only when user scrolls to them
8. **Use intersection observer** - Trigger AJAX only when section becomes visible

---

## PRIORITY MATRIX

### 🔴 Critical (Do First - Highest Impact)

**All Pages:**
- Enable render caching (saves 3-4 seconds)
- Add defer to JavaScript files (saves 0.5-1 second)

**Stock Page:**
- Parallelize independent API calls (saves 2-3 seconds)

**News Page:**
- Fix N+1 queries with eager loading (saves 1-2 seconds)

**IPO Calendar:**
- Convert inline JS to JSON (saves 1-2 seconds)

---

### 🟡 High Priority (Do Second - Good ROI)

**IPO Calendar:**
- Upgrade FullCalendar library (saves 1-2 seconds)

**Market Guide:**
- Lazy load Drupal View (saves 1.5-2.5 seconds)
- Consolidate duplicate loops (saves 0.5-1 second)

**Stock Page:**
- Extend cache durations (saves 1-2 seconds on repeat visits)

---

### 🟢 Medium Priority (Do Third - Nice to Have)

**All Pages:**
- Move template logic to preprocessing (saves 0.2-0.5 seconds)
- Optimize event handlers (saves 0.2-0.5 seconds)

**News Page:**
- Eliminate duplicate rendering (saves 0.2-0.4 seconds)
- Move social URLs to PHP (saves 0.3-0.5 seconds)

**Stock Page:**
- Lazy load below-fold content (improves perceived performance)
- Batch related AJAX requests (saves 0.3-0.5 seconds)

---

## EXPECTED RESULTS BY PAGE

| Page | Current | After Critical Fixes | After All Fixes | Target Met? |
|------|---------|---------------------|-----------------|-------------|
| IPO Calendar | 4-5s | 2-2.5s | <1.5s | ✅ Yes |
| Market Guide | 4-5s | 2-2.5s | <1.5s | ✅ Yes |
| News | 4-5s | 1-1.5s | <1s | ✅ Yes |
| Stock | 6-10s | 2-3.5s | 2-2.5s | ✅ Yes |

---

## QUICK WIN CHECKLIST

Start with these for maximum impact in minimum time:

- [ ] Fix News page N+1 queries (eager load authors)
- [ ] Convert IPO Calendar inline JS to JSON
- [ ] Parallelize Stock page API calls (Group 1: Overview, Fundamental, Technical, etc.)
- [ ] Lazy load Market Guide Drupal View
- [ ] Consolidate Market Guide duplicate paragraph loops

**Expected improvement from quick wins alone: 60-70% faster**
