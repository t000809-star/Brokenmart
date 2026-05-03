# MiniMart

MiniMart is a small online store. The entire app lives in a single file: `index.html`.

## Setup

There is no build step. Open `index.html` in a browser and you're off:

```bash
open index.html
```

Or serve it from the directory if you want a real `http://` origin (helps with some browser features):

```bash
python3 -m http.server 8080
# → http://localhost:8080
```

## Seed accounts

Three pre-baked users you can sign in with once login is working:

| Email | Password |
|---|---|
| `alice@example.com` | `password123` |
| `bob@example.com`   | `password456` |
| `admin@minimart.com`| `admin123` |

State (cart, orders, users) lives in JavaScript variables — refreshing the page resets the in-memory store.

## Your job

There are bugs. Find them. Fix them. Open multiple AI threads — one per category — and run them in parallel.

The bugs span 7 categories:

- **A.** UI / interactivity
- **B.** Calculations
- **C.** Routing & state
- **D.** API / data fetching
- **E.** Auth
- **F.** Security / OWASP
- **G.** Missing features / UX gaps

The whole app is one file. Lots of "components" are written from scratch instead of reused — there are 3 product-card variants, 3 money formatters, multiple primary-button classes that look identical. Some of that is style preference; some of it is hiding bugs. Treat the code with suspicion.

If you get stuck, **[HINTS.md](HINTS.md)** has a clue for every bug — enough to point you at the right area without giving away the fix. Use it sparingly.

## Routes

The app is a hash-routed SPA. Once you know the hash table you can poke at hidden pages directly:

| Hash | Page |
|---|---|
| `#home` | Home |
| `#products` | Products list |
| `#product/<id>` | Product detail |
| `#cart` | Cart |
| `#checkout` | Checkout |
| `#signin` | Sign in |
| `#signup` | Sign up |
| `#account?userId=<id>` | Account |
| `#admin` | Admin dashboard *(not linked from the nav)* |
| `#order/<id>` | Order detail *(not linked from the nav)* |

## Off-limits

The `docs/` folder is the instructor's answer key. **Do not open it during the exercise.** Looking at it spoils the hunt for everyone.
