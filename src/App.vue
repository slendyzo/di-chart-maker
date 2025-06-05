<script setup>
import ChartGenerator from './components/ChartGenerator.vue'; // Now active
</script>

<template>
  <header>
    <img alt="Delphi Intelligence Logo" class="logo" src="./assets/logo.svg" width="213" height="30" />
    <h1>Delphi Intelligence Chart Generator</h1>
  </header>

  <main>
    <ChartGenerator /> <!-- Now active -->
  </main>

  <footer>
    <p>© {{ new Date().getFullYear() }} Delphi Intelligence</p>
  </footer>
</template>

<style scoped>
/* Styles specific to App.vue - this is for the overall page layout */
header {
  line-height: 1.5;
  padding: 2rem 0;
  border-bottom: 1px solid var(--color-border); /* We'll define this in main.css */
  text-align: center; /* Center header content */
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

h1 {
  font-size: 2rem;
  font-weight: 600;
  color: var(--color-heading); /* We'll define this in main.css */
}

main {
  padding: 2rem 0;
  min-height: 70vh; /* Ensure main content takes up space */
}

footer {
  padding: 1rem 0;
  text-align: center;
  font-size: 0.9em;
  color: var(--color-text-light); /* We'll define this in main.css */
  border-top: 1px solid var(--color-border);
}

/* You can add more responsive styles if needed */
/* @media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
} */
</style>