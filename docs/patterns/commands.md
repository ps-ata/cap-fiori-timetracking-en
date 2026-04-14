# 🎯 3. Command Pattern (10 Commands!)

**Files:** `srv/handler/commands/*.ts`

commands encapsulate complex business operations:

| Command                      | Purpose                                        |
| ---------------------------- | ---------------------------------------------- |
| `CreateTimeEntryCommand`     | validation, user lookup, factory, calculation  |
| `UpdateTimeEntryCommand`     | change detection, recalculation                |
| `GenerateMonthlyCommand`     | generate month with stats                      |
| `GenerateYearlyCommand`      | generate year with holidays                    |
| `GetDefaultParamsCommand`    | default parameters for generation              |
| `GetMonthlyBalanceCommand`   | monthly balance with criticality               |
| `GetCurrentBalanceCommand`   | cumulative total balance                       |
| `GetRecentBalancesCommand`   | historical balances (6 months)                 |
| `GetVacationBalanceCommand`  | vacation balance calculation                   |
| `GetSickLeaveBalanceCommand` | sick leave calculation                         |
