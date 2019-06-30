<template>
  <div>
    <Navbar />
    <div id="body" class="container">
      <nuxt />
    </div>
  </div>
</template>

<script>
import Navbar from '~/components/navbar.vue'

export default {
  components: { Navbar }
}
</script>

<style lang="scss">
#body {
  padding-top: 1rem;
}

a {
  color: #56223F;

  &:hover { color: #794C66;}
}
</style>
