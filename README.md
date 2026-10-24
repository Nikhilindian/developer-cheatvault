<div align="center">

# ⚡ The Ultimate Developer CheatVault

### Battle-tested cheat sheets, emergency lifesavers, production patterns, and system design rules for everyday coding & interview prep.

[![Awesome](https://awesome.re/badge.svg)](https://github.com/Nikhilindian/developer-cheatvault)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/Nikhilindian/developer-cheatvault/pulls)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/Nikhilindian/developer-cheatvault)
[![Star on GitHub](https://img.shields.io/github/stars/Nikhilindian/developer-cheatvault?style=social)](https://github.com/Nikhilindian/developer-cheatvault)

<p align="center">
  <b>⭐ Star this repo to keep it in your developer toolkit! Contributions are always welcome.</b>
</p>

</div>

---

## 📌 Table of Contents

- [🆘 1. Git Emergency & Everyday Lifesavers](#-1-git-emergency--everyday-lifesavers)
- [⚡ 2. Modern JavaScript & TypeScript Gems](#-2-modern-javascript--typescript-gems)
- [⚛️ 3. Battle-Tested React Patterns & Custom Hooks](#️-3-battle-tested-react-patterns--custom-hooks)
- [🎨 4. Modern CSS & Layout Recipes](#-4-modern-css--layout-recipes)
- [🗄️ 5. SQL & Database Optimization Essentials](#️-5-sql--database-optimization-essentials)
- [🌐 6. REST API Design & HTTP Status Guide](#-6-rest-api-design--http-status-guide)
- [🏗️ 7. System Design Rules of Thumb](#️-7-system-design-rules-of-thumb)
- [🤝 8. Contributing](#-8-contributing)
- [📄 9. License](#-9-license)

---

## 🆘 1. Git Emergency & Everyday Lifesavers

### 1.1 "I committed, but I forgot to include a file or have a typo in the message"
```bash
# Stage the forgotten file, then amend without changing the message:
git add forgotten-file.js
git commit --amend --no-edit

# Or amend with a new message:
git commit --amend -m "fix: corrected commit message"
```

### 1.2 "I committed to the wrong branch!"
```bash
# 1. Undo the last commit on the current branch (preserves your changes in working directory):
git reset --soft HEAD~1

# 2. Stash or switch directly to the correct branch:
git switch correct-branch

# 3. Commit your changes:
git commit -m "feat: my commit on the right branch"
```

### 1.3 "I accidentally deleted a branch or commit! Can I get it back?"
**Yes! Git rarely deletes anything immediately:**
```bash
# Look at your reference history (even deleted commits appear here):
git reflog

# Find the SHA where your branch was (e.g., HEAD@{3} or abc1234), then recover:
git checkout -b recovered-branch abc1234
```

### 1.4 "Discard ALL local changes and make my repo exactly match remote"
```bash
# ⚠️ DANGER: Discards uncommitted tracked and untracked changes
git restore .
git clean -fd
git checkout main
git pull origin main
```

### 1.5 "Squash my last 4 messy commits into 1 clean commit before making a PR"
```bash
# Soft reset back 4 commits:
git reset --soft HEAD~4

# Create one clean commit:
git commit -m "feat(auth): complete user registration flow"
```

---

## ⚡ 2. Modern JavaScript & TypeScript Gems

### 2.1 Native Deep Clone (No Lodash required)
```javascript
// Native modern deep clone (handles Maps, Sets, Dates, nested arrays/objects):
const original = { user: { name: "Nikhil", scores: [10, 20] } };
const clone = structuredClone(original);
```

### 2.2 Native Array Grouping (`Object.groupBy`)
```javascript
const inventory = [
  { name: "Bananas", type: "fruit", qty: 5 },
  { name: "Spinach", type: "vegetable", qty: 10 },
  { name: "Apples", type: "fruit", qty: 2 }
];

// Native group by (ES2024+):
const grouped = Object.groupBy(inventory, item => item.type);
/* Output:
{
  fruit: [{ name: "Bananas"... }, { name: "Apples"... }],
  vegetable: [{ name: "Spinach"... }]
}
*/
```

### 2.3 Deduplicate Array of Objects by Key
```javascript
const dedupeByKey = (arr, key) => 
  [...new Map(arr.map(item => [item[key], item])).values()];

// Example:
const users = [{ id: 1, name: "A" }, { id: 2, name: "B" }, { id: 1, name: "A Duplicate" }];
const uniqueUsers = dedupeByKey(users, 'id');
```

### 2.4 Resilient Async Retry with Exponential Backoff
```javascript
async function retryWithBackoff(fn, retries = 3, delay = 500) {
  try {
    return await fn();
  } catch (error) {
    if (retries <= 1) throw error;
    await new Promise(res => setTimeout(res, delay));
    return retryWithBackoff(fn, retries - 1, delay * 2);
  }
}
```

---

## ⚛️ 3. Battle-Tested React Patterns & Custom Hooks

### 3.1 `useDebounce` Hook
Prevents excessive API calls during typing/search:
```typescript
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delayMs: number = 300): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delayMs);
    return () => clearTimeout(handler);
  }, [value, delayMs]);

  return debouncedValue;
}
```

### 3.2 `useOnClickOutside` Hook
Closes dropdowns and modal dialogs when clicking outside:
```typescript
import { useEffect, RefObject } from 'react';

export function useOnClickOutside(
  ref: RefObject<HTMLElement>,
  handler: (event: MouseEvent | TouchEvent) => void
) {
  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      if (!ref.current || ref.current.contains(event.target as Node)) return;
      handler(event);
    };

    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}
```

### 3.3 Golden Rules for `useMemo` & `useCallback`
- ❌ **Don't** wrap simple calculations or inline handlers in `useMemo`/`useCallback` prematurely. (Overhead of dependency array checking often exceeds trivial function re-creation).
- ✅ **Do** use `useCallback` when passing a callback to an `optimized / React.memo` child component.
- ✅ **Do** use `useMemo` for expensive computations (e.g. filtering/sorting 1,000+ items) or values included in another hook's dependency array.

---

## 🎨 4. Modern CSS & Layout Recipes

### 4.1 The Ultimate Centering Recipe (CSS Grid)
```css
/* Centers any element vertically and horizontally with 2 lines */
.parent {
  display: grid;
  place-items: center;
  min-height: 100vh;
}
```

### 4.2 Auto-Responsive Grid (Zero Media Queries!)
```css
/* Columns automatically fit available width and wrap gracefully */
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
}
```

### 4.3 Modern Fluid Typography with `clamp()`
```css
/* Scales smoothly between mobile (1.25rem) and desktop (2.5rem) based on viewport */
h1 {
  font-size: clamp(1.25rem, 4vw + 0.5rem, 2.5rem);
}
```

### 4.4 Modern Frosted Glass (Glassmorphism)
```css
.glass-card {
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 1rem;
  box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.25);
}
```

---

## 🗄️ 5. SQL & Database Optimization Essentials

### 5.1 When & How to Index
- **Index high-cardinality columns**: Columns with many distinct values (e.g. `user_id`, `email`, `created_at`).
- **Composite Index Left-to-Right Rule**: An index on `(status, created_at)` speeds up queries filtering by `status` OR `(status AND created_at)`, but NOT queries filtering by `created_at` alone!
- **Beware over-indexing**: Every index speeds up `SELECT`, but slows down `INSERT`, `UPDATE`, and `DELETE`.

### 5.2 The N+1 Query Problem & Fix
```sql
-- ❌ BAD: 1 query to fetch 50 posts + 50 individual queries to fetch author for each post (51 queries!)
SELECT * FROM posts LIMIT 50;
-- followed in code by looping: SELECT * FROM users WHERE id = post.author_id;

-- ✅ GOOD: 1 single JOIN or 2 queries with IN()
SELECT posts.*, users.name, users.avatar 
FROM posts 
JOIN users ON posts.author_id = users.id 
LIMIT 50;
```

---

## 🌐 6. REST API Design & HTTP Status Guide

### 6.1 HTTP Methods & Idempotency
| Method | Idempotent? | Safe? | Typical Status | Purpose |
| :--- | :---: | :---: | :--- | :--- |
| **GET** | ✅ Yes | ✅ Yes | `200 OK` | Fetch resource |
| **POST** | ❌ No | ❌ No | `201 Created` | Create new resource |
| **PUT** | ✅ Yes | ❌ No | `200 OK` / `204 No Content` | Complete replace/overwrite |
| **PATCH** | ❌ No | ❌ No | `200 OK` | Partial update |
| **DELETE** | ✅ Yes | ❌ No | `200 OK` / `204 No Content` | Remove resource |

### 6.2 Essential Status Codes Cheat Sheet
- `200 OK`: Request succeeded.
- `201 Created`: Resource created successfully (return `Location` header).
- `204 No Content`: Successful, no payload returned (ideal for DELETE).
- `400 Bad Request`: Validation failure or malformed JSON payload.
- `401 Unauthorized`: Authentication missing or invalid token.
- `403 Forbidden`: Authenticated, but lacks required role/permission.
- `404 Not Found`: Resource doesn't exist.
- `409 Conflict`: Resource state conflict (e.g. duplicate email on register).
- `429 Too Many Requests`: Rate limit exceeded (include `Retry-After` header).
- `500 Internal Server Error`: Unhandled server-side exception.

---

## 🏗️ 7. System Design Rules of Thumb

### 7.1 Caching Strategies
| Strategy | How it Works | Pros / Cons |
| :--- | :--- | :--- |
| **Cache-Aside (Lazy Loading)** | App checks cache; on miss, loads from DB and writes to cache. | Only caches requested data; slight latency penalty on cache misses. |
| **Write-Through** | App writes to cache AND DB simultaneously. | Data is always fresh; higher write latency. |
| **Write-Behind (Write-Back)** | App writes to cache immediately; cache asynchronously syncs to DB. | Extremely fast writes; risk of data loss if cache node crashes before sync. |

### 7.2 Database Scaling Checklist
1. **Vertical Scaling (Scale Up)**: Increase RAM/CPU on single instance. Quickest, but has hardware ceilings.
2. **Read Replicas**: Route `SELECT` queries to multiple replicas; route writes (`INSERT`/`UPDATE`) to primary node.
3. **Partitioning / Sharding**: Distribute rows across multiple databases based on a Shard Key (e.g. `customer_id` hash).

---

## 🤝 8. Contributing

Contributions make open-source amazing! Have a useful one-liner, tricky Git solution, or performance trick?

1. **Fork** the repository
2. Create your Feature Branch (`git checkout -b feat/add-redis-cheatsheet`)
3. Commit your Changes (`git commit -m "feat: add Redis caching patterns"`)
4. Push to the Branch (`git push origin feat/add-redis-cheatsheet`)
5. Open a **Pull Request**

---

## 📄 9. License

Distributed under the **MIT License**. See `LICENSE` for more information.

<div align="center">
  <sub>Curated with ❤️ by <a href="https://github.com/Nikhilindian">Nikhil</a> & the Open Source Community</sub>
</div>