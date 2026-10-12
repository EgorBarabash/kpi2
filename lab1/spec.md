Намір
Побудувати концептуальну модель сутностей і зв'язків (ERD) для платформи аматорських кіберспортивних турнірів, щоб відобразити повний життєвий цикл турніру - від складу команди та подачі заявки до турнірної сітки, результатів матчів і статистики гравців - без деталей фізичної реалізації бази даних.

Типи
team_status: "active", "disbanded"
team_member_status: "active", "left", "kicked"
tournament_status: "registration", "ongoing", "finished"
tournament_participant_status: "pending", "accepted", "rejected"
encounter_status: "ongoing", "scheduled", "finished"
team_role: "player", "captain"

Сутності

User (Користувач) - зареєстрована особа: гравець або організатор. id (PK), email (UK), hashed_password, nickname (UK), name, description, avatar_url, created_at.

Game_Account (Ігровий акаунт) - прив'язка користувача до його акаунта в конкретній грі. user_id (PK, FK), game_name, game_account_id (UK).

Team (Команда) - група гравців, що виступає разом. id (PK), name (UK), avatar_url, created_at, max_team_players, team_status.

Team_Member (Учасник команди) - членство користувача в команді з роллю та історією. id (PK), team_id (FK), user_id (FK), joined_at, leave_at, team_member_status, team_role.

Tournament (Турнір) - змагання з однієї гри, яке створює організатор. id (PK), organizer_id (FK), game_name, name, created_at, start_at, prize_pool, currency, min_teams, max_teams, min_team_players, max_team_players, tournament_status.

Tournament_Application (Заявка на турнір) - заявка команди на участь у турнірі. tournament_id (PK, FK), team_id (PK, FK), tournament_participant_status.

Tournament_Application_Member (Гравець заявки) - гравець, заявлений у складі команди на турнір. tournament_id (PK, FK), tournament_application_id (FK), user_id (PK, FK).

Encounter (Зустріч) - протистояння двох заявок у турнірній сітці. id (PK), tournament_id (FK), stage, tournament_application_id_1 (FK), tournament_application_id_2 (FK), winner_tournament_application_id (FK), next_encounter_id (FK), encounter_status.

Match (Матч) - окрема гра (мапа) в межах зустрічі. id (PK), encounter_id (FK), match_game_id, map_order, winner_tournament_application_id (FK); пара (encounter_id, map_order) унікальна.

Player_Statistic (Статистика гравця) - показники гравця в конкретному матчі. match_id (PK, FK), user_id (PK, FK), kills, deaths, assists.

Зв'язки
1. Гравець може бути учасником багатьох команд, і команда може містити багатьох учасників (N:M, реалізовано через сутність Team_Member).
2. Користувач може створити багато турнірів, але турнір має лише одного організатора (1:N).
3. Команда може подати заявку на багато турнірів, і турнір містить багато команд-учасників (N:M, реалізовано через сутність Tournament_Application).
4. Один користувач може бути зареєстрованим у багатьох заявках на турніри від команди, та одна заявка від команди на турнір може містити багато користувачів (N:M, реалізовано через сутність Tournament_Application_Member).
5. Турнір може містити багато Tournament_Application_Member, але кожен Tournament_Application_Member належить лише одному турніру (1:N).
6. Турнір містить багато зустрічей, кожна зустріч належить строго одному турніру (1:N).
7. Заявка команди може брати участь у багатьох зустрічах, а в зустрічі беруть участь дві заявки (1:N, оскільки команд завжди дві й одна з них переможець, у сутність Encounter додано три атрибути: ідентифікатор першої команди, другої команди та команди-переможця).
8. Зустріч має посилання на наступну зустріч у сітці для автоматичного просування переможця (рекурсивний зв'язок 1:N).
9. Зустріч складається з кількох окремих матчів, кожен матч належить строго одній зустрічі (1:N).
10. Кожен матч містить статистику для багатьох гравців, а гравець має статистику в багатьох матчах (N:M, реалізовано через сутність Player_Statistic).
11. Користувач може мати багато ігрових акаунтів, кожен акаунт належить одному користувачу (1:N).

Критерії прийняття
1. Результат має бути згенерований у декларативному синтаксисі Mermaid (тип erDiagram).
2. Усі первинні ключі (PK) повинні мати назву id та тип даних uuid.
3. Текст зв'язків у синтаксисі Mermaid має бути англійською та без лапок.
4. Зв'язки між сутностями мають бути показані концептуально (через кардинальність), без використання суто фізичних сполучних (асоціативних) таблиць.
