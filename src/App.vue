<template>
  <div>
    <n-grid cols="4" item-responsive>
      <!-- Status Box -->
      <n-gi span="4 600:1">
        <n-card title="Board Status">
          <p>Board number: {{ boardNumber }}</p>
          <p>Status: 
            <span v-if="isActive" style="color: green;">Active</span>
            <span v-else style="color: red;">Inactive</span>
          </p>
          <n-button type="success" @click="showModal = true"> Create New Entity</n-button>
        </n-card>
      </n-gi>
      <!-- List Box -->
      <n-gi span="4 600:3">
        <n-card title="Entities List">
        <n-list v-if="list.length > 0">
          <n-list-item v-for="item in list" :key="item['id']">
            <n-card :title="item['name']">
              <p>Connected to Port: {{ item['port'] }}</p>
              <p>You can control this item's state from here 
                <n-switch v-model:value="item['state']" @update:value="updateOnClick(item['port'], item['state'])"/>
              </p>
              <n-space v-if="item['timer'] != null">
                <p>Timer is set to turn 
                  <span v-if="item['timer'].state == 1" class="green">On</span>
                  <span v-else>Off</span>, at {{ item['timer'].executionDate }}
                  <n-button secondary type="warning"> Remove Timer </n-button>
                </p>
              </n-space>
              <n-space>
                <n-button type="error" @click="DeletePort(item['port'])"> Delete </n-button>
                <n-button type="info" @click="showTimerModal = true" v-if="item['timer'] == null"> Set Timer </n-button>
              </n-space>
            </n-card>
          </n-list-item>
        </n-list>
        <p v-else>No Entities Registered</p>
        </n-card>
      </n-gi>
    </n-grid>
    <!-- Create New Entity Model -->
    <n-modal v-model:show="showModal">
      <n-card style="width: 600px" title="Create New Entity" :bordered="false" size="huge" role="dialog" aria-modal="true">
        <p>Item Name</p>
        <n-input type="text" placeholder="Item Name Here" v-model:value="namevalue" :style="{width: '90%'}"/>
        <p>Port Number</p>
        <n-select v-model:value="portvalue" :options="options" :style="{width: '90%'}"/>
        <br/>
        <p>Initial State <n-switch v-model:value="statevalue" /></p>
        <br/>
        <br/>
        <n-button type="info" @click="ConfirmCreatingNewItem"> Create New Entity</n-button>
      </n-card>
    </n-modal>
  </div>
</template>

<script>
import { RouterLink, RouterView } from 'vue-router'
import { ref } from 'vue'

export default {
  setup() {
    return {
      timer: ref(''),
      boardNumber: ref(0),
      isActive: ref(false),
      list: ref([]),
      // creatw
      showModal: ref(false),
      showTimerModal: ref(false),
      options: ref([]),
      optionsRaw: ref([2, 4, 5, 12, 13, 14, 15, 18, 19, 21, 22, 23, 25, 26, 27, 32, 33, 34, 35]),
      portvalue: ref(2),
      statevalue: ref(true),
      namevalue: ref('')
    };
  },
  mounted() {
    this.UpdateData();
    this.timer = setInterval(this.UpdateData, 5000);
    for(var i = 0; i < this.optionsRaw.length; i++) {
      this.options[i] = {
        label: "Port " + this.optionsRaw[i],
        value: this.optionsRaw[i],
      };
    }
  },
  methods: {
    UpdateData() {
      this.axios.post("https://devmohamedgaber-001-site1.atempurl.com/ui/getstatus.php").then((res) => {
        console.log(res);
        this.boardNumber = res.data.info['boardNumber']
        this.isActive = res.data.isActive
        let countedElements = 1;

        res.data.entities.forEach(element => {
          if(element['name'] == "") {
            element['name'] = "Item " + countedElements;
            countedElements++;
          }
          element['state'] = element['state'] == 1 ? true : false;
          res.data.timers.forEach(timer => {
            if(timer['port'] == element['port'])
            {
              element['timer'] = timer;
            }
          });
        });
        this.list = res.data.entities
      });
    },
    updateOnClick(port, state) {
      var params = new URLSearchParams();
      params.append('port', port);
      params.append('state', state ? 1 : 0);
      this.axios.post('https://devmohamedgaber-001-site1.atempurl.com/ui/updatePinState.php', params);
    },
    ConfirmCreatingNewItem() {
      var params = new URLSearchParams();
      params.append('name', this.namevalue);
      params.append('port', this.portvalue);
      params.append('state', this.statevalue ? 1 : 0);
      this.axios.post('https://devmohamedgaber-001-site1.atempurl.com/ui/addNewEntity.php', params).then(() => {
        this.UpdateData();
        this.showModal = false;
      });
    },
    DeletePort(portNumber) {
      var params = new URLSearchParams();
      params.append('port', portNumber);
      this.axios.post('https://devmohamedgaber-001-site1.atempurl.com/ui/deleteport.php', params).then(() => {
        this.UpdateData();
      });
      
    }
  },
  beforeUnmount() {
    clearInterval(this.timer);
  }
}
</script>