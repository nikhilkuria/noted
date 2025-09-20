# Noted - Modern Notes Application

A Vue.js-based static notes application with markdown support, built with Vite and designed for static site generation. Fully static deployment on GitHub Pages with automatic builds via GitHub Actions.

## Overview

Noted is a reimplementation of a static notes application, migrating from vanilla JavaScript to Vue.js with Static Site Generation (SSG) capabilities. The application provides a clean, minimalist interface for creating, managing, and organizing notes with full markdown support. Built as a single-user application with desktop-first responsive design.

## Features

### Core Functionality
- **Create and Edit Notes**: Full markdown editor with live preview
- **Tag System**: Organize notes with customizable tags
- **Search & Filter**: Quick search across all notes
- **Responsive Design**: Optimized for desktop, tablet, and mobile devices
- **API Integration**: Backend-agnostic design with configurable API endpoints

### Technical Features
- **Static Site Generation**: Pre-rendered pages for optimal performance
- **Vue 3 Composition API**: Modern, reactive architecture
- **Vite Build System**: Fast development and optimized production builds
- **Markdown Support**: Full markdown rendering with syntax highlighting
- **TypeScript Support**: Type-safe development experience

## Tech Stack

- **Framework**: Vue.js 3.x with TypeScript
- **Build Tool**: Vite 5.x
- **Styling**: Tailwind CSS 3.x + Vuetify 3.x
- **UI Components**: Vuetify 3.x
- **Markdown**: marked.js / markdown-it
- **State Management**: Pinia (Vue's official state management)
- **Routing**: Vue Router 4.x (with individual routes per note)
- **HTTP Client**: Axios / Native Fetch API
- **SSG**: vite-plugin-ssg for full static generation
- **Deployment**: GitHub Pages with GitHub Actions CI/CD
- **Testing**: Vitest for unit testing

## Project Structure

```
noted/
├── src/
│   ├── api/              # API service layer
│   ├── assets/           # Static assets
│   ├── components/       # Vue components
│   │   ├── common/       # Shared components
│   │   ├── notes/        # Note-specific components
│   │   └── layout/       # Layout components
│   ├── composables/      # Vue composables
│   ├── pages/            # Page components
│   ├── router/           # Vue Router configuration
│   ├── stores/           # Pinia stores
│   ├── styles/           # Global styles
│   ├── types/            # TypeScript types
│   ├── utils/            # Utility functions
│   ├── App.vue           # Root component
│   └── main.ts           # Application entry point
├── public/               # Public assets
├── dist/                 # Build output (deployed to GitHub Pages)
├── .github/
│   └── workflows/        # GitHub Actions workflows
│       └── deploy.yml    # Automatic deployment to GitHub Pages
├── .env                  # Environment variables
├── vite.config.ts        # Vite configuration with SSG setup
├── tsconfig.json         # TypeScript configuration
├── tailwind.config.js    # Tailwind configuration
└── package.json          # Dependencies and scripts
```

## Getting Started

### Prerequisites
- Node.js 18.x or higher
- npm 9.x or pnpm 8.x

### Installation

```bash
# Clone the repository
git clone https://github.com/nikhilkuria/noted.git
cd noted

# Install dependencies
npm install

# Copy environment variables
cp .env.example .env

# Configure your API endpoint in .env
```

### Development

```bash
# Start development server
npm run dev

# Start with API mocking (if implemented)
npm run dev:mock
```

### Building for Production

```bash
# Build static site with SSG
npm run build

# Preview production build locally
npm run preview

# Deploy to GitHub Pages (automatic via GitHub Actions on push to main)
git push origin main
```

### GitHub Pages Deployment

The application automatically deploys to GitHub Pages when you push to the `main` branch. The deployment process:

1. GitHub Actions workflow triggers on push to `main`
2. Builds the static site with all notes pre-rendered
3. Deploys the `dist` folder to GitHub Pages
4. Site available at: `https://[username].github.io/noted/`

To set up GitHub Pages:
1. Go to Settings → Pages in your GitHub repository
2. Set source to "GitHub Actions"
3. The workflow will handle the deployment automatically

## Configuration

### API Configuration

Configure your API endpoints in `.env`:

```env
VITE_API_BASE_URL=https://nikhilkuriakose.app.n8n.cloud/webhook
VITE_API_TIMEOUT=10000
VITE_PUBLIC_PATH=/noted/
```

### Build Configuration

Customize the build in `vite.config.ts`:

```typescript
export default defineConfig({
  base: '/noted/', // GitHub Pages base path
  ssgOptions: {
    script: 'async',
    formatting: 'minify',
    mock: true, // Pre-render with mock data during build
    includedRoutes: async () => {
      // Fetch all notes and generate routes
      const notes = await fetchNotes()
      return [
        '/',
        '/notes',
        ...notes.map(note => `/notes/${note.id}`)
      ]
    }
  }
})
```

## API Integration

The application expects a REST API with the following endpoints:

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/notes` | Fetch all notes |
| GET | `/notes/{id}` | Fetch single note |
| POST | `/notes` | Create new note |
| PUT | `/notes/{id}` | Update existing note |
| DELETE | `/notes/{id}` | Delete note |

### Note Data Structure

```typescript
interface Note {
  id: string;
  title: string;
  body_markdown: string;
  tags: string[];
  created_at?: string;
  updated_at?: string;
}
```

## Development Guidelines

### Component Structure
- Use Vue 3 Composition API with `<script setup>`
- Implement proper TypeScript typing
- Follow single responsibility principle
- Use composables for shared logic

### State Management
- Use Pinia for global state
- Keep component state local when possible
- Implement proper error handling

### Performance
- Leverage Vue's lazy loading for routes
- Optimize images and assets
- Implement virtual scrolling for large lists
- Use proper caching strategies

## Scripts

```json
{
  "dev": "vite",
  "build": "vite-ssg build",
  "preview": "vite preview",
  "lint": "eslint src --ext .vue,.ts",
  "lint:fix": "eslint src --ext .vue,.ts --fix",
  "type-check": "vue-tsc --noEmit",
  "test": "vitest",
  "test:ui": "vitest --ui",
  "test:coverage": "vitest --coverage"
}
```

## Browser Support

- Chrome/Edge (latest 2 versions)
- Firefox (latest 2 versions)
- Safari 14+
- Mobile browsers (iOS Safari, Chrome Mobile)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

## Future Improvements

These features are planned for future releases:

### Enhanced Features
- **Offline Support**: Local storage with sync capabilities when back online
- **Dark Mode**: Theme switching support for better accessibility
- **Tag Filtering**: Advanced filtering by multiple tags
- **Export/Import**: Backup and restore notes functionality
- **Rich Text Editor**: Alternative to markdown for non-technical users
- **Note Templates**: Pre-defined templates for common note types
- **Note History**: Version control and revision history

### Technical Enhancements
- **PWA Support**: Make the app installable and work offline
- **Real-time Sync**: WebSocket support for instant updates
- **Full-text Search**: Server-side search for better performance
- **Image Optimization**: Automatic image compression and lazy loading
- **Accessibility**: WCAG 2.1 Level AA compliance
- **Mobile Optimization**: Mobile-first redesign
- **E2E Testing**: Comprehensive end-to-end test coverage

### Performance
- **Virtual Scrolling**: For handling large note collections (1000+)
- **Code Splitting**: Dynamic imports for faster initial load
- **Service Workers**: Advanced caching strategies
- **CDN Integration**: Static asset delivery optimization

## License

MIT License - see LICENSE file for details

## Acknowledgments

- Original notes application: [github.com/nikhilkuria/notes](https://github.com/nikhilkuria/notes)
- Built with Vue.js and Vite
- Styled with Tailwind CSS
- Deployed on GitHub Pages