# amplicomm-shopify-assignment

## Basic Setup

1. Imported 100 sample products, with exactly 15 products tagged as **"featured"** at random positions.
2. Created a separate public GitHub repository named **amplicomm-shopify-assignment**.
3. Logged into the Shopify store via CLI and pulled the theme to the local environment.
4. Uploaded the theme to the Git repository, connected the Shopify store with Git, and published the theme from the repository.
5. Applied changes in the local environment, verified functionality, and pushed the final version to Git.

---

## Functionality Explanation

### `main-collection.liquid`

1. **Increased pagination limit to 250 (Shopify's maximum limit)**
   **Reason:**
   Tagged products are randomly positioned, and Liquid filtering works only on the products loaded on the page. If the limit is too low (e.g., 20), some featured products may not be included in the loaded set.

2. **Added `data-featured` attribute to `<li>`**

```liquid
<li data-featured="{% if product.tags contains 'featured' %}true{% else %}false{% endif %}">
```

**Reason:**
This helps categorize featured and non-featured products, allowing JavaScript to create separate arrays for display logic.

---

### `product-grid.liquid`

3. **Added `id` attribute to `<ul>`**

```html
<ul id="product-list">
```

**Reason:**
Used as a selector in JavaScript to access and manipulate product grid items.

---

### `util-product-grid-card-size.liquid`

4. **Adjusted product grid size from 250px to 200px (medium view)**

```liquid
{% assign product_card_size = '200px' %}
```

**Reason:**
The default layout shows 4 products per row. Reducing the size allows 5 products per row, improving UI and ensuring all 15 featured products are displayed neatly in 3 rows.

---

### `results.list.js`

1. **Variable Initialization**

```javascript
let allProducts = [];
let renderedIndex = 0;
```

* `allProducts`: Stores products not shown initially
* `renderedIndex`: Tracks how many products have been rendered

2. **Initialize Products on Page Load**

```javascript
function initProducts() { ... }
```

Displays the first batch of products.

3. **Get Product List**

```javascript
const list = document.getElementById("product-list");
const items = Array.from(list.children);
```

* `list`: Container element
* `items`: Array of product elements

4. **Separate Featured and Normal Products**

```javascript
let featured = [];
let normal = [];

items.forEach(item => {
  if (item.dataset.featured === "true") {
    featured.push(item);
  } else {
    normal.push(item);
  }
});
```

5. **Limit Initial Products**

```javascript
featured = featured.slice(0, 15);
const firstNormal = normal.slice(0, 21 - featured.length);
allProducts = normal.slice(21 - featured.length);
```

* Shows up to 15 featured products
* Fills remaining slots with normal products (total ~20)
* Stores remaining products for lazy loading

6. **Clear Product List**

```javascript
list.innerHTML = "";
```

7. **Append Initial Products**

```javascript
[...featured, ...firstNormal].forEach(el => list.appendChild(el));
renderedIndex = 0;
```

8. **Load More Products (Infinite Scroll)**

```javascript
function loadMoreProducts() {
  if (renderedIndex >= allProducts.length) return;

  const list = document.getElementById("product-list");
  const nextItems = allProducts.slice(renderedIndex, renderedIndex + 20);

  nextItems.forEach(el => list.appendChild(el));
  renderedIndex += 20;
}
```

9. **Scroll Detection**

```javascript
window.addEventListener("scroll", function() {
  if (window.innerHeight + window.scrollY >= document.body.offsetHeight - 300) {
    loadMoreProducts();
  }
});
```

10. **Run on Page Load**

```javascript
document.addEventListener("DOMContentLoaded", initProducts);
```

---

## Important Notes

1. Shopify operates on a client-side rendering approach for this case. Filtering tagged products before loading is not possible, which is why 250 products are fetched instead of 20.

2. A slight flickering issue occurs on page load:

   * Products initially render in default order via Liquid
   * JavaScript then rearranges them to prioritize featured products

3. An alternative Liquid-based approach (using two loops) was considered, but due to Shopify limitations:

   * Sorting and filtering apply separately to each loop
   * This prevents consistent global ordering

---

## Code Example

```liquid
{% paginate collection.products by 250 %}
  {% capture children %}

    {% for product in collection.products %}
      {% if product.tags contains 'featured' %}
      <li
        id="{{ section.id }}-{{ product.id }}"
        class="product-grid__item product-grid__item--{{ forloop.index0 }}"
        data-page="{{ paginate.current_page }}"
        data-product-id="{{ product.id }}"
        data-featured="true"
        ref="cards[]"
      >
        {% content_for 'block', type: '_product-card', id: 'product-card', closest.product: product %}
      </li>
      {% endif %}
    {% endfor %}

    {% for product in collection.products %}
      {% unless product.tags contains 'featured' %}
      <li
        id="{{ section.id }}-{{ product.id }}"
        class="product-grid__item product-grid__item--{{ forloop.index0 }}"
        data-page="{{ paginate.current_page }}"
        data-product-id="{{ product.id }}"
        data-featured="false"
        ref="cards[]"
      >
        {% content_for 'block', type: '_product-card', id: 'product-card', closest.product: product %}
      </li>
      {% endunless %}
    {% endfor %}

  {% endcapture %}

  {% render 'product-grid',
    section: section,
    children: children,
    products: collection.products,
    paginate: paginate,
    enable_infinite_scroll: section.settings.enable_infinite_scroll
  %}
{% endpaginate %}
```
