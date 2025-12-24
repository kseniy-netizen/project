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
        <div class="form-card">
            <h1 class="form-title">Форма регистрации</h1>

            <form @submit.prevent="onSubmit" novalidate class="form">
                <!-- Имя -->
                <div class="form-group">
                    <label for="name" class="form-label">Имя</label>
                    <input type="text" id="name" v-model.trim="form.name" @blur="onBlur('name')"
                        :class="inputClass('name')" required minlength="2" placeholder="Иван" />
                    <div v-if="touched.name && errors.name" class="error-message">
                        {{ errors.name }}
                    </div>
                </div>

                <!-- Email -->
                <div class="form-group">
                    <label for="email" class="form-label">Email</label>
                    <input type="email" id="email" v-model.trim="form.email" @blur="onBlur('email')"
                        :class="inputClass('email')" required placeholder="user@example.com" />
                    <div v-if="touched.email && errors.email" class="error-message">
                        {{ errors.email }}
                    </div>
                </div>

                <!-- Возраст -->
                <div class="form-group">
                    <label for="age" class="form-label">Возраст</label>
                    <input type="number" id="age" v-model.number="form.age" @blur="onBlur('age')"
                        :class="inputClass('age')" required min="16" placeholder="например, 18" />
                    <small v-if="form.age !== undefined && form.age !== ''">
                        Тип данных: {{ typeof form.age }}
                    </small>
                    <small v-else-if="form.age === ''" class="warning">
                        Введите число
                    </small>
                    <div v-if="touched.age && errors.age" class="error-message">
                        {{ errors.age }}
                    </div>
                </div>
                <!-- Роль -->
                <div class="form-group">
                    <label for="role" class="form-label">Роль</label>
                    <select id="role" v-model="form.role" @blur="onBlur('role')" :class="inputClass('role')" required>
                        <option value="" disabled>Выберите роль</option>
                        <option value="student">Студент</option>
                        <option value="mentor">Наставник</option>
                        <option value="employer">Работодатель</option>
                    </select>
                    <div v-if="touched.role && errors.role" class="error-message">
                        {{ errors.role }}
                    </div>
                </div>

                <!-- Интересы -->
                <div class="form-group">
                    <label class="form-label">Интересы</label>
                    <fieldset @blur.capture="onBlur('interests')" :class="['interests-group', inputClass('interests')]">
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
                    <input type="password" id="password" v-model.lazy="form.password" @blur="onBlur('password')"
                        :class="inputClass('password')" required minlength="8" placeholder="минимум 8 символов" />
                    <div v-if="touched.password && errors.password" class="error-message">
                        {{ errors.password }}
                    </div>
                </div>

                <!-- Подтверждение пароля -->
                <div class="form-group">
                    <label for="password2" class="form-label">Подтверждение пароля</label>
                    <input type="password" id="password2" v-model.lazy="form.password2" @blur="onBlur('password2')"
                        :class="inputClass('password2')" required minlength="8" placeholder="повторите пароль" />
                    <div v-if="touched.password2 && errors.password2" class="error-message">
                        {{ errors.password2 }}
                    </div>
                </div>

                <!-- Комментарий -->
                <div class="form-group">
                    <label for="comment" class="form-label">Комментарий (опционально)</label>
                    <textarea id="comment" v-model.trim="form.comment" @blur="onBlur('comment')" class="form-control"
                        rows="3" placeholder="Короткое сообщение"></textarea>
                </div>

                <!-- Согласие -->
                <div class="form-group">
                    <label class="checkbox-label">
                        <input type="checkbox" v-model="form.agree" @blur="onBlur('agree')" />
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

            <!-- Данные для самопроверки -->
            <pre class="debug">{{ form }}</pre>
        </div>
    </div>
</template><style scoped>
.form-container {
    padding: 30px;
    min-height: 100vh;
    background: #f8fafc;
}

.form-card {
    max-width: 600px;
    margin: 0 auto;
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
    padding: 30px;
}

.form-title {
    text-align: center;
    margin-bottom: 25px;
    color: #1e293b;
    font-size: 24px;
}

.form {
    display: grid;
    gap: 20px;
}

.form-group {
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.form-label {
    font-weight: 500;
    color: #374151;
    font-size: 14px;
}

.form-control {
    padding: 10px 14px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    font-size: 14px;
    transition: border-color 0.2s;
}

.form-control:focus {
    outline: none;
    border-color: #3b82f6;
}

.form-control.is-invalid {
    border-color: #dc2626;
}

.form-control.is-valid {
    border-color: #16a34a;
}

.interests-group {
    border: 1px solid #d1d5db;
    border-radius: 6px;
    padding: 15px;
    margin: 0;
}

.interests-group.is-invalid {
    border-color: #dc2626;
}

.checkbox-grid {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-top: 8px;
}

.checkbox-label {
    display: flex;
    align-items: center;
    gap: 10px;
    cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
    width: 16px;
    height: 16px;
}

.submit-btn {
    background-color: #3b82f6;
    color: white;
    padding: 12px 24px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 500;
    font-size: 16px;
    transition: background-color 0.2s;
    margin-top: 10px;
}

.submit-btn:hover {
    background-color: #2563eb;
}

.error-message {
    color: #dc2626;
    font-size: 13px;
    margin-top: 4px;
}

.warning {
    color: #d97706;
    font-size: 13px;
}

.debug {
    background: #f3f4f6;
    padding: 20px;
    border-radius: 6px;
    margin-top: 30px;
    font-size: 13px;
    border: 1px solid #e5e7eb;
    overflow-x: auto;
}
</style>