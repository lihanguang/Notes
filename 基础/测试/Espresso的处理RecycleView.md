#### RecyclerView的特殊性

RecyclerView不继承于AdapterView，所以无法使用AdapterView的相关API，Espresso提供了专门的RecyclerView相关的API，但是所提供的API不够健全。

#### 获取RecyclerView的itemview中任意View

```java
public class RecyclerViewMatcher {
  private final int recyclerViewId;

  public RecyclerViewMatcher(int recyclerViewId) {
    this.recyclerViewId = recyclerViewId;
  }

  public Matcher<View> atPosition(final int position) {
    return atPositionOnView(position, -1);
  }

  public Matcher<View> atPositionOnView(final int position, final int targetViewId) {
	// 自定义一个Matcher来进行获取
    return new TypeSafeMatcher<View>() {
      Resources resources = null;
      View childView;

      public void describeTo(Description description) {
        String idDescription = Integer.toString(recyclerViewId);
        if (this.resources != null) {
          try {
            idDescription = this.resources.getResourceName(recyclerViewId);
          } catch (Resources.NotFoundException var4) {
            idDescription = String.format("%s (resource name not found)",
                    new Object[] { Integer.valueOf
                            (recyclerViewId) });
          }
        }

        description.appendText("with id: " + idDescription);
      }
	 // 进行View的匹配
      public boolean matchesSafely(View view) {

        this.resources = view.getResources();

        if (childView == null) {
          RecyclerView recyclerView =
                  (RecyclerView) view.getRootView().findViewById(recyclerViewId);
          if (recyclerView != null && recyclerView.getId() == recyclerViewId) {
            childView = recyclerView.findViewHolderForAdapterPosition(position).itemView;
          }
          else {
            return false;
          }
        }

        if (targetViewId == -1) {
          return view == childView;
        } else {
          View targetView = childView.findViewById(targetViewId);
          return view == targetView;
        }

      }
    };
  }
}
```

