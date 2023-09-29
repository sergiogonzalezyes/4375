<template>
  <v-app id="app">
    <!-- AppBar -->
    <v-app-bar app flat v-if="!isLoginPage">
      <!-- Menu for Small Screens -->
      <v-menu offset-y v-if="isSmallScreen">
        <template v-slot:activator="{ on, attrs }">
          <v-btn icon v-bind="attrs" v-on="on">
            <v-icon>mdi-menu</v-icon>
          </v-btn>
        </template>
        <v-list>
          <!-- Dynamic links -->
          <router-link v-for="link in filteredLinks" :key="link.name" :to="link.path">
            <v-list-item link>
              <v-list-item-title>{{ link.name }}</v-list-item-title>
            </v-list-item>
          </router-link>
          <v-divider class="my-2"></v-divider>
          <v-list-item @click="handleAuthAction">
            <v-list-item-title>
              {{ isAuthenticated ? 'Logout' : 'Login' }}
            </v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>

      <!-- Icon for Large Screens -->
      <v-app-bar-nav-icon @click="drawer = !drawer" v-else></v-app-bar-nav-icon>

      <v-container class="mx-auto d-flex align-center justify-center">
        <!-- Conditional greeting based on logged-in user -->
        <span class="me-4" v-if="isAuthenticated">
          Hello, {{ username }}!
        </span>
        <!-- Optional: Default message for not logged in users -->
        <span class="me-4" v-else>
          Welcome!
        </span>
      </v-container>
    </v-app-bar>

    <!-- Sidebar Navigation (Drawer) for Large Screens -->
    <v-navigation-drawer v-model="drawer" app v-if="!isSmallScreen && !isLoginPage">
      <v-list :rounded="true">
        <!-- Dynamic links -->
        <router-link v-for="link in filteredLinks" :key="link.name" :to="link.path">
          <v-list-item link>
            {{ link.name }}
          </v-list-item>
        </router-link>
        <v-divider class="my-2"></v-divider>
        <v-list-item @click="handleAuthAction">
          {{ isAuthenticated ? 'Logout' : 'Login' }}
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <!-- Main Content -->
    <v-main class="bg-grey-lighten-3">
      <v-container>
        <router-view></router-view>
      </v-container>
    </v-main>
  </v-app>
</template>


<script>
export default {
  data() {
    return {
      drawer: false,
      links: [
        { name: 'Home', path: '/home' },
        { name: 'Notifications', path: '/notifications', requiresAuth: true , roles: ['admin', 'customer', 'barber'] },
        { name: 'Services', path: '/services', requiresAuth: false },
        { name: 'Profile', path: '/profile', requiresAuth: true, roles: ['admin', 'customer', 'barber'] },
      ],
      isSmallScreen: false, // Initial state
    };
  },
  created() {
    window.addEventListener('resize', this.handleResize);
    this.handleResize();
  },
  destroyed() {
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    handleResize() {
      this.isSmallScreen = window.innerWidth < 600;
    },
    handleAuthAction() {
      if (this.isAuthenticated) {
        this.$store.dispatch('app/logout')
          .then(() => {
            this.$router.push('/login')
              .catch(err => {
                if (err.name !== 'NavigationDuplicated') throw err;
              });
          });
      } else {
        this.$router.push('/login')
          .catch(err => {
            if (err.name !== 'NavigationDuplicated') throw err;
          });
      }
    },
  },
  computed: {
    isAuthenticated() {
      return this.$store.state.app.isAuthenticated;
    },
    username() {
      return this.$store.state.app.username;
    },
    isLoginPage() {
      return this.$route.path === '/login';
    },
    // Compute the links to be shown based on authentication and role
    filteredLinks() {
      return this.links.filter(link => {
        if (link.requiresAuth && !this.isAuthenticated) return false;
        if (link.requiresGuest && this.isAuthenticated) return false;
        if (link.roles && !link.roles.includes(this.$store.state.app.role)) return false;
        return true;
      });
    }
  },
};
</script>

<style scoped>

</style>