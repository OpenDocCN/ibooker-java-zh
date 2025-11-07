# 附录 B. Spring Data JPA 关键字使用

表 B.1 描述了 Spring Data JPA 支持的关键字以及每个方法名如何在 JPQL 中转换。

表 B.1 Spring Data JPA 和生成的 JPQL 中的关键字使用

| 关键词 | 示例 | 生成的 JPQL |
| --- | --- | --- |
| `Is`, `Equals` | `findByUsername`, `findByUsernameIs`, `findByUsernameEquals` | `. . . where e.username = ?1` |
| `And` | `findByUsernameAndRegistrationDate` | `. . . where e.username = ?1 and e.registrationDate = ?2` |
| `Or` | `findByUsernameOrRegistrationDate` | `. . . where e.username = ?1 or e.registrationDate = ?2` |
| `LessThan` | `findByRegistrationDateLessThan` | `. . . where e.registrationDate < ?1` |
| `LessThanEqual` | `findByRegistrationDateLessThanEqual` | `. . . where e.registrationDate <= ?1` |
| `GreaterThan` | `findByRegistrationDateGreaterThan` | `. . . where e.registrationDate > ?1` |
| `GreaterThanEqual` | `findByRegistrationDateGreaterThanEqual` | `. . . where e.registrationDate >= ?1` |
| `Between` | `findByRegistrationDateBetween` | `. . . where e.registrationDate between ?1 and ?2` |
| `OrderBy` | `findByRegistrationDateOrderByUsernameDesc` | `. . . where e.registrationDate = ?1 order by e.username desc` |
| `Like` | `findByUsernameLike` | `. . . where e.username like ?1` |
| `NotLike` | `findByUsernameNotLike` | `. . . where e.username not like ?1` |
| `Before` | `findByRegistrationDateBefore` | `. . . where e.registrationDate < ?1` |
| `After` | `findByRegistrationDateAfter` | `. . . where e.registrationDate > ?1` |
| `Null`, `IsNull` | `findByRegistrationDate(Is)Null` | `. . . where e.registrationDate is null` |
| `NotNull`, `IsNotNull` | `findByRegistrationDate(Is)NotNull` | `. . . where e.registrationDate is not null` |
| `Not` | `findByUsernameNot` | `. . . where e.username <> ?1` |
| `In` | `findByRegistrationDateIn`(Collection<LocalDate> dates) | `. . . where e.registrationDate in ?1` |
| `NotIn` | `findByRegistrationDateNotIn`(Collection<LocalDate> dates) | `. . . where e.registrationDate not in ?1` |
| `True` | `findByActiveTrue` | `. . . where e.active = true` |
| `False` | `findByActiveFalse` | `. . . where e.active = false` |
| `StartingWith` | `findByUsernameStartingWith` | `. . . where e.username like ?1%` |
| `EndingWith` | `findByUsernameEndingWith` | `. . . where e.username like %?1` |
| `Containing` | `findByUsernameContaining` | `. . . where e.username like %?1%` |
| `IgnoreCase` | `findByUsernameIgnoreCase` | `. . . where UPPER(e.username) = UPPER(?1)` |
