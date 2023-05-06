<template>
    <n-card title="Entities List">
        <n-list>
            <n-list-item v-for="item in list" :key="item['id']">
                <n-card :title="item['name']">
                    <p>Connected to Port: {{ item['port'] }}</p>
                    <p>You can control this item's state from here 
                        <n-switch v-model:value="item['state']" @update:value="updateOnClick(item['port'], item['state'])"/>
                    </p>
                    
                    
                </n-card>
            </n-list-item>
        </n-list>
    </n-card>
</template>
<script>
import { ref } from 'vue'
    export default {
        setup() {
            const list = ref([]);
            return {
                list,
            };
        },
        mounted() {
            this.axios.get("http://devmohamedgaber-001-site1.gtempurl.com/ui/getstatus.php").then((res) => {
                let countedElements = 1;
                res.data.entities.forEach(element => {
                    if(element['name'] == "") {
                        element['name'] = "Item " + countedElements;
                        countedElements++;
                    }
                    element['state'] = element['state'] == 1 ? true : false;
                });
                this.list = res.data.entities
            });
        },
        methods: {
            updateOnClick(port, state) {
                var params = new URLSearchParams();
                params.append('port', port);
                params.append('state', state ? 1 : 0);
                this.axios.post('http://devmohamedgaber-001-site1.gtempurl.com/ui/updatePinState.php', params);
            }
        }
    }
</script>