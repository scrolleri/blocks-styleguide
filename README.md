# Scrolleri JavaScript Style Guide

_A mostly reasonable approach to Frontify Blocks, JavaScript, and TypeScript._

## Git

We have one main branch and we open pull requets for changes to the blocks.

## Branch naming

An example of a good branch naming `[category]/[block-name]-[objective]` or if it's more general `[category]/[objective]`

Categories:

* chore
* fix
* feature
* renovate
* poc

## State management

We have been using Recoil for state management but are considering evaluating XState to determine which works better for our use cases.

The key reason for considering XState is that Recoil team has stopped developing it. Additionally, XState's useSelector hook can help avoid unnecessary re-renders.

## Folder naming

File names should be in kebab-case and include a `.[context]` suffix if necessary.

Example:

```md
- card/
  - card.tsx
  - card.module.css
  - elements/
    - card-header.tsx
    - card-body.tsx
- contexts/
  - app.context.ts
  - app.context.provider.tsx
```

## Folder structure

```md
- packages
  - shared
    - components
    - utils
      - number.ts
      - dom.ts
      - string.ts
  
  - pdf-export/
    - block/
      - pdf-export.block.ts
      - pdf-export.hooks.ts
      - pdf-export.settings.ts
      - pdf-export.ts
    - components/
      - ui/
      - overlays/
      - elements/
      - layouts/
      - project specific general folders
    - hooks/
      - use-id.ts
    - utils/
      - pdf-export.utils.ts
      - constants.ts
    - types/
      - settings.ts
      - definitions.ts
    index.ts
```
  
## Variable naming conventions

Use meaningful variable names, with lowerCamelCase for regular variables and UPPER_SNAKE_CASE for constants.

```ts
const bucket = []
bucket.push({ id: 1 }) // it's still a variable

let queue = []
queue = [{ id: '2' }]
```

### Primitives

```ts
const API_HOST = 'https://example.com/api'
```

Single-word local variables can be lowercase or uppercase:

```ts
const PI = 3.14
const pi = 3.14
````

Literal Constant Objects

```ts
const HttpStatus = {
  NotFound: 404,
  Unauthorized: 401
} as const
```

## TypeScript

### Union Types

```ts
type Nullable<T> = T | null
```

Using type of type or enum is the developer decision based on their feeling about the use case.

### Type suffix

You can use the Type suffix for one-line field types.

```ts
type UserType = "user" | "admin"
```

But some cases you can ignore writing Type.

```ts
type Actions =
  | {type: 'add'}
  | {type: 'remove'}
  | {type: 'update'; payload: Data}
```

### Constant Enums

```ts
const enum ArticleTypeKind {
  Post = 'post',
  Article = 'article'
}
```

The choice between using a union type or an enum is up to the developer, depending on the use case.

### API Services

```ts
// shared/api/services/article-service

// types.ts
interface Article {
  id: string
  type: ArticleTypeKind
}

// dto.ts
interface ArticleCreateDto {
  name: string
  slug: string
}

// article-service.ts
class ArticleService {
  static async findAll() {
    const {data} = await api.get<Article[]>('articles')
    return data
  }
  static async findOne(id: string) {
    const {data} = await api.get<Article>(`articles/${id}`)
    return data
  }
  static async create(input: ArticleCreateDto) {
    const {data} = await api.post<Article>('articles', input)
    return data
  }
}
```

## Block settings

General interfaces should have a Shape suffix, particularly when representing a specific data structure.

```ts
// block-name/types/definitions.ts
const enum MobileStyleKind {
  Grid = 'grid',
  Compact = 'compact',
}

// block-name/types/settings.ts
interface SidebarSettings {
  mobileStyle: MobileStyleKind
}

export interface SettingsShape extends SidebarSettings {
  tiles: TileShape[]
}
```

## React

Use function components for React, and const () => {} syntax for helper functions.

```tsx
interface UserCardProps {
  children: React.ReactNode
  user: User
}

function UserCard({ user }: UserCardProps) {
  return <div>...</div>
}
```

## A collection of project specific helper functions

you can use a class with static methods same as ArticleService to have the related utility functions under one namespace.
