#### Intent作用：

- 开启一个Activity。实现组件之间通信（Activity之间的切换），通信的数据封装在Bundle中
- 开启一个Service
- 传递广播

#### Intent对象的属性

- component Name。Activity的包名，完整类名

- Action和Data属性
  
  ```java
  Intent intent = new Intent();
  ImageButton imageButton = (ImageButton)v;
  if(imageButton.getId() == R.id.imageButton_phone) {
      intent.setAction(Intent.ACTION_CALL);
      intent.setData(Uri.parse("tel:18852958252"));
      startActivity(intent);
  }
  ```

- Action和Category属性

  