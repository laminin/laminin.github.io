---
layout:     post
title:      Android junit tests with Jenkins and Github
date:       2016-05-20 20:11:04
author:     Franklin
summary:    An article about how to write simple junit test cases and setup a jenkin server locally to run the test cases.
categories: junit github jenkins android
#thumbnail:  heart
tags:
 - andorid
 - junit
 - jenkins
---

## steps:

### 1. Create an android project.

### 2. Create MathUtils.java(sample class, we will write test cases for this class) and write functions to add, subtract, multiply, divide two numbers.

 ![desk](http://i.imgur.com/7QO1I4Y.png)


  MathUtils.java should be something like
  {% highlight java %}
  public class MathUtils {
    public int add (int a, int b){
        return a + b;
    }
    public int subtract (int a, int b){
        return a - b;
    }
    public int multiply (int a, int b){
        return a * b;
    }
    public int divide (int a, int b){
        return a / b;
    }
  }
  {% endhighlight %}


### 3. Introduction to local junit test cases in Android.
  [Local junit tests][0] are tests that run on your local machine, with out needing access to the Android framework or an Android device. If your unit test has no dependencies or only has simple dependencies on Android you should run your test on a local development machine. This testing approach is efficient because it helps you avoid the overhead of loading the target app and unit test code onto a physical device or emulator every time your test is run. Consequently, the execution time for running unit test is greatly reduced. For more information please read [this][1].


### 4. Write test cases for the MathUtils methods with junit and data providers.
  Open MathUtils class and follow the steps

    a. Right click -> Go To -> Test

    b. Click Create New Test ...

    c. select all the methods ( add(a:int, b:int):int, subtract(a:int, b:int):int, multiply(a:int,
      b:int):int, divide(a:int, b:int):int )

    d. click ok

    e. Choose Destination Directory with is under ../app/src/test/java/com/...

    f. click ok

    g. Now navigate to app/src/test/java/.../MathUtilsTest

![desk](http://i.imgur.com/htsupbJ.png?1)

![desk](http://i.imgur.com/mpKgZc5.png?1)

![desk](http://i.imgur.com/FdoJYJi.png?1)

  you should be able to see all the test methods now. It should be something like

  {% highlight java %}
  public class MathUtilsTest {
    @Test
    public void add() throws Exception {

    }
    @Test
    public void subtract() throws Exception {

    }
    @Test
    public void multiply() throws Exception {

    }
    @Test
    public void divide() throws Exception {

    }
  }
  {% endhighlight %}



In order to add junit dataprovider, add the following dependency in build.gradle file.

  {% highlight java %}
  dependencies {
    testCompile 'junit:junit:4.12'
    testCompile group: 'com.tngtech.java', name: 'junit-dataprovider', version: '1.10.0'
  }
  {% endhighlight %}


Create MathUtilsDataProvider class inside app/src/test/java/..../

Create *testAddData()* method (name can be anything) to return data to the test cases with *@DataProvider* annotation

  {% highlight java %}
  @DataProvider
  public static Object[][] testAddData(){
    int data1[] = {1, 2}; // {input1, input2}
    int data2[] = {5, 2};
    int data3[] = {5, 5};
    return new Object[][] {
      {data1, 3}, // {input collection, output}
      {data2, 7},
      {data3, 11} // failure test case.
    };
  }
  {% endhighlight %}

in the data[] array data\[0], data\[1] are input.

in the return object array first element is the input and the second is expected output.

Similarly add create data provider for subtraction, multiplication, division.

Now *MathUtilsDataProvider* should look something like this.

  {% highlight java %}
  class MathUtilsDataProviders {
      @DataProvider
      public static Object[][] testAddData(){
          int data1[] = {1, 2}; // {input1, input2}
          int data2[] = {5, 2};
          int data3[] = {5, 5};
          return new Object[][] {
                  {data1, 3}, // {input collection, output}
                  {data2, 7},
                  {data3, 11}
          };
      }

      @DataProvider
      public static Object[][] testSubtractData(){
          int data1[] = {3, 2};
          int data2[] = {5, 2};
          int data3[] = {5, 5};
          return new Object[][] {
                  {data1, 1},
                  {data2, 3},
                  {data3, 1}
          };
      }

      @DataProvider
      public static Object[][] testMultiplyData(){
          int data1[] = {1, 2};
          int data2[] = {5, 2};
          int data3[] = {5, 5};
          return new Object[][] {
                  {data1, 2},
                  {data2, 10},
                  {data3, 20}
          };
      }

      @DataProvider
      public static Object[][] testDivideData(){
          int data1[] = {2, 1};
          int data2[] = {6, 2};
          int data3[] = {5, 5};
          return new Object[][] {
                  {data1, 2},
                  {data2, 3},
                  {data3, 0}
          };
      }
  }
  {% endhighlight %}

  How the Test cases take the data from data provider ?

  Add *@RunWith(DataProviderRunner.class)* annotation above class - This allows to run the test method with parameter.

  Add the following annotation after @Test annotation on every method of MathUtilsTest

  {% highlight java %}
  @UseDataProvider(value = "testAddData", location = MathUtilsDataProvider.class)
  {% endhighlight %}

  here testAddData is the *DataProvider's* method name.

  We need MathUtils object to test the add, subtract, multiply and divide methods.
  Create a @Before method to setup mathUtils object as follows.

  {% highlight java %}
  private MathUtils mathUtils;

  @Before
  public void setup(){
      mathUtils = new MathUtils();
  }
  {% endhighlight %}

  Now modify the add Test as follows.

  {% highlight java %}
  @Test
  @UseDataProvider(value = "testAddData", location = MathUtilsDataProviders.class)
  public void add(int inputData[], int expectedOutput) throws Exception {
      assertTrue(mathUtils.add(inputData[0], inputData[1]) == expectedOutput);
  }
  {% endhighlight %}

  Similarly change all the other methods.

  Now your MathUtilsTest should look something like this.

  {% highlight java %}
  @RunWith(DataProviderRunner.class)
  public class MathUtilsTest {

    private MathUtils mathUtils;

    @Before
    public void setup(){
        mathUtils = new MathUtils();
    }

    @Test
    @UseDataProvider(value = "testAddData", location = MathUtilsDataProviders.class)
    public void add(int inputData[], int expectedOutput) throws Exception {
        assertTrue(mathUtils.add(inputData[0], inputData[1]) == expectedOutput);
    }

    @Test
    @UseDataProvider(value = "testSubtractData", location = MathUtilsDataProviders.class)
    public void subtract(int inputData[], int expectedOutput) throws Exception {
        assertTrue(mathUtils.subtract(inputData[0], inputData[1]) == expectedOutput);
    }

    @Test
    @UseDataProvider(value = "testMultiplyData", location = MathUtilsDataProviders.class)
    public void multiply(int inputData[], int expectedOutput) throws Exception {
        assertTrue(mathUtils.multiply(inputData[0], inputData[1]) == expectedOutput);
    }

    @Test
    @UseDataProvider(value = "testDivideData", location = MathUtilsDataProviders.class)
    public void divide(int inputData[], int expectedOutput) throws Exception {
        assertTrue(mathUtils.divide(inputData[0], inputData[1]) == expectedOutput);
    }

  }
  {% endhighlight %}

  ![desk](http://i.imgur.com/lkAfW5v.png?1)

Run the test cases locally by clicking the arrow button on each method or on class name or by right clicking the MathUtilsTest -> RunMathUtilsTest.

![desk](http://i.imgur.com/R3engg4.png?1)

after running this we should be able to see 12 tests done: 4 failed message.

![desk](http://i.imgur.com/3v9L39N.png?1)

If you want to see test coverage use *runtestwithcoverage* option.

![desk](http://i.imgur.com/rvGB87s.png?1)

![desk](http://i.imgur.com/h0BJNgX.png?1)

### 5. Push the code to github.

  https://github.com/Franklin2412/JenkinsTestApp.git

### 6. Setting up jenkins server locally.

I use homebrew(http://brew.sh/) to install Jenkins.

  {% highlight sh %}
  brew install jenkins
  {% endhighlight %}

  This install jenkins locally. (/usr/local/opt/jenkins)

  if you want to run jenkins on every login you can use
  {% highlight sh %}
  ln -sfv /usr/local/opt/jenkins/homebrew.mxcl.jenkins.plist ~/Libray/LaunchAgents
  {% endhighlight %}
  Then run
  {% highlight sh %}
  launchctl load ~/Libray/LaunchAgents/homebrew.mxcl.jenkins.plist
  {% endhighlight %}

![desk](http://i.imgur.com/T63aDyK.png)

  At this point you should be able to see jenkins home at
  {% highlight sh %}
  http://locahost:8080/
  {% endhighlight %}
  if you want Configure ip or port open
  {% highlight sh %}
  ~/Libray/LaunchAgents/homebrew.mxcl.jenkins.plist and edit.
  {% endhighlight %}

  Install Required plugins.

  Mangage Jenkins - > Mangage Plugins -> Available Tabs  

  1. Email Extension Plugin

  2. Git plugin

  3. Gradle plugin

  4. HTML Publisher plugin


  Job Configuration Steps.

    1. To create a new job -> click _New Item_.

![desk](http://i.imgur.com/14Sbg5N.png?1)

    2. Enter name (This is your jenkins job name)

    3. Select Freestyle project and click OK

    4. Under Source Code Management

        select Git

        Fill Repository URL : https://github.com/Franklin2412/JenkinsTestApp.git

![desk](http://i.imgur.com/Xbg0GGp.png?1)

    5. Under Build

        click Add build step -> Invoke Gradle script

        Select Use Gradle Wrapper

![desk](http://i.imgur.com/ccuNOfi.png?1)

    6. Click Add build step -> Execute Shell

        Add the following command

        ./gradlew test

    7. Under Post-build Actions

        Click Add post-build action -> Publish HTML report

        Click Add

        Give following paths

        HTML directory to archive	: app/build/reports/tests/debug

        Index page[s]: index.html

        TEST-com.laminin.franklinmichael.junitjenkinstestapp.MathUtilsTest.xml

![desk](http://i.imgur.com/iZ1OXvp.png?1)

    8. Click apply and save to save the configurations.

### 8. Run the test cases using jenkins.
    Click build now button.

### 9. Generate and display test results.
    Once build is done you can see the HTML Report option inside the job
    at (http://localhost:5500/job/GitHubJunit/)

![desk](http://i.imgur.com/najDl6S.png?1)

![desk](http://i.imgur.com/cDDPb4n.png?1)

### 10. Build on every push.
    To build this job on every push

    Click Poll SCM option in Build Trigger option and set the schedule values as * * * * *
    This will keep listening github repo for any new push. if there is a new push it will automatically build the job.

![desk](http://i.imgur.com/eiSyOO3.png?1)

### 11. Send email notification.

    Click Add post-build action dropdown from Post Build Action section and select E-mail Notification

    Fill all the email address space separated.

    Click apply and save

    Click Manage jenkins then Configure

    Go to Extended E-mail Notification

    Click advance and fill the values.

    SMTP server	-> smtp.gmail.com

    Default user E-mail suffix ->	@gmail.com

    User Name	-> franklin2412@gmail.in

    Password	-> **********

    SMTP port	-> 557

    Charset	-> UTF-8

    Build the job again. If any test case fails it will generate report and send it to mails.



[0]: https://developer.android.com/training/testing/start/index.html
[1]: https://developer.android.com/training/testing/unit-testing/local-unit-tests.html
