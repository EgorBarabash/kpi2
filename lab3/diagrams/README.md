# Sequence-діаграми UC-04 "Подати заявку на турнір"

Згенеровано за `ai/prompts/prompt2.txt` (spec v2), перегенеровано за `ai/prompts/prompt3.txt` (spec v3, фідбек `ai/feedback/prompt2-1…3.md`) і `ai/prompts/prompt5.txt` (spec v4, аудит `ai/answers/questions2.md` + `answers2.md`), ADR 0012-0014. Джерела: `lab2/scenarios.feature`, `lab2/spec.md`, `lab1/artifact/artifact.mmd`.

## Індекс

| Сценарій | Файл | BR | Тип | Фрагменти |
|---|---|---|---|---|
| S-01 Подає заявку з допустимим складом | [S-01.md](S-01.md) | BR-01…BR-07 | успішний | `alt` BR-07 |
| S-02 Замінює гравця без акаунта | [S-02.md](S-02.md) | BR-03 | альтернативний | `loop` + `alt` BR-03 |
| S-03 Команда вже має заявку на розгляді | [S-03.md](S-03.md) | BR-01 | помилковий | `alt` BR-01 |
| S-04 Після відхилення - нова заявка | [S-04.md](S-04.md) | BR-01 | успішний | `alt` BR-01 |
| S-05 Гравець уже заявлений за іншу команду | [S-05.md](S-05.md) | BR-02 | помилковий | `alt` BR-02 |
| S-06 Немає гравця з акаунтом для заміни | [S-06.md](S-06.md) | BR-03 | помилковий | `alt` BR-03 |
| S-07 Турнір уже розпочато | [S-07.md](S-07.md) | BR-04 | помилковий | `alt` BR-04 |
| S-08 Кількість гравців у складі | [S-08.md](S-08.md) | BR-05 | успішний + помилковий | `alt` BR-05 (обидві гілки повні) |
| S-09 Тренер без ролі player не входить до складу | [S-09.md](S-09.md) | BR-05, BR-07 | успішний | `alt` BR-07 |
| S-10 Усі місця зайнято | [S-10.md](S-10.md) | BR-06 | помилковий | `alt` BR-06 |
| S-11 Учасник без ролі captain або coach | [S-11.md](S-11.md) | BR-07 | помилковий | `alt` BR-07 |
| S-12 Тренер подає заявку | [S-12.md](S-12.md) | BR-07 | успішний | `alt` BR-07 + `note` |

Запити до БД за викликами (атрибути ER у snake_case, параметри методів у camelCase):

| Виклик | Репозиторій | Запит до БД |
|---|---|---|
| `findActiveRoles(teamId, userId)` | TeamMemberRepository | `query(team_member + team_member_role, {team_id, user_id, status: active, unassigned_at: null})` |
| `findById(tournamentId)` | TournamentRepository | `query(tournament, {id: tournamentId})` |
| `countAccepted(tournamentId)` | ApplicationRepository | `query(tournament_application, {tournament_id, status: accepted})` |
| `findNotRejectedByTeam(tournamentId, teamId)` | ApplicationRepository | `query(tournament_application, {tournament_id, team_id, status ≠ rejected})` |
| `findActivePlayers(teamId)` | TeamMemberRepository | `query(team_member + team_member_role, {team_id, role: player, status: active, unassigned_at: null})` |
| `findUsersWithAccount(playerIds, gameId)` | GameAccountRepository | `query(game_account, {user_id in playerIds, game_id})` |
| `findUsersWithAccount(candidateIds, gameId)` (S-02) | GameAccountRepository | `query(game_account, {user_id in candidateIds, game_id})` |
| `findPlayersInNotRejected(tournamentId, playerIds)` | ApplicationRepository | `query(tournament_application + lists, {tournament_id, user_id in playerIds, status ≠ rejected})` |
| `save(application)` | ApplicationRepository | `insert(tournament_application + lists, {tournament_id, team_id, status: pending, lists: playerIds})` |

## Самоперевірка за критеріями spec v4 (розділ 6)

| # | Критерій | Стан | Примітка |
|---|---|---|---|
| 1 | Одна діаграма на S-01…S-12, у кожній `alt`/`opt`/`loop` | ✅ | 12 файлів, у кожному щонайменше один `alt` |
| 2 | `par` або `loop` у наборі; кожен `alt` з двома гілками | ✅ | `loop доки заявку не створено` у S-02; усі `alt` мають `else` |
| 3 | Activation bar охоплює фрагмент | ✅ | WebUI, Controller, Service активуються до `alt`/`loop` і деактивуються після `end` |
| 4 | Return на кожен синхронний виклик | ✅ | У коротких гілках - return помилки Service -> Controller (розділ 4: "лише return помилки") |
| 5 | Методи з параметрами; `query`/`insert` з атрибутами ER у snake_case; склад через `lists` | ✅ | Таблиця вище; склад - `tournament_application + lists`, не атрибут Tournament_Application |
| 6 | Репозиторії відповідають ER, учасники лише з розділу 3 | ✅ | Актор скрізь "<ім'я> (Представник команди)", зокрема Марта в S-11 та Ігор у S-12 |
| 7 | Кроки, статуси, тексти помилок, дані - як у Gherkin | ✅ | S-04, S-07, S-10, S-11 - "допустимий склад" без імен; у S-04 результат перевірки акаунтів - "усі мають акаунт" |
| 8 | Однаковий порядок BR, після помилки save немає | ✅ | BR-07 -> BR-04 -> BR-06 -> BR-01 -> BR-05 -> BR-03 -> BR-02 -> save |
| 9 | Немає учасників і викликів із розділу 5 | ✅ | Немає `critical`, транзакцій, аутентифікації, UC-10/UC-24 |
| 10 | Mermaid `sequenceDiagram` у ```mermaid | ✅ | Усі 12 проходять `mermaid.parse` (mermaid 11), зокрема activate/deactivate. Перетин підписів - перевірити на рендері |

## Що виправлено

### За фідбеком (prompt3, spec v3)

| Фідбек | Було (prompt2) | Стало (prompt3) | Діаграми |
|---|---|---|---|
| `prompt2-1.md` - userId Controller -> Service | Сигнатура з 4 параметрами була лише на діаграмах, spec її не називав | Spec v3, розділ 4 (S-01): WebUI -> Controller - 3 параметри, Controller -> Service - з userId. Діаграми вже так виглядали, тепер вони відповідають spec | без змін |
| `prompt2-2.md` - `findReplacementPlayers` | Виклик без репозиторію в spec, запит по Team_Member_Role і Game_Account | Кандидати = результат `findActivePlayers` мінус склад (`Note`), далі `findUsersWithAccount(candidateIds, gameId)` у GameAccountRepository; у S-06 кандидатів немає, виклик не робиться | S-02, S-06 |
| `prompt2-3.md` - `loop` у S-02 | `loop поки у складі є гравець без акаунта` - do-while | `loop доки заявку не створено` - умова істинна перед обома проходами, хибна після save | S-02 |

### За аудитом (prompt5, spec v4)

| Питання аудиту | Було (prompt3) | Стало (prompt5) | Діаграми |
|---|---|---|---|
| Q1 - фільтри TeamMemberRepository | `query(team_member_role, {teamId, userId, active})` - таких атрибутів у Team_Member_Role немає | `query(team_member + team_member_role, {team_id, user_id, status: active, unassigned_at: null})` | усі 12 |
| Q2 - склад заявки | `query(tournament_application, {userId in playerIds, …})` | `query(tournament_application + lists, {tournament_id, user_id in playerIds, status ≠ rejected})` | S-01, S-02, S-04, S-05, S-08, S-12 |
| Q3 - дані S-04 | склад Олена, Андрій, Марта, Богдан, Ірина (немає в Gherkin) | "допустимий склад", результат акаунтів - "усі мають акаунт" | S-04 |
| Q4 - актор S-11 | "Марта (учасник команди)" | "Марта (Представник команди)"; так само "Ігор (Представник команди)" у S-12 | S-11, S-12 |
| Q5 - `save` | `query(tournament_application, insert {…})` | `insert(tournament_application + lists, {tournament_id, team_id, status: pending, lists: playerIds})` | S-01, S-02, S-04, S-08, S-12 |
| Q6 - імена атрибутів | camelCase у фільтрах | snake_case з ER у `query`/`insert`, camelCase у методах | усі 12 |

Кроки, фрагменти й порядок перевірок не змінювалися. PNG у `png/` треба перерендерити для всіх 12 діаграм.

## Відкриті питання: spec не дає однозначної відповіді

1. **S-09 і контролер.** Розділ 4 (S-09) пише WebUI -> `getAvailablePlayers(teamId)` -> сервіс без контролера; на діаграмі - як у S-01: Controller -> Service `getAvailablePlayers(teamId, userId)`.
2. **S-07 у критерії 7.** Критерій перелічує S-04, S-10, S-11, але Gherkin S-07 теж "з допустимим складом"; діаграма S-07 пише без імен.
3. **Дії актора.** Повідомлення Представник -> WebUI ("надсилає заявку…") - дії користувача, а не методи; критерій 5 не уточнює, чи стосується він актора.
