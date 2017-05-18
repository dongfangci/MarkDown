#JUnit单元测试#
##目录结构##
 JUnit 的最佳实践：单元测试代码和被测试代码使用一样的包，不同的目录。  
您打算把单元测试代码放在什么地方呢？把它和被测试代码混在一起，这显然会照成混乱，因为单元测试代码是不会出现在最终产品中的。建议您分别为单元测试代码与被测试代码创建单独的目录，并保证测试代码和被测试代码使用相同的包名。这样既保证了代码的分离，同时还保证了查找的方便。
##被测方法##
测试类 TestWordDealUtil 之所以使用“Test”开头，完全是为了更好的区分测试类与被测试类。测试方法 wordFormat4DBNormal 调用执行被测试方法 WordDealUtil.wordFormat4DB，以判断运行结果是否达到设计预期的效果。需要注意的是，测试方法 wordFormat4DBNormal 需要按照一定的规范书写：
测试方法必须使用注解 org.junit.Test 修饰。
测试方法必须使用 public void 修饰，而且不能带有任何参数。
assertEquals 是由 JUnit 提供的一系列判断测试结果是否正确的静态断言方法（位于类 org.junit.Assert 中）之一，我们使用它将执行结果 result 和预期值“employee_info”进行比较，来判断测试是否成功。
##单元测试结果##
单元测试代码不是用来证明您是对的，而是为了证明您没有错。因此单元测试的范围要全面，比如对边界值、正常值、错误值得测试；对代码可能出现的问题要全面预测，而这也正是需求分析、详细设计环节中要考虑的。例如：
```
public class TestWordDealUtil { 
   ……
    // 测试 null 时的处理情况
    @Test public void wordFormat4DBNull(){ 
        String target = null; 
        String result = WordDealUtil.wordFormat4DB(target); 
        
        assertNull(result); 
    } 
    
    // 测试空字符串的处理情况
    @Test public void wordFormat4DBEmpty(){ 
        String target = ""; 
        String result = WordDealUtil.wordFormat4DB(target); 
        
        assertEquals("", result); 
    } 
 
    // 测试当首字母大写时的情况
    @Test public void wordFormat4DBegin(){ 
        String target = "EmployeeInfo"; 
        String result = WordDealUtil.wordFormat4DB(target); 
        
        assertEquals("employee_info", result); 
    } 
    
    // 测试当尾字母为大写时的情况
    @Test public void wordFormat4DBEnd(){ 
        String target = "employeeInfoA"; 
        String result = WordDealUtil.wordFormat4DB(target); 
        
        assertEquals("employee_info_a", result); 
    } 
    
    // 测试多个相连字母大写时的情况
    @Test public void wordFormat4DBTogether(){ 
        String target = "employeeAInfo"; 
        String result = WordDealUtil.wordFormat4DB(target); 
        
        assertEquals("employee_a_info", result); 
    } 
}
```
JUnit 将测试失败的情况分为两种：failure 和 error。Failure 一般由单元测试使用的断言方法判断失败引起，它表示在测试点发现了问题；而 error 则是由代码异常引起，这是测试目的之外的发现，它可能产生于测试代码本身的错误（测试代码也是代码，同样无法保证完全没有缺陷），也可能是被测试代码中的一个隐藏的 bug。
##Fixture##
何谓 Fixture ？它是指在执行一个或者多个测试方法时需要的一系列公共资源或者数据，例如测试环境，测试数据等等。在编写单元测试的过程中，您会发现在大部分的测试方法在进行真正的测试之前都需要做大量的铺垫——为设计准备 Fixture 而忙碌。这些铺垫过程占据的代码往往比真正测试的代码多得多，而且这个比率随着测试的复杂程度的增加而递增。当多个测试方法都需要做同样的铺垫时，重复代码的“坏味道”便在测试代码中弥漫开来。这股“坏味道”会弄脏您的代码，还会因为疏忽造成错误，应该使用一些手段来根除它。
JUnit 专门提供了设置公共 Fixture 的方法，同一测试类中的所有测试方法都可以共用它来初始化 Fixture 和注销 Fixture。和编写 JUnit 测试方法一样，公共 Fixture 的设置也很简单，您只需要：
使用注解 org,junit.Before 修饰用于初始化 Fixture 的方法。
使用注解 org.junit.After 修饰用于注销 Fixture 的方法。
保证这两种方法都使用 public void 修饰，而且不能带有任何参数。
遵循上面的三条原则，编写出的代码大体是这个样子：
```
// 初始化 Fixture 方法
@Before public void init(){ …… } 
 
// 注销 Fixture 方法
@After public void destroy(){ …… }
```
这样，在每一个测试方法执行之前，JUnit 会保证 init 方法已经提前初始化测试环境，而当此测试方法执行完毕之后，JUnit 又会调用 destroy 方法注销测试环境。注意是每一个测试方法的执行都会触发对公共 Fixture 的设置，也就是说使用注解 Before 或者 After 修饰的公共 Fixture 设置方法是方法级别的。这样便可以保证各个独立的测试之间互不干扰，以免其它测试代码修改测试环境或者测试数据影响到其它测试代码的准确性。
![](http://i.imgur.com/k2DPEjH.gif)
可是，这种 Fixture 设置方式还是引来了批评，因为它效率低下，特别是在设置 Fixture 非常耗时的情况下（例如设置数据库链接）。而且对于不会发生变化的测试环境或者测试数据来说，是不会影响到测试方法的执行结果的，也就没有必要针对每一个测试方法重新设置一次 Fixture。因此在 JUnit 4 中引入了类级别的 Fixture 设置方法，编写规范如下：
使用注解 org,junit.BeforeClass 修饰用于初始化 Fixture 的方法。
使用注解 org.junit.AfterClass 修饰用于注销 Fixture 的方法。
保证这两种方法都使用 public static void 修饰，而且不能带有任何参数。
类级别的 Fixture 仅会在测试类中所有测试方法执行之前执行初始化，并在全部测试方法测试完毕之后执行注销方法（图 6）。代码范本如下：
```
// 类级别 Fixture 初始化方法
@BeforeClass public static void dbInit(){ …… } 
    
// 类级别 Fixture 注销方法
    @AfterClass public static void dbClose(){ …… }
```
![](http://i.imgur.com/j6GLkdo.gif)
##异常以及时间测试##
注解 org.junit.Test 中有两个非常有用的参数：expected 和 timeout。参数 expected 代表测试方法期望抛出指定的异常，如果运行测试并没有抛出这个异常，则 JUnit 会认为这个测试没有通过。这为验证被测试方法在错误的情况下是否会抛出预定的异常提供了便利。举例来说，方法 supportDBChecker 用于检查用户使用的数据库版本是否在系统的支持的范围之内，如果用户使用了不被支持的数据库版本，则会抛出运行时异常 UnsupportedDBVersionException。测试方法 supportDBChecker 在数据库版本不支持时是否会抛出指定异常的单元测试方法大体如下：
```
@Test(expected=UnsupportedDBVersionException.class) 
    public void unsupportedDBCheck(){ 
       ……}
```
注解 org.junit.Test 的另一个参数 timeout，指定被测试方法被允许运行的最长时间应该是多少，如果测试方法运行时间超过了指定的毫秒数，则 JUnit 认为测试失败。这个参数对于性能测试有一定的帮助。例如，如果解析一份自定义的 XML 文档花费了多于 1 秒的时间，就需要重新考虑 XML 结构的设计，那单元测试方法可以这样来写：
```
@Test(timeout=1000) 
    public void selfXMLReader(){ 
       ……
}
```
##忽略测试方法##
JUnit 提供注解 org.junit.Ignore 用于暂时忽略某个测试方法，因为有时候由于测试环境受限，并不能保证每一个测试方法都能正确运行。例如下面的代码便表示由于没有了数据库链接，提示 JUnit 忽略测试方法 unsupportedDBCheck：
```
@ Ignore(“db is down”) 
@Test(expected=UnsupportedDBVersionException.class) 
    public void unsupportedDBCheck(){ 
       ……}
```
是一定要小心。注解 org.junit.Ignore 只能用于暂时的忽略测试，如果需要永远忽略这些测试，一定要确认被测试代码不再需要这些测试方法，以免忽略必要的测试点。
##测试运行器##
又一个新概念出现了——测试运行器，JUnit 中所有的测试方法都是由它负责执行的。JUnit 为单元测试提供了默认的测试运行器，但 JUnit 并没有限制您必须使用默认的运行器。相反，您不仅可以定制自己的运行器（所有的运行器都继承自 org.junit.runner.Runner），而且还可以为每一个测试类指定使用某个具体的运行器。指定方法也很简单，使用注解 org.junit.runner.RunWith 在测试类上显式的声明要使用的运行器即可：
```
@RunWith(CustomTestRunner.class) 
 public class TestWordDealUtil { 
……
 }
```
显而易见，如果测试类没有显式的声明使用哪一个测试运行器，JUnit 会启动默认的测试运行器执行测试类（比如上面提及的单元测试代码）。一般情况下，默认测试运行器可以应对绝大多数的单元测试要求；当使用 JUnit 提供的一些高级特性（例如即将介绍的两个特性）或者针对特殊需求定制 JUnit 测试方式时，显式的声明测试运行器就必不可少了。
##测试套件##
在实际项目中，随着项目进度的开展，单元测试类会越来越多，可是直到现在我们还只会一个一个的单独运行测试类，这在实际项目实践中肯定是不可行的。为了解决这个问题，JUnit 提供了一种批量运行测试类的方法，叫做测试套件。这样，每次需要验证系统功能正确性时，只执行一个或几个测试套件便可以了。测试套件的写法非常简单，您只需要遵循以下规则：
创建一个空类作为测试套件的入口。
使用注解 org.junit.runner.RunWith 和 org.junit.runners.Suite.SuiteClasses 修饰这个空类。
将 org.junit.runners.Suite 作为参数传入注解 RunWith，以提示 JUnit 为此类使用套件运行器执行。
将需要放入此测试套件的测试类组成数组作为注解 SuiteClasses 的参数。
保证这个空类使用 public 修饰，而且存在公开的不带有任何参数的构造函数。
```
package com.ai92.cooljunit; 
 
 import org.junit.runner.RunWith; 
 import org.junit.runners.Suite; 
……
 
 /** 
 * 批量测试 工具包 中测试类
 * @author Ai92 
 */ 
 @RunWith(Suite.class) 
 @Suite.SuiteClasses({TestWordDealUtil.class}) 
 public class RunAllUtilTestsSuite { 
 }
```
上例代码中，我们将前文提到的测试类 TestWordDealUtil 放入了测试套件 RunAllUtilTestsSuite 中，在 Eclipse 中运行测试套件，可以看到测试类 TestWordDealUtil 被调用执行了。测试套件中不仅可以包含基本的测试类，而且可以包含其它的测试套件，这样可以很方便的分层管理不同模块的单元测试代码。但是，您一定要保证测试套件之间没有循环包含关系，否则无尽的循环就会出现在您的面前……。
##参数化测试##
为了保证单元测试的严谨性，我们模拟了不同类型的字符串来测试方法的处理能力，为此我们编写大量的单元测试方法。可是这些测试方法都是大同小异：代码结构都是相同的，不同的仅仅是测试数据和期望值。有没有更好的方法将测试方法中相同的代码结构提取出来，提高代码的重用度，减少复制粘贴代码的烦恼？在以前的 JUnit 版本上，并没有好的解决方法，而现在您可以使用 JUnit 提供的参数化测试方式应对这个问题。
参数化测试的编写稍微有点麻烦（当然这是相对于 JUnit 中其它特性而言）：
为准备使用参数化测试的测试类指定特殊的运行器 org.junit.runners.Parameterized。
为测试类声明几个变量，分别用于存放期望值和测试所用数据。
为测试类声明一个使用注解 org.junit.runners.Parameterized.Parameters 修饰的，返回值为 java.util.Collection 的公共静态方法，并在此方法中初始化所有需要测试的参数对。
为测试类声明一个带有参数的公共构造函数，并在其中为第二个环节中声明的几个变量赋值。
编写测试方法，使用定义的变量作为参数进行测试。
我们按照这个标准，重新改造一番我们的单元测试代码：
```
package com.ai92.cooljunit; 
 
import static org.junit.Assert.assertEquals; 
import java.util.Arrays; 
import java.util.Collection; 
 
import org.junit.Test; 
import org.junit.runner.RunWith; 
import org.junit.runners.Parameterized; 
import org.junit.runners.Parameterized.Parameters; 
 
@RunWith(Parameterized.class) 
public class TestWordDealUtilWithParam { 
 
        private String expected; 
     
        private String target; 
     
        @Parameters 
        public static Collection words(){ 
            return Arrays.asList(new Object[][]{ 
                {"employee_info", "employeeInfo"},      // 测试一般的处理情况
                {null, null},                           // 测试 null 时的处理情况
                {"", ""},                               // 测试空字符串时的处理情况
                {"employee_info", "EmployeeInfo"},      // 测试当首字母大写时的情况
                {"employee_info_a", "employeeInfoA"},   // 测试当尾字母为大写时的情况
                {"employee_a_info", "employeeAInfo"}    // 测试多个相连字母大写时的情况
            }); 
        } 
     
         /** 
         * 参数化测试必须的构造函数
         * @param expected     期望的测试结果，对应参数集中的第一个参数
         * @param target     测试数据，对应参数集中的第二个参数
         */ 
        public TestWordDealUtilWithParam(String expected , String target){ 
            this.expected = expected; 
            this.target = target; 
        } 
     
         /** 
         * 测试将 Java 对象名称到数据库名称的转换
         */ 
        @Test public void wordFormat4DB(){ 
            assertEquals(expected, WordDealUtil.wordFormat4DB(target)); 
        } 
```
很明显，代码瘦身了。在静态方法 words 中，我们使用二维数组来构建测试所需要的参数列表，其中每个数组中的元素的放置顺序并没有什么要求，只要和构造函数中的顺序保持一致就可以了。现在如果再增加一种测试情况，只需要在静态方法 words 中添加相应的数组即可，不再需要复制粘贴出一个新的方法出来了。
##JUnit和Ant##
随着项目的进展，项目的规模在不断的膨胀，为了保证项目的质量，有计划的执行全面的单元测试是非常有必要的。但单靠 JUnit 提供的测试套件很难胜任这项工作，因为项目中单元测试类的个数在不停的增加，测试套件却无法动态的识别新加入的单元测试类，需要手动修改测试套件，这是一个很容易遗忘得步骤，稍有疏忽就会影响全面单元测试的覆盖率。
当然解决的方法有多种多样，其中将 JUnit 与构建利器 Ant 结合使用可以很简单的解决这个问题。Ant —— 备受赞誉的 Java 构建工具。它凭借出色的易用性、平台无关性以及对项目自动测试和自动部署的支持，成为众多项目构建过程中不可或缺的独立工具，并已经成为事实上的标准。Ant 内置了对 JUnit 的支持，它提供了两个 Task：junit 和 junitreport，分别用于执行 JUnit 单元测试和生成测试结果报告。使用这两个 Task 编写构建脚本，可以很简单的完成每次全面单元测试的任务。
不过，在使用 Ant 运行 JUnit 之前，您需要稍作一些配置。打开 Eclipse 首选项界面，选择 Ant -> Runtime 首选项（见 图 7），将 JUnit 4.1 的 JAR 文件添加到 Classpath Tab 页中的 Global Entries 设置项里。记得检查一下 Ant Home Entries 设置项中的 Ant 版本是否在 1.7.0 之上，如果不是请替换为最新版本的 Ant JAR 文件。
![](http://i.imgur.com/R2bXBUY.jpg)
剩下的工作就是要编写 Ant 构建脚本 build.xml。虽然这个过程稍嫌繁琐，但这是一件一劳永逸的事情。现在我们就把前面编写的测试用例都放置到 Ant 构建脚本中执行，为项目 coolJUnit 的构建脚本添加一下内容：
```
<?xml version="1.0"?> 
<!-- ============================================= 
    auto unittest task    
    ai92                                                                
    ========================================== -->
<project name="auto unittest task" default="junit and report" basedir="."> 
 
        <property name="output folder" value="bin"/> 
 
        <property name="src folder" value="src"/> 
    
        <property name="test folder" value="testsrc"/> 
    
        <property name="report folder" value="report" /> 
 
        <!-- - - - - - - - - - - - - - - - - - 
         target: test report folder init                      
        - - - - - - - - - - - - - - - - - -->
        <target name="test init"> 
            <mkdir dir="${report folder}"/> 
        </target> 
    
        <!-- - - - - - - - - - - - - - - - - - 
         target: compile                      
        - - - - - - - - - - - - - - - - - -->
        <target name="compile"> 
            <javac srcdir="${src folder}" destdir="${output folder}" /> 
            <echo>compilation complete!</echo> 
        </target> 
 
        <!-- - - - - - - - - - - - - - - - - - 
         target: compile test cases                      
        - - - - - - - - - - - - - - - - - -->
        <target name="test compile" depends="test init"> 
            <javac srcdir="${test folder}" destdir="${output folder}" /> 
            <echo>test compilation complete!</echo> 
        </target> 
    
        <target name="all compile" depends="compile, test compile"> 
        </target> 
    
        <!-- ======================================== 
         target: auto test all test case and output report file                      
        ===================================== -->
        <target name="junit and report" depends="all compile"> 
            <junit printsummary="on" fork="true" showoutput="true"> 
                <classpath> 
                    <fileset dir="lib" includes="**/*.jar"/> 
                    <pathelement path="${output folder}"/> 
                </classpath> 
                <formatter type="xml" /> 
                <batchtest todir="${report folder}"> 
                    <fileset dir="${output folder}"> 
                        <include name="**/Test*.*" /> 
                    </fileset> 
                </batchtest> 
            </junit> 
            <junitreport todir="${report folder}"> 
                <fileset dir="${report folder}"> 
                    <include name="TEST-*.xml" /> 
                </fileset> 
                <report format="frames" todir="${report folder}" /> 
            </junitreport> 
        </target> 
</project>
```
Target junit report 是 Ant 构建脚本中的核心内容，其它 target 都是为它的执行提供前期服务。Task junit 会寻找输出目录下所有命名以“Test”开头的 class 文件，并执行它们。紧接着 Task junitreport 会将执行结果生成 HTML 格式的测试报告（图 8）放置在“report folder”下。
为整个项目的单元测试类确定一种命名风格。不仅是出于区分类别的考虑，这为 Ant 批量执行单元测试也非常有帮助，比如前面例子中的测试类都已“Test”打头，而测试套件则以“Suite”结尾等等。
现在执行一次全面的单元测试变得非常简单了，只需要运行一下 Ant 构建脚本，就可以走完所有流程，并能得到一份详尽的测试报告。您可以在 Ant 在线手册中获得上面提及的每一个 Ant 内置 task 的使用细节。

























