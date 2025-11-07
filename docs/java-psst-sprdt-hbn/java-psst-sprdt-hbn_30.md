# 附录 D. Spring Data MongoDB 关键字用法

表 D.1 描述了 Spring Data MongoDB 支持的关键字以及每个方法名称如何生成查询条件。

表 D.1 Spring Data MongoDB 关键字用法及其生成的条件

| 关键字 | 示例 | 条件 |
| --- | --- | --- |
| `Is`, `Equals` | `findByUsername(String name)``findByUsernameIs(String name)``findByUsernameEquals(String name)` | `{"username":"name"}` |
| `And` | `findByUsernameAndEmail(String username, String email)` | `{"username":"username", "email":"email"}` |
| `Or` | `findByUsernameOrEmail (String username, String email)` | `{ "$or" : [{ "username" : "username"}, { "email" : "email"}]}` |
| `LessThan` | `findByRegistrationDateLessThan(LocalDate date)` | `{ "registrationDate" : { "$lt" : { "$date" : "date"}}}` |
| `LessThanEqual` | `findByRegistrationDateLessThanEqual(LocalDate date)` | `{ "registrationDate" : { "$lte" : { "$date" : "date"}}}` |
| `GreaterThan` | `findByRegistrationDateGreaterThan(LocalDate date)` | `{ "registrationDate" : { "$gt" : { "$date" : "date"}}}` |
| `GreaterThanEqual` | `findByRegistrationDateGreaterThanEqual(LocalDate date)` | `{ "registrationDate" : { "$gte" : { "$date" : "date"}}}` |
| `Between` | `findByRegistrationDateBetween(LocalDate from, LocalDate to)` | `"registrationDate" : { "$gte" : { "$date" : "date"}, "$lte" : { "$date" : "date"}}}` |
| `OrderBy` | `findByRegistrationDateOrderByUsernameDesc(LocalDate date)` | `"registrationDate" : { "$date" : "date"}}` |
| `Like` | `findByUsernameLike(String name)` | `{ "username" : { "$regularExpression" : { "pattern" : "name", "options" : ""}}}` |
| `NotLike` | `findByUsernameNotLike(String name)` | `{ "username" : { "$not" : { "$regularExpression" : { "pattern" : "name", "options" : ""}}}}` |
| `Before` | `findByRegistrationDateBefore(LocalDate date)` | `{ "registrationDate" : { "$lt" : { "$date" : "date"}}}` |
| `After` | `findByRegistrationDateAfter(LocalDate date)` | `{ "registrationDate" : { "$gt" : { "$date" : "date"}}}` |
| `Null`, `IsNull` | `findByRegistrationDate(Is)Null()` | `{ "registrationDate" : null}` |
| `NotNull`, `IsNotNull` | `findByRegistrationDate(Is)NotNull()` | `{ "registrationDate" : { "$ne" : null}}` |
| `Not` | `findByUsernameNot(String name)` | `{ "username" : { "$ne" : "name"}}` |
| `In` | `findByRegistrationDateIn`(Collection<LocalDate> dates)` | `"registrationDate" : { "$in" : [{ "$date" : "date1"},  .  { "$date" : "daten"}]}}` |
| `NotIn` | `findByRegistrationDateNotIn`(Collection<LocalDate> dates)` | `"registrationDate" : { "$nin" : [{ "$date" : "date1"},  . . .  { "$date" : "daten"}]}}` |
| `True`, `IsTrue` | `findByActiveTrue()` | `{ "active" : true}` |
| `False`, `IsFalse` | `findByActiveFalse()` | `{ "active" : false}` |
| `StartingWith` | `findByUsernameStartingWith(String name)` | `{ "username" : { "$regularExpression" : { "pattern" : "^name", "options" : ""}}}` |
| `以...结尾` | `findByUsernameEndingWith(String name)` | `{ "username" : { "$regularExpression" : { "pattern" : "name$", "options" : ""}}}` |
| `包含` | `findByUsernameContaining(String name)` | `{ "pattern" : ".*name.*", "options" : ""}}}` |
| `忽略大小写` | `findByUsernameIgnoreCase(String name)` | `{ "username" : { "$regularExpression" : { "pattern" : "^name$", "options" : "i"}}}` |
