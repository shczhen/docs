---
title: Transaction Restraints
summary: Learn about transaction restraints in TiDB.
---

# トランザクションの制限 {#transaction-restraints}

このドキュメントでは、TiDBのトランザクション制限について簡単に紹介します。

## 分離レベル {#isolation-levels}

TiDBでサポートされている分離レベルは、 <strong>RC（読み取りコミット）</strong>と<strong>SI（スナップショット分離）</strong>です。ここで、 <strong>SI</strong>は基本的に<strong>RR（繰り返し読み取り）</strong>分離レベルと同等です。

![isolation level](/media/develop/transaction_isolation_level.png)

## スナップショットアイソレーションはファントムリードを回避できます {#snapshot-isolation-can-avoid-phantom-reads}

TiDBの`SI`の分離レベルは<strong>ファントム読み取り</strong>を回避できますが、ANSI /ISOSQL標準の`RR`は回避できません。

次の2つの例は、<strong>ファントムの読み取り</strong>が何であるかを示しています。

-   例1：<strong>トランザクションA</strong>は最初にクエリに従って`n`行を取得し、次に<strong>トランザクションB</strong>はこれらの`n`行以外の`m`行を変更するか、<strong>トランザクションA</strong>のクエリに一致する`m`行を追加します。<strong>トランザクションA</strong>がクエリを再度実行すると、条件に一致する行が`n+m`行あることがわかります。ファントムのようなものなので、<strong>ファントムリード</strong>と呼ばれます。

-   例2：<strong>管理者A</strong>は、データベース内のすべての学生の成績を特定のスコアからABCDEの成績に変更しますが、<strong>管理者B</strong>は、この時点で特定のスコアのレコードを挿入します。<strong>管理者A</strong>が変更を終了し、まだ変更されていないレコード（<strong>管理者B</strong>によって挿入されたレコード）がまだあることを検出したとき。それは<strong>幻の読み取り</strong>です。

## SIは書き込みスキューを回避できません {#si-cannot-avoid-write-skew}

TiDBのSI分離レベルでは、<strong>書き込みスキュー</strong>例外を回避できません。 `SELECT FOR UPDATE`の構文を使用して、<strong>書き込みスキュー</strong>の例外を回避できます。

<strong>書き込みスキュー</strong>例外は、2つの同時トランザクションが異なるが関連するレコードを読み取るときに発生し、各トランザクションは読み取ったデータを更新して、最終的にトランザクションをコミットします。複数のトランザクションで同時に変更できないこれらの関連レコード間に制約がある場合、最終結果は制約に違反します。

たとえば、病院の医師シフト管理プログラムを作成しているとします。病院では通常、同時に複数の医師が待機している必要がありますが、最小要件は、少なくとも1人の医師が待機していることです。医師は、シフト中に少なくとも1人の医師が待機している限り、シフトをドロップできます（たとえば、気分が悪い場合）。

現在、医師`Alice`と`Bob`が待機している状況があります。どちらも気分が悪いので、病気休暇を取ることにしました。彼らはたまたま同時にボタンをクリックします。次のプログラムでこのプロセスをシミュレートしてみましょう。

{{&lt;コピー可能&quot;&quot;&gt;}}

```java
package com.pingcap.txn.write.skew;

import com.zaxxer.hikari.HikariDataSource;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class EffectWriteSkew {
    public static void main(String[] args) throws SQLException, InterruptedException {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost:4000/test?useServerPrepStmts=true&cachePrepStmts=true");
        ds.setUsername("root");

        // prepare data
        Connection connection = ds.getConnection();
        createDoctorTable(connection);
        createDoctor(connection, 1, "Alice", true, 123);
        createDoctor(connection, 2, "Bob", true, 123);
        createDoctor(connection, 3, "Carol", false, 123);

        Semaphore txn1Pass = new Semaphore(0);
        CountDownLatch countDownLatch = new CountDownLatch(2);
        ExecutorService threadPool = Executors.newFixedThreadPool(2);

        threadPool.execute(() -> {
            askForLeave(ds, txn1Pass, 1, 1);
            countDownLatch.countDown();
        });

        threadPool.execute(() -> {
            askForLeave(ds, txn1Pass, 2, 2);
            countDownLatch.countDown();
        });

        countDownLatch.await();
    }

    public static void createDoctorTable(Connection connection) throws SQLException {
        connection.createStatement().executeUpdate("CREATE TABLE `doctors` (" +
                "    `id` int(11) NOT NULL," +
                "    `name` varchar(255) DEFAULT NULL," +
                "    `on_call` tinyint(1) DEFAULT NULL," +
                "    `shift_id` int(11) DEFAULT NULL," +
                "    PRIMARY KEY (`id`)," +
                "    KEY `idx_shift_id` (`shift_id`)" +
                "  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin");
    }

    public static void createDoctor(Connection connection, Integer id, String name, Boolean onCall, Integer shiftID) throws SQLException {
        PreparedStatement insert = connection.prepareStatement(
                "INSERT INTO `doctors` (`id`, `name`, `on_call`, `shift_id`) VALUES (?, ?, ?, ?)");
        insert.setInt(1, id);
        insert.setString(2, name);
        insert.setBoolean(3, onCall);
        insert.setInt(4, shiftID);
        insert.executeUpdate();
    }

    public static void askForLeave(HikariDataSource ds, Semaphore txn1Pass, Integer txnID, Integer doctorID) {
        try(Connection connection = ds.getConnection()) {
            try {
                connection.setAutoCommit(false);

                String comment = txnID == 2 ? "    " : "" + "/* txn #{txn_id} */ ";
                connection.createStatement().executeUpdate(comment + "BEGIN");

                // Txn 1 should be waiting for txn 2 done
                if (txnID == 1) {
                    txn1Pass.acquire();
                }

                PreparedStatement currentOnCallQuery = connection.prepareStatement(comment +
                        "SELECT COUNT(*) AS `count` FROM `doctors` WHERE `on_call` = ? AND `shift_id` = ?");
                currentOnCallQuery.setBoolean(1, true);
                currentOnCallQuery.setInt(2, 123);
                ResultSet res = currentOnCallQuery.executeQuery();

                if (!res.next()) {
                    throw new RuntimeException("error query");
                } else {
                    int count = res.getInt("count");
                    if (count >= 2) {
                        // If current on-call doctor has 2 or more, this doctor can leave
                        PreparedStatement insert = connection.prepareStatement( comment +
                                "UPDATE `doctors` SET `on_call` = ? WHERE `id` = ? AND `shift_id` = ?");
                        insert.setBoolean(1, false);
                        insert.setInt(2, doctorID);
                        insert.setInt(3, 123);
                        insert.executeUpdate();

                        connection.commit();
                    } else {
                        throw new RuntimeException("At least one doctor is on call");
                    }
                }

                // Txn 2 done, let txn 1 run again
                if (txnID == 2) {
                    txn1Pass.release();
                }
            } catch (Exception e) {
                // If got any error, you should roll back, data is priceless
                connection.rollback();
                e.printStackTrace();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

SQLログ：

{{< copyable "" >}}

```sql
/* txn 1 */ BEGIN
    /* txn 2 */ BEGIN
    /* txn 2 */ SELECT COUNT(*) as `count` FROM `doctors` WHERE `on_call` = 1 AND `shift_id` = 123
    /* txn 2 */ UPDATE `doctors` SET `on_call` = 0 WHERE `id` = 2 AND `shift_id` = 123
    /* txn 2 */ COMMIT
/* txn 1 */ SELECT COUNT(*) AS `count` FROM `doctors` WHERE `on_call` = 1 and `shift_id` = 123
/* txn 1 */ UPDATE `doctors` SET `on_call` = 0 WHERE `id` = 1 AND `shift_id` = 123
/* txn 1 */ COMMIT
```

実行結果：

{{< copyable "" >}}

```sql
mysql> SELECT * FROM doctors;
+----+-------+---------+----------+
| id | name  | on_call | shift_id |
+----+-------+---------+----------+
|  1 | Alice |       0 |      123 |
|  2 | Bob   |       0 |      123 |
|  3 | Carol |       0 |      123 |
+----+-------+---------+----------+
```

どちらのトランザクションでも、アプリケーションは最初に2人以上の医師が待機しているかどうかを確認します。もしそうなら、それは一人の医者が安全に休暇を取ることができると仮定します。データベースはスナップショットアイソレーションを使用するため、両方のチェックで`2`が返され、両方のトランザクションが次のステージに進みます。 `Alice`は彼女の記録を非番に更新し、 `Bob`も同様に更新します。両方のトランザクションが正常にコミットされます。現在、少なくとも1人の医師が待機している必要があるという要件に違反する当直医はいない。次の図（<em><strong>データ集約型アプリケーションの設計</strong></em>から引用）は、実際に何が起こるかを示しています。

![Write Skew](/media/develop/write-skew.png)

次に、書き込みスキューの問題を回避するために、サンプルプログラムを`SELECT FOR UPDATE`を使用するように変更しましょう。

{{&lt;コピー可能&quot;&quot;&gt;}}

```java
package com.pingcap.txn.write.skew;

import com.zaxxer.hikari.HikariDataSource;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class EffectWriteSkew {
    public static void main(String[] args) throws SQLException, InterruptedException {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost:4000/test?useServerPrepStmts=true&cachePrepStmts=true");
        ds.setUsername("root");

        // prepare data
        Connection connection = ds.getConnection();
        createDoctorTable(connection);
        createDoctor(connection, 1, "Alice", true, 123);
        createDoctor(connection, 2, "Bob", true, 123);
        createDoctor(connection, 3, "Carol", false, 123);

        Semaphore txn1Pass = new Semaphore(0);
        CountDownLatch countDownLatch = new CountDownLatch(2);
        ExecutorService threadPool = Executors.newFixedThreadPool(2);

        threadPool.execute(() -> {
            askForLeave(ds, txn1Pass, 1, 1);
            countDownLatch.countDown();
        });

        threadPool.execute(() -> {
            askForLeave(ds, txn1Pass, 2, 2);
            countDownLatch.countDown();
        });

        countDownLatch.await();
    }

    public static void createDoctorTable(Connection connection) throws SQLException {
        connection.createStatement().executeUpdate("CREATE TABLE `doctors` (" +
                "    `id` int(11) NOT NULL," +
                "    `name` varchar(255) DEFAULT NULL," +
                "    `on_call` tinyint(1) DEFAULT NULL," +
                "    `shift_id` int(11) DEFAULT NULL," +
                "    PRIMARY KEY (`id`)," +
                "    KEY `idx_shift_id` (`shift_id`)" +
                "  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin");
    }

    public static void createDoctor(Connection connection, Integer id, String name, Boolean onCall, Integer shiftID) throws SQLException {
        PreparedStatement insert = connection.prepareStatement(
                "INSERT INTO `doctors` (`id`, `name`, `on_call`, `shift_id`) VALUES (?, ?, ?, ?)");
        insert.setInt(1, id);
        insert.setString(2, name);
        insert.setBoolean(3, onCall);
        insert.setInt(4, shiftID);
        insert.executeUpdate();
    }

    public static void askForLeave(HikariDataSource ds, Semaphore txn1Pass, Integer txnID, Integer doctorID) {
        try(Connection connection = ds.getConnection()) {
            try {
                connection.setAutoCommit(false);

                String comment = txnID == 2 ? "    " : "" + "/* txn #{txn_id} */ ";
                connection.createStatement().executeUpdate(comment + "BEGIN");

                // Txn 1 should be waiting for txn 2 done
                if (txnID == 1) {
                    txn1Pass.acquire();
                }

                PreparedStatement currentOnCallQuery = connection.prepareStatement(comment +
                        "SELECT COUNT(*) AS `count` FROM `doctors` WHERE `on_call` = ? AND `shift_id` = ? FOR UPDATE");
                currentOnCallQuery.setBoolean(1, true);
                currentOnCallQuery.setInt(2, 123);
                ResultSet res = currentOnCallQuery.executeQuery();

                if (!res.next()) {
                    throw new RuntimeException("error query");
                } else {
                    int count = res.getInt("count");
                    if (count >= 2) {
                        // If current on-call doctor has 2 or more, this doctor can leave
                        PreparedStatement insert = connection.prepareStatement( comment +
                                "UPDATE `doctors` SET `on_call` = ? WHERE `id` = ? AND `shift_id` = ?");
                        insert.setBoolean(1, false);
                        insert.setInt(2, doctorID);
                        insert.setInt(3, 123);
                        insert.executeUpdate();

                        connection.commit();
                    } else {
                        throw new RuntimeException("At least one doctor is on call");
                    }
                }

                // Txn 2 done, let txn 1 run again
                if (txnID == 2) {
                    txn1Pass.release();
                }
            } catch (Exception e) {
                // If got any error, you should roll back, data is priceless
                connection.rollback();
                e.printStackTrace();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

SQLログ：

{{< copyable "" >}}

```sql
/* txn 1 */ BEGIN
    /* txn 2 */ BEGIN
    /* txn 2 */ SELECT COUNT(*) AS `count` FROM `doctors` WHERE on_call = 1 AND `shift_id` = 123 FOR UPDATE
    /* txn 2 */ UPDATE `doctors` SET on_call = 0 WHERE `id` = 2 AND `shift_id` = 123
    /* txn 2 */ COMMIT
/* txn 1 */ SELECT COUNT(*) AS `count` FROM `doctors` WHERE `on_call` = 1 FOR UPDATE
At least one doctor is on call
/* txn 1 */ ROLLBACK
```

実行結果：

{{< copyable "" >}}

```sql
mysql> SELECT * FROM doctors;
+----+-------+---------+----------+
| id | name  | on_call | shift_id |
+----+-------+---------+----------+
|  1 | Alice |       1 |      123 |
|  2 | Bob   |       0 |      123 |
|  3 | Carol |       0 |      123 |
+----+-------+---------+----------+
```

## <code>savepoint</code>とネストされたトランザクションはサポートされていません {#code-savepoint-code-and-nested-transactions-are-not-supported}

TiDBは`savepoint`メカニズムをサポートしてい<em><strong>ない</strong></em>ため、 `PROPAGATION_NESTED`伝播動作をサポートしていません。アプリケーションが`PROPAGATION_NESTED`伝播動作を使用する<strong>JavaSpring</strong>フレームワークに基づいている場合は、ネストされたトランザクションのロジックを削除するために、アプリケーション側でそれを適応させる必要があります。

<strong>Spring</strong>でサポートされている`PROPAGATION_NESTED`の伝播動作は、ネストされたトランザクションをトリガーします。これは、現在のトランザクションとは独立して開始される子トランザクションです。ネストされたトランザクションの開始時に`savepoint`が記録されます。ネストされたトランザクションが失敗した場合、トランザクションは`savepoint`状態にロールバックします。ネストされたトランザクションは外部トランザクションの一部であり、外部トランザクションと一緒にコミットされます。

次の例は、 `savepoint`のメカニズムを示しています。

{{< copyable "" >}}

```sql
mysql> BEGIN;
mysql> INSERT INTO T2 VALUES(100);
mysql> SAVEPOINT svp1;
mysql> INSERT INTO T2 VALUES(200);
mysql> ROLLBACK TO SAVEPOINT svp1;
mysql> RELEASE SAVEPOINT svp1;
mysql> COMMIT;
mysql> SELECT * FROM T2;
+------+
|  ID   |
+------+
|  100 |
+------+
```

## 大規模な取引制限 {#large-transaction-restrictions}

基本的な原則は、トランザクションのサイズを制限することです。 KVレベルでは、TiDBには単一トランザクションのサイズに制限があります。 SQLレベルでは、1行のデータが1つのKVエントリにマップされ、インデックスを追加するたびに1つのKVエントリが追加されます。 SQLレベルでの制限は次のとおりです。

-   単一行の最大レコードサイズは`120 MB`です。 TiDBv5.0以降のバージョンでは`performance.txn-entry-size-limit`で構成できます。以前のバージョンの値は`6 MB`です。
-   サポートされる単一トランザクションの最大サイズは`10 GB`です。 TiDBv4.0以降のバージョンでは`performance.txn-total-size-limit`で構成できます。以前のバージョンの値は`100 MB`です。

サイズ制限と行制限の両方について、トランザクション実行中のトランザクションのエンコードと追加キーのオーバーヘッドも考慮する必要があることに注意してください。最適なパフォーマンスを実現するには、100〜500行ごとに1つのトランザクションを書き込むことをお勧めします。

## 自動コミットされた<code>SELECT FOR UPDATE</code>ステートメントはロックを待機しません {#auto-committed-code-select-for-update-code-statements-do-not-wait-for-locks}

現在、自動コミットされた`SELECT FOR UPDATE`ステートメントにロックは追加されていません。その効果を次の図に示します。

![The situation in TiDB](/media/develop/autocommit_selectforupdate_nowaitlock.png)

これは、MySQLとの既知の非互換性の問題です。この問題は、明示的な`BEGIN;COMMIT;`ステートメントを使用して解決できます。
