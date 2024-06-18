## 创建账户

我们安装`MySQL`后，通常需要创建一个账户。下面是一个创建账户的示例：

!!! info "说明"
    - `username` 替换为要设置的账号
    - `host` 替换为允许登录的主机，如果设置为 '%' 则表示允许所有主机
    - `password` 替换为要设置的密码

```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

下一步，对该账户进行授权

!!! info "说明"
    - `database_name` 替换为你要授权的数据库名称，设置 '*' 表示所有数据库

```sql
GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost' IDENTIFIED BY 'password';
```

刷新权限

```sql
FLUSH PRIVILEGES;
```

## 低版本排名

`MySQL 5.7` 不支持 `DENSE_RANK()`、`RANK()` 和 `ROW_NUMBER()` 这些窗口函数。这些功能是在`MySQL 8.0`版本中首次引入的。对于`MySQL 5.7`及更低版本，如果你需要实现类似排名的功能，可以采用变量和自连接等方法来模拟，但这些方法通常较为复杂且效率相对较低。

以下是一个`MySQL 5.7`的排名示例：

=== "并列不占位排名:material-information-outline:{ title="有相同排名时，不占据下一个排名的位置，例如：1、2、2、3" }"

    ```sql
    SELECT
        CASE
            WHEN @qty = tm.qty THEN @rank
            WHEN @qty := tm.qty THEN @rank := @rank + 1 
        END AS qty_rank,
        tm.qty
    FROM
        (
        SELECT
            t.qty
        FROM
            table_name t
        ORDER BY t.qty DESC 
        ) tm,
    ( SELECT @rank := 0, @qty := NULL ) r
    ```

=== "并列占位排名:material-information-outline:{ title="有相同排名时，下一个排名顺延一位，例如：1、2、2、4" }"

    ```sql
    SELECT
        qty
    FROM
        (
        SELECT
            @rank := @rank + 1 AS rank_temp,
            @incrnum :=
                CASE
                    WHEN @qty = tm.qty THEN @incrnum 
                    WHEN @qty := tm.qty THEN @rank
                END AS qty_rank, 
                tm.qty
            FROM
                (
                SELECT
                    t.qty
                FROM table_name t
                ORDER BY t.qty DESC 
                ) tm,
            ( SELECT @rank := 0, @qty := NULL, @incrnum := 0) r 
        ) AS r 
    WHERE r.qty_rank<10
    ```

:arrow_up: [<font size="2">回到顶部</font>][top]

[top]: #