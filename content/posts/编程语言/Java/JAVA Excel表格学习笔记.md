+++
author = "pikachu"
title = "JAVA Excel表格学习笔记"
date = "2018-09-26"
description = " "
tags = [
    ""
]
categories = [
    "it","java"
]

+++


```
// 创建工作簿
HSSFWorkbook wb = new HSSFWorkbook();

// 创建工作表
HSSFSheet sheet = wb.createSheet();
// 设置工作表的索引行的宽度，15 * 256表示15个字符宽度
sheet.setColumnWidth(0, 15 * 256);
sheet.setColumnWidth(1, 15 * 256);
sheet.setColumnWidth(2, 15 * 256);
sheet.setColumnWidth(3, 15 * 256);

// 设置表格格式
HSSFCellStyle style = wb.createCellStyle();

// 设置字体
HSSFFont font = wb.createFont();
font.setFontName("宋体");
font.setFontHeightInPoints((short) 12);

// 设置单元格类型
style.setAlignment(HSSFCellStyle.ALIGN_CENTER);
style.setVerticalAlignment(HSSFCellStyle.VERTICAL_CENTER);
style.setFont(font);
style.setWrapText(true);

// 合并起止行0，0，起止列0，3单元格
sheet.addMergedRegion(new CellRangeAddress(0, 0, 0, 3));

// 创建行
HSSFRow row0 = sheet.createRow(0);
row0.setHeight((short) 600);

// 在行中创建单元格，并设置内容及格式
HSSFCell cell0 = row0.createCell(0);
cell0.setCellValue("七彩评价社团信息");
cell0.setCellStyle(style);

// 通过获取的list数据在表格中进行填充数据
for (LeagueMemberDto stu : list) {
	HSSFRow rowX = sheet.createRow(num++);
	cell = rowX.createCell(0);
	cell.setCellValue(stu.getName());
	cell.setCellStyle(style);
}


/** 若要为创建的工作簿进行下载，可以先下载到服务器（也可以不用），再由客户端下载 **/

// 文件下载的地址（服务器）
String upLoadPath = request.getSession().getServletContext().getRealPath("/") + "/download/athleteFile/";
String filePath = upLoadPath + fileName;

// 创建下载的文件夹
File dirFile = new File(upLoadPath);
if (!dirFile.exists() || !dirFile.isDirectory()) {
	dirFile.mkdirs();
}

// (在服务器) 生成文件
workbook.write(new FileOutputStream(filePath));

// 下载文件
FileUtil.download(filePath, fileName + ".xls", request, response, false);

```



### EXCEL文件下载：

excel文件下载时可以直接获取response的getOutputStream()进行文件流的输出，再设置response的相关头信息即可实现浏览器下载，代码如下：

```
String fileName = "人员信息.xls";

// 更换文件名的编码格式，否则在浏览器无法正常显示
fileName = response.encodeURL(new String(fileName.getBytes("utf-8"), "iso8859-1"));

response.reset(); // 清空输出流
response.setContentType("application/vnd.ms-excel;charset=utf-8\"");
response.setHeader("Content-Disposition", "attachment;filename=" + fileName);

outputStream = response.getOutputStream();
workbook.write(outputStream);
```


### EXCEL文件导入

**JAR包支持：**

![jar](https://user-images.githubusercontent.com/38284818/48669315-09845500-eb3d-11e8-8504-1e1e580a9056.JPG)



**ERROR提醒：**

- 前端在使用form进行input文件的上传时，需要为相应input设置<i>**name**</i>属性，否则无法正常上传
- 将input标签加上<i>**multiple**</i>属性表示可上传多个文件

- 小数字符串转化为整型：转换为Int类型String num ="1.00"; int abc =<i>**Double.valueOf(num).intValue();**</i>



**前端：**
使用jquery实现同一个form中，不同按钮使用不用servlet进行处理，代码如下：

```
<form action="../ExcelImportServlet" method="post" enctype="multipart/form-data" id="handle" name="handle">
	<div class="input-group">
		<div class="custom-file">
			<input type="file" class="custom-file-input" id="inputGroupFile04" name="file" multiple>
			<label class="custom-file-label" for="inputGroupFile04">Choose
				file</label>
		</div>
		<div class="input-group-append">
			<button class="btn btn-outline-secondary" onclick="$.handler.uploadExcel()">upload</button>
		</div>
		<div class="input-group-append">
			<button class="btn btn-outline-secondary" onclick="$.handler.download()">download</button>
		</div>
	</div>
</form>


<script type="text/javascript">
	$(function() {
		
			$.handler = {
				download: function() {
					$("#handle").attr("action","../ExcelDownloadServlet");
				},
			
				uploadExcel: function() {
					$("#handle").attr({
						action : "../ExcelImportServlet",
						method : "post",
						enctype: "multipart/form-data"
					});
				}
			}
	})
	
</script>
调用：<button class="btn btn-outline-secondary" onclick="$.handler.uploadExcel()">upload</button>
```


**后端：**
excel文件导入时前端可以使用form表单上传文件（将input标签加上<i>**multiple**</i>属性表示可上传多个文件），同时需要将form表单属性设置为： <i>**enctype="multipart/form-data"**</i>，上传后使用**Apache文件上传组件**处理文件上传，代码如下：

```
public static int importExcel(HttpServletRequest request) {

// 使用Apache文件上传组件处理文件上传步骤：

// 为解析类提供配置信息
DiskFileItemFactory factory = new DiskFileItemFactory();

// 创建解析类的实例
ServletFileUpload sfu = new ServletFileUpload(factory);

// 设置单个上传的文件大小
sfu.setFileSizeMax(1024 * 1024 * 5);
//设置总上传文件的文件大小
sfu.setSizeMax(1024 * 1024 * 50);

try {
	List<UserDto> userLists = new ArrayList<>();
	List<FileItem> fileItems = sfu.parseRequest(request);

	System.out.println("FileItem的数目为：" + fileItems.size());
	for (FileItem item : fileItems) {

		if (item.isFormField()) {

		} else {
			String fileName = item.getName();
			String fileType = fileName.substring(fileName.lastIndexOf("."));

			System.out.println("文件名后缀为：" + fileName);
			if (fileType.equals(".xls") || fileType.equals(".xlsx")) {

				InputStream is = item.getInputStream();
				HSSFWorkbook workbook = new HSSFWorkbook(is);
				HSSFSheet sheet;
				HSSFRow row;

				System.out.println("工作簿的表数为：" + workbook.getNumberOfSheets());
				// 遍历工作表
				for (int i = 0; i < workbook.getNumberOfSheets(); i++) {
					sheet = workbook.getSheetAt(i);

					System.out.println("工作表的行数为：" + sheet.getLastRowNum());
					// 遍历行
					for (int j = sheet.getFirstRowNum() + 1; j <= sheet.getLastRowNum(); j++) {
						UserDto user = new UserDto();
						row = sheet.getRow(j);

						user.setId(Double.valueOf(row.getCell(0).toString()).intValue());
						user.setName(row.getCell(1).toString().trim());
						user.setSex(row.getCell(2).toString().trim());
						user.setEmail(row.getCell(3).toString().trim());
						user.setTelephone(row.getCell(4).toString().trim());

						userLists.add(user);
					}

					// 将读取的数据导入数据库
						DBBean.insertUserExcelToDB(userLists);
					}

				} else {
					return -1;
				}
			}
		}

	} catch (FileUploadException e) {
		logger.debug(e.getMessage(), e);
		return 0;
	} catch (IOException e) {
		logger.debug(e.getMessage(), e);
		return 0;
	}
	return 1;
}
```
