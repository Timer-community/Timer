<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Luna — AI‑ассистент</title>
    <style>
        * {margin:0;padding:0;box-sizing:border-box}
        body {font-family:sans-serif;background:#667eea;height:100vh;display:flex;justify-content:center;align-items:center}
        .chat {width:90%;max-width:800px;height:90vh;background:white;border-radius:12px;display:flex;flex-direction:column}
        .msgs {flex-grow:1;padding:20px;overflow-y:auto;display:flex;flex-direction:column;gap:15px}
        .msg {max-width:80%;padding:12px 18px;border-radius:18px}
        .user {align-self:flex-end;background:#3f51b5;color:white}
        .ai {align-self:flex-start;background:#e3f2fd}
        .input {display:flex;padding:15px;border-top:1px solid #e0e0e0}
        #inp {flex-grow:1;padding:12px;border:1px solid #bbdefb;border-radius:25px;outline:none}
        button {padding:10px 20px;border:none;border-radius:25px;cursor:pointer}
        .auth-modal {display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5)}
        .auth-form {background:white;margin:20% auto;width:80%;max-width:400px;padding:20px;border-radius:10px}
        .train-modal {display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.5)}
        .train-form {background:white;margin:15% auto;width:80%;max-width:500px;padding:20px;border-radius:10px}
    </style>
</head>
<body>
    <!-- Модальное окно авторизации -->
    <div id="auth-modal" class="auth-modal">
        <div class="auth-form">
            <h3 id="auth-title">Вход в аккаунт</h3>
            <form id="auth-form">
                <input type="text" id="username" placeholder="Логин" required style="width:100%;padding:8px;margin:5px 0">
                <input type="password" id="password" placeholder="Пароль" required style="width:100%;padding:8px;margin:5px 0">
                <div style="text-align:right;margin-top:15px">
                    <button type="button" id="switch-auth">Регистрация</button>
            <button type="submit">Войти</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Модальное окно обучения -->
    <div id="train-modal" class="train-modal">
        <div class="train-form">
            <h3>Обучить Luna</h3>
            <label>Вопрос (ключевые слова):<br>
                <input type="text" id="train-question" style="width:100%;padding:8px;margin:5px 0" placeholder="привет, здравствуй">
            </label>
            <label>Ответ ассистента:<br>
                <textarea id="train-answer" style="width:100%;padding:8px;margin:5px 0;height:80px" placeholder="Здравствуйте! Чем могу помочь?"></textarea>
            </label>
            <!-- ДОБАВЛЕНО: кнопка сохранения обучений на аккаунте -->
            <div style="text-align:right;margin-top:15px">
                <button type="button" id="cancel-train">Отмена</button>
                <button type="button" id="save-train">Сохранить обучение</button>
                <button type="button" id="export-train">Сохранить на аккаунт</button>
            </div>
        </div>
    </div>

    <div class="chat">
        <div style="background:#3f51b5;color:white;padding:15px;display:flex;justify-content:space-between;align-items:center">
            <div style="font-size:1.3rem;font-weight:bold">Luna</div>
            <div id="user-status" style="color:white;font-size:.9rem"></div>
        </div>
        <div class="msgs" id="msgs">
            <div class="msg ai" style="text-align:center;color:#757575;">Войдите в аккаунт, чтобы начать чат с Luna.</div>
        </div>
        <div class="input">
            <input type="text" id="inp" placeholder="Введите сообщение..." disabled>
            <button id="send" disabled>Отправить</button>
        </div>
        <!-- Кнопка «Обучить Luna» -->
        <button id="train-btn" disabled style="position:absolute;top:10px;right:60px;">Обучить Luna</button>
        <!-- Кнопка входа/выхода -->
        <button id="auth-toggle-btn" style="position:absolute;top:10px;left:10px;">Войти</button>
    </div>

    <script>
        // Получаем все элементы DOM
        const msgs = document.getElementById('msgs');
        const inp = document.getElementById('inp');
        const sendBtn = document.getElementById('send');
        const authModal = document.getElementById('auth-modal');
        const trainModal = document.getElementById('train-modal');
        const userStatus = document.getElementById('user-status');
        const usernameInput = document.getElementById('username');
        const passwordInput = document.getElementById('password');
        const switchAuthBtn = document.getElementById('switch-auth');
        const authForm = document.getElementById('auth-form');
        const trainBtn = document.getElementById('train-btn');
        const authToggleBtn = document.getElementById('auth-toggle-btn');
        // ДОБАВЛЕНО: получаем кнопку экспорта обучений
        const exportTrainBtn = document.getElementById('export-train');

        let currentUser = null;
        let isLoginMode = true;

        // База знаний ассистента (глобальная + пользовательская)
        const globalKB = [
            {k: ['привет', 'здравствуй'], r: 'Здравствуйте! Я Luna**Лу́на)**. Чем могу помочь?' },
            {k: ['как дела'], r: 'У меня всё отлично, спасибо! А у вас?' },
            {k: ['что умеешь'], r: 'Я могу отвечать на вопросы, помогать с задачами и учиться новому!' },
            {k: ['пока'], r: 'До свидания! Буду рада помочь снова!' }
        ];

        // Инициализация — проверяем сохранённый аккаунт
        if (localStorage.getItem('currentUser')) {
            currentUser = JSON.parse(localStorage.getItem('currentUser'));
            updateUI();
        } else {
            updateAuthButton();
        }

        function showAuthModal() {
            authModal.style.display = 'block';
            document.getElementById('auth-title').textContent = isLoginMode ? 'Вход' : 'Регистрация';
        }

        function hideAuthModal() {
            authModal.style.display = 'none';
            usernameInput.value = '';
            passwordInput.value = '';
        }

        function showTrainModal() {
            trainModal.style.display = 'block';
        }

        function hideTrainModal() {
            trainModal.style.display = 'none';
            document.getElementById('train-question').value = '';
            document.getElementById('train-answer').value = '';
        }

        // Обработчик переключения между входом и регистрацией
        switchAuthBtn.addEventListener('click', () => {
            isLoginMode = !isLoginMode;
            showAuthModal();
        });

                // Обработчик формы авторизации
        authForm.addEventListener('submit', function(e) {
            e.preventDefault();
            const username = usernameInput.value;
            const password = passwordInput.value;

            if (isLoginMode) {
                // Проверка логина
                const users = JSON.parse(localStorage.getItem('users') || '[]');
                const user = users.find(u => u.username === username && u.password === password);
                if (user) {
                    currentUser = user;
            localStorage.setItem('currentUser', JSON.stringify(user));
            hideAuthModal();
            updateUI();
        } else {
            alert('Неверный логин или пароль');
        }
    } else {
        // Регистрация нового пользователя
        const users = JSON.parse(localStorage.getItem('users') || '[]');
        if (users.some(u => u.username === username)) {
            alert('Пользователь уже существует');
            return;
        }
        const newUser = {
            username,
            password,
            history: [],
            userKB: [] // пользовательская база знаний
        };
        users.push(newUser);
        localStorage.setItem('users', JSON.stringify(users));
        currentUser = newUser;
        localStorage.setItem('currentUser', JSON.stringify(newUser));
        hideAuthModal();
        updateUI();
    }
});

function updateUI() {
    userStatus.textContent = `Пользователь: ${currentUser.username}`;
    inp.disabled = false;
    sendBtn.disabled = false;
    trainBtn.disabled = false;
    authToggleBtn.textContent = 'Выйти';

    // Загружаем историю чата
    if (currentUser.history && currentUser.history.length > 0) {
        msgs.innerHTML = '';
        currentUser.history.forEach(msg => addMsg(msg.text, msg.isUser));
    } else {
        msgs.innerHTML = '<div class="msg ai" style="text-align:center;color:#757575;">Начните общение с Luna!</div>';
    }

    // ДОБАВЛЕНО: загружаем пользовательские обучения при авторизации
    if (currentUser.userKB && currentUser.userKB.length > 0) {
        updateUserKB();
        console.log('Пользовательские обучения загружены:', currentUser.userKB.length, 'правил');
    }

    // Обновляем базу знаний пользователя
    updateUserKB();
}

function updateAuthButton() {
    if (currentUser) {
        authToggleBtn.textContent = 'Выйти';
    } else {
        authToggleBtn.textContent = 'Войти';
    }
}

// Функция выхода из аккаунта
function logout() {
    localStorage.removeItem('currentUser');
    currentUser = null;
    userStatus.textContent = '';
    inp.disabled = true;
    sendBtn.disabled = true;
    trainBtn.disabled = true;
    msgs.innerHTML = '<div class="msg ai" style="text-align:center;color:#757575;">Вы вышли из аккаунта. Войдите снова, чтобы продолжить чат.</div>';
    updateAuthButton();
}

authToggleBtn.addEventListener('click', () => {
    if (currentUser) {
        logout();
    } else {
        showAuthModal();
    }
});

function updateUserKB() {
    // Объединяем глобальную и пользовательскую базы знаний
    currentKB = [...globalKB, ...(currentUser.userKB || [])];
}

let currentKB = [];

// ДОБАВЛЕНО: функция экспорта обучений на аккаунт
function exportTrainingsToAccount() {
    if (!currentUser) {
        alert('Сначала войдите в аккаунт!');
        return;
    }

    // Получаем всех пользователей из localStorage
    const users = JSON.parse(localStorage.getItem('users') || '[]');

    // Находим текущего пользователя в списке
    const userIndex = users.findIndex(u => u.username === currentUser.username);

    if (userIndex !== -1) {
        // Обновляем пользовательскую базу знаний
        users[userIndex].userKB = currentUser.userKB || [];

        // Сохраняем обновлённый список пользователей
        localStorage.setItem('users', JSON.stringify(users));

        alert('Обучения успешно сохранены на аккаунте!');
    } else {
        alert('Ошибка: пользователь не найден!');
    }
}

// Функция добавления обучения
function addTraining(question, answer) {
    const keywords = question.split(',').map(k => k.trim().toLowerCase());
    const newEntry = {k: keywords, r: answer};

    currentUser.userKB = currentUser.userKB || [];
    currentUser.userKB.push(newEntry);

    // Сохраняем в текущий сеанс
    localStorage.setItem('currentUser', JSON.stringify(currentUser));
    updateUserKB(); // Обновляем текущую базу знаний

    alert('Обучение успешно сохранено в текущем сеансе!');
}

// Получение ответа ассистента с учётом пользовательских обучений
function getResp(msg) {
    const lMsg = msg.toLowerCase();
    for (const item of currentKB) {
        if (item.k.some(k => lMsg.includes(k))) return item.r;
    }
    return 'Не совсем поняла ваш вопрос. Уточните, пожалуйста!';
}

// Обработчики для обучения
trainBtn.addEventListener('click', showTrainModal);

document.getElementById('cancel-train').addEventListener('click', hideTrainModal);

// ДОБАВЛЕНО: обработчик для кнопки экспорта обучений
exportTrainBtn.addEventListener('click', exportTrainingsToAccount);

document.getElementById('save-train').addEventListener('click', () => {
    const question = document.getElementById('train-question').value.trim();
    const answer = document.getElementById('train-answer').value.trim();


    if (!question || !answer) {
        alert('Заполните оба поля!');
        return;
    }

    addTraining(question, answer);
    hideTrainModal();
});

// Обработка закрытия модального окна обучения при клике вне формы
window.addEventListener('click', e => {
    if (e.target === trainModal) {
        hideTrainModal();
    }
});

function addMsg(txt, isUser) {
    const div = document.createElement('div');
    div.className = `msg ${isUser ? 'user' : 'ai'}`;
    div.textContent = txt;
    msgs.appendChild(div);
    msgs.scrollTop = msgs.scrollHeight;

    // Сохраняем в историю
    if (currentUser) {
        currentUser.history = currentUser.history || [];
        currentUser.history.push({text: txt, isUser});
        localStorage.setItem('currentUser', JSON.stringify(currentUser));
    }
}

// Улучшение: очистка ввода при нажатии Esc
inp.addEventListener('keydown', e => {
    if (e.key === 'Escape') {
        inp.value = '';
    }
});

// Дополнительная защита: блокировка повторной отправки во время обработки
let isProcessing = false;

sendBtn.addEventListener('click', () => {
    if (isProcessing) return;

    const txt = inp.value.trim();
    if (!txt) return;

    isProcessing = true;
    addMsg(txt, true);

    // Имитация задержки ответа ассистента
    setTimeout(() => {
        addMsg(getResp(txt), false);
        isProcessing = false;
    }, 800); // Увеличили задержку до 800 мс для реалистичности

    inp.value = '';
});

// Отправка по Enter
inp.addEventListener('keypress', e => {
    if (e.key === 'Enter') sendBtn.click();
});

// Инициализация при загрузке
document.addEventListener('DOMContentLoaded', () => {
    updateAuthButton();
});
</script>
</body>
</html>
