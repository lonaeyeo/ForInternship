## 流式数据处理

<!--在我接触到java8流式处理的时候，我的第一感觉是流式处理让集合操作变得简洁了许多，通常我们需要多行代码才能完成的操作，借助于流式处理可以在一行中实现。-->

比如我们希望对一个包含整数的集合中筛选出所有的偶数，并将其封装成为一个新的List返回，那么在java8之前，我们需要通过如下代码实现：

```java
List<Integer> evens = new ArrayList<>();
for (final Integer num : nums) {
    if (num % 2 == 0) {
        evens.add(num);
    }
}
```

通过java8的流式处理，我们可以将代码简化为：

```java
List<Integer> evens = nums.stream().filter(num -> num % 2 == 0).collect(Collectors.toList());
```

解释：`stream()`操作**将集合转换成一个流**，`filter()`执行我们自定义的筛选处理，这里是通过lambda表达式筛选出所有偶数，最后我们通过`collect()`对结果进行封装处理，并通过`Collectors.toList()`指定其封装成为一个List集合返回。

java8的流式处理极大的简化了对于集合的操作，通过内部迭代来实现对流的处理。实际上不光是集合，包括数组、文件等，类似于写SQL语句，一个流式处理可以分为三个部分：转换成流、中间操作、终端操作。如下图：

<img src="https://images2015.cnblogs.com/blog/848293/201611/848293-20161103143545315-2110948414.png" alt="img" style="zoom: 80%;" /> 



举个栗子，先定义一个简单的学生实体类：

```java
public class Student {
    /** 学号 */
    private long id;
    private String name;
    private int age;
    /** 年级 */
    private int grade;
    /** 专业 */
    private String major;
    /** 学校 */
    private String school;
    // 省略getter和setter
}

// 初始化
List<Student> students = new ArrayList<Student>() {
    {
        add(new Student(20160001, "孔明", 20, 1, "土木工程", "武汉大学"));
        add(new Student(20160002, "伯约", 21, 2, "信息安全", "武汉大学"));
        add(new Student(20160003, "玄德", 22, 3, "经济管理", "武汉大学"));
        add(new Student(20160004, "云长", 21, 2, "信息安全", "武汉大学"));
        add(new Student(20161001, "翼德", 21, 2, "机械与自动化", "华中科技大学"));
        add(new Student(20161002, "元直", 23, 4, "土木工程", "华中科技大学"));
        add(new Student(20161003, "奉孝", 23, 4, "计算机科学", "华中科技大学"));
        add(new Student(20162001, "仲谋", 22, 3, "土木工程", "浙江大学"));
        add(new Student(20162002, "鲁肃", 23, 4, "计算机科学", "浙江大学"));
        add(new Student(20163001, "丁奉", 24, 5, "土木工程", "南京大学"));
    }
};
```



### 中间操作

#### filter( )

filter定义为：`Stream<T> filter(Predicate<? super T> predicate)`，filter接受一个谓词`Predicate`，通过这个谓词定义筛选条件，在介绍lambda表达式时我们介绍过`Predicate`是一个函数式接口，其包含一个`test(T t)`方法，该方法返回`boolean`。现在我们希望从集合`students`中筛选出所有武汉大学的学生，那么我们可以通过filter来实现，并将筛选操作作为参数传递给filter：

```java
List<Student> whuStudents = students.stream()                                    .filter(student -> "武汉大学".equals(student.getSchool()))                         
           .collect(Collectors.toList());
```

#### distinct( )

distinct操作类似于我们在写SQL语句时，添加的`DISTINCT`关键字，用于去重处理，distinct基于`Object.equals(Object)`实现，筛选出所有不重复的偶数：

```java
List<Integer> evens = nums.stream().filter(num -> num % 2 == 0).distinct()               .collect(Collectors.toList());
```

#### limit( )

 limit操作也类似于SQL语句中的`LIMIT`关键字，不过相对功能较弱，limit返回包含前n个元素的流，当集合大小小于n时，则返回实际长度，比如下面的例子返回前两个专业为`土木工程`专业的学生：

```java
List<Student> civilStudents = students.stream()                                    .filter(student -> "土木工程".equals(student.getMajor())).limit(2)                          .collect(Collectors.toList());
```

#### skip( )

skip操作与limit操作相反，如同其字面意思一样，是跳过前n个元素，比如我们希望找出排序在2之后的土木工程专业的学生，那么可以实现为：

```java
List<Student> civilStudents = students.stream()                                    .filter(student -> "土木工程".equals(student.getMajor()))                                    .skip(2).collect(Collectors.toList());
```



### 映射

在SQL中，借助`SELECT`关键字后面添加需要的字段名称，可以仅输出我们需要的字段数据，而流式处理的映射操作也是实现这一目的，在java8的流式处理中，主要包含两类映射操作：map和flatMap。

#### map( )

举例说明，假设筛选出所有专业为**计算机科学**的学生姓名，通过map将学生实体映射成为学生姓名字符串，具体实现如下：

```java
List<String> names = students.stream()
.filter(student -> "计算机科学" .equals(student.getMajor()))
.map(Student::getName).collect(Collectors.toList());
```



#### flatMap( )

flatMap与map的区别在于 **flatMap是将一个流中的每个值都转成一个个流，然后再将这些流扁平化成为一个流**。举例说明，假设我们有一个字符串数组`String[] strs = {"java8", "is", "easy", "to", "use"};`，我们希望输出构成这一数组的所有非重复字符，那么我们可能首先会想到如下实现：

```java
List<String[]> distinctStrs = Arrays.stream(strs)
                                .map(str -> str.split(""))  // 映射成为Stream<String[]>
                                .distinct()
                                .collect(Collectors.toList());
```

在执行map操作以后，我们得到是一个包含多个字符串（构成一个字符串的字符数组）的流，此时执行**distinct**操作是**基于在这些字符串数组之间的对比**，所以达不到我们希望的目的，此时的输出为：

```java
[j, a, v, a, 8]
[i, s]
[e, a, s, y]
[t, o]
[u, s, e]
```

distinct只有对于一个包含多个字符的流进行操作才能达到我们的目的，即对`Stream<String>`进行操作。此时flatMap就可以达到我们的目的：

```java
List<String> distinctStrs = Arrays.stream(strs)
                                .map(str -> str.split(""))  // 映射成为Stream<String[]>
                                .flatMap(Arrays::stream)  // 扁平化为Stream<String>
                                .distinct()
                                .collect(Collectors.toList());
```

flatMap将由map映射得到的`Stream<String[]>`，转换成由各个字符串数组映射成的流`Stream<String>`，再将这些小的流扁平化成为一个由所有字符串构成的大流`Steam<String>`，从而能够达到我们的目的。
与map类似，flatMap也提供了针对特定类型的映射操作：`flatMapToDouble(Function<? super T,? extends DoubleStream> mapper)`，`flatMapToInt(Function<? super T,? extends IntStream> mapper)`，`flatMapToLong(Function<? super T,? extends LongStream> mapper)`。



### 终端操作

#### 查找

##### allMatch( )

用于检测是否全部都满足指定的参数行为：

```java
boolean isAdult = students.stream().allMatch(student -> student.getAge() >= 18);
```

##### anyMatch( )

检测是否存在一个或多个满足指定的参数行为：

##### noneMathch( )

检测是否不存在满足指定行为的元素:

##### findFirst( )

返回满足条件的第一个元素:

```java
Optional<Student> optStu = students.stream().filter(student -> "土木工程".equals(student.getMajor())).findFirst();
```

##### findAny( )

findAny不一定返回第一个，而是返回任意一个:



#### 归约

前面的例子中大部分都是通过`collect(Collectors.toList())`对数据封装返回，如果目标不是返回一个新的集合，而是希望对经过参数化操作后的集合进行进一步的运算，那么我们可用对集合实施归约操作。java8的流式处理提供了`reduce`方法来达到这一目的。

前面通过mapToInt将`Stream<Student>`映射成为`IntStream`，并通过`IntStream`的sum方法求得所有学生的年龄之和，实际上我们通过归约操作，也可以达到这一目的，实现如下：

```java
// 前面例子中的方法
int totalAge = students.stream()
                .filter(student -> "计算机科学".equals(student.getMajor()))
                .mapToInt(Student::getAge).sum();
// 归约操作
int totalAge = students.stream()
                .filter(student -> "计算机科学".equals(student.getMajor()))
                .map(Student::getAge)
                .reduce(0, (a, b) -> a + b);

// 进一步简化
int totalAge2 = students.stream()
                .filter(student -> "计算机科学".equals(student.getMajor()))
                .map(Student::getAge)
                .reduce(0, Integer::sum);

// 采用无初始值的重载版本，需要注意返回Optional
Optional<Integer> totalAge = students.stream()
                .filter(student -> "计算机科学".equals(student.getMajor()))
                .map(Student::getAge)
                .reduce(Integer::sum);  // 去掉初始值
```



#### 收集

前面利用`collect(Collectors.toList())`是一个简单的收集操作，是对处理结果的封装，对应的还有`toSet`、`toMap`，以满足对于结果组织的需求。这些方法均来自于`java.util.stream.Collectors`，称之为收集器。