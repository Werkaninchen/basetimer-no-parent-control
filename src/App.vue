<template>
    <div id="app">
        <BaseTimer class="pb" v-bind:reps="reps" v-bind:timeLimit="lim" />
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
            is_break: false,
            did_start: false,
        };
    },
    components: {
        BaseTimer,
    },
    mounted() {
        bus.$on("inter_done", this.next);
    },
    methods: {
        start() {
            if (!this.did_start) {
                this.lim = this.brk;
                this.did_start = true;
            }
            this.is_break = !this.is_break;
            console.log(this.lim);
            console.log(this.is_break);
            bus.$emit("start");
        },
        reset() {
            this.did_start = false;
            bus.$emit("reset");
        },
        next() {
            if (this.reps <= 0) {
                bus.$emit("completed");
                this.did_start = false;
                return;
            }
            if (this.is_break) {
                this.lim = this.work;
            } else {
                this.lim = this.brk;
                this.reps = this.reps - 1;
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
.pb {
    margin-bottom: 100vh;
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
