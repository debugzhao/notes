#### 学习路线

- Activity概述
- 创建、配置、启动、关闭Activity
- 多个Activity的使用
- 如何使用Fragment

#### 创建和配置Activity

创建和配置Activity

- 创建继承自Activity的Activity
- 重写需要回调的方法 如onCreate()
- 设置需要显示的视图   setContentView(R.layout.activity_main)
- **简洁方式：直接创建Activity向导**

#### 启动和关闭Activity

```java
public void startMyActivity(View view){
    Intent intent = new Intent(MainActivity.this,myActivity.class);
    startActivity(intent);
}
```

```java
//关闭Activity
public void finish(View view){
	finish();
}
```

#### 使用Bundle实现Activity之间交换数据

#### Fragment

作用：

- 用来在一个Activity描述一个行为或者一些界面；
- 可以在多个Activity中重用一个Fragment
- 可以使用多个Fragment在一个Activity中

Fragment声明周期

![](C:\Users\Administrator\Desktop\md\Android\image\fragment声明周期.png)

在Activity中创建Fragment

```xml
<fragment
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:name="com.example.fragment.ListFragment"
    android:id="@+id/list"
    android:layout_weight="1"
/>
```

在Activity运行时动态添加Fragment

```java
DetailFragment detailFragment = new DetailFragment();
FragmentTransaction transaction = getFragmentManager().beginTransaction();
transaction.add(R.id.fl,detailFragment);
transaction.commit();
```

