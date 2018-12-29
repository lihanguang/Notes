#### 配置

* 配置的类型

  ```
  --global 全局配置
  --local 本地仓库配置
  --system 系统配置
  ```

* 添加一个配置

  ```
  git config [配置类型] section.key = "value"
  ```

* 查看当前的配置

  ```
  git config --list
  git confit -l
  // 可以和配置的类型一起使用
  ```

* 编辑配置文件

  ```
  git config --edit
  git config -e
  // 可以和配置的类型一起使用
  ```

*  增加一个配置项

  ```
  git config --add section.key value 
  // 可以和配置的类型一起使用
  ```

* 查看一个配置项

  ```
  git config --get section.key
  // 可以和配置的类型一起使用
  ```

* 删除一个配置项

  ```
  git config --unset section.key
  // 可以和配置的类型一起使用
  ```

  