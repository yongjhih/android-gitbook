# Test - robolectric

![robolectric](http://robolectric.org/images/robolectric-stacked-3f7ad42c.png)

robolectric 提供 Android 相關的實體，例如：`Activity`, `Context` 以便離線測試。

```java
@RunWith(RobolectricTestRunner.class)
public class MyActivityTest {

  @Test
  public void clickingButton_shouldChangeResultsViewText() throws Exception {
    MyActivity activity = Robolectric.setupActivity(MyActivity.class);

    Button button = (Button) activity.findViewById(R.id.button);
    TextView results = (TextView) activity.findViewById(R.id.results);

    button.performClick();
    assertThat(results.getText().toString()).isEqualTo("Robolectric Rocks!");
  }
}
```