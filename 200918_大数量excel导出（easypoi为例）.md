##### 大数量excel导出（easypoi为例）

- 导出工具类方法

  ```java
  /**
  * 使用模板导出excel到本地
  *
  * @param fileName 导出文件名
  * @param params   模板参数
  * @param data     导出数据
  * @param path     存于本地的路径
  */
  public static void exportTemplateToLocal(String fileName, TemplateExportParams params, Object data, String path) {
      Map map;
      if (data instanceof Map) {
          map = (Map) data;
      } else {
          String json = JsonUtils.object2Json(data);
          map = JsonUtils.json2Object(json, Map.class);
      }
      FileOutputStream out = null;
      Workbook workbook = null;
      File file = new File(path);
      if (!file.exists()) {
          file.mkdirs();
      }
      try {
          out = new FileOutputStream(new File(path + fileName));
          workbook = cn.afterturn.easypoi.excel.ExcelExportUtil.exportExcel(params, map);
          workbook.write(out);
      } catch (Exception e) {
          e.printStackTrace();
      } finally {
          try {
              if (out != null) {
                  out.close();
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
          try {
              if (workbook != null) {
                  workbook.close();
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  
  /**
  * 删除指定目录下的文件
  * @param workspaceRootPath
  */
  public static void clearFiles(String workspaceRootPath) {
      File file = new File(workspaceRootPath);
      if (file.exists()) {
          deleteFile(file);
      }
  }
  
  private static void deleteFile(File file) {
      if (file.isDirectory()) {
          File[] files = file.listFiles();
          if (files != null) {
              for (File file1 : files) {
                  file1.delete();
              }
          }
      }
  }
  ```

- 压缩工具类

  ```java
  public class CompressUtil {
  
      static final int BUFFER = 8192;
  
      /**
       * 压缩文件夹到目标目录下
       * @param srcPath
       * @param dstPath
       * @throws IOException
       */
      public static void compress(String srcPath, String dstPath) throws IOException {
          File srcFile = new File(srcPath);
          File dstFile = new File(dstPath);
          if (!srcFile.exists()) {
              throw new FileNotFoundException(srcPath + "不存在！");
          }
          FileOutputStream out = null;
          ZipOutputStream zipOut = null;
          try {
              out = new FileOutputStream(dstFile);
              CheckedOutputStream cos = new CheckedOutputStream(out, new CRC32());
              zipOut = new ZipOutputStream(cos);
              String baseDir = "";
              compress(srcFile, zipOut, baseDir);
          } finally {
              if (null != zipOut) {
                  zipOut.close();
                  out = null;
              }
              if (null != out) {
                  out.close();
              }
          }
      }
  
      private static void compress(File file, ZipOutputStream zipOut, String baseDir) throws IOException {
          if (file.isDirectory()) {
              compressDirectory(file, zipOut, baseDir);
          } else {
              compressFile(file, zipOut, baseDir);
          }
      }
  
      /**
       * 压缩一个目录
       */
      private static void compressDirectory(File dir, ZipOutputStream zipOut, String baseDir) throws IOException {
          File[] files = dir.listFiles();
          for (int i = 0; i < files.length; i++) {
              compress(files[i], zipOut, baseDir + dir.getName() + "/");
          }
      }
  
      /**
       * 压缩一个文件
       */
      private static void compressFile(File file, ZipOutputStream zipOut, String baseDir) throws IOException {
          if (!file.exists()) {
              return;
          }
          BufferedInputStream bis = null;
          try {
              bis = new BufferedInputStream(new FileInputStream(file));
              ZipEntry entry = new ZipEntry(baseDir + file.getName());
              zipOut.putNextEntry(entry);
              int count;
              byte data[] = new byte[BUFFER];
              while ((count = bis.read(data, 0, BUFFER)) != -1) {
                  zipOut.write(data, 0, count);
              }
          } finally {
              if (null != bis) {
                  bis.close();
              }
          }
      }
  
      public static void decompress(String zipFile, String dstPath) throws IOException {
          File pathFile = new File(dstPath);
          if (!pathFile.exists()) {
              pathFile.mkdirs();
          }
          ZipFile zip = new ZipFile(zipFile);
          for (Enumeration entries = zip.entries(); entries.hasMoreElements(); ) {
              ZipEntry entry = (ZipEntry) entries.nextElement();
              String zipEntryName = entry.getName();
              InputStream in = null;
              OutputStream out = null;
              try {
                  in = zip.getInputStream(entry);
                  String outPath = (dstPath + "/" + zipEntryName).replaceAll("\\*", "/");
                  ;
                  //判断路径是否存在,不存在则创建文件路径
                  File file = new File(outPath.substring(0, outPath.lastIndexOf('/')));
                  if (!file.exists()) {
                      file.mkdirs();
                  }
                  //判断文件全路径是否为文件夹,如果是上面已经上传,不需要解压
                  if (new File(outPath).isDirectory()) {
                      continue;
                  }
  
                  out = new FileOutputStream(outPath);
                  byte[] buf1 = new byte[1024];
                  int len;
                  while ((len = in.read(buf1)) > 0) {
                      out.write(buf1, 0, len);
                  }
              } finally {
                  if (null != in) {
                      in.close();
                  }
  
                  if (null != out) {
                      out.close();
                  }
              }
          }
          zip.close();
      }
  
      /**
       * 将目标文件响应给浏览器
       * @param downloadFilePath
       * @param fileName
       * @param response
       */
      public static void downloadFile(String downloadFilePath, String fileName, HttpServletResponse response) {
          //String downloadFilePath = "/root/fileSavePath/";//被下载的文件在服务器中的路径,
          //String fileName = "demo.xml";//被下载文件的名称
          File file = new File(downloadFilePath);
          if (file.exists()) {
              response.setContentType("application/force-download");// 设置强制下载不打开
              response.addHeader("Content-Disposition", "attachment;fileName=" + fileName);
              byte[] buffer = new byte[1024];
              FileInputStream fis = null;
              BufferedInputStream bis = null;
              try {
                  fis = new FileInputStream(file);
                  bis = new BufferedInputStream(fis);
                  OutputStream outputStream = response.getOutputStream();
                  int i = bis.read(buffer);
                  while (i != -1) {
                      outputStream.write(buffer, 0, i);
                      i = bis.read(buffer);
                  }
              } catch (Exception e) {
                  e.printStackTrace();
              } finally {
                  if (bis != null) {
                      try {
                          bis.close();
                      } catch (IOException e) {
                          e.printStackTrace();
                      }
                  }
                  if (fis != null) {
                      try {
                          fis.close();
                      } catch (IOException e) {
                          e.printStackTrace();
                      }
                  }
              }
          }
      }
  }
  ```

- 示例（每次读取1W条存在本地目录下，再压缩成一个文件响应给浏览器）

  ```java
  private final static String PATH = "/excels/temp/"; // 存放excel路径
  private final static String COMPRESS_PATH = "/excels/"; //打包zip存放路径
  TemplateExportParams params = new TemplateExportParams("excel_template/template.xlsx"); //获取导出模板
  Map<String, Object> map = new HashMap<>(); // 创建导出数据map
  Integer total = mapper.getTotal(); //获取总数
  Integer result=total/10000; //获取循环次数
  Integer remainder=total%10000; //判断是否刚好整除
  if(remainder != 0){
  	result =result + 1; //不整除则加一次循环
  }
  for(int i=1;i<=result ;i++){
  	PageHelper.startPage(i,10000);
  	List<Entity> list = mapper.getList(); //分页每次查询出1W条
  	map.put("list", list); // 放入数据
  	// 将该1W数据新建excel存放在本地
  	ExcelUtils.exportTemplateToLocal("export.xlsx", params, map, PATH);
  }
  // 压缩文件,将存放excels目录打包
  CompressUtil.compress(PATH, COMPRESS_PATH + "desc.zip");
  // 响应文件,将打包的zip文件响应给浏览器
  CompressUtil.downloadFile(COMPRESS_PATH + "desc.zip", "desc.zip", response);
  // 删除文件,将文件删除
  CompressUtil.clearFiles(PATH);
  ```

  思想：每次读1W条存入一个excel到本地，然后将该文件夹打包，再响应给浏览器，最后删除文件