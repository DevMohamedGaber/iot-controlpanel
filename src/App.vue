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
        <n-list hoverable v-if="list.length > 0">
          <n-list-item v-for="item in list" :key="item['id']">
            <template #suffix>
              <n-switch v-model:value="item['state']" @update:value="updateOnClick(item['port'], item['state'])"/>
            </template>
            <n-thing :title="item['name']" content-style="margin-top: 10px;">
              <template #description>
                <n-space size="small" style="margin-top: 4px">
                  <n-tag :bordered="false" type="info" size="small">
                    Port {{ item['port'] }}
                  </n-tag>
                  <n-tag :bordered="false" :type="item['state'] ? 'success' : 'error'" size="small">
                    {{ item['state'] ? "On" : "Off" }}
                  </n-tag>
                </n-space>
              </template>
              <n-list v-if="item['timers'].length > 0">
                <h4>Timers List</h4>
                <n-list-item v-for="(timer, index) in item['timers']" :key="index">
                  Timer is set to turn 
                  <span v-if="timer.state == 1" class="green">On</span>
                  <span v-else class="red">Off</span>, at {{ timer["executionDate"] }}
                  <n-button secondary type="warning" @click="DeleteTimer(timer['id'])">Remove</n-button>
                </n-list-item>
              </n-list>
              <n-space>
                <n-button type="error" @click="DeletePort(item['port'])"> Delete </n-button>
                <n-button type="info" @click="ShowSetTimerForm(item['port'])"> Set Timer </n-button>
              </n-space>
            </n-thing>
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
    <!-- CAdd Timer Model -->
    <n-modal v-model:show="showTimerModal" block-scroll="false"
      positive-text="Confirm"
      negative-text="Cancel">

      <n-card style="width: 600px" role="dialog" aria-modal="true" hoverable="true">
        <h3 class="modalHeader">{{'Add action on port ' + timerPort}}</h3>
        <h4>please select the state of this Port &nbsp; 
          <n-switch v-model:value="timerState"/>
        </h4>
        <n-tabs type="card" justify-content="space-evenly">

          <n-tab-pane name="timer" tab="Timer">
            <n-space justify="space-around">
              <n-input-number v-model:value="timerHours" clearable min="0" placeholder="Hours" style="width: 150px" />
              :
              <n-input-number v-model:value="timerMinuts" clearable min="0" max="59" placeholder="Minuts"  style="width: 150px" />
              :
              <n-input-number v-model:value="timerSeconds" clearable min="0" max="59" placeholder="Seconds"  style="width: 150px" />
            </n-space>
            <br />
            <n-button type="info" @click="AddTimer">Schedule Timer</n-button>
          </n-tab-pane>

          <n-tab-pane name="date" tab="Schadule">
            <h5>please Pick date and time</h5>
            <n-date-picker v-model:value="timerTime" value-format="hh:mm:ss" type="datetime" clearable />
            <br />
            <n-button type="info" @click="AddSchedule">Schedule Timer</n-button>
          </n-tab-pane>
        </n-tabs>
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
      serverURL: ref('https://devmohamedgaber-001-site1.atempurl.com/'),
      timer: ref(''),
      updateInterval: ref(2000),
      boardNumber: ref(0),
      isActive: ref(false),
      list: ref([]),
      // create modal
      showModal: ref(false),
      options: ref([]),
      optionsRaw: ref([2, 4, 5, 12, 13, 14, 15, 18, 19, 21, 22, 23, 25, 26, 27, 32, 33, 34, 35]),
      portvalue: ref(2),
      statevalue: ref(true),
      namevalue: ref(''),
      // timer modal
      showTimerModal: ref(false),
      timerPort: ref(0),
      timerState: ref(1),
      timerDate: ref(null),
      timerTime: ref(null),
      timerHours: ref(null),
      timerMinuts: ref(null),
      timerSeconds: ref(null)
    };
  },
  mounted() {
    this.UpdateData();
    this.timer = setInterval(this.UpdateData, this.updateInterval);
    for(var i = 0; i < this.optionsRaw.length; i++) {
      this.options[i] = {
        label: "Port " + this.optionsRaw[i],
        value: this.optionsRaw[i],
      };
    }
  },
  methods: {
    UpdateData() {
      this.axios.post(this.serverURL + "ui/getstatus.php").then((res) => {
        //console.log(res);
        this.boardNumber = res.data.info['boardNumber']
        this.isActive = res.data.isActive
        let countedElements = 1;

        res.data.entities.forEach(element => {
          if(element['name'] == "") {
            element['name'] = "Item " + countedElements;
            countedElements++;
          }
          element['state'] = element['state'] == 1 ? true : false;
          element['timers'] = [];
          res.data.timers.forEach(timer => {
            if(timer['port'] == element['port'])
            {
              element['timers'].push(timer);
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
      this.axios.post(this.serverURL + "ui/updatePinState.php", params);
    },
    ConfirmCreatingNewItem() {
      var params = new URLSearchParams();
      params.append('name', this.namevalue);
      params.append('port', this.portvalue);
      params.append('state', this.statevalue ? 1 : 0);
      this.axios.post(this.serverURL + "ui/addNewEntity.php", params).then(() => {
        this.UpdateData();
        this.showModal = false;
      });
    },
    ShowSetTimerForm(port) {
      console.log("entered");
      this.showTimerModal = true;
      this.timerPort = port;
    },
    DeletePort(portNumber) {
      var params = new URLSearchParams();
      params.append('port', portNumber);
      this.axios.post(this.serverURL + "ui/deleteport.php", params).then(() => {
        this.UpdateData();
      });
      
    },
    AddSchedule() {
      var params = new URLSearchParams();
      params.append('port', this.timerPort);
      params.append('state', this.timerState ? 1 : 0);
      params.append('dateTime', this.timerTime / 1000);
      this.axios.post(this.serverURL + "ui/ScheduleTimer.php", params).then((res) => {
        this.showTimerModal = false;
        this.timerPort = 0;
        this.timerState = true;
        this.timerDate = null;
        this.timerTime = null;
      });
    },
    AddTimer() {
      let hours = this.timerHours == null ? 0 : this.timerHours * 3600;
      let minutes = this.timerMinuts == null ? 0 : this.timerMinuts * 60;
      let seconds = this.timerSeconds == null ? 0 : this.timerSeconds;
      let now = new Date();
      let targetTime = (now / 1000) + hours + minutes + seconds;

      var params = new URLSearchParams();
      params.append('port', this.timerPort);
      params.append('state', this.timerState ? 1 : 0);
      params.append('dateTime', targetTime);
      this.axios.post(this.serverURL + "ui/ScheduleTimer.php", params).then((res) => {
        this.showTimerModal = false;
        this.timerPort = 0;
        this.timerHours = null;
        this.timerMinuts = null;
        this.timerSeconds = null;
      });
    },
    DeleteTimer(id) {
      var params = new URLSearchParams();
      params.append('id', id);
      this.axios.post(this.serverURL + "ui/deleteTimer.php", params);
    }
  },
  beforeUnmount() {
    clearInterval(this.timer);
  }
}
</script>

<style >
.red {
  color: red;
}
.green {
  color: green;
}
.modalHeader {
  margin-top: 0;
}
.v-binder-follower-content {
  top: 10vh !important;
}
</style>