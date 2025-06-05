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
  padding-bottom: 2rem; /* Add some padding below header */
  /* border-bottom: 1px solid var(--color-border); */ /* Removed border from original example */
  text-align: center;
  display: flex; /* For aligning logo and title */
  flex-direction: column; /* Stack logo above title */
  align-items: center; /* Center items horizontally */
  margin-bottom: 2rem; /* Space below header */
}

.logo {
  display: block;
  /* Adjust width and height as needed, make sure to keep aspect ratio */
  height: 40px; /* Example height, adjust to your logo's aspect ratio and desired size */
  width: auto; /* Let width adjust automatically to maintain aspect ratio */
  margin-bottom: 1rem; /* Space between logo and title */
}

h1 {
  font-size: 1.75rem; /* Adjust size as per your branding */
  font-weight: 600; /* Or as per your brand */
  color: var(--color-heading);
  margin: 0; /* Reset margin for h1 */
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