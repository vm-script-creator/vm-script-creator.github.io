<template>
    <v-form>
        <v-text-field 
            label="Hostname" 
            counter="15"
            v-model="hostname"
        >
            
        </v-text-field>
    </v-form>
</template>

<script>
    export default {
        props: {
            hostname: {
                type: String
            },
        }
    }
</script>