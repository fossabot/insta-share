<template>
  <header id="header">
    <div class="header-title">
      <h1>InstaShare<span class="emoji">⚡</span></h1>
      <span>Instant File Sharing powered by IPFS Protocol</span>
    </div>
    <div class="header-menu">
      <nav class="header-navbar">
        <router-link :to="{ name: 'home' }" active-class="active" exact>Home</router-link>

        <a href="https://github.com/nyancodeid/insta-share" target="_blank" rel="noopener">Github</a>

        <router-link :to="{ name: 'about' }" active-class="active" exact>About</router-link>
      </nav>
    </div>
  </header>
</template>

<script>
export default {
  name: "AppHeader"
}
</script>

<style lang="scss">
#header {
  display: flex;
  justify-content: space-between;
  align-items: center;

  padding: 2em 64px 1.5em 64px;
  border-bottom: 2px dashed var(--secondary-color);

  .header-title {
    h1 {
      font-size: 1.7rem;
      margin: 0 0 8px 0;

      span.emoji {
        font-size: 1.6rem;
      }
    }
    span {
      font-size: .9rem;
    }
  }

  .header-menu {
    display: flex;
    align-items: center;

    nav {
      display: flex;
      text-align: right;
      padding-right: 32px;

      a {
        color: var(--contrast-color);
        margin-right: 16px;
        padding-bottom: 8px;
        text-decoration: none;

        border-bottom: 1px solid;
        cursor: pointer;

        &.active {
          font-weight: bold;
        }
      }
    }
  }
}
</style>