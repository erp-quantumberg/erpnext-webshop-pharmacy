# Pharmacy

Pharmacy is a Frappe application for an online pharmacy storefront. It adds
pharmacy-specific website sections and content management while using the
Frappe/ERPNext webshop for catalog and shopping-cart functionality.

## Features

- Pharmacy storefront homepage with hero banners, categories, promotional
  images and videos, product carousels, and products grouped by tags.
- Admin-managed hero banners, promotion images, promotion videos, blog posts,
  and newsletter subscriptions.
- Public API methods for product tags and products, blog listings and details,
  contact form submissions, and newsletter subscriptions.
- Login-aware cart and wishlist counts, plus an add-to-cart method for signed-in
  users.
- Product-list image customization for the ERPNext webshop interface.
- Website pages and content sections for contact information, about, privacy,
  and terms and conditions.

## Requirements

### Required Frappe apps

Install the following dependencies on the same Bench site before installing
`pharmacy`:

| App | Requirement | Used for |
| --- | --- | --- |
| **Frappe Framework** (`frappe`) | Required | App runtime, DocTypes, database access, and APIs. |
| **ERPNext** (`erpnext`) | Required | Item, Item Group, Item Price, Website Item, and wishlist data and features. |
| **Frappe Webshop** (`webshop`) | Required for cart features | Cart methods imported from `webshop.webshop.shopping_cart.cart`. Depending on the Frappe/ERPNext release, Webshop may be included with ERPNext or installed as a separate app. Ensure its `webshop` module is available on the site. |

Use mutually compatible Frappe and ERPNext releases. This repository does not
pin their versions; confirm the target branch and versions used by your Bench
before installation.

### Runtime and development tools

- Python 3.10 or later (declared by `pyproject.toml`).
- A working Frappe Bench environment and its supported database, Redis, and
  Node.js tooling. Follow the Frappe installation guide for the prerequisites
  appropriate to your operating system.
- `pre-commit` for the repository's configured formatting and lint checks
  (development only).

This repository has no root `package.json`; it is not a standalone npm
frontend. You do not need to run `npm install` in this app directory. Bench
manages the assets and dependencies of the installed Frappe apps.

## Installation

The following assumes you already have a Bench and a site. Replace the
repository URL and branch with the source and compatible branch you intend to
use.

Install ERPNext and the compatible Webshop module on the site before installing
`pharmacy`. If either required app is not already present in the Bench, fetch it
and install it on the site following the instructions for the selected release.

```bash
cd /path/to/your/frappe-bench
bench get-app --branch <branch> <repository-url>
bench --site <site-name> install-app pharmacy
bench --site <site-name> migrate
bench build --app pharmacy
bench restart
```

## Setup

After installation:

1. Sign in to the site's Desk with a System Manager account.
2. Add or review ERPNext Items, Item Groups, prices, and Website Items. Publish
   the products that should appear in the storefront, and add tags to products
   used by the product-by-tag sections.
3. Set the relevant Item Groups to show on the website.
4. Create and activate **Hero Banner Image**, **Promotion Images**, and
   **Promotion Videos** records. Set their order sequence where available.
5. Add **Blogs** records for blog content and images.
6. Configure an outgoing email account in Frappe for newsletter notification
   emails.
7. Open the site's root page to view the storefront.

The storefront reads its category, hero-banner, and promotion-video data from
Frappe records. Product and cart behavior also depends on the ERPNext catalog,
website publication state, and webshop configuration.

## API methods

The following whitelisted methods are provided by this app and can be called
through Frappe's `/api/method/` endpoint:

| Method | Purpose |
| --- | --- |
| `pharmacy.api.products.get_product_tags` | Lists tags associated with Items. |
| `pharmacy.api.products.get_products_by_tag` | Returns published products for a tag. |
| `pharmacy.api.blogs.get_blogs` | Lists blog records. |
| `pharmacy.api.blogs.get_blog` | Returns one blog by its record name. |
| `pharmacy.api.cart.add_to_cart` | Adds a product to the signed-in user's cart. |
| `pharmacy.api.cart.get_cart_count` | Returns the signed-in user's cart quantity. |
| `pharmacy.api.cart.get_wishlist_count` | Returns the signed-in user's wishlist count. |
| `pharmacy.api.contact.submit_contact` | Submits the storefront contact form. |
| `pharmacy.api.email.subscribe_user` | Adds a newsletter subscription. |

Methods marked for guest access are callable without signing in, but cart
updates require an authenticated user.

## Configuration notes

- The app metadata does not pin Frappe or ERPNext versions. Keep those apps on
  compatible releases and test upgrades on a non-production site first.
- Products must exist in ERPNext and be published as Website Items to appear in
  the storefront. Product-by-tag results use ERPNext Item tags.
- Newsletter mail is sent through Frappe's configured outgoing email service.
- The contact endpoint has a direct SMTP implementation. Before deploying,
  ensure its SMTP settings are stored securely outside tracked source files;
  rotate any SMTP credential that has previously been committed to the
  repository.
- Do not put site-specific credentials, secrets, or customer data in this
  README or in version control.
- The app currently includes legal-information pages, but their text should be
  reviewed and customized for the business and jurisdictions where the site
  operates.

## Troubleshooting

### `ModuleNotFoundError: No module named 'webshop'`

Check that ERPNext and the compatible Webshop module are installed on the site.
Confirm the apps are available on the same site, then run
`bench --site <site-name> migrate` and restart the Bench.

### Products or categories do not appear

Confirm the Item is enabled, its Website Item is published, it has a website
route, and the relevant Item Group is configured to show on the website. For
tag-based product sections, ensure the Item has the expected tag.

### Banner or promotion content is missing

Check that the relevant record exists, is marked active, has the required
image/video attached, and has an order sequence if the section uses one. After
deploying code changes, run `bench build --app pharmacy` and restart the Bench.

### Email notifications are not delivered

For newsletter messages, verify the default outgoing email account and Frappe
email queue. For contact form messages, verify the contact endpoint's SMTP
configuration and server connectivity; do not add SMTP credentials to tracked
files.

### Tests fail or the app cannot be installed

Check that the site has compatible Frappe and ERPNext apps installed, then run
`bench --site <site-name> migrate`. To run this app's tests, use:

```bash
bench --site <site-name> run-tests --app pharmacy
```

## Contributing

Contributions are welcome. Please keep changes focused, follow the existing
Frappe app conventions, and test changes on a compatible Bench site.

Install the configured pre-commit hooks from the app directory:

```bash
cd apps/pharmacy
pre-commit install
pre-commit run --all-files
```

The repository's hooks include Ruff, ESLint, Prettier, and pyupgrade. For
application tests, see the command in [Troubleshooting](#troubleshooting).

## License

The app declares the MIT license in its metadata. See [license.txt](license.txt).
Before distributing the project, replace the template copyright year and
copyright holder in that file with the applicable details.
