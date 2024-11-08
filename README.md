import os
import random
from time import sleep
from pyrogram import Client, idle, errors
from pyrogram.types import User

# Telegram API credentials
api_id = 24916286  # Вставьте свой API ID
api_hash = "98531cb09c662bb13289cb280349d79b"  # Вставьте свой API Hash

# ANSI-коды для цвета
RED = '\033[91m'
GREEN = '\033[92m'
YELLOW = '\033[93m'
PURPLE = '\033[95m'
CYAN = '\033[96m'
ENDC = '\033[0m'

# Название файла сессии
session_name = "my_account.session"

# Функция приветствия
def display_welcome():
    print(CYAN + """
    ___            _ _    
   |_ _|_ ____   _(_) |_ ___ _ __
    | || '_ \ \ / / | __/ _ \ '__|
    | || | | \ V /| | || __/ |
   |___|_| |_|\_/ |_|\__|_|
   Создатель >> топ все круто
    """ + ENDC)

# Функция выбора сессии
def choose_session():
    print(YELLOW + "\nВыберите команду:" + ENDC)
    print("1 - Удалить сессию и создать новую")
    print("2 - Продолжить с текущей сессией")
    return input("Введите номер команды: ").strip()

# Функция отображения меню команд
def display_menu():
    print(YELLOW + "\nВыберите команду:" + ENDC)
    print("1 - Парсер (сохранить участников в файл parsi.txt)")
    print("2 - Отправить сообщение участникам из файла parsi1.txt")
    print("3 - Сохранить участников в файл pars2.txt и добавить их в другой чат")
    return input("Введите номер команды: ").strip()

# Функция выбора чата по названию
def choose_chat_by_username(app, message):
    print(PURPLE + message + ENDC)
    chat_username = input(PURPLE + "Введите название группы в формате @названиегруппы: " + ENDC).strip()
    while True:
        if chat_username.startswith("@"):
            try:
                chat = app.get_chat(chat_username)
                print(GREEN + f"Чат '{chat.title}' успешно выбран." + ENDC)
                return chat.id
            except errors.PeerIdInvalid:
                print(RED + "Неверное название группы. Попробуйте еще раз." + ENDC)
                chat_username = input(PURPLE + "Введите название группы в формате @названиегруппы: " + ENDC).strip()
        else:
            print(RED + "Неверный формат названия группы. Название должно начинаться с '@'." + ENDC)
            chat_username = input(PURPLE + "Введите название группы в формате @названиегруппы: " + ENDC).strip()

# Основная программа
try:
    # Выбор сессии
    session_choice = choose_session()

    # Удаление существующей сессии и создание новой
    if session_choice == '1':
        if os.path.exists(session_name):
            try:
                os.remove(session_name)
                print(GREEN + "Старая сессия удалена." + ENDC)
            except Exception as e:
                print(RED + f"Не удалось удалить сессию: {e}" + ENDC)
        print(PURPLE + "Введите свои данные для создания новой сессии." + ENDC)
        app = Client("my_account", api_id=api_id, api_hash=api_hash)

    # Продолжение с текущей сессией
    elif session_choice == '2':
        print(GREEN + "Вы продолжили с текущей сессией." + ENDC)
        app = Client("my_account")

    else:
        print(RED + "Неверный выбор, завершение программы." + ENDC)
        exit()

    app.start()
    display_welcome()

    while True:
        # Отображение меню команд и выбор метода
        method = display_menu()

        if method == '1':
            # Команда 1: Парсинг участников и сохранение в файл parsi.txt
            chat_id_from = choose_chat_by_username(app, "Введите название группы для парсинга участников")
            print("Загрузка участников...")

            try:
                members = list(app.get_chat_members(chat_id_from))  # Преобразуем в список
                with open("parsi.txt", "w") as f:
                    for member in members:
                        if isinstance(member.user, User) and member.user.username:
                            f.write(f"@{member.user.username}\n")
                    print(GREEN + "Список участников сохранен в 'parsi.txt'" + ENDC)
            except errors.ChatAdminRequired:
                print(RED + "Ошибка: Нужны права администратора для получения участников." + ENDC)

        elif method == '2':
            # Команда 2: Парсинг участников, сохранение в файл parsi1.txt и отправка сообщения
            chat_id_from = choose_chat_by_username(app, "Введите название группы для парсинга участников")
            print("Загрузка участников...")

            try:
                members = list(app.get_chat_members(chat_id_from))  # Преобразуем генератор в список
                with open("parsi1.txt", "w") as f:
                    for member in members:
                        if isinstance(member.user, User) and member.user.username:
                            f.write(f"@{member.user.username}\n")
                print(GREEN + f"Список участников сохранен в 'parsi1.txt'. Всего участников: {len(members)}" + ENDC)
            except errors.ChatAdminRequired:
                print(RED + "Ошибка: Нужны права администратора для получения участников." + ENDC)
                continue

            # Запрос текста сообщения
            message_text = input(YELLOW + "Введите текст для рассылки (ссылка будет добавлена автоматически): " + ENDC)

            # Запрос интервала отправки сообщений
            while True:
                try:
                    min_interval = float(input(YELLOW + "Введите минимальный интервал между сообщениями (в секундах): " + ENDC))
                    max_interval = float(input(YELLOW + "Введите максимальный интервал между сообщениями (в секундах): " + ENDC))
                    if min_interval > 0 and max_interval > min_interval:
                        break
                    else:
                        print(RED + "Минимальный интервал должен быть больше 0, а максимальный интервал должен быть больше минимального." + ENDC)
                except ValueError:
                    print(RED + "Некорректный ввод. Пожалуйста, введите число." + ENDC)

            # Создаем ссылку-приглашение для чата
            try:
                invite_link = app.export_chat_invite_link(chat_id_from)
                print(GREEN + f"Ссылка-приглашение для чата: {invite_link}" + ENDC)
            except errors.ChatAdminRequired:
                print(RED + "Ошибка: Нужны права администратора для создания ссылки-приглашения." + ENDC)
                invite_link = None
                continue

            # Запрос количества участников для отправки сообщения
            max_count = len(members)
            while True:
                try:
                    msg_count = int(input(YELLOW + f"Сколько участникам отправить сообщение? (максимум {max_count}): " + ENDC))
                    if 0 < msg_count <= max_count:
                        break
                    else:
                        print(RED + f"Пожалуйста, введите число от 1 до {max_count}." + ENDC)
                except ValueError:
                    print(RED + "Некорректный ввод. Пожалуйста, введите число." + ENDC)

            # Отправка сообщения в ЛС пользователям из файла parsi1.txt
            sent_msg_count = 0
            processed_users = []
            with open("parsi1.txt", "r") as f:
                usernames = [line.strip() for line in f if line.strip()]

            for username in usernames[:msg_count]:
                interval = random.uniform(min_interval, max_interval)  # Случайный интервал
                sleep(interval)  # Задержка между сообщениями

                try:
                    app.send_message(username, f"{message_text}\n\n{invite_link}")
                    sent_msg_count += 1
                    processed_users.append(username)
                    print(GREEN + f"Сообщение отправлено пользователю {username}" + ENDC)
                except errors.FloodWait as e:
                    print(RED + f"Превышен лимит: ждем {e.value} секунд." + ENDC)
                    sleep(e.value)
                except errors.PeerIdInvalid:
                    processed_users.append(username)
                    print(RED + f"Не удалось отправить сообщение пользователю {username}" + ENDC)
                except Exception as e:
                    processed_users.append(username)
                    print(RED + f"Ошибка при отправке сообщения {username}: {e}" + ENDC)

            # Удаление обработанных пользователей из parsi1.txt
            remaining_users = [user for user in usernames if user not in processed_users]
            with open("parsi1.txt", "w") as f:
                for user in remaining_users:
                    f.write(f"{user}\n")
            print(GREEN + f"Я удалил из parsi1.txt {len(processed_users)} пользователей, теперь в файле осталось {len(remaining_users)}." + ENDC)

        elif method == '3':
            # Команда 3: Парсинг участников и добавление их в другой чат
            chat_id_from = choose_chat_by_username(app, "Введите название группы для парсинга участников")
            print("Загрузка участников...")

            try:
                members = list(app.get_chat_members(chat_id_from))  # Преобразуем генератор в список
                with open("pars2.txt", "w") as f:
                    for member in members:
                        if isinstance(member.user, User) and member.user.username:
                            f.write(f"@{member.user.username}\n")
                print(GREEN + f"Список участников сохранен в 'pars2.txt'. Всего участников: {len(members)}" + ENDC)
            except errors.ChatAdminRequired:
                print(RED + "Ошибка: Нужны права администратора для получения участников." + ENDC)
                continue

            # Запрос количества участников для добавления
            max_count = len(members)
            while True:
                try:
                    add_count = int(input(YELLOW + f"Сколько участников добавить? (максимум {max_count}): " + ENDC))
                    if 0 < add_count <= max_count:
                        break
                    else:
                        print(RED + f"Пожалуйста, введите число от 1 до {max_count}." + ENDC)
                except ValueError:
                    print(RED + "Некорректный ввод. Пожалуйста, введите число." + ENDC)

            # Запрос интервала между добавлениями
            while True:
                try:
                    interval = float(input(YELLOW + "Введите интервал между добавлениями (в секундах): " + ENDC))
                    if interval > 0:
                        break
                    else:
                        print(RED + "Интервал должен быть положительным числом." + ENDC)
                except ValueError:
                    print(RED + "Некорректный ввод. Пожалуйста, введите число." + ENDC)

            # Выбор чата для добавления участников
            chat_id_to = choose_chat_by_username(app, "Выберите группу для добавления участников из файла pars2.txt")
            added_count = 0
            processed_users = []

            # Чтение пользователей из файла и добавление в чат с интервалом
            with open("pars2.txt", "r") as f:
                usernames = [line.strip() for line in f if line.strip()]

            for username in usernames[:add_count]:
                try:
                    sleep(interval)
                    user = app.get_users(username)  # Получаем объект пользователя по имени
                    app.add_chat_members(chat_id_to, user.id)  # Используем ID для добавления
                    added_count += 1
                    processed_users.append(username)
                    print(GREEN + f"Пользователь {username} успешно добавлен." + ENDC)
                except errors.FloodWait as e:
                    print(RED + f"Превышен лимит: ждем {e.value} секунд." + ENDC)
                    sleep(e.value)
                except errors.UserAlreadyParticipant:
                    processed_users.append(username)
                    print(YELLOW + f"Пользователь {username} уже в чате." + ENDC)
                except errors.UserNotMutualContact:
                    processed_users.append(username)
                    print(YELLOW + f"Ошибка: пользователь {username} не является взаимным контактом, пропущен." + ENDC)
                except Exception as e:
                    processed_users.append(username)
                    print(RED + f"Ошибка при добавлении пользователя {username}: {e}" + ENDC)

            # Удаление обработанных пользователей из pars2.txt
            remaining_users = [user for user in usernames if user not in processed_users]
            with open("pars2.txt", "w") as f:
                for user in remaining_users:
                    f.write(f"{user}\n")
            print(GREEN + f"Я удалил из pars2.txt {len(processed_users)} пользователей, теперь в файле осталось {len(remaining_users)}." + ENDC)

        # Запрос на продолжение
        continue_operation = input("Хотите продолжить? (да/нет): ").strip().lower()
        if continue_operation != 'да':
            break

    app.stop()
except Exception as e:
    print(RED + f"Ошибка: {e}" + ENDC)
finally:
    idle()
