<template>
    <div id="app">
        <BaseTimer
            v-bind:reps="reps"
            v-bind:repsDone="repsDone"
            v-bind:timeLimit="lim"
        />
        <div class="pb status-wraper" style="font-size: 50px">
            <p v-if="is_break">Break</p>
            <p v-else>Work</p>
        </div>
        <div>
            <div>
                <div class="vert">
                    <p>Work time:</p>
                    <input
                        type="number"
                        name="work"
                        id="work"
                        v-model.number="work"
                    />
                </div>
                <div class="vert">
                    <p>Break time:</p>
                    <input
                        type="number"
                        name="brk"
                        id="brk"
                        v-model.number="brk"
                    />
                </div>
                <div class="vert">
                    <p>repetitions</p>
                    <input
                        type="number"
                        name="reps"
                        id="reps"
                        v-model.number="reps"
                    />
                </div>
            </div>
            <div>
                <button v-on:click="start">Start</button>
                <button v-on:click="reset">Reset</button>
            </div>
        </div>
    </div>
</template>

<script>
import BaseTimer from "./components/BaseTimer";
import { bus } from "./main";

export default {
    name: "App",

    data() {
        return {
            work: 20,
            brk: 10,
            reps: 8,
            lim: 10,
            repsDone: 1,
            is_break: false,
            did_start: false,
        };
    },
    components: {
        BaseTimer,
    },
    mounted() {
        bus.$on("inter_done", this.next);
        bus.$on("clicked", this.start);
    },
    methods: {
        start() {
            if (!this.did_start) {
                this.lim = this.brk;
                this.did_start = true;
            }
            this.is_break = !this.is_break;
            bus.$emit("start");
        },
        reset() {
            this.did_start = false;
            this.repsDone = 1;
            this.is_break = false;
            bus.$emit("reset");
        },
        next() {
            if (this.repsDone > this.reps) {
                bus.$emit("completed");
                this.did_start = false;
                this.repsDone = 1;
                return;
            }
            if (this.is_break) {
                this.lim = this.work;
            } else {
                this.lim = this.brk;
                this.repsDone++;
            }

            this.is_break = !this.is_break;
            bus.$emit("start");
        },
    },
};
</script>

<style lang="scss" >
.vert {
    display: flex;
}
body {
    background-color: #00ff00;
}
.status-wraper {
    font-weight: bolder;
}
.pb {
    margin-bottom: 80vh;
}
#app {
    font-family: "Avenir", Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 100%;
}
</style>
