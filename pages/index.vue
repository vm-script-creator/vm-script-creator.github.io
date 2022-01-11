<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card>
        <v-card-title class="headline">
          Welcome to the VM Script Creator
        </v-card-title>
        <v-card-text>
          <p>VM Script Creator allows you to create customisable PowerShell scripts that can automate the setup of your VMs (creation of users, services, debloating, etc).</p>
          <p>Thank you for using VM Script Creator, press "Create new script" to get started.</p>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn
            color="primary"
            nuxt
            to="/create-script"
          >
            Create new script
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'IndexPage'
}
</script>
