---
title: ʹ��Jxlsʵ��excel����
tags: springboot,Jxls,JPA
grammar_zjwJava: true
---

 - ����springboot��Ŀ
    - ��������
      -  `<dependency>
            <groupId>org.jxls</groupId>
            <artifactId>jxls</artifactId>
            <version>2.6.0</version>
        </dependency>`
	  - `<dependency>
            <groupId>net.sf.jxls</groupId>
            <artifactId>jxls-reader</artifactId>
            <version>1.0.6</version>
        </dependency>`

 - ����ʵ����
   

	  ``` 
	@Data
	@Entity
	@Table(name = "sys_users")
	public class SysUsers {

		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		@Column(name = "id")
		private Integer id;

		@Column(name = "name")
		private String name;

		@Column(name = "age")
		private Integer age;

		@Column(name = "addr")
		private String addr;

		@Column(name = "address_id")
		private Long address_id;
	```
	  �˴�ʹ��lombok��@Dataע������getter()��setter()������
  - ��д����ӿ�

	``` 
	@GetMapping("/get/user/export")
		public void getUserExport(HttpServletResponse response){
			List<SysUsers> sysUsersList = sysUserJxlsService.list();
			Context context = new Context();
			context.putVar("data",sysUsersList);
			response.setHeader("content-Type", "application/vnd.ms-excel");
			String fileName = "�û���Ϣ"  + DateTimeHelper.formatToString(new Date(),"yyyy_MM_dd") + ".xlsx";
			try {
				//response.setHeader("Content-Disposition", "attachment:filename=" + URLEncoder.encode(fileName, "utf-8"));
				response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(fileName, "utf-8"));
				OutputStream outputStream = response.getOutputStream();
				InputStream inputStream = getClass().getClassLoader().getResourceAsStream("excel/templateJxls_user.xlsx");
				JxlsHelper.getInstance().processTemplate(inputStream,outputStream,context);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	```
	sysUserJxlsService.list();�ǵ���service���ȡ�����б������˴�����˵����Ȼ��õ�List<SysUsers>���飻
	����Context���󣬵���.putVar("data",sysUsersList);���������鴫�룬��data��ֵ��Ҫע�⣬�����Ĵ���excelģ���ʱ����õ���
	`excel/templateJxls_user.xlsx`����ǻ�ȡ��Ŀ�ж�Ӧ��Excelģ������·����
- ����Excelģ��
	![enter description here](./images/fi.png)
	���ȴ�����ʵ�������Զ�Ӧ�ı��⣬�ڹ�����ĵ�һ����Ԫ�������ע`jx:area(lastCell="E3")`,���������A1��E3��
	![enter description here](./images/se.png)
	��������ݿ�ʼ�ĵ�һ����Ԫ�������ע`jx:each(items="data" var="item" lastCell="E3")`�����С�data������֮ǰContext����������ʱ���ֵ��item����ѭ��ʱ��Ԫ�أ������� item.�ֶ��� �õ���Ҫ��ʵ������������һ�£�E3�����������
- ���������࣬������Ŀ������[http://localhost:8080/get/user/export](http://localhost:8080/get/user/export)��������