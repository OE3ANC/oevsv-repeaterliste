<script lang="ts">
  import '../app.css';
  import ThemeToggle from '$lib/components/ThemeToggle.svelte';

  let { children } = $props();

  function getStoredTheme(): 'light' | 'dark' | 'system' {
    if (typeof localStorage === 'undefined') return 'system';
    const stored = localStorage.getItem('theme');
    if (stored === 'light' || stored === 'dark' || stored === 'system') return stored;
    return 'system';
  }

  let theme = $state<'light' | 'dark' | 'system'>(getStoredTheme());

  function resolveTheme(t: 'light' | 'dark' | 'system'): 'light' | 'dark' {
    if (t === 'system') {
      return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    }
    return t;
  }

  function setTheme(t: 'light' | 'dark' | 'system') {
    theme = t;
    localStorage.setItem('theme', t);
    document.documentElement.setAttribute('data-theme', resolveTheme(t));
  }

  $effect(() => {
    const mq = window.matchMedia('(prefers-color-scheme: dark)');
    function handleChange() {
      if (theme === 'system') {
        document.documentElement.setAttribute('data-theme', resolveTheme('system'));
      }
    }
    mq.addEventListener('change', handleChange);
    return () => mq.removeEventListener('change', handleChange);
  });
</script>

<ThemeToggle {theme} onToggle={setTheme} />

{@render children()}
