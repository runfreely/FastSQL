# 一.FastSql简介
一个基于spring-jdbc的简单ORM框架，使用了NamedParameterJdbcTemplate类，可以加速你的数据库开发。

数据库设计约定：

1.使用uuid字符串做为主键类型

 
# 二.BaseDAO
应用中数据访问类需要继承这个类，进行各种操作

## 1.准备数据
新建表student
```
CREATE TABLE `student` (
  `id` varchar(36) NOT NULL,
  `name` varchar(10) NOT NULL,
  `age` int(11) DEFAULT NULL,
  `birthday` date DEFAULT NULL,
  `home_address` varchar(255) DEFAULT NULL,
   PRIMARY KEY (`id`)
)
```
新建实体类Student.java，同名或驼峰下转划线相同可以省略@TableName
```
@TableName("student") 
public class Student {
    private String id;
    private String name;
    private Integer age;
    private Date birthday;
    private String homeAddress;
    //省略getter和setter
}

```
新建数据访问类StudentDAO.java, 并实现 BaseDAO（泛型中Student为对应的实体类）

```
@Repository
public class StudentDAO extends BaseDAO<Student> {
     
}
```

## 2.数据保存 ，继承自BaseDAO中的方法

### public String save(E object) 
插入对象中的值到数据库，null值在数据库中会设置为NULL
```
Student student = new Student();
//student.setId(UUID.randomUUID().toString()); //不指定id将会自动把id保存为uuid
student.setName("小丽");
student.setBirthday(new Date());
student.setHomeAddress("");

String id = studentDao.save(student);//获取保存成功的id
```
等价如下SQL语句（注意：age被设置为null）
```
INSERT INTO student(id,name,age,birthday,home_address) 
 VALUES 
('622bca40-4c64-43aa-8819-447718bdafa5','小丽',NULL,'2017-07-11','')
```


### public String saveIgnoreNull(E object)  
插入对象中非null的值到数据库
```
Student student = new Student();
//student.setId(UUID.randomUUID().toString());//不指定id将会自动把id保存为uuid
student.setName("小丽");
student.setBirthday(new Date());
student.setHomeAddress("");
 
String id =  studentDao.saveIgnoreNull(student);//获取保存成功的id
```
等价如下SQL语句（注意：没有对age进行保存，在数据库层面age将会保存为该表设置的默认值，如果没有设置默认值，将会被保存为null ）
```
INSERT INTO student(id,name,birthday,home_address) 
 VALUES 
('622bca40-4c64-43aa-8819-447718bdafa5','小丽','2017-07-11','')

```
## 3.数据删除 ，继承自BaseDAO中的方法

### public int delete(String id) 
根据id删除数据
```
int deleteRowNumber = studentDao.delete("22b66bcf-1c2e-4713-b90d-eab17182b565");
```
等价如下SQL语句
```
DELETE FROM student WHERE id='22b66bcf-1c2e-4713-b90d-eab17182b565'
```

### public int deleteAll()
删除某个表所有行
```
int number = studentDao.deleteAll()//获取删除的行数量
```

### public int  deleteInBatch(List<String> ids) 和 public int deleteInBatch(String... ids)
根据id列表批量删除数据(所有删除语句将会一次性提交到数据库)
```
List<String> ids = new ArrayList<>();
ids.add("467641d2-e344-45e9-9e0e-fd6152f80867");
ids.add("881c80a1-8c93-4bb7-926e-9a8bc9799a72");
int number = studentDao.deleteInBatch(ids);//返回成功删除的数量
```

## 4.数据修改 ，继承自BaseDAO中的方法

### String update(E entity) 
根据对象进行更新（null字段在数据库中将会被设置为null），对象中id字段不能为空 

### String updateIgnoreNull(E entity) 
根据对象进行更新（只更新实体中非null字段），对象中id字段不能为空 

### String update(String id, Map<String, Object> updateColumnMap) 
使用id根据map进行更新
```
Map<String, Object> map = new HashMap<>();
map.put("home_address", "成都");// map.put("homeAddress", "成都") -- 使用实体字段作为key也可以
map.put("birthday", new Date());
map.put("age", null);

studentDao.update("17661a16-e77b-4979-8a25-c43a489d42ad", map);

```
等价如下SQL语句
```
UPDATE student SET home_address='成都', birthday='2017-07-17',age=NULL WHERE id='22b66bcf-1c2e-4713-b90d-eab17182b565'
```
## 5.单表查询 ，继承自BaseDAO中的方法
