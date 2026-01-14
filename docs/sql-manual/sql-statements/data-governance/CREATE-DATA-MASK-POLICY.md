---
{
    "title": "CREATE DATA MASK POLICY",
    "language": "en",
    "description": "Explain can view the rewritten execution plan."
}
---

## Description

Explain can view the rewritten execution plan. 

## Syntax

```sql
CREATE DATA MASK POLICY [ IF NOT EXISTS ] <policy_name> 
ON <col_name> 
TO { <user_name> | ROLE <role_name> } 
USING <mask_type> [LEVEL <priority>];
```
## Required Parameters

**<policy_name>**

> column data mask policy name

**<col_name>**

> column name

## Optional Parameters

**<user_name>**

> User name, cannot be created for root and admin users

**<role_name>**

> Role name

**<mask_type>**

> Data mask type. see MASK_TYPE list

## Access Control Requirements

The user executing this SQL command must have at least the following privileges:

| Privilege                | Object | Notes |
| ------------------------ | ------ | ----- |
| ADMIN_PRIV or GRANT_PRIV | Global |       |

## MASK_TYPE

| 名称                        | 含义                            | 表达式                                                                                                |
|:--------------------------|:------------------------------|:---------------------------------------------------------------------------------------------------|
| MASK_REDACT | 写字母用 x 代替，大写字母用 X 代替，数字用 0 代替 | regexp_replace(regexp_replace(regexp_replace({col},'([A-Z])', 'X'),'([a-z])','x'),'([0-9])','0')   |
| MASK_SHOW_LAST_4 | 只显示最后4个字符，其他用 X 代替            | LPAD(RIGHT({col}, 4), CHAR_LENGTH({col}), 'X')                                                     |
| MASK_SHOW_FIRST_4 | 只显示前4个字符，其他用 X 代替             | RPAD(LEFT({col}, 4), CHAR_LENGTH({col}), 'X')                                                      |
| MASK_HASH | 使用 sha256 对值进行 hash           |    hex(sha2({col}, 256))           |
| MASK_NULL | 使用 NULL 对值进行覆盖                |    NULL           |
| MASK_DATE_SHOW_YEAR | 对日期类型，只显示年份                   |    date_trunc({col}, 'year')           |
| MASK_DEFAULT | 显示字段类型的默认值                    |               |
| MASK_NONE | 保持原样                          |               |


## Examples

1. Create a set of data mask policies

  ```sql
    CREATE DATA MASK POLICY test_policy_1 ON internal.test.t1.c1
    TO jack USING MASK_HASH;
    
    CREATE DATA MASK POLICY test_policy_2 ON internal.test.t1.c2
    TO Role r1 USING MASK_NULL;
    
    CREATE DATA MASK POLICY test_policy_3 ON internal.test.t1.c1
    TO jack USING MASK_NONE LEVEL 1;
  ```
