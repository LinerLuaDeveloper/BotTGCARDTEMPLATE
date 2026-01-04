import sqlite3
import telebot
from telebot import types
import time
import random
import logging
from datetime import datetime, timedelta
import re
import json
import os
from collections import defaultdict

# Настройка логирования
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

bot = telebot.TeleBot('8205728685:AAEX0xtuHGthCb4ZYy0i18CqA8DGymkNtPs')

# Время запуска бота (для игнорирования старых сообщений)
BOT_START_TIME = time.time()

# Файлы для сохранения данных
CARDS_DB_FILE = 'cards_database.json'
HIDDEN_CARDS_FILE = 'hidden_cards.json'
SHOP_STATUS_FILE = 'shop_status.json'
CRAFT_RECIPES_FILE = 'craft_recipes.json'

# Сначала определяем переменные как глобальные
CARDS_DATABASE = {}
HIDDEN_CARDS = set()
SHOP_ENABLED = True  # По умолчанию магазин включен
CRAFT_RECIPES = {}  # Рецепты крафта

# Редкости карточек и их стоимости
RARITIES = {
    "🟤": {"name": "Обычная", "coins": 1, "chance": 30},
    "⚪️": {"name": "Необычная", "coins": 3, "chance": 25},
    "🟢": {"name": "Редкая", "coins": 5, "chance": 20},
    "🟠": {"name": "Супер редкая", "coins": 7, "chance": 10},
    "🟣": {"name": "Эпическая", "coins": 10, "chance": 8},
    "🟡": {"name": "Легендарная", "coins": 50, "chance": 4},
    "🔴": {"name": "Мифическая", "coins": 70, "chance": 2},
    "💎": {"name": "Алмазная", "coins": 100, "chance": 0.5},
    "👑": {"name": "Божественная", "coins": 500, "chance": 0.3},
    "❔": {"name": "Секретная", "coins": 1000, "chance": 0.2},
    "🖥️": {"name": "Админская", "coins": 1777, "chance": 0}
}

# Инициализация базы карточек по умолчанию
DEFAULT_CARDS = {
    "Инвертированный Симба": {
        "rarity": "⚪️",
        "coins": 3,
        "description": "Кот инвертированный в противоположные цвета.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/3627c1357b3564d8ad82e3543d7c0513/inverssimba.jpg",
        "craft_only": False
    },
    "Grow а Симба, Steal a Алиса": {
        "rarity": "🟣",
        "coins": 10,
        "description": "Коты попали в роблокс игры!",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/0be979231af670f1c11e76996a1385ac/growandstealsimba.jpg",
        "craft_only": False
    },
    "Лайнер": {
        "rarity": "🖥️",
        "coins": 1777,
        "description": "Это лайнер. Ну да, лучшая карточка в боте, что такого? Вам просто повезло, ничего особенного.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/028e0013f79f399c59713d06789c0a89/liner.jpg",
        "craft_only": False
    },
    "Симба на миссии": {
        "rarity": "👑",
        "coins": 500,
        "description": "Кот-агент 😎",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/9b6889f07b9329cee41340fb0664b766/simbainmission.jpg",
        "craft_only": False
    },
    "Симба грабитель": {
        "rarity": "⚪️",
        "coins": 3,
        "description": "Ии ультануло. Кот грабитель.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/ecce48ba328c1250f640150053944a10/simbarobber.jpg",
        "craft_only": False
    },
    "Симба удивлён": {
        "rarity": "⚪️",
        "coins": 3,
        "description": "Просто удивлённый кот, скорее всего он увидел мотылька.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/b3e302e21aeeb6adcec0245d9c13ed18/simbawow.jpg",
        "craft_only": False
    },
    "Симба злой король": {
        "rarity": "🔴",
        "coins": 70,
        "description": "Кот который выглядит жутковато..",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/81733e770f94b9e8a65ff5567341e673/simbaking.jpg",
        "craft_only": False
    },
    "Симбакула": {
        "rarity": "🟢",
        "coins": 5,
        "description": "Рыба?.. АКУЛА! А, это всё таки кот..",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/df2807cffe2f3cd124aadab2f6d76884/simbashark.jpg",
        "craft_only": False
    },
    "Симба с Алисой на окне": {
        "rarity": "🟠",
        "coins": 7,
        "description": "Две кошки вместе.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/25330c0dee36510e4fc593b76dc9eb79/simbawithalisa.jph.jpg",
        "craft_only": False
    },
    "Симба лежит на земле": {
        "rarity": "🟣",
        "coins": 10,
        "description": "Кот который охотится на птиц в кормушке.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/d353493d5bc7c6ae849b03c935dc6fd8/simbasitingrass.jpg",
        "craft_only": False
    },
    "Симба-клоун": {
        "rarity": "🔴",
        "coins": 100,
        "description": "Кот-клоун. Буквально каждый из нас похож на него.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/39d1c6d31d9adace2338f12e74fd3364/clownsimba.jpg",
        "craft_only": False
    },
    "Симба полицейский": {
        "rarity": "🟡",
        "coins": 50,
        "description": "Кот-полицейский, настоящий страж порядка!",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/26132586be78a56dbb0cd51c4a5c3696/simbapolice.jpg",
        "craft_only": False
    },
    "Симба с рыбкой": {
        "rarity": "🟤",
        "coins": 1,
        "description": "Кот с пластиковой рыбкой.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/c7568dfac2c49ca6335e493e2bb8a597/simbawithfish.jpg",
        "craft_only": False
    },
    "Алиса-убийца": {
        "rarity": "🟣",
        "coins": 10,
        "description": "Кошка с острым ножом.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/7f8a704aa7184492f6a0ea57e2246410/alicekiller.jpg",
        "craft_only": False
    },
    "Злая и Добрая Алиса": {
        "rarity": "🔴",
        "coins": 70,
        "description": "Две светящиеся кошки. Вроде бы они одинаковые, а вроде и нет..",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/71b056bc6b7f24f4dbc7e436f63d511d/godandevilsimba.jpg",
        "craft_only": False
    },
    "Пиксельная Алиса": {
        "rarity": "🟠",
        "coins": 7,
        "description": "Кошка немножко пиксель.",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/fe260aa01b4a3fd3567ad30f9a84f65b/pixelalisa.jpg",
        "craft_only": False
    },
    "Алиса-красотка": {
        "rarity": "🟡",
        "coins": 50,
        "description": "Кошка-красотка. Целуйте экраны>:)",
        "image": "https://sprutio.beget.com/image_cache/gagarin7/alexa3j5/642ef4a209468d212bdb64c2d9ff630f/alisabeat.jpg",
        "craft_only": False
    }
}

# Загрузка сохраненных данных
def load_saved_data():
    global CARDS_DATABASE, HIDDEN_CARDS, SHOP_ENABLED, CRAFT_RECIPES
    
    # Загрузка карточек
    if os.path.exists(CARDS_DB_FILE):
        try:
            with open(CARDS_DB_FILE, 'r', encoding='utf-8') as f:
                loaded_cards = json.load(f)
                CARDS_DATABASE.update(loaded_cards)
            logger.info(f"Загружено {len(loaded_cards)} карточек из файла")
        except Exception as e:
            logger.error(f"Ошибка загрузки карточек: {e}")
            # Если ошибка, используем карточки по умолчанию
            CARDS_DATABASE.update(DEFAULT_CARDS)
    else:
        # Если файла нет, используем карточки по умолчанию
        CARDS_DATABASE.update(DEFAULT_CARDS)
    
    # Загрузка скрытых карточек
    if os.path.exists(HIDDEN_CARDS_FILE):
        try:
            with open(HIDDEN_CARDS_FILE, 'r', encoding='utf-8') as f:
                hidden_list = json.load(f)
                HIDDEN_CARDS.update(hidden_list)
            logger.info(f"Загружено {len(hidden_list)} скрытых карточек")
        except Exception as e:
            logger.error(f"Ошибка загрузки скрытых карточек: {e}")
    
    # Загрузка статуса магазина
    if os.path.exists(SHOP_STATUS_FILE):
        try:
            with open(SHOP_STATUS_FILE, 'r', encoding='utf-8') as f:
                shop_status = json.load(f)
                SHOP_ENABLED = shop_status.get('enabled', True)
            logger.info(f"Статус магазина загружен: {'включен' if SHOP_ENABLED else 'выключен'}")
        except Exception as e:
            logger.error(f"Ошибка загрузки статуса магазина: {e}")
    
    # Загрузка рецептов крафта
    if os.path.exists(CRAFT_RECIPES_FILE):
        try:
            with open(CRAFT_RECIPES_FILE, 'r', encoding='utf-8') as f:
                CRAFT_RECIPES.update(json.load(f))
            logger.info(f"Загружено {len(CRAFT_RECIPES)} рецептов крафта")
        except Exception as e:
            logger.error(f"Ошибка загрузки рецептов крафта: {e}")

# Сохранение данных
def save_cards_database():
    try:
        with open(CARDS_DB_FILE, 'w', encoding='utf-8') as f:
            json.dump(CARDS_DATABASE, f, ensure_ascii=False, indent=2)
    except Exception as e:
        logger.error(f"Ошибка сохранения карточек: {e}")

def save_hidden_cards():
    try:
        with open(HIDDEN_CARDS_FILE, 'w', encoding='utf-8') as f:
            json.dump(list(HIDDEN_CARDS), f, ensure_ascii=False, indent=2)
    except Exception as e:
        logger.error(f"Ошибка сохранения скрытых карточек: {e}")

def save_shop_status():
    try:
        with open(SHOP_STATUS_FILE, 'w', encoding='utf-8') as f:
            json.dump({'enabled': SHOP_ENABLED}, f, ensure_ascii=False, indent=2)
    except Exception as e:
        logger.error(f"Ошибка сохранения статуса магазина: {e}")

def save_craft_recipes():
    try:
        with open(CRAFT_RECIPES_FILE, 'w', encoding='utf-8') as f:
            json.dump(CRAFT_RECIPES, f, ensure_ascii=False, indent=2)
    except Exception as e:
        logger.error(f"Ошибка сохранения рецептов крафта: {e}")

# Владельцы бота
OWNER_IDS = [7599616968, 5872295617, 8112013114]

# КД на кнопки (в секундах)
BUTTON_COOLDOWNS = {
    'get_card': 5,
    'profile': 3,
    'my_cards': 3,
    'shop': 3,
    'all_cards': 3,
    'craft': 3
}

# Система антиспама - НАСТРОЙКИ
SPAM_DETECTION_WINDOW = 10  # Секунды для окна детекции спама
SPAM_MESSAGE_LIMIT = 6      # Максимум сообщений в окне (уменьшил для лучшей детекции)
SPAM_COMMAND_LIMIT = 4      # Максимум одинаковых команд в окне
AUTO_MUTE_DURATIONS = [     # Прогрессивные муты за спам
    timedelta(minutes=5),   # 1-е нарушение: 5 минут
    timedelta(minutes=15),  # 2-е нарушение: 15 минут
    timedelta(minutes=30),  # 3-е нарушение: 30 минут
    timedelta(hours=1),     # 4-е нарушение: 1 час
    timedelta(hours=3),     # 5-е нарушение: 3 часа
    timedelta(hours=6),     # 6-е нарушение: 6 часов
    timedelta(days=1)       # 7-е нарушение: 1 день
]

# Словари для хранения состояний
CARD_ADD_STATES = {}
PROMO_CREATION_STATES = {}
CARD_EDIT_STATES = {}
MESSAGE_OWNERS = {}
ACTIVE_SELECTIONS = {}  # Для отслеживания активных выборов карточек
CARD_SELLING_STATES = {}  # Для отслеживания карточек, которые уже выставлены на продажу
USER_SELLING_STATES = {}  # Для отслеживания пользователей, которые находятся в процессе продажи
USER_PRICE_INPUT_STATES = {}  # Для отслеживания пользователей, которые вводят цену
USER_CRAFT_STATES = {}  # Для отслеживания пользователей в процессе крафта
recipe_states = {}  # Для состояний создания рецептов

# Система антиспама - ГЛОБАЛЬНЫЕ ПЕРЕМЕННЫЕ
user_message_history = defaultdict(list)  # История сообщений по пользователям
user_spam_violations = defaultdict(int)   # Количество нарушений антиспама по пользователям
ignored_users = set()  # Пользователи, которых игнорируем после мута
user_last_message_time = {}  # Время последнего сообщения пользователя
user_message_count = defaultdict(int)  # Счетчик сообщений пользователя
user_command_history = defaultdict(list)  # История команд пользователя

def init_db():
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            user_id INTEGER PRIMARY KEY,
            username TEXT,
            coins INTEGER DEFAULT 0,
            total_cards INTEGER DEFAULT 0
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS user_cards (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER,
            rarity TEXT,
            card_name TEXT,
            obtained_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (user_id) REFERENCES users (user_id)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS market (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            seller_id INTEGER,
            card_id INTEGER,
            price INTEGER,
            listing_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (seller_id) REFERENCES users (user_id),
            FOREIGN KEY (card_id) REFERENCES user_cards (id)
        )
    ''')
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS cooldowns (
            user_id INTEGER PRIMARY KEY,
            last_card_time INTEGER,
            FOREIGN KEY (user_id) REFERENCES users (user_id)
        )
    ''')
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS button_cooldowns (
            user_id INTEGER,
            button_type TEXT,
            last_press_time INTEGER,
            PRIMARY KEY (user_id, button_type)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS promocodes (
            code TEXT PRIMARY KEY,
            reward_type TEXT NOT NULL,
            reward_value TEXT NOT NULL,
            uses_left INTEGER DEFAULT 1,
            created_by INTEGER,
            created_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (created_by) REFERENCES users (user_id)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS used_promocodes (
            user_id INTEGER,
            promo_code TEXT,
            used_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            PRIMARY KEY (user_id, promo_code),
            FOREIGN KEY (user_id) REFERENCES users (user_id),
            FOREIGN KEY (promo_code) REFERENCES promocodes (code)
        )
    ''')

    cursor.execute('''
        CREATE TABLE IF NOT EXISTS bans (
            user_id INTEGER PRIMARY KEY,
            username TEXT,
            reason TEXT,
            banned_by INTEGER,
            banned_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (banned_by) REFERENCES users (user_id)
        )
    ''')

    # Таблица для мутов
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS mutes (
            user_id INTEGER PRIMARY KEY,
            username TEXT,
            muted_by INTEGER,
            reason TEXT,
            muted_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            unmute_time TIMESTAMP,
            violation_count INTEGER DEFAULT 1,
            FOREIGN KEY (muted_by) REFERENCES users (user_id)
        )
    ''')

    conn.commit()
    conn.close()

def get_user(user_id, username):
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()

    cursor.execute('SELECT * FROM users WHERE user_id = ?', (user_id,))
    user = cursor.fetchone()

    if not user:
        cursor.execute('INSERT INTO users (user_id, username, coins, total_cards) VALUES (?, ?, 0, 0)',
                       (user_id, username))
        conn.commit()
        user = (user_id, username, 0, 0)

    conn.close()
    return user

def get_random_card():
    available_cards = [card for card in CARDS_DATABASE.keys() 
                      if card not in HIDDEN_CARDS and not CARDS_DATABASE[card].get('craft_only', False)]
    if not available_cards:
        return random.choice(list(CARDS_DATABASE.keys()))
    return random.choice(available_cards)

def is_owner(user_id):
    return user_id in OWNER_IDS

def is_user_banned(user_id):
    """Проверяет, забанен ли пользователь"""
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM bans WHERE user_id = ?', (user_id,))
    banned = cursor.fetchone()
    conn.close()
    return banned is not None

def is_user_muted(user_id):
    """Проверяет, заглушен ли пользователь"""
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    cursor.execute('SELECT unmute_time FROM mutes WHERE user_id = ?', (user_id,))
    result = cursor.fetchone()
    conn.close()
    
    if not result:
        return False
    
    unmute_time = result[0]
    if unmute_time:
        # Проверяем, не истек ли мут
        if datetime.now() > datetime.fromisoformat(unmute_time):
            # Удаляем просроченный мут
            conn = sqlite3.connect('cats.db')
            cursor = conn.cursor()
            cursor.execute('DELETE FROM mutes WHERE user_id = ?', (user_id,))
            conn.commit()
            conn.close()
            
            # Удаляем из списка игнорируемых
            if user_id in ignored_users:
                ignored_users.remove(user_id)
                
            return False
        return True
    
    return True  # Перманентный мут

def get_ban_info(user_id):
    """Получает информацию о бане пользователя"""
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    cursor.execute('SELECT username, reason, banned_by, banned_time FROM bans WHERE user_id = ?', (user_id,))
    ban_info = cursor.fetchone()
    conn.close()
    return ban_info

def get_mute_info(user_id):
    """Получает информацию о муте пользователя"""
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    cursor.execute('SELECT username, reason, muted_by, muted_time, unmute_time, violation_count FROM mutes WHERE user_id = ?', (user_id,))
    mute_info = cursor.fetchone()
    conn.close()
    return mute_info

def check_button_cooldown(user_id, button_type):
    """Проверяет КД на кнопку"""
    current_time = time.time()
    
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    
    cursor.execute('SELECT last_press_time FROM button_cooldowns WHERE user_id = ? AND button_type = ?', 
                   (user_id, button_type))
    result = cursor.fetchone()
    
    if result:
        last_press = result[0]
        cooldown_time = BUTTON_COOLDOWNS.get(button_type, 3)
        
        if current_time - last_press < cooldown_time:
            conn.close()
            return False
    
    cursor.execute('''
        INSERT OR REPLACE INTO button_cooldowns (user_id, button_type, last_press_time) 
        VALUES (?, ?, ?)
    ''', (user_id, button_type, current_time))
    
    conn.commit()
    conn.close()
    return True

def is_valid_url(url):
    """Проверяет, является ли строка валидным URL"""
    regex = re.compile(
        r'^(?:http|ftp)s?://'
        r'(?:(?:[A-Z0-9](?:[A-Z0-9-]{0,61}[A-Z0-9])?\.)+(?:[A-Z]{2,6}\.?|[A-Z0-9-]{2,}\.?)|'
        r'localhost|'
        r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})'
        r'(?::\d+)?'
        r'(?:/?|[/?]\S+)$', re.IGNORECASE)
    return re.match(regex, url) is not None

def store_message_owner(message_id, user_id):
    """Сохраняет владельца сообщения с кнопками"""
    MESSAGE_OWNERS[message_id] = user_id

def check_message_owner(message_id, user_id):
    """Проверяет, принадлежит ли сообщение пользователю"""
    return MESSAGE_OWNERS.get(message_id) == user_id

def add_active_selection(user_id, card_id, selection_type):
    """Добавляет активный выбор карточки"""
    key = f"{user_id}_{selection_type}"
    ACTIVE_SELECTIONS[key] = card_id

def check_active_selection(user_id, card_id, selection_type):
    """Проверяет, выбрал ли пользователь уже эту карточку"""
    key = f"{user_id}_{selection_type}"
    return ACTIVE_SELECTIONS.get(key) == card_id

def remove_active_selection(user_id, selection_type):
    """Удаляет активный выбор"""
    key = f"{user_id}_{selection_type}"
    if key in ACTIVE_SELECTIONS:
        del ACTIVE_SELECTIONS[key]

# СИСТЕМА АНТИСПАМА - ОСНОВНЫЕ ФУНКЦИИ
def check_spam_early(user_id, message_text):
    """БЫСТРАЯ ПРОВЕРКА на спам ДО обработки команды"""
    current_time = time.time()
    
    # Если пользователь в списке игнорируемых - сразу пропускаем
    if user_id in ignored_users:
        return True
    
    # Проверяем слишком частые сообщения
    if user_id in user_last_message_time:
        time_since_last = current_time - user_last_message_time[user_id]
        if time_since_last < 0.5:  # Меньше 0.5 секунды между сообщениями
            user_message_count[user_id] += 1
        else:
            user_message_count[user_id] = 1
    else:
        user_message_count[user_id] = 1
    
    user_last_message_time[user_id] = current_time
    
    # Если слишком много сообщений за короткое время
    if user_message_count[user_id] > 4:  # Более 4 сообщений подряд быстро
        logger.warning(f"Пользователь {user_id} отправляет сообщения слишком быстро: {user_message_count[user_id]} сообщений")
        return True
    
    # Обновляем историю команд
    if user_id in user_command_history:
        user_command_history[user_id].append({
            'time': current_time,
            'text': message_text,
            'type': 'command' if message_text.startswith('/') or message_text in [
                '🎴 Получить карточку', '📊 Мой профиль', '📋 Мои карточки', 
                '🛒 Магазин', '📚 Все карточки', '🔨 Крафт'
            ] else 'text'
        })
        
        # Очищаем старые записи (старше 10 секунд)
        user_command_history[user_id] = [
            cmd for cmd in user_command_history[user_id] 
            if current_time - cmd['time'] <= SPAM_DETECTION_WINDOW
        ]
        
        # Проверяем на спам команд
        if len(user_command_history[user_id]) >= SPAM_COMMAND_LIMIT:
            # Проверяем одинаковые команды
            command_counts = {}
            for cmd in user_command_history[user_id]:
                if cmd['type'] == 'command':
                    command_counts[cmd['text']] = command_counts.get(cmd['text'], 0) + 1
            
            for cmd_text, count in command_counts.items():
                if count >= SPAM_COMMAND_LIMIT:
                    logger.warning(f"Пользователь {user_id} спамит командой '{cmd_text}' {count} раз")
                    return True
    else:
        user_command_history[user_id] = [{
            'time': current_time,
            'text': message_text,
            'type': 'command' if message_text.startswith('/') or message_text in [
                '🎴 Получить карточку', '📊 Мой профиль', '📋 Мои карточки', 
                '🛒 Магазин', '📚 Все карточки', '🔨 Крафт'
            ] else 'text'
        }]
    
    return False

def auto_mute_user(user_id, username, reason="Автоматическое обнаружение спама"):
    """Автоматически заглушает пользователя за спам"""
    try:
        # Получаем количество предыдущих нарушений
        violation_count = user_spam_violations.get(user_id, 0) + 1
        user_spam_violations[user_id] = violation_count
        
        # Определяем длительность мута
        if violation_count <= len(AUTO_MUTE_DURATIONS):
            mute_duration = AUTO_MUTE_DURATIONS[violation_count - 1]
        else:
            mute_duration = AUTO_MUTE_DURATIONS[-1]  # Максимальная длительность
        
        unmute_time = datetime.now() + mute_duration
        
        # Добавляем мут в базу данных
        conn = sqlite3.connect('cats.db')
        cursor = conn.cursor()
        
        cursor.execute('''
            INSERT OR REPLACE INTO mutes (user_id, username, muted_by, reason, unmute_time, violation_count) 
            VALUES (?, ?, ?, ?, ?, ?)
        ''', (user_id, username, 0, f"{reason} (нарушение #{violation_count})", 
              unmute_time.isoformat(), violation_count))
        
        conn.commit()
        conn.close()
        
        # Добавляем пользователя в список игнорируемых
        ignored_users.add(user_id)
        
        # Очищаем историю пользователя
        if user_id in user_command_history:
            del user_command_history[user_id]
        if user_id in user_message_count:
            del user_message_count[user_id]
        if user_id in user_last_message_time:
            del user_last_message_time[user_id]
        
        # Логируем мут
        logger.warning(f"Пользователь {user_id} (@{username}) автоматически заглушен на {mute_duration}. Нарушение #{violation_count}")
        
        # Отправляем уведомление пользователю В ЛИЧНЫЕ СООБЩЕНИЯ
        try:
            hours = mute_duration.seconds // 3600
            minutes = (mute_duration.seconds % 3600) // 60
            
            mute_message = f"🔇 Вы были автоматически заглушены за спам!\n\n"
            mute_message += f"📝 Причина: {reason}\n"
            mute_message += f"🔢 Нарушение: #{violation_count}\n"
            
            if hours > 0:
                mute_message += f"⏰ Мут истечет через: {hours}ч {minutes}м\n"
            else:
                mute_message += f"⏰ Мут истечет через: {minutes} минут\n"
            
            mute_message += f"\n⚠️ Вам доступны только команды:\n"
            mute_message += f"• 🎴 Получить карточку\n"
            mute_message += f"• 📊 Мой профиль\n\n"
            mute_message += f"Если вы считаете, что это ошибка, свяжитесь с администратором."
            
            bot.send_message(user_id, mute_message)
        except Exception as e:
            logger.error(f"Не удалось отправить сообщение о муте пользователю {user_id}: {e}")
        
        # Уведомляем администраторов ТОЛЬКО В ЛС
        for admin_id in OWNER_IDS:
            try:
                admin_message = f"🚨 АВТОМУТ: Пользователь @{username} ({user_id})\n"
                admin_message += f"📝 Причина: {reason}\n"
                admin_message += f"🔢 Нарушение: #{violation_count}\n"
                admin_message += f"⏰ Длительность: {hours}ч {minutes}м"
                bot.send_message(admin_id, admin_message)
            except:
                pass
        
        return True
    except Exception as e:
        logger.error(f"Ошибка при автоматическом муте пользователя {user_id}: {e}")
        return False

def cleanup_old_data():
    """Очищает старые данные из памяти"""
    current_time = time.time()
    
    # Очищаем старые сообщения из истории команд
    for user_id in list(user_command_history.keys()):
        user_history = user_command_history[user_id]
        user_history = [
            cmd for cmd in user_history 
            if current_time - cmd['time'] <= SPAM_DETECTION_WINDOW * 2
        ]
        if user_history:
            user_command_history[user_id] = user_history
        else:
            del user_command_history[user_id]
    
    # Очищаем счетчики для неактивных пользователей
    cutoff_time = current_time - 300  # 5 минут
    inactive_users = [
        uid for uid, last_time in user_last_message_time.items()
        if last_time < cutoff_time
    ]
    
    for uid in inactive_users:
        if uid in user_message_count:
            del user_message_count[uid]
        if uid in user_last_message_time:
            del user_last_message_time[uid]

# ОСНОВНОЙ ДЕКОРАТОР С АНТИСПАМОМ
def antispam_protection(func):
    """Главный декоратор для защиты от спама"""
    def wrapper(message):
        if message.date < BOT_START_TIME:
            return
            
        user_id = message.from_user.id
        username = message.from_user.username or message.from_user.first_name
        
        # Периодическая очистка старых данных
        cleanup_old_data()
        
        # Если пользователь заглушен - проверяем, не истек ли мут
        if is_user_muted(user_id):
            # Удаляем из списка игнорируемых если мут истек
            if user_id in ignored_users:
                # Проверяем в базе данных
                conn = sqlite3.connect('cats.db')
                cursor = conn.cursor()
                cursor.execute('SELECT unmute_time FROM mutes WHERE user_id = ?', (user_id,))
                result = cursor.fetchone()
                conn.close()
                
                if result and result[0]:
                    unmute_time = datetime.fromisoformat(result[0])
                    if datetime.now() > unmute_time:
                        # Мут истек
                        if user_id in ignored_users:
                            ignored_users.remove(user_id)
                        # Продолжаем обработку
                        pass
                    else:
                        # Мут еще действует - игнорируем
                        return
                else:
                    # Нет информации о муте - удаляем из игнорируемых
                    if user_id in ignored_users:
                        ignored_users.remove(user_id)
        
        # Если пользователь в списке игнорируемых - игнорируем
        if user_id in ignored_users:
            return
        
        # Проверка на спам (включая команды!)
        message_text = message.text or ""
        
        # Проверяем только если это не пустое сообщение
        if message_text:
            if check_spam_early(user_id, message_text):
                # Обнаружен спам - автомут
                auto_mute_user(user_id, username, "Спам командами/сообщениями")
                return
        
        # Проверяем бан
        if is_user_banned(user_id):
            try:
                bot.send_message(user_id, "🚫 Вы забанены в боте!")
            except:
                pass
            return
        
        # Проверяем мут
        if is_user_muted(user_id):
            mute_info = get_mute_info(user_id)
            if mute_info:
                username_db, reason, muted_by, muted_time, unmute_time, violation_count = mute_info
                
                # Проверяем, не истек ли мут
                if unmute_time and datetime.now() > datetime.fromisoformat(unmute_time):
                    # Удаляем просроченный мут
                    conn = sqlite3.connect('cats.db')
                    cursor = conn.cursor()
                    cursor.execute('DELETE FROM mutes WHERE user_id = ?', (user_id,))
                    conn.commit()
                    conn.close()
                    
                    # Удаляем из списка игнорируемых
                    if user_id in ignored_users:
                        ignored_users.remove(user_id)
                    
                    # Продолжаем выполнение
                    return func(message)
                
                # Разрешаем только определенные команды для заглушенных
                allowed_commands = ["🎴 Получить карточку", "📊 Мой профиль", "/start"]
                
                if message_text in allowed_commands:
                    return func(message)
                else:
                    # ИГНОРИРУЕМ команду, НЕ отправляем сообщение в группе
                    # Только логируем
                    logger.info(f"Заглушенный пользователь {user_id} пытался использовать команду: {message_text}")
                    return
        
        # Если все проверки пройдены - выполняем функцию
        return func(message)
    
    return wrapper

# ДЕКОРАТОР ДЛЯ CALLBACK С АНТИСПАМОМ
def antispam_callback_protection(func):
    """Декоратор для защиты callback от спама"""
    def wrapper(call):
        user_id = call.from_user.id
        
        # Если пользователь в списке игнорируемых - игнорируем
        if user_id in ignored_users:
            bot.answer_callback_query(call.id)
            return
        
        # Проверяем бан
        if is_user_banned(user_id):
            bot.answer_callback_query(call.id, "🚫 Вы забанены в боте!", show_alert=True)
            return
        
        # Проверяем мут
        if is_user_muted(user_id):
            mute_info = get_mute_info(user_id)
            if mute_info:
                username, reason, muted_by, muted_time, unmute_time, violation_count = mute_info
                
                # Проверяем, не истек ли мут
                if unmute_time and datetime.now() > datetime.fromisoformat(unmute_time):
                    # Удаляем просроченный мут
                    conn = sqlite3.connect('cats.db')
                    cursor = conn.cursor()
                    cursor.execute('DELETE FROM mutes WHERE user_id = ?', (user_id,))
                    conn.commit()
                    conn.close()
                    
                    # Удаляем из списка игнорируемых
                    if user_id in ignored_users:
                        ignored_users.remove(user_id)
                    
                    # Продолжаем выполнение
                    return func(call)
                
                # Разрешаем только определенные callback для заглушенных
                allowed_callbacks = ["view_all_cards", "view_my_collection", "page_all_", "page_my_"]
                call_data = call.data
                
                for allowed in allowed_callbacks:
                    if call_data.startswith(allowed):
                        return func(call)
                
                # Игнорируем callback для заглушенных
                bot.answer_callback_query(call.id)
                return
        
        # Если все проверки пройдены - выполняем функцию
        return func(call)
    
    return wrapper

# НОВЫЕ КОМАНДЫ ДЛЯ ВЛАДЕЛЬЦЕВ - УПРАВЛЕНИЕ МУТАМИ И АНТИСПАМОМ
@bot.message_handler(commands=['mute'])
@antispam_protection
def mute_user_command(message):
    user_id = message.from_user.id
    
    if not is_owner(user_id):
        bot.send_message(message.chat.id, "❌ У вас нет прав для использования этой команды!")
        return
    
    try:
        parts = message.text.split(' ', 3)
        if len(parts) < 3:
            bot.send_message(message.chat.id, "❌ Используйте: /mute @username время [причина]\n\n"
                                           "Время можно указать:\n"
                                           "• 1h - 1 час\n"
                                           "• 30m - 30 минут\n"
                                           "• 2d - 2 дня\n"
                                           "• 0 - навсегда")
            return
            
        username = parts[1].replace('@', '')
        time_str = parts[2].lower()
        reason = parts[3] if len(parts) > 3 else "Спам/Флуд"
        
        conn = sqlite3.connect('cats.db')
        cursor = conn.cursor()
        cursor.execute('SELECT user_id FROM users WHERE username = ?', (username,))
        target_user = cursor.fetchone()
        
        if not target_user:
            bot.send_message(message.chat.id, f"❌ Пользователь @{username} не найден!")
            conn.close()
            return
            
        target_user_id = target_user[0]
        
        # Проверяем, не забанен ли уже пользователь
        if is_user_banned(target_user_id):
            bot.send_message(message.chat.id, f"❌ Пользователь @{username} уже забанен!")
            conn.close()
            return
        
        # Парсим время
        unmute_time = None
        if time_str != "0":
            try:
                if time_str.endswith('h'):
                    hours = int(time_str[:-1])
                    unmute_time = datetime.now() + timedelta(hours=hours)
                elif time_str.endswith('m'):
                    minutes = int(time_str[:-1])
                    unmute_time = datetime.now() + timedelta(minutes=minutes)
                elif time_str.endswith('d'):
                    days = int(time_str[:-1])
                    unmute_time = datetime.now() + timedelta(days=days)
                else:
                    bot.send_message(message.chat.id, "❌ Неверный формат времени!")
                    conn.close()
                    return
            except ValueError:
                bot.send_message(message.chat.id, "❌ Неверный формат времени!")
                conn.close()
                return
        
        # Проверяем, не заглушен ли уже пользователь
        cursor.execute('SELECT violation_count FROM mutes WHERE user_id = ?', (target_user_id,))
        existing_mute = cursor.fetchone()
        
        if existing_mute:
            violation_count = existing_mute[0] + 1
        else:
            violation_count = 1
        
        # Добавляем или обновляем мут
        cursor.execute('''
            INSERT OR REPLACE INTO mutes (user_id, username, muted_by, reason, unmute_time, violation_count) 
            VALUES (?, ?, ?, ?, ?, ?)
        ''', (target_user_id, username, user_id, reason, 
              unmute_time.isoformat() if unmute_time else None, violation_count))
        
        # Добавляем в список игнорируемых
        ignored_users.add(target_user_id)
        
        # Сбрасываем счетчик нарушений антиспама
        if target_user_id in user_spam_violations:
            user_spam_violations[target_user_id] = violation_count
        
        conn.commit()
        conn.close()
        
        # Формируем сообщение о муте
        if unmute_time:
            time_left = unmute_time - datetime.now()
            hours = time_left.seconds // 3600
            minutes = (time_left.seconds % 3600) // 60
            time_message = f"{hours}ч {minutes}м"
        else:
            time_message = "навсегда"
        
        bot.send_message(message.chat.id, 
                        f"✅ Пользователь @{username} заглушен!\n\n"
                        f"⏰ Время: {time_message}\n"
                        f"📝 Причина: {reason}\n"
                        f"🔢 Нарушение: #{violation_count}")
        
        # Отправляем уведомление пользователю В ЛИЧНЫЕ СООБЩЕНИЯ
        try:
            if unmute_time:
                mute_text = f"🔇 Вы были заглушены в боте!\n\n" \
                          f"📝 Причина: {reason}\n" \
                          f"🔢 Нарушение: #{violation_count}\n" \
                          f"⏰ Мут истечет через: {hours}ч {minutes}м\n\n" \
                          f"Вам доступны только команды:\n" \
                          f"• 🎴 Получить карточку\n" \
                          f"• 📊 Мой профиль\n\n" \
                          f"Если вы считаете, что это ошибка, свяжитесь с администратором."
            else:
                mute_text = f"🔇 Вы были заглушены в боте навсегда!\n\n" \
                          f"📝 Причина: {reason}\n" \
                          f"🔢 Нарушение: #{violation_count}\n\n" \
                          f"Вам доступны только команды:\n" \
                          f"• 🎴 Получить карточку\n" \
                          f"• 📊 Мой профиль\n\n" \
                          f"Если вы считаете, что это ошибка, свяжитесь с администратором."
            
            bot.send_message(target_user_id, mute_text)
        except:
            pass
            
    except Exception as e:
        bot.send_message(message.chat.id, f"❌ Ошибка: {str(e)}")

@bot.message_handler(commands=['unmute'])
@antispam_protection
def unmute_user_command(message):
    user_id = message.from_user.id
    
    if not is_owner(user_id):
        bot.send_message(message.chat.id, "❌ У вас нет прав для использования этой команды!")
        return
    
    try:
        parts = message.text.split()
        if len(parts) != 2:
            bot.send_message(message.chat.id, "❌ Используйте: /unmute @username")
            return
            
        username = parts[1].replace('@', '')
        
        conn = sqlite3.connect('cats.db')
        cursor = conn.cursor()
        cursor.execute('SELECT user_id FROM mutes WHERE username = ?', (username,))
        muted_user = cursor.fetchone()
        
        if not muted_user:
            bot.send_message(message.chat.id, f"❌ Пользователь @{username} не найден в списке заглушенных!")
            conn.close()
            return
            
        target_user_id = muted_user[0]
        
        # Удаляем из мутов
        cursor.execute('DELETE FROM mutes WHERE user_id = ?', (target_user_id,))
        
        # Удаляем из списка игнорируемых
        if target_user_id in ignored_users:
            ignored_users.remove(target_user_id)
        
        # Сбрасываем счетчик нарушений
        if target_user_id in user_spam_violations:
            del user_spam_violations[target_user_id]
        
        conn.commit()
        conn.close()
        
        bot.send_message(message.chat.id, f"✅ Пользователь @{username} разглушен!")
        
        # Отправляем уведомление пользователю В ЛИЧНЫЕ СООБЩЕНИЯ
        try:
            bot.send_message(target_user_id, "🎉 Ваш мут снят! Теперь вы снова можете использовать все функции бота.")
        except:
            pass
            
    except Exception as e:
        bot.send_message(message.chat.id, f"❌ Ошибка: {str(e)}")

@bot.message_handler(commands=['mutelist'])
@antispam_protection
def mute_list_command(message):
    user_id = message.from_user.id
    
    if not is_owner(user_id):
        bot.send_message(message.chat.id, "❌ У вас нет прав для использования этой команды!")
        return
    
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    
    cursor.execute('''
        SELECT username, reason, muted_by, muted_time, unmute_time, violation_count 
        FROM mutes 
        ORDER BY muted_time DESC
    ''')
    mutes = cursor.fetchall()
    conn.close()
    
    if not mutes:
        bot.send_message(message.chat.id, "❌ Нет заглушенных пользователей!")
        return
    
    mutes_text = "📋 Список заглушенных пользователей:\n\n"
    
    for username, reason, muted_by, muted_time, unmute_time, violation_count in mutes:
        mutes_text += f"👤 @{username}\n"
        mutes_text += f"📝 Причина: {reason}\n"
        mutes_text += f"🔢 Нарушение: #{violation_count}\n"
        
        if unmute_time:
            unmute_dt = datetime.fromisoformat(unmute_time)
            if datetime.now() > unmute_dt:
                mutes_text += f"⏰ Статус: Истек (требуется удаление)\n"
            else:
                time_left = unmute_dt - datetime.now()
                hours = time_left.seconds // 3600
                minutes = (time_left.seconds % 3600) // 60
                mutes_text += f"⏰ Осталось: {hours}ч {minutes}м\n"
        else:
            mutes_text += f"⏰ Статус: Бессрочный\n"
        
        mutes_text += f"📅 Заглушен: {muted_time[:10]}\n\n"
    
    bot.send_message(message.chat.id, mutes_text)

@bot.message_handler(commands=['clearexpiredmutes'])
@antispam_protection
def clear_expired_mutes_command(message):
    user_id = message.from_user.id
    
    if not is_owner(user_id):
        bot.send_message(message.chat.id, "❌ У вас нет прав для использования этой команды!")
        return
    
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    
    cursor.execute('SELECT user_id, username, unmute_time FROM mutes')
    all_mutes = cursor.fetchall()
    
    cleared_count = 0
    for user_id_db, username, unmute_time in all_mutes:
        if unmute_time:
            unmute_dt = datetime.fromisoformat(unmute_time)
            if datetime.now() > unmute_dt:
                cursor.execute('DELETE FROM mutes WHERE user_id = ?', (user_id_db,))
                cleared_count += 1
                
                # Удаляем из списка игнорируемых
                if user_id_db in ignored_users:
                    ignored_users.remove(user_id_db)
                
                # Отправляем уведомление пользователю В ЛИЧНЫЕ СООБЩЕНИЯ
                try:
                    bot.send_message(user_id_db, "🎉 Ваш мут истек! Теперь вы снова можете использовать все функции бота.")
                except:
                    pass
    
    conn.commit()
    conn.close()
    
    bot.send_message(message.chat.id, f"✅ Удалено {cleared_count} просроченных мутов!")

@bot.message_handler(commands=['antispamstats'])
@antispam_protection
def antispam_stats_command(message):
    user_id = message.from_user.id
    
    if not is_owner(user_id):
        bot.send_message(message.chat.id, "❌ У вас нет прав для использования этой команды!")
        return
    
    stats_text = "🚨 Статистика антиспама:\n\n"
    
    # Активные нарушения
    stats_text += f"👥 Пользователей в истории: {len(user_command_history)}\n"
    stats_text += f"⚠️ Активные нарушения: {len(user_spam_violations)}\n"
    stats_text += f"🔇 Игнорируемых пользователей: {len(ignored_users)}\n"
    
    # Топ нарушителей
    if user_spam_violations:
        stats_text += "\n📊 Топ нарушителей:\n"
        sorted_violations = sorted(user_spam_violations.items(), key=lambda x: x[1], reverse=True)[:5]
        
        for user_id_db, count in sorted_violations:
            try:
                user = bot.get_chat(user_id_db)
                username = user.username or user.first_name
                stats_text += f"  • @{username}: {count} нарушений\n"
            except:
                stats_text += f"  • ID {user_id_db}: {count} нарушений\n"
    
    # Активные муты
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    cursor.execute('SELECT COUNT(*) FROM mutes')
    active_mutes = cursor.fetchone()[0]
    conn.close()
    
    stats_text += f"\n🔇 Активных мутов: {active_mutes}\n"
    
    # Параметры системы
    stats_text += f"\n⚙️ Параметры системы:\n"
    stats_text += f"  • Окно детекции: {SPAM_DETECTION_WINDOW} сек\n"
    stats_text += f"  • Лимит сообщений: {SPAM_MESSAGE_LIMIT}\n"
    stats_text += f"  • Лимит команд: {SPAM_COMMAND_LIMIT}\n"
    
    bot.send_message(message.chat.id, stats_text)

@bot.message_handler(commands=['resetspamstats'])
@antispam_protection
def reset_spam_stats_command(message):
    user_id = message.from_user.id
    
    if not is_owner(user_id):
        bot.send_message(message.chat.id, "❌ У вас нет прав для использования этой команды!")
        return
    
    try:
        parts = message.text.split()
        if len(parts) == 1:
            # Сброс всей статистики
            user_command_history.clear()
            user_spam_violations.clear()
            user_message_count.clear()
            user_last_message_time.clear()
            ignored_users.clear()
            bot.send_message(message.chat.id, "✅ Вся статистика антиспама сброшена!")
            
        elif len(parts) == 2:
            # Сброс для конкретного пользователя
            username = parts[1].replace('@', '')
            
            conn = sqlite3.connect('cats.db')
            cursor = conn.cursor()
            cursor.execute('SELECT user_id FROM users WHERE username = ?', (username,))
            target_user = cursor.fetchone()
            conn.close()
            
            if not target_user:
                bot.send_message(message.chat.id, f"❌ Пользователь @{username} не найден!")
                return
            
            target_user_id = target_user[0]
            
            if target_user_id in user_command_history:
                del user_command_history[target_user_id]
            
            if target_user_id in user_spam_violations:
                del user_spam_violations[target_user_id]
            
            if target_user_id in user_message_count:
                del user_message_count[target_user_id]
            
            if target_user_id in user_last_message_time:
                del user_last_message_time[target_user_id]
            
            if target_user_id in ignored_users:
                ignored_users.remove(target_user_id)
            
            bot.send_message(message.chat.id, f"✅ Статистика антиспама для @{username} сброшена!")
            
    except Exception as e:
        bot.send_message(message.chat.id, f"❌ Ошибка: {str(e)}")

# ОСНОВНЫЕ КОМАНДЫ С АНТИСПАМ ЗАЩИТОЙ
@bot.message_handler(commands=['start'])
@antispam_protection
def start_command(message):
    user_id = message.from_user.id
    username = message.from_user.username or message.from_user.first_name

    get_user(user_id, username)

    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    btn1 = types.KeyboardButton('🎴 Получить карточку')
    btn2 = types.KeyboardButton('📊 Мой профиль')
    btn3 = types.KeyboardButton('📋 Мои карточки')
    btn4 = types.KeyboardButton('🛒 Магазин')
    btn5 = types.KeyboardButton('📚 Все карточки')
    btn6 = types.KeyboardButton('🔨 Крафт')
    markup.add(btn1, btn2, btn3, btn4)
    markup.add(btn5, btn6)

    welcome_text = f"""🐱 Добро пожаловать в мир карточек с котами, {username}!

✨ Доступные команды:
• 🎴 Получить карточку
• 📊 Мой профиль  
• 📋 Мои карточки
• 🛒 Магазин
• 📚 Все карточки
• 🔨 Крафт

🎁 Для получения бонусов зайдите в нашего бота @CardsCatsBot

💡 *Магазин работает только в личных сообщениях с ботом!*"""

    bot.send_message(message.chat.id, welcome_text, reply_markup=markup, parse_mode='Markdown')

@bot.message_handler(func=lambda message: message.text == '🎴 Получить карточку')
@antispam_protection
def get_card(message):
    user_id = message.from_user.id
    
    if not check_button_cooldown(user_id, 'get_card'):
        bot.send_message(message.chat.id, "⏳ Подождите немного перед следующим получением карточки!")
        return
    
    username = message.from_user.username or message.from_user.first_name
    get_user(user_id, username)
    
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    cursor.execute('SELECT last_card_time FROM cooldowns WHERE user_id = ?', (user_id,))
    result = cursor.fetchone()
    
    current_time = int(time.time())
    cooldown_seconds = 2 * 3600
    
    if result and (current_time - result[0]) < cooldown_seconds:
        time_left = cooldown_seconds - (current_time - result[0])
        hours = time_left // 3600
        minutes = (time_left % 3600) // 60
        bot.send_message(message.chat.id, f"⏰ Следующую карточку можно будет получить через: {hours}ч {minutes}м")
        conn.close()
        return
    
    card_name = get_random_card()
    card_data = CARDS_DATABASE[card_name]
    
    cursor.execute('UPDATE users SET coins = coins + ?, total_cards = total_cards + 1 WHERE user_id = ?',
                   (card_data["coins"], user_id))
    cursor.execute('INSERT OR REPLACE INTO cooldowns (user_id, last_card_time) VALUES (?, ?)',
                   (user_id, current_time))
    cursor.execute('INSERT INTO user_cards (user_id, rarity, card_name) VALUES (?, ?, ?)',
                   (user_id, card_data["rarity"], card_name))
    
    conn.commit()
    conn.close()
    
    card_text = f"""Вам выпала карточка!

🖼 Карточка: "{card_name}"
⭐️ Редкость: {card_data['rarity']} {RARITIES[card_data['rarity']]['name']}
💰 Монеты: +{card_data['coins']} монет 
📝 Описание: {card_data['description']}"""

    try:
        bot.send_photo(message.chat.id, card_data['image'], caption=card_text)
    except Exception as e:
        logger.error(f"Error sending photo: {e}")
        bot.send_message(message.chat.id, card_text)

@bot.message_handler(func=lambda message: message.text == '📊 Мой профиль')
@antispam_protection
def show_stats(message):
    user_id = message.from_user.id
    
    if not check_button_cooldown(user_id, 'profile'):
        bot.send_message(message.chat.id, "⏳ Подождите немного перед следующим просмотром профиля!")
        return
    
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    
    cursor.execute('SELECT coins, total_cards FROM users WHERE user_id = ?', (user_id,))
    user_stats = cursor.fetchone()
    
    if not user_stats:
        bot.send_message(message.chat.id, "❌ Вы ещё не начали собирать карточки!")
        conn.close()
        return

    coins, total_cards = user_stats

    cursor.execute('''
        SELECT rarity, COUNT(*) as count 
        FROM user_cards 
        WHERE user_id = ? 
        GROUP BY rarity 
        ORDER BY count DESC
    ''', (user_id,))
    cards_by_rarity = cursor.fetchall()
    conn.close()

    stats_text = f"""📊 Ваша статистика:

💰 Монеты: {coins} монет
🎴 Всего карточек: {total_cards} шт.

📈 Коллекция по редкостям:"""

    for rarity, count in cards_by_rarity:
        rarity_name = RARITIES[rarity]["name"]
        stats_text += f"\n{rarity} {rarity_name}: {count} шт."

    bot.send_message(message.chat.id, stats_text)

@bot.message_handler(func=lambda message: message.text in ['📋 Мои карточки', '🛒 Магазин', '📚 Все карточки', '🔨 Крафт'])
@antispam_protection
def handle_other_commands(message):
    user_id = message.from_user.id
    command = message.text
    
    if command == '📋 Мои карточки':
        if not check_button_cooldown(user_id, 'my_cards'):
            bot.send_message(message.chat.id, "⏳ Подождите немного перед следующим просмотром карточек!")
            return
        
        conn = sqlite3.connect('cats.db')
        cursor = conn.cursor()

        cursor.execute('''
            SELECT rarity, card_name, obtained_date 
            FROM user_cards 
            WHERE user_id = ? 
            ORDER BY obtained_date DESC 
            LIMIT 20
        ''', (user_id,))
        cards = cursor.fetchall()
        conn.close()

        if not cards:
            bot.send_message(message.chat.id, "❌ У вас пока нет карточек!")
            return

        cards_text = "📋 Ваши последние карточки:\n\n"

        for i, (rarity, card_name, date) in enumerate(cards, 1):
            rarity_name = RARITIES[rarity]["name"]
            cards_text += f"{i}. {rarity} {card_name} ({rarity_name})\n"

        bot.send_message(message.chat.id, cards_text)
    
    elif command == '🛒 Магазин':
        shop_menu(message)
    
    elif command == '📚 Все карточки':
        if not check_button_cooldown(user_id, 'all_cards'):
            bot.send_message(message.chat.id, "⏳ Подождите немного перед следующим просмотром!")
            return
        
        markup = types.InlineKeyboardMarkup()
        markup.add(types.InlineKeyboardButton("📖 Просмотр всех карточек", callback_data="view_all_cards"))
        markup.add(types.InlineKeyboardButton("📊 Список моих карточек", callback_data="view_my_collection"))
        
        bot.send_message(message.chat.id, 
                        "📚 Выберите режим просмотра карточек:",
                        reply_markup=markup)
    
    elif command == '🔨 Крафт':
        if not check_button_cooldown(user_id, 'craft'):
            bot.send_message(message.chat.id, "⏳ Подождите немного перед следующим действием!")
            return
        
        markup = types.InlineKeyboardMarkup()
        markup.add(types.InlineKeyboardButton("📖 Просмотр рецептов крафта", callback_data="view_craft_recipes"))
        markup.add(types.InlineKeyboardButton("🔨 Создать карточку", callback_data="start_craft"))
        
        bot.send_message(message.chat.id,
                        "🔨 Мастерская крафта:\n\n"
                        "Здесь вы можете создавать уникальные карточки, объединяя другие карточки!",
                        reply_markup=markup)

def shop_menu(message):
    user_id = message.from_user.id
    
    # Проверяем, что это личные сообщения с ботом
    if message.chat.type != 'private':
        bot.send_message(message.chat.id, "❌ Магазин работает только в личных сообщениях с ботом! Напишите мне в ЛС.")
        return
    
    if not check_button_cooldown(user_id, 'shop'):
        bot.send_message(message.chat.id, "⏳ Подождите немного перед следующим действием в магазине!")
        return
    
    # Проверяем, включен ли магазин
    if not SHOP_ENABLED:
        bot.send_message(message.chat.id, "❌ Магазин временно отключен администратором!")
        return
    
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    btn1 = types.KeyboardButton('💰 Купить карточки')
    btn2 = types.KeyboardButton('💎 Продать карточки')
    btn3 = types.KeyboardButton('📋 Мои объявления')
    btn4 = types.KeyboardButton('🎴 Получить карточку')
    btn5 = types.KeyboardButton('📊 Мой профиль')
    markup.add(btn1, btn2, btn3)
    markup.add(btn4, btn5)

    shop_text = """🛒 Добро пожаловать в магазин карточек!

💰 **Купить карточки** - просмотреть карточки других игроков
💎 **Продать карточки** - выставить свои карточки на продажу
📋 **Мои объявления** - управление вашими продажами

⏰ *Внимание: процесс продажи отменяется через 20 секунд бездействия*"""

    bot.send_message(message.chat.id, shop_text, reply_markup=markup, parse_mode='Markdown')

# ОБРАБОТЧИК КНОПОК С АНТИСПАМ ЗАЩИТОЙ
@bot.callback_query_handler(func=lambda call: call.data.startswith(('view_', 'craft_', 'select_craft_card_', 'page_', 'craft_recipe_')))
@antispam_callback_protection
def handle_view_callback(call):
    user_id = call.from_user.id
    chat_id = call.message.chat.id

    if not check_button_cooldown(user_id, 'all_cards'):
        bot.answer_callback_query(call.id, "⏳ Подождите немного перед следующим действием!", show_alert=True)
        return

    if call.data == 'view_all_cards':
        show_all_cards_page(call.message, user_id, 0)
        
    elif call.data == 'view_my_collection':
        show_user_collection_page(call.message, user_id, 0)
        
    elif call.data == 'view_craft_recipes':
        show_craft_recipes(call.message, user_id)
        
    elif call.data == 'start_craft':
        start_craft_selection(call.message, user_id)
        
    elif call.data.startswith('select_craft_card_'):
        card_name = call.data.split('_', 3)[3]
        process_craft_selection(call.message, user_id, card_name)
        
    elif call.data.startswith('page_all_'):
        page = int(call.data.split('_')[2])
        show_all_cards_page(call.message, user_id, page)
        
    elif call.data.startswith('page_my_'):
        page = int(call.data.split('_')[2])
        show_user_collection_page(call.message, user_id, page)
        
    elif call.data.startswith('craft_recipe_'):
        card_name = call.data.split('_', 2)[2]
        show_recipe_details(call.message, user_id, card_name)

    bot.answer_callback_query(call.id)

# ФУНКЦИИ ДЛЯ ПРОСМОТРА КАРТОЧЕК (остаются без изменений)
def show_all_cards_page(message, user_id, page):
    cards_per_page = 10
    all_cards = list(CARDS_DATABASE.keys())
    total_pages = (len(all_cards) + cards_per_page - 1) // cards_per_page
    
    start_idx = page * cards_per_page
    end_idx = start_idx + cards_per_page
    page_cards = all_cards[start_idx:end_idx]
    
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    
    text = f"📚 Все карточки (Страница {page + 1}/{total_pages}):\n\n"
    
    for i, card_name in enumerate(page_cards, start_idx + 1):
        card_data = CARDS_DATABASE[card_name]
        
        # Проверяем, есть ли карточка у пользователя
        cursor.execute('SELECT id FROM user_cards WHERE user_id = ? AND card_name = ?', (user_id, card_name))
        has_card = cursor.fetchone() is not None
        
        status = "✅" if has_card else "❌"
        craft_only = "🔨" if card_data.get('craft_only', False) else ""
        
        text += f"{status} {craft_only} {i}. {card_data['rarity']} {card_name}\n"
    
    conn.close()
    
    markup = types.InlineKeyboardMarkup()
    
    # Кнопки навигации
    nav_buttons = []
    if page > 0:
        nav_buttons.append(types.InlineKeyboardButton("⬅️ Назад", callback_data=f"page_all_{page-1}"))
    if page < total_pages - 1:
        nav_buttons.append(types.InlineKeyboardButton("Вперед ➡️", callback_data=f"page_all_{page+1}"))
    
    if nav_buttons:
        markup.row(*nav_buttons)
    
    try:
        bot.edit_message_text(text, message.chat.id, message.message_id, reply_markup=markup)
    except:
        bot.send_message(message.chat.id, text, reply_markup=markup)

def show_user_collection_page(message, user_id, page):
    cards_per_page = 10
    
    conn = sqlite3.connect('cats.db')
    cursor = conn.cursor()
    
    cursor.execute('SELECT COUNT(DISTINCT card_name) FROM user_cards WHERE user_id = ?', (user_id,))
    total_user_cards = cursor.fetchone()[0]
    total_pages = (total_user_cards + cards_per_page - 1) // cards_per_page
    
    cursor.execute('''
        SELECT card_name, COUNT(*) as count 
        FROM user_cards 
        WHERE user_id = ? 
        GROUP BY card_name 
        ORDER BY card_name
        LIMIT ? OFFSET ?
    ''', (user_id, cards_per_page, page * cards_per_page))
    
    user_cards = cursor.fetchall()
    conn.close()
    
    text = f"📊 Моя коллекция (Страница {page + 1}/{total_pages}):\n\n"
    
    for card_name, count in user_cards:
        if card_name in CARDS_DATABASE:
            card_data = CARDS_DATABASE[card_name]
            craft_only = "🔨" if card_data.get('craft_only', False) else ""
            text += f"✅ {craft_only} {card_data['rarity']} {card_name} ×{count}\n"
        else:
            text += f"✅ {card_name} ×{count}\n"
    
    markup = types.InlineKeyboardMarkup()
    
    # Кнопки навигации
    nav_buttons = []
    if page > 0:
        nav_buttons.append(types.InlineKeyboardButton("⬅️ Назад", callback_data=f"page_my_{page-1}"))
    if page < total_pages - 1:
        nav_buttons.append(types.InlineKeyboardButton("Вперед ➡️", callback_data=f"page_my_{page+1}"))
    
    if nav_buttons:
        markup.row(*nav_buttons)
    
    try:
        bot.edit_message_text(text, message.chat.id, message.message_id, reply_markup=markup)
    except:
        bot.send_message(message.chat.id, text, reply_markup=markup)

# ОБРАБОТЧИК НЕИЗВЕСТНЫХ КОМАНД
@bot.message_handler(func=lambda message: message.text and message.text.startswith('/'))
@antispam_protection
def handle_unknown_command(message):
    command = message.text.split()[0]
    
    if command not in ['/start', '/promo', '/mute', '/unmute', '/mutelist', 
                      '/clearexpiredmutes', '/antispamstats', '/resetspamstats',
                      '/banbot', '/unbanbot', '/offshop', '/onshop', '/shopstatus',
                      '/clearinventory', '/removecoins', '/removecard', '/addcard',
                      '/deletecard', '/hide_card', '/unhide_card', '/cards_list',
                      '/give_coins', '/give_card', '/reset_cooldown', '/stats',
                      '/broadcast', '/promolist', '/deletepromo', '/createpromo',
                      '/editcard', '/addrecipe', '/deleterecipe', '/recipes']:
        
        bot.send_message(message.chat.id, 
                        "❌ Неизвестная команда!\n\n"
                        "✨ Доступные команды:\n"
                        "• /start - начать работу\n"
                        "• /promo КОД - активировать промокод\n\n"
                        "Используйте кнопки меню для навигации.")

# ОБРАБОТЧИК ЛЮБЫХ ДРУГИХ СООБЩЕНИЙ
@bot.message_handler(func=lambda message: True)
@antispam_protection
def handle_any_message(message):
    # Игнорируем любые другие сообщения (не команды, не кнопки)
    pass

# ЗАПУСК БОТА С ЗАГРУЗКОЙ ДАННЫХ
if __name__ == "__main__":
    # Загружаем сохраненные данные
    load_saved_data()
    
    init_db()
    logger.info("Бот запущен!")
    logger.info(f"Статус магазина: {'ВКЛЮЧЕН' if SHOP_ENABLED else 'ВЫКЛЮЧЕН'}")
    logger.info(f"Загружено карточек: {len(CARDS_DATABASE)}")
    logger.info(f"Загружено рецептов крафта: {len(CRAFT_RECIPES)}")
    
    # Запускаем периодическую очистку старых данных
    import threading
    
    def periodic_cleanup():
        while True:
            time.sleep(300)  # Каждые 5 минут
            cleanup_old_data()
            logger.debug("Очищены старые данные антиспама")
    
    cleanup_thread = threading.Thread(target=periodic_cleanup, daemon=True)
    cleanup_thread.start()
    
    while True:
        try:
            logger.info("Запуск polling...")
            bot.polling(none_stop=True, timeout=60, long_polling_timeout=30)
        except Exception as e:
            logger.error(f"Ошибка бота: {e}")
            logger.info("Перезапуск через 10 секунд...")
            time.sleep(10)
