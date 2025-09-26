# JavaScript Style Guide

_A mostly reasonable approach to Frontify Blocks, JavaScript, and TypeScript._

## Git

We follow linear git branching with a single main branch. All changes are made through pull requests.

## Branch naming

Format: `[type]/[description]`

Examples:

- `feat/add-dark-mode`
- `fix/grid-alignment`
- `chore/update-dependencies`

## State management

We use Zustand for state management. Zustand provides a lightweight, TypeScript-friendly solution with minimal boilerplate and excellent performance characteristics.

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

### Standard Block Structure

```md
- packages/
  - shared/
    - components/
    - utils/
      - number.ts
      - dom.ts
      - string.ts

  - [block-name]/
    - src/
      - block/
        - [block-name].block.tsx        # Main block component
        - [block-name].tsx               # Block logic
        - [block-name].module.css        # Block styles
      - components/                      # UI components
        - ui/                           # Generic UI components
        - overlays/                     # Modal/overlay components
        - elements/                     # Basic building blocks
        - layouts/                      # Layout components
        - [feature-name]/               # Feature-specific components
          - [component].tsx
          - [component].module.css
          - index.ts
      - contexts/                       # React contexts for state sharing
        - [context-name].context.ts
        - [context-name].context-provider.tsx
      - hooks/                          # Custom React hooks
        - use-[functionality].ts
      - settings/                       # Block configuration
        - settings.ts                   # Settings definition
        - settings.hooks.ts             # Settings-related hooks
        - types.ts                      # Settings types
        - enums.ts                      # Settings enums
        - index.ts                      # Exports
      - stores/                         # State management (Zustand)
        - [feature].store.ts
      - types/                          # TypeScript definitions
        - definitions.ts                # General type definitions
        - enum.ts                       # Shared enums
        - index.ts                      # Type exports
      - utils/                          # Utility functions
        - [domain]-utils.ts             # Domain-specific utilities
        - constants.ts                  # Constants
        - defaults.ts                   # Default values
        - enums.ts                      # Utility enums
      - index.ts                        # Main export
      - global.d.ts                     # Global type declarations
```

### Directory Purpose Guide

| Directory | Purpose | Examples |
|-----------|---------|----------|
| `block/` | Core block implementation | Main component, settings, exports |
| `components/` | Reusable UI components | Buttons, cards, modals, forms |
| `contexts/` | React context providers | Theme context, user context |
| `hooks/` | Custom React hooks | `useBreakpoint`, `useGridConfig` |
| `settings/` | Block configuration & settings | Settings schema, validations |
| `stores/` | State management | Zustand stores |
| `types/` | TypeScript type definitions | Interfaces, types, enums |
| `utils/` | Helper functions & utilities | Formatters, validators, calculations |

### Examples

#### PDF Export Block

```md
- pdf-export/
  - src/
    - block/
      - pdf-export.block.tsx
      - pdf-export.hooks.ts
      - pdf-export.settings.ts
    - components/
      - export-button/
      - preview-modal/
    - utils/
      - pdf-generation.ts
      - export-config.ts
```

#### Layout Grid Block

```md
- layout-grid/
  - src/
    - block/
      - layout-grid.block.tsx
    - components/
      - tile/
      - grid-background/
      - breakpoint-toolbar/
    - hooks/
      - use-grid-config.ts
      - use-breakpoint.ts
    - stores/
      - tiles.store.ts
```

#### Gallery Block

```md
- gallery/
  - src/
    - block/
      - gallery.block.tsx
    - components/
      - image-viewer/
      - thumbnail-grid/
      - lightbox/
    - hooks/
      - use-image-loader.ts
    - utils/
      - image-optimization.ts
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

Using type or enum is the developer's decision based on their feeling about the use case.

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
// Generic API client with better error handling and features
class ApiClient {
  private abortControllers = new Map<string, AbortController>()

  async request<T>(
    method: string,
    endpoint: string,
    options?: {
      params?: Record<string, any>
      body?: any
      signal?: AbortSignal
      retries?: number
      cache?: boolean
    }
  ): Promise<T> {
    const retries = options?.retries ?? 0

    try {
      const response = await api[method.toLowerCase()](endpoint, {
        ...options?.body,
        params: options?.params,
        signal: options?.signal
      })

      return response.data
    } catch (error) {
      if (retries > 0 && this.isRetryable(error)) {
        await this.delay(1000 * (3 - retries)) // exponential backoff
        return this.request(method, endpoint, { ...options, retries: retries - 1 })
      }
      throw this.transformError(error)
    }
  }

  private isRetryable(error: any): boolean {
    return error.response?.status >= 500 || error.code === 'ECONNABORTED'
  }

  private transformError(error: any): Error {
    if (error.response?.status === 404) {
      return new NotFoundError(error.response.data.message)
    }
    if (error.response?.status === 401) {
      return new UnauthorizedError()
    }
    return new ApiError(error.message, error.response?.status)
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
  }

  cancelRequest(key: string) {
    this.abortControllers.get(key)?.abort()
    this.abortControllers.delete(key)
  }
}

// Custom error classes
class ApiError extends Error {
  constructor(message: string, public statusCode?: number) {
    super(message)
  }
}
class NotFoundError extends ApiError {}
class UnauthorizedError extends ApiError {}

// Generic resource factory
function createResourceService<T, CreateDto = Partial<T>, UpdateDto = Partial<T>>(
  resource: string,
  client = new ApiClient()
) {
  return {
    async findAll(params?: Record<string, any>): Promise<T[]> {
      return client.request<T[]>('GET', resource, { params, retries: 2 })
    },

    async findOne(id: string | number): Promise<T> {
      return client.request<T>('GET', `${resource}/${id}`, { retries: 2 })
    },

    async create(data: CreateDto): Promise<T> {
      return client.request<T>('POST', resource, { body: data })
    },

    async update(id: string | number, data: UpdateDto): Promise<T> {
      return client.request<T>('PATCH', `${resource}/${id}`, { body: data })
    },

    async delete(id: string | number): Promise<void> {
      return client.request<void>('DELETE', `${resource}/${id}`)
    },

    // Additional useful methods
    async findByIds(ids: (string | number)[]): Promise<T[]> {
      return Promise.all(ids.map(id => this.findOne(id)))
    },

    async exists(id: string | number): Promise<boolean> {
      try {
        await this.findOne(id)
        return true
      } catch (error) {
        if (error instanceof NotFoundError) return false
        throw error
      }
    }
  }
}

// Usage - much cleaner and more powerful
const ArticleService = createResourceService<Article, ArticleCreateDto>('articles')

// Or with custom methods if needed
const EnhancedArticleService = {
  ...createResourceService<Article, ArticleCreateDto>('articles'),

  async findByAuthor(authorId: string): Promise<Article[]> {
    return new ApiClient().request<Article[]>('GET', 'articles', {
      params: { authorId },
      retries: 2
    })
  },

  async publish(id: string): Promise<Article> {
    return new ApiClient().request<Article>('POST', `articles/${id}/publish`)
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
