**Install**
`brew install goose`

*-dir* คือ path folder ที่เก็บไฟล์ migrations
-sql คือ นามสกุลไฟล์ที่ต้องการ

**Create** 
`goose -dir internal/db/schema/migrations create add_some_column sql`

**Up**
`goose -dir internal/db/schema/migrations postgres "postgresql://root:root@127.0.0.1:5432/go-backend-clean-architecture-db?sslmode=disable" up`