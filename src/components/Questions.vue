<template>
    <div class="questions-ctr">
        <div class="progress">
            <div class="bar" :style="{ width: `${questionsAnswered / questions.length * 100}%`}"></div>
            <div class="status">{{ questionsAnswered }} out of {{ questions.length }} questions answered</div>
        </div>

        <div class="timer-container">
            <svg class="timer-spinner" width="120" height="120">
                <!-- 🎨 Définition du gradient -->
                <defs>
                    <linearGradient id="gradient-timer" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" style="stop-color:#667eea;stop-opacity:1" />
                        <stop offset="100%" style="stop-color:#764ba2;stop-opacity:1" />
                    </linearGradient>
                    
                    <!-- Gradient pour l'alerte -->
                    <linearGradient id="gradient-warning" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" style="stop-color:#ff6b6b;stop-opacity:1" />
                        <stop offset="100%" style="stop-color:#ee5a24;stop-opacity:1" />
                    </linearGradient>
                </defs>
                
                <circle class="timer-bg" cx="60" cy="60" r="52" />
                <circle 
                    class="timer-progress"
                    :class="{ 'timer-warning': timeLeft <= 3 }"
                    cx="60" 
                    cy="60" 
                    r="52"
                    :style="{ 
                        strokeDashoffset: circleOffset,
                        stroke: timeLeft <= 3 ? 'url(#gradient-warning)' : 'url(#gradient-timer)'
                    }"
                />
            </svg>
            <div class="timer-text" :class="{ 'timer-text-warning': timeLeft <= 3 }">
                <span class="timer-number">{{ timeLeft }}</span>
                <span class="timer-label">sec</span>
            </div>
        </div>

        <transition-group name="fade">
            <div class="single-question" v-for="(question, index) in questions" :key="question.q"
                v-show="index === questionsAnswered">
                <div class="question">{{ question.q }}</div>
                <div class="answers">
                    <div class="answer" v-for="answer in question.answers" :key="answer.text" @click.prevent="answerQuestion(answer.is_correct)">{{ answer.text }}</div>
                </div>
            </div>
        </transition-group>
    </div>
</template>

<script>
export default {
    name: "Question",
    emits: ['question-answered'],
    props: {
        questions: {
            type: Object,
            required: true
        },
        questionsAnswered: {
            type: Number,
            required: true
        },
        timer: {
            type: Number,
            required: true
        }
    },
    data() {
        return {
            timeLeft: this.timer,
            timerInterval: null
        };
    },
    computed: {
        // 🎯 NOUVEAU : Calcul de la progression du cercle
        circleOffset() {
            const radius = 52;
            const circumference = 2 * Math.PI * radius; // Périmètre du cercle
            const percentRemaining = this.timeLeft / this.timer;
            return circumference - (circumference * percentRemaining);
        }
    },
    watch: {
        questionsAnswered() {
            this.startTimer();
        }
    },
    mounted() {
        this.startTimer();
    },
    beforeUnmount() {
        clearInterval(this.timerInterval);
    },
    methods: {
        answerQuestion(is_correct) {
            clearInterval(this.timerInterval);
            this.$emit('question-answered', is_correct);
        },
        startTimer() {
            clearInterval(this.timerInterval);
            this.timeLeft = this.timer;
            
            this.timerInterval = setInterval(() => {
                this.timeLeft--;
                
                if (this.timeLeft <= 0) {
                    clearInterval(this.timerInterval);
                    this.$emit('time-up');
                }
            }, 1000);
        }
    }
};
</script>

<style></style>