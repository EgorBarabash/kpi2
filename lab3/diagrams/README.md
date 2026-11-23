# Sequence-діаграми UC-04 "Подати заявку на турнір"

Згенеровано за `ai/prompts/prompt2.txt` (spec v2, ADR 0012-0014). Джерела: `lab2/scenarios.feature`, `lab2/spec.md`, `lab1/artifact/artifact.mmd`.

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

Запити до БД за викликами:

| Виклик | Репозиторій | `query(таблиця, filter)` |
|---|---|---|
| `findActiveRoles(teamId, userId)` | TeamMemberRepository | `team_member_role, {teamId, userId, active}` |
| `findById(tournamentId)` | TournamentRepository | `tournament, {id: tournamentId}` |
| `countAccepted(tournamentId)` | ApplicationRepository | `tournament_application, {tournamentId, status: accepted}` |
| `findNotRejectedByTeam(tournamentId, teamId)` | ApplicationRepository | `tournament_application, {tournamentId, teamId, status ≠ rejected}` |
| `findActivePlayers(teamId)` | TeamMemberRepository | `team_member_role, {teamId, role: player, active}` |
| `findUsersWithAccount(playerIds, gameId)` | GameAccountRepository | `game_account, {userId in playerIds, gameId}` |
| `findReplacementPlayers(teamId, gameId, playerIds)` | TeamMemberRepository | `team_member_role, {teamId, role: player, active, account gameId, userId not in playerIds}` |
| `findPlayersInNotRejected(tournamentId, playerIds)` | ApplicationRepository | `tournament_application, {tournamentId, userId in playerIds, status ≠ rejected}` |
| `save(application)` | ApplicationRepository | `tournament_application, insert {teamId, tournamentId, status: pending, playerIds}` |

## Самоперевірка за критеріями spec (розділ 6)

| # | Критерій | Стан | Примітка |
|---|---|---|---|
| 1 | Одна діаграма на S-01…S-12, у кожній `alt`/`opt`/`loop` | ✅ | 12 файлів, у кожному щонайменше один `alt` |
| 2 | `par` або `loop` у наборі; кожен `alt` з двома гілками | ✅ | `loop` у S-02; усі `alt` мають `else` |
| 3 | Activation bar охоплює фрагмент | ✅ | WebUI, Controller, Service активуються до `alt`/`loop` і деактивуються після `end` |
| 4 | Return на кожен синхронний виклик | ⚠️ | У коротких гілках є лише return `Service -> Controller` (spec: "лише return помилки"), без `Controller -> WebUI` |
| 5 | Виклики - методи з параметрами, БД - `query(таблиця, filter)` | ⚠️ | Див. відкриті питання 1, 2, 4 |
| 6 | Репозиторії відповідають ER, учасники лише з розділу 3 | ⚠️ | Див. відкрите питання 3 |
| 7 | Кроки, статуси, тексти помилок збігаються з Gherkin, `ApplicationError(BR-xx)` | ✅ | Дані: Олена/Степан/Тарас, "Сови"/Сергій, "Леви"/Андрій, "Літній кубок" ongoing, "Зимовий кубок" 16/16 |
| 8 | Однаковий порядок BR, після помилки save немає | ✅ | BR-07 -> BR-04 -> BR-06 -> BR-01 -> BR-05 -> BR-03 -> BR-02 -> save; у помилкових гілках `note` "save не викликається" |
| 9 | Немає учасників і викликів із розділу 5 | ✅ | Немає `critical`, транзакцій, аутентифікації, UC-10/UC-24 |
| 10 | Mermaid `sequenceDiagram` у ```mermaid | ✅ | Усі 12 проходять `mermaid.parse` (mermaid 11), зокрема перевірку activate/deactivate. Перетин підписів треба переглянути на рендері (GitHub) |

## Відкриті питання: spec не дає однозначної відповіді

1. **userId у сервісі.** Spec: WebUI -> `submitApplication(tournamentId, teamId, playerIds)`, а `findActiveRoles(teamId, userId)` потребує userId із сесії. На діаграмах: WebUI -> Controller - 3 параметри, Controller -> Service - `submitApplication(..., userId)` (так само `getAvailablePlayers(teamId, userId)` у S-09). Spec цієї сигнатури не називає.
2. **`save` і формат `query(таблиця, filter)`.** Для вставки фільтра немає; використано `query(tournament_application, insert {...})`.
3. **Репозиторій для `findReplacementPlayers`.** Spec не каже, якому репозиторію належить виклик. Обрано TeamMemberRepository, але запит зачіпає дві сутності ER (Team_Member_Role і Game_Account), а розділ 3 каже "один запит" на репозиторій.
4. **Дії актора.** Повідомлення Представник -> WebUI ("надсилає заявку…") - дії користувача, а не методи; критерій 5 не уточнює, чи стосується він актора.
5. **Семантика `loop` у S-02.** Умова "поки у складі є гравець без акаунта" перевіряється після проходу: 2-й прохід (з Тарасом) теж усередині циклу й завершується save. Це do-while, а не while.
6. **Коротка гілка S-06.** Протилежна гілка `[є кандидати]` - це S-02; показано лише return `ApplicationError(BR-03)` про Сергія.
