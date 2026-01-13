# AGENTS.md - Development Guide for linkTalk AAC App

This guide is designed for AI coding agents working on the linkTalk AAC (Augmentative and Alternative Communication) application.

## Project Overview

linkTalk is a Hungarian-language AAC communication app built as a single-page web application. It helps users with communication difficulties build sentences using visual word buttons with speech synthesis and ARASAAC pictograms.

**Technology Stack:**
- Vanilla HTML5/CSS3/JavaScript (ES6+)
- Tailwind CSS (CDN)
- Web Speech API
- ARASAAC API for pictograms
- Single-file architecture (`index.html`)

## Build/Test/Lint Commands

This project uses a **single-file architecture** with no build system.

### Development
```bash
# No build step required - direct browser execution
open index.html

# For local development server (recommended for CORS/API testing)
python -m http.server 8000
# or
npx serve .

# Access at http://localhost:8000
```

### Testing
- **No automated testing framework** - relies on manual browser testing
- **Single test**: Open `index.html` in browser after changes
- **API testing**: Use development server to avoid CORS issues
- **Browser compatibility**: Test in modern browsers with Web Speech API support

### Linting/Formatting
- **No linting tools** - rely on browser developer tools for error detection
- **Validation**: Use browser console to check for JavaScript errors
- **API monitoring**: Check Network tab for ARASAAC API call success/failure

## Code Style Guidelines

### File Structure
```
/Users/attila/verbal-app/
├── .git/           # Git repository
└── index.html      # Complete application (HTML + CSS + JS)
```

### JavaScript Guidelines

#### Code Organization
- **ES6 Classes**: `CacheManager`, `ArasaacAPI`, `PictogramSearch`
- **Function declarations**: For main UI functions (`init()`, `loadCategory()`, `speakWord()`)
- **Event listeners**: Centralized in `setupEventListeners()`
- **Initialization**: Single `init()` function called on DOMContentLoaded
- **Async/Await**: Preferred over Promises for API calls

#### Naming Conventions
- **camelCase**: Variables and functions (`currentCategory`, `speakWord`)
- **PascalCase**: Classes (`ArasaacAPI`, `PictogramSearch`)
- **UPPER_CASE**: Constants (`ARASAAC_API.baseUrl`)
- **kebab-case**: CSS classes and HTML IDs

#### Error Handling Pattern
```javascript
async function apiFunction(parameter) {
    try {
        const cached = CacheManager.getCache(`key_${parameter}`);
        if (cached) return cached;
        
        const response = await fetch(url);
        if (!response.ok) throw new Error(`API error: ${response.status}`);
        
        const data = await response.json();
        CacheManager.setCache(`key_${parameter}`, data);
        return data;
    } catch (error) {
        console.warn('API call failed:', error);
        return fallbackValue; // Always provide fallback
    }
}
```

#### Import/Export
- **No modules**: All code in single file scope
- **No imports**: All dependencies via CDN
- **Global variables**: Minimal global state (`currentCategory`, `sentence`)

### CSS Guidelines
- **Tailwind-first**: Use utility classes extensively
- **Custom CSS**: Only for AAC-specific features not covered by Tailwind
- **Fitzgerald Key Colors**: Grammatical type color coding system
  - Nouns: Yellow (`#FFD700`)
  - Verbs: Green (`#32CD32`) 
  - Adjectives: Blue (`#4169E1`)
  - Social: Pink (`#FF69B4`)

### Data Structures

#### Vocabulary Object
```javascript
{
    id: 'unique-identifier',           // String identifier
    label: 'hungarian-word',           // Display text
    category: 'core|food|activities|feelings',
    grammaticalType: 'noun|verb|adjective|social',
    pictogramId: 2462                  // ARASAAC ID
}
```

#### State Management
- **Minimal global state**: `currentCategory`, `sentence` array
- **Caching**: Map objects for in-memory, localStorage for persistence
- **Immutability**: Create new arrays instead of mutating

## API Integration

### ARASAAC Endpoints
- `GET /pictograms/hu/bestsearch/{term}` - Search Hungarian pictograms
- `GET /keywords/hu` - Load keywords for autocomplete
- Static images: `https://static.arasaac.org/pictograms/{id}/{id}_{resolution}.png`

### Image Loading Strategy
1. **Direct ARASAAC ID**: Try original pictogram ID with multiple resolutions
2. **Auto-search fallback**: Search ARASAAC API if original fails
3. **Text placeholder**: Colored fallback as final option

### Caching Strategy
- Search results: 1 hour
- Keywords: 7 days
- Pictogram metadata: 24 hours
- Auto-cleanup on app start

## Key Functions Reference

### Core Functions
- `init()` - App initialization (index.html:722)
- `setupEventListeners()` - Event binding (index.html:739)
- `loadCategory(category)` - Load vocabulary category (index.html:797)
- `createCommunicationButton(word)` - Create word button (index.html:810)

### Sentence Management
- `addToSentence(word)` - Add word to sentence (index.html:960)
- `removeFromSentence(index)` - Remove word at index (index.html:1049)
- `clearSentence()` - Clear entire sentence (index.html:1055)
- `updateSentenceDisplay()` - Refresh sentence UI (index.html:966)

### Speech & Audio
- `speakWord(text)` - Text-to-speech for single word (index.html:1061)
- `speakSentence()` - Text-to-speech for full sentence (index.html:1079)

### Classes
- `CacheManager` - Handle localStorage/memory caching (index.html:216)
- `ArasaacAPI` - ARASAAC API integration (index.html:286)
- `PictogramSearch` - Search functionality with debouncing (index.html:412)

## Development Workflow

1. **Edit**: Make changes directly in `index.html`
2. **Test**: Refresh browser, use local server for API testing
3. **Debug**: Browser dev tools, monitor Network tab for API calls
4. **Commit**: Use git for version control
5. **Deploy**: Upload single HTML file to web server

## Testing Checklist

Essential tests for manual verification:
- [ ] Speech synthesis works in target browsers
- [ ] ARASAAC API integration and fallbacks work
- [ ] Search returns relevant Hungarian results
- [ ] Image fallback system handles failures gracefully
- [ ] Vocabulary categories load with pictograms
- [ ] Sentence building/clearing functions work
- [ ] Keyboard shortcuts (Enter, Space, 1-4, Delete, Backspace)
- [ ] Responsive design on different screen sizes
- [ ] Hungarian character display
- [ ] Accessibility (high contrast, touch targets 44px+)

## Common Patterns

### Adding New Vocabulary
```javascript
// Add to appropriate category in vocabulary object
{
    id: 'new-word',
    label: 'új szó',
    category: 'core',
    grammaticalType: 'noun',
    pictogramId: 2462  // Use actual ARASAAC ID
}
```

### Accessibility Requirements
- **Touch targets**: Minimum 44px for mobile
- **Color coding**: Follow Fitzgerald Key system
- **Keyboard navigation**: Support all essential shortcuts
- **Screen readers**: Use semantic HTML and ARIA where needed
- **High contrast**: Ensure readability on all backgrounds

This guide ensures consistent development practices for AAC applications with ARASAAC integration.