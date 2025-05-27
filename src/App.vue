
<script setup>
  import Home from './views/Home.vue'
</script>

<template>
  <header>
    <h1 id="main-title">Solid Quiz</h1>
  </header>

  <main>
    <Home />
  </main>

  <footer>
    <nav>
      <ul>
        <li>
          <a href="https://www.mallambicpierre.fr" target="_blank">https://www.mallambicpierre.fr</a>
        </li>
        <li>
          <a href="https://opentdb.com" target="_blank">Open Trivia Database</a>
        </li>
      </ul>
    </nav>
  </footer>
</template>

<script>
  export default {
    components: {
      Home
    }
  }
</script>