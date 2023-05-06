<template>
    <n-card title="Board Status">
        <p>Board number: {{ boardNumber }}</p>
        <p>Status: 
            <span v-if="isActive" style="color: green;">Active</span>
            <span v-else style="color: red;">Inactive</span>
        </p>
        <router-link to="/createEntity">
            <n-button type="success"> Create New Entity</n-button>
        </router-link>
    </n-card>
</template>
<script>

import { ref } from 'vue'
    export default {
        setup() {
            const boardNumber = ref(0);
            const timer = ref('');
            const isActive = ref(false);
            return {
                boardNumber,
                isActive,
                timer
            };
        },
        mounted() {
            this.UpdateData();
            this.timer = setInterval(this.UpdateData, 5000);
        },
        methods: {
            UpdateData() {
                this.axios.post("http://devmohamedgaber-001-site1.gtempurl.com/ui/getstatus.php").then((res) => {
                    console.log(res);
                    this.boardNumber = res.data.info['boardNumber']
                    this.isActive = res.data.isActive
                });
            },
            CancelUpdater() {
                clearInterval(this.timer);
            }
        },
        beforeUnmount() {
            this.CancelUpdater();
        }
    }
</script>