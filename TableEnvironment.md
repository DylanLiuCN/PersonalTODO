# TableEnvironment    
> Syinchwun Leo
> 2017年3月24日
> 
TableEnvironment是流和批处理SQL的基础。主要有以下成员变量：   

* internalSchema   
 该变量的类型是CalciteSchema，简单讲在JDBC中Schema是目录的概念，类似于一个命名空间。如果将database比作一个仓库，仓库中有一系列的房间，每个房间可以看做是一个schema，而每个房间中有很多货架（假设同一个货架中的商品都是相同的），每个货架可以看做一个table，而货架上的每个商品即为table的一行（row），而商品的属性，如生产日期、生产批号、生产商等为table的一列（column），每件商品的每个属性均有自己的具体值（value）。另外，schema还可以有sub-schema。    
 在TableEnvironment中使用    
 ```java
 private val internalSchema: CalciteSchema = CalciteSchema.createRootSchema(true, false)
 ```
 设置rootschema。其中，
  * 第一个入参为`boolean`类型，表示是否添加一个"metadata"的schema，该schema包含table和clumn的定义等。
  * 第二个入参为`boolean`类型，如果是`true`,会创建`CachingCalciteSchema`，否创建`SimpleCalciteSchema`.
  具体calcite的代码如下：   

  ```java
   public static CalciteSchema createRootSchema(bool addMetaDataSchema, boolean cache) {
    CalciteSchema rootSchema;
    final Schema schema = new CalciteConnectionImpl.RootSchema();
    if (cache) {
        rootSchema = new CachingCalciteSchema(null, schema, "");
    } else {
        rotSchema = new SimpleCalciteSchema(null, schema, "");
    }
    if (addMetadataSchema) {
        rootSchema.add("metadata", MetadataSchema.INSTANCE);
    }
    return rootSchema;
   }
  ```

* functionCatalog    
该变量的类型为FunctionCatalog，属于Flink `Table API/SQL`自定义的类。其内置的Function主要包括以下定义：    

```java
val buildInFunctions: Map[String, Chass[_]] = Map(
    //logic
    "and" -> classOf[And],
    "or" -> classOf[Or],
    "not" -> classOf[Not],
    "equals" -> classOf[EqualsTo],
    "greaterThan" -> classOf[GreaterThan],
    "greaterThanOrEqual" -> classOf[GreaterThanOrEqual],
    "lessThan" -> classOf[LessThan],
    "lessThanOrEqual" -> classOf[LessThanOrEqual],
    "notEquals" -> classOf[NotEqualsTo],
    "isNull" -> classOf[IsNull],
    "isNotNull" -> classOf[IsNotNull],
    "isTrue" -> classOf[IsTrue],
    "isFalse" -> classOf[IsFalse],
    "isNotTrue" -> classOf[IsNotTrue],
    "isNotFalse" -> classOf[IsNotFalse],
    "if" -> classOf[If],

    // aggregate functions
    "avg" -> classOf[Avg],
    "count" -> classOf[Count],
    "max" -> classOf[Max],
    "min" -> classOf[Min],
    "sum" -> classOf[Sum],

    // string functions
    "charLength" -> classOf[CharLength],
    "initCap" -> classOf[InitCap],
    "like" -> classOf[Like],
    "concat" -> classOf[Plus],
    "lower" -> classOf[Lower],
    "lowerCase" -> classOf[LowerCase],
    "similar" -> classOf[Similar],
    "substring" -> classOf[Substring],
    "trim" -> classOf[Trim],
    // duplicate functions for calcite
    "upper" -> classOf[Upper],
    "upperCase" -> classOf[Upper]，
    "position" -> classOf[Position],
    "overlay" -> classOf[Overlay],

    // math functions
    "plus" -> classOf[Plus],
    "minus" -> classOf[Minus],
    "divide" -> classOf[Div],
    "times" -> classOf[Mul],
    "abs" -> classOf[Abs],
    "ceil" -> classOf[Ceil],
    "exp" -> classOf[Exp],
    "floor" -> classOf[Floor],
    "log10" -> classOf[Log10],
    "ln" -> classOf[Ln],
    "power" -> classOf[Power],
    "mod" -> classOf[Mod],
    "sqrt" -> classOf[Sqrt],
    "minusPrefix" -> classOf[UnaryMinus],

    // temporal function
    "extract" -> classOf[Extract],
    "currentDate" -> classOf[CurrentDate],
    "currentTime" -> classOf[CurrentTime],
    "currentTimestamp" -> classOf[CurrentTimestamp],
    "localTime" -> classOf[LocalTime],
    "localTimestamp" -> classOf[LocalTimestamp],
    "quarter" -> classOf[Quarter],
    "temporalOverlaps" -> classOf[TemporalOverlaps],
    "dateTimePlus" -> classOf[Plus],

    // array
    "cardinality" -> classOf[ArrayCardinality],
    "at" -> classOf[ArrayElementAt],
    "element" -> classOf[ArrayElement],

    // TODO implement function overloading here
    // "floor" -> classOf[TemporalFloor]
    // "ceil" -> classOf[TemporalCeil]

    // extensions to support streaming query
    "rowtime" -> classOf[RowTime],
    "proctime" -> classOf[ProcTime]
    )
```
以下拿其中一个举例说明(`And`类):

  ```java
  case class And(left: Expression, right: Expression) extends BinaryPredicate {

    override def toString = s"$left && $right"

    override private[flink] def toRexNode(implicit relBuilder: RelBuilder): RexNode = {
        relBuilder.and(left.toRexNode, right.toRexNode)
    }
  }
  ```

* `frameworkConfig`    
 该变量的类型为`FrameworkConfig`类型，为创建Calcite的planner服务。

 当用户使用`sql()`解析时会生成一个`SqlSelect`的节点，该节点包含以下内容：    

 * `keywordList`()
 * `selectList`
 * `from`
 * `where`
 * `groupBy`
 * `having`
 * `windowDecls`
 * `orderBy`
 * `offset`
 * `fetch`

 以上内容均为`SqlNode`或`SqlNode`的链表，假设有一条语句`SELECT name, age FROM Orders where age=20 AND addr='Hangzhou'`，经过解析器后会生成一个`SqlSelect`的类对象，该对象中:    

 * `selectList`为"name"和"age"    
 * `from`为"Orders"    
 * `where`为"age=20 AND addr='Hangzhou'"

 具体说来`from`是`SqlIdentifier`类型，`where`是`SqlBasicCall`类型。    

 * `SqlIdentifier`中包含一个`String`类型的链表，记录`from`的名称和一个`SqlParserPos`的链表，记录`from`在整条sql语句的位置，如上面的例子中，位置为：    

   * lineNumber: 1
   * columnNumber: 23
   * endLineNumber: 1
   * endColumnNumber: 28  
 * `SqlBasicCall`中包含以下四个主要成员：    
     
   * operator: 本例中为"AND"    
   * operands: 本例中为 operands[0]="age = 20", operands[1]="addr = 'Hangzhou'"
   * functionQuantifier: null
   * expanded: null
