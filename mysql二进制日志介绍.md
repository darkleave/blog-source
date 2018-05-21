---
title: MySql binary log 二进制日志介绍
date: 2018-05-17 11:07:42
tags: [MySql]
---

[二进制日志格式](#5.4.4.1)
[设置二进制日志格式](#5.4.4.2)
[混合二进制日志格式](#5.4.4.3)
[针对mysql数据库表变化的日志格式](#5.4.4.4)

<!--more-->

二进制日志包含了描述数据库变化例如表创建操作或者表数据变化的“事件”。它同样包含了一些潜在性可能造成改变的statements声明事件(比如,,一个可能没有相应匹配的DELETE语句),除非使用了`row-based logging`.二进制日志同样包含了一些信息关于每个声明持有这些已更新数据的时间.MySql 的二进制日志存在两个重要的目的:

* 为了复制，master 服务器上的二进制日志提供了一系列数据改变的记录到slave 从服务器.master 服务器发送包含这些事件的二进制日志给它的从服务器，通过执行这些事件，从服务器可以完成与master服务器同样的数据变更.详细查看[Section 17.2, “Replication Implementation”](https://dev.mysql.com/doc/refman/5.5/en/replication-implementation.html).

* 某些数据恢复操作需要使用二进制日志.在一个备份呗还原之后,二进制日志当中的事件在备份被重新执行之后将被记录.这些事件将随着数据库更新到备份完成的这个时间节点.详细查看[Section 7.5, “Point-in-Time (Incremental) Recovery Using the Binary Log”](https://dev.mysql.com/doc/refman/5.5/en/point-in-time-recovery.html).

二进制日志并不应用于像select 或者 show 这些不造成数据变更的声明.如果想要记录所有声明(比如，标记一个问题查询),使用`general query log`.详细查看[Section 5.4.3, “The General Query Log”](https://dev.mysql.com/doc/refman/5.5/en/query-log.html).

运行的服务器如果开启了二进制日志功能会使性能稍微下降.但是，开启二进制日志来达到复制(从库)或者还原等操作，实际的好处是要远远超出的》

二进制日志需要收到保护(加密),因为日志记录的声明可能会包含密码.[Section 6.1.2.3, “Passwords and Logging”](https://dev.mysql.com/doc/refman/5.5/en/password-logging.html).

下述的讨论描述了一些服务器选项和变量，用于设置二进制日志的一些操作和行为.完整的列表查看[Section 17.1.3.4, “Binary Log Options and Variables”](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html).

想要开启二进制日志,在启动服务器时需要添加**[--log-bin[=base_name]](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#option_mysqld_log-bin)**选项.如果没有提供**base_name**选项,那么默认的名称的值将是在`-bin`之后的`pid-file`选项(默认将是主机名称).如果提供了**base name**,服务器将会在这个数据目录写入文件,除非**base name**前面有一个描述不同路径的绝对路径名.推荐明确地定义base name,而不是使用默认的主机名.详情查看[ Section B.5.7, “Known Issues in MySQL”](https://dev.mysql.com/doc/refman/5.5/en/bugs.html).

如果你在日志名称提供一个扩展，(比如,[--log-bin=base_name.extension](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#option_mysqld_log-bin)),这个扩展就会被默默地无视并且移除.

[mysqld](https://dev.mysql.com/doc/refman/5.5/en/mysqld.html)添加了一个数字扩展到二进制日志 base name 来生成二进制文件名称.每当服务器创建一个log file 这个数字都会增加, 因此会创建已排序的连续文件. 服务器会在启动或者刷新时按照规则创建新的文件.服务器同样会在当前日志大小达到设置的最大值时([max_binlog_size](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#sysvar_max_binlog_size))自动地创建一个二进制日志文件. 如果你使用大型事务，同时因为事务是一次性写入文件，不会发生分隔文件的情况，因此一个二进制文件超越最大限制的情况仍然是存在的.

为了持续跟踪哪个二进制文件已经被使用,[mysqld](https://dev.mysql.com/doc/refman/5.5/en/mysqld.html)创建了一个二进制索引日志文件,包含了所有已使用的二进制日志文件名称. 默认情况下，这个索引文件拥有与普通二进制日志文件同样的base name,只是会带有一个**'.index'**的索引名称. 你可以通过[--log-bin-index[=file_name]](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#option_mysqld_log-bin-index)命令来改变索引文件的名称. 你不能在`mysqld`运行的时候手动编辑这个文件,因为那样做会让`mysqld`感到困惑啊喵！.

`binary log file`二进制日志文件通常表现为独特的以数字顺序编号并且包含数据库事件的文件.`binary log` 二进制日志一般包含数字编号的二进制日志文件集合 及其索引文件.

一个拥有[SUPER](https://dev.mysql.com/doc/refman/5.5/en/privileges-provided.html#priv_super)权限的客户端能够通过命令`SET sql_log_bin=0`关闭它自身声明的二进制日志输出.详情查看[ Section 5.1.7, “Server System Variables”](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html).

记录在二进制日志当中的事件格式依赖于二进制记录格式.mysql支持三种日志格式,`row-based logging`,`statement-based logging` 和`mixed-base logging`. 使用的二进制记录格式取决于mysql的版本.更多日志记录格式的详情，查看[Section 5.4.4.1, “Binary Logging Formats”](https://dev.mysql.com/doc/refman/5.5/en/binary-log-formats.html). 二进制日志格式的详细信息查看[MySQL Internals: The Binary Log](https://dev.mysql.com/doc/internals/en/binary-log.html).

服务器视 [--binlog-do-db](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#option_mysqld_binlog-do-db)命令和[--binlog-ignore-db](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#option_mysqld_binlog-ignore-db)命令等同于[--replicate-do-db](https://dev.mysql.com/doc/refman/5.5/en/replication-options-slave.html#option_mysqld_replicate-do-db)和[--replicate-ignore-db](https://dev.mysql.com/doc/refman/5.5/en/replication-options-slave.html#option_mysqld_replicate-ignore-db).具体查看[Section 17.2.3.1, “Evaluation of Database-Level Replication and Binary Logging Options”](https://dev.mysql.com/doc/refman/5.5/en/replication-rules-db-options.html).

如果你是准备从一个NDB集群复制到一个单独的MySQL服务器,你必须认识到[NDB](https://dev.mysql.com/doc/refman/5.5/en/mysql-cluster.html)存储使用一些默认的二进制记录选项值(包括定义`NDB`的选项,比如[--ndb-log-update-as-write](https://dev.mysql.com/doc/refman/5.5/en/mysql-cluster-replication-conflict-resolution.html#option_mysqld_ndb-log-update-as-write),区别于使用其它存储引擎. 如果不做出一些修正,这些不同的地方将引起master 和 slave 主从之间二进制日志的分歧. 更多查看[Replication from NDB to other storage engines](https://dev.mysql.com/doc/refman/5.5/en/mysql-cluster-replication-issues.html#mysql-cluster-replication-ndb-to-non-ndb) . 尤其是当你使用的是非事务性的存储引擎，比如在从库使用[MyISAM](https://dev.mysql.com/doc/refman/5.5/en/myisam-storage-engine.html),详细查看[Replication from NDB to a nontransactional storage engine](https://dev.mysql.com/doc/refman/5.5/en/mysql-cluster-replication-issues.html#mysql-cluster-replication-ndb-to-nontransactional).

一个用于复制的从服务器，默认不写入自己的二进制日志文件,任何数据的改变都是从主服务器master接收.为了记录这些改变,使用[--log-slave-updates](https://dev.mysql.com/doc/refman/5.5/en/replication-options-slave.html#option_mysqld_log-slave-updates)选项及 [--log-bin](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#option_mysqld_log-bin)选项(详情查看[ Section 17.1.3.3, “Replication Slave Options and Variables”](https://dev.mysql.com/doc/refman/5.5/en/replication-options-slave.html). 当一个slave 同样是在复制链上的另一个slave的master，那么这种情况正好适用.

你能通过[RESET MASTER](https://dev.mysql.com/doc/refman/5.5/en/reset-master.html)命令删除所有二进制日志文件,或者使用[PURGE BINARY LOGS](https://dev.mysql.com/doc/refman/5.5/en/purge-binary-logs.html)命令删除其中一部分.详情查看[Section 13.7.6.6, “RESET Syntax”](https://dev.mysql.com/doc/refman/5.5/en/reset.html), 和 [Section 13.4.1.1, “PURGE BINARY LOGS Syntax”](https://dev.mysql.com/doc/refman/5.5/en/purge-binary-logs.html).

如果你正在使用replication复制功能，那么你不应当在master上删除任何旧的二进制日志文件,除非你完全确认没有任何的从服务区需要去使用它们.比如，你的从服务器从来没有落后主服务器三天的日志差距，那么你可以在master上执行[mysqladmin flush-logs](https://dev.mysql.com/doc/refman/5.5/en/mysqladmin.html)命令，然后移除三天之前的所有旧日志.你可以手动移除文件,但是更好的选择是使用[PURGE BINARY LOGS](https://dev.mysql.com/doc/refman/5.5/en/purge-binary-logs.html)命令,它同时会为你安全地更新二进制日志索引文件.(同时还能添加一个日期参数). 详情查看[Section 13.4.1.1, “PURGE BINARY LOGS Syntax”](https://dev.mysql.com/doc/refman/5.5/en/purge-binary-logs.html).

你可以通过[mysqlbinlog](https://dev.mysql.com/doc/refman/5.5/en/mysqlbinlog.html)工具来展示二进制日志文件的内容.当你想要通过日志重新运行声明来进行还原的操作这就很有用了.比如，你可以从二进制日志更新MySQL 服务器,如下:

	shell> mysqlbinlog log_file | mysql -h server_name

[mysqlbinlog](https://dev.mysql.com/doc/refman/5.5/en/mysqlbinlog.html) 同样能用来展示复制从服务器的中继日志文件内容，因为它们是相同格式的二进制日志文件.更多`mysqlbinlog`工具及其如何使用的相关信息,查看[Section 4.6.7, “mysqlbinlog — Utility for Processing Binary Log Files”](https://dev.mysql.com/doc/refman/5.5/en/mysqlbinlog.html).更多关于二进制日志和还原操作的相关信息,查看[Section 7.5, “Point-in-Time (Incremental) Recovery Using the Binary Log”](https://dev.mysql.com/doc/refman/5.5/en/point-in-time-recovery.html).

二进制日志记录动作会在声明或者事务完成之后，但在任何锁释放或者任何提交完成之前 完成.这确保了日志是按照commit的顺序记录的.

非事务性表的更新事件会在执行之后立刻存储在二进制日志当中.

在一个未完成提交的事务当中,所有对事务性表的更新([UPDATE](https://dev.mysql.com/doc/refman/5.5/en/update.html),[DELETE](https://dev.mysql.com/doc/refman/5.5/en/delete.html)或者[INSERT](https://dev.mysql.com/doc/refman/5.5/en/insert.html)，比如`InnoDB`,都会被缓存,知道一个[COMMIT]（https://dev.mysql.com/doc/refman/5.5/en/commit.html）被服务器接收到. 在这个时候，**mysqld**会在[COMMIT](https://dev.mysql.com/doc/refman/5.5/en/commit.html)执行之前，将整个事务写进到二进制日志当中.

对非事务性表的变更无法回滚.如果一个包含非事务性表的变更的事务被回滚，整个事务会在[ROLLBACK](https://dev.mysql.com/doc/refman/5.5/en/commit.html)声明的最后被记录进日志来确保所有对表的变更都能被复制.

当一个控制事务的线程启动,它分配一个[binlog_cache_size](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#sysvar_binlog_cache_size)设置的buffer 给buffer statements声明.如果一个声明大于这个buffer,这个线程就会打开一个临时文件来存储这个线程. 临时文件将会被线程结束时被删除.

[Binlog_cache_use](https://dev.mysql.com/doc/refman/5.5/en/server-status-variables.html#statvar_Binlog_cache_use) 状态变量展示了已经使用这个buffer(也可能同时使用了一个临时文件)来存储声明的事务的数量. [Binlog_cache_disk_use](https://dev.mysql.com/doc/refman/5.5/en/server-status-variables.html#statvar_Binlog_cache_disk_use) 状态变量展示了那些事务当中实际使用临时文件的数量. 这两个变量可以用来调整[binlog_cache_size](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#sysvar_binlog_cache_size)的大小以避免使用临时文件》

[max_binlog_cache_size](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#sysvar_max_binlog_cache_size) 系统变量(默认 4GB,同时也是最大值了)可以用来限制一个多声明事务使用的缓存总大小. 如果一个事务超过这个值很多字节,那么它会失败并且回滚,最小值是4096 个字节.

如果你正在使用二进制日志和`row based logging`格式,并发插入会被转换为普通插入比如:`CREATE ... SELECT` 或者 [INSERT ... SELECT]声明.  这是为了确保能够通过应用日志正确的进行备份的还原操作. 如果你是使用`statement-based logging` 格式，原始的声明会被写入到日志当中.

二进制日志格式有一些已知的限制可能会影响从备份还原的操作.详细查看[Section 17.4.1, “Replication Features and Issues”](https://dev.mysql.com/doc/refman/5.5/en/replication-features.html).

保存程序的二进制日志已经完成，详细查看[Section 20.7, “Binary Logging of Stored Programs”](https://dev.mysql.com/doc/refman/5.5/en/stored-programs-logging.html).

需要注意的是二进制日志格式在MySQL5.5版及其相近版本之间是有所差别的，原因是为了增强replication复制功能. 详细查看[ Section 17.4.2, “Replication Compatibility Between MySQL Versions”](https://dev.mysql.com/doc/refman/5.5/en/replication-compatibility.html).

当写入到`MyISAM` 表时，写入到二进制日志文件和二进制日志索引文件都是用的同一种方式. 详细查看[Section B.5.3.4, “How MySQL Handles a Full Disk”](https://dev.mysql.com/doc/refman/5.5/en/full-disk.html).

默认情况下,二进制日志是不会在每次写入都同步到磁盘的.所以如果操作系统或者主机(不仅仅是MySQL服务器)宕机的话，这部分二进制日志包含的声明改变等都会丢失. 为了防止出现这个问题，你可以设置二进制日志每**`N`**次写入到二进制日志就会触发同步到磁盘的操作.  通过[sync_binlog](https://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#sysvar_sync_binlog)系统变量来进行设置.  详细查看[ Section 5.1.7, “Server System Variables”](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html). `sync_binlog`设置为 1 是最安全的值,但同时也是最慢的方式. 而且即使设置为1，同样也可能存在因为宕机而引起的表内容和二进制日志内容之间的不同步. 比如，如果你是使用`InnoDB` 表和 MySQL服务器来完成一个[COMMIT](https://dev.mysql.com/doc/refman/5.5/en/commit.html) 声明,它会把整个事务写入到二进制日志然后提交这个事务到`InnoDB`. 如果服务器在这两个操作之间宕机的话,事务会在`InnoDB`重启的时候被回滚,但是它同样会存在于二进制日志当中. 为了解决这个问题, 你需要把 [--innodb_support_xa](https://dev.mysql.com/doc/refman/5.5/en/innodb-parameters.html#sysvar_innodb_support_xa)设置为1. 尽管这个选项在InnoDB当中更适应于支持XA事务, 但它同样能确保二进制日志和InnoDB数据文件被同步.

为了这个设置提供的更大程度的安全性,MySQL 服务器应该同样被配置为在每个事务都同步二进制日志和`InnoDB`日志到磁盘. `InnoDB` 日志默认情况下是会同步的, `sync_binlog=1` 用来设置同步二进制日志. 这个选项影响的是当宕机重启之后,在完成事务回滚的过程中,MySQL服务器扫描最新的二进制日志文件来手机事务**`xid`**值并且在二进制文件当中计算最新有效的位点. MySQL服务器接着就会告诉`InnoDB` 去完成所有成功写入到二进制日志的 prepared transactions , 并且阶段二进制日志到最新的有效位点. 这能确保二进制日志反馈正确的`InnoDB`表数据. 并且因此,slave 服务器也会通过master 来同步(已经被回滚的声明不会再被slave服务器收到).

如果MySQL 服务器发现宕机还原情况下二进制日志比它实际应有的大小要小的话，那么它至少缺少了了一个成功的`InnoDB`提交事务. 如果`sync_binlog=1`设置为1的话这种情况就不应当会发生,然后 文件系统当它们被请求的时候会完成一次实际的同步(有些可能不会),所以服务器会打印一个错误信息 **The binary log file_name is shorter than its expected size**. 在这种情况下,二进制日志不再正确并且replication应当从一个全新的master's data 快照重启.

下列系统变量会影响 replication slave,当转换二进制日志的时候:

* [sql_mode](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_sql_mode)(除了[NO_DIR_IN_CREATE](https://dev.mysql.com/doc/refman/5.5/en/sql-mode.html#sqlmode_no_dir_in_create) 模式是不支持replicated的;参考[ Section 17.4.1.38, “Replication and Variables”](https://dev.mysql.com/doc/refman/5.5/en/replication-features-variables.html))

* [foreign_key_checks](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_foreign_key_checks)

* [unique_checks](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_unique_checks)


* [character_set_client](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_character_set_client)

* [collation_connection](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_collation_connection)

* [collation_database](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_collation_database)

* [collation_server](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_collation_server)

* [sql_auto_is_null](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_sql_auto_is_null)

