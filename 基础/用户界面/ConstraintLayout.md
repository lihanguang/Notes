#### ConstraintLayout的优势
* 相比较RelativeLayout，ConstraintLayout的性能大约提高40%
* 比传统布局的布局层次更少

#### 相对布局相关属性
* layout_constraintxxx_toyyyOf  源控件的xxx边和目标控件的yyy边对齐
* xxx和yyy可以是left、right、top、bottom、start、end、baseline

#### Margin相关
* 支持一般的margin设置
* layout_goneMarginXXX 在目标控件visible为gone的时候应用的margin

#### Bias 
* 遇到相反约束的时候，指定偏向的比例（0-1）0为偏向左上，1为偏向右下，0.5为居中
* layout_constraintHorizontal_bias 水平
* layout_constraintVertical_bias   垂直

#### Circular positioning 
* 圆形约束
* layout_constraintCircle : 关联的控件ID
* layout_constraintCircleRadius : 半径
* layout_constraintCircleAngle : 角度

#### Widgets dimension constraints 宽高属性
* 指定大小，比如20dp
* WRAP_CONTENT 自适应
* 0dp MATCH_CONSTRAINT（由ConstraintLayout根据情况决定）
* MATCH_PARENT 不支持

#### GuideLine
* orientation:取值为‘vertical’和‘horizontal’
* layout_constraintGuide_begin：距离顶部的距离
* layout_constraintGuide_end：距离底部的距离
* layout_constraintGuide_percent：百分比