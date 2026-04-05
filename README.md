Shopify Featured Products Assignment
Overview

This project demonstrates a Shopify collection page where:

100 sample products are present.
Exactly 15 products are tagged "featured" in random positions.
Infinite scroll loads 20 products per request.
All featured products always appear at the top of the collection.
Remaining products load after the featured ones.
Sorting and filtering continue to work normally.
Duplicate products are prevented.

Basic Setup
Imported 100 sample products, with exactly 15 products tagged "featured" at random positions.
Created a separate public GitHub repository named amplicomm-shopify-assignment.
Logged into the Shopify store via CLI and pulled the theme locally.
Uploaded the theme to GitHub, connected the Shopify store, and published the theme.
Applied changes locally, verified functionality, and pushed the final version to Git.

Functionality Explanation
main-collection.liquid
Increased pagination limit to 250 (Shopify's maximum limit).
Reason: Tagged products are in random positions. Filtering by tag in Liquid only works on products loaded on the page. If the limit is set too low (e.g., 20), some featured products may not appear.

Added data-featured attribute to <li>:

<li data-featured="{% if product.tags contains 'featured' %}true{% else %}false{% endif %}">
Reason: Categorizes featured and non-featured products for use in JavaScript arrays.

Note: Initially, I tried to categorize featured vs non-featured products entirely in Liquid using two loops. However, Shopify applies sorting and filtering separately for each loop, which breaks expected behavior. Therefore, the final solution handles separation client-side via JavaScript.

product-grid.liquid

Added id to <ul> element:

<ul id="product-list">
Reason: Used as a selector in JavaScript to fetch child product grid items.
util-product-grid-card-size.liquid

Decreased product grid size from 250px to 200px for medium view:

{% assign product_card_size = '200px' %}
Reason: Medium view originally shows 4 products per row. Reducing the size allows 5 products per row, ensuring all 15 featured products display properly in 3 rows.
results.list.js

Variable Initialization

let allProducts = [];
let renderedIndex = 0;
allProducts stores extra products not displayed initially.
renderedIndex tracks how many extra products have been appended.

Initialize products on page load

function initProducts() { ... }
Displays the first 20 products: 15 featured + 5 normal.

Get product list

const list = document.getElementById("product-list");
const items = Array.from(list.children);

Separate featured and normal products

let featured = [];
let normal = [];
items.forEach(item => {
  if (item.dataset.featured === "true") {
    featured.push(item);
  } else {
    normal.push(item);
  }
});

Limit number of products to show initially

featured = featured.slice(0, 15);
const firstNormal = normal.slice(0, 21 - featured.length);
allProducts = normal.slice(21 - featured.length);

Clear product list before rendering

list.innerHTML = "";

Append initial products

[...featured, ...firstNormal].forEach(el => list.appendChild(el));
renderedIndex = 0;

Load more products on scroll

function loadMoreProducts() {
  if (renderedIndex >= allProducts.length) return;
  const nextItems = allProducts.slice(renderedIndex, renderedIndex + 20);
  nextItems.forEach(el => list.appendChild(el));
  renderedIndex += 20;
}

Scroll event detection

window.addEventListener("scroll", function() {
  if (window.innerHeight + window.scrollY >= document.body.offsetHeight - 300) {
    loadMoreProducts();
  }
});

Run initialization after DOM loaded

document.addEventListener("DOMContentLoaded", initProducts);
Important Notes & Limitations
Shopify is client-side, so filtering by tag before fetching all products is not possible. Hence, we fetch 250 products per page.
Slight flicker occurs because normal products load first, then featured products are rearranged via JavaScript.
Attempting to separate featured products in Liquid using two loops breaks sorting and filtering because Shopify applies them separately per loop. Therefore, JavaScript separation ensures correct order and infinite scroll.
Sorting (Best Selling, Price Low → High, Price High → Low) works normally after rearranging featured products.
Filters temporarily override featured pinning, matching Shopify’s default behavior.
Solution scales for large collections by handling separation and infinite scroll client-side without performance issues.
Edge Case Handling
Edge Case	Expected Behavior
No featured products	Normal infinite scroll behavior applies; no featured pinning
Filters applied	Sorting and filtering temporarily override featured logic
Very large collection	Fetch up to 250 products; infinite scroll handles remaining products efficiently
Featured products in later pages	Already displayed featured products are not repeated
Attempted Code in Liquid (Not Implemented)

I tried categorizing featured and non-featured products entirely in Liquid using two loops:

{% for product in collection.products %}
  {% if product.tags contains 'featured' %}
    <!-- featured product markup -->
  {% endif %}
{% endfor %}

{% for product in collection.products %}
  {% unless product.tags contains 'featured' %}
    <!-- normal product markup -->
  {% endunless %}
{% endfor %}

Issue: Shopify applies sorting and filters separately for each loop, which breaks sorting/filtering consistency. Therefore, this approach was not implemented in the final solution.