# Аудит діаграм (після prompt3) проти ER і Gherkin: питання й варіанти

Звірено: `lab3/diagrams/S-01.md` … `S-12.md` ↔ `lab1/spec.md`, `lab1/artifact/artifact.mmd` ↔ `lab2/scenarios.feature`, `lab2/spec.md` (BR).
Рішення не ухвалюю - лише питання й варіанти. Відповіді - у `answers2.md`.

## Що узгоджено (перевірено, питань немає)

- TournamentRepository: id, status, game_id, min_team_players, max_team_players, max_teams - усі є в Tournament. BR-05 бере ліміт турніру, а не `Team.max_team_players`, як у BR.
- ApplicationRepository `countAccepted`, `findNotRejectedByTeam`: tournament_id, team_id, status є в Tournament_Application. `tournament_participant_status` має ще `invalid`; фільтр `status ≠ rejected` враховує його як "не rejected" - це збігається з BR-01 і BR-02.
- GameAccountRepository: user_id, game_id є в Game_Account.
- Team.status active не перевіряється - це передумова UC-04 (розділ 3).
- Дані й тексти помилок S-01, S-02, S-03, S-05, S-06, S-07, S-08, S-10, S-12 збігаються з Gherkin. Кандидати в S-02 (Ірина, Тарас, Василь, Олег) випливають з таблиці гравців Передумови.

## Питання

**Q1. Фільтри TeamMemberRepository і атрибути ER - розділ 3, критерії 5 і 6.**
`query(team_member_role, {teamId, userId, active})` і `{teamId, role: player, active}`: у Team_Member_Role є лише team_member_id, role, assigned_at, unassigned_at. team_id, user_id і status (active) належать Team_Member, активність ролі - `unassigned_at` порожній. Запит фактично йде по двох сутностях, а розділ 3 каже "один запит".
- A) Дозволити в розділі 3 запит по двох сутностях: `query(team_member + team_member_role, {team_id, user_id, status: active, unassigned_at: null})`.
- B) Два запити: спершу Team_Member (`team_id`, `user_id`, `status: active`), потім Team_Member_Role (`team_member_id`, `unassigned_at: null`) - більше стрілок, але кожен запит по одній сутності.
- C) Лишити як є, але в розділі 3 записати, що `team_member_role` у фільтрі - узагальнена назва для Team_Member + Team_Member_Role.

**Q2. Склад заявки як атрибут Tournament_Application - розділ 3 (ApplicationRepository), критерій 6.**
`findPlayersInNotRejected` фільтрує `tournament_application` за `userId`, `save` вставляє `playerIds`. В ER склад - N:M-зв'язок `Tournament_Application }o--o{ User : lists` без власних атрибутів (lab1 spec, зв'язок 4), user_id у Tournament_Application немає.
- A) Підписувати запит по зв'язку явно: `query(tournament_application + lists, {tournament_id, user_id in playerIds, status ≠ rejected})`.
- B) `save` показати двома запитами: заявка (`tournament_application`) і склад (`lists`).
- C) Лишити як є й записати в розділі 3, що склад зберігається разом із заявкою, тож фільтр за гравцями - частина запиту до `tournament_application`.

**Q3. Дані S-04, яких немає в Gherkin - розділ 4 (S-04), критерій 7.**
Gherkin S-04: "з допустимим складом", без імен. Діаграма задає склад Олена, Андрій, Марта, Богдан, Ірина і такий самий результат `findUsersWithAccount`. У S-07, S-10, S-11 діаграми вже пишуть "з допустимим складом".
- A) Правило в критерії 7: якщо Gherkin не називає склад, діаграма пише "допустимий склад" без імен.
- B) Назвати склад у spec (розділ 4, S-04) явно - тоді це рішення spec, а не діаграми.

**Q4. Актор у S-11 - розділ 3, критерій 6.**
Підпис "Марта (учасник команди)" не збігається з розділом 3, де актор лише "Представник команди". Марта якраз не має ролі, що робить її Представником.
- A) Підписати "Марта (Представник команди)" - актор той самий, роль перевіряє BR-07.
- B) Додати в розділ 3 примітку, що в S-11 актор - учасник без ролі captain/coach.

**Q5. `save` і формат `query(таблиця, filter)` - критерій 5.**
Для вставки фільтра немає; на діаграмах `query(tournament_application, insert {...})`.
- A) Дописати в критерій 5 формат для вставки, наприклад `insert(таблиця, дані)`.
- B) Лишити `query(таблиця, insert {...})` і записати це в розділі 3.

**Q6. Імена атрибутів у фільтрах - критерій 5.**
У діаграмах camelCase (`tournamentId`), в ER snake_case (`tournament_id`). Spec не задає формат.
- A) У фільтрах `query` - імена з ER (snake_case), у викликах методів - camelCase.
- B) Лишити camelCase скрізь.
