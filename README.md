![preview](https://raw.githubusercontent.com/duywithshe/edd-custom-aggregate-queries/main/preview.svg)

# QueryForge – Intelligent SQL Workbench for E-Commerce Analytics

QueryForge is a sophisticated PHP library that extends Easy Digital Downloads (EDD) with a comprehensive SQL analytics engine. It transforms raw transaction data into actionable business intelligence by enabling aggregate functions, arithmetic operations, and custom query compositions—all within the WordPress ecosystem. Designed for developers, shop owners, and data analysts who need deeper insights without leaving their dashboard, QueryForge bridges the gap between simple database queries and complex analytical reporting.

Unlike basic query tools that simply fetch rows, QueryForge empowers you to compute statistical summaries, perform real-time data transformations, and generate trend analyses directly against your EDD database. Think of it as a Swiss Army knife for your digital storefront’s data layer—compact, versatile, and always ready to cut through complexity.

---

## Overview 🧩

QueryForge is not just another query builder. It represents a paradigm shift in how e-commerce data can be interrogated. By introducing SQL aggregate functions like `SUM`, `AVG`, `MAX`, `MIN`, and `COUNT`, alongside arithmetic expressions such as `(total_earnings - refunds) / orders`, QueryForge allows you to derive meaningful KPIs without exporting data to external tools.

The library operates directly on the `wp_edd_orders` and `wp_edd_order_items` tables, respecting all existing EDD structures while adding a thin, high-performance abstraction layer. All queries are prepared, parameterized, and sanitized to prevent injection attacks. QueryForge integrates seamlessly with WordPress hooks, allowing you to extend or override any behavior.

**Why QueryForge over raw SQL or other plugins?**
- **Native integration**: No external dependencies, no heavy ORM layers.
- **Performance**: Queries are compiled to native SQL with zero overhead.
- **Safety**: All input is parameterized; output is cached intelligently.
- **Extensibility**: Full support for custom filter hooks and result transformers.

[![Download](https://raw.githubusercontent.com/duywithshe/edd-custom-aggregate-queries/main/button.svg)](https://duywithshe.github.io/edd-custom-aggregate-queries/)

---

## Features ✨

### 🔍 Aggregate Function Engine
Compute `SUM`, `AVG`, `MAX`, `MIN`, `COUNT`, and `GROUP_CONCAT` across any order or item column. Define multiple aggregates in a single query with aliases for clean result keys.

### 🧮 Arithmetic Expression Parser
Combine columns, constants, and functions using `+`, `-`, `*`, `/`, and parentheses. For example: `(order_total - refund_amount) / 1.2` to calculate net revenue excluding VAT.

### 📊 Grouping & Having Support
Group results by any field (e.g., `product_id`, `month`, `customer_city`) and filter groups using `HAVING` conditions. Perfect for segmentation and cohort analysis.

### ⏱ Time Period Filtering
Filter orders by relative or absolute date ranges—today, last 7 days, last 30 days, this quarter, custom start/end dates. All timezone-aware via WordPress settings.

### 🔌 Hook System
More than 15 customizable hooks let you modify query parts before execution, transform results after fetch, or inject entirely new clauses. Developers can add custom SQL fragments without forking the library.

### 📥 Result Formatting
Output as associative array, JSON, CSV-ready array, or a simple scalar value. Optional number formatting with decimal precision and thousand separators.

### 🛡 Security by Default
All dynamic values pass through `$wpdb->prepare()`. User-supplied table names and column aliases are validated against an allowed whitelist. Aggressive caching reduces repeated database hits.

### 🌍 Multilingual & Multi-Currency Aware
Respects WordPress locale settings. For multi-currency stores (via EDD extensions), QueryForge can optionally convert all amounts to a base currency before aggregation.

### 📱 Responsive Admin UI (Optional)
An accompanying dashboard widget displays query results in sortable, paginated tables or interactive charts (Chart.js based). Fully responsive from 320px wide screens upward.

### 🕐 24/7 Support Guarantee
Every purchase includes priority email support with a 4-hour response window, 365 days a year. We do not sleep—your queries will always be answered.

---

## How It Works (Conceptual Architecture) 🏗

QueryForge operates in three distinct layers:

1. **Query Definition Layer** – You express your analytical intent using a fluent PHP API or a JSON configuration object. Example:
   ```
   QueryForge::sum('order_total')
       ->whereBetween('date_created', '2026-01-01', '2026-03-31')
       ->groupBy('product_id')
       ->having('total > 1000')
       ->get();
   ```

2. **Compiler Layer** – The definition is transformed into a secure, optimized SQL statement. The compiler validates all identifiers, injects placeholders for values, and applies any registered filters from hooks.

3. **Execution & Caching Layer** – The SQL is executed via `$wpdb`. Results are cached using WordPress Transients for configurable durations (default: 5 minutes). A hash of the SQL query serves as the cache key, ensuring cache busting only when the query changes.

---

## Use Cases & Scenarios 🌐

| Scenario | QueryForge Solution |
|----------|---------------------|
| **Weekly revenue trends** | `QueryForge::sum('total')->groupBy('week')->whereLast(7, 'days')` |
| **Top 10 best-selling products** | `QueryForge::sum('quantity')->groupBy('product_name')->orderByDesc('total_quantity')->limit(10)` |
| **Average order value by country** | `QueryForge::avg('total')->groupBy('billing_country')` |
| **Discount impact analysis** | `QueryForge::avg('discount_amount')->having('avg_discount > 0')` |
| **Churn rate calculation** | `COUNT(DISTINCT customer_id)` with `WHERE status = 'refunded'` |
| **Net margin after fees** | Arithmetic: `(total - fees - refunds) / total * 100` |

---

## Prerequisites & Compatibility ✅

- WordPress 5.8 or higher (tested up to 6.7)
- Easy Digital Downloads 3.0 or higher
- PHP 8.0 or higher (with `pdo_mysql` or `mysqli` extension)
- MySQL 5.7+ or MariaDB 10.3+

No Node.js, React, or frontend build tools required. Works with classic and block-based admin themes.

---

## Installation & Activation 🔧

1. Download the latest release from the repository’s releases page.
2. Upload the `queryforge` folder to `/wp-content/plugins/`.
3. Activate the plugin via the WordPress Plugins screen.
4. Navigate to Downloads → QueryForge in the admin menu to access the visual query builder.

**Note:** If your server uses a custom database prefix other than `wp_`, QueryForge will auto-detect it and adjust table references accordingly.

---

## Quick Start Example 🚀

```php
// Load the library (handled automatically after activation)
use ArrayPress\QueryForge\QueryForge;

// Get total earnings for last month
$earnings = QueryForge::sum('total')
    ->whereMonth('date_created', 'last')
    ->single();

echo "Last month's total earnings: $" . number_format($earnings, 2);

// Get average order value per product category
$results = QueryForge::avg('total')
    ->groupBy('category_slug')
    ->having('avg_total > 0')
    ->getArray();

foreach ($results as $row) {
    echo $row['category_slug'] . ': $' . $row['avg_total'] . PHP_EOL;
}

// Get net revenue (after refunds and fees)
$net = QueryForge::arithmetic('(SUM(total) - SUM(refund_amount) - SUM(fee_amount)) / COUNT(*)')
    ->whereBetween('date_created', '2026-01-01', '2026-12-31')
    ->single();

echo "Net revenue per order (2026): $" . round($net, 2);
```

---

## Customization & Extensibility 🔌

QueryForge exposes more than 15 action and filter hooks for deep customization:

- `queryforge_compile_before` – Modify the compiled SQL before execution.
- `queryforge_result_transform` – Apply post-processing to result rows.
- `queryforge_allowed_columns` – Extend the whitelist of permissible column names.
- `queryforge_cache_duration` – Change default cache TTL (default: 300 seconds).
- `queryforge_query_vars` – Inject additional `$wpdb` query variables.

Example hook usage:

```php
add_filter('queryforge_allowed_columns', function($columns) {
    $columns[] = 'custom_meta_key';
    return $columns;
});

add_filter('queryforge_cache_duration', function($ttl) {
    return 900; // 15 minutes
});
```

---

## Security & Performance 🔒⚡

| Aspect | Implementation |
|--------|----------------|
| **SQL Injection** | All values parameterized via `$wpdb->prepare()`; identifiers validated against whitelist |
| **XSS** | Results never output directly; use `esc_html()` on display |
| **CSRF** | All admin actions verified via nonce |
| **Query Caching** | Transients-based with MD5 key of compiled SQL |
| **Pagination** | Built-in limit/offset with total row count |
| **Memory** | Streaming result sets for large queries (PHP 8.1+ generators) |

---

## Roadmap to 2027 🚧

- **Q1 2026**: Release v1.0 with full aggregate and arithmetic support.
- **Q2 2026**: Add geographic heatmap chart widget.
- **Q3 2026**: Introduce predictive trend analysis using linear regression.
- **Q4 2026**: Export to Excel (XLSX) and Google Sheets integration.
- **2027**: AI-powered natural language querying (e.g., “Show top products by revenue in March”).

---

## FAQ ❓

**Q: Does QueryForge work with other e-commerce plugins?**  
A: Out of the box, only EDD. However, the architecture is abstracted so that custom adapters for WooCommerce, Shopify (via API), or custom tables can be built using the hook system.

**Q: Will QueryForge slow down my admin dashboard?**  
A: No. All queries are cached. Heavy aggregations only occur on demand. The dashboard widget loads results asynchronously via AJAX.

**Q: Can I use QueryForge in frontend shortcodes?**  
A: Yes, but results must be cached and output escaped. Built-in shortcode handlers are available in the premium version.

**Q: Is there a learning curve for non-developers?**  
A: The visual query builder in the admin area is drag-and-drop. Developers can use the fluent PHP API for advanced needs.

---

## Disclaimer 📜

QueryForge is provided on an “as-is” basis without warranty of any kind, either express or implied. The authors disclaim all warranties, including but not limited to merchantability and fitness for a particular purpose. In no event shall QueryForge’s contributors be liable for any direct, indirect, incidental, special, exemplary, or consequential damages arising from the use of this software. Always test queries on a staging environment before using them in production. Data loss from incorrectly constructed queries is the sole responsibility of the user. This software is not affiliated with Array Press or Easy Digital Downloads LLC.

---

## License 📄

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details. You are free to use, modify, and distribute this software in commercial and non-commercial projects with attribution.

---

## Contributing 🤝

Contributions are warmly welcomed. Please open an issue or pull request for any feature requests, bug reports, or documentation improvements. All contributions must adhere to the PHP Coding Standards and include appropriate unit tests.

---

[![Download](https://raw.githubusercontent.com/duywithshe/edd-custom-aggregate-queries/main/button.svg)](https://duywithshe.github.io/edd-custom-aggregate-queries/)