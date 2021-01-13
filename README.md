





```java

package com.excel;


import java.io.File;
import java.util.List;
import java.util.regex.Pattern;

import org.apache.commons.lang3.StringUtils;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.excel.read.Company;
import com.excel.read.User;
import com.practice.core.Jacksons;
import com.practice.core.excel.read.ExcelReader;

/**
 *  简洁、易用、优雅
 *
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = ReadExcelDemo.class)
public class ReadExcelDemo {
	private static File file;
	static {
		file = new File("D:\\01.xls");
	}
	
	
	@Test
	public void test1() {
		List<User> results = ExcelReader.builder(file) // 创建excel文件读取器
					.map(User.class) // 读取数据映射的对象
					.read("name", "mobile", "age", "height") // 从列坐标0开始，指定依次读取excel表的每列到所映射的对象属性
					.execute() // 执行读取操作
					.getResult(); //获取读取结果
		System.out.println(Jacksons.toJson(results));
		// [{"name":"张无忌","mobile":"13695856985","age":28,"height":68.0},{"name":"杨过","mobile":"13658958569","age":32,"height":67.0}]
	}
	
	
	@Test
	public void test2() {
		List<User> results = ExcelReader.builder(file) // 创建excel文件读取器
					.map(User.class) // 读取数据映射的对象
					.read(0, "name") // 读取坐标为0列（即第1列）到属性name
					.read(2, "age")  // 读取坐标为2列到属性age
					.read(4, "height") // 读取坐标为4列到属性height
					.execute() // 执行读取操作
					.getResult(); //获取读取结果
		System.out.println(Jacksons.toJson(results));
		// [{"name":"张无忌","age":28,"height":41.0},{"name":"杨过","age":32,"height":42.0}]
	}
	
	
	
	@Test
	public void test01() {
		ExcelReader excel = ExcelReader.builder(file);
		// 读取excel表的数据封装到指定对象，最简单的写法
		excel.map(User.class) // 指定读取数据映射的对象
				.read("name", "mobile", "age", "height") // 读取excel表的数据，依次指定列所映射的对象属性
			.execute();
		/* 同等于以下写法
		excel.map(User.class)
				 .read("name")
				 .read("mobile")
				 .read("age")
				 .read("height")
			 .execute();
		*/
		List<User> userList = excel.getResult();
		System.out.println(Jacksons.toJson(userList));
	}
	
	
	@Test
	public void test02() {
		// 上一个例子中，我们对excel表依次逐列地读取，可省略坐标。但有时并不需要读取每一列，因此可以通过指定坐标来读取对应的列
		ExcelReader excel = ExcelReader.builder(file);
		excel.map(User.class)
				 .read(0, "name") // 读取坐标为0列，即第1列
				 .read(2, "age")  // 读取坐标为2列，即第3列
				 .read(4, "height")
			 .execute();
		List<User> userList = excel.getResult(User.class);
		System.out.println(Jacksons.toJson(userList));
	}
	
	
	@Test
	public void test03() {
		// 有时候我们往往需要对读取的数据进行加工，比如格式，运算等
		ExcelReader excel = ExcelReader.builder(file);
		excel.map(User.class)
				 .read(0, "name")
				 .read(2, "age", (e) -> { // e 为excel表单元格的字符串，此处进行了回调，返回值将作为对应的对象属性（age）的值
					 return Integer.valueOf(e) + 1;
				 })
				 .read(4, "height")
			 .execute();
		
		String errorMessage = excel.toErrorMessage();
		List<User> userList = excel.getResult(User.class);
		System.out.println("验证提示：" + errorMessage);
		System.out.println("读取数据：" + Jacksons.toJson(userList));
	}
	
	
	@Test
	public void test04() {
		// 除了数据加工，数据验证也是必不可少的
		ExcelReader excel = ExcelReader.builder(file);
		excel.map(User.class)
				 .read(0, "name")
				 .read(2, "age", (e, r) -> { // 参数e是单元格的值，参数r是行对象
					 r.validate(StringUtils.isBlank(e), "年龄不能为空"); // 当验证不通过时将终止回调中的以下代码的执行
					 r.validate(!isNumber(e) , "年龄必须是数字");
					 int age = Integer.valueOf(e) + 1;
					 r.validate(age < 30, "年龄必须不小于30岁");
					 return age;
				 })
				 .read(4, "height")
			 .execute();
		
		String errorMessage = excel.toErrorMessage();
		List<User> userList = excel.getResult(User.class);
		System.out.println(errorMessage);
		System.out.println("读取数据：" + Jacksons.toJson(userList));
	}
	
	
	@Test
	public void test05() {
		ExcelReader excel = ExcelReader.builder(file);
		excel.map(User.class)
				 .read(0, "name")
				 .read(2, "age", (e, r) -> {
					 r.validate(StringUtils.isBlank(e), "年龄不能为空");
					 r.validate(!isNumber(e) , "年龄必须是数字");
					 int age = Integer.valueOf(e) + 1;
					 r.validate(age < 30, "年龄必须不小于30岁");
					 return age;
				 })
				 .read(3, "height")
				 .read(5, "province")
				 .read(6, "city", (e, r) -> {
					 r.validate(StringUtils.isBlank(e), "市不能为空");
					 String province = r.getCellValue(5);
					 r.validate(!isExists(province, e), e + "不存在");
					 return e;
				 })
			 .execute();
		
		String errorMessage = excel.toErrorMessage();
		List<User> userList = excel.getResult(User.class);
		System.out.println(errorMessage);
		System.out.println("读取数据：" + Jacksons.toJson(userList));
	}
	
	
	@Test
	public void test06() {
		ExcelReader excel = ExcelReader.builder(file);
		excel.map(User.class)
			 .map(Company.class, "cp")
				 .read(0, "name")
				 .read(2, "age", (e, r) -> {
					 r.validate(StringUtils.isBlank(e), "年龄不能为空");
					 r.validate(!isNumber(e) , "年龄必须是数字");
					 int age = Integer.valueOf(e) + 1;
					 r.validate(age < 30, "年龄必须不小于30岁");
					 return age;
				 })
				 .read(3, "height")
				 .read(5, "province")
				 .read(6, "city", (e, r) -> {
					 r.validate(StringUtils.isBlank(e), "市不能为空");
					 String province = r.getCellValue(5);
					 r.validate(!isExists(province, e), e + "不存在");
					 return e;
				 })
				 .read(7, "cp.name")
				 .read(8, "cp.work")
			 .execute();
		
		String errorMessage = excel.toErrorMessage();
		List<User> userList = excel.getResult(User.class);
		List<Company> companyList = excel.getResult(Company.class);
		
		System.out.println(errorMessage);
		System.out.println("读取数据：" + Jacksons.toJson(userList));
		System.out.println("读取数据：" + Jacksons.toJson(companyList));
	}
	
	
	
	@Test
	public void test07() {
		ExcelReader excel = ExcelReader.builder(file);
			excel.read(0, User::getName)
				 .read(2, User::getAge, (e, r) -> {
					 r.validate(StringUtils.isBlank(e), "年龄不能为空");
					 r.validate(!isNumber(e) , "年龄必须是数字");
					 int age = Integer.valueOf(e) + 1;
					 r.validate(age < 30, "年龄必须不小于30岁");
					 return age;
				 })
				 .read(3, User::getHeight)
				 .read(5, User::getProvince)
				 .read(6, User::getCity, (e, r) -> {
					 r.validate(StringUtils.isBlank(e), "市不能为空");
					 String province = r.getCellValue(5);
					 r.validate(!isExists(province, e), e + "不存在");
					 return e;
				 })
				 .read(7, Company::getName)
				 .read(8, Company::getWork)
			 .execute();
		
		String errorMessage = excel.toErrorMessage();
		List<User> userList = excel.getResult(User.class);
		List<Company> companyList = excel.getResult(Company.class);
		
		System.out.println(errorMessage);
		System.out.println("读取数据：" + Jacksons.toJson(userList));
		System.out.println("读取数据：" + Jacksons.toJson(companyList));
	}
	
	
	public boolean isExists(String province, String city) { 
		return true;
	}
	
	
	public boolean isNumber(String str) { 
        Pattern pattern = Pattern.compile("^[-\\+]?[\\d]*$"); 
        return pattern.matcher(str).matches(); 
	}
	
	
}




```
