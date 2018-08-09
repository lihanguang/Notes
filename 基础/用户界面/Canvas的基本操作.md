<table>
  <thead>
    <tr>
      <th>操作类型</th>
      <th>相关API</th>
      <th>备注</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>绘制颜色</td>
      <td>drawColor, drawRGB, drawARGB</td>
      <td>使用单一颜色填充整个画布</td>
    </tr>
    <tr>
      <td>绘制基本形状</td>
      <td>drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc</td>
      <td>依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧</td>
    </tr>
    <tr>
      <td>绘制图片</td>
      <td>drawBitmap, drawPicture</td>
      <td>绘制位图和图片</td>
    </tr>
    <tr>
      <td>绘制文本</td>
      <td>drawText,    drawPosText, drawTextOnPath</td>
      <td>依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字</td>
    </tr>
    <tr>
      <td>绘制路径</td>
      <td>drawPath</td>
      <td>绘制路径，绘制贝塞尔曲线时也需要用到该函数</td>
    </tr>
    <tr>
      <td>顶点操作</td>
      <td>drawVertices, drawBitmapMesh</td>
      <td>通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用</td>
    </tr>
    <tr>
      <td>画布剪裁</td>
      <td>clipPath,    clipRect</td>
      <td>设置画布的显示区域</td>
    </tr>
    <tr>
      <td>画布快照</td>
      <td>save, restore, saveLayerXxx, restoreToCount, getSaveCount</td>
      <td>依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 回滚到指定状态、 获取保存次数</td>
    </tr>
    <tr>
      <td>画布变换</td>
      <td>translate, scale, rotate, skew</td>
      <td>依次为 位移、缩放、 旋转、错切</td>
    </tr>
    <tr>
      <td>Matrix(矩阵)</td>
      <td>getMatrix, setMatrix, concat</td>
      <td>实际上画布的位移，缩放等操作的都是图像矩阵Matrix， 只不过Matrix比较难以理解和使用，故封装了一些常用的方法。</td>
    </tr>
  </tbody>
</table>