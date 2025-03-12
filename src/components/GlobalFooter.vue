<template>
  <footer>
    <p>
      Created by Braeden Kilburn |
      <a
        href="https://github.com/braedenkilburn/coin-game"
        target="_blank"
        rel="noopener noreferrer"
      >
        View Source Code
      </a>
    </p>
  </footer>
</template>

<style scoped lang="scss">
footer {
  padding: 1rem;
  font-size: 0.9rem;
  color: var(--color-text);
  opacity: 0.8;
  background-color: var(--color-background-soft);
  width: 100%;
  text-align: center;

  a {
    color: var(--color-heading);
    text-decoration: none;
    transition: opacity 0.2s ease;

    &:hover {
      opacity: 0.7;
    }
  }
}
</style>
