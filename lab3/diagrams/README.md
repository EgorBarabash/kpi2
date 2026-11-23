# Sequence-діаграми UC-04 "Подати заявку на турнір"

Згенеровано за `ai/prompts/prompt2.txt` (spec v2), перегенеровано за `ai/prompts/prompt3.txt` (spec v3, фідбек `ai/feedback/prompt2-1…3.md`), ADR 0012-0014. Джерела: `lab2/scenarios.feature`, `lab2/spec.md`, `lab1/artifact/artifact.mmd`.

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
| `findUsersWithAccount(candidateIds, gameId)` (S-02) | GameAccountRepository | `game_account, {userId in candidateIds, gameId}` |
| `findPlayersInNotRejected(tournamentId, playerIds)` | ApplicationRepository | `tournament_application, {tournamentId, userId in playerIds, status ≠ rejected}` |
| `save(application)` | ApplicationRepository | `tournament_application, insert {teamId, tournamentId, status: pending, playerIds}` |

## Самоперевірка за критеріями spec (розділ 6)

| # | Критерій | Стан | Примітка |
|---|---|---|---|
| 1 | Одна діаграма на S-01…S-12, у кожній `alt`/`opt`/`loop` | ✅ | 12 файлів, у кожному щонайменше один `alt` |
| 2 | `par` або `loop` у наборі; кожен `alt` з двома гілками | ✅ | `loop [доки заявку не створено]` у S-02; усі `alt` мають `else` |
| 3 | Activation bar охоплює фрагмент | ✅ | WebUI, Controller, Service активуються до `alt`/`loop` і деактивуються після `end` |
| 4 | Return на кожен синхронний виклик | ⚠️ | У коротких гілках є лише return `Service -> Controller` (spec: "лише return помилки"), без `Controller -> WebUI` |
| 5 | Виклики - методи з параметрами, БД - `query(таблиця, filter)` | ⚠️ | Сигнатури Controller -> Service тепер зі spec (розділ 4, S-01); лишаються відкриті питання 1, 2 |
| 6 | Репозиторії відповідають ER, учасники лише з розділу 3 | ✅ | `findReplacementPlayers` прибрано; кожен виклик належить репозиторію з розділу 3 |
| 7 | Кроки, статуси, тексти помилок збігаються з Gherkin, `ApplicationError(BR-xx)` | ✅ | Дані: Олена/Степан/Тарас, "Сови"/Сергій, "Леви"/Андрій, "Літній кубок" ongoing, "Зимовий кубок" 16/16 |
| 8 | Однаковий порядок BR, після помилки save немає | ✅ | BR-07 -> BR-04 -> BR-06 -> BR-01 -> BR-05 -> BR-03 -> BR-02 -> save; у помилкових гілках `note` "save не викликається" |
| 9 | Немає учасників і викликів із розділу 5 | ✅ | Немає `critical`, транзакцій, аутентифікації, UC-10/UC-24 |
| 10 | Mermaid `sequenceDiagram` у ```mermaid | ✅ | Усі 12 проходять `mermaid.parse` (mermaid 11), зокрема перевірку activate/deactivate. Перетин підписів треба переглянути на рендері (GitHub) |

## Що виправлено за фідбеком

| Фідбек | Було (prompt2) | Стало (prompt3) | Діаграми |
|---|---|---|---|
| `prompt2-1.md` - userId Controller -> Service | Сигнатура з 4 параметрами була лише на діаграмах, spec її не називав | Spec v3, розділ 4 (S-01): WebUI -> Controller - 3 параметри, Controller -> Service - з userId. Діаграми вже так виглядали, тепер вони відповідають spec | без змін |
| `prompt2-2.md` - `findReplacementPlayers` | Виклик без репозиторію в spec, запит по Team_Member_Role і Game_Account | Кандидати = результат `findActivePlayers` мінус склад (`Note`), далі `findUsersWithAccount(candidateIds, gameId)` у GameAccountRepository; у S-06 кандидатів немає, виклик не робиться | S-02, S-06 |
| `prompt2-3.md` - `loop` у S-02 | `loop поки у складі є гравець без акаунта` - do-while | `loop доки заявку не створено` - умова істинна перед обома проходами, хибна після save | S-02 |

Решту діаграм (S-01, S-03…S-05, S-07…S-12) не змінено. Усі 12 знову проходять `mermaid.parse`. PNG у `png/` для S-02 і S-06 треба перерендерити.

## Відкриті питання: spec не дає однозначної відповіді

1. **`save` і формат `query(таблиця, filter)`.** Для вставки фільтра немає; використано `query(tournament_application, insert {...})`.
2. **Дії актора.** Повідомлення Представник -> WebUI ("надсилає заявку…") - дії користувача, а не методи; критерій 5 не уточнює, чи стосується він актора.
3. **S-09 і userId.** Spec (розділ 4, S-09) лишає WebUI -> `getAvailablePlayers(teamId)` -> сервіс без контролера; на діаграмі - так само, як у S-01: Controller -> Service `getAvailablePlayers(teamId, userId)`.
4. **Коротка гілка S-06.** Протилежна гілка `[є кандидати]` - це S-02; показано лише return `ApplicationError(BR-03)` про Сергія.
