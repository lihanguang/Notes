* 有的手机设置Shader时不会马上生效。但是通过点击事件设置一个会导致重绘的操作（换字体、对齐方式和设置背景颜色）之后又会生效，目前还不清楚原因。解决办法是在设置Shader的时候同时使用setColor方法给他设置一个独一无二的颜色

* 斜体会导致一些文字的右边部分被遮挡。这个BUG的原因是TextView的onDraw方法中使用了

  ```java
  canvas.clipRect(clipLeft, clipTop, clipRight, clipBottom);
  ```

  这行代码对画布进行了裁剪，导致文字无法全部绘制出来。

  一个不完美的解决办法是自己绘制文字，跳过这一段代码。但是因为TextView内部不只是绘制文字，还包括了绘制光兵、drawables等绘制操作，所以这种解决办法只适用于不需要光标和drawables的情况。

  ```java
      @Override
      protected void onDraw(Canvas canvas) {
          if (mIsEditing) { // mIsEditing用来标记是不是编辑状态，如果是编辑状态则使用原来的绘制
              super.onDraw(canvas);
          } else {
              // 不是输入的情况自己画
              if (mFakeCanvas == null) {
                  initFakeCanvas();
              }
              super.onDraw(mFakeCanvas);
              Layout layout = getLayout();
              if (layout == null) {
                  super.onDraw(canvas);
                  return;
              }
              // 这是导致斜体情况下部分文字会被遮挡一部分的原因
              //canvas.clipRect(getCompoundPaddingLeft(), getExtendedPaddingTop(), getWidth() - getCompoundPaddingRight(), getHeight() - getExtendedPaddingBottom());
              canvas.translate(getCompoundPaddingLeft(), getExtendedPaddingTop());
              layout.draw(canvas, null, null, 0);
          }
      }
      
      private void initFakeCanvas() {
          mFakeCanvas = new Canvas();
      }
  ```

* 硬件加速的一些坑

  * 如果关掉硬件加速，那么TextView在放大的情况下会有严重的锯齿（无解）
  * 在硬件加速下，TextView放大到一定倍数（不同手机并不相同）会导致花屏（无解）
  * 在硬件加速下，如果有使用emoji表情并且旋转角度之后，通过getDrawingCache()获取Bitmap时emoji的显示会有问题（无解）

* 在一些手机（目前发现有三星的Android5）中，如果在使用了Shader的情况下输入emoji表情，那么emoji表情可能会被渲染为带有Shader效果的矩形（无解）

* EditText有一个xml属性textCursorDrawable可以设置光标的样式，但是没有Java代码可以动态改变光标的颜色，这里可以通过反射来改变（Android 9 限制了非SDK接口的访问后可能失效）

  ```java
      /**
       * 通过反射来修改光标颜色
       * @param color
       */
      private void setCursorColor(int color) {
          try {
              Field fCursorDrawableRes = TextView.class.getDeclaredField("mCursorDrawableRes");
              fCursorDrawableRes.setAccessible(true);
              int mCursorDrawableRes = fCursorDrawableRes.getInt(mCaptionTextView);
              Field fEditor = TextView.class.getDeclaredField("mEditor");
              fEditor.setAccessible(true);
              Object editor = fEditor.get(mCaptionTextView);
              Class<?> clazz = editor.getClass();
              Field fCursorDrawable = clazz.getDeclaredField("mCursorDrawable");
              fCursorDrawable.setAccessible(true);
              Drawable[] drawables = new Drawable[1];
              drawables[0] = mCaptionTextView.getContext().getResources().getDrawable(mCursorDrawableRes);
              if (drawables[0] instanceof GradientDrawable) {
                  ((GradientDrawable)drawables[0]).setColor(color);
              } else {
                  drawables[0].setColorFilter(color, PorterDuff.Mode.SRC_IN);
              }
              fCursorDrawable.set(editor, drawables);
          } catch (NoSuchFieldException e) {
              e.printStackTrace();
              TraceLog.e(TAG, e.toString());
          } catch (IllegalAccessException e) {
              e.printStackTrace();
              TraceLog.e(TAG, e.toString());
          }
      }
  ```

