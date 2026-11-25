<template>
  <!-- Скелет компонента формы обратной связи.
       Реализация функционала будет добавлена в следующем коммите. -->
  <div class="feedback-modal">
    <div class="modal-overlay">
      <div class="modal-window">
        <button class="modal-close" type="button">×</button>
        <h2 class="modal-title">Оставить отзыв</h2>

        <!-- TODO: поле оценки (5 звёзд) -->
        <div class="form-group">
          <label>Оценка мероприятия</label>
        </div>

        <!-- TODO: поле комментария (textarea, макс. 500 символов) -->
        <div class="form-group">
          <label>Комментарий</label>
        </div>

        <!-- TODO: кнопка отправки -->
        <button class="btn-submit" type="button">Отправить отзыв</button>
      </div>
    </div>
  </div>
</template>

<script setup>
// Начальная заготовка компонента FeedbackModal.
// Структура шаблона размечена, логика будет реализована далее.
</script>

<style scoped>
.feedback-modal {
  /* стили будут добавлены позже */
}
</style>
