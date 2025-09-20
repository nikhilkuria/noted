# Noted Application - Technical Documentation

## Current Implementation Analysis

The existing notes application is a vanilla JavaScript single-page application with the following characteristics:

### Architecture Overview
- **Frontend**: Pure HTML/CSS/JavaScript with no framework
- **Styling**: Tailwind CSS via CDN
- **Markdown**: Marked.js for markdown parsing
- **Backend**: n8n webhook endpoints for data persistence
- **State Management**: Local variables and localStorage caching

### Key Components

#### 1. API Layer
- Base URL: `https://nikhilkuriakose.app.n8n.cloud/webhook`
- Endpoints follow RESTful patterns
- Note: DELETE endpoint has a typo (`/notesd/{id}` instead of `/notes/{id}`)
- Implements caching with localStorage fallback

#### 2. Data Model
```javascript
{
  id: string,
  title: string,
  body_markdown: string,
  tags: string[],
  created_at?: string,
  updated_at?: string
}
```

#### 3. UI Components
- Modal-based note editor with split view (markdown/preview)
- Grid layout for note cards
- Search/filter functionality
- Responsive design with Tailwind classes

## Vue.js Migration Architecture

### Final Architecture Decisions

Based on the requirements, here are the finalized architectural decisions:

#### 1. Technology Stack
```
Core:
- Vue 3.5+ with Composition API
- Full TypeScript implementation
- Vite 5.x for build tooling
- vite-plugin-ssg for FULL static site generation

State & Routing:
- Pinia for state management
- Vue Router 4 with individual routes per note (/notes/:id)

Styling & UI:
- Tailwind CSS 3.x (PostCSS)
- Vuetify 3.x for UI components
- Minimalist design (maintaining current aesthetic)
- Desktop-first responsive approach

Testing:
- Vitest for unit tests only
- No E2E tests in initial implementation

Deployment:
- GitHub Pages hosting
- GitHub Actions for CI/CD
- Automatic deployment on push to main branch
```

#### 2. Component Architecture

```
src/components/
├── notes/
│   ├── NoteCard.vue         # Individual note display
│   ├── NoteGrid.vue         # Grid container
│   ├── NoteEditor.vue       # Markdown editor component
│   ├── NoteViewer.vue       # Read-only note display
│   └── NoteSearch.vue       # Search/filter component
├── common/
│   ├── BaseModal.vue        # Reusable modal wrapper
│   ├── BaseButton.vue       # Consistent button styling
│   ├── TagInput.vue         # Tag management component
│   └── MarkdownPreview.vue  # Markdown renderer
└── layout/
    ├── AppHeader.vue        # Application header
    └── AppLayout.vue        # Main layout wrapper
```

#### 3. State Management (Pinia)

```typescript
// stores/notes.ts
interface NotesState {
  notes: Note[]
  currentNote: Note | null
  searchQuery: string
  selectedTags: string[]
  isLoading: boolean
  error: string | null
}

// Actions
- fetchNotes()
- createNote(note: Partial<Note>)
- updateNote(id: string, updates: Partial<Note>)
- deleteNote(id: string)
- searchNotes(query: string)
- filterByTags(tags: string[])
```

#### 4. API Service Layer

```typescript
// src/api/notes.service.ts
class NotesAPI {
  private baseURL: string
  private cache: CacheManager

  async getAllNotes(): Promise<Note[]>
  async getNote(id: string): Promise<Note>
  async createNote(data: CreateNoteDTO): Promise<Note>
  async updateNote(id: string, data: UpdateNoteDTO): Promise<Note>
  async deleteNote(id: string): Promise<void>
}

// src/api/cache.manager.ts
class CacheManager {
  private storage: Storage
  private memory: Map<string, any>

  get<T>(key: string): T | null
  set<T>(key: string, value: T, ttl?: number): void
  invalidate(pattern?: string): void
}
```

#### 5. Composables

```typescript
// composables/useNotes.ts
- useNotesList()     // List management
- useNoteEditor()    // Editor state
- useMarkdown()      // Markdown parsing
- useSearch()        // Search functionality
- useTags()          // Tag management
- useLocalStorage()  // Persistent storage
```

#### 6. Static Site Generation Strategy (FULL SSG)

```typescript
// vite.config.ts
export default {
  base: '/noted/', // GitHub Pages repository name
  plugins: [
    vue(),
    ViteSsg({
      // Full static generation - ALL notes pre-rendered at build time
      async onBeforePageRender(route) {
        if (route.startsWith('/notes/')) {
          const noteId = route.split('/').pop()
          const note = await fetchNote(noteId)
          return { note }
        }
      },

      // Generate all routes at build time
      includedRoutes: async () => {
        const notes = await fetchAllNotes() // Fetch from API at build time
        return [
          '/',
          '/notes',
          ...notes.map(note => `/notes/${note.id}`) // Individual routes for SEO
        ]
      },

      // SSG Options
      mock: true,
      format: 'esm',
      script: 'async'
    })
  ]
}
```

### Migration Plan

#### Phase 1: Setup & Infrastructure
1. Initialize Vue project with Vite and TypeScript
2. Configure ESLint, Prettier
3. Setup Tailwind CSS and Vuetify 3
4. Configure vite-plugin-ssg for full static generation
5. Setup GitHub Actions workflow

#### Phase 2: Core Components
1. Implement UI components using Vuetify 3
2. Create note-specific components
3. Setup Pinia stores
4. Implement API service layer with n8n endpoints

#### Phase 3: Feature Parity (Current Implementation Only)
1. Note CRUD operations
2. Basic search functionality
3. Tag display (no filtering in v1)
4. Markdown editor with live preview
5. Local storage caching

#### Phase 4: Static Generation & Deployment
1. Configure full SSG with all routes
2. Setup GitHub Pages deployment
3. Configure base path for GitHub Pages
4. Test build process with real API data

#### Phase 5: Testing & Optimization
1. Unit tests for composables and stores
2. Ensure max 1000 notes performance
3. Build optimization for GitHub Pages
4. Verify automatic deployment pipeline

### Performance Optimizations

1. **Static Pre-rendering**: All notes rendered at build time
2. **Pagination**: Handle up to 1000 notes efficiently
3. **Debounced Search**: Client-side search optimization
4. **Build Optimizations**: Tree shaking, minification, compression
5. **Lazy Loading**: For markdown editor component
6. **Cache Strategy**: Browser caching for static assets

### Security Considerations

1. **Input Sanitization**: Sanitize markdown input
2. **XSS Prevention**: Use Vue's built-in protection
3. **API Security**: Implement proper CORS, rate limiting
4. **Environment Variables**: Never expose sensitive data
5. **Content Security Policy**: Configure CSP headers

### Development Workflow

```bash
# Development
npm run dev           # Start dev server
npm run build        # Build static site with SSG
npm run preview      # Preview production build

# Quality
npm run lint         # Lint code
npm run lint:fix     # Auto-fix linting issues
npm run type-check   # TypeScript checking
npm run test         # Run unit tests
npm run test:ui      # Run tests with UI
npm run test:coverage # Test coverage report

# Deployment (Automatic)
git push origin main  # Triggers GitHub Actions deployment
```

### Folder Structure Details

```
noted/
├── src/
│   ├── api/
│   │   ├── client.ts           # Axios/fetch instance
│   │   ├── notes.service.ts    # Notes API
│   │   └── cache.manager.ts    # Cache management
│   ├── components/
│   │   ├── notes/              # Note-specific components
│   │   ├── common/             # Shared components
│   │   └── layout/             # Layout components
│   ├── composables/
│   │   ├── useNotes.ts        # Notes logic
│   │   ├── useMarkdown.ts     # Markdown handling
│   │   └── useSearch.ts       # Search logic
│   ├── pages/
│   │   ├── HomePage.vue       # Landing page
│   │   ├── NotesPage.vue      # Notes list
│   │   └── NoteDetail.vue     # Single note view
│   ├── router/
│   │   └── index.ts           # Router configuration
│   ├── stores/
│   │   ├── notes.ts           # Notes store
│   │   └── ui.ts              # UI state store
│   ├── styles/
│   │   ├── main.css           # Global styles
│   │   └── markdown.css       # Markdown styles
│   ├── types/
│   │   ├── note.ts            # Note interfaces
│   │   └── api.ts             # API types
│   ├── utils/
│   │   ├── date.ts            # Date utilities
│   │   └── markdown.ts        # Markdown utilities
│   ├── App.vue
│   └── main.ts
├── public/
│   └── assets/                 # Static assets
├── tests/
│   ├── unit/                   # Unit tests
│   ├── component/              # Component tests
│   └── e2e/                    # E2E tests (future)
├── .github/
│   └── workflows/              # GitHub Actions
├── vite.config.ts              # Vite configuration
├── tsconfig.json               # TypeScript config
├── tailwind.config.js          # Tailwind config
├── vuetify.config.ts           # Vuetify 3 theme config
└── package.json                # Dependencies
```

### Environment Variables

```env
# API Configuration (n8n endpoints maintained)
VITE_API_BASE_URL=https://nikhilkuriakose.app.n8n.cloud/webhook
VITE_API_TIMEOUT=10000
VITE_API_RETRY_ATTEMPTS=3

# GitHub Pages Configuration
VITE_PUBLIC_PATH=/noted/
VITE_BASE_URL=https://[username].github.io/noted/

# Build Configuration
VITE_BUILD_MODE=ssg
VITE_SSG_FETCH_AT_BUILD=true
```

### Commands and Scripts

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc --noEmit && vite-ssg build",
    "preview": "vite preview",
    "lint": "eslint src --ext .vue,.ts",
    "lint:fix": "eslint src --ext .vue,.ts --fix",
    "type-check": "vue-tsc --noEmit",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "clean": "rimraf dist node_modules/.vite"
  }
}
```

### Deployment Configuration

#### GitHub Pages Deployment
1. **Hosting**: GitHub Pages only
2. **URL**: `https://[username].github.io/noted/`
3. **CI/CD**: GitHub Actions automatic deployment on push to main
4. **Build Process**: Full SSG with all notes pre-rendered
5. **Base Path**: Configured as `/noted/` for GitHub Pages

#### GitHub Actions Workflow
- Triggers on push to main branch
- Builds static site with vite-ssg
- Deploys dist folder to gh-pages branch
- No manual intervention required

### Known Challenges & Solutions

1. **API Endpoint Typo**: Fix DELETE endpoint path in migration (`/notesd/{id}` → `/notes/{id}`)
2. **CORS Issues**: May need proxy configuration during development
3. **1000 Notes Limit**: Implement pagination for note grid
4. **Build Time**: API calls during SSG may increase build time
5. **SEO**: Individual note routes enable proper SEO and sharing

### Key Implementation Notes

1. **Single User Focus**: No authentication or multi-user features
2. **Desktop First**: Optimize for desktop, ensure mobile compatibility
3. **Minimalist Design**: Maintain current clean aesthetic with Vuetify
4. **Feature Parity**: Only implement current features, no new additions
5. **Full Static**: Every note has its own static HTML page
6. **UI Library**: Vuetify 3 for consistent Material Design components

### Vuetify 3 Integration

#### Theme Configuration
```typescript
// vuetify.config.ts
export default createVuetify({
  theme: {
    defaultTheme: 'light',
    themes: {
      light: {
        colors: {
          primary: '#1976D2',
          secondary: '#424242',
          accent: '#82B1FF',
          error: '#FF5252',
          info: '#2196F3',
          success: '#4CAF50',
          warning: '#FFC107'
        }
      }
    }
  },
  defaults: {
    VCard: {
      elevation: 2,
      rounded: 'lg'
    },
    VBtn: {
      variant: 'flat',
      rounded: 'lg'
    }
  }
})
```

#### Key Vuetify Components to Use
- `v-app`: Main application wrapper
- `v-app-bar`: Header with search and add button
- `v-card`: Note cards in grid
- `v-dialog`: Modal for note editor/viewer
- `v-text-field`: Input fields
- `v-textarea`: Markdown editor
- `v-chip`: Tags display
- `v-btn`: Action buttons
- `v-grid`: Responsive layout system
- `v-skeleton-loader`: Loading states