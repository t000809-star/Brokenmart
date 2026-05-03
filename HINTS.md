# MiniMart — Hints

Each bug below is a clue, not a fix. Read the hint, find the spot, and figure out the fix yourself. If you really get stuck, the full answer key is in `docs/INSTRUCTOR-BUGS.md` (instructor-only — try not to peek).

There are **32 bugs** across 7 categories. Pace yourself — some take 30 seconds, some take 5+ minutes of reading.

---

## A. UI / Interactivity (4)

1. **The big orange "Add to Cart" button on the product detail page does nothing.** Click it. Notice the qty +/− buttons next to it work. So the button renders fine — what's missing? Look in `renderProductDetail()`.
2. **In the cart, the +/− steppers change the displayed numbers but the grand total never moves.** Where do the steppers update state, and where does `paintCartSummary()` read state from? They aren't talking to each other.
3. **The category buttons in the products-page sidebar don't filter anything.** Click them — the active highlight moves but the grid doesn't change. Find the function that builds the grid (`paint()` inside `renderProducts`). Does it actually use `PRODUCTS_PAGE_STATE.category`?
4. **Same shape: the search input updates state on every keystroke but the grid never refilters.** Look at the `input` listener and at `paint()` — neither one does anything with the search string.

## B. Calculations (3)

5. **Apply the `WELCOME10` promo. Tax doesn't go down.** Tax should be charged on the *post-discount* subtotal, not the original. Order matters in `paintCartSummary()`.
6. **Add up exactly $50 of products. Shipping is still $5.** The hint is in the number: the comparison is *strictly greater than*, not *greater or equal*. Two places — cart and checkout.
7. **Apply `WELCOME10`. Discount line shows `−$1.00`.** The promo was supposed to be ten percent. Look at how `discount` is computed.

## C. Routing & state (3)

8. **Click any product card → URL becomes `#product/undefined`.** Look at the card-renderer functions. They build the link from a property that doesn't exist on the product object. (Compare the link to the actual data in `PRODUCTS`.)
9. **Add to cart → refresh the page → cart is empty.** The cart is held in a JS variable that resets on every load. There's no localStorage hydration. Hydrate on `load`, persist on every mutation.
10. **After (eventually) signing in, you land on `#undefined`.** Look at the success path in `renderSignin()` — the redirect string is built from a value the server doesn't return.

## D. API / data fetching (3)

11. **`/products` loads forever.** Open the console — there's a `ReferenceError`. The page calls a function that doesn't exist (the real one has a slightly different name). Compare the call site to the `apiGet*` functions defined in the script.
12. **Product cards on the home page have empty titles.** The data loaded fine — `console.log(MM.PRODUCTS[0])` shows a `name` field. The renderer is reading a different property name.
13. **Click "Place Order" on a slow connection. The success banner fires before the order actually saves.** The handler isn't `await`ing the API call. Look for `apiCreateOrder({...})` — is there an `await` in front of it?

## E. Auth (4)

14. **Sign-in always fails, even with correct credentials.** Use the demo-account auto-fill button on the signin page — it gives you Alice's known-good password. Login still fails. So the credentials aren't the problem. Look at the body of the `apiLogin` call — what field name does it pass? What field does `apiLogin` read?
15. **Create a new account → you're sent to the signin page with no message.** A new user shouldn't have to log in again immediately. Compare what happens on signin success vs. signup success.
16. **Sign in, then click "Sign out" in the navbar. Your name is still there until you refresh.** `doLogout` clears `localStorage` but forgets about the in-memory state.
17. **The "Forgot password?" link on the signin page goes to a 404.** The href points to a route that isn't handled by the router.

## F. Security / OWASP (5)

18. **Order detail page leaks orders across users.** Sign in as Alice (`#account?userId=1` shows her orders — `#order/1001`). Then visit `#order/1002`. That's Bob's order. No ownership check.
19. **Account page leaks profiles.** Visit `#account?userId=1`, then change the URL to `#account?userId=2`. You're now seeing Bob's profile. The page reads `userId` from the URL, not from the session.
20. **`#admin` is reachable by anyone.** No auth check. Open it while signed out — you're in.
21. **Visit `#product/25` (Moleskine Notebook). An XSS alert fires.** The product description is being injected with `innerHTML`. The seed product 25's description has an `<img src=x onerror="...">` payload. Either sanitize the HTML, render as text, or strip the payload.
22. **Open the Network tab → look at the response of any product fetch (or `console.log(MM.PRODUCTS)`).** Every product has `sellerEmail` and `sellerPhone`. The frontend never displays these — they shouldn't be in the response.

## G. Missing features / UX gaps (2)

23. **Once an item is in the cart, there's no way to remove it.** No remove button, and the stepper floors at 1.
24. **Open the cart while empty.** The header has a "Continue shopping" link, but the body of the page is blank. Add a real empty-state UI.

---

## Tier 2 — extras (8)

25. **Decode your JWT.** After signing in, open the console: `localStorage.getItem('token')`. Paste the value into [jwt.io](https://jwt.io). The payload contains your plaintext password.
26. **View source.** Search the file for `admin_override_` (or just `ADMIN_KEY`). There's a hardcoded admin key sitting in client-side code — exactly where it shouldn't be.
27. **Open `#signin?redirect=https://example.com`.** Sign in (once the auth bug is fixed). You're redirected to whatever the `redirect` param says, no allowlist. That's an open redirect.
28. **Open the console. Refresh the page.** The JWT is `console.log`ged on every load.
30. **Run `MM.__debugUsers()` in the console.** Every user object comes back, plaintext passwords included. Two problems here: passwords stored unhashed, and a debug endpoint exposed to the client.
32. **Right-click any product image → Inspect.** The `<img>` tags have no `alt` attribute. Accessibility miss across every image renderer.
33. **Navigate to `/products` or any product page.** The loading state is the word "Loading…" instead of skeleton placeholders that match the final layout.
35. **Try signing up with email `notanemail`.** Accepted. The email input is `type="text"` instead of `type="email"`, and there's no JS-side validation either.

---

## Suggested workflow

1. Open the app, click around for 5 minutes, jot down anything that feels broken.
2. Open one AI thread per category (A through G) and feed it the relevant hints from this doc + the file.
3. Run them in parallel — that's the meta-lesson of the exercise.
4. Test your fix in the browser before moving on. The bugs are small (each fixable in <10 lines).

Good luck.
