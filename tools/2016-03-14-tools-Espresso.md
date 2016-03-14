Espresso Testing Scenarios
===========

0X00: Setting up Espresso Testing Environment
------------
```groovy
	android{
		...
		defaultConfig{
			testInstrumentationRunner 'android.support.test.runner.AndroidJUnitRunner'
		}
		packagingOptions{
            exclude 'LICENSE.txt'
        }
		...
	}
	dependencies {
        testCompile 'junit:junit:4.12'
        androidTestCompile 'com.android.support.test:runner:0.3'
        androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2'
    }
```

0X01: Simple Interactions between Two Activities
-------------
Now, let us launch a simple testing demo between two activities.

Activity SimpleCheck is the entrance, and we test functions of two buttons: Check and Cancel. Clicking check will pass the content to the next activity WelcomeActivity, while button cancel will display other text in WelcomeActivity.

![SimpleCheck](/imgs/20160314_Espresso_01.png)
![Check_WelcomeActivity](/imgs/20160314_Espresso_02.png)
![Cancel_WelcomeActivity](/imgs/20160314_Espresso_03.png)

Then we create a class, SimpleCheckTest, to test functions of two buttons in the directory androidTest/java/com.six.utillib/ .

```java
	@RunWith(AndroidJUnit4.class)
	@LargeTest
	public class SimpleCheckTest {
		  @Rule
		  public ActivityTestRule<SimpleCheck> mActivityRule = new ActivityTestRule<>(SimpleCheck.class);
		  @Test
		  public void inputPinThenCheck(){
		  //type text in EditText with a Id of R.id.et_pwd, then close softKeyboard and click the check button.
                  onView(withId(R.id.et_pwd)).perform(typeText(TEST_PIN), closeSoftKeyboard());
                  onView(withId(R.id.btn_check)).perform(click());
		  //finally, we check the content of the TextView with the Id, R.id.tv_result, whether it matches what we want or not.                  
                  onView(withId(R.id.tv_result)).check(matches(withText(TEST_PIN)));
          }
	}
```

0X02: AdapterView && List as the data source
-------------
Here, we are going to show an example about testing AdapterViews, such as ListView, within Espresso framework.
As all we know, a ListView usually has more than one item, and those item have the same resource Id, so one of the most important thing is to identify one specific item during the test.
Overriding Espresso.onData method would be a good idea by customizing your own rule to pick the specific item out.

```java
	@RunWith(AndroidJUnit4.class)
	@LargeTest
	public class ListViewTest {
		@Rule
		public ActivityTestRule<ListViewSample> mActivityRule = new ActivityTestRule<>(ListViewSample.class);
        
        @Test
        public void itemTextClick() {
            onRow(TEXT_ITEM_30).onChildView(withId(R.id.rowContentTextView)).perform(click());
            onView(withId(R.id.selection_row_value)).check(matches(withText(TEXT_ITEM_30_SELECTED)));
        }
        
        private static DataInteraction onRow(String str) {
        //the matcher here is to find one matching entry within a Map.
            return onData(hasEntry(equalTo(ListViewSample.ROW_TEXT), is(str)));
        }
    }
```

0X03: AdapterView && JavaBean as the data source
-------------
However, sometimes, we use a JavaBean to store data rather than a list, a map or something like that. In that case, we need our own Matcher to tell Espresso how we distinguish a data item we want.

```java
	public class BeanMatcher {
	
	    public static Matcher<Object> withName(String expectedText) {
	        return withName(equalTo(expectedText));
	    }
	
	    public static Matcher<Object> withName(final Matcher<String> expectedObject) {
	        return new BoundedMatcher<Object, Student>(Student.class) {
	
	            @Override
	            public void describeTo(Description description) {
	                description.appendText("with name: " + description);
	            }
	
				//matchesSafely is very important as we set our rules to match here.
	            @Override
	            protected boolean matchesSafely(Student item) {
	                return expectedObject.matches(item.toString());
	            }
	        };
	    }
	}
```
Now, we can create an DataInteraction using BeanMatcher.

```java
	@RunWith(AndroidJUnit4.class)
	@LargeTest
	public class ListViewBeanTest {
	    private static final String LAST_ITEM_ID = "99";
	    private static final String ITEM_STRING_33 = "id: 33; name: student-33; age: 66";
	
	    @Rule
	    public ActivityTestRule<ListViewBeanSample> mActivityRule = new ActivityTestRule<>(ListViewBeanSample.class);
	
	    @Test
	    public void listScrolling() {
	        onRow(ITEM_STRING_33).check(matches(isCompletelyDisplayed()));
	    }
	
	    private DataInteraction onRow(String target) {
	        return onData(allOf(instanceOf(Student.class), BeanMatcher.withName(target)));
	    }
	}
```

0X04: AsyncTask
-------------
There are some steps to test classes that extend AsyncTask.
+ Step one:  add a static boolean flag within onPostExecute method to check whether this task is finished.
```java
    private class GetAsyncTask extends AsyncTask<Void, Integer, String> {

        @Override
        protected String doInBackground(Void... params) {
            isFinished = false;
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                return null;
            }
            return "OK";
        }

        @Override
        protected void onPostExecute(String result) {
            super.onPostExecute(result);
            isFinished = true;
            tvResult.setText("Six: " + result);
        }
    }
```

+ Step two: create a class to implement IdlingResource interface.

```java
	public class AsyncIdlingRes implements IdlingResource{
	    private ResourceCallback resourceCallback;
	
	    @Override
	    public String getName() {
	        return AsyncIdlingRes.class.getSimpleName();
	    }
	
	    @Override
	    public boolean isIdleNow() {
	        boolean isIdleNow = AsyncTaskActivity.isFinished;
	        if(isIdleNow){
	            resourceCallback.onTransitionToIdle();
	        }
	        return isIdleNow;
	    }
	
	    @Override
	    public void registerIdleTransitionCallback(ResourceCallback callback) {
	        this.resourceCallback = callback;
	    }
	}
```

+ Step three: write the testing class.

```java
	@RunWith(AndroidJUnit4.class)
	@LargeTest
	public class AsyncTest {
	    @Rule
	    public ActivityTestRule<AsyncTaskActivity> mActRule = new ActivityTestRule<>(AsyncTaskActivity.class);
	
	    @Test
	    public void async(){
	        onView(withId(R.id.btn_get)).perform(click());
	
	        IdlingPolicies.setMasterPolicyTimeout(6, TimeUnit.MILLISECONDS);
	        IdlingPolicies.setIdlingResourceTimeout(10, TimeUnit.MILLISECONDS);
	        IdlingResource idlingResource = new AsyncIdlingRes();
	        Espresso.registerIdlingResources(idlingResource);
	
	        onView(withId(R.id.tv_result)).check(matches(withText(RESULT)));
	
	        Espresso.unregisterIdlingResources(idlingResource);
	    }
	}
```

All right, four scenarios of espresso are above, and click [here](https://github.com/hellenxu/Utils/tree/master/utillib/src/androidTest/java/com/six/utillib/testing) to see the whole sample code.
