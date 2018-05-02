---
title: fastjson学习记录 
tags: 知识收集
grammar_cjkRuby: true
---

# JSONObject

- 字符串 => JSONObject
```java
JSONObject jsonObject = JSONObject.parseObject(JSON_OBJ_STR);
```

- JSONObject => 字符串
```java
// 第一种方式
String jsonString = JSONObject.toJSONString(jsonObject);

// 第二种方式
String jsonString = jsonObject.toJSONString();
```

# JSONArray

- 字符串 => JSONArray
```java
JSONArray jsonArray = JSONArray.parseArray(JSON_ARRAY_STR);
```

- JSONArray => 字符串
```java
//第一种方式
String jsonString = JSONArray.toJSONString(jsonArray);

// 第二种方式
String jsonString = jsonArray.toJSONString();
```

- 遍历JSONArray
```java
//遍历方式1
int size = jsonArray.size();
for (int i = 0; i < size; i++) {
    JSONObject jsonObject = jsonArray.getJSONObject(i);
    System.out.println("studentName:  " + jsonObject.getString("studentName") + ":" + "  studentAge:  "
            + jsonObject.getInteger("studentAge"));
}

//遍历方式2
for (Object obj : jsonArray) {
    JSONObject jsonObject = (JSONObject) obj;
    System.out.println("studentName:  " + jsonObject.getString("studentName") + ":" + "  studentAge:  "
            + jsonObject.getInteger("studentAge"));
}
```

# Java Bean

- JSONObject/JSONArray => Java Bean
```java
//JSONObject
//第一种方式,使用TypeReference<T>类,由于其构造方法使用protected进行修饰,故创建其子类
Student student = JSONObject.parseObject(JSON_OBJ_STR, new TypeReference<Student>() {});

//第二种方式,使用Gson的思想
Student student = JSONObject.parseObject(JSON_OBJ_STR, Student.class);


//JSONArray
//第一种方式
List<Student> students = new ArrayList<Student>();
Student student = null;
for (Object object : jsonArray) {

    JSONObject jsonObjectone = (JSONObject) object;
    String studentName = jsonObjectone.getString("studentName");
    Integer studentAge = jsonObjectone.getInteger("studentAge");

    student = new Student(studentName,studentAge);
    students.add(student);
}

//第二种方式,使用TypeReference<T>类,由于其构造方法使用protected进行修饰,故创建其子类
List<Student> studentList = JSONArray.parseObject(JSON_ARRAY_STR, new TypeReference<ArrayList<Student>>() {});

//第三种方式,使用Gson的思想
List<Student> studentList1 = JSONArray.parseArray(JSON_ARRAY_STR, Student.class);

```

- Java Bean => JSONObject/JSONArray String
```java
String jsonString = JSONObject.toJSONString(student);

String jsonString = JSONArray.toJSONString(students);

```


# 总结

- String => json对象
```java
JSONObject jsonObject = JSONObject.parseObject(JSON_OBJ_STR);
JSONArray jsonArray = JSONArray.parseArray(JSON_ARRAY_STR);
```

- Java Bean => json String
```java
String jsonString = javaBean.toJSONString();
```

- 使用json对象
```java
int a = jsonObject.getInteger("key");
double b = jsonObject.getDouble("key");

JSONObject obj = jsonObject.getJSONObject("key");
JSONArray array = jsonObject.getJSONArray("key");
```