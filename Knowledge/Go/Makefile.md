**Example command pass args**
```Makefile
.PHONY: migration-create
migration-create:
	@echo "Creating migration"
	@goose -dir internal/db/schema/migrations create $(name) sql
```
**Usage**
```bash
make migration-create name=initial
```
**Result**
```bash
Creating migration
2024/03/19 11:15:06 Created new file: internal/db/schema/migrations/20240319041506_initial.sql
```
**Include** คือการโหลดตัวแปรจากไฟล์
```makefile
include .env
```