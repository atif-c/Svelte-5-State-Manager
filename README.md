# Svelte 5 State Manager

A lightweight, generic state management class for Svelte 5 projects that **bridges in-memory reactive state with persistent storage**. Work with Svelte 5 `$state` runes, letting you sync with your chosen storage backend (localStorage, APIs, databases, etc.).

## Features

-   **Reactive State**: Built on Svelte 5's `$state` rune for automatic reactivity
-   **Configurable Debouncing**: Customizable timing with immediate execution support
-   **Storage Agnostic**: Works with any storage backend (localStorage, IndexedDB, API endpoints, etc.)
-   **Type Safe**: Full TypeScript support with generic type constraints
-   **Deep Cloning**: Prevents reference mutations between state and storage

## Installation

Copy the `StateManager.ts` file into your Svelte 5 project.

## Quick Start

```typescript
import { StateManager } from './StateManager';

interface AppSettings {
    theme: 'light' | 'dark';
    language: string;
    notifications: boolean;
}

// Create a state manager that syncs in-memory state with localStorage
const settingsManager = new StateManager<AppSettings>(
    // Load function - retrieve from persistent storage → memory
    async () => {
        const saved = localStorage.getItem('app-settings');
        return saved
            ? JSON.parse(saved)
            : {
                  theme: 'light',
                  language: 'en',
                  notifications: true,
              };
    },
    // Save function - persist memory changes → storage
    async data => {
        localStorage.setItem('app-settings', JSON.stringify(data));
    },
    // Debounce options - optimize storage writes
    { delay: 500, maxWait: 2000 }
);

// Initialize: Load from persistent storage into reactive memory
await settingsManager.load();

// Export state
export let settings = settingsManager.state;
```

## Usage in Svelte Components

```typescript
<script lang="ts">
  import { settingsManager } from './stores/settings';

  // The state is reactive in-memory - changes instantly update the UI
  let { state } = settingsManager;

  function toggleTheme() {
    state.theme = state.theme === 'light' ? 'dark' : 'light';
  }
</script>

<button on:click={toggleTheme}>
  Current theme: {state.theme} <!-- Updates instantly -->
</button>

<label>
  <input
    type="checkbox"
    bind:checked={state.notifications}
  />
  Enable notifications
</label>
```

**How it works:**

1. **Load**: Storage → Memory (on initialization)
2. **Mutate**: Direct in-memory changes (instant UI updates)
3. **Sync**: Memory → Storage (debounced)

## API Reference

### Constructor

```typescript
new StateManager<T>(
  loadCallback: () => Promise<T> | T,
  saveCallback?: (state: T) => Promise<void> | void,
  debounceOptions?: DebounceOptions
)
```

#### Parameters

-   **`loadCallback`**: Function to load initial state data
    -   Can be synchronous or asynchronous
    -   Should return the complete state object
-   **`saveCallback`** _(optional)_: Function to persist state changes
    -   Receives a deep clone of current state
    -   Can be synchronous or asynchronous
-   **`debounceOptions`** _(optional)_: Configuration for save debouncing
    -   `delay`: Milliseconds to wait after last change (default: 0)
    -   `maxWait`: Maximum milliseconds before forcing save (default: 0)
    -   `immediate`: Execute save immediately on first change (default: false)

### Properties

-   **`state`**: The reactive state object (Svelte 5 `$state`)

### Methods

-   **`load()`**: Loads state from persistent storage into reactive memory
-   **`save()`**: Manually triggers a sync from memory to persistent storage

## Important Notes

### Type Constraints

-   State type `T` must be structured-cloneable/serializable
-   Functions, DOM nodes, and certain class instances will not work properly
-   Use plain objects, arrays, primitives, and serializable data only

### Reactivity Best Practices

-   Mutate state fields directly: `manager.state.theme = 'dark'`
-   Avoid replacing the entire state object
-   Use `$state.snapshot(manager.state)` to get a non-reactive copy for external use

### Error Handling

-   Load errors are thrown and logged to console
-   Save errors are logged but may be thrown asynchronously (with debouncing)
-   Always wrap `load()` calls in try-catch blocks

## Advanced Configuration

### Custom Debouncing

```typescript
const manager = new StateManager(loadFn, saveFn, {
    delay: 300, // Wait 300ms after last change
    maxWait: 2000, // Force save after 2 seconds maximum
    immediate: true, // Save immediately on first change
});
```

### Multiple State Managers

```typescript
// Separate managers for different concerns
const userSettings = new StateManager(loadUserSettings, saveUserSettings);
const appCache = new StateManager(loadCache, saveCache, { delay: 100 });
const gameState = new StateManager(loadGame, saveGame, { immediate: true });
```
