# mycat单表插入(二)

## 3.获取路由的结果

![mycat单表插入-Page-3](mycat%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%8D%95%E8%A1%A8%E6%8F%92%E5%85%A5%EF%BC%88%E4%BA%8C%EF%BC%89/mycat%E5%8D%95%E8%A1%A8%E6%8F%92%E5%85%A5-Page-3.png)

- ## 【 1 - 2 】【 11】获取路由的主流程

  ```java
  【RouteService.java】
  	public RouteResultset route(SystemConfig sysconf, SchemaConfig schema,
  			int sqlType, String stmt, String charset, ServerConnection sc)
  			throws SQLNonTransientException {
  		RouteResultset rrs = null;
  		String cacheKey = null;
  
  		//......省略代码
      
  		int hintLength = RouteService.isHintSql(stmt);
  		if(hintLength != -1){
  		//......省略代码
  		} else {
  			stmt = stmt.trim();
  			//策略模式 策略工厂？？
  			rrs = RouteStrategyFactory.getRouteStrategy().route(sysconf, schema, sqlType, stmt,
  					charset, sc, tableId2DataNodeCache);
  		}
  
  		if (rrs != null && sqlType == ServerParse.SELECT && rrs.isCacheAble()) {
  			sqlRouteCache.putIfAbsent(cacheKey, rrs);
  		}
  		//TODO 检查迁移路由？
  		checkMigrateRule(schema.getName(),rrs,sqlType);
  		return rrs;
  	}    
  
  
  【AbstractRouteStrategy.java】
  	@Override
  	public RouteResultset route(SystemConfig sysConfig, SchemaConfig schema, int sqlType, String origSQL,
  			String charset, ServerConnection sc, LayerCachePool cachePool) throws SQLNonTransientException {
  
  		//对应schema标签checkSQLschema属性，把表示schema的字符去掉
  		if (schema.isCheckSQLSchema()) {
  			origSQL = RouterUtil.removeSchema(origSQL, schema.getName());
  		}
  
  		/**
       * 处理一些路由之前的逻辑
       * 全局序列号，父子表插入
       */
  		if ( beforeRouteProcess(schema, sqlType, origSQL, sc) ) {
  			return null;
  		}
  
  		/**
  		 * SQL 语句拦截
  		 */
  		String stmt = MycatServer.getInstance().getSqlInterceptor().interceptSQL(origSQL, sqlType);
  		if (!origSQL.equals(stmt) && LOGGER.isDebugEnabled()) {
  			LOGGER.debug("sql intercepted to " + stmt + " from " + origSQL);
  		}
  
  
  		RouteResultset rrs = new RouteResultset(stmt, sqlType);
  
  
  	//......省略代码
      
  		/**
  		 * 检查是否有分片
  		 */
  		if (schema.isNoSharding() && ServerParse.SHOW != sqlType) {
  			rrs = RouterUtil.routeToSingleNode(rrs, schema.getDataNode(), stmt);
  		} else {
  			RouteResultset returnedSet = routeSystemInfo(schema, sqlType, stmt, rrs);
  			if (returnedSet == null) {
  				rrs = routeNormalSqlWithAST(schema, stmt, rrs, charset, cachePool,sqlType,sc);
  			}
  		}
  
  		return rrs;
  	}
  
  	/**    
  ```

- 【3-6】路由的前置处理

  ```java
  【AbstractRouteStrategy.java】
  /**
  	 * 路由之前必要的处理
  	 * 主要是全局序列号插入，还有子表插入
  	 */
  	private boolean beforeRouteProcess(SchemaConfig schema, int sqlType, String origSQL, ServerConnection sc)
  			throws SQLNonTransientException {
  		
  		return // 处理 id 使用 全局序列号
  				RouterUtil.processWithMycatSeq(schema, sqlType, origSQL, sc)
  						//处理ER子表
  				|| (sqlType == ServerParse.INSERT && RouterUtil.processERChildTable(schema, origSQL, sc))
  						//处理ID自增
  						|| (sqlType == ServerParse.INSERT && RouterUtil.processInsert(schema, sqlType, origSQL, sc));
  	}
  ```

## 4.获取mysql连接，执行sql

![mycat单表插入-Page-4](mycat%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%8D%95%E8%A1%A8%E6%8F%92%E5%85%A5%EF%BC%88%E4%BA%8C%EF%BC%89/mycat%E5%8D%95%E8%A1%A8%E6%8F%92%E5%85%A5-Page-4.png)

- 1-8获取mysql的执行过程
- 9-13发送sql到mysql的服务器，执行sql语句

## 5.响应sql结果

![image-20200409211559659](mycat%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%8D%95%E8%A1%A8%E6%8F%92%E5%85%A5%EF%BC%88%E4%BA%8C%EF%BC%89/image-20200409211559659.png)

- 1到 4 处理mysql server数据响应
- 5到8发送插入成功结果给mysql