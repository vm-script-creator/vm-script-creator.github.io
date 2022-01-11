<template>
   <v-container fluid>
     <v-row>
       <v-col col="3">
         <ScriptOptionsMenu @switchSettings="switchPage" />
       </v-col>
       <v-col sm="9">
         <home v-if="currentPage == 'default'" />
         <initialSetup v-if="currentPage == 'initialSetup'" 
                        v-bind:hostname="hostname"/>
          <p>{{hostname}}</p>
       </v-col>
     </v-row>
   </v-container>
</template>

<script>
import ScriptOptionsMenu from "../components/ScriptOptionsMenu.vue";
import home from "../components/scriptCreator/home.vue";
import initialSetup from "../components/scriptCreator/initialSetup.vue";

export default {
    name: "CreateScript",
    components: { ScriptOptionsMenu, initialSetup, home },
    computed: {
    },
    data() {
      return {
        currentPage: "default",
        hostname: "hi",
      }
    },
    methods: {
      switchPage(page) {
        // alert(page);
        this.currentPage = page;
      }
    }
}
</script>
