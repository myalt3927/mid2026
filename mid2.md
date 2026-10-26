```C
#include <stdio.h>
#include <sqlite3.h>

// SELECT로 찾은 데이터를 출력한다.
static int printRow(void *unused, int count, char **values, char **columns)
{
    (void)unused;
    for (int i = 0; i < count; i++)
        printf("%s: %s  ", columns[i], values[i] ? values[i] : "NULL");
    puts("");
    return 0;
}

int main(void)
{
    sqlite3 *db;
    if (sqlite3_open("students.db", &db) != SQLITE_OK) {
        fprintf(stderr, "%s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }

    // 테이블 생성 → 3명 저장 → 이름으로 검색
    int result = sqlite3_exec(db,
        "CREATE TABLE IF NOT EXISTS students(id INTEGER PRIMARY KEY, name TEXT, age INTEGER);"
        "INSERT OR IGNORE INTO students VALUES(1,'김철수',20),(2,'이영희',21),(3,'박민수',22);"
        "SELECT * FROM students WHERE name='김철수';",
        printRow, NULL, NULL);

    if (result != SQLITE_OK) fprintf(stderr, "%s\n", sqlite3_errmsg(db));
    sqlite3_close(db);
    return result != SQLITE_OK;
}
```
