```java
    public static ZdryReportEntity getEntityBySql(MysqlConfig mysqlConfig, String zjhm) {

        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        ZdryReportEntity entity = null;
        try {
            conn = MysqlConnection.getConnection(mysqlConfig);
            //定义sql
            String sql = "SELECT IDENTITY_NUMBER,LAT,LNG FROM ads_zhengwu_12385_hsjczdrqmd where IDENTITY_NUMBER=?";
            ps = conn.prepareStatement(sql);
            //获取执行sql的对象
            ps.setString(1, zjhm);
            //执行sql
            rs = ps.executeQuery();
            entity = getReport(rs);
        } catch (Exception e) {
            log.error(e.getMessage());
        } finally {
            MysqlConnection.close(rs, ps, conn);
        }
        return entity;
    }
```

```json
"mysqlConfig": {
    "password":"m3P333(48185_9%c",
    "url":"jdbc:mysql://10.197.4.35:15067/szg_qz_enterprise",
    "user":"gz_zhongda"
}
```

