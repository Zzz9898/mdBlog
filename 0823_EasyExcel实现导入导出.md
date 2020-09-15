```
title: EasyExcel实现导入导出
tags: EasyExcel
grammer_zjwJava: 
```

- 引入依赖

  - ```
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>easyexcel</artifactId>
        <version>1.1.2-beta5</version>
    </dependency>
    ```

- 创建对应实体类

  - ```java
    public class TestEntity extends BaseRowModel {
    
        @ExcelProperty(value = {"Test", "姓名"}, index = 0)
        private String name;
    
        @ExcelProperty(value = {"Test", "年龄"}, index = 1)
        private int age;
    
        @ExcelProperty(value = {"Test", "邮箱"}, index = 2)
        private String email;
    }
    ```

    实体需要继承BaseRowModel，@ExcelProperty指定每个字段的列名称，以及下标位置。

- 导出

  - ```java
    @GetMapping("/get/data")
    public void get(HttpServletResponse response) throws IOException {
            String fileName = "测试TestEasyExcel";
            ServletOutputStream out = response.getOutputStream();
            ExcelWriter writer = new ExcelWriter(out, ExcelTypeEnum.XLSX, true);
            Sheet sheet = new Sheet(1, 0, TestEntity.class, "sheet one", null);
            writer.write(getData(), sheet);
            response.setContentType("multipart/form-data");
            response.setCharacterEncoding("utf-8");
            response.setHeader("Content-disposition", "attachment;filename=" + new String( fileName.getBytes("gb2312"), "ISO8859-1" ) + ".xlsx");
            writer.finish();
            out.flush();
        }
    
    private List<TestEntity> getData() {
            List<TestEntity> list = new ArrayList<>();
            TestEntity zs = new TestEntity("zs", 21, "zs@sx.com");
            TestEntity li = new TestEntity("li", 19, "li@sx.com");
            TestEntity wr = new TestEntity("wr", 20, "wr@sx.com");
            list.add(zs);
            list.add(li);
            list.add(wr);
            return list;
        }
    ```

    其中getData();方法模拟数据获取，其中response需要设置响应资源的MIME类型、比编码以及响应头。

  - 请求接口，导出Excel

    ![](.\response.png)

- 导入

  - 引入工具类

    ```java
    public class EasyExcelUtil {
    
        /**
         * 读取某个 sheet 的 Excel
         *
         * @param excel    文件
         * @param rowModel 实体类映射，继承 BaseRowModel 类
         * @return Excel 数据 list
         */
        public static List<Object> readExcel(MultipartFile excel, BaseRowModel rowModel) throws IOException {
            return readExcel(excel, rowModel, 1, 1);
        }
    
        /**
         * 读取某个 sheet 的 Excel
         * @param excel       文件
         * @param rowModel    实体类映射，继承 BaseRowModel 类
         * @param sheetNo     sheet 的序号 从1开始
         * @param headLineNum 表头行数，默认为1
         * @return Excel 数据 list
         */
        public static List<Object> readExcel(MultipartFile excel, BaseRowModel rowModel, int sheetNo, int headLineNum) throws IOException {
            ExcelListener excelListener = new ExcelListener();
            ExcelReader reader = getReader(excel, excelListener);
            if (reader == null) {
                return null;
            }
            reader.read(new Sheet(sheetNo, headLineNum, rowModel.getClass()));
            return excelListener.getDatas();
        }
    
        /**
         * 读取指定sheetName的Excel(多个 sheet)
         * @param excel    文件
         * @param rowModel 实体类映射，继承 BaseRowModel 类
         * @return Excel 数据 list
         * @throws IOException
         */
        public static List<Object> readExcel(MultipartFile excel, BaseRowModel rowModel,String sheetName) throws IOException {
            ExcelListener excelListener = new ExcelListener();
            ExcelReader reader = getReader(excel, excelListener);
            if (reader == null) {
                return null;
            }
            for (Sheet sheet : reader.getSheets()) {
                if (rowModel != null) {
                    sheet.setClazz(rowModel.getClass());
                }
                //读取指定名称的sheet
                if(sheet.getSheetName().contains(sheetName)){
                    reader.read(sheet);
                    break;
                }
            }
            return excelListener.getDatas();
        }
    
        /**
         * 返回 ExcelReader
         * @param excel 需要解析的 Excel 文件
         * @param excelListener new ExcelListener()
         * @throws IOException
         */
        private static ExcelReader getReader(MultipartFile excel,ExcelListener excelListener) throws IOException {
            String filename = excel.getOriginalFilename();
            if(filename != null && (filename.toLowerCase().endsWith(".xls") || filename.toLowerCase().endsWith(".xlsx"))){
                InputStream is = new BufferedInputStream(excel.getInputStream());
                return new ExcelReader(is, null, excelListener, false);
            }else{
                return null;
            }
        }
    
    }
    ```

  - 创建监听器

    ```java
    public class ExcelListener extends AnalysisEventListener {
        //可以通过实例获取该值
        private List<Object> datas = new ArrayList<Object>();
    
        @Override
        public void invoke(Object o, AnalysisContext analysisContext) {
            datas.add(o);//数据存储到list，供批量处理，或后续自己业务逻辑处理。
        }
    
        List<Object> getDatas() {
            return datas;
        }
    
        public void setDatas(List<Object> datas) {
            this.datas = datas;
        }
    
        public void doAfterAllAnalysed(AnalysisContext analysisContext) {
            // datas.clear();//解析结束销毁不用的资源
        }
    }
    ```

  - 方法

    ```java
    @PostMapping("/post/data")
        public void post(@RequestParam("file") MultipartFile file) {
            try {
                List<Object> objects;
                objects = EasyExcelUtil.readExcel(file, new TestEntity(), 1, 2);
                if (objects == null){
                    return;
                }
                for (Object object : objects) {
                    TestEntity testEntity = (TestEntity) object;
                    System.out.println(testEntity.toString());
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    ```

    readExcel();方法中，读取第一个sheet，不读两行标题，返回List<Object>数据后，转换为TestEntity，就是所需要的数据了。

  - 打印结果

    ![](.\print.png)