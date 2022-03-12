# PostgreSQL: Schema

### Schema & Search path

* Schema is a subset of a table.
* The default schema is **public**.
*   The two following commands are the same:

    ```sql
    SELECT * FROM users;
    ```

    ```sql
    SELECT * FROM public.users;
    ```
* Postgres will search the table following the order of **search\_path**
*   We can set search\_path by this command:

    ```sql
    SET search_path = 'schema_name';
    ```

    * SET changes the run-time configuration, only affects by the current session.
    * [PostgreSQL Team, SET](https://www.postgresql.org/docs/9.1/sql-set.html)
* To provide the security function we should use **schema**, but I don't what security that Postgres team mean.

### Search Path and connection pool

> Q: connection ใหม่จะถูกสร้างตอนไหน
>
> A: ต้องใล่ code ใน golang/sql&#x20;

> Q: 1 session ต่อ 1 connection&#x20;
>
> A: ใช่

> Note: SET search\_path มีผลแค่ session ปัจจุบัน&#x20;

> Q: ทำไงให้ ทุกๆ connection ใน connection pool มี search\_path มีค่าเดียวกัน&#x20;
>
> A: Set ในระดับ user: เพิ่ม query command ใน code&#x20;
>
> A: Set ตรง connection string: ให้ทุกๆ connection ใช้ config เดียวกัน ชัดเจนกว่า และไม่ specific กับ user แต่ ขึ้นกับ code, trust ในตัว code

> Note: Exec or Query function will call Conn() for retrieving a connection. https://go.dev/doc/database/manage-connections&#x20;

> Note: Conn() return connection in cache or create a new one.&#x20;

> Note: How to test https://github.com/golang/go/blob/3b5eec937018be98549dea7067964018f0e5824c/src/database/sql/sql.go#L1289&#x20;
>
> จาก source code บรรทัดนี้ ลอง open connection `db.Conn(context.Background())` เยอะๆ แล้วลอง query เพื่อดูว่าแต่ละ connection มี search path เป็นอย่างไร

* Further reading:
  * [Crunchy data, Demystifying Schemas & search\_path through Examples](https://blog.crunchydata.com/blog/demystifying-schemas-search\_path-through-examples)
