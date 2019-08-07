``` java
public SQLiteOpenHelper(Context context, String name, CursorFactory factory, int version) {
        this(context, name, factory, version, null);
    }

    public SQLiteDatabase getWritableDatabase() {
        synchronized (this) {
            return getDatabaseLocked(true);
        }
    }

  private SQLiteDatabase getDatabaseLocked(boolean writable) {
      .......
      db.beginTransaction();
      try {
              if (version == 0) {
                   onCreate(db);
              } else {
                   if (version > mNewVersion) {
                         onDowngrade(db, version, mNewVersion);
                   } else {
                         onUpgrade(db, version, mNewVersion);
                   }
              }
               db.setVersion(mNewVersion);
                db.setTransactionSuccessful();
              } finally {
                 db.endTransaction();
              }
  }

```

在SQLiteOpenHelper的构造函数中，包含了一个version的参数。这个参数即是数据库的版本。 所以，我们可以通过修改version来实现数据库的升级。 当version大于原数据库版本时，onUpgrade()会被触发，可以在该方法中编写数据库升级逻辑。具体的数据库升级逻辑示例可参考这里.


常用的SQL增删改查：
- 增：INSERT INTO table_name (列1, 列2,…) VALUES (值1, 值2,….)
- 删： DELETE FROM 表名称 WHERE 列名称 = 值
- 改：UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
- 查：SELECT 列名称（通配是*符号） FROM 表名称
ps:操作数据表是:ALTER TABLE。该语句用于在已有的表中添加、修改或删除列。
- ALTER TABLE table_name ADD column_name datatype
- ALTER TABLE table_name DROP COLUMN column_name
- ALTER TABLE table_name_old RENAME TO table_name_new