<template>
<v-container v-if="user != null" class="fill-height d-flex flex-column">
    <v-card class="rounded-lg w-100 bg-grey-lighten-3">
        <v-card-title class="d-flex flex-row mt-2">
            <p class="text-h4 font-weight-black">ADMIN PANEL</p>
            <p class="ml-auto text-overline font-weight-bold">{{time.year}}/{{time.month}}/{{time.day}} {{time.hour}}:{{time.minute}}</p>
        </v-card-title>
        <v-card-text class="d-flex flex-row">
            <p class="text-button text-grey-darken-1 font-weight-bold">ACSS</p>
            <v-btn class="ml-auto" variant="text" density="compact" color="black" icon="mdi-exit-to-app" @click="quit"/>
        </v-card-text>
        <v-divider class="mx-6"></v-divider>
        <v-card-text class="d-flex flex-column mr-5">
            <div class="d-flex flex-row">
                <p class="text-button text-black font-weight-bold ml-auto">charges</p>
                <p class="text-button text-green font-weight-bold ml-3">{{ report.charge_times }}</p>
            </div>
            <div class="d-flex flex-row">
                <p class="text-button text-black font-weight-bold ml-auto">charging time</p>
                <p class="text-button text-red font-weight-bold ml-3">{{ report.total_duration }}</p>
                <p class="text-overline text-grey-darken-1 font-weight-bold ml-1">H</p>
                <p class="text-button text-black font-weight-bold ml-6">amount</p>
                <p class="text-button text-green font-weight-bold ml-3">{{ report.total_amount }}</p>
            </div>
            <div class="d-flex flex-row">
                </div>
            <div class="d-flex flex-row">
                <p class="text-button text-black font-weight-bold ml-auto">charge fee</p>
                <p class="text-button text-grey-darken-1 font-weight-bold ml-3">{{ report.charge_fee }}</p>
                <p class="text-button text-black font-weight-bold ml-3">+</p>
                <p class="text-button text-black font-weight-bold ml-3">service fee</p>
                <p class="text-button text-grey-darken-1 font-weight-bold ml-3">{{ report.service_fee }}</p>
                <p class="text-button text-black font-weight-bold ml-3">=</p>
                <p class="text-button text-black font-weight-bold ml-3">income</p>
                <p class="text-button text-green font-weight-bold ml-3">{{ report.total_fee }}</p>
            </div>
        </v-card-text>
    </v-card>

    <v-spacer class="my-3"></v-spacer>
    
    <v-card class="rounded-lg w-100">
        <v-card-title>
            <p class="text-h6 font-weight-bold text-grey-darken-1">PENDING</p>
        </v-card-title>
        <v-card-text class="d-flex flex-wrap">
            <template v-for="req in wait_list">
                <v-chip label variant="outlined" color="grey-darken-2" class="ma-1">
                    <p class="text-button font-weight-bold" :class="req.mode == 1 ? 'text-green' : 'text-blue'">{{ req.mode == 1 ? 'F' : 'T' }}</p>
                    <p class="text-button font-weight-bold ml-2">{{ req.car_id }}</p>
                    <p class="text-button ml-3 text-orange">{{ req.amount }}</p>
                </v-chip>
            </template>
        </v-card-text>
    </v-card>

    <v-spacer class="my-3"></v-spacer>

    <!-- 当前状态 -->
    <template v-for="pile in piles">
    <div class="w-100 d-flex flex-row">
        <v-card color="grey-lighten-2">
            <v-card-title class="d-flex flex-column align-center">
                <p class="font-weight-bold" :class="pile.status == 2 ? 'text-red' : 'text-green'">{{ pile.id }}</p>
            </v-card-title>
            <v-divider></v-divider>
            <v-card-text class="d-flex flex-column justify-center align-center">
                <p class="text-overline font-weight-bold">{{ pile.time }}H</p>
                <p class="text-overline font-weight-bold">{{ pile.amount }}kWh</p>
                <template v-if="pile.status == 2">
                    <v-btn variant="text" icon="mdi-refresh-circle" color="blue" class="mt-2" @click="alter_pile_state(pile.id,1)"/>
                </template>
                <template v-else>
                    <v-btn variant="text" icon="mdi-close-circle" color="red-lighten-1" class="mt-2" @click="alter_pile_state(pile.id,0)"/>
                </template>
            </v-card-text>
        </v-card>
        <template v-for="task in pile.tasks">
            <template v-if="task.valid">
                <v-card class="ml-3 w-50">
                    <v-card-title class="d-flex flex-row">
                        <v-chip label>
                            <p class="text-overline font-weight-bold">{{ task.car_id }}</p>
                            <p class="text-overline font-weight-bold ml-4" :class="status_color[String(task.status)]">{{ status_str[String(task.status)] }}</p>
                        </v-chip>
                        <p class="ml-4 text-overline font-weight-bold text-grey">{{ task.user_id }}</p>
                    </v-card-title>
                    <v-list>
                        <v-list-item>
                            <div class="d-flex flex-row">
                                <p class="text-button font-weight-bold">CHARGE</p>
                                <p class="text-button font-weight-bold ml-6" :class="task.status == 1 ? 'text-blue' : 'text-grey'">{{ task.charged_amount }}</p>
                                <p class="text-button font-weight-bold text-grey ml-1"> / {{ task.request_amount }}</p>
                            </div>
                        </v-list-item>
                        <v-list-item>
                            <div class="d-flex flex-row">
                                <p class="text-button font-weight-bold">START</p>
                                <p class="text-button font-weight-bold text-grey ml-6">{{ task.start_time != "" ? task.start_time : 'N/A' }}</p>
                            </div>
                        </v-list-item>
                        <v-list-item>
                            <div class="d-flex flex-row">
                                <p class="text-button font-weight-bold">REMAIN</p>
                                <p class="text-button font-weight-bold text-grey ml-6">{{ task.remain == -1 ? "N/A" : task.remain }} </p>
                                <p class="text-button font-weight-bold text-grey ml-1">{{ task.remain == -1 ? "" : "H"}}</p>
                            </div>
                        </v-list-item>
                    </v-list>
                </v-card>
            </template>
            <template v-else>
                <v-card class="ml-3 d-flex w-50">
                    <v-card-text class="d-flex flex-column justify-center align-center text-grey font-weight-black">EMPTY</v-card-text>
                </v-card>
            </template>
            
        </template>
    </div>
    <v-spacer class="my-3"></v-spacer>
    </template>

</v-container>
</template>

<script setup>
import axios from 'axios';
import router from '../router/index';
import { ref, reactive, inject, getCurrentInstance, onMounted }  from 'vue'

const user = localStorage.getItem('admin');
var query_timer = null
const time = inject('time');
const time_sync = inject('time_sync');
const sys_timer = inject('sys_timer');

const quit = () => {
    localStorage.removeItem('admin');
    clearInterval(query_timer);
    clearInterval(sys_timer);
    router.push('/admin/login');
}

const status_str = {
    '-1': 'PENDING',
    '0': 'WAITING',
    '1': 'CHARGING',
    '2': 'FINISTHED'
}

const status_color = {
    '-1': 'text-grey',
    '0': 'text-orange',
    '1': 'text-green',
    '2': 'text-blue'
}

const report = reactive({
    charge_times: 0,
    service_fee:0.0,
    charge_fee:0.0,
    total_fee: 0.0,
    total_amount:0.0,
    total_duration:0.0
})

const update_report = () => {
    console.log('update report')
    axios.get('/api/admin/query/report', {
        params:{
            start:"",
            end:""
        }
    }).then(res => {
        console.log(res.data)
        if(res.data.status == 0){
            report.charge_times = res.data.data.count
            report.service_fee = Number(res.data.data.service).toFixed(2)
            report.charge_fee = Number(res.data.data.charge).toFixed(2)
            report.total_fee = Number(res.data.data.total).toFixed(2)
            report.total_amount = Number(res.data.data.amount).toFixed(2)
            report.total_duration = Number(res.data.data.duration).toFixed(2)
        }else{
            console.log('请求失败: ' + res.data.message);
        }
    }).catch(err => {
        console.log(err);
    })
}


const wait_list = reactive([])

const alter_pile_state = (pile_id, status) => {
    console.log('query wait area')
    axios.post('/api/admin/alter/pile',{
        pile_id: String(pile_id),
        status: parseInt(status)
    }).then(res => {
        console.log(res.data)
        if(res.data.status == 0){
            console.log('操作成功');
            start_query();
        }else{
            alert('操作失败: ' + res.data.message);
            console.log('操作失败: ' + res.data.message);
        }
    }).catch(err => {
        console.log(err);
    })
}

const query_wait = () => {
    console.log('query wait area')
    axios.get('/api/admin/query/waitlist').then(res => {
        console.log(res.data)
        if(res.data.status == 0){
            wait_list.splice(0)
            if(res.data.data.length == 0){
                return
            }
            res.data.data.forEach((item) => {
                wait_list.push(reactive({
                    car_id:item.car_id,
                    amount:Number(item.amount).toFixed(1),
                    mode:item.mode,
                }))
            })
        }else{
            console.log('请求失败: ' + res.data.message);
        }
    }).catch(err => {
        console.log(err);
    })
}

class Task{
    constructor(){
        this.user_id = '';
        this.car_id = '';
        this.valid = false;
        this.status = 0;
        this.request_amount = 0;
        this.start_time = '';
        this.charged_amount = 0;
        this.remain = -1;
    }

    update(input){
        this.user_id = input.user_id;
        this.car_id = input.car_id;
        this.status = input.status;
        this.start_time = input.start_time;
        this.request_amount = Number(input.all_amount).toFixed(2);
        this.charged_amount = Number(input.charged_amount).toFixed(2);
        this.remain = input.time_remain;
        if(this.remain >= 0){
            this.remain = Number(this.remain / 3600).toFixed(1);
        }
    }
}

class Pile{
    constructor(pile_id, pile_mode){
        this.id = pile_id;
        this.mode = pile_mode;
        this.status = 0;
        this.amount = 0;
        this.time = 0;
        this.tasks = [
            reactive(new Task()),
            reactive(new Task()),
        ]
    }

    update(){
        axios.get('/api/admin/query/state', {
            params:{
                pile_id: "" + this.id,
            }
        }).then(res => {
            console.log(res.data)
            if(res.data.status == 0){
                this.amount = Number(res.data.data.amount).toFixed(1);
                this.time = Number(res.data.data.time / 3600).toFixed(1);
                this.status = res.data.data.status;
                var charging = res.data.data.charging;
                var waiting = res.data.data.waiting;
                if(charging != null){
                    this.tasks[0].update(charging);
                    this.tasks[0].valid = true;
                }else{
                    this.tasks[0].valid = false;
                }
                if(waiting != null && waiting.length > 0){
                    this.tasks[1].update(waiting[0]);
                    this.tasks[1].valid = true;
                }else{
                    this.tasks[1].valid = false;
                }
            }else{
                console.log('请求失败: ' + res.data.message);
            }
        }).catch(err => {
            console.log(err);
        })
    }
}

const piles = [
    reactive(new Pile("F1", 1)),
    reactive(new Pile("F2", 1)),
    reactive(new Pile("T1", 0)),
    reactive(new Pile("T2", 0)),
    reactive(new Pile("T3", 0)),
]

const start_query = () => {
    console.log('start query');
    query_wait();
    update_report();
    for(var i in piles){
        piles[i].update();
    }
}

if(user != null){
    time_sync();
    start_query();
    query_timer = setInterval(start_query, 5 * 1000);
}else{
    quit();
}
 
</script>