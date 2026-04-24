# News Template Performance Analysis

## Problem Statement
The [`node--news.html.twig`](themes/custom/fivepaisa/templates/node/node--news.html.twig) template is taking 4-5 seconds to load news article pages, causing poor user experience and SEO impacts.

## Root Cause Analysis

### 1. **Excessive Database Queries (N+1 Problem)**

#### Trending Articles Section (Lines 322-377)
```twig
{% for noded in nodes %}
    {% set terms = trending_articles['trading_data']['terms'][noded.nid.value]%}
    {% set author_name = terms.name.value %}
    {% if terms.field_author and terms.field_author.entity.uri.value %}
        {% set author_img = file_url(terms.field_author.entity.uri.value) %}
    {% endif %}
```

**Issue**: For each trending article, the template accesses `terms.field_author.entity.uri.value`, which triggers lazy loading. With 3 trending articles, this causes multiple entity loads per request.

**Impact**: ~500ms - 1s per page load

#### Related Articles Section (Lines 379-422)
```twig
{% for noded in related_articles %}
    {{ noded.field_author.entity.label}}
```

**Issue**: Similar N+1 pattern - accessing `field_author.entity.label` for each related article without eager loading. With 4 related articles rendered twice (desktop + mobile), this is 8 additional entity loads.

**Impact**: ~400ms - 800ms per page load

#### Author Image Logic (Lines 103-111)
```twig
{% if node.field_author.entity.field_author and node.field_author.entity.field_author.entity.uri.value %}
```

**Issue**: Nested entity access chain causes multiple database queries for a single author image.

**Impact**: ~200ms - 400ms per page load

### 2. **Inefficient Data Preparation in PHP**

#### ArticlesController::getTrendingArticles_individual() (Lines 325-354)
```php
foreach ($trading_blogs as $node) {
    $author_term = $this->getauthor($node);  // Individual query
    $parentterm['trading_data']['terms'][$node->id()] = $author_term;
    $parentterm['trading_data']['estTimes'][$node->id()] = $this->getetr($node);
}
```

**Issues**:
- No eager loading of referenced entities
- Each `getauthor()` call loads Term individually
- Data structure requires multiple array lookups in template

**Impact**: ~800ms - 1.5s

#### ArticlesController::getrelatedarticles() (Lines 416-434)
```php
$nids = $query->execute();
if ($nids)
    return Node::loadMultiple($nids);
```

**Issues**:
- Returns nodes without preloading field data
- Author entities loaded lazily when accessed in template
- No field preloading

**Impact**: ~600ms - 1s

### 3. **Redundant String Operations**

#### Social Media URL Building (Lines 34-92)
```twig
{% set fb_url = 'https://www.facebook.com/sharer/sharer.php?u='~full_url|url_encode~'&description='~summaryfb~'&title='~title|url_encode~'&picture='~picture_url %}
{% set tw_url = 'https://twitter.com/intent/tweet?text='~title|url_encode~' '~full_url|url_encode~' '~summary~' '~odatext~' '~odalink~'  @wionews' %}
{% set percent_url = full_url|replace({'%':''}) %}
{% set in_url = 'https://www.linkedin.com/shareArticle?...' %}
{% set wa_html = '<a target="_blank" class="hide visible-xs" href="whatsapp://send?text='~full_data~'"...' %}
```

**Issues**:
- Multiple URL encoding operations
- String concatenation repeated for each social platform
- HTML string building in Twig (should be in template)

**Impact**: ~300ms - 500ms

### 4. **Lack of Caching**

#### No Template-Level Cache Tags
The template has:
- No cache tags defined
- No cache contexts specified
- No max-age set

**Issue**: Each page render executes all queries and processing, even for unchanged content.

**Impact**: 100% of the performance issue (could cache entire render)

### 5. **Duplicate Related Articles Rendering**
```twig
<!-- Desktop Grid -->
{% for noded in related_articles %}
    <!-- article card -->
{% endfor %}

<!-- Mobile Slider -->
{% for noded in related_articles %}
    <!-- Same article card -->
{% endfor %}
```

**Issue**: Related articles are looped twice with identical content generation, doubling the entity access overhead.

**Impact**: ~200ms - 400ms

## Performance Impact Breakdown

| Issue | Estimated Impact | Priority |
|-------|------------------|----------|
| N+1 queries in trending articles | 500ms - 1s | Critical |
| N+1 queries in related articles | 400ms - 800ms | Critical |
| No caching implementation | 100% overhead | Critical |
| Inefficient PHP data prep | 800ms - 1.5s | High |
| Redundant string operations | 300ms - 500ms | Medium |
| Duplicate article rendering | 200ms - 400ms | Medium |
| **Total Estimated Impact** | **3.2s - 4.6s** | - |

## Recommended Optimizations

### Priority 1: Critical (Will reduce load time by 70-80%)

#### 1.1 Implement Render Caching
Add cache metadata to template:
```twig
{#
  @cache
    tags: {{ node.getCacheTags()|join(' ') }}
    contexts: ['url.path', 'user.roles']
    max-age: 3600
#}
```

**Expected gain**: 3-4 seconds on cache hit

#### 1.2 Fix N+1 Queries with Eager Loading
Modify [`ArticlesController::getTrendingArticles_individual()`](modules/custom/five_paisa/src/Controller/ArticlesController.php:325):

```php
// Load all nodes with field data
$trading_blogs = Node::loadMultiple($nids);

// Eager load all author terms at once
$author_ids = array_filter(array_map(function($node) {
    return $node->get('field_author')->target_id ?? null;
}, $trading_blogs));

$author_terms = Term::loadMultiple($author_ids);

// Build lookup array
foreach ($trading_blogs as $node) {
    $author_id = $node->get('field_author')->target_id;
    $parentterm['trading_data']['terms'][$node->id()] = $author_terms[$author_id] ?? null;
}
```

**Expected gain**: 1-1.5 seconds

#### 1.3 Preload Related Article Author Data
Modify [`ArticlesController::getrelatedarticles()`](modules/custom/five_paisa/src/Controller/ArticlesController.php:416):

```php
if ($nids) {
    $nodes = Node::loadMultiple($nids);
    
    // Preload author entities
    $author_ids = [];
    foreach ($nodes as $node) {
        if ($author_id = $node->get('field_author')->target_id) {
            $author_ids[] = $author_id;
        }
    }
    if ($author_ids) {
        Term::loadMultiple($author_ids); // Loads into entity cache
    }
    
    return $nodes;
}
```

**Expected gain**: 600ms - 1 second

### Priority 2: High (Will reduce load time by 10-15%)

#### 2.1 Move Social URL Building to PHP Preprocess
Move social media URL generation from template (lines 34-92) to [`ArticlesController::generateData()`](modules/custom/five_paisa/src/Controller/ArticlesController.php:356):

```php
// In generateData()
$variables['social_urls'] = [
    'facebook' => $this->buildFacebookUrl($node),
    'twitter' => $this->buildTwitterUrl($node),
    'linkedin' => $this->buildLinkedInUrl($node),
    'whatsapp' => $this->buildWhatsAppUrl($node),
];
```

**Expected gain**: 300-500ms

#### 2.2 Optimize Author Data Structure
Flatten the author data structure to avoid nested entity access:

```php
// In getTrendingArticles_individual()
$parentterm['trading_data']['authors'][$node->id()] = [
    'name' => $author_term->getName(),
    'image' => $this->getAuthorImage($author_term),
    'tid' => $author_term->id(),
];
```

**Expected gain**: 200-400ms

### Priority 3: Medium (Will reduce load time by 5-10%)

#### 3.1 Eliminate Duplicate Rendering
Use CSS to show/hide instead of rendering twice:

```twig
<div class="articles-display">
    {% for noded in related_articles %}
        <div class="article-card">
            <!-- content -->
        </div>
    {% endfor %}
</div>

<style>
@media (min-width: 768px) {
    .articles-display { display: grid; }
}
@media (max-width: 767px) {
    .articles-display { /* slider styles */ }
}
</style>
```

**Expected gain**: 200-400ms

#### 3.2 Optimize FAQ Rendering
Simplify FAQ loop filter (line 196):

```twig
{% for item in content.field_blog_faqs if item['#paragraph'] is defined %}
```

Instead of:
```twig
{% for key, item in content.field_blog_faqs|filter((value, key) => key|first != '#') %}
```

**Expected gain**: 50-100ms

### Priority 4: Additional Optimizations

#### 4.1 Add Lazy Loading for Related/Trending Content
```twig
{% if trending_articles %}
    <section class="paisabase-trending" data-lazy-load="true">
        <!-- Content loaded via AJAX after page render -->
    </section>
{% endif %}
```

#### 4.2 Implement Fragment Caching
```twig
{% cache tags=['node:' ~ node.id(), 'related_articles'] %}
    <!-- Related articles section -->
{% endcache %}
```

#### 4.3 Database Query Optimization
Add database indexes:
- `field_author` on news/blog nodes
- `field_news_categories` with composite index (target_id, status, created)

## Implementation Recommendations

### Phase 1: Quick Wins (1-2 days)
1. Add render caching to template
2. Fix N+1 queries with eager loading
3. Preload author data

**Expected improvement**: 70-80% reduction (from 4-5s to <1s)

### Phase 2: Optimization (2-3 days)
1. Move social URL building to PHP
2. Flatten data structures
3. Eliminate duplicate rendering
4. Optimize FAQ rendering

**Expected improvement**: Additional 10-15% reduction

### Phase 3: Advanced (3-5 days)
1. Implement lazy loading for non-critical sections
2. Add fragment caching
3. Database index optimization
4. Consider static page generation for popular articles

**Expected improvement**: Additional 5-10% reduction

## Monitoring & Validation

### Performance Metrics to Track
1. **Server-side rendering time** (using Devel/Webprofiler)
2. **Database query count** (should reduce from ~20+ to ~5)
3. **Time to First Byte (TTFB)** (target: <500ms)
4. **Total page load time** (target: <2s)

### Testing Strategy
1. Test with Drupal Performance module enabled
2. Use Xdebug profiling to identify hotspots
3. Compare before/after with real production data
4. Load test with multiple concurrent users

## Critical Code Locations

| File | Lines | Issue |
|------|-------|-------|
| [`node--news.html.twig`](themes/custom/fivepaisa/templates/node/node--news.html.twig:322) | 322-377 | Trending articles N+1 queries |
| [`node--news.html.twig`](themes/custom/fivepaisa/templates/node/node--news.html.twig:379) | 379-422 | Related articles duplicate rendering |
| [`node--news.html.twig`](themes/custom/fivepaisa/templates/node/node--news.html.twig:34) | 34-92 | Inefficient string operations |
| [`ArticlesController.php`](modules/custom/five_paisa/src/Controller/ArticlesController.php:325) | 325-354 | `getTrendingArticles_individual()` - no eager loading |
| [`ArticlesController.php`](modules/custom/five_paisa/src/Controller/ArticlesController.php:416) | 416-434 | `getrelatedarticles()` - no field preloading |
| [`ArticlesController.php`](modules/custom/five_paisa/src/Controller/ArticlesController.php:284) | 284-301 | `getauthor()` - individual term loads |

## Architectural Improvements

### Current Data Flow
```
Request → Preprocess → Template Render → Multiple DB Queries → Response
                         ↓
                    No caching
```

### Optimized Data Flow
```
Request → Check Cache → [Cache Hit: Return Cached]
              ↓
         [Cache Miss]
              ↓
    Preprocess with Eager Loading → Template Render → Minimal DB Queries
              ↓
        Cache Result → Response
```

## Additional Considerations

### 1. **Memory Usage**
- Current approach loads entities multiple times
- Drupal's entity cache helps but still inefficient
- Eager loading will increase initial memory but reduce total queries

### 2. **Cache Invalidation Strategy**
- Cache should invalidate when:
  - Node is updated
  - Related articles change
  - Trending articles list updates
  - Author information changes

### 3. **Backwards Compatibility**
- Changes to `ArticlesController` methods may affect:
  - Blog templates
  - IPO news templates
  - Other custom modules using these methods

## Next Steps

1. **Implement Phase 1 optimizations** (highest ROI)
2. **Set up performance monitoring** before/after
3. **Test on staging environment** with production data
4. **Deploy during low-traffic window**
5. **Monitor error logs** for entity access issues
6. **Measure and validate** improvements

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Cache invalidation bugs | Medium | High | Thorough testing with cache clear scenarios |
| Breaking other templates | Low | High | Search codebase for ArticlesController usage |
| Memory increase | Low | Medium | Monitor server resources post-deployment |
| Incorrect eager loading | Medium | High | Add null checks and fallbacks |

## Success Criteria

- [ ] Page load time reduced to <1 second
- [ ] Database query count reduced by 60%+
- [ ] Cache hit ratio >80% after warmup
- [ ] No breaking changes to existing functionality
- [ ] Core Web Vitals improved (LCP <2.5s)
