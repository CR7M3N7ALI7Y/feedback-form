<template>
  <!--
    ToastNotification.vue
    Всплывающее уведомление, которое появляется после отправки отзыва.
    Автоматически скрывается через 3 секунды или по клику.
  -->
  <transition name="toast-slide">
    <div v-if="visible" class="toast" role="status" aria-live="polite">
      <span class="toast-icon">✓</span>
      <span class="toast-message">{{ message }}</span>
      <button
        class="toast-close"
        type="button"
        aria-label="Закрыть уведомление"
        @click="close"
      >
        ×
      </button>
    </div>
  </transition>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({
  message: {
    type: String,
    default: 'Спасибо за отзыв!'
  },
  // Сколько миллисекунд показывать перед автоскрытием
  duration: {
    type: Number,
    default: 3000
  }
})

const emit = defineEmits(['close'])

const visible = ref(false)
let timer = null

function startTimer() {
  timer = setTimeout(() => close(), props.duration)
}

function clearTimer() {
  if (timer) {
    clearTimeout(timer)
    timer = null
  }
}

function close() {
  clearTimer()
  visible.value = false
  // Даём проиграться анимации исчезновения, затем сообщаем родителю
  setTimeout(() => emit('close'), 200)
}

onMounted(() => {
  visible.value = true
  startTimer()
})

onBeforeUnmount(() => {
  clearTimer()
})
</script>

<style scoped>
.toast {
  position: fixed;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 14px 20px;
  background: #2b7a4b;
  color: #fff;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 500;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
  z-index: 1100;
  max-width: calc(100% - 32px);
}

.toast-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 22px;
  height: 22px;
  background: rgba(255, 255, 255, 0.25);
  border-radius: 50%;
  font-size: 13px;
  flex-shrink: 0;
}

.toast-message {
  flex: 1;
}

.toast-close {
  background: transparent;
  border: none;
  color: #fff;
  font-size: 20px;
  line-height: 1;
  cursor: pointer;
  padding: 0 4px;
  opacity: 0.8;
}
.toast-close:hover {
  opacity: 1;
}

/* Анимация появления */
.toast-slide-enter-active,
.toast-slide-leave-active {
  transition: all 0.3s ease;
}
.toast-slide-enter-from,
.toast-slide-leave-to {
  opacity: 0;
  transform: translate(-50%, 20px);
}

/* Адаптивность */
@media (max-width: 640px) {
  .toast {
    bottom: 16px;
    left: 16px;
    right: 16px;
    transform: none;
    max-width: none;
    font-size: 16px;
    padding: 16px 18px;
  }
  .toast-slide-enter-from,
  .toast-slide-leave-to {
    transform: translateY(20px);
  }
}
</style>
