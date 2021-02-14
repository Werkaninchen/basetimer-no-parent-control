<template>
    <div class="base-timer" v-on:click="clckd">
        <svg
            class="base-timer__svg"
            viewBox="0 0 100 100"
            xmlns="http://www.w3.org/2000/svg"
        >
            <g class="base-timer__circle">
                <circle
                    class="base-timer__path-elapsed"
                    cx="50"
                    cy="50"
                    r="45"
                ></circle>
                <path
                    :stroke-dasharray="circleDasharray"
                    class="base-timer__path-remaining"
                    d="
            M 50, 50
            m -45, 0
            a 45,45 0 1,0 90,0
            a 45,45 0 1,0 -90,0
          "
                ></path>
            </g>
        </svg>
        <span class="base-timer__label">{{ formattedTimeLeft }}</span>
        <p class="base-timer__sub_label">{{ repsDone }} / {{ reps }}</p>
    </div>
</template>

<script>
const FULL_DASH_ARRAY = 283;
import { bus } from "../main";

export default {
    props: {
        timeLimit: Number,
        reps: Number,
        repsDone: Number,
    },
    data() {
        return {
            timePassed: 0,
            timerInterval: null,
        };
    },

    computed: {
        circleDasharray() {
            return `${(this.timeFraction * FULL_DASH_ARRAY).toFixed(0)} 283`;
        },

        formattedTimeLeft() {
            const timeLeft = this.timeLeft;
            const minutes = Math.floor(timeLeft / 60);
            let seconds = timeLeft % 60;

            if (seconds < 10) {
                seconds = `0${seconds}`;
            }

            if (minutes <= 0) {
                return `${seconds}`;
            }
            return `${minutes}:${seconds}`;
        },

        timeLeft() {
            return this.timeLimit - this.timePassed;
        },

        timeFraction() {
            const rawTimeFraction = this.timeLeft / this.timeLimit;
            return (
                rawTimeFraction - (1 / this.timeLimit) * (1 - rawTimeFraction)
            );
        },
    },

    watch: {
        timeLeft(newValue) {
            if (newValue === 0 || newValue < 0) {
                this.onTimesUp();
            }
        },
    },
    mounted() {
        bus.$on("start", this.startTimer);
        bus.$on("reset", this.resetTimer);
        bus.$on("completed", this.resetTimer);
    },

    methods: {
        clckd() {
            bus.$emit("clicked");
        },
        onTimesUp() {
            clearInterval(this.timerInterval);
            this.timePassed = 0;
            this.timerInterval = null;
            bus.$emit("inter_done");
        },
        resetTimer() {
            clearInterval(this.timerInterval);
            this.timePassed = 0;
            this.timerInterval = null;
        },
        startTimer() {
            if (this.timerInterval == null) {
                this.timerInterval = setInterval(
                    () => (this.timePassed += 1),
                    1000
                );
            } else {
                clearInterval(this.timerInterval);
                this.timerInterval = null;
            }
        },
    },
};
</script>

<style lang="scss" scoped>
.base-timer {
    position: relative;
    width: 300px;
    height: 300px;

    &__svg {
        transform: scaleX(-1);
    }

    &__circle {
        fill: none;
        stroke: none;
    }

    &__path-elapsed {
        stroke-width: 7px;
        stroke: grey;
    }

    &__path-remaining {
        stroke-width: 7px;
        stroke-linecap: round;
        transform: rotate(90deg);
        transform-origin: center;
        transition: 1s linear all;
        fill-rule: nonzero;
        stroke: currentColor;
        color: #dc372f !important;
    }

    &__label {
        position: absolute;
        width: 300px;
        height: 300px;
        top: -10%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 120px;
    }
    &__sub_label {
        position: absolute;
        width: 300px;
        height: 200px;
        top: 20%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 50px;
    }
}
</style>
