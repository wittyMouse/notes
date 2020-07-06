`DECR, DECRBY, DEL, EXISTS, EXPIRE, GET, GETSET, HDEL, HEXISTS, HGET, HGETALL, HINCRBY, HKEYS, HLEN, HMGET, HMSET, HSET, HVALS, INCR, INCRBY, KEYS, LINDEX, LLEN, LPOP, LPUSH, LRANGE, LREM, LSET, LTRIM, MGET, MSET, MSETNX, MULTI, PEXPIRE, RENAME, RENAMENX, RPOP, RPOPLPUSH, RPUSH, SADD, SCARD, SDIFF, SDIFFSTORE, SET, SETEX, SETNX, SINTER, SINTERSTORE, SISMEMBER, SMEMBERS, SMOVE, SORT, SPOP, SRANDMEMBER, SREM, SUNION, SUNIONSTORE, TTL, TYPE, ZADD, ZCARD, ZCOUNT, ZINCRBY, ZRANGE, ZRANGEBYSCORE, ZRANK, ZREM, ZREMRANGEBYSCORE, ZREVRANGE, ZSCORE`

Redis是被称为键值存储的数据库家族中的一员。

键值存储的本质是能够在键内部存储一些数据（称为值）的能力。仅当我们知道用于存储数据的确切密钥时，以后才能检索该数据。

Redis通常被称为数据结构服务器，因为它具有外部键值外壳，但是每个值都可以包含复杂的数据结构，例如字符串，列表，哈希或有序数据结构（称为排序集和概率）数据结构，例如超级日志。

作为第一个示例，我们可以使用命令SET将值“ fido”存储在键“ server：name”中：

`SET server:name "fido"`

Redis将永久存储我们的数据，因此我们稍后可以询问“密钥服务器：名称中存储的值是多少？” Redis会回复“ fido”：

`GET server:name` => "fido"

有一个命令可以测试给定密钥是否存在：

`EXISTS server:name` => 1

`EXISTS server:blabla` => 0

提示：您可以单击上面的命令以自动执行它们。箭头（=>）之后的文本显示了预期的输出。

Redis提供的其他基本操作包括：DEL删除给定的键和关联值， INCR原子地增加存储在给定键上的数字：

`SET connections 10`

`INCR connections` => 11

`INCR connections` => 12

`DEL connections`

`INCR connections` => 1

还可以将键中包含的数字增加特定数量：

`INCRBY connections 100` => 101

并且有类似的命令以减少密钥的值。

`DECR connections` => 100

`DECRBY connections 10` => 90

使用递增和递减命令操作Redis字符串时，您正在实现counter。计数器是Redis的非常流行的应用程序。

INCR有一些特别之处。如果我们可以自己编写一点代码，为什么还要提供这样的操作？毕竟，它很简单：

`x = GET count`

`x = x + 1`

`SET count x`

问题在于，只有在有单个客户端使用密钥的情况下，以这种方式进行增量操作才有效。查看两个客户端同时访问此密钥会发生什么：
1. 客户端A读取计数为10。
2. 客户端B读取的计数为10。
3. 客户A增加10，并将计数设置为11。
4. 客户端B增加10，并将计数设置为11。

我们希望该值为12，但实际上为11！这是因为以这种方式增加值不是原子操作。在Redis中调用 INCR命令可以防止这种情况的发生，因为它是原子操作。

由单个命令实现的所有Redis操作都是原子的，包括对更复杂的数据结构进行操作的操作。因此，当您使用修改某些值的Redis命令时，无需考虑并发访问。

可以告诉Redis密钥只能存在一定时间。这是通过与EXPIRE和TTL命令，并且由类似PEXPIRE和PTTL进行操作的使用时间以毫秒为单位的而不是秒的命令。

`SET resource:lock "Redis Demo"`

`EXPIRE resource:lock 120`

这将导致密钥resource：lock在120秒内删除。您可以使用TTL命令测试密钥存在多长时间。它返回直到将其删除的秒数。

`TTL resource:lock` => 113

// after 113s

`TTL resource:lock` => -2

键的TTL的-2表示该键不存在（不再存在）。一个-1的TTL的关键手段，它永远都不会过期的。请注意，如果您设置一个键，其TTL将被重置。

`SET resource:lock "Redis Demo 1"`

`EXPIRE resource:lock 120`

`TTL resource:lock` => 119

`SET resource:lock "Redis Demo 2"`

`TTL resource:lock` => -1

该SET命令实际上是能够为了接受进一步的参数可以直接设置生存时间（TTL）到一个键，这样你就可以改变一个键的值，并在单个原子操作，同时设置其TTL：

`SET resource:lock "Redis Demo 3" EX 5`

`TTL resource:lock` => 5

也可以取消密钥的生存时间，以删除过期并再次将其永久化。

`SET resource:lock "Redis Demo 3" EX 5`

`PERSIST resource:lock`

`TTL resource:lock` => -1

Redis还支持几种更复杂的数据结构。我们要看的第一个是列表。列表是一系列有序值。一些与列表交互的重要的命令是RPUSH，LPUSH，LLEN，LRANGE，LPOP和RPOP。只要还不存在其他类型的键，就可以立即将其用作列表。

这个概念通常适用于每个Redis数据结构：您不必先创建密钥，然后再添加密钥，但是您可以直接使用命令来添加新元素。作为副作用，如果密钥不存在，则会创建该密钥。同样，在执行某些命令后将变为空的键将自动从键空间中删除。

RPUSH将新元素放在列表的末尾。

`RPUSH friends "Alice"`

`RPUSH friends "Bob"`

LPUSH将新元素放在列表的开头。

`LPUSH friends "Sam"`

LRANGE给出列表的子集。它以要检索的第一个元素的索引为第一个参数，以要检索的最后一个元素的索引为第二个参数。第二个参数的值-1表示检索元素直到列表的末尾，-2表示包括最多倒数第二个，依此类推。

`LRANGE friends 0 -1` => 1) "Sam", 2) "Alice", 3) "Bob"

`LRANGE friends 0 1` => 1) "Sam", 2) "Alice"

`LRANGE friends 1 2` => 1) "Alice", 2) "Bob"

到目前为止，我们探索了使您可以向列表中添加元素的命令以及使您可以检查列表范围的LRANGE。Redis列表的基本功能是能够删除列表开头或结尾的元素，并同时将其返回给客户端。

LPOP从列表中删除第一个元素并返回它。

`LPOP friends` => "Sam"

RPOP从列表中删除最后一个元素并返回它。

`RPOP friends` => "3"

请注意，列表现在仅包含四个元素：

`LLEN friends` => 4

`LRANGE friends 0 -1` => 1) "Alice" 2) "Bob" 3) "1" 4) "2"

既RPUSH和LPUSH命令是可变参数，以便可以指定在相同的命令执行多个元件。

`RPUSH friends 1 2 3` => 6

提示：RPUSH和LPUSH在操作后返回列表的总长度。

您还可以使用LLEN获取列表的当前长度。

`LLEN friends` => 6

我们将要看的下一个数据结构是一个集合。集合与列表相似，不同之处在于它没有特定的顺序，每个元素只能出现一次。这两种数据结构都非常有用，因为虽然列表中的元素可以快速访问顶部或底部附近的元素，并且保留元素的顺序，但在集合中可以快速测试成员资格，即立即访问知道是否添加了给定的元素。此外，集合中的给定元素只能存在于单个副本中。

使用集合时，一些重要的命令是SADD，SREM，SISMEMBER，SMEMBERS和SUNION。

SADD将给定的成员添加到集合中，该命令也是可变的。

`SADD superpowers "flight"`

`SADD superpowers "x-ray vision" "reflexes"`

SREM从集合中删除给定的成员，返回1或0表示该成员是否确实存在。

`SREM superpowers "reflexes"` => 1

`SREM superpowers "making pizza"` => 0

SISMEMBER测试给定值是否在集合中。如果该值存在，则返回1，否则返回0。

`SISMEMBER superpowers "flight"` => 1

`SISMEMBER superpowers "reflexes"` => 0

SMEMBERS返回此集合的所有成员的列表。

`SMEMBERS superpowers` => 1) "flight", 2) "x-ray vision"

SUNION组合两个或更多集合，并返回所有元素的列表。

`SADD birdpowers "pecking"`

`SADD birdpowers "flight"`

`SUNION superpowers birdpowers` => 1) "pecking", 2) "x-ray vision", 3) "flight"

SADD的返回值与SREM的返回值一样重要。如果添加的元素已存在，则返回0，否则返回1：

`SADD superpowers "flight"` => 0

`SADD superpowers "invisibility"` => 1

集合还具有与LPOP和RPOP非常相似的命令 ，以便从集合中提取元素并将它们通过一次操作返回给客户端。但是，由于集合不是有序的数据结构，因此在这种情况下返回（和删除）的元素完全是随意的。

`SADD letters a b c d e f` => 6

`SPOP letters 2` => 1) "c" 2) "a"
    
密钥名称后面的SPOP参数是我们希望它返回并从集合中删除的元素数。

现在，该集合将仅包含其余元素：

`SMEMBERS letters` => 1) "b" 2) "d" 3) "e" 4) "f"

还有一个命令可以返回随机元素而不将其从集合中删除，这称为SRANDMEMBER。您可以自己尝试，其参数类似于SPOP，但是如果您指定负数而不是正数，它也可能返回重复元素。

集合是一种非常方便的数据类型，但是由于它们没有排序，因此对于许多问题来说效果不佳。这就是Redis 1.2引入排序集的原因。

排序集类似于常规集，但是现在每个值都有一个关联的分数。该分数用于对集合中的元素进行排序。

```
ZADD hackers 1940 "Alan Kay"
ZADD hackers 1906 "Grace Hopper"
ZADD hackers 1953 "Richard Stallman"
ZADD hackers 1965 "Yukihiro Matsumoto"
ZADD hackers 1916 "Claude Shannon"
ZADD hackers 1969 "Linus Torvalds"
ZADD hackers 1957 "Sophie Wilson"
ZADD hackers 1912 "Alan Turing"
```

在这些示例中，得分是出生年份，而值是著名黑客的名字。

`ZRANGE hackers 2 4` => 1) "Claude Shannon", 2) "Alan Kay", 3) "Richard Stallman"

简单的字符串，集合和排序集合已经完成了很多工作，但是Redis可以处理另一种数据类型：哈希。

哈希是字符串字段和字符串值之间的映射，因此它们是表示对象的理想数据类型（例如：具有多个字段（例如姓名，姓氏，年龄等）的用户）：

`HSET user:1000 name "John Smith"`

`HSET user:1000 email "john.smith@example.com"`

`HSET user:1000 password "s3cret"`

要获取保存的数据，请使用HGETALL：

`HGETALL user:1000`

您也可以一次设置多个字段：

`HMSET user:1001 name "Mary Jones" password "hidden" email "mjones@example.com"`

如果您只需要一个可能的单个字段值：

`HGET user:1001 name` => "Mary Jones"

哈希字段中的数值的处理方式与简单字符串中的数值完全相同，并且有一些操作以原子方式递增此值。

```
HSET user:1000 visits 10
HINCRBY user:1000 visits 1 => 11
HINCRBY user:1000 visits 10 => 21
HDEL user:1000 visits
HINCRBY user:1000 visits 1 => 1
```
