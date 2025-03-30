
# 🔐 CASL – Permissions Documentation

This document explains how to use **CASL (Code Access Security Layer)** to manage and enforce permissions across both frontend and backend using a shared `auth` package.

> CASL official website: [https://casl.js.org](https://casl.js.org)  
> GitHub repo: [https://github.com/stalniy/casl](https://github.com/stalniy/casl)

---

## 📦 Installation

Install core package:

```bash
npm install @casl/ability
```

If you're using React:

```bash
npm install @casl/react
```

---

## 🧠 Core Concepts

### ➤ Abilities

An **ability** defines what actions a user can or cannot perform on specific resources.

```ts
can('read', 'Article');
can('update', 'Article', { authorId: user.id });
```

### ➤ Actions

These are string labels for user actions, e.g.:

- `'read'`, `'create'`, `'update'`, `'delete'`
- or custom: `'publish'`, `'like'`, `'vote'`

### ➤ Subjects

Subjects represent resource types:

- `'Article'`, `'User'`, `'Invoice'`
- `'all'` (wildcard for any resource)

### ➤ Conditions

Conditions limit actions to specific object attributes:

```ts
can('update', 'Article', { authorId: user.id });
```

### ➤ `can(action, subject)`

Used to check if a user is allowed to perform an action.

```ts
ability.can('read', 'Article'); // true or false
ability.can('update', articleInstance); // subject as object
```

---

## ⚙️ Ability Builder

```ts
import { AbilityBuilder, Ability } from '@casl/ability';

const defineAbilitiesFor = (user) => {
  const { can, cannot, build } = new AbilityBuilder(Ability);

  if (user.role === 'admin') {
    can('manage', 'all'); // Full access
  } else {
    can('read', 'Article');
    can('update', 'Article', { authorId: user.id });
    cannot('delete', 'Article');
  }

  return build({
    detectSubjectType: item => item.__type || item.constructor.name,
  });
};
```

[CASL – AbilityBuilder Documentation](https://casl.js.org/v6/en/guide/intro#defining-abilities)

---

## 🧩 Special Keywords: `manage` and `all`

### `manage`

A wildcard **action** that covers all possible actions (read, update, delete, etc).

```ts
can('manage', 'Article'); // Equivalent to allowing all actions on Article
```

### `all`

A wildcard **subject** that represents any resource.

```ts
can('read', 'all'); // Can read any resource
can('manage', 'all'); // Can do anything
```

[CASL – Actions and Subjects](https://casl.js.org/v6/en/guide/intro#actions-subjects)

---

## 🧠 Object-Based Checks

You can pass a resource instance instead of a string subject:

```ts
const post = { title: 'Hello', authorId: 1, __type: 'Post' };
ability.can('update', post); // will check conditions
```

To enable that, use `detectSubjectType` when building the ability:

```ts
return build({
  detectSubjectType: item => item.__type || item.constructor.name,
});
```

---

## ⚙️ Conditions in Depth

You can use conditions like this:

```ts
can('update', 'Post', { authorId: user.id });
```

### Supported operators (MongoDB-style):

- `$eq`, `$ne`
- `$gt`, `$gte`, `$lt`, `$lte`
- `$in`, `$nin`
- `$and`, `$or`, `$nor`, `$regex`, `$exists`

Example:

```ts
can('read', 'Post', {
  status: 'published',
  views: { $gt: 1000 }
});
```

[CASL – Conditions](https://casl.js.org/v6/en/guide/conditions)

---

## 🧪 Evaluating Rules

```ts
const ability = defineAbilitiesFor(currentUser);

if (ability.can('read', 'Article')) {
  // allow rendering or access
}
```

Or with object:

```ts
ability.can('update', { title: '...', authorId: 1, __type: 'Post' });
```

---

## 🔄 Shared Auth Package (Frontend + Backend)

Recommended structure for a shared `auth` package:

```
auth/
├── abilities/
│   └── index.ts
├── types/
│   └── subjects.ts
├── rules/
│   └── user.rules.ts
├── index.ts
```

### `subjects.ts`

```ts
export type Subjects = 'Article' | 'User' | 'all';
```

### `user.rules.ts`

```ts
import { AbilityBuilder, AbilityClass, PureAbility } from '@casl/ability';
import { Subjects } from '../types/subjects';

type Actions = 'manage' | 'read' | 'create' | 'update' | 'delete';
export type AppAbility = PureAbility<[Actions, Subjects]>;

export const defineAbilitiesFor = (user: { id: number; role: string }): AppAbility => {
  const { can, cannot, build } = new AbilityBuilder<AppAbility>(PureAbility as AbilityClass<AppAbility>);

  if (user.role === 'admin') {
    can('manage', 'all');
  } else {
    can('read', 'Article');
    can('update', 'Article', { authorId: user.id });
  }

  return build({
    detectSubjectType: (item) => item?.__type || item?.constructor?.name,
  });
};
```

---

## ✅ Best Practices

- ✅ Centralize permission rules using `defineAbilitiesFor(user)`
- ✅ Always use `detectSubjectType` for object-based checks
- ✅ Reuse shared logic across backend and frontend
- ✅ Use `can()` and `cannot()` for explicit control
- ✅ Test your rules independently
- ❌ Don’t rely solely on frontend checks for security

---

## 📚 References

- [CASL Documentation](https://casl.js.org/)
- [API Reference](https://casl.js.org/v6/en/api/casl-ability)
- [React Integration](https://casl.js.org/v6/en/advanced/react)
- [Conditions Guide](https://casl.js.org/v6/en/guide/conditions)
- [Best Practices](https://casl.js.org/v6/en/guide/best-practices)
- [MongoDB Integration](https://casl.js.org/v6/en/advanced/mongo)
