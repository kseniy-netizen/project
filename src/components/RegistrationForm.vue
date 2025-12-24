<script setup>
import { reactive, computed } from 'vue'

// ЗАДАНИЕ 1: Контролируемые поля
const form = reactive({
  name: '',
  email: '',
  age: undefined,
  role: 'student',
  interests: [],
  password: '',
  password2: '',
  comment: '',
  agree: false
})

// ЗАДАНИЕ 2: Стратегия показа ошибок
const touched = reactive({
  name: false,
  email: false,
  age: false,
  role: false,
  interests: false,
  password: false,
  password2: false,
  comment: false,
  agree: false
})

function onBlur(field) {
  touched[field] = true
}

// ЗАДАНИЕ 3: Валидация
const reEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

const errors = computed(() => {
  return {
    name: form.name.trim().length >= 2 ? '' : 'Имя должно содержать минимум 2 символа',
    email: form.email.trim() ? 
      (reEmail.test(form.email) ? '' : 'Неверный формат email') : 
      'Укажите email',
    age: form.age === null || form.age === '' ? 
      'Укажите возраст' : 
      (typeof form.age === 'number' && form.age >= 16 ? '' : 'Возраст должен быть 16+'),
    role: form.role ? '' : 'Выберите роль',
    interests: form.interests.length > 0 ? '' : 'Выберите хотя бы один интерес',
    password: form.password.length >= 8 ? '' : 'Пароль — минимум 8 символов',
    password2: form.password === form.password2 ? '' : 'Пароли не совпадают',
    agree: form.agree ? '' : 'Необходимо согласие с правилами'
  }
})

const isValid = computed(() => {
  return Object.values(errors.value).every(error => error === '')
})

// ЗАДАНИЕ 4: Подсветка полей
function inputClass(field) {
  if (!touched[field]) return 'form-control'
  return errors.value[field] ? 'form-control is-invalid' : 'form-control is-valid'
}

// ЗАДАНИЕ 6: Поведение submit
function onSubmit(e) {
  e.preventDefault()
  
  // Помечаем все поля как touched
  Object.keys(touched).forEach(k => {
    touched[k] = true
  })
  
  if (!isValid.value) {
    console.log('Ошибки валидации:', errors.value)
    return
  }
  
  alert('Форма валидна! Данные готовы к отправке в API.')
  console.log('Данные формы:', JSON.stringify(form, null, 2))
}
</script>

<template>
  <div class="form-container">
    <h1 class="form-title">Форма регистрации</h1>
    
    <form @submit.prevent="onSubmit" novalidate class="form">
      <!-- Имя -->
      <div class="form-group">
        <label for="name" class="form-label">Имя</label>
        <input
          type="text"
          id="name"
          v-model.trim="form.name"
          @blur="onBlur('name')"
          :class="inputClass('name')"
          required
          minlength="2"
          placeholder="Иван"
        />
        <div v-if="touched.name && errors.name" class="error-message">
          {{ errors.name }}
        </div>
      </div>

      <!-- Email -->
      <div class="form-group">
        <label for="email" class="form-label">Email</label>
        <input
          type="email"
          id="email"
          v-model.trim="form.email"
          @blur="onBlur('email')"
          :class="inputClass('email')"
          required
          placeholder="user@example.com"
        />
        <div v-if="touched.email && errors.email" class="error-message">
          {{ errors.email }}
        </div>
      </div>

           <!-- Возраст -->
            <div class="form-group">
                <label for="age" class="form-label">Возраст</label>
                <input type="number" id="age" v-model.number="form.age" @blur="onBlur('age')" :class="inputClass('age')"
                    required min="16" placeholder="например, 18" />

                <!-- ПРАВИЛЬНЫЙ ВЫВОД ТИПА: -->
                <small v-if="form.age !== undefined && form.age !== ''">
                    Тип данных: {{ typeof form.age }}
                </small>
                <small v-else-if="form.age === ''" style="color: #f59e0b;">
                    Введите число
                </small>

                <div v-if="touched.age && errors.age" class="error-message">
                    {{ errors.age }}
                </div>
            </div>

      <!-- Роль -->
       <div class="form-group">
        <label for="role" class="form-label">Роль</label>
        <select
          id="role"
          v-model="form.role"
          @blur="onBlur('role')"
          :class="inputClass('role')"
          required
        >
          <option value="" disabled>Выберите роль</option>
          <option value="student">Студент</option>
          <option value="mentor">Наставник</option>
          <option value="employer">Работодатель</option>
        </select>
        <div v-if="touched.role && errors.role" class="error-message">
          {{ errors.role }}
        </div>
</div>

<!-- Интересы (ЗАДАНИЕ 5: Групповые поля) -->
<div class="form-group">
        <label class="form-label">Интересы</label>
        <fieldset 
          @blur.capture="onBlur('interests')"
          :class="['interests-group', inputClass('interests')]"
        >
          <legend>Интересы (выберите минимум один)</legend>
          <div class="checkbox-grid">
            <label class="checkbox-label">
              <input type="checkbox" value="frontend" v-model="form.interests" />
              <span>Frontend</span>
            </label>
            <label class="checkbox-label">
              <input type="checkbox" value="backend" v-model="form.interests" />
              <span>Backend</span>
            </label>
            <label class="checkbox-label">
              <input type="checkbox" value="devops" v-model="form.interests" />
              <span>DevOps</span>
            </label>
          </div>
</fieldset>
<div v-if="touched.interests && errors.interests" class="error-message">
          {{ errors.interests }}
        </div>
</div>

<!-- Пароль -->
<div class="form-group">
        <label for="password" class="form-label">Пароль</label>
        <input
          type="password"
          id="password"
          v-model.lazy="form.password"
          @blur="onBlur('password')"
          :class="inputClass('password')"
          required
          minlength="8"
          placeholder="минимум 8 символов"
        />
        <div v-if="touched.password && errors.password" class="error-message">
          {{ errors.password }}
        </div>
</div>

<!-- Подтверждение пароля -->
<div class="form-group">
        <label for="password2" class="form-label">Подтверждение пароля</label>
        <input
          type="password"
          id="password2"
          v-model.lazy="form.password2"
          @blur="onBlur('password2')"
          :class="inputClass('password2')"
          required
          minlength="8"
          placeholder="повторите пароль"
        />
        <div v-if="touched.password2 && errors.password2" class="error-message">
          {{ errors.password2 }}
        </div>
</div>

<!-- Комментарий -->
<div class="form-group">
        <label for="comment" class="form-label">Комментарий (опционально)</label>
        <textarea
          id="comment"
          v-model.trim="form.comment"
          @blur="onBlur('comment')"
          class="form-control"
          rows="3"
          placeholder="Короткое сообщение"
        ></textarea>
      </div>

<!-- Согласие -->
<div class="form-group">
        <label class="checkbox-label">
          <input
            type="checkbox"
            v-model="form.agree"
            @blur="onBlur('agree')"
          />
          <span>Я согласен с правилами</span>
        </label>
        <div v-if="touched.agree && errors.agree" class="error-message">
          {{ errors.agree }}
        </div>
</div>

<!-- Кнопка отправки -->
<button type="submit" class="submit-btn">
        Отправить
      </button>
</form>

<!-- ЗАДАНИЕ: Вывод объекта данных для самопроверки -->
<pre class="debug">{{ form }}</pre>
</div>
</template>

<style scoped>
.form-container {
    max-width: 520px;
    margin: 0 auto;
    padding: 1rem;
}

.form-title {
    margin-bottom: 1.5rem;
}

.form {
    display: grid;
    gap: 1rem;
}
.form-group {
display: flex;
flex-direction: column;
gap: 0.25rem;
}

.form-label {
font-weight: 500;
}

.form-control {
padding: 0.5rem;
border: 1px solid #cbd5e1;
border-radius: 0.5rem;
}

.form-control.is-invalid {
border-color: #e74444;
}

.form-control.is-valid {
border-color: #22c55e;
}

.interests-group {
border: 1px solid #cbd5e1;
border-radius: 0.5rem;
padding: 0.75rem;
}

.interests-group.is-invalid {
border-color: #e74444;
}

.checkbox-grid {
display: flex;
flex-direction: column;
gap: 0.5rem;
margin-top: 0.5rem;
}

.checkbox-label {
display: flex;
align-items: center;
gap: 0.5rem;
}

.submit-btn {
background-color: #3b82f6;
color: white;
padding: 0.75rem;
border: none;
border-radius: 0.5rem;
cursor: pointer;
}

.error-message {
color: #b91c1c;
font-size: 0.9rem;
}

.debug {
background: #f1f5f9;
padding: 1rem;
border-radius: 0.5rem;
margin-top: 1.5rem;
font-size: 0.875rem;
}
</style>