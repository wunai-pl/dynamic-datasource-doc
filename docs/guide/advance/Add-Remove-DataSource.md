# 动态添加移除数据源

```java
@RestController
@AllArgsConstructor
@RequestMapping("/datasources")
@Api(tags = "添加删除数据源")
public class LoadController {

  private final DataSource dataSource;
  private final DataSourceCreator dataSourceCreator; //3.3.1及以下版本使用這個
  private final DefaultDataSourceCreator dataSourceCreator; //3.3.2及以上版本使用這個
  private final BasicDataSourceCreator basicDataSourceCreator;
  private final JndiDataSourceCreator jndiDataSourceCreator;
  private final DruidDataSourceCreator druidDataSourceCreator;
  private final HikariDataSourceCreator hikariDataSourceCreator;

  @GetMapping
  @ApiOperation("获取当前所有数据源")
  public Set<String> now() {
      DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
      return ds.getCurrentDataSources().keySet();
  }

  @PostMapping("/add")
  @ApiOperation("通用添加数据源（推荐）")
  public Set<String> add(@Validated @RequestBody DataSourceDTO dto) {
      DataSourceProperty dataSourceProperty = new DataSourceProperty();
      BeanUtils.copyProperties(dto, dataSourceProperty);
      DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
      DataSource dataSource = dataSourceCreator.createDataSource(dataSourceProperty);
      ds.addDataSource(dto.getPollName(), dataSource);
      return ds.getCurrentDataSources().keySet();
  }

  @PostMapping("/addBasic")
  @ApiOperation(value = "添加基础数据源", notes = "调用Springboot内置方法创建数据源，兼容1,2")
  public Set<String> addBasic(@Validated @RequestBody DataSourceDTO dto) {
      DataSourceProperty dataSourceProperty = new DataSourceProperty();
      BeanUtils.copyProperties(dto, dataSourceProperty);
      DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
      DataSource dataSource = basicDataSourceCreator.createDataSource(dataSourceProperty);
      ds.addDataSource(dto.getPollName(), dataSource);
      return ds.getCurrentDataSources().keySet();
  }

  @PostMapping("/addJndi")
  @ApiOperation("添加JNDI数据源")
  public Set<String> addJndi(String pollName, String jndiName) {
      DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
      DataSource dataSource = jndiDataSourceCreator.createDataSource(jndiName);
      ds.addDataSource(pollName, dataSource);
      return ds.getCurrentDataSources().keySet();
  }

  @PostMapping("/addDruid")
  @ApiOperation("基础Druid数据源")
  public Set<String> addDruid(@Validated @RequestBody DataSourceDTO dto) {
      DataSourceProperty dataSourceProperty = new DataSourceProperty();
      BeanUtils.copyProperties(dto, dataSourceProperty);
      DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
      DataSource dataSource = druidDataSourceCreator.createDataSource(dataSourceProperty);
      ds.addDataSource(dto.getPollName(), dataSource);
      return ds.getCurrentDataSources().keySet();
  }

  @PostMapping("/addHikariCP")
  @ApiOperation("基础HikariCP数据源")
  public Set<String> addHikariCP(@Validated @RequestBody DataSourceDTO dto) {
      DataSourceProperty dataSourceProperty = new DataSourceProperty();
      BeanUtils.copyProperties(dto, dataSourceProperty);
      DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
      DataSource dataSource = hikariDataSourceCreator.createDataSource(dataSourceProperty);
      ds.addDataSource(dto.getPollName(), dataSource);
      return ds.getCurrentDataSources().keySet();
  }

  @DeleteMapping
  @ApiOperation("删除数据源")
  public String remove(String name) {
      DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
      ds.removeDataSource(name);
      return "删除成功";
  }
```
