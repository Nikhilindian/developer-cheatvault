# Developer Cheat Vault

A collection of practical commands, common code snippets, and architecture notes for day-to-day software development.

---

## Contents

- [Git Recovery & Common Tasks](#git-recovery--common-tasks)
- [JavaScript & TypeScript Snippets](#javascript--typescript-snippets)
- [React Hooks & Patterns](#react-hooks--patterns)
- [CSS & Layout Snippets](#css--layout-snippets)
- [SQL & Database Patterns](#sql--database-patterns)
- [HTTP & REST Reference](#http--rest-reference)
- [System Architecture Notes](#system-architecture-notes)
- [License](#license)

---

## Git Recovery & Common Tasks

### Amend last commit (add forgotten file or change message)
```bash
# Add forgotten file without editing commit message:
git add <file>
git commit --amend --no-edit

# Update commit message only:
git commit --amend -m "new message"
```

### Move recent commit to a new or different branch
```bash
# Reset branch pointer back one commit (keeps working tree untouched):
git reset --soft HEAD~1

# Switch to desired branch and commit:
git switch <target-branch>
git commit -m "commit message"
```

### Recover dropped commit or deleted branch
```bash
# View reflog to locate target commit SHA:
git reflog

# Recreate branch pointing to that commit:
git checkout -b <branch-name> <commit-sha>
```

### Discard local changes and align with remote
```bash
# Warning: discards uncommitted changes permanently
git restore .
git clean -fd
git fetch origin
git reset --hard origin/main
```

### Squash last N commits into one
```bash
# Soft reset back N commits:
git reset --soft HEAD~N
git commit -m "clean commit message"
```

---

## JavaScript & TypeScript Snippets

### Deep clone (Native)
```javascript
// Native structuredClone handles Maps, Sets, Dates, and nested objects
const copy = structuredClone(original);
```

### Group array of objects by key (`Object.groupBy`)
```javascript
const items = [
  { name: 'Apple', type: 'fruit' },
  { name: 'Carrot', type: 'vegetable' },
  { name: 'Banana', type: 'fruit' },
];

const grouped = Object.groupBy(items, item => item.type);
// { fruit: [...], vegetable: [...] }
```

### Deduplicate array of objects by property
```javascript
const uniqueByKey = (arr, key) =>
  [...new Map(arr.map(item => [item[key], item])).values()];

const deduped = uniqueByKey(users, 'id');
```

### Async retry with exponential backoff
```javascript
async function retry(fn, retries = 3, delay = 500) {
  try {
    return await fn();
  } catch (err) {
    if (retries <= 1) throw err;
    await new Promise(r => setTimeout(r, delay));
    return retry(fn, retries - 1, delay * 2);
  }
}
```

---

## React Hooks & Patterns

### `useDebounce`
Delays value update until specified idle duration passes (useful for search inputs).

```typescript
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delayMs: number = 300): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delayMs);
    return () => clearTimeout(timer);
  }, [value, delayMs]);

  return debouncedValue;
}
```

### `useOnClickOutside`
Fires a callback when clicks or touch events land outside a target element (modals, popovers, dropdowns).

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

---

## CSS & Layout Snippets

### Centering with CSS Grid
```css
.parent {
  display: grid;
  place-items: center;
}
```

### Responsive grid without media queries
```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
}
```

### Fluid typography (`clamp`)
```css
/* Scales between 1.25rem and 2.5rem based on viewport width */
h1 {
  font-size: clamp(1.25rem, 4vw + 0.5rem, 2.5rem);
}
```

### Glassmorphism card
```css
.card {
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 0.75rem;
}
```

---

## SQL & Database Patterns

### Composite Index: Left-to-Right Prefix Rule
An index defined on `(status, created_at)` can be utilized for:
- Filters on `status` alone
- Filters on `status AND created_at`

It will **not** be used for queries filtering on `created_at` alone without leading `status`.

### Resolving the N+1 Query Problem
```sql
-- Problem: 1 query for posts + N queries inside application loop to fetch authors
SELECT * FROM posts LIMIT 50;

-- Solution: Single JOIN
SELECT posts.*, users.name, users.avatar
FROM posts
JOIN users ON posts.author_id = users.id
LIMIT 50;
```

---

## HTTP & REST Reference

### Methods & Idempotency
| Method | Idempotent | Safe | Typical Success Code | Intended Use |
| :--- | :---: | :---: | :--- | :--- |
| `GET` | Yes | Yes | `200 OK` | Read resource |
| `POST` | No | No | `201 Created` | Create resource |
| `PUT` | Yes | No | `200 OK` / `204 No Content` | Full replace |
| `PATCH` | No | No | `200 OK` | Partial update |
| `DELETE` | Yes | No | `200 OK` / `204 No Content` | Delete resource |

### Common Status Codes
- `200 OK`: Request succeeded.
- `201 Created`: Resource successfully created.
- `204 No Content`: Succeeded, no body in response.
- `400 Bad Request`: Validation error or invalid payload.
- `401 Unauthorized`: Missing or invalid credentials.
- `403 Forbidden`: Authenticated, but insufficient permissions.
- `404 Not Found`: Resource does not exist.
- `409 Conflict`: Request conflict (e.g., unique key violation).
- `422 Unprocessable Entity`: Syntactically valid request, but semantic validation failed.
- `429 Too Many Requests`: Rate limit hit.
- `500 Internal Server Error`: Unhandled server exception.

---

## System Architecture Notes

### Caching Patterns
| Pattern | Mechanism | Tradeoff |
| :--- | :--- | :--- |
| **Cache-Aside** | Application reads from cache; on miss, reads from DB and populates cache. | Cache only contains requested data; first read carries cache miss latency. |
| **Write-Through** | Writes update cache and DB synchronously. | Consistent data; higher write latency. |
| **Write-Behind** | Writes hit cache immediately, batch written to DB asynchronously. | Very low write latency; risk of data loss if cache fails before flushing. |

---

## Contributing

Pull requests are welcome. Please keep code snippets dependency-free, clear, and focused on practical use cases.

---

## License

[MIT](LICENSE)