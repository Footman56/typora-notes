# 基础准备

文档：https://easyexcel.opensource.alibaba.com/docs/current/

```xml
<easyexcel.version>3.1.1</easyexcel.version>
<dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>easyexcel</artifactId>
                <version>${easyexcel.version}</version>
</dependency>
```

Easy excel  是将表头和内容分开，属于不同的对象领域

```java
# data 是数据，writeSheet 属于表头
ExcelWriter write(Collection<?> data, WriteSheet writeSheet)
```

+ ExcelWriter  核心书写功能；专门负责**写操作**
+ WriteWorkbook：POI 中workbook 的概念；定义了整个excel文件的信息；支持向OutputStream、File 中输出

+ WriteSheet：POI中workSheet概念；定义了sheet中信息，记录了表头、自定义样式策略

+ head： 可以分成两种形式

  + class : 带注解的类

  + List<List<String>> ：自定义表头，用来控制是否合并表头：最外层表示列，最内层表示行

    ```
    excelHeader.add(new HeaderFieldDO("基础信息", "姓名"));
    excelHeader.add(new HeaderFieldDO("基础信息", "年龄"));
    excelHeader.add(new HeaderFieldDO("基础信息", "性别"));
    ```

    <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202404021704892.png" alt="image-20240402170418531" style="zoom:50%;" />

+ data: List<List<String>>   最外层表示行，最里层就是一行数据-并且有序

大致流程：

1. 创建ExcelWriter
2. 指定要写的WriteSheet
3. 指定要写的内容

```java
ExcelWriter excelWriter = EasyExcel.write(file).excelType(ExcelTypeEnum.XLSX)
                    .build();

WriteSheet writeSheet =  EasyExcel.writerSheet(fileName).head(heads)
                    .registerWriteHandler(new ExcelLockStrategy(lockedRowNumMap))
                    .build();

excelWriter.write(data, writeSheet);
excelWriter.finish();
```



# 写固定表格

## 一级表头

创建导出对象

@ExcelProperty 注解的作用是确定表头

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class UserEO {
    
    @ExcelProperty("姓名")
    private String employeeName;

    @ExcelProperty("年龄")
    private Integer age;

    @ExcelProperty("性别")
    private String sex;

    @ExcelProperty("分数")
    private Double score;

    @ExcelProperty("出生日期")
    @DateTimeFormat("yyyy-MM-dd hh:mm:ss")
    private Date dateBirth;
}
```

```java
/**
     * 输出流
     * @param outputStream
     */
    @Override
    public void downExcel(ServletOutputStream outputStream) {
        // 待输出的对象
        List<UserEO> userEOList = new ArrayList<>();
        UserEO userEO = UserEO.builder()
                .employeeName("lizhi")
                .age(24)
                .dateBirth(new Date())
                .sex("boy")
                .score(100D)
                .build();
        userEOList.add(userEO);

        // 要指定输出的对象，并且要指定sheet名称，内容的数据内容
        EasyExcelFactory.write(outputStream, UserEO.class).sheet("用户").doWrite(userEOList);
    }
```

controller 接口

```java
 @GetMapping("/download")
    public void download(HttpServletResponse response) {
        try {
            response.reset();
            response.setContentType("application/vnd.ms-excel");
        //Content-Disposition的作用：告知浏览器以何种方式显示响应返回的文件，用浏览器打开还是以附件的形式下载到本地保存
        //attachment表示以附件方式下载   inline表示在线打开   "Content-Disposition: inline; filename=文件名.mp3"
        //filename表示文件的默认名称，因为网络传输只支持URL编码的相关支付，因此需要将文件名URL编码后进行传输,前端收到后需要反编码才能获取到真正的名称
            response.setHeader("Content-disposition",
                    "attachment;filename=user_excel_" + System.currentTimeMillis() + ".xlsx");
            response.setCharacterEncoding("utf-8");
            // 这里URLEncoder.encode可以防止中文乱码 当然和easyexcel没有关系
            userService.downExcel(response.getOutputStream());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

## 多级表头

```java
 @Override
    public File downExcelDynamicState() {
        // 获取上传地址
        File file = null;
        try {
            // 表头
            List<HeaderFieldDO> excelHeader = new ArrayList<>();
            excelHeader.add(new HeaderFieldDO("基础信息", "姓名"));
            excelHeader.add(new HeaderFieldDO("基础信息", "年龄"));
            excelHeader.add(new HeaderFieldDO("基础信息", "性别"));
            List<List<String>> heads = new ArrayList<>(excelHeader.size());
            for (HeaderFieldDO headerFieldDO : excelHeader) {
                List<String> fieldNames = new ArrayList<>();
                fieldNames.add(headerFieldDO.getFiled());
                fieldNames.add(headerFieldDO.getFieldName());
                heads.add(fieldNames);
            }
          

           // 页面内容
            List<List<String>> exportContentDO = new ArrayList<>();
            List<String> stringArrayList = new ArrayList<>();
            stringArrayList.add("张三");
            stringArrayList.add("24");
            stringArrayList.add("男");
            exportContentDO.add(stringArrayList);
          
            List<String> stringArrayList1 = new ArrayList<>();
            stringArrayList1.add("李四");
            stringArrayList1.add("25");
            stringArrayList1.add("女");
            exportContentDO.add(stringArrayList1);

            Map<Integer, List<Integer>> lockedRowNumMap = new HashMap<>();
            lockedRowNumMap.put(1, Lists.newArrayList(2, 1));
            lockedRowNumMap.put(2, Lists.newArrayList(2));
            lockedRowNumMap.put(3, Lists.newArrayList(0));

            file = File.createTempFile("exportBatchTargetRegistrationTemp", ".xlsx");

            String fileName = "BATCH_TARGET_REGISTRATION_FILE_NAME";
            ExcelWriter excelWriter = EasyExcel.write(file).excelType(ExcelTypeEnum.XLSX)
                    .build();
          // 构造sheet
            WriteSheet writeSheet = EasyExcel.writerSheet(fileName).head(heads)
                    .registerWriteHandler(new ExcelLockStrategy(lockedRowNumMap))
                    .build();
          // 写内容到文件中
            excelWriter.write(exportContentDO, writeSheet);
            excelWriter.finish();
        } catch (Exception e) {
            log.error("exportPlanAssessDetail,导出异常", e);
        }
        return file;
    }
```

controller

```java
@GetMapping("/download1")
    public void download1(HttpServletResponse response) {
        try {
            response.reset();
            response.setContentType("application/vnd.ms-excel");
            response.setHeader("Content-disposition",
                    "attachment;filename=user_excel_" + System.currentTimeMillis() + ".xlsx");
            File file = userService.downExcelDynamicState();

            // 读取写好的文件，转成向响应输出流输出
            try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));) {
                byte[] buff = new byte[1024];
                OutputStream os = response.getOutputStream();
                int i = 0;
                while ((i = bis.read(buff)) != -1) {
                    os.write(buff, 0, i);
                    os.flush();
                }
            } catch (IOException e) {
                log.error("{}", e);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```



# 读固定表格

# 写动态表格

核心就在于 表头

```java
 ExcelWriter excelWriter = EasyExcel.write(file).excelType(ExcelTypeEnum.XLSX)
                    .build();

WriteSheet writeSheet =  EasyExcel.writerSheet(fileName).head(xxxxxxx)
                    .registerWriteHandler(new ExcelLockStrategy(lockedRowNumMap))
                    .build();

excelWriter.write(xxxxxx, writeSheet);
excelWriter.finish();
```

# 读动态表格

# 写样式

# 写合并单元格

重写策略方法

+ 时机：绘制完单元格

```java
/**
 * excel 单元格合并规则
 *  上下行数据相同 则合并单元格
 *@author peilizhi
 *@date 2024/4/2 17:49
 **/
@Data
public class ExcelRowMergeStrategy implements CellWriteHandler {


    /**
     * excel 中最大需要合并单元格的行号
     * 一般是数据最后一行
     */
    private int maxRow;

    public ExcelRowMergeStrategy() {
    }
    public ExcelRowMergeStrategy(int maxRow) {
        this.maxRow = maxRow;
    }
    
    @Override
    public void afterCellDispose(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, List<WriteCellData<?>> cellDataList, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {
        if (isHead) {
            // 不处理表头，只处理内容
            return;
        }
        Sheet sheet = writeSheetHolder.getSheet();
        // 当前行
        int curRowIndex = cell.getRowIndex();
        int curColIndex = cell.getColumnIndex();
        if (curRowIndex > maxRow) {
            // 超过最大行
            return;
        }
        // 获取当前行的数据
        Object currValue = getValue(cell);
        // 获取上一行数据
        Cell preCell = sheet.getRow(curRowIndex - 1).getCell(curColIndex);
        Object preValue = getValue(preCell);
        if (Objects.equals(currValue, preValue)) {
            // 内容相同,判断是否上一行是否已经合并了，如果合并了要修复单元格，否则要新增一条
            List<CellRangeAddress> mergedRegions = sheet.getMergedRegions();

            boolean isMerged = false;
            int index = -1;
            CellRangeAddress mergedCellRangeAddress = null;
            for (int i = 0; i < mergedRegions.size(); i++) {
                CellRangeAddress cellAddresses = mergedRegions.get(i);
                if (cellAddresses.isInRange(curRowIndex - 1, curColIndex)) {
                    // 要删除再插入
                    isMerged = true;
                    mergedCellRangeAddress = cellAddresses;
                    index = i;
                    break;
                }
            }

            if (isMerged) {
                // 如果有已经合并的数据，就删除，再次添加
                sheet.removeMergedRegion(index);
                CellRangeAddress cellRangeAddress = new CellRangeAddress(mergedCellRangeAddress.getFirstRow(), curRowIndex, mergedCellRangeAddress.getFirstColumn(), mergedCellRangeAddress.getLastColumn());
                sheet.addMergedRegionUnsafe(cellRangeAddress);
            } else {
                CellRangeAddress cellRangeAddress = new CellRangeAddress(curRowIndex - 1, curRowIndex, curColIndex, curColIndex);
                sheet.addMergedRegionUnsafe(cellRangeAddress);
            }
        }
    }

    /**
     * 获取单元格数据
     *
     * @param cell
     * @return
     */

    private Object getValue(Cell cell) {
        if (null == cell) {
            return null;
        }
        CellType cellType = cell.getCellType();
        switch (cellType) {
            case STRING:
                return cell.getStringCellValue();
            case NUMERIC:
                return cell.getNumericCellValue();
            case BOOLEAN:
                return cell.getBooleanCellValue();
            default:
                return null;
        }
    }
}
```

# 读合并单元格

