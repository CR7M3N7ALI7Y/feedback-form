<template>
  <!--
    FeedbackModal.vue
    Модальное окно формы обратной связи для веб-приложения «Конференции.РФ».
    Открывается на странице просмотра заявок после завершения мероприятия.
  -->
  <div v-if="!sent">
    <!-- Кнопка открытия формы (видна, пока отзыв не отправлен) -->
    <button class="btn-open" type="button" @click="openModal">
      Оставить отзыв
    </button>
  </div>

  <!-- Состояние «Ваш отзыв отправлен» (после успешной отправки) -->
  <div v-else class="feedback-sent">
    <span class="sent-icon">✓</span>
    Ваш отзыв отправлен
  </div>

  <!-- Модальное окно -->
  <transition name="modal-fade">
    <div
      v-if="isOpen"
      class="modal-overlay"
      @click.self="closeModal"
      @keydown.esc="closeModal"
      tabindex="0"
      ref="overlayRef"
    >
      <div
        class="modal-window"
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
      >
        <!-- Кнопка закрытия (крестик) -->
        <button
          class="modal-close"
          type="button"
          aria-label="Закрыть"
          @click="closeModal"
        >
          ×
        </button>

        <h2 id="modal-title" class="modal-title">Оставить отзыв</h2>
        <p class="modal-subtitle">Оцените прошедшее мероприятие</p>

        <form class="feedback-form" @submit.prevent="submitForm" novalidate>
          <!-- Поле оценки (звёзды) -->
          <div class="form-group">
            <label class="form-label">Оценка мероприятия <span class="required">*</span></label>
            <div class="stars" role="radiogroup" aria-label="Оценка от 1 до 5">
              <button
                v-for="star in 5"
                :key="star"
                type="button"
                class="star"
                :class="{ active: star <= rating, hover: star <= hoverRating }"
                :aria-label="`Оценка ${star} из 5`"
                @click="setRating(star)"
                @mouseover="hoverRating = star"
                @mouseleave="hoverRating = 0"
              >
                ★
              </button>
            </div>
            <span v-if="errors.rating" class="error-text">{{ errors.rating }}</span>
          </div>

          <!-- Поле комментария -->
          <div class="form-group">
            <label class="form-label" for="comment">Комментарий</label>
            <textarea
              id="comment"
              v-model="comment"
              class="form-textarea"
              rows="4"
              maxlength="500"
              placeholder="Поделитесь впечатлениями о мероприятии..."
            ></textarea>
            <div class="comment-meta">
              <span v-if="errors.comment" class="error-text">{{ errors.comment }}</span>
              <span class="char-counter">{{ comment.length }} / 500</span>
            </div>
          </div>

          <!-- Кнопка отправки -->
          <button class="btn-submit" type="submit" :disabled="submitting">
            {{ submitting ? 'Отправка...' : 'Отправить отзыв' }}
          </button>
        </form>
      </div>
    </div>
  </transition>

  <!-- Toast-уведомление -->
  <ToastNotification
    v-if="toastVisible"
    message="Спасибо за отзыв!"
    @close="toastVisible = false"
  />
</template>

<script setup>
import { ref, reactive, nextTick, watch } from 'vue'
import ToastNotification from './ToastNotification.vue'

// --- Состояние компонента ---
const isOpen = ref(false)        // модальное окно открыто
const sent = ref(false)          // отзыв отправлен (скрывает кнопку)
const toastVisible = ref(false)  // toast-уведомление видно
const submitting = ref(false)    // идёт отправка на сервер

const rating = ref(0)            // выбранная оценка (0 — не выбрана)
const hoverRating = ref(0)       // звезда под курсором (для подсветки)
const comment = ref('')          // текст комментария

const overlayRef = ref(null)     // ссылка на overlay для управления фокусом

// Ошибки валидации по полям
const errors = reactive({
  rating: '',
  comment: ''
})

// --- Открытие / закрытие модального окна ---
function openModal() {
  isOpen.value = true
  // Переводим фокус на overlay, чтобы работало закрытие по Esc
  nextTick(() => {
    overlayRef.value?.focus()
  })
}

function closeModal() {
  isOpen.value = false
  // Сбрасываем только ошибки, введённые данные сохраняем
  errors.rating = ''
  errors.comment = ''
}

// Закрытие по Esc: дополнительный слушатель на document,
// так как focus на overlay может теряться.
watch(isOpen, (open) => {
  if (open) {
    document.addEventListener('keydown', onKeydown)
  } else {
    document.removeEventListener('keydown', onKeydown)
  }
})

function onKeydown(e) {
  if (e.key === 'Escape') {
    closeModal()
  }
}

// --- Работа с оценкой ---
function setRating(value) {
  rating.value = value
  if (errors.rating) errors.rating = ''
}

// --- Валидация ---
function validate() {
  let valid = true
  errors.rating = ''
  errors.comment = ''

  // Оценка обязательна
  if (!rating.value || rating.value < 1) {
    errors.rating = 'Пожалуйста, выберите оценку от 1 до 5'
    valid = false
  }

  // Комментарий не обязателен, но не более 500 символов
  if (comment.value.length > 500) {
    errors.comment = 'Комментарий не должен превышать 500 символов'
    valid = false
  }

  return valid
}

// --- Отправка формы ---
async function submitForm() {
  // Валидация на стороне клиента
  if (!validate()) {
    return
  }

  submitting.value = true
  try {
    // Имитация сохранения отзыва в базе данных.
    // В реальном приложении здесь будет запрос к API, например:
    // await api.post('/feedback', { rating: rating.value, comment: comment.value })
    await new Promise((resolve) => setTimeout(resolve, 400))

    // Успешная отправка: показываем toast, закрываем окно,
    // показываем текст «Ваш отзыв отправлен».
    isOpen.value = false
    sent.value = true
    toastVisible.value = true

    // Автоскрытие toast через 3 секунды
    setTimeout(() => {
      toastVisible.value = false
    }, 3000)

    // Сброс полей для следующего раза
    rating.value = 0
    comment.value = ''
  } finally {
    submitting.value = false
  }
}
</script>

<style scoped>
/* Кнопка открытия формы */
.btn-open {
  display: inline-block;
  padding: 10px 22px;
  background: #2b7a4b;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}
.btn-open:hover {
  background: #22613c;
}

/* Состояние «Ваш отзыв отправлен» */
.feedback-sent {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 18px;
  background: #e8f5e9;
  color: #2b7a4b;
  border: 1px solid #a5d6a7;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 600;
}
.sent-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 22px;
  height: 22px;
  background: #2b7a4b;
  color: #fff;
  border-radius: 50%;
  font-size: 14px;
}

/* Overlay — затемнённый фон */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 16px;
  outline: none;
}

/* Окно модалки */
.modal-window {
  position: relative;
  width: 100%;
  max-width: 460px;
  max-height: 90vh;
  overflow-y: auto;
  background: #fff;
  border-radius: 12px;
  padding: 28px 24px 24px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.25);
}

/* Кнопка закрытия */
.modal-close {
  position: absolute;
  top: 10px;
  right: 12px;
  width: 36px;
  height: 36px;
  background: transparent;
  border: none;
  font-size: 26px;
  line-height: 1;
  color: #666;
  cursor: pointer;
  border-radius: 6px;
}
.modal-close:hover {
  background: #f0f0f0;
  color: #333;
}

.modal-title {
  margin: 0 0 4px;
  font-size: 22px;
  font-weight: 700;
  color: #1a1a1a;
}
.modal-subtitle {
  margin: 0 0 20px;
  font-size: 14px;
  color: #666;
}

/* Форма */
.feedback-form {
  display: flex;
  flex-direction: column;
  gap: 18px;
}
.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.form-label {
  font-size: 14px;
  font-weight: 600;
  color: #333;
}
.required {
  color: #d32f2f;
}

/* Звёзды */
.stars {
  display: flex;
  gap: 6px;
}
.star {
  width: 44px;
  height: 44px;
  background: #f5f5f5;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  font-size: 24px;
  line-height: 1;
  color: #ccc;
  cursor: pointer;
  transition: all 0.15s;
}
.star.hover,
.star.active {
  color: #ffb300;
  border-color: #ffb300;
  background: #fff8e1;
}

/* Текстовое поле */
.form-textarea {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 14px;
  font-family: inherit;
  resize: vertical;
  box-sizing: border-box;
}
.form-textarea:focus {
  outline: none;
  border-color: #2b7a4b;
  box-shadow: 0 0 0 3px rgba(43, 122, 75, 0.15);
}

.comment-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
}
.char-counter {
  color: #999;
  margin-left: auto;
}

/* Сообщение об ошибке */
.error-text {
  color: #d32f2f;
  font-size: 12px;
}

/* Кнопка отправки */
.btn-submit {
  padding: 12px 20px;
  background: #2b7a4b;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}
.btn-submit:hover:not(:disabled) {
  background: #22613c;
}
.btn-submit:disabled {
  background: #9bbfa9;
  cursor: not-allowed;
}

/* Анимация появления */
.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.2s;
}
.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}

/* === Адаптивность для мобильных устройств (390×844) === */
@media (max-width: 640px) {
  .modal-overlay {
    padding: 0;
    align-items: flex-end;
  }
  .modal-window {
    max-width: 100%;
    width: 100%;
    max-height: 92vh;
    border-radius: 16px 16px 0 0;
    padding: 24px 18px 22px;
  }
  .modal-title {
    font-size: 20px;
  }
  /* Элементы формы крупнее — удобно нажимать пальцем */
  .star {
    width: 52px;
    height: 52px;
    font-size: 28px;
  }
  .btn-submit,
  .btn-open {
    padding: 14px 20px;
    font-size: 16px;
  }
  .modal-close {
    width: 42px;
    height: 42px;
    font-size: 30px;
  }
  .form-textarea {
    font-size: 16px; /* чтобы iOS не масштабировал страницу при фокусе */
  }
}
</style>
