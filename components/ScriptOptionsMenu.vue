<template>
  <v-card width="256" class="mx-auto" height="500">
    <v-navigation-drawer permanent>
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title class="text-h6">
            Setup phases
          </v-list-item-title>
          <!-- <v-list-item-subtitle>
            subtext
          </v-list-item-subtitle> -->
        </v-list-item-content>
      </v-list-item>

      <v-divider></v-divider>

      <v-list dense nav>
        <v-list-item v-for="item in items" v-if="item.computerType == 'both' ? true : item.computerType == computerType"
          :key="item.title" @click="$emit('switchSettings', item.page)" button>
          <v-list-item-content>
            <v-list-item-title>
              {{items.filter(item => item.computerType == 'both' ? true : item.computerType == computerType).indexOf(item) + 1}}.
              {{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
  </v-card>
</template>

<script>
  export default {
    data() {
      return {
        items: [{
            title: 'Initial setup',
            page: 'initialSetup',
            computerType: 'both'
          },
          {
            title: 'Networking',
            page: 'networking',
            computerType: 'both'
          },
          {
            title: 'Server Roles',
            page: 'serverRoles',
            computerType: 'server'
          },
          {
            title: 'Other services',
            page: 'otherServices',
            computerType: 'both'
          },
          {
            title: 'Users',
            page: 'users',
            computerType: 'both'
          },
          {
            title: 'Build',
            page: 'build',
            computerType: 'both'
          }
        ]
      }
    },
    computed: {
      computerType() {
        return this.isServer ? 'server' : 'client'
      }
    },
    methods: {
      goTo(page) {
        this.$router.push(page);
      }
    },
    props: {
      isServer: {
        type: Boolean
      }
    }
  }

</script>
