查看 ContentProvider 的源代码，我们可以看到在 Provider 基类中已经支持了数据库的批量操作

```java
// Override this to handle requests to perform a batch of operations, or the default implementation will iterate over the operations and call

public @NonNull ContentProviderResult[] applyBatch(
            @NonNull ArrayList<ContentProviderOperation> operations)
                    throws OperationApplicationException {
        final int numOperations = operations.size();
        final ContentProviderResult[] results = new ContentProviderResult[numOperations];
        for (int i = 0; i < numOperations; i++) {
            results[i] = operations.get(i).apply(this, results, i);
        }
        return results;
    }
```

我们可以通过调用 provider 的方法批量对数据库做数据修改，通过这个函数的注释可以看到，基类的默认 实现只是会把所有的 operation 拿出来逐个执行，对数据库的写入性能并没有太大的提升。

数据库的事务:数据库事务是由一组数据库操作序列组成，事务作为一个整体被执行。

事务的原子性:包含在其中的对数据库的操作序列最终要么全部执行，要么全部不执行。当全部执行时， 事务对数据库的修改将生效;当全部不执行时，数据库维持原有的状态，不会被修改。

我们可以通过覆写 ContentProvider 的 applyBatch(Stringauthority, ArrayList<ContentProviderOperation> operations)方法来实现对事务处理的支持


```java

public @NonNull ContentProviderResult[] applyBatch(
            @NonNull ArrayList<ContentProviderOperation> operations)
                    throws OperationApplicationException {
        SQLiteDatabase db = mDbHelper.getWritableDatabase();
        db.beginTransaction();
        try {
            ContentProviderResult[] results = super.applyBatch(operations); 
            db.setTransactionSuccessful();
            return results;
            } finally { 
                db.endTransaction();
            }
    }
```

在同一个事务中执行批量对数据库的修改，可以大大提高写入数据库的效率。