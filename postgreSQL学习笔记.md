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
  
  
  #### sql记录
  ```sql
  -- OVER()是一个开窗函数 对集合进行聚合计算
-- PARTITION BY dept_id 对单位ID标识进行分区
-- position()提供从头查找返回第一个匹配到字符串的下标
WITH t1 AS ( SELECT ID, NAME, dept_id, adid FROM investigation.ia_adc_danad WHERE dept_id = 1067246875800000064 -- 从危险区记录表中查询出等于此单位ID标识的t1临时表
),
T4 AS (
	SELECT T
		.ID AS wxqId,-- 危险区ID标识
		T.NAME AS wxqname,-- 危险区名称
		T.dept_id,-- 单位ID标识
		T.adid -- 所属行政区划ID
		
	FROM
		(
		SELECT
			t1.ID,
			t1.NAME,
			t1.adid,
			t1.dept_id,
			ROW_NUMBER ( ) OVER ( PARTITION BY t1.dept_id ORDER BY random( ) ) AS ROW -- 以单位识别ID进行分区并 对结果集进行随机排序后为结果集表新增一个row字段再为每一条记录的row字段以第一条记录为1生成一个连续的整数对其进行填充
			
		FROM
			t1 
		) T -- 危险区记录数据随机排序，但是row字段有序的临时表 T
		
	WHERE
		ROW <= 25 -- 返回危险区记录数据前五条数据
		
	),
	t5 AS (
	SELECT
		tt.*, -- tt 由t4包装的一个临时表， 将tt表中所有字段内容平铺到t5临时表中
		COALESCE ( t5.phone_number, '' ) AS mobile, -- 当责任人联系电话为空时返回'', 否者返回责任人联系电话
		COALESCE ( t5.NAME, '' ) AS contacts_name, -- 同理， 责任人名字
		COALESCE ( t5.duty_type, '' ) AS charge_position, -- 同理， 危险区责任人类型
		COALESCE ( t5.job, '' ) AS job -- 同理， 责任人职务
	FROM
		(
		SELECT
			t4.wxqname,-- 危险区名称
			-- 原来SQL的写法
			--( SELECT td.NAME FROM "system".sys_dept td WHERE td.ID = t4.dept_id ) AS adname, -- 从系统数据库中的系统部门表中获取部门名称
			--( SELECT td.pid FROM "system".sys_dept td WHERE td.ID = t4.dept_id ) AS pid, -- 同理，从系统数据库中的系统部门表获取上级ID
			td.NAME AS adname, -- 部门名称
			td.pid AS pid, -- 上级ID
			t4.adid, -- 所属行政区划ID
			t4.wxqId -- 危险区ID标识
		FROM
			t4 -- t4 是危险区记录数据随机排序，但是row字段有序的临时表 T 的前5条数据
			-- 我将其改进为连接查询
			LEFT JOIN "system".sys_dept td ON t4.dept_id = td.ID -- t4临时表与系统部门表td关联
		) tt -- tt 由t4包装的一个临时表
		LEFT JOIN ( SELECT * FROM warning.duty_contacts WHERE POSITION ( '2' IN duty_type ) > 0 ) t5 -- 从水雨情预警数据库中的责任人联系表中查询行政责任人的相关信息作为临时表t5, 当责任人兼有一种或几种不同的职责时POSITION()函数返回的下标一定大于零所以能将此类型的责任人查询出来
-- duty_contacts 表字段解释
-- duty_contacts.type 表示联系人类型.1：危险区责任人；2：自动预警人员
-- duty_contacts.duty_type 表示危险区责任人类型（仅type为1该字段有值）。
-- duty_contacts.danad_id  危险区ID（仅type为1该字段有值）
-- 1：监测巡查责任人；2：预警转移责任人；3：行政责任人
 ON tt.wxqid = t5.danad_id --  危险区记录数据随机排序临时表tt与责任人联系表t5关联
	) -- t5 临时表封装完成
	
	SELECT
	t6.NAME AS sznm, -- 上级部门名称
	t5.adname, -- 部门名称
	COALESCE ( t7.adnm, '' ) AS xzname, -- 同理， 乡镇名字
	t5.wxqname, -- 危险区名称
	t5.contacts_name, -- 危险区责任人名字
	t5.mobile, -- 危险区责任人电话
	t5.job, -- 危险区责任人职务
	t5.charge_position -- 危险区责任人类型
FROM
	t5
LEFT JOIN "system".sys_dept t6 ON t5.pid = t6.ID  -- t5临时表与系统部门表t6关联，pid 为上级部门ID
	LEFT JOIN warning.ad_info_b t7 ON t5.adid = t7.ID -- t5临时表与受灾影响区域的信息表t7关联, adid 为所属行政区划ID
  
  ```
