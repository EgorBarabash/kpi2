# Специфікація ERD платформи аматорських кіберспортивних турнірів

## Намір

Побудувати концептуальну модель сутностей і зв'язків (ERD) для платформи аматорських кіберспортивних турнірів, щоб відобразити повний життєвий цикл турніру - від складу команди та подачі заявки до турнірної сітки, результатів матчів і статистики гравців - без деталей фізичної реалізації бази даних.

## Типи

- **team_status**: active, disbanded
- **team_member_status**: active, left, kicked
- **tournament_status**: registration, ongoing, finished
- **tournament_participant_status**: pending, accepted, rejected
- **encounter_status**: ongoing, scheduled, finished
- **team_role**: player, captain

## Сутності

**User (Користувач)** - зареєстрована особа: гравець або організатор.
Атрибути: id (PK), email (UK), hashed_password, nickname (UK), name, description, avatar_url, created_at.

**Game_Account (Ігровий акаунт)** - прив'язка користувача до його акаунта в конкретній грі.
Атрибути: user_id (PK, FK), game_name, game_account_id (UK).

**Team (Команда)** - група гравців, що виступає разом.
Атрибути: id (PK), name (UK), avatar_url, created_at, max_team_players, team_status.

**Team_Member (Учасник команди)** - членство користувача в команді з роллю та історією.
Атрибути: id (PK), team_id (FK), user_id (FK), joined_at, leave_at, team_member_status, team_role.

**Tournament (Турнір)** - змагання з однієї гри, яке створює організатор.
Атрибути: id (PK), organizer_id (FK), game_name, name, created_at, start_at, prize_pool, currency, min_teams, max_teams, min_team_players, max_team_players, tournament_status.

**Tournament_Application (Заявка на турнір)** - заявка команди на участь у турнірі.
Атрибути: tournament_id (PK, FK), team_id (PK, FK), tournament_participant_status.

**Tournament_Application_Member (Гравець заявки)** - гравець, заявлений у складі команди на турнір.
Атрибути: tournament_id (PK, FK), tournament_application_id (FK), user_id (PK, FK).

**Encounter (Зустріч)** - протистояння двох заявок у турнірній сітці.
Атрибути: id (PK), tournament_id (FK), stage, tournament_application_id_1 (FK), tournament_application_id_2 (FK), winner_tournament_application_id (FK), next_encounter_id (FK), encounter_status.

**Match (Матч)** - окрема гра (мапа) в межах зустрічі.
Атрибути: id (PK), encounter_id (FK), match_game_id, map_order, winner_tournament_application_id (FK).
Пара (encounter_id, map_order) унікальна.

**Player_Statistic (Статистика гравця)** - показники гравця в конкретному матчі.
Атрибути: match_id (PK, FK), user_id (PK, FK), kills, deaths, assists.

## Зв'язки

1. **User - Team (N:M)**. Гравець може бути учасником багатьох команд, і команда може містити багатьох учасників. Реалізовано через сутність Team_Member.
2. **User - Tournament (1:N)**. Користувач може створити багато турнірів, але турнір має лише одного організатора.
3. **Team - Tournament (N:M)**. Команда може подати заявку на багато турнірів, і турнір містить багато команд-учасників. Реалізовано через сутність Tournament_Application.
4. **User - Tournament_Application (N:M)**. Один користувач може бути зареєстрованим у багатьох заявках на турніри від команди, а одна заявка від команди на турнір може містити багато користувачів. Реалізовано через сутність Tournament_Application_Member.
5. **Tournament - Tournament_Application_Member (1:N)**. Турнір може містити багато Tournament_Application_Member, але кожен Tournament_Application_Member належить лише одному турніру.
6. **Tournament - Encounter (1:N)**. Турнір містить багато зустрічей, кожна зустріч належить строго одному турніру.
7. **Tournament_Application - Encounter (1:N)**. Заявка команди може брати участь у багатьох зустрічах, а в зустрічі беруть участь дві заявки. Оскільки команд завжди дві й одна з них переможець, у сутність Encounter додано три атрибути: ідентифікатор першої команди, другої команди та команди-переможця.
8. **Encounter - Encounter (1:N, рекурсивний)**. Зустріч має посилання на наступну зустріч у сітці для автоматичного просування переможця.
9. **Encounter - Match (1:N)**. Зустріч складається з кількох окремих матчів, кожен матч належить строго одній зустрічі.
10. **Match - User (N:M)**. Кожен матч містить статистику для багатьох гравців, а гравець має статистику в багатьох матчах. Реалізовано через сутність Player_Statistic.
11. **User - Game_Account (1:N)**. Користувач може мати багато ігрових акаунтів, кожен акаунт належить одному користувачу.

## Критерії прийняття

1. Зв'язки між сутностями мають бути показані концептуально (через кардинальність), без використання суто фізичних сполучних (асоціативних) таблиць.
2. Усі первинні ключі (PK) повинні мати назву id та тип даних uuid.
3. Текст зв'язків у синтаксисі Mermaid має бути англійською та без лапок.