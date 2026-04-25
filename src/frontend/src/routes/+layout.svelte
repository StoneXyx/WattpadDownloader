<script>
  import "../app.css";
  let { children } = $props();
</script>

<svelte:head>
  <style>
    body { min-height: 100vh; }
  </style>
</svelte:head>

{@render children()}
