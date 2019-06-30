<template>
  <b-navbar toggleable="lg" class="navbar-default">
    <b-navbar-brand to="/">
      Archive Search
    </b-navbar-brand>

    <b-navbar-toggle target="nav-collapse" />

    <b-collapse id="nav-collapse" is-nav>
      <!--b-navbar-nav>
        <b-nav-item to="/categories">
          Каталог
        </b-nav-item>
      </b-navbar-nav-->

      <!-- Right aligned nav items -->
    </b-collapse>
  </b-navbar>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'Navbar',
  computed: {
    ...mapGetters(['isAuthenticated', 'loggedInUser'])
  },
  methods: {
    async logout() {
      await this.$auth.logout()
    }
  }
}
</script>

<style lang="scss">
.navbar-default {
    background-color: #370622;
    border-color: #3b1032;
    color: white;

  .navbar-brand {
    color: white;
    &:hover, &:active, &:visited { color: rgba(white, .5); }

    > li > a, > li > a:hover {
      color: white;
    }
  }

  .navbar-nav {
    .nav-link, .nav-link:hover { color: white; }
  }
}
</style>
