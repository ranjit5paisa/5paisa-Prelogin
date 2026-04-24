# Market Guide Page Performance Analysis

## Overview
The [`node--market-guide.html.twig`](../themes/custom/fivepaisa/templates/node/node--market-guide.html.twig) page is taking **more than 4 seconds** to load. This document outlines the identified performance bottlenecks and provides actionable optimization recommendations.

---

## Critical Performance Bottlenecks

### 1. **Drupal View Rendering (Line 230)**

**Issue:**
```twig
{{ drupal_view('learn_about', 'block_1', tid) }}
```

This synchronously renders a Drupal View which:
- Executes complex database queries without caching
- Loads multiple nodes/entities eagerly
- Has no lazy loading or placeholders
- Blocks the entire page rendering until complete

**Impact:** **~1.5-2.5 seconds**

**Solution:**
Use lazy builders to defer view rendering:
```php
// In fivepaisa_preprocess_node()
if ($node_type == 'market_guide') {
  $tid = $variables['node']->get('field_learn_about_type')->target_id;
  $variables['learn_about_section'] = [
    '#lazy_builder' => [
      'Drupal\views\Element\View::lazyBuilder',
      ['learn_about', 'block_1', $tid]
    ],
    '#create_placeholder' => TRUE,
  ];
}
```

Then in the template:
```twig
{{ learn_about_section }}
```

---

### 2. **Duplicate Paragraph Entity Loading (Lines 136-173)**

**Issue:**
The template loops over `node.field_marke` **twice**:

**First loop (lines 136-146):** Builds table of contents
```twig
{% for courses in node.field_marke %}
  {% if courses.entity.field_text.value is not empty %}
    <li><a href="#Stock{{ loop.index }}">{{ courses.entity.field_text.value }}</a></li>
  {% endif %}
{% endfor %}
```

**Second loop (lines 150-173):** Renders content
```twig
{% for courses in node.field_marke %}
  <div class="bodybnwrapper market-para">
    <h2>{{ courses.entity.field_text.value }}</h2>
    {{ courses.entity.field_description.value | raw }}
  </div>
{% endfor %}
```

Each iteration loads paragraph entities from the database. Looping twice doubles the query load.

**Impact:** **~0.5-1 second**

**Solution:**
Consolidate into a single loop:
```twig
{% if node.field_marke.0 %}
  {# Store items for later use #}
  {% set market_items = [] %}
  {% for courses in node.field_marke %}
    {% if courses.entity.field_text.value is not empty %}
      {% set market_items = market_items|merge([{
        'text': courses.entity.field_text.value,
        'description': courses.entity.field_description.value,
        'index': loop.index
      }]) %}
    {% endif %}
  {% endfor %}

  {# Table of Contents #}
  <ul class="list">
    {% for item in market_items %}
      <li><a href="#Stock{{ item.index }}">{{ item.text }}</a></li>
    {% endfor %}
  </ul>

  {# Content #}
  {% for item in market_items %}
    <div class="bodybnwrapper market-para {% if item.index > 1 %}hide-content{% endif %}" id="Stock{{ item.index }}">
      <h2>{{ item.text }}</h2>
      {{ item.description | raw }}
    </div>
  {% endfor %}
{% endif %}
```

---

### 3. **Render-Blocking Resources**

**Issue:**
The [`market-guide-lib`](../themes/custom/fivepaisa/fivepaisa.libraries.yml) library (lines 1071-1079) loads:
```yaml
market-guide-lib:
  version: 4.0
  css:
    theme:
      css/market-guide-stylesheet.css: {}
  js:
    js/market-guide.js: {}
  dependencies:
    - fivepaisa/faq-library
```

- No async/defer attributes on JavaScript
- Blocking CSS in `<head>`
- Additional blocking JS from `faq-library` dependency

**Impact:** **~0.3-0.5 seconds**

**Solution:**
Update library definition:
```yaml
market-guide-lib:
  version: 5.0  # Increment version
  css:
    theme:
      css/market-guide-stylesheet.css: {}
  js:
    js/market-guide.js: { attributes: { defer: true } }
  dependencies:
    - fivepaisa/faq-library
```

Also update `faq-library` (line 19-22):
```yaml
faq-library:
  version: 4.0  # Increment version
  js:
    js/faq.min.js: { attributes: { defer: true } }
```

---

### 4. **Multiple Path Conditionals in Template (Lines 86-104)**

**Issue:**
9 different URL string comparisons execute on every page load:
```twig
{% if ('/stock-market-guide/mutual-funds/' in path_url) %}
  <img loading="lazy" src="https://storage.googleapis.com/5paisa-prod-storage/pages/ipo/MF.gif">
  <h3>Unlock Growth with Direct Mutual Funds!</h3>
{% elseif ('/stock-market-guide/ipo' in path_url) %}
  <img loading="lazy" src="https://storage.googleapis.com/5paisa-prod-storage/pages/ipo/IPO.gif">
  <h3>IPO Investing made simple!</h3>
{% elseif (('/stock-market-guide/derivatives-trading-basics' in path_url) or ('/stock-market-guide/derivatives-trading' in path_url)) %}
  <img loading="lazy" src="https://storage.googleapis.com/5paisa-prod-storage/pages/market-guide/stock-market-guide-derivatives.gif">
  <h3>Looking to explore Derivatives trading?</h3>
{% else %}
  {# More conditionals... #}
{% endif %}
```

**Impact:** **~0.1-0.2 seconds**

**Solution:**
Move logic to preprocessing in [`fivepaisa.theme`](../themes/custom/fivepaisa/fivepaisa.theme):

```php
if ($node_type == 'market_guide') {
  $path_url = $variables['path_url'] ?? \Drupal::service('path.current')->getPath();
  
  // Determine form section
  $form_config = [
    'image' => 'https://storage.googleapis.com/5paisa-prod-storage/pages/pdf/V2_11-Jul-GIFS.gif',
    'heading' => 'Want to start your Investment Journey?',
  ];
  
  if (strpos($path_url, '/stock-market-guide/mutual-funds/') !== false) {
    $form_config = [
      'image' => 'https://storage.googleapis.com/5paisa-prod-storage/pages/ipo/MF.gif',
      'heading' => 'Unlock Growth with Direct Mutual Funds!',
    ];
  } elseif (strpos($path_url, '/stock-market-guide/ipo') !== false) {
    $form_config = [
      'image' => 'https://storage.googleapis.com/5paisa-prod-storage/pages/ipo/IPO.gif',
      'heading' => 'IPO Investing made simple!',
    ];
  } elseif (strpos($path_url, '/stock-market-guide/derivatives-trading') !== false) {
    $form_config = [
      'image' => 'https://storage.googleapis.com/5paisa-prod-storage/pages/market-guide/stock-market-guide-derivatives.gif',
      'heading' => 'Looking to explore Derivatives trading?',
    ];
  } elseif (strpos($path_url, '/stock-market-guide/online-trading/') !== false) {
    $form_config['heading'] = 'Ready to start your Trading Journey?';
  } elseif (strpos($path_url, '/stock-market-guide/commodity-trading-basics/') !== false) {
    $form_config['heading'] = 'Looking to explore Commodity Trading?';
  }
  
  $variables['form_config'] = $form_config;
}
```

Then in template, simplify to:
```twig
<img loading="lazy" src="{{ form_config.image }}" alt="demat" title="demat" class="hidden-xs">
<h3>{{ form_config.heading }}</h3>
```

---

### 5. **Uncached External Resources**

**Issue:**
Multiple external images loaded on each request:
- Line 70: Google Preferred Source badge
- Lines 87-96: Conditional GIF images from Google Cloud Storage

**Impact:** **~0.2-0.4 seconds** (network latency)

**Solution:**
1. **Preload critical images** in preprocessing:
```php
$variables['#attached']['html_head_link'][] = [
  [
    'rel' => 'preload',
    'as' => 'image',
    'href' => 'https://storage.googleapis.com/5paisa-prod-storage/pages/images/preferred-source-google.webp',
  ],
];
```

2. **Use CDN caching headers** - ensure Google Cloud Storage buckets have proper cache-control headers

3. **Consider hosting critical images locally** if external CDN is slow

---

### 6. **Missing Cache Configuration**

**Issue:**
No explicit cache tags, contexts, or max-age directives in the template or preprocessing.

**Impact:** Pages regenerated on every request

**Solution:**
Add cache metadata in preprocessing:

```php
if ($node_type == 'market_guide') {
  // Add cache tags
  $variables['#cache']['tags'][] = 'node:' . $nid;
  $variables['#cache']['tags'][] = 'node_type:market_guide';
  
  // Add cache contexts
  $variables['#cache']['contexts'][] = 'url.path';
  $variables['#cache']['contexts'][] = 'user.roles';
  
  // Set cache max-age (1 hour = 3600 seconds)
  $variables['#cache']['max-age'] = 3600;
}
```

---

## Implementation Roadmap

### **Phase 1: Quick Wins (1-2 hours)**
High impact, low effort optimizations:

- [ ] Add defer attributes to JavaScript libraries
- [ ] Move path conditionals to preprocessing
- [ ] Add basic cache metadata
- [ ] Enable Drupal page cache if not already enabled

**Expected improvement:** -1.5 to -2 seconds

---

### **Phase 2: Template Optimization (2-4 hours)**
Moderate effort, high impact:

- [ ] Consolidate duplicate paragraph loops
- [ ] Convert Drupal view to lazy builder
- [ ] Preload critical external resources
- [ ] Add lazy loading to all images

**Expected improvement:** -1.5 to -2 seconds

---

### **Phase 3: Infrastructure (4-8 hours)**
Higher effort, sustained benefits:

- [ ] Enable BigPipe module for progressive rendering
- [ ] Configure Redis/Memcache for render cache
- [ ] Add database indexes on field_learn_about_type
- [ ] Implement CDN for static assets
- [ ] Enable PHP OpCode cache

**Expected improvement:** -0.5 to -1 second

---

### **Phase 4: Advanced Optimization (8+ hours)**
Long-term performance improvements:

- [ ] Implement lazy loading for below-fold content
- [ ] Split critical CSS inline, defer non-critical
- [ ] Create separate mobile template variant
- [ ] Implement service worker for offline caching
- [ ] Add HTTP/2 server push for critical resources

**Expected improvement:** -0.5 to -1 second

---

## Expected Performance Gains Summary

| Optimization | Time Saved | Effort |
|--------------|------------|--------|
| View Lazy Loading | 1.5-2.5s | Medium |
| Paragraph Loop Consolidation | 0.5-1.0s | Low |
| JS Async Loading | 0.3-0.5s | Low |
| Path Logic Preprocessing | 0.1-0.2s | Low |
| Cache Configuration | 0.3-0.5s | Low |
| External Resource Optimization | 0.2-0.4s | Low |
| **TOTAL** | **2.9-4.6s** | - |

**Target:** Reduce page load from **4+ seconds** to **<2 seconds**
