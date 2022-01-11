<template>
  <v-container fluid>
    <v-snackbar v-model="snackbar">
      {{ snackBarText }}
    </v-snackbar>

    <v-row>
      <v-col col="3">
        <ScriptOptionsMenu @switchSettings="switchPage" :isServer="setupDetails.isServer" />
      </v-col>
      <v-col sm="9">
        <home v-if="currentPage == 'default'" />
        <div v-if="currentPage == 'initialSetup'">
          <v-checkbox v-model="setupDetails.isServer"
            :label="`Computer is a: ${setupDetails.isServer ? 'Server' : 'Workstation'}`" />
          <v-checkbox v-model="setupDetails.changeHostname"
            :label="`Change hostname: ${setupDetails.changeHostname.toString()}`" />
          <v-text-field label="Hostname" counter="15" v-model="setupDetails.hostname"
            :disabled="!setupDetails.changeHostname" />
          <v-text-field label="Admin password (REQUIRED)" counter="256" v-model="setupDetails.adminPwd" />
          <v-checkbox v-model="setupDetails.changeAdminPassword"
            :label="`Change admin password: ${setupDetails.changeAdminPassword.toString()}`" />
        </div>
        <div v-if="currentPage == 'serverRoles'">
          <v-checkbox v-model="setupDetails.ADDS.install"
            :label="`Install Active Directory Domain Services: ${setupDetails.ADDS.install.toString()}`" />
          <div v-if="setupDetails.ADDS.install">
            <v-text-field label="Domain Name (e.g. lab.local) (REQUIRED)" counter="256"
              v-model="setupDetails.ADDS.domainName" />
            <v-text-field label="Domain Netbios Name (e.g. LAB) (REQUIRED)" counter="256"
              v-model="setupDetails.ADDS.netbiosName" />
          </div>
        </div>
        <div v-if="currentPage == 'otherServices'">
          <v-checkbox v-model="setupDetails.enableWinRM"
            :label="`Enable WinRM: ${setupDetails.enableWinRM.toString()}`" />
          <v-checkbox v-model="setupDetails.enableRDP" :label="`Enable RDP: ${setupDetails.enableRDP.toString()}`" />
          <v-checkbox v-model="setupDetails.ADCS.install"
            :label="`Install ADCS: ${setupDetails.ADCS.install.toString()}`" />
        </div>
        <div v-if="currentPage == 'users'">
          <v-text-field v-model="userSam" label="Sam Account Name (REQUIRED)" />
          <v-text-field v-model="userDisplayName" label="Display Name (REQUIRED)" />
          <v-text-field v-model="userPassword" label="Password (REQUIRED)" />
          <v-checkbox v-model="userWinRMAccess" label="WinRM Access" />
          <v-btn elevation="2" large class="mb-4" @click="addUser" color="green">Add User</v-btn>
          <v-data-table dense :headers="userTableHeaders" :items="setupDetails.users" item-key="name"
            class="elevation-1" v-model="selectedUsers" :singleSelect="false" show-select></v-data-table>
          <v-btn fab small class="mt-2" v-if="selectedUsers.length > 0" @click="deleteUsers" color="red">
            <v-icon>mdi-delete</v-icon>
          </v-btn>
        </div>
        <div v-if="currentPage =='build'">
          <p>There will be 3 scripts being created (Only 2 if the computer is a client):</p>
          <p>1. Customizing Windows via registry tweaks, debloating it, changing hostname, changing admin password and
            other things that require a reboot.</p>
          <p>2. Installing ADDS, setting up the forest, and other server stuff that do require a reboot (ONLY IF THE
            COMPUTER IS A SERVER)</p>
          <p>3. Creating files, installing services that do not require a reboot, creating users, and other tasks that
            do not require a reboot.</p>
          <v-btn elevation="2" large class="mb-4" @click="buildScript" color="primary">Build Scripts</v-btn>
          <v-textarea v-model="script1" label="Script #1" :disabled="true" />
          <v-btn elevation="2" large class="mb-4" @click="downloadScript(1)">Download Script #1</v-btn>
          <v-textarea v-model="script2" label="Script #2" v-if="setupDetails.isServer" :disabled="true" />
          <v-btn elevation="2" large class="mb-4" @click="downloadScript(2)" v-if="setupDetails.isServer">Download
            Script #2</v-btn>
          <v-textarea v-model="script3" label="Script #3" :disabled="true" />
          <v-btn elevation="2" large class="mb-4" @click="downloadScript(3)">Download Script #3</v-btn>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import ScriptOptionsMenu from "../components/ScriptOptionsMenu.vue";
  import home from "../components/scriptCreator/home.vue";
  import generatePowershell from "../utils/generatePowershell";
  export default {
    name: "CreateScript",
    components: {
      ScriptOptionsMenu,
      home
    },
    computed: {},
    data() {
      return {
        script1: '',
        script2: '',
        script3: '',
        snackbar: false,
        snackBarText: '',
        userSam: '',
        userDisplayName: '',
        userPassword: '',
        userWinRMAccess: false,
        selectedUsers: [],
        currentPage: "default",
        userTableHeaders: [{
          text: "Name",
          value: "samAccountName"
        }, {
          text: "Display Name",
          value: "name"
        }, {
          text: "Password",
          value: "password"
        }, {
          text: "WinRM Access",
          value: "winRM"
        }],
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
        this.setupDetails.users = this.setupDetails.users.filter(user => !this.selectedUsers.includes(
          user)) // console.log(this.selectedUsers)
        this.selectedUsers = []
      },
      buildScript() {
        generatePowershell(this.setupDetails).then(script => {
          this.script1 = script.powershell1;
          this.script2 = script.powershell2;
          this.script3 = script.powershell3;
        })
      },
      downloadScript(n) {
        // copilot wrote this function
        let script = "";
        switch (n) {
          case 1:
            script = this.script1;
            break;
          case 2:
            script = this.script2;
            break;
          case 3:
            script = this.script3;
            break;
        }
        const element = document.createElement('a');
        element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(script));
        element.setAttribute('download', `script-${n}.ps1`);
        element.style.display = 'none';
        document.body.appendChild(element);
        element.click();
        document.body.removeChild(element);
      }
    }
  }

</script>
