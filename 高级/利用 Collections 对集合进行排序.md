# 利用 Collections 对集合进行排序

由于 Collections 的 sort 方法对其规定了泛型

```java
public static <T extends Comparable<? super T>> void sort(List<T> list)
```

所以，想要使用sort方法，排序的对象必须**实现 Comparable 接口**

以User为例，对其 name 属性进行排序

```java
ArrayList<User> userList = new ArrayList<>();
userList.add(new User(1, "Tom", 3)); // id 姓名 年龄
userList.add(new User(2, "Jack", 4));
userList.add(new User(3, "Sarah", 5));
```



### 第一种方法：使用 Comparator 类

```java
Comparator<User> c = new Comparator<User>() {
    @Override
    public int compare(User u1, User u2) { 
        // 这里的 compareTo()方法是 String 类重写的，我们的 getName() 方法返回 String 类型，可以直接使用
        // String 类的 compareTo() 方法附文尾
        return u1.getName().compareTo(u2.getName());
    }
}
Collections.sort(userList, c);
```

**上面也可以写成 lambda 表达式**

```java
Comparator<User> c = (u1, u2) -> u1.getName().compareTo(u2.getName());
Collections.sort(userList, c);
```

**还可以使用 Comparator 的静态方法 comparing()**

```java
Comparator<User> c = Comparator.comparing(User:getName()); // 推荐
Collections.sort(userList, c);
```



### 第二种方法：让 User 类实现 Comparable 接口

```java
public class User implements Comparable<User> {
    // 省略各种属性，getter setter toString...
    // 在 User 类里重写 compareTo() 方法
    public int compareTo(User u){
        return name.compareTo(u.name);
    }
}

// 这样，在刚刚的地方就可以直接使用 Collections.sort() 了
Collections.sort(userList);
```



**附：String 类的 compareTo 方法**

```java
public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
```

