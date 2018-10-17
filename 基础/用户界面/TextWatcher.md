### TextWatcher

* TextWatcher主要用来监控EditText或TextView内容的变化。主要有以下三种回调：

  ```java
  public void beforeTextChanged(CharSequence s, int start,
                                    int count, int after)
  // 在内容变化之前调用，所以 s 为内容变化之前的内容
  // start 为在s的start位置开始变化
  // count 为s中被替换掉的个数（原内容）
  // after 为替换的个数（新内容）
  ```

  ```java
  public void onTextChanged(CharSequence s, int start, int before, int count)
  // 在内容变化之时调用，所以 s 为内容变化之后的内容
  // start 为在s的start位置开始变化
  // before 为s中被替换掉的个数（原内容）
  // count 为替换的个数（新内容）
  ```

  ```java
  public void afterTextChanged(Editable s);
  // s 为变化后的内容
  ```

  