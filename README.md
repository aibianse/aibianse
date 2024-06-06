### Hi there 👋

<!--
**aibianse/aibianse** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->



Presto官方文档：https://prestodb.io/docs/current/index.html

常用函数，关键字
json_extract_scalar：解析json	--官方文档：https://prestodb.io/docs/current/functions/json.html?highlight=json_extract_scalar#json_extract_scalar
示例Json串：{"modulus_test": "{\"modulus_name\":\"100\"}}
函数语法使用：select json_extract_scalar(modulus_test, '$.modulus_name') from table_name;

date_add：日期时间增加函数，场景：2021-12-25 01:24:51，往后增加2小时
函数语法使用：select cast(date_add('hour',2,cast('2021-12-25 01:24:51' as timestamp)) as varchar)

regexp_like：模糊匹配，相当于like，like '%PC%' or like '%LS%' or like '%IC%'
函数语法使用：where not regexp_like(waybill_number,'PC|LS|IC')

to_unixtime：日期时间格式转换成秒级别的时间戳
函数语法使用：select cast(to_unixtime(cast('2017-08-30 10:36:15' as timestamp)) as bigint)

from_unixtime：时间戳格式转换成日期格式
函数语法使用：select from_unixtime(1661156691000 / 1000)

date_format：时分秒日期格式转换特定日期格式
select date_format(from_unixtime(1661156691000 / 1000), '%Y-%m')

substring：字符串切片，第一个参数：需要切片的字符串，第二个参数：起始索引（注意点：Presto索引从1开始，不是从0开始），第三个参数：切片步长
函数语法使用：select substring('2022-03-01',6,5)
示例运行结果：03-01

cardinality：获取数组长度
函数语法使用：select cardinality(split('a/b/c','/'))

truncate：小数位保留特定位数，并不四舍五入
函数语法使用：select truncate(66.42315, 4)

coalesce：判空
函数语法使用：coalesce(test,test_1,0)，多个字段，取第一个不为空的字段，如果test,test_1字段值都为空，则赋值=0

多个值取最大值和最小值函数
select greatest(0,1)，返回最大值
select least(0,1)，返回最小值

把查询出来的数据，创建一张临时表存放；orc：数据存储格式
create table table_name with(format='orc') as
select * from table_name

cross join unnest：按某个字段（列表类型）逐个打平出来
select * from test_1 as t1 cross join unnest (t1.name_list) as v1(name_list_test)

开窗函数：row_number() over(partition by xxx order by xxx desc)
查询出来的每一行记录生成一个序号，依次排序且不会重复 从1开始
row_number() over(partition by employee_number)

向下挪位函数，lag(挪位的字段,挪位的位数,第一位的默认值)
lag(operator_time,1,operator_time) over(partition by employee_number order by operator_time)

向下挪位函数，lead(挪位的字段,挪位的位数,第一位的默认值)
lead

正则匹配，首尾的"#"号替换掉
select regexp_replace('#75583198#75580268#','^#|#$','')

array_agg：把查询的结果聚合成数组
select array_agg(xxx) from xxx group by xxx

array_join：把数组中的元素用特定分隔符连接成字符串
select array_join(array_agg(xxx), '-') from xxx

position：字符在字符串的起始位置，可用判断是否包含特定字符
select position('test' in 'test_abc')

array_intersect：返回两个数组的交集
select array_intersect(split('a-b-c','-'),split('a-b-c-d-e','-'))

array_distinct：数组去重
select array_distinct(split('a-b-c-a-b-b','-'))

array_sort：数组排序
select array_sort(split('a-b-c-a-b-b','-'))

array_except：返回两个数组的差集
select array_except(split('a-b-c','-'),split('a-b-c-d-e','-'))

drop table if exists vdm_pms.v_o_operator_modulus_main_config_ymj;


-- create table vdm_pms.v_o_operator_modulus_main_config_ymj with(format='orc') as -- presto语法
create table vdm_pms.v_o_operator_modulus_main_config_ymj stored as orc as -- spark语法
select * from ads_pms.ads_pms_emp_opt_perf_amount_detail_merge_cm where perf_month = '2023-06';

set session hive.insert_existing_partitions_behavior = 'overwrite';
insert into vdm_pms.v_o_operator_test_temp_001_ymj
select * from test

set session hive.insert_existing_partitions_behavior = 'overwrite';
insert into vdm_pms.v_o_operator_test_temp_001_ymj

with test_1 as (
    select min(num) as num, name_list_test
    from (select '1' as num, name_list_test
          from (select *
                from vdm_pms.v_o_operator_test_temp_001_ymj
                where co_operator_number is not null
                  and co_operator_number <> '') as t1
                   cross join
               unnest(split(t1.co_operator_number, '-')) as v1(name_list_test))
    group by name_list_test)

-- select row_number() over(partition by name_list_test order by name_list_test desc),* from test_1

   , test_2 as (
    select t1.*, t2.number_list
    ,rand(cardinality(split(number_list, '-'))) +1 as rand_1
    ,rand(cardinality(split(number_list, '-'))) +1 as rand_2
    ,rand(cardinality(split(number_list, '-'))) +1 as rand_3
    ,array_distinct(split(number_list, '-')) as split_test
    from (select '1' as num, * from vdm_pms.v_o_operator_test_temp_001_ymj) as t1
             inner join
         (select '1' as num, array_join(array_agg(name_list_test), '-') as number_list
          from (select * from test_1 limit 10000)
          group by num) as t2 on t1.num = t2.num)
          
          -- select * from test_2
select id,
       waybill_number,
       waybill_type,
       waybill_count,
       scan_number,
       scan_number_order,
       uploader_name,
       uploader_employee_number,
       uploader_network,
       uploader_network_id,
       scan_time,
       uploader_date,
       scan_status,
       trace_id,
       enabled_flag,
       created_by,
       creation_date,
       updated_by,
       updation_date,
       sign_detail_type,
       serial_number,
      array_join(array_distinct(split(concat(split_test[rand_1], '-'
                                           , split_test[rand_2], '-'
                                           , split_test[rand_3]
                             )
          , '-')), '-') as co_operator_number,
       co_operator_name,
       etl_date,
       etl_time,
       insert_time
from test_2
