**Q1. Фільтри TeamMemberRepository і атрибути ER**
**Моя відповідь:** A. Один запит по двох сутностях: `query(team_member + team_member_role, {team_id, user_id, status: active, unassigned_at: null})`.

**Q2. Склад заявки як атрибут Tournament_Application**
**Моя відповідь:** A. Запит по зв'язку явно: `query(tournament_application + lists, {tournament_id, user_id in playerIds, status ≠ rejected})`.

**Q3. Дані S-04, яких немає в Gherkin**
**Моя відповідь:** A. Правило в критерії 7: якщо Gherkin не називає склад, діаграма пише "допустимий склад" без імен і не вигадує даних.

**Q4. Актор у S-11**
**Моя відповідь:** A. "Марта (Представник команди)" - актор той самий, що в розділі 3; що в Марти немає ролі captain або coach, показує BR-07.

**Q5. `save` і формат `query(таблиця, filter)`**
**Моя відповідь:** A. Окремий формат для вставки: `insert(tournament_application + lists, {tournament_id, team_id, status: pending, lists: playerIds})`.

**Q6. Імена атрибутів у фільтрах**
**Моя відповідь:** A. У фільтрах `query` та `insert` - імена з ER (snake_case), у викликах методів - camelCase.
