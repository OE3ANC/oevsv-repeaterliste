<script lang="ts">
  let { theme = 'system', onToggle } = $props<{
    theme?: 'light' | 'dark' | 'system';
    onToggle: (t: 'light' | 'dark' | 'system') => void;
  }>();

  const next: Record<string, 'light' | 'dark' | 'system'> = {
    system: 'light',
    light: 'dark',
    dark: 'system',
  };

  const labels: Record<string, string> = {
    system: 'Auto',
    light: 'Hell',
    dark: 'Dunkel',
  };

  function cycle() {
    onToggle(next[theme]);
  }
</script>

<button class="theme-toggle" onclick={cycle} title={labels[theme]} aria-label={`Theme: ${labels[theme]}`}>
  {#if theme === 'system'}
    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <rect x="2" y="3" width="20" height="14" rx="2" ry="2"/>
      <line x1="8" y1="21" x2="16" y2="21"/>
      <line x1="12" y1="17" x2="12" y2="21"/>
    </svg>
  {:else if theme === 'light'}
    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <circle cx="12" cy="12" r="5"/>
      <line x1="12" y1="1" x2="12" y2="3"/>
      <line x1="12" y1="21" x2="12" y2="23"/>
      <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/>
      <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/>
      <line x1="1" y1="12" x2="3" y2="12"/>
      <line x1="21" y1="12" x2="23" y2="12"/>
      <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/>
      <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/>
    </svg>
  {:else}
    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/>
    </svg>
  {/if}
</button>

<style>
  .theme-toggle {
    position: fixed;
    top: 1rem;
    right: 1rem;
    background: none;
    border: 1px solid var(--color-border);
    border-radius: 50%;
    width: 40px;
    height: 40px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--color-text);
    background-color: var(--color-bg);
    z-index: 100;
    transition: background-color 0.3s, color 0.3s, border-color 0.3s;
  }

  .theme-toggle:hover {
    background: var(--color-table-row-hover);
  }
</style>
