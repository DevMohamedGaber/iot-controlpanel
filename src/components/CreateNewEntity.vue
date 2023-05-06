<template>
    <div>
        <h2>Create New Entity</h2>
        <p>Item Name</p>
        <n-input type="text" placeholder="Item Name Here" v-model:value="namevalue" :style="{width: '90%'}"/>
        <p>Port Number</p>
        <n-select v-model:value="portvalue" :options="options" :style="{width: '90%'}"/>
        <p>Initial State <n-switch v-model:value="statevalue" /></p>
        <br/>
        <br/>
        <n-button type="info" @click="ConfirmCreatingNewItem"> Create New Entity</n-button>
    </div>
</template>

<script>
import { ref } from 'vue'
export default {
    setup() {
        const optionsRaw = ref([2, 4, 5, 12, 13, 14, 15, 18, 19, 21, 22, 23, 25, 26, 27, 32, 33, 34, 35]);
        const options = ref([]);
        const portvalue = ref(2);
        const statevalue = ref(true);
        const namevalue = ref('');
        return {
            options,
            optionsRaw,
            portvalue,
            statevalue,
            namevalue
        }
    },
    mounted() {
        for(var i = 0; i < this.optionsRaw.length; i++) {
            this.options[i] = {
                label: "Port " + this.optionsRaw[i],
                value: this.optionsRaw[i],
            };
        }
    },
    methods: {
        ConfirmCreatingNewItem() {
            this.axios.post('http://devmohamedgaber-001-site1.gtempurl.com/ui/addNewEntity.php', {
                name: this.namevalue,
                port: this.portvalue,
                state: this.statevalue
            }, {headers: {
                'content-type': 'application/form-data',
                "Access-Control-Allow-Origin": "*"
            }})
        }
    }
}
</script>