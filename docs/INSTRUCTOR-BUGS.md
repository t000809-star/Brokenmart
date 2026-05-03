# MiniMart — Instructor Answer Key

**Do not show this file to students before the exercise.**

The whole app lives in `index.html` (~1570 lines). 32 bugs planted across 7 categories. All 24 Tier-1 items are present plus 8 Tier-2 items.

Line numbers are accurate against the as-shipped file. They will shift if students edit nearby code — function names and snippets are the durable references.

---

## A. UI / Interactivity (4)

### 1. Add to Cart button has no click handler
- **Where:** `renderProductDetail()` around line 876 — the `<button id="add-to-cart-btn">` is rendered but never wired up. The qty +/− buttons get listeners; this one doesn't.
- **Fix:** After rendering the detail HTML (around line 924), add:
  ```js
  document.getElementById("add-to-cart-btn").addEventListener("click", () => {
    const existing = CART.find(c => c.productId === product.id);
    if (existing) existing.quantity += qty;
    else CART.push({ productId: product.id, name: product.name, price: product.price, image: product.image, quantity: qty });
    renderNavbar();
    showToast("Added to cart");
  });
  ```

### 2. Quantity steppers in cart don't update the cart total
- **Where:** `attachCartLineHandlers()`, lines 1015–1037. The +/− buttons mutate a local `localQty` variable but never touch `CART`, so `paintCartSummary()` recomputes from the unchanged source.
- **Fix:** Mutate `item.quantity` (the actual `CART` reference) and call `paintCartSummary()`:
  ```js
  row.querySelector(".line-dec").addEventListener("click", () => {
    item.quantity = Math.max(1, item.quantity - 1);
    qtyEl.textContent = item.quantity;
    totalEl.textContent = priceToString(item.price * item.quantity);
    paintCartSummary();
    attachPromoHandler();
  });
  // (mirror change for line-inc)
  ```

### 3. Category filter doesn't filter the product list
- **Where:** `renderProducts().paint()` around line 815. `paint()` only sorts; it doesn't filter by `PRODUCTS_PAGE_STATE.category`. Clicking category buttons (around line 837) toggles the `.active` class but no filtering runs.
- **Fix:** Inside `paint()`, filter before sorting:
  ```js
  const filtered = products.filter(p =>
    PRODUCTS_PAGE_STATE.category === "All" || p.category === PRODUCTS_PAGE_STATE.category
  );
  const list = [...filtered].sort(/* same as before */);
  ```
  Wire the cat-btn click to update state and call `paint()`:
  ```js
  btn.addEventListener("click", () => {
    PRODUCTS_PAGE_STATE.category = btn.dataset.cat;
    document.querySelectorAll(".cat-btn").forEach(b => b.classList.remove("active"));
    btn.classList.add("active");
    paint();
  });
  ```

### 4. Search input doesn't filter the product list
- **Where:** Same `renderProducts()` block, around line 829 — the `input` listener writes to `PRODUCTS_PAGE_STATE.search` but never re-paints, and `paint()` doesn't apply the filter.
- **Fix:** Extend the filter chain to include `p.name.toLowerCase().includes(PRODUCTS_PAGE_STATE.search.toLowerCase())`. Call `paint()` from the input listener.

---

## B. Calculations (3)

### 5. Tax is calculated before the discount is applied
- **Where:** `paintCartSummary()`, line 1041.
- **Fix:** Compute the discount first, then `const tax = (subtotal - discount) * 0.10;`.

### 6. Free-shipping off-by-one — exactly $50 still pays shipping
- **Where:** `paintCartSummary()` line 1043, also `renderCheckout()` line 1086.
- **Fix:** `subtotal >= 50 ? 0 : 5` in both places.

### 7. `WELCOME10` applies as a flat $1 instead of 10%
- **Where:** `paintCartSummary()`, line 1042.
- **Fix:** `const discount = APPLIED_CODE === "WELCOME10" ? subtotal * 0.10 : 0;`.

---

## C. Routing & state (3)

### 8. Product cards link to `#product/undefined`
- **Where:** `renderProductCardForHome()` line 732, `renderProductCardForList()` line 747 — both build the href from `product.productId`, but the data only has `id`.
- **Fix:** Change `product.productId` to `product.id` in both functions.

### 9. Cart wipes on refresh — no persistence
- **Where:** `CART` declared as a top-level `let` around line 388 with no localStorage hydration.
- **Fix:** On `load`, hydrate: `try { CART = JSON.parse(localStorage.getItem("cart") || "[]"); } catch {}`. After every cart mutation, persist: `localStorage.setItem("cart", JSON.stringify(CART));`.

### 10. After login, redirect goes to `#undefined` instead of `#account`
- **Where:** `renderSignin()`, line 1233 — `navigate("#" + data.redirectTo)`, but `data.redirectTo` is never set on the login response.
- **Fix:** `navigate("#account?userId=" + data.user.id)`.

---

## D. API / data fetching (3)

### 11. Products page calls `apiGetProductList()` — that function doesn't exist
- **Where:** `renderProducts()`, line 805. The real function is `apiGetProducts()`.
- **Fix:** Change `await apiGetProductList()` to `await apiGetProducts()`. Promise rejects silently and the loading state never goes away — the `catch` block re-renders the same loading message.

### 12. Product cards render `product.title` — data only has `name`
- **Where:** `renderProductCardForHome()` line 737, `renderProductCardForList()` line 752.
- **Fix:** Change `product.title` to `product.name` in both renderers.

### 13. Place Order doesn't `await` — toast fires before the order saves
- **Where:** `placeOrder()` around line 1148 — `apiCreateOrder({...})` is fire-and-forget. The success banner renders immediately and `CART` is cleared whether or not the API resolved.
- **Fix:** Make `placeOrder` async and `await apiCreateOrder({...})` before clearing the cart and rendering the success banner. Show an error banner on rejection.

---

## E. Auth (4)

### 14. Login form sends `username`; API expects `email`
- **Where:** `renderSignin()`, line 1222 — `apiLogin({ username: email, password })`. `apiLogin` does `USERS.find(x => x.email === body.email && ...)`, so `body.email` is `undefined` and login always fails. Use the demo-account auto-fill buttons to confirm credentials are correct — then notice login still fails.
- **Fix:** `apiLogin({ email, password })`.

### 15. After signup, user is not auto-logged in
- **Where:** `renderSignup()`, line 1281 — `navigate("#signin")` after a successful signup, with no message.
- **Fix:**
  ```js
  const data = await apiSignup({ email, password, name });
  JWT_TOKEN = data.token;
  CURRENT_USER = data.user;
  localStorage.setItem("token", data.token);
  localStorage.setItem("user", JSON.stringify(data.user));
  renderNavbar();
  navigate("#account?userId=" + data.user.id);
  ```

### 16. Logout removes localStorage but does not clear runtime state
- **Where:** `doLogout()`, lines 1502–1506 — only `localStorage.removeItem(...)` calls. The navbar still shows the user as signed in until the page is refreshed.
- **Fix:**
  ```js
  function doLogout() {
    localStorage.removeItem("token");
    localStorage.removeItem("user");
    JWT_TOKEN = null;
    CURRENT_USER = null;
    renderNavbar();
    navigate("#home");
  }
  ```

### 17. "Forgot password?" link points to `#forgot` (404)
- **Where:** `renderSignin()`, line 1190 — the `auth-meta` block has `<a href="#forgot">`. The router has no `forgot` case.
- **Fix:** Either remove the link, change the target, or implement a `#forgot` route.

---

## F. Security / OWASP (5)

### 18. IDOR on order detail — no ownership check
- **Where:** `renderOrderDetail()` and `apiGetOrder()` — the function returns any order by ID with no check that `CURRENT_USER.id === order.userId`. Try `#order/1001`, `#order/1002`, `#order/1003` while signed in as anyone (or signed out).
- **Fix:** In `apiGetOrder`, throw if `!CURRENT_USER || CURRENT_USER.id !== o.userId`. (For real apps, do this server-side, not in client code.)

### 19. IDOR on account — userId taken from URL
- **Where:** `renderAccount()` reads `params.userId` from the hash query string (around line 1334). Try `#account?userId=2` while signed in as Alice — you'll see Bob's order history.
- **Fix:** Derive userId from `CURRENT_USER`, not the URL. Block the page if `CURRENT_USER` is null.

### 20. Admin no auth check
- **Where:** `renderAdmin()` — renders unconditionally. Anyone can hit `#admin` directly.
- **Fix:** Guard at the top of the function:
  ```js
  if (!CURRENT_USER || CURRENT_USER.email !== "admin@minimart.com") {
    navigate("#signin");
    return;
  }
  ```

### 21. Stored XSS via `innerHTML`
- **Where:** `renderProductDetail()`, lines 913–914 — `desc.innerHTML = product.description`. The seed data for product `id: "25"` (Moleskine Classic Notebook, line 359) contains `<img src=x onerror="alert('XSS')">`. Visit `#product/25` and the alert fires.
- **Fix:** Render the description as text (`textContent`) or sanitize with a library before injecting. Strip the `<img>` from product 25's description in the seed data.

### 22. Sensitive data leak — `sellerEmail` and `sellerPhone` on every product
- **Where:** `PRODUCTS` array, lines 335–366. The fields are returned by `apiGetProducts` / `apiGetProduct` even though the UI never displays them. Visible via `console.log(MM.PRODUCTS)` or by inspecting the `apiGetProducts` return value.
- **Fix:** In `apiGetProducts` / `apiGetProduct`, strip those fields before returning: `({ sellerEmail, sellerPhone, ...rest }) => rest`.

---

## G. Missing features / UX gaps (2)

### 23. No "Remove from cart" — quantity stepper floors at 1
- **Where:** `attachCartLineHandlers()` line 1025 — `Math.max(1, localQty - 1)`. There's no remove button, so once an item is in the cart there's no way to take it out.
- **Fix:** Add a remove button to `renderCartLine()`. Add a `removeFromCart(productId)` helper that splices from `CART`, calls `paintCartSummary()`, and re-renders the cart line list.

### 24. Empty cart shows blank space — no empty state
- **Where:** `renderCart()` — when `CART` is empty, the line list renders nothing and the summary just shows zeros. There's a "Continue shopping" link in the header but no large empty-state CTA in the body.
- **Fix:** Early-return an empty-state UI when `CART.length === 0`.

---

## Tier 2 — also planted (8)

### 25. JWT payload contains plaintext password
- **Where:** `apiLogin()` line 424 and `apiSignup()` line 433 — `makeFakeJwt({ ..., password: u.password, ... })`. Decode the resulting token at jwt.io to read it. The `localStorage.token` after sign-in is the live one.
- **Fix:** Drop `password` from the payload.

### 26. Hardcoded "admin override" key in client-side source
- **Where:** Line 331 — `const ADMIN_KEY = "admin_override_8f3a..."`. Visible to anyone who views source. Also re-exposed via `window.MM.ADMIN_KEY` near the bottom of the file.
- **Fix:** Delete the constant and the `window.MM.ADMIN_KEY` reference. Real secrets belong server-side.

### 27. Open redirect on `#signin?redirect=...`
- **Where:** `renderSignin()`, line 1231 — `if (redirect) window.location.href = redirect;` with no allowlist.
- **Fix:** Only follow the redirect if it starts with `#` (in-app) or `/` (relative). Reject absolute URLs.

### 28. JWT logged on every page load
- **Where:** `load` event listener, line 1559 — `console.log("Auth token:", stored)`.
- **Fix:** Remove the line.

### 30. Plaintext passwords in store, exposed via `__debugUsers()`
- **Where:** `USERS` array (lines 369–373) holds plaintext passwords. `__debugUsers()` (line 466) returns the full user list including passwords. Also exposed via `window.MM.__debugUsers` at the bottom of the file. Open the browser console and run `MM.__debugUsers()`.
- **Fix:** Delete `__debugUsers`, remove the `window.MM` exposure, and store passwords hashed (e.g. SHA-256 or, in real life, bcrypt).

### 32. Product images have no `alt` attribute
- **Where:** Every `<img>` tag in the renderers — `renderProductCardForHome`, `renderProductCardForList`, `renderProductDetail`, `renderCartLine`, `renderOrderDetail`, `categoryTile`, the order-review lines in checkout.
- **Fix:** Add `alt="<product or category name>"` to every image template.

### 33. No loading skeletons — pages show "Loading..." text or blank
- **Where:** `renderProducts`, `renderProductDetail`, `renderAccount`, `renderOrderDetail` all use `<div class="empty-section">Loading...</div>` instead of skeleton placeholders.
- **Fix:** Render placeholder cards/rows that match the final layout dimensions while data loads.

### 35. No email format validation on signup
- **Where:** `renderSignup()` line 1254 — `<input id="su-email" type="text" required />`. You can register with `notanemail`.
- **Fix:** Change to `<input type="email">` and/or add a regex check before submitting.

---

## Summary

| Category | Bug count |
|---|---|
| A. UI / Interactivity | 4 |
| B. Calculations | 3 |
| C. Routing & state | 3 |
| D. API / data fetching | 3 |
| E. Auth | 4 |
| F. Security / OWASP | 5 |
| G. Missing features / UX | 2 |
| Tier 2 (extras) | 8 |
| **Total** | **32** |

All Tier-1 items (1–24) are present.
