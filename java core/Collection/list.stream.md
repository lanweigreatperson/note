jdk8中list.stream的用法

~~~java
public class Test {

  public static void main(String[] args) {
    List<Student> list = new ArrayList<>();
    Student student1 = new Student();
    student1.setAge("12");
    student1.setSex(0);
    Student student5 = new Student();
    student5.setAge("18");
    student5.setSex(0);
    Student student2 = new Student();
    student2.setAge("13");
    student2.setSex(2);
    Student student7 = new Student();
    student7.setAge("18");
    student7.setSex(2);
    Student student3 = new Student();
    student3.setAge("11");
    student3.setSex(1);
    Student student4 = new Student();
    student4.setAge("18");
    student4.setSex(1);
    Student student6 = new Student();
    student6.setAge("18");
    student6.setSex(2);
    list.add(student1);
    list.add(student2);
    list.add(student3);
    list.add(student4);
    list.add(student5);
    list.add(student6);
    list.add(student7);
    List<Demo> demos = new ArrayList<Demo>();
    demos = printData(demos, list);
//         printSexequal0(demos);
//         filterAge(demos);
//         sort(demos);
//         pour(demos);
//         pour2(demos);
//         moreSort(demos);
//         morePour(demos);
    groupByAge(demos);

  }

  /**
   * 数据打印
   */
  public static List<Demo> printData(List<Demo> demos, List<Student> list) {
    demos = list.stream().map(student -> new Demo(student.getAge(), student.getSex()))
        .collect(Collectors.toList());
//        demos.forEach(demo ->{
//            System.out.println(demo.getAge());
//        });
    return demos;
  }

  /**
   * 打印性别为0的数据
   */
  public static void printSexequal0(List<Demo> demos) {
    List<Demo> collect = demos.stream().filter(demo -> demo.getSex() == 0).distinct()
        .collect(Collectors.toList());
    collect.forEach(item -> {
      System.out.println(item.getAge() + ":" + item.getSex());
    });
  }

  /**
   * 过滤年龄大于12的信息
   */
  public static void filterAge(List<Demo> demos) {
    List<Demo> collect = demos.stream().filter(demo -> Integer.valueOf(demo.getAge()) > 12)
        .collect(Collectors.toList());
    collect.forEach(demo -> {
      System.out.println(demo.getAge() + ":" + demo.getSex());
    });
  }

  /**
   * 数据排序
   */
  public static void sort(List<Demo> demos) {
    List<Demo> collect = demos.stream().sorted((s1, s2) -> s1.getAge().compareTo(s2.getAge()))
        .collect(Collectors.toList());
    collect.forEach(demo -> {
      System.out.println(demo.getAge());
    });
  }

  /**
   * 倒叙
   */
  public static void pour(List<Demo> demos) {
    ArrayList<Demo> demoArray = (ArrayList<Demo>) demos;
    demoArray.sort(Comparator.comparing(Demo::getAge).reversed());
    demoArray.forEach(demo -> {
      System.out.println(demo.getAge());
    });
  }

  /**
   * 倒叙2
   */
  public static void pour2(List<Demo> demos) {
    ArrayList<Demo> demoArray = (ArrayList<Demo>) demos;
    Comparator<Demo> comparator = (h1, h2) -> h1.getAge().compareTo(h2.getAge());
    demoArray.sort(comparator.reversed());
    demoArray.forEach(demo -> {
      System.out.println(demo.getAge());
    });
  }

  /**
   * 多条件排序--正序
   */
  public static void moreSort(List<Demo> demos) {
    demos.sort(Comparator.comparing(Demo::getSex).thenComparing(Demo::getAge));
    demos.forEach(demo -> {
      System.out.println(demo.getSex() + ":" + demo.getAge());
    });
  }

  /**
   * 多条件倒叙
   */
  public static void morePour(List<Demo> demos) {
    demos.sort(Comparator.comparing(Demo::getSex).reversed().thenComparing(Demo::getAge));
    demos.forEach(demo -> {
      System.out.println(demo.getSex() + ":" + demo.getAge());
    });
  }

  /**
   * 分组
   */
  public static void groupByAge(List<Demo> demos) {
    Map<String, List<Demo>> collect = demos.stream().collect(Collectors.groupingBy(Demo::getAge));
    collect.forEach((key, value) -> {
      value.forEach(demo -> {
        System.out.println(key + ":" + demo.getSex());
      });
    });
  }
}

class Student {

  private String age;
  private Integer sex;

  public String getAge() {
    return age;
  }

  public void setAge(String age) {
    this.age = age;
  }

  public Integer getSex() {
    return sex;
  }

  public void setSex(Integer sex) {
    this.sex = sex;
  }

}

class Demo {

  private String name;
  private String age;
  private Integer sex;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getAge() {
    return age;
  }

  public void setAge(String age) {
    this.age = age;
  }

  public Integer getSex() {
    return sex;
  }

  public void setSex(Integer sex) {
    this.sex = sex;
  }

  public Demo(String age, Integer sex) {
    super();
    this.age = age;
    this.sex = sex;
  }


}
~~~

