# Apace poi

引入pom

```xml
 				<!--xls(03)-->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>4.1.2</version>
        </dependency>
        <!--xlsx(07)-->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>4.1.2</version>
        </dependency>
```



## HSSF

这种方式导出的文件格式为office 2003专用格式，即.xls

 优点是导出数据速度快，但是最多65536行数据，多于65536 会报错

写入excel 

```java
/**
     * 使用HSSF导出excel
     * 这种方式导出的文件格式为office 2003专用格式，即.xls
     * 优点是导出数据速度快，但是最多65536行数据
     */
    public static void exportFileHSSF() {
        // 1、创建excel工作薄 HSSF 形式
        Workbook workbook = new HSSFWorkbook();
        // 2、创建 表
        Sheet sheet = workbook.createSheet("第一表");
        // 3、写入数据 (如果数据大于65536的话,会执行失败)
        for (int i = 0; i < 65536; i++) {
            // 4、创建行, 行下标从0开始
            Row row = sheet.createRow(i);
            for (int j = 0; j < 10; j++) {
                // 5、创建列
                Cell cell = row.createCell(j);
                // 6、设置值
                cell.setCellValue(i * j);
            }
        }
        // 写入文件
        try (FileOutputStream fileOutputStream = new FileOutputStream(FILE_PATH + "HSSF.xls")) {
            // 写入到文件中
            workbook.write(fileOutputStream);
        } catch (Exception e) {
            System.out.println(e);
        }
    }
```

读取excel

```java
   /**
     * 导入excel
     */
    public static void importFileHSSF() throws Exception {
        // 获取文件
        try (FileInputStream fileInputStream = new FileInputStream(FILE_PATH + "HSSF.xls")) {
            // 读取文件到工作薄
            Workbook workbook = new HSSFWorkbook(fileInputStream);
            // 获取任意表格
            Sheet sheet = workbook.getSheetAt(0);

            // 获取所有行
            Iterator<Row> rowIterator = sheet.rowIterator();
            while (rowIterator.hasNext()) {
                Row row = rowIterator.next();
                // 获取所有列
                Iterator<Cell> cellIterator = row.cellIterator();
                while (cellIterator.hasNext()) {
                    Cell cell = cellIterator.next();
                    // 获取单元格类型
                    // 根据类型获取对应的单元格值
                    CellType cellType = cell.getCellType();
                    switch (cellType) {
                        case _NONE:
                            System.out.println("未知类型");
                            break;
                        case BLANK:
                            System.out.println("空单元格");
                            break;
                        case STRING:
                            System.out.println("cellType[String]: " + cell.getStringCellValue());
                            break;
                        case BOOLEAN:
                            System.out.println("cellType[boolean]: " + cell.getBooleanCellValue());
                            break;
                        case FORMULA:
                            System.out.println("cellType[formula]: " + cell.getCellFormula());
                            break;
                        case NUMERIC:
                            // 是时间格式
                            if (DateUtil.isCellDateFormatted(cell)) {
                                System.out.println("cellType[date]: " + cell.getDateCellValue().toString());
                            } else {
                                // 不是时间格式
                                System.out.println("cellType[numeric]: " + cell.getNumericCellValue());
                            }
                            break;
                        case ERROR:
                            System.out.print("[数据类型错误]");
                            break;
                    }
                }
            }
        } catch (Exception e) {
            throw new Exception(e);
        }
    }
```

解析后得到的data 是List<Map<String,String>> 形式的，无法与业务对象绑定上，此时需要将 Map 转换成Bean对象的

使用Apache 的beanUtils 能够实现 bean 与map 之间的转换

```xml

        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>1.9.3</version>
        </dependency>
```

```java
 Demo demo = new Demo();
        demo.setAge(12);
        demo.setId(1111L);
        demo.setName("xiapo");

        // bean 2 map
        Map<String, String> stringMap = BeanUtils.describe(demo);
        System.out.println(JsonUtil.toJson(stringMap));

        Demo demo2 = new Demo();
        // map 2 bean 结果在第一个参数
        BeanUtils.populate(demo2, stringMap);
        System.out.println("demo2 = " + demo2);
```





## XSSF

XSSF方式支持大批量数据导出，所有的数据先写入内存再导出，容易出现内存溢出

导入行数无限制

```java
 /**
     * XSSF方式支持大批量数据导出，所有的数据先写入内存再导出，容易出现内存溢出
     */
    public static void exportFileXSSF() {
        // 1、创建excel工作薄 XSSF 形式
        Workbook workbook = new XSSFWorkbook();
        // 2、创建 表
        Sheet sheet = workbook.createSheet("第一表");
        // 3、写入数据 (如果数据大于65536的话,会执行失败)
        for (int i = 0; i < 65538; i++) {
            // 4、创建行, 行下标从0开始
            Row row = sheet.createRow(i);
            for (int j = 0; j < 10; j++) {
                // 5、创建列
                Cell cell = row.createCell(j);
                // 6、设置值
                cell.setCellValue(i * j);
            }
        }
        // 写入文件
        try (FileOutputStream fileOutputStream = new FileOutputStream(FILE_PATH + "XSSF.xlsx")) {
            // 写入到文件中
            workbook.write(fileOutputStream);
        } catch (Exception e) {
            System.out.println(e);
        }
    }
```

## SXSSF

SXSSF方式是XSSF的拓展,避免内存溢出

```java
/**
     * SXSSF方式是XSSF的拓展,避免内存溢出
     * 但是运行比较慢
     */
    public static void exportFileSXSSF() {
        // 1、创建excel工作薄 XSSF 形式
        Workbook workbook = new SXSSFWorkbook();
        // 2、创建 表
        Sheet sheet = workbook.createSheet("第一表");
        // 3、写入数据 (如果数据大于65536的话,会执行失败)
        for (int i = 0; i < 65538; i++) {
            // 4、创建行, 行下标从0开始
            Row row = sheet.createRow(i);
            for (int j = 0; j < 10; j++) {
                // 5、创建列
                Cell cell = row.createCell(j);
                // 6、设置值
                cell.setCellValue(i * j);
            }
        }
        // 写入文件
        try (FileOutputStream fileOutputStream = new FileOutputStream(FILE_PATH + "SXSSF.xlsx")) {
            // 写入到文件中
            workbook.write(fileOutputStream);
        } catch (Exception e) {
            System.out.println(e);
        }
    }
```

性能比较

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220424170103085.png" alt="image-20220424170103085" style="zoom:50%;" />

**其实HSSF、XSSF、SXSSF这几种读取的差别不大，不同点在于Workbook 的定义方式**

## 含表头

```java
/**
     * 创建Workbook
     *
     * @param headers 表头
     * @param data    List :行, Map<String, String> :key-表头，value-单元格的值
     */
    public static Workbook createWork(List<String> headers, List<Map<String, String>> data) {
        Workbook workbook = new XSSFWorkbook();
        // 安全创建sheet名称
        String safeSheetName = WorkbookUtil.createSafeSheetName("第一页");
        Sheet sheet = workbook.createSheet(safeSheetName);

        int rowNum = 0;
        Row row = sheet.createRow(rowNum);
        // 创建表头
        buildHeadRow(row, headers);
        // 下一行
        rowNum++;
        for (Map<String, String> cellMap : data) {
            Row dataRow = sheet.createRow(rowNum);
            // 保证顺序与表头顺序相同
            for (int i = 0; i < headers.size(); i++) {
                String key = headers.get(i);
                // 数据中没有表头的行就跳过
                if (StringUtil.isBlank(key)) {
                    continue;
                }
                String cellValue = cellMap.get(headers.get(i));
                Cell cell = dataRow.createCell(i);
                cell.setCellValue(cellValue);
            }
            //行数+1
            rowNum++;
        }
        return workbook;
    }

    /**
     * 创建表头
     *
     * @param row     行
     * @param headers 表头数据
     */
    public static void buildHeadRow(Row row, List<String> headers) {
        for (int i = 0; i < headers.size(); i++) {
            Cell cell = row.createCell(i);
            cell.setCellValue(headers.get(i));
        }
    }
```



## 含表头

```java
/**
     * 解析excel
     *
     * @param inputStream 文件流
     * @return headers、data
     */
    public static ParsalExcelDO parseExcel(InputStream inputStream) {
        ParsalExcelDO parsalExcelDO = new ParsalExcelDO();
        // 读取文件成 工作薄
        try (Workbook workbook = new XSSFWorkbook(inputStream)) {
            Sheet sheet = workbook.getSheetAt(0);

            Iterator<Row> rowIterator = sheet.rowIterator();

            // 获取表头
            Row headRow = rowIterator.next();
            List<String> headers = new ArrayList<>();
            final Iterator<Cell> cellIterator = headRow.cellIterator();
            while (cellIterator.hasNext()) {
                Cell cell = cellIterator.next();
                headers.add(cell.getStringCellValue());
            }
            parsalExcelDO.setHeaders(headers);

            List<Map<String, String>> data = new ArrayList<>();
            // 遍历后续节点
            while (rowIterator.hasNext()) {
                Row dataRow = rowIterator.next();
                Map<String, String> cellMap = new HashMap<>();

                // 每行的单元格
                for (int i = 0; i < headers.size(); i++) {
                    Cell cell = dataRow.getCell(i);
                    cellMap.put(headers.get(i), cell.getStringCellValue());
                }
                data.add(cellMap);
            }
            parsalExcelDO.setData(data);
        } catch (Exception ignored) {
        }
        return parsalExcelDO;
    }
```



# Easy POI

通过excel 与bean对象进行绑定

```xml
<dependencies>
    <dependency>
        <groupId>cn.afterturn</groupId>
        <artifactId>easypoi-base</artifactId>
        <version>4.1.0</version>
    </dependency>
    <dependency>
        <groupId>cn.afterturn</groupId>
        <artifactId>easypoi-web</artifactId>
        <version>4.1.0</version>
    </dependency>
    <dependency>
        <groupId>cn.afterturn</groupId>
        <artifactId>easypoi-annotation</artifactId>
        <version>4.1.0</version>
    </dependency>
</dependencies>
```

采用注解来定义bean

```java
@Data
@Builder
public class UserEO {

    @Excel(name = "姓名")
    private String name;

    @Excel(name = "年龄")
    private Integer old;

    @Excel(name = "操作时间", format = "yyyy-MM-dd HH:mm:ss", width = 20.0)
    private Date time;
}
```

## 导出

```java
 // 创建对象集合
        List<UserEO> userEOS = new ArrayList<>();
        userEOS.add(UserEO.builder()
                .name("小红")
                .old(12)
                .time(new Date())
                .build());

        userEOS.add(UserEO.builder()
                .name("小里")
                .old(20)
                .time(new Date())
                .build());

        // ExportParams 第一个参数表头，第二个参数 sheet 名称
        Workbook workbook = ExcelExportUtil.exportExcel(new ExportParams("xiaox", "hhh"), UserEO.class, userEOS);
        // 写入文件
        try (FileOutputStream fileOutputStream = new FileOutputStream(FILE_PATH + "easypoi.xlsx")) {
            // 写入到文件中
            workbook.write(fileOutputStream);
        } catch (Exception e) {
            System.out.println(e);
        }
```

## 导入

```java

        // 采用默认设置即可，有标题、表头
        ImportParams importParams = new ImportParams();
        // 标题行数，有就是1，没有就是0
        importParams.setTitleRows(1);
        // 表头行数，有就是1，没有就是0
        importParams.setHeadRows(1);
        // 获取文件
        try (FileInputStream fileInputStream = new FileInputStream(FILE_PATH + "easypoi.xlsx")) {
            // 读取文件 到bean对象
            List<UserEO> objectList = ExcelImportUtil.importExcel(fileInputStream, UserEO.class, importParams);
            System.out.println("JsonUtil.toJson(objectList) = " + JsonUtil.toJson(objectList));

        } catch (Exception e) {
            System.out.println(e);
        }
```

这种导入是能够明确excel 与bean对象有对应关系，如果没有这种关系的话，导入、导出就只能使用 List<Map<String,String>>  这种形式。

并且ExcelExportUtil 的工具类进行导出的时候必有标题属性

```java
// 自定义导出
/**
*entity 导出配置
* entityList 表头
* dataSet 数据
*/
public static Workbook exportExcel(ExportParams entity, List<ExcelExportEntity> entityList,
                                       Collection<?> dataSet) {
        Workbook workbook = getWorkbook(entity.getType(), dataSet.size());
        ;
        new ExcelExportService().createSheetForMap(workbook, entity, entityList, dataSet);
        return workbook;
    }




//封装表头
    List<ExcelExportEntity> entityList = new ArrayList<ExcelExportEntity>();
    entityList.add(new ExcelExportEntity("姓名", "name"));
    entityList.add(new ExcelExportEntity("年龄", "age"));
    ExcelExportEntity entityTime = new ExcelExportEntity("操作时间", "time");
    entityTime.setFormat("yyyy-MM-dd HH:mm:ss");
    entityTime.setWidth(20.0);
    entityList.add(entityTime);
    //封装数据体
    List<Map<String, Object>> dataList = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        Map<String, Object> userEntityMap = new HashMap<>();
        userEntityMap.put("name", "张三" + i);
        userEntityMap.put("age", 20 + i);
        userEntityMap.put("time", new Date(System.currentTimeMillis() + i));
        dataList.add(userEntityMap);
    }
    //生成excel文档
    Workbook workbook = ExcelExportUtil.exportExcel(new ExportParams("学生","用户信息"), entityList, dataList);
```

```java
// 自定义导入
 ImportParams params = new ImportParams();
    params.setTitleRows(1);
    params.setHeadRows(1);
    long start = new Date().getTime();
    List<Map<String, Object>> list = ExcelImportUtil.importExcel(new File("/Users/panzhi/Documents/easypoi-user2.xls"),
            Map.class, params);



 public static <T> List<T> importExcel(File file, Class<?> pojoClass, ImportParams params) {
        FileInputStream in = null;
        try {
            in = new FileInputStream(file);
            return new ExcelImportService().importExcelByIs(in, pojoClass, params, false).getList();
        } catch (ExcelImportException e) {
            throw new ExcelImportException(e.getType(), e);
        } catch (Exception e) {
            LOGGER.error(e.getMessage(), e);
            throw new ExcelImportException(e.getMessage(), e);
        } finally {
            IOUtils.closeQuietly(in);
        }
    }
```
