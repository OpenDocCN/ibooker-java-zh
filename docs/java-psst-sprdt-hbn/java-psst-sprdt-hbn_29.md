# 附录 C. Spring Data JDBC 关键字使用

表 C.1 描述了 Spring Data JDBC 支持的关键字以及每个方法名如何生成查询条件。

表 C.1 Spring Data JDBC 中关键字的使用及其生成的条件

| 关键字 | 示例 | 条件 |
| --- | --- | --- |
| `Is`, `Equals` | `findByUsername(String name)` `findByUsernameIs(String name)` `findByUsernameEquals(String name)` | `username = name` |
| `And` | `findByUsernameAndRegistrationDate(String name, LocalDate date)` | `username = name and registrationDate = date` |
| `Or` | `findByUsernameOrRegistrationDate(String name, LocalDate date)` | `username = name or registrationDate = name` |
| `LessThan` | `findByRegistrationDateLessThan(LocalDate date)` | `registrationDate < date` |
| `LessThanEqual` | `findByRegistrationDateLessThanEqual(LocalDate date)` | `registrationDate <= date` |
| `GreaterThan` | `findByRegistrationDateGreaterThan(LocalDate date)` | `registrationDate > date` |
| `GreaterThanEqual` | `findByRegistrationDateGreaterThanEqual(LocalDate date)` | `registrationDate >= date` |
| `Between` | `findByRegistrationDateBetween(LocalDate from, LocalDate to)` | `registrationDate between from and to` |
| `OrderBy` | `findByRegistrationDateOrderByUsernameDesc(LocalDate date)` | `registrationDate = date order by username desc` |
| `Like` | `findByUsernameLike(String name)` | `username like name` |
| `NotLike` | `findByUsernameNotLike(String name)` | `username not like name` |
| `Before` | `findByRegistrationDateBefore(LocalDate date)` | `registrationDate < date` |
| `After` | `findByRegistrationDateAfter(LocalDate date)` | `registrationDate > date` |
| `Null`, `IsNull` | `findByRegistrationDate(Is)Null()` | `registrationDate is null` |
| `NotNull`, `IsNotNull` | `findByRegistrationDate(Is)NotNull()` | `registrationDate is not null` |
| `Not` | `findByUsernameNot(String name)` | `username <> name` |
| `In` | `findByRegistrationDateIn`(Collection<LocalDate> dates) | `registrationDate in (date1,  . . .  dateN)` |
| `NotIn` | `findByRegistrationDateNotIn`(Collection<LocalDate> dates) | `registrationDate not in (date1,  . . .  dateN)` |
| `True`, `IsTrue` | `findByActiveTrue()` | `active is true` |
| `False`, `IsFalse` | `findByActiveFalse()` | `active is false` |
| `StartingWith` | `findByUsernameStartingWith(String name)` | `username like name%` |
| `EndingWith` | `findByUsernameEndingWith(String name)` | `username like %name` |
| `Containing` | `findByUsernameContaining(String name)` | `username like %name%` |
| `IgnoreCase` | `findByUsernameIgnoreCase(String name)` | `UPPER(username) = UPPER(name)` |
