### 系统设计

**如果要你设计一个高性能、高可用的系统，你会怎么设计**

系统设计--缓存设计--数据库设计--消息队列异步--熔断限流--监控报警

1.系统拆分：1. **微服务**拆分（按功能单一性），将一个系统拆分为多个子系统，用 RPC 来搞。然后每个系统连一个数据库，这样本来就一个库，现在多个数据库，不也可以扛高并发么。2. **分布式**架构+负载均衡分而治之，多服务器部署

2.缓存：大部分的高并发场景，都是读多写少，那你完全可以在数据库和缓存里都写一份，然后读的时候大量走缓存不就得了。所以你可以考虑考虑你的项目里，那些承载主要请求的读场景，怎么用缓存来抗高并发。

3.消息队列：Redis 来承载写那肯定不行，人家是缓存，数据随时就被 LRU 了，数据格式还无比简单，没有事务支持。

大量的写请求灌入 MQ 里，后边系统消费后慢慢写，控制在 MySQL 承载范围之内。所以你得考虑考虑你的项目里，那些承载复杂写业务逻辑的场景里，如何用 MQ 来异步写，提升并发性。（区分异步和解耦，解耦对应微服务架构，大量写任务存消息队列？）

4.分库分表：数据库层面还是免不了抗高并发的要求，好吧，那么就将一个数据库拆分为多个库，多个库来扛更高的并发；然后将一个表拆分为多个表，每个表的数据量保持少一点，提高 SQL 跑的性能。

5.读写分离：数据库可能也是读多写少，没必要所有请求都集中在一个库上吧，可以搞个主从架构，主库写入，从库读取下·

6.限流和熔断，一般秒杀活动会使用hystrix实现限流熔断

7.监控和报警： Grafana 

**是否考虑安全性，安全性设计有什么**

降级（保证核心服务可用），熔断，限流（防止服务器过载），监控

先熔断：一个服务不可用导致大量不可用

后降级：上游调用下游，下游故障，熔断，上游改为调用降级流程

**怎么理解微服务的，他的优缺点**

把应用程序按业务功能拆分成小型独立自治的服务，每个服务有自己的数据库和业务逻辑，能独立部署，扩展和管理，服务之间通过http或rpc等方式通信

优点：松耦合，便于维护扩展，各个业务团队可以独立开发，可以使用不同的技术栈，因此更灵活

缺点：编写更复杂，开销更大（通信储存等），跨库事务，数据一致性

**使用redis+lua+nginx限流**

编写了一个 Lua 脚本，负责与 Redis 进行交互，基于nginx的OpenResty 本身就提供了相关的包，导入模块后，连接redis就可以使用，配置限流目标（如ip）限流速率，桶容量，redis在这里面就起到了一个桶的作用，key是bucket:{服务名}:{IP}，val是令牌数，然后本身redis是原子操作，所以这样是安全的

另一种思路是用zset的形式，score存过期时间（为什么不直接用redis的过期时间，因为这种方式过期时间太频繁了，联想redis过期删除策略，而且时间控制精度也不够高）

为什么用redis：nginx可能有多个，构成集群，需要多服务间共享限流状态，计数器的状态只存在于当前 Nginx 工作进程的内存中。

redis故障恢复好，生态系统成熟

**四种限流算法**

固定窗口会出现流量突刺；滑动窗口实现较复杂，难以选则合适的滑动单位；漏桶会固定处理请求的速率，无法应对高峰期，容易造成请求延时或拒绝（桶中装请求，新来请求看能否入桶，能就放行）；令牌桶有桶容量可以蓄积令牌，但并不是无上限的，所以可以一定程度上应对突发，（当前时间减上一次生成的时间计算生成了多少令牌，再计算桶中有多少令牌，在计算是否大于需要消耗的令牌数，是的话扣除令牌）通过生成速率和消耗速率（限制的是一段时间内平均被调用次数），

ratelimiter源码：有预热和预支令牌机制，**通过限制后面请求的等待时间，来支持一定程度的突发请求(预消费)**通过一个变量记录下一次请求可以获取令牌的起始时间，如果当前请求5个令牌，桶里只有1个，不用等待生成，而是更新下一次请求的允许时间，然后放行）

还要结合阻塞等待机制，计算需要等待多长时间（下一次请求的允许时间减去当前时间（因为预消费）），并挂起等待

![image-20240406112817247](C:\Users\16776\AppData\Roaming\Typora\typora-user-images\image-20240406112817247.png)

第二个请求来的时候，需要等待请求1预消费的1s，然后自己预消费3s（因为要三个令牌），然后第三个请求需要先阻塞等待3s

### mysql

#### 其他sql题

**写一条 SQL 语句，在学生表、课程表(ID, TEACHER_ID, TEACHER_NAME, COURSE_ID, COURSE_NAME)、学生_课程表三个表中找出没选"XXX"老师课的学生**

找所有学生not in（根据课程id连接两个表，根据老师名称找到学生id）

SELECT DISTINCT s.*
FROM students s
WHERE s.student_id NOT IN (
    SELECT sc.student_id
    FROM student_courses sc
    JOIN courses c ON sc.course_id = c.course_id
    WHERE c.teacher_name = 'XXX'
);

**例：mysql主播表**： 用户id，房间id，开播时间，下播时间，房间开启状态（0、1），日期 第一问：求每个主播每一天的开播时长 第二问：求每个主播一天内每小时的开播时长

#### **牛客sql题**

[牛客网在线编程_SQL篇_非技术快速入门 (nowcoder.com)](https://www.nowcoder.com/exam/oj?page=1&tab=SQL篇&topicId=199)

常考：用了group by和order by，count函数，秒



15:gpa在3.5以上的山东大学或gpa在3.8以上的复旦大学,两个条件查询,union

16:查gpa最高:max(列) round(数,1)保留一位小数

17:查男性有多少人,gpa平均多少:avg(列)

18:分组计算:每个学校不同性别的用户分组,avg算的是分组后的avg

19:分组过滤:聚合函数(avg)结果作为筛选条件时，不能用where，而是用having语法 select avg() from t group by x having a or b

20

有一个表，里面有用户名和用户分数，怎么找第一名，找第二名，第一名有重复怎么找第二名

1. order by score desc limit 1
2. 第二名只有一个人 where score <(select max(score) from table) order by desc limit 1
3. 第二名有多个人：子查询找到第二高的分数，然后再查 select distinct score from table order by limit 1 offset 1  

```
select q.device_id,q.question_id,q.result
from user_profile u
**inner join** question_practice_detail q **on** u.device_id = q.device_id
where u.university = '浙江大学'
order by question_id
```

**补：**例left join 有一个北理工的人，没答题，但是用户表里有信息，如果用left join就会出现该信息，然后题目具体信息的部分为null

要注意基础表是哪个





多表查询

21 join

22 count/count（distinct），group，join（可以用where代替）

23 三张表，两次join，groupg给两个参数

24 三张表 可以join+where也可以 where拼接and条件

25 分别查看学校为山东大学或者性别为男性,结果不去重 union all (不用where or是因为 or自带去重和union不加all效果是一样的)

常用函数:

26:条件函数 if (x=n,a,b）表示如果x=n,则返回a，否则就是b了。

27:条件函数 ( case when 1 then 1 when 2 then 2 else 3 end 列名)

28:日期函数,如果日期格式是 date:2021-05-03,可以用year(date)取出年,month(date)取出月day(date) (也可用date like '2021-08%')

29:日期函数



#### 执行顺序

**sql执行顺序**  FROM - ON - JOIN - **WHERE** - GROUP BY（开始使用select中的别名，后面都可以使用）avg/sum - WITH - **HAVING** - **SELECT** - DISTINCT - ORDER BY - LIMIT

1. from(包括from中的子语句)
2. join 如果from后面是多张表，join关联，会首先对前两个表执行一个笛卡尔乘积，这时候就会生成第一个虚拟表T1（注意：这里会选择相对小的表作为基础表）；
3. on 对虚表T1进行on筛选，得到虚表T2，如果还有第三、第四个表将重复此过程。
4. where 对虚表T2进行where条件过滤，得到T3
5. group by 将T3中的唯一的值组合成为一组，得到虚拟表T4。如果应用了group by，那么后面的所有步骤都只能操作T4的列或者是执行聚合函数（count、sum、avg等）。
6. avg,sum.... 等聚合函数  聚合函数只是对分组的结果进行一些处理，拿到某些想要的聚合值，例如求和，统计数量等，并不生成虚拟表。
7. having 应用having筛选器，生成T5。HAVING子句主要和GROUP BY子句配合使用，having筛选器是第一个也是为唯一一个应用到已分组数据的筛选器。
8. select  执行select操作，选择指定的列，插入到虚拟表T6中。
9. distinct 对T6表中的记录进行去重，得到T7，实际使用时如果使用了group by，那么distinct将是多余的
10. order by  排序是很需要成本的，除非你必须要排序，否则最好不要指定order by
11. limit 取出指定行的记录，产生虚表T9，limit后面的参数可以是 一个limit m ，也可以是limit m n，表示从第m条到第n条数据。很多开发人员喜欢使用该语句来解决分页问题。对于小数据，使用LIMIT子句没有任何问题，当数据量非常大的时候，使用LIMIT m, n是非常低效的。因为LIMIT的机制是每次都是从头开始扫描，如果需要从第60万行开始，读取3条数据，就需要先扫描定位到60万行，然后再进行读取，而扫描的过程是一个非常低效的过程。所以，对于大数据处理时，是非常有必要在应用层建立一定的缓存机制

#### 子查询

```
select device_id, question_id, result
from question_practice_detail
where device_id in (
  ``select device_id from user_profile
  ``where university=``'浙江大学'
)
order by question_id
```

#### 连接查询

```

select university,
count(question_id)/count(distinct(q.device_id)) 
from question_practice_detail q
inner join user_profile u on q.device_id = u.device_id
group by university
order by university asc
```

#### 组合查询

```
select device_id,gender,age,gpa from
user_profile
where university = '山东大学' 
'''union all'''
select device_id,gender,age,gpa from
user_profile
where gender = 'male'
```

#### rank over（开窗）

分组：多用于分类，再聚合计算，平均最大最小总和等，分组后相当于丢失了组内每一行数据的详细信息，只有组这个概念了，返回的也是每个组的聚合值，一般行数小于原表行数.

​		用了group后，前面select要么是group的列，要么是聚合函数

开窗：行数不变，后面新增一个或多个列代表窗口函数的计算结果

​		一般和select后其他列放一起，select a, b, rank()  over(partition  by()  order  by)

rank：1，2，2，4（并列第二没第三）

dense rank：1，2，2，3

不加partition：123456

加partition：123123，即分区内独立排序

```
取score前三名的学生，有并列的就都取出来
SELECT student_id, score
FROM (
    -- 子查询：为每个学生的成绩分配排名
    SELECT 
        student_id, 
        score,
        DENSE_RANK() OVER (ORDER BY score DESC) as ranking
    FROM student_scores
) ranked_scores
-- 筛选出排名在前三位的学生
WHERE ranking <= 3;
```

```
关联3张表的结果，差ranking和play_pv字段，分组，开窗，然后统计
select month,ranking,song_name,play_pv
from(
    select
        month,
        rank() over(partition by month order by play_pv desc,song_id) as ranking,
        song_id,
        song_name,
        play_pv
    from(
    #每个月每首歌的播放次数 
        select 
            month(pl.fdate) as month, 
            pl.song_id,
            #一定是一个聚合，否则数据库不知道用哪个名字代表该group 
            max(song_name) as song_name,
            count(1) as play_pv
        from play_log pl
        inner join user_info ui on pl.user_id = ui.user_id
        inner join song_info si on pl.song_id = si.song_id
        where ui.age between 18 and 25
        and si.singer_name = '周杰伦'
        group by month(pl.fdate),pl.song_id
        ) a
    ) a
where ranking<=3
order by month,ranking
```

#### with as

with as 相当于为一个查询结果创建一个临时表，后续查询使用这个临时表，查询结束清除.

可以提取复杂子查询逻辑，是主查询逻辑更清楚，大型查询，将逻辑拆分为多个CTE更好组织代码，便于理解维护

with as 定义的sql片段被称为 CTE (Common Table Expression,公共表达式)

```sql
WITH cte_name ( column_name1,2,3... )  AS (
    CTE_query_definition 
)
WITH nv_user(myid,mysex,myname) AS (
    SELECT * FROM user WHERE sex='nv' ORDER BY id DESC
)
```

with 临时表名 as ( sql 语句 1 ) select xx from 临时表名

```
WITH  a AS (  
        SELECT  
            month(fdate) AS month,  
            t1.user_id,  
            t1.song_id,  
            t2.song_name,  
            t2.singer_name,  
            t3.age  
        FROM  
            play_log t1  
        INNER JOIN song_info t2 ON t1.song_id = t2.song_id  
        INNER JOIN user_info t3 ON t1.user_id = t3.user_id  
        WHERE  
            t3.age BETWEEN 18 AND 25  
            AND t2.singer_name = '周杰伦'  
            AND YEAR(fdate) = 2022  
    ),  
    b AS (  
        SELECT  
            month,  
            song_id,
            song_name,  
            COUNT(*) AS play_pv  
        FROM  
            a  
        GROUP BY  
            month, song_id,song_name  
    ),  
    c AS (  
        SELECT  
            month,  
            song_name,  
            play_pv,  
            ROW_NUMBER() OVER (  
                PARTITION BY month  
                ORDER BY play_pv DESC, song_id   
            ) AS ranking  
        FROM  
            b  
    )  
SELECT  
    month,  
    ranking,  
    song_name,  
    play_pv  
FROM  
    c  
WHERE  
    ranking <= 3  
ORDER BY  
    month,  
    ranking;
```

#### mysql如何实现如果不存在就插入如果存在就更新？

```
INSERT INTO users (id, name) VALUES (1, 'Alice')
	ON DUPLICATE KEY UPDATE name = VALUES(name);
```

#### 高质量sql建议

1. 查询SQL尽量不要使用select *，而是select具体字段。

2. 如果知道查询结果只有一条或者只要最大/最小一条记录，建议用limit 1

   - 加上limit 1后,只要找到了对应的一条记录,就不会继续向下扫描了,效率将会大大提高。
   - 当然，如果 name 是唯一索引的话，是不必要加上limit 1了，因为limit的存在主要就是为了防止全表扫描，从而提高性能，如果一个语句本身可以预知不用全表扫描，有没有limit ，性能的差别并不大。

3. 应尽量避免在where子句中使用or来连接条件

   - 使用or可能会使索引失效，从而全表扫描。

4. 优化like语句

   - 日常开发中，如果用到模糊关键字查询，很容易想到like，但是like很可能让你的索引失效。
   - 基于最左前缀原则，如果对拥有索引的字段进行查询，建议将“%”放在后面。例如： ydUserId = “abc%”;

5. 使用联合索引时，注意索引列的顺序，一般遵循最左匹配原则。

6. 尽量避免在索引列上使用mysql的内置函数

   - 索引列上使用mysql的内置函数，索引失效

7. Inner join 、left join、right join，优先使用Inner join，如果是left join，左边表结果尽量小

   - Inner join 内连接，在两张表进行连接查询时，只保留两张表中完全匹配的结果集
   - left join 在两张表进行连接查询时，会返回左表所有的行，即使在右表中没有匹配的记录。
   - right join 在两张表进行连接查询时，会返回右表所有的行，即使在左表中没有匹配的记录。

8. 应尽量避免在 where 子句中使用 != 或 < > 操作符，否则将引擎放弃使用索引而进行全表扫描（可能发现范围太大，但一般 ≥ 和＞本身在使用索引上和性能上差异不大)

9. 慎用distinct关键字

   distinct 关键字一般用来过滤重复记录，以返回不重复的记录。在查询一个字段或者很少字段的情况下使用时，给查询带来优化效果。但是在字段很多的时候使用，却会大大降低查询效率。

10. where子句中考虑使用默认值代替null。

    - 如果把null值，换成默认值，很多时候让走索引成为可能，同时，表达意思会相对清晰一点。
    - 这点作为参考，实际数据库设计的时候，出现很多不设置默认值的情况

11. exist & in的合理利用

    - 因为exists查询的理解就是，先执行主查询，获得数据后，再放到子查询中做条件验证，根据验证结果（true或者false），来决定主查询的数据结果是否得意保留。

12. 尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型

    - 相对于数字型字段，字符型会降低查询和连接的性能，并会增加存储开销。

13. 索引不适合建在有大量重复数据的字段上，如性别这类型数据库字段。

    - 因为SQL优化器是根据表中数据量来进行查询优化的，如果索引列有大量重复数据，Mysql查询优化器推算发现不走索引的成本更低，很可能就放弃索引了。

14. 尽可能使用varchar/nvarchar 代替 char/nchar。
