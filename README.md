# mysql触发器使用udf添加拓展功能调用接口

## Windows下添加拓展功能
- 安装脚本打开http连接工具 curl // 命令行访问接口使用
- 生成udf工具（windows为.dll文件；linux为.so文件）在mysql生成拓展功能命令；
    ```bash
    DROP FUNCTION IF EXISTS sys_exec; // 执行任意命令，并将输出返回。
    CREATE FUNCTION sys_exec RETURNS integer SONAME 'lib_mysqludf_sys_x64.dll'; // 

    DROP FUNCTION IF EXISTS sys_eval; // 执行任意命令，并将退出码返回。
    CREATE FUNCTION sys_eval RETURNS string SONAME 'lib_mysqludf_sys_x64.dll';

    Drop FUNCTION IF EXISTS sys_get; // 获取一个环境变量。
    Create FUNCTION sys_get RETURNS string SONAME 'lib_mysqludf_sys.so';

    Drop FUNCTION IF EXISTS sys_set; // 创建或修改一个环境变量。
    Create FUNCTION sys_set RETURNS int SONAME 'lib_mysqludf_sys.so';
    ```

## Linux下添加拓展功能
- lib_mysqludf_sys.so复制到mysql/lib/plugin目录下。
- 在mysql中创建函数(根据需要选取)：
    ```bash
    Drop FUNCTION IF EXISTS lib_mysqludf_sys_info;
    Drop FUNCTION IF EXISTS sys_get;
    Drop FUNCTION IF EXISTS sys_set;
    Drop FUNCTION IF EXISTS sys_exec;
    Drop FUNCTION IF EXISTS sys_eval;
    Create FUNCTION lib_mysqludf_sys_info RETURNS string SONAME 'lib_mysqludf_sys.so';
    Create FUNCTION sys_get RETURNS string SONAME 'lib_mysqludf_sys.so';
    Create FUNCTION sys_set RETURNS int SONAME 'lib_mysqludf_sys.so';
    Create FUNCTION sys_exec RETURNS int SONAME 'lib_mysqludf_sys.so';
    Create FUNCTION sys_eval RETURNS string SONAME 'lib_mysqludf_sys.so';
    ```
- 创建触发器并执行触发器代码
    ```bash
    // 创建触发器
    CREATE TRIGGER CheckInOrOut AFTER INSERT ON checkinout for each row

    // 执行触发器
    BEGIN
    SET @address = 'curl http://www.baidu.com';
    SELECT a.userid into @id FROM checkinout a order by id desc LIMIT 0,1; 
    SELECT a.badgenumber into @userId FROM userinfo a,checkinout b where a.userid=b.userid AND b.userid=@id LIMIT 0,1;
    SET @url=CONCAT(@address, @userId);
    SELECT sys_eval(@url) into @aa;
    END
    ```