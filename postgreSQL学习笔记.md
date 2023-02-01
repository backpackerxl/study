#### sql 第一天
``` sql
-- 按值班班次ID查询值班记录列表(新接口)
-- 临时表t1
-- 当file_ids 为空时将file_id设置为空， 
-- 反之将file_ids中按，拆分为file_id不同其他字段相同的多条记录
WITH t1 AS (
	SELECT ID
		,
		schedule_id,
		record_time,
		"content",
		file_cd,
		record_type,
		crt_by,
		crt_dt,
		upd_dt,
		file_ids,
		NULL AS file_id 
		FROM
		duty_record -- 值班记录表
	WHERE
		schedule_id = 'b0bf52521cab2166f82fb57fcc142f5b' 
		AND file_ids IS NULL UNION -- 连接运算符， 相当于+号操作， 作用：将多个查询结果集合并为一个最终结果集
	SELECT ID
		,
		schedule_id,
		record_time,
		"content",
		file_cd,
		record_type,
		crt_by,
		crt_dt,
		upd_dt,
		file_ids,
		regexp_split_to_table( file_ids, ',' ) file_id -- 将file_ids中按，拆分为file_id不同其他字段相同的多条记录
	FROM
		duty_record 
	WHERE
		schedule_id = 'b0bf52521cab2166f82fb57fcc142f5b' 
		AND file_ids IS NOT NULL 
	) SELECT
	t1.ID,
	t1.schedule_id,
	t1.record_time,
	t1."content",
	t1.file_cd,
	t1.record_type,
	t1.crt_by,
	t1.crt_dt,
	t1.upd_dt,
	t1.file_ids file_result_list,
	so.ID oss_id,
	so.creator,
	so.create_date,
	so.file_name,
	so.suffix 
FROM
	t1
	LEFT JOIN SYSTEM.sys_oss so ON t1.file_id = so.ID -- 找出存放在系统表中的附件信息
	ORDER BY
	t1.record_time DESC,
	so.create_date DESC; -- 将最后录入的数据最先展示
  ```
