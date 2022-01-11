<template>
  <v-container fluid>
        <v-snackbar
          v-model="snackbar"
        >
      {{ snackBarText }}
    </v-snackbar>

    <v-row>
      <v-col col="3">
        <ScriptOptionsMenu @switchSettings="switchPage" :isServer="setupDetails.isServer" />
      </v-col>
      <v-col sm="9">
        <home v-if="currentPage == 'default'" />
        <div v-if="currentPage == 'initialSetup'">
          <v-checkbox v-model="setupDetails.isServer" :label="`Computer is a: ${setupDetails.isServer ? 'Server' : 'Workstation'}`" />
          <v-checkbox v-model="setupDetails.changeHostname" :label="`Change hostname: ${setupDetails.changeHostname.toString()}`" />
          <v-text-field label="Hostname" counter="15" v-model="setupDetails.hostname" :disabled="!setupDetails.changeHostname" />
          <v-checkbox v-model="setupDetails.changeAdminPassword" :label="`Change admin password: ${setupDetails.changeAdminPassword.toString()}`" />
          <v-text-field label="Admin password" counter="256" v-model="setupDetails.adminPwd" :disabled="!setupDetails.changeAdminPassword" />
        </div>
        <div v-if="currentPage == 'serverRoles'">
          <v-checkbox v-model="setupDetails.ADDS.install" :label="`Install Active Directory Domain Services: ${setupDetails.ADDS.install.toString()}`" />
          <div v-if="setupDetails.ADDS.install">
            <v-text-field label="Domain Name (e.g. lab.local)" counter="256" v-model="setupDetails.ADDS.domainName" />
            <v-text-field label="Domain Netbios Name (e.g. LAB)" counter="256" v-model="setupDetails.ADDS.netbiosName" />
          </div>
        </div>
        <div v-if="currentPage == 'otherServices'">
          <v-checkbox v-model="setupDetails.enableWinRM" :label="`Enable WinRM: ${setupDetails.enableWinRM.toString()}`" />
          <v-checkbox v-model="setupDetails.enableRDP" :label="`Enable RDP: ${setupDetails.enableRDP.toString()}`" />
          <v-checkbox v-model="setupDetails.ADCS.install" :label="`Install ADCS: ${setupDetails.ADCS.install.toString()}`" />
        </div>
        <div v-if="currentPage == 'users'">
            <v-text-field v-model="userSam" label="Sam Account Name" />
            <v-text-field v-model="userDisplayName" label="Display Name" />
            <v-text-field v-model="userPassword" label="Password" />
            <v-checkbox v-model="userWinRMAccess" label="WinRM Access" />
            <v-btn
              elevation="2"
              large
              class="mb-4"
              @click="addUser"
            >Add User</v-btn>
            <v-data-table
              dense
              :headers="userTableHeaders"
              :items="setupDetails.users"
              item-key="name"
              class="elevation-1"
              v-model="selectedUsers"
              :singleSelect="false"
              show-select
            ></v-data-table> 
            <v-btn
              fab
              small
              dark
              class="mt-2"
              v-if="selectedUsers.length > 0"
              @click="deleteUsers"
            >
              <v-icon>mdi-delete</v-icon>
            </v-btn>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import ScriptOptionsMenu from "../components/ScriptOptionsMenu.vue";
  import home from "../components/scriptCreator/home.vue";

  export default {
    name: "CreateScript",
    components: {
      ScriptOptionsMenu,
      home
    },
    computed: {},
    data() {
      return {
        snackbar: false,
        snackBarText: '',
        userSam: '',
        userDisplayName: '',
        userPassword: '',
        userWinRMAccess: false,
        selectedUsers: [],
        currentPage: "default",
        userTableHeaders: [{text: "Name", value: "samAccountName"}, {text: "Display Name", value: "name"}, {text: "Password", value: "password"}, {text: "WinRM Access", value: "winRM"}],
        setupDetails: {
          hostname: "",
          adminPwd: "",

          // PROPS
          changeHostname: false,
          changeAdminPassword: false,
          isServer: false,
          
          // ADDS
          ADDS: {
            install: false,
            domainName: "",
            netbiosName: ""
          },
          // ADCS
          ADCS: {
            install: false
          },

          // other services
          enableWinRM: false,
          enableRDP: false,

          users: []
        }
      }
    },
    methods: {
      switchPage(page) {
        this.currentPage = page;
      },
      addUser() {
        if (this.setupDetails.users.map(u => u.samAccountName).includes(this.userSam)) {
          this.snackBarText = `User ${this.userSam} already exists`;
          this.snackbar = true;
          return;
        }
        this.setupDetails.users.push({
          name: this.userDisplayName,
          samAccountName: this.userSam,
          password: this.userPassword,
          winRM: this.userWinRMAccess
        });
        this.userSam = "";
        this.userDisplayName = "";
        this.userPassword = "";
        this.userWinRMAccess = false;
      },
      deleteUsers() {
        this.setupDetails.users = this.setupDetails.users.filter(user => !this.selectedUsers.includes(user)) // console.log(this.selectedUsers)
        this.selectedUsers = []
      }
    }
  }

</script>
