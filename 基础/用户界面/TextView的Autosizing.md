#### XML设置
```
<TextView
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:autoSizeTextType="uniform" 
    android:autoSizeMinTextSize="2sp"
    android:autoSizeMaxTextSize="12sp"
    android:autoSizeStepGranularity="1sp"
    android:autoSizePresetSizes="@array/autosize_text_sizes"
/>
```
* autoSizeTextType：开关
* autoSizeMinTextSize：最小大小
* autoSizeMaxTextSize：最大大小
* autoSizeStepGranularity：变化的粒度
* autoSizePresetSizes：接受一组大小数组，指定适应后的大小
 

#### Java设置
```
TextViewCompat.setAutoSizeTextTypeWithDefaults(textView, autoSizeTextType);//开关

TextViewCompat.setAutoSizeTextTypeUniformWithConfiguration(textView,
autoSizeMinTextSize, autoSizeMaxTextSize, autoSizeStepGranularity, unit);//配置

TextViewCompat.setAutoSizeTextTypeUniformWithPresetSizes(textView, presetSizes, unit)//数组方式 
```

#### 注意
* TextView 必须限定尺寸
* Autosizeing 不能作用在 EditText 中
* 预设尺寸不一定都命中
* 和 singleLine 冲突，应使用maxline=1

    