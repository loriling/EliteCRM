# MYSQL的TDE

从5.7.11开始，mysql开始支持物理表空间的加密，它使用两层加密架构。
包括：主密钥（master key） 和 表空间加密密钥（tablespace key）

主密钥用于加密加密密钥，加密后的加密密钥存储在表空间文件的header中。加密密钥用于加密数据
当用户想访问加密的表时，innoDB会先用 主密钥 对之前存储在表空间header中被加密的加密密钥进行解密，得到明文的加密密钥。

再用 加密密钥 解密数据信息。加密密钥是不会被改变的（除非进行alter table testt encrytion=NO/YES）。

而 主密钥 可以通过以下命令随时改变





### 使用keyring_file插件对数据库表进行加密存储

1. windows中修改my.ini（linux中修改my.cnf，配置信息也从dll改成so，路径也改成linux中的路径）

   ```ini
   [mysqld]
   ...
   # plugin_load
   early-plugin-load=keyring_file.dll
   keyring_file_data='C:\ProgramData\MySQL\MySQL Server 8.0\keyring'
   ```

2. 重启mysql服务

3. 相关sql操作

   ```sql
   -- 查看插件是否安装成功
   SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS WHERE PLUGIN_NAME LIKE 'keyring%';
   
   -- 修改CHAT_MESSAGE表为加密表
   ALTER TABLE CHAT_MESSAGE ENCRYPTION='Y';
   
   -- 查看哪些表启用了加密
   SELECT TABLE_SCHEMA, TABLE_NAME, CREATE_OPTIONS FROM INFORMATION_SCHEMA.TABLES WHERE CREATE_OPTIONS LIKE '%ENCRYPTION=''Y''%'
   
   -- 修改CHAT_MESSAGE表为非加密表
   ALTER TABLE CHAT_MESSAGE ENCRYPTION='N';
   ```

   

