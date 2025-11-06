<template>
    <div class="container">
        <h1>秒表</h1>
        <div class="display">{{ formattedTime }}</div>
        <div class="controls">
            <button @click="start" :disabled="isRunning">开始</button>
            <button @click="pause" :disabled="!isRunning">暂停</button>
            <button @click="lap" :disabled="!isRunning">记圈</button>
            <button @click="reset">重置</button>
        </div>
        <div class="laps-container">
            <h2>分圈记录</h2>
            <div class="laps-list">
                <div v-if="laps.length === 0" class="no-laps">暂无分圈记录</div>
                <div v-else>
                    <div
                        v-for="lap in laps"
                        :key="lap.lapNumber"
                        class="lap-item"
                        :class="{
                            fastest: laps.length > 1 && lap.lapTime === fastestLap,
                            slowest: laps.length > 1 && lap.lapTime === slowestLap
                        }"
                    >
                        <div class="lap-number">圈 {{ lap.lapNumber }}</div>
                        <div class="lap-time">{{ formatTime(lap.lapTime) }}</div>
                        <div class="lap-total">累计: {{ formatTime(lap.totalTime) }}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
</template>

<script>
export default {
    name: 'App',
    data() {
        return {
            startTime: null,
            pausedTime: 0,
            isRunning: false,
            animationFrameId: null,
            laps: [],
            lastLapTime: 0,
            now: 0
        };
    },
    computed: {
        formattedTime() {
            const elapsed = this.isRunning && this.startTime !== null
                ? performance.now() - this.startTime
                : this.pausedTime;
            return this.formatTime(elapsed);
        },
        fastestLap() {
            if (this.laps.length === 0) return 0;
            return Math.min(...this.laps.map(l => l.lapTime));
        },
        slowestLap() {
            if (this.laps.length === 0) return 0;
            return Math.max(...this.laps.map(l => l.lapTime));
        }
    },
    methods: {
        start() {
            if (!this.isRunning) {
                this.startTime = performance.now() - this.pausedTime;
                this.isRunning = true;
                this.update();
            }
        },
        pause() {
            if (this.isRunning) {
                this.isRunning = false;
                this.pausedTime = performance.now() - this.startTime;
                if (this.animationFrameId) {
                    cancelAnimationFrame(this.animationFrameId);
                    this.animationFrameId = null;
                }
            }
        },
        lap() {
            if (this.isRunning) {
                const currentTime = performance.now() - this.startTime;
                const lapTime = currentTime - this.lastLapTime;
                this.lastLapTime = currentTime;
                this.laps.push({
                    lapNumber: this.laps.length + 1,
                    lapTime: lapTime,
                    totalTime: currentTime
                });
            }
        },
        reset() {
            this.isRunning = false;
            this.startTime = null;
            this.pausedTime = 0;
            this.laps = [];
            this.lastLapTime = 0;
            if (this.animationFrameId) {
                cancelAnimationFrame(this.animationFrameId);
                this.animationFrameId = null;
            }
        },
        update() {
            if (!this.isRunning) return;
            this.now = performance.now();
            this.animationFrameId = requestAnimationFrame(() => this.update());
        },
        formatTime(milliseconds) {
            const totalSeconds = Math.floor(milliseconds / 1000);
            const minutes = Math.floor(totalSeconds / 60);
            const seconds = totalSeconds % 60;
            const ms = Math.floor(milliseconds % 1000);
            return `${this.pad(minutes)}:${this.pad(seconds)}.${this.padMs(ms)}`;
        },
        pad(num) {
            return num.toString().padStart(2, '0');
        },
        padMs(num) {
            return num.toString().padStart(3, '0');
        }
    }
};
</script>

<style scoped>
/* 使用全局样式类名，局部无需覆盖 */
</style>
