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
	
 - �༭Excel����ģ��
   ![enter description here](./images/th.png)
   
 - ����Excelģ��༭xml�ļ�
	``` 
	<?xml version="1.0" encoding="UTF-8"?>
	<workbook>
		<worksheet name="Sheet1">
			<section startRow="0" endRow="1"/>
			<loop startRow="2" endRow="2" items="sysUsersList" var="sysUsers" varType="com.zjw.entity.SysUsers">
				<section startRow="1" endRow="1">
					<mapping row="1" col="0">sysUsers.id</mapping>
					<mapping row="1" col="1">sysUsers.name</mapping>
					<mapping row="1" col="2">sysUsers.age</mapping>
					<mapping row="1" col="3">sysUsers.addr</mapping>
					<mapping row="1" col="4">sysUsers.address_id</mapping>
				</section>
				<loopbreakcondition>
					<rowcheck offset="0">
						<cellcheck offset="0"></cellcheck>
					</rowcheck>
				</loopbreakcondition>
			</loop>
		</worksheet>
	</workbook>
	```
	��һ��name�������ѡ��Excel����һ������������ƣ�
	section�о��Ǳ���ķ�Χ��ռ�˶����У��ڽ�������ʱ�����Ե����ҵ�Excel�еı�����ռ�˵�һ�У������Ǵӵ�0��ʼ��1������
	loop�о��Ǵ�����Ҫѭ��ͳ�������ˣ��ҵ������Ǵӵ�3�п�ʼ������startRow=��2����������һ��һ��ѭ���ģ�����endRow��һ���ͺ��ˣ�
	items�е�ֵ��Ҫע�⣬��֮���д����ʱ�õ���Ҳ�������ֵ��ʵ����ļ��ϣ�
	var����ѭ��ʱ���ݵ����ƣ�varType���Ƕ�Ӧʵ��������·����
	loop�е�section���Ǵ�ѭ���п�ʼ��Χ������1�ͺ��ˣ�
	mapping����Excel��ÿһ��ÿһ����Ԫ����ʵ�������ֶεĶ�Ӧ����������ʵ���������Զ�Ӧ���˿����ˡ�
	loopbreakcondition�о���ѭ���������жϣ���Ҫ����ʲô�������������ã��˴�����������ֵ�ͽ�����
	
- ��д���սӿ�
	``` 
	@PostMapping("/post/user/import")
		public void postUserImport(MultipartFile file){
			List<SysUsers> sysUsersList = new ArrayList<>();
			try {
				InputStream inputStreamXML = new BufferedInputStream(getClass().getClassLoader().getResourceAsStream("UserJxls.xml"));
				XLSReader xlsReader = ReaderBuilder.buildFromXML(inputStreamXML);
				InputStream inputStream = new BufferedInputStream(file.getInputStream());

				SysUsers sysUsers = new SysUsers();
				Map<String,Object> map = new HashMap<>();
				map.put("sysUsers",sysUsers);
				map.put("sysUsersList",sysUsersList);
				xlsReader.read(inputStream,map);
				sysUserJxlsService.createAll(sysUsersList);
			}catch (Exception e){
				e.printStackTrace();
			}
		}
	```
	sysUsersList ���Ƕ�Ӧxml��items����������ݶ���ֵ�������У�
	inputStreamXML���Ǽ���xml�ļ��������ֵUserJxls.xml���ļ������ڵ����·����
	inputStream�Ƕ�ȡ���ܵ�MultipartFile�ļ�����Excel�ļ�������MultipartFile���ͽ��գ�
	sysUsers����ʵ��������ٴ���Map���󣬽�ʵ�����ʵ���༯����ӵ�Map�У�keyֵ��xml��var��ֵ��items��ֵ��Ӧ��
	xlsReader.read(inputStream,map);ʹ�ø÷����������ݶ�ȡ��ʵ���༯���У���ʱsysUsersList����Excel�е������ˡ�