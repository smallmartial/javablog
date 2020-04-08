# mycat单表插入(一)

## 1.单库单表插入，交互图：

![image-20200408144230433](mycat%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%8D%95%E8%A1%A8%E6%8F%92%E5%85%A5/image-20200408144230433.png)

mycat server的请求流程如下：

1.mycat server接收 客户端发来的请求。

2.然后获得路由，进行路由

3.获取mysql的链接，执行sql

4.响应执行之后的结果，发送给客户端。

## 2.执行sql的过程

![mycat单表插入-Page-1](mycat%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%8D%95%E8%A1%A8%E6%8F%92%E5%85%A5/mycat%E5%8D%95%E8%A1%A8%E6%8F%92%E5%85%A5-Page-1.png)

- （1、2）的过程是接收一条sql命令。
- （3）不同的mysql命令会分发到不同的方法执行，INSERT/SELECT/UPDATE/DELETE 等 SQL 归属于 MySQLPacket.COM_QUERY

```java
/**
 * 前端命令处理器
 *
 * @author mycat
 */
public class FrontendCommandHandler implements NIOHandler
{
//INSERT/SELECT/UPDATE/DELETE 等 SQL 归属于 MySQLPacket.COM_QUERY
    @Override
    public void handle(byte[] data)
    {
       //.....省略代码
        switch (data[4])
        {
            case MySQLPacket.COM_INIT_DB:
                commands.doInitDB();
                source.initDB(data);
                break;
                //查询命令
            case MySQLPacket.COM_QUERY:
               //计数查询
                commands.doQuery();
                //执行查询命令
                source.query(data);
                break;
            case MySQLPacket.COM_PING:
                commands.doPing();
                source.ping();
                break;
            case MySQLPacket.COM_QUIT:
                commands.doQuit();
                source.close("quit cmd");
                break;
            case MySQLPacket.COM_PROCESS_KILL:
                commands.doKill();
                source.kill(data);
                break;

				//......

        }
    }

}
```

- （4）将二进制数组解析成sql

  ```java
  //FrontendConnection.java	
  public void query(byte[] data) {
  		
  		// 取得语句
  		String sql = null;		
  		try {
  			MySQLMessage mm = new MySQLMessage(data);
  			mm.position(5);
  			sql = mm.readString(charset);
  		} catch (UnsupportedEncodingException e) {
  			writeErrMessage(ErrorCode.ER_UNKNOWN_CHARACTER_SET, "Unknown charset '" + charset + "'");
  			return;
  		}		
  		//执行sql语句
  		this.query( sql );
  	}
  ```

- （5）解析Sql语句

  ```java
  public class ServerQueryHandler implements FrontendQueryHandler {
  	//......
      
  	@Override
  	public void query(String sql) {
  		
  		ServerConnection c = this.source;
  		if (LOGGER.isDebugEnabled()) {
  			LOGGER.debug(new StringBuilder().append(c).append(sql).toString());
  		}
  		//解析sql语句
  		int rs = ServerParse.parse(sql);
  		int sqlType = rs & 0xff;
  		
  		switch (sqlType) {
  		//explain sql
  		case ServerParse.EXPLAIN:
  			ExplainHandler.handle(sql, c, rs >>> 8);
  			break;
  		//explain2 datanode=? sql=?
  		case ServerParse.EXPLAIN2:
  			Explain2Handler.handle(sql, c, rs >>> 8);
  			break;
  		case ServerParse.SET:
  			SetHandler.handle(sql, c, rs >>> 8);
  			break;
  		case ServerParse.SHOW:
  			ShowHandler.handle(sql, c, rs >>> 8);
  			break;
  		case ServerParse.SELECT:
  			SelectHandler.handle(sql, c, rs >>> 8);
  			break;
  		case ServerParse.START:
  			StartHandler.handle(sql, c, rs >>> 8);
  			break;
  		case ServerParse.BEGIN:
  			BeginHandler.handle(sql, c);
  			break;
  		//不支持oracle的savepoint事务回退点
  		case ServerParse.SAVEPOINT:
  			SavepointHandler.handle(sql, c);
  			break;
  		case ServerParse.KILL:
  			KillHandler.handle(sql, rs >>> 8, c);
  			break;
  		//不支持KILL_Query
  		case ServerParse.KILL_QUERY:
  			LOGGER.warn(new StringBuilder().append("Unsupported command:").append(sql).toString());
  			c.writeErrMessage(ErrorCode.ER_UNKNOWN_COM_ERROR,"Unsupported command");
  			break;
  		case ServerParse.USE:
  			UseHandler.handle(sql, c, rs >>> 8);
  			break;
  		case ServerParse.COMMIT:
  			c.commit();
  			break;
  		case ServerParse.ROLLBACK:
  			c.rollback();
  			break;
  		case ServerParse.HELP:
  			LOGGER.warn(new StringBuilder().append("Unsupported command:").append(sql).toString());
  			c.writeErrMessage(ErrorCode.ER_SYNTAX_ERROR, "Unsupported command");
  			break;
  		case ServerParse.MYSQL_CMD_COMMENT:
  			c.write(c.writeToBuffer(OkPacket.OK, c.allocate()));
  			break;
  		case ServerParse.MYSQL_COMMENT:
  			c.write(c.writeToBuffer(OkPacket.OK, c.allocate()));
  			break;
          case ServerParse.LOAD_DATA_INFILE_SQL:
              c.loadDataInfileStart(sql);
              break;
  		case ServerParse.MIGRATE:
  			MigrateHandler.handle(sql,c);
  			break;
  		case ServerParse.LOCK:
          	c.lockTable(sql);
          	break;
          case ServerParse.UNLOCK:
          	c.unLockTable(sql);
          	break;
  		default:
  			if(readOnly){
  				LOGGER.warn(new StringBuilder().append("User readonly:").append(sql).toString());
  				c.writeErrMessage(ErrorCode.ER_USER_READ_ONLY, "User readonly");
  				break;
  			}
  			c.execute(sql, rs & 0xff);
  		}
  	}
  
  }
  
  
  public final class ServerParse {
      //......
      
  	//解析Sql语句
      public static int parse(String stmt) {
  		int length = stmt.length();
  		//FIX BUG FOR SQL SUCH AS /XXXX/SQL
  		int rt = -1;
  		for (int i = 0; i < length; ++i) {
  			switch (stmt.charAt(i)) {
  			case ' ':
  			case '\t':
  			case '\r':
  			case '\n':
  				continue;
  			case '/':
  				// such as /*!40101 SET character_set_client = @saved_cs_client
  				// */;
  				if (i == 0 && stmt.charAt(1) == '*' && stmt.charAt(2) == '!' && stmt.charAt(length - 2) == '*'
  						&& stmt.charAt(length - 1) == '/') {
  					return MYSQL_CMD_COMMENT;
  				}
  			case '#':
  				i = ParseUtil.comment(stmt, i);
  				if (i + 1 == length) {
  					return MYSQL_COMMENT;
  				}
  				continue;
  			case 'A':
  			case 'a':
  				rt = aCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'B':
  			case 'b':
  				rt = beginCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'C':
  			case 'c':
  				rt = commitOrCallCheckOrCreate(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'D':
  			case 'd':
  				rt = deleteOrdCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'E':
  			case 'e':
  				rt = explainCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'I':
  			case 'i':
  				rt = insertCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  				case 'M':
  				case 'm':
  					rt = migrateCheck(stmt, i);
  					if (rt != OTHER) {
  						return rt;
  					}
  					continue;
  			case 'R':
  			case 'r':
  				rt = rCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'S':
  			case 's':
  				rt = sCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'T':
  			case 't':
  				rt = tCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'U':
  			case 'u':
  				rt = uCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'K':
  			case 'k':
  				rt = killCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'H':
  			case 'h':
  				rt = helpCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			case 'L':
  			case 'l':
  				rt = lCheck(stmt, i);
  				if (rt != OTHER) {
  					return rt;
  				}
  				continue;
  			default:
  				continue;
  			}
  		}
  		return OTHER;
  	}
  	//......
  
  }
  ```

- （6）执行sql

  ```java
   // 【ServerConnection.java】
  	public void execute(String sql, int type) {
  		//连接状态检查
  		if (this.isClosed()) {
  			LOGGER.warn("ignore execute ,server connection is closed " + this);
  			return;
  		}
  		// 事务状态检查
  		if (txInterrupted) {
  			writeErrMessage(ErrorCode.ER_YES,
  					"Transaction error, need to rollback." + txInterrputMsg);
  			return;
  		}
  
  		// 检查当前使用的DB
  		String db = this.schema;
  		boolean isDefault = true;
  		if (db == null) {
  			db = SchemaUtil.detectDefaultDb(sql, type);
  			if (db == null) {
  				writeErrMessage(ErrorCode.ERR_BAD_LOGICDB, "No MyCAT Database selected");
  				return;
  			}
  			isDefault = false;
  		}
  		
  		// 兼容PhpAdmin's, 支持对MySQL元数据的模拟返回
  		//// TODO: 2016/5/20 支持更多information_schema特性
  		if (ServerParse.SELECT == type 
  				&& db.equalsIgnoreCase("information_schema") ) {
  			MysqlInformationSchemaHandler.handle(sql, this);
  			return;
  		}
  
  		if (ServerParse.SELECT == type 
  				&& sql.contains("mysql") 
  				&& sql.contains("proc")) {
  			
  			SchemaUtil.SchemaInfo schemaInfo = SchemaUtil.parseSchema(sql);
  			if (schemaInfo != null 
  					&& "mysql".equalsIgnoreCase(schemaInfo.schema)
  					&& "proc".equalsIgnoreCase(schemaInfo.table)) {
  				
  				// 兼容MySQLWorkbench
  				MysqlProcHandler.handle(sql, this);
  				return;
  			}
  		}
  		//  对应时序图第8步
  		SchemaConfig schema = MycatServer.getInstance().getConfig().getSchemas().get(db);
  		if (schema == null) {
  			writeErrMessage(ErrorCode.ERR_BAD_LOGICDB,
  					"Unknown MyCAT Database '" + db + "'");
  			return;
  		}
  
  		//fix navicat   SELECT STATE AS `State`, ROUND(SUM(DURATION),7) AS `Duration`, CONCAT(ROUND(SUM(DURATION)/*100,3), '%') AS `Percentage` FROM INFORMATION_SCHEMA.PROFILING WHERE QUERY_ID= GROUP BY STATE ORDER BY SEQ
  		if(ServerParse.SELECT == type &&sql.contains(" INFORMATION_SCHEMA.PROFILING ")&&sql.contains("CONCAT(ROUND(SUM(DURATION)/"))
  		{
  			InformationSchemaProfiling.response(this);
  			return;
  		}
  		
  		/* 当已经设置默认schema时，可以通过在sql中指定其它schema的方式执行
  		 * 相关sql，已经在mysql客户端中验证。
  		 * 所以在此处增加关于sql中指定Schema方式的支持。
  		 */
  		if (isDefault && schema.isCheckSQLSchema() && isNormalSql(type)) {
  			SchemaUtil.SchemaInfo schemaInfo = SchemaUtil.parseSchema(sql);
  			if (schemaInfo != null && schemaInfo.schema != null && !schemaInfo.schema.equals(db)) {
  				SchemaConfig schemaConfig = MycatServer.getInstance().getConfig().getSchemas().get(schemaInfo.schema);
  				if (schemaConfig != null)
  					schema = schemaConfig;
  			}
  		}
  		//路由计算
  		routeEndExecuteSQL(sql, type, schema);
  
  	}
  
  	public void routeEndExecuteSQL(String sql, final int type, final SchemaConfig schema) {
  		// 路由计算
  		RouteResultset rrs = null;
  		try {
  			rrs = MycatServer
  					.getInstance()
  					.getRouterservice()
  					.route(MycatServer.getInstance().getConfig().getSystem(),
  							schema, type, sql, this.charset, this);
  
  		} catch (Exception e) {
  			StringBuilder s = new StringBuilder();
  			LOGGER.warn(s.append(this).append(sql).toString() + " err:" + e.toString(),e);
  			String msg = e.getMessage();
  			writeErrMessage(ErrorCode.ER_PARSE_ERROR, msg == null ? e.getClass().getSimpleName() : msg);
  			return;
  		}
  		if (rrs != null) {
  			// session执行 对应时序图第9步
  			session.execute(rrs, rrs.isSelectForUpdate()?ServerParse.UPDATE:type);
  		}
  		
   	}
  	
  ```

  

 