# **Findbugs custom detectors**

# **1.**  **Development environment **

- **IntelliJ IDEA 2019.3.4**   
- **jdk1.8**

- **FindBugs-IDEA**
- **Bytecode viewer**

# **2.**  **Findbugs  custom detectors **

-    **detect  hardcode about the filename**
-    **detect hardcode about const in condition**
-    **detect the name about boolean method**

​**for example** 

   ```java

 /**************detect  hardcode about the filename********************/

//this is a  hardcode about the filename   
chessImage = Toolkit.getDefaultToolkit().getImage("F:/Image/chess/3.png");

//should change soft code 
chessImage = Toolkit.getDefaultToolkit().getImage(filename);

 /****************detect hardcode about const in condition**********************/

//the condition in for or if or while 
//suggest use the soft code 
 for(int i=0;i<100;i++){
     System.out.println(1);
 }
 if(a<200){
     System.out.println(1);      
 }
 while(e<100){
     System.out.println(e);
     e++;
 }

//suggest to change 
 for(int i=0;i<SIZE;i++){
     System.out.println(1);
 }
 if(a<CONDTION_SIZE){
     System.out.println(1);      
 }
 while(e<MAXSIZE){
     System.out.println(e);
     e++;
 }


 /******************detect the name about boolean method************************/    

//bad name about boolean method
 private boolean empty(){
      return true;
 }
//suggest to begin with "is" or "Is" 
 private boolean isEmpty(){
      return true;
 }
 private boolean IsEmpty(){
      return true;
 }

   ```

# **3.**  **Custom detector development process**



##   **detect  hardcode about the filename** 

  

 >**1，find the hardcode about the filename**
 >
 >
 >
 >![1587531768716](https://github.com/mybanking/picture/blob/master/1587531768716.png?raw=true)
 >
 >
 >
 >2，complie the class ，then view Byte encoding about it
 >
 >
 >
 >![1587531660871](https://github.com/mybanking/picture/blob/master/1587531660871.png?raw=true)
 >
 >
 >
 >3，code the detector
 >
 >​                         **Toolkit.getDefaultToolkit().getImage();**
 >
 >​                          **imageicon.setIcon(new ImageIcon());**
 >
 >detect the filename is or not to be hard code 
 >
 >code the detector HardNamegetImage
 >
 >```java
 >package com.h3xstream.findsecbugs.kdy;//path
 >
 >import edu.umd.cs.findbugs.BugInstance; 
 >import edu.umd.cs.findbugs.BugReporter; 
 >import edu.umd.cs.findbugs.bcel.OpcodeStackDetector;
 >import org.apache.bcel.classfile.Code; 
 >
 >public class HardNamegetImage extends OpcodeStackDetector {
 >
 >    BugReporter bugReporter;
 >    public HardNamegetImage(BugReporter bugReporter) {
 >        this.bugReporter = bugReporter;
 >    }
 >     /**
 >     * visit方法，在每次进入字节码方法的时候调用 在每次进入新方法的时候清空标志位
 >     */
 >    @Override
 >    public void visit(Code obj) {
 >        super.visit(obj);
 >    }
 >    /**
 >     * 每扫描一条字节码就会进入sawOpcode方法
 >
 >     * @param seen
 >
 >     *字节码的枚举值
 >
 >     */
 >
 >    public void sawOpcode(int seen) {
 >        //判断seen值   判断类  判断方法名  判断下一个操作是不是 LDC
 >        if (seen == INVOKESTATIC) {
 >            if (getClassConstantOperand().equals("java/awt/Toolkit")
 >                    && getNameConstantOperand().equals("getDefaultToolkit")
 >                       && getSigConstantOperand().equals("()Ljava/awt/Toolkit;")
 >                        && getNextOpcode()==LDC){
 >
 >                BugInstance bug = new BugInstance(this, "HardNamegetImage",
 >                        NORMAL_PRIORITY).addClassAndMethod(this).addSourceLine(
 >                        this, getPC());
 >                bugReporter.reportBug(bug);
 >            }
 >        }
 >         //判断seen值   判断类  判断方法名  判断上一个操作是不是 LDC
 >        else if (seen == INVOKESPECIAL) {
 >            if (getClassConstantOperand().equals("javax/swing/ImageIcon")
 >                    && getNameConstantOperand().equals("<init>")
 >                    && getSigConstantOperand().equals("(Ljava/lang/String;)V")
 >                    && getPrevOpcode(1)==LDC){
 >
 >                BugInstance bug = new BugInstance(this, "HardNamegetImage",
 >                        NORMAL_PRIORITY).addClassAndMethod(this).addSourceLine(
 >                        this, getPC());
 >                bugReporter.reportBug(bug);
 >            }
 >        }
 >
 >
 >    }
 >
 >}
 >
 >```
 >
 >4,configure findbug.xml  messages.xml  
 >
 >append this to  findbug.xml
 >
 >```java
 ><Detector class="com.h3xstream.findsecbugs.kdy.HardNamegetImage"  speed="fast" reports=" HardNamegetImage" hidden="false" />
 >    
 ><BugPattern abbrev="HardNamegetImage" type="HardNamegetImage" category="PERFORMANCE" />
 >```
 >
 >![1587532777918](https://github.com/mybanking/picture/blob/master/1587532777918.png?raw=true)
 >
 >append this to  messages.xml 
 >
 >```java
 >    <!--HardNamegetImage-->
 >    <Detector class="com.h3xstream.findsecbugs.kdy.HardNamegetImage">
 >
 >        <Details>
 >            <![CDATA[<p>Hardcoded of file or varies <p>suggest use soft code]]>
 >        </Details>
 >
 >    </Detector>
 >
 >    <BugPattern type="HardNamegetImage">
 >        <ShortDescription>Hardcoded of file or varies</ShortDescription>
 >        <LongDescription>{1}suggest use soft code of filename or varies</LongDescription>
 >
 >        <Details>
 >            <![CDATA[<p>suggest use soft code</p> ]]>
 >        </Details>
 >    </BugPattern>
 >
 >    <BugCode abbrev="HardNamegetImage">Hardcoded of file or varies</BugCode>
 >```
 >
 >![1587532930532](https://github.com/mybanking/picture/blob/master/1587532930532.png?raw=true)
 >
 >5,put  HardNamegetImage.class  and findbugs.xml and messages.xml to  
 >
 >the  findsecbugs-plugin-1.10.1.jar  ,  then restart the IDEA 
 >
 >6,begin testing
 >
 >![1587533467145](https://github.com/mybanking/picture/blob/master/1587533467145.png?raw=true)
 >
 >



##   **detect hardcode about const in condition**

>The process is similar to the above, and directly code
>
>Bytecode view 
>
>![1587534360541](https://github.com/mybanking/picture/blob/master/1587534360541.png?raw=true)
>
>code the detector Constofcondition
>
>```java
>package com.h3xstream.findsecbugs.kdy;
>
>import edu.umd.cs.findbugs.BugInstance;
>import edu.umd.cs.findbugs.BugReporter;
>import edu.umd.cs.findbugs.bcel.OpcodeStackDetector;
>import org.apache.bcel.classfile.Code;
>
>public class Constofcondition extends OpcodeStackDetector {
>
>    BugReporter bugReporter;
>    public Constofcondition(BugReporter bugReporter) {
>        this.bugReporter = bugReporter;
>    }
>    @Override
>
>    public void visit(Code obj) {
>        super.visit(obj);
>    }
>
>    public void sawOpcode(int seen) {
>
>        if(seen==IF_ICMPGE &&
>                (getPrevOpcode(1)==BIPUSH||getPrevOpcode(1)==SIPUSH||getPrevOpcode(1)<=ICONST_5)){
>            BugInstance bug = new BugInstance(this, "Constofcondition",
>                    NORMAL_PRIORITY).addClassAndMethod(this).addSourceLine(
>                    this, getPC());
>            bugReporter.reportBug(bug);
>        }
>    }
>}
>
>```
>
>findbugs.xml
>
>```java
>    <Detector class="com.h3xstream.findsecbugs.kdy.Constofcondition"  speed="fast" reports=" HardNamegetImage" hidden="false" />
>
>
>    <BugPattern abbrev="Constofcondition" type="Constofcondition" category="PERFORMANCE" />
>```
>
>messages.xml
>
>```java
><!--Constofcondition-->
>
>    <Detector class="com.h3xstream.findsecbugs.kdy.Constofcondition">
>
>        <Details>
>            <![CDATA[<p>Const in condition <p>suggest not to use const in condition]]>
>        </Details>
>
>    </Detector>
>
>    <BugPattern type="Constofcondition">
>        <ShortDescription>Const in condition </ShortDescription>
>        <LongDescription>{1}suggest not to use const in condition</LongDescription>
>
>        <Details>
>            <![CDATA[<p>Const in condition</p> ]]>
>        </Details>
>    </BugPattern>
>
>    <BugCode abbrev="Constofcondition">Const in condition</BugCode>
>```
>
>testing
>
>![1587534988385](https://github.com/mybanking/picture/blob/master/1587534988385.png?raw=true)
>
>



##  **detect the name about boolean method**

>The process is similar to the above, and directly code
>
>code the detector BooleanMethodName
>
>```java
>package com.h3xstream.findsecbugs.kdy;
>
>import edu.umd.cs.findbugs.BugInstance;
>import edu.umd.cs.findbugs.BugReporter;
>import edu.umd.cs.findbugs.MethodAnnotation;
>import edu.umd.cs.findbugs.bcel.PreorderDetector;
>import org.apache.bcel.classfile.Method;
>
>public class BooleanMethodName extends PreorderDetector{
>
>    BugReporter bugReporter;
>    public BooleanMethodName(BugReporter bugReporter) {
>        this.bugReporter = bugReporter;
>    }
>
>
>    @Override
>    public void visit(Method me){
>        if(("Boolean".equals(me.getReturnType().toString())||"boolean".equals(me.getReturnType().toString()))
>                && me.getName().length()>=2
>                && (!("is".equals(me.getName().substring(0,2)))&&!("Is".equals(me.getName().substring(0,2)))) )
>        {
>            bugReporter.reportBug(new BugInstance(this, "BooleanMethodName", NORMAL_PRIORITY).
>                    addClass(getDottedClassName()).addMethod(MethodAnnotation.fromVisitedMethod(this)));
>        }
>    }
>
>
>}
>
>```
>
>findbugs.xml
>
>```java
>    <Detector class="com.h3xstream.findsecbugs.kdy.BooleanMethodName"  speed="fast" reports="BooleanMethodName" hidden="false" />
>        <BugPattern abbrev="BooleanMethodName" type="BooleanMethodName" category="PERFORMANCE" />
>```
>
>messages.xml
>
>```java
><!--   BooleanMethodName -->
>
>    <Detector class="com.h3xstream.findsecbugs.kdy.BooleanMethodName">
>
>        <Details>
>            <![CDATA[<p>suggest the name of boolean type begins with 'is' or 'Is'<p>suggest the name of boolean type begins with 'is' or 'Is' ]]>
>        </Details>
>
>    </Detector>
>    <BugPattern type="BooleanMethodName">
>        <ShortDescription>suggest the name of boolean type begins with 'is' or 'Is'</ShortDescription>
>        <LongDescription>{1}suggest the name of boolean type begins with 'is' or 'Is'</LongDescription>
>
>        <Details>
>            <![CDATA[<p>suggest the name of boolean type begins with 'is' or 'Is'</p> ]]>
>        </Details>
>    </BugPattern>
>
>    <BugCode abbrev="BooleanMethodName">Boolean types are named starting with 'is' or 'Is'</BugCode>
>```
>
>testing
>
>![1587535332150](https://github.com/mybanking/picture/blob/master/1587535332150.png?raw=true)
>
>



