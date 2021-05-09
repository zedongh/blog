---
layout: post
title:  "The Painless Guide to ANTLR4"
categories: antlr4 parser
date: 2019-04-30 01:22:07 +0000
---

---
## I. Parser Generator 
---
&nbsp;&nbsp;&nbsp;&nbsp;文法(Grammar)用来形式化描述语言的样子，给定文法，按照文法实现它的Parser的方式有很多，一般只有两种选择: 

1. 手写Parser
2. 使用Parser生成器(Parser Generator)

&nbsp;&nbsp;&nbsp;&nbsp;手写Parser一般采用自顶向下的递归下降<sup>[1]</sup>或算符优先级分析<sup>[2]</sup>。由于文法确定了，基本上它的Parser也就确定了，所以就诞生了Parser生成器这样的工具，为了省去手写实现的步骤。正如大家都喜欢用正则定义模式来匹配字符串而不是自己去实现状态机，因为模式串确定，对应的自动机也就是确定了，而自动机构建的过程是正则引擎的实现的。

&nbsp;&nbsp;&nbsp;&nbsp;Parser生成器的选择有很多，自顶向下(LL)，自下而上(LR)的都有，有的只有语法分析，有的词法分析和语法分析都有，实现也有很多：Flex, YACC, Bison, JFlex, PEG, JavaCC...目前从使用的易用性，工具的集成性以及个人对Java语言的偏好，Antlr4是目前不错的选择。

---
## II. Usage
---
### 1. Environment Configuration
&nbsp;&nbsp;&nbsp;&nbsp;工具只需要Intellij-IDEA就够了(Eclipse向右转...)，只用三步完成所有配置：
1. 安装`ANTLR v4 grammar plugin`
2. 创建`maven`项目
3. `pom.xml`添加如下配置：
   ```xml
    <dependencies>
        <!-- 添加运行时依赖，也可以不使用 -->
        <dependency>
            <groupId>org.antlr</groupId>
            <artifactId>antlr4-runtime</artifactId>
            <version>4.5.3</version>
        </dependency>
    </dependencies>

   <build>
        <plugins>
            <plugin>
                <groupId>org.antlr</groupId>
                <artifactId>antlr4-maven-plugin</artifactId>
                <version>4.5.3</version>
                <configuration>
                    <listener>true</listener> <!-- 为了生成XXXListener -->
                    <visitor>true</visitor>  <!-- 为了生成XXXVisitor-->
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>antlr4</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
   ```
&nbsp;&nbsp;&nbsp;&nbsp;准备工作就完成了, Easy! 当然别忘了`import changes`！

---
### 2. Calculator Project
---
&nbsp;&nbsp;&nbsp;&nbsp;从一个简单的`Calculator`项目介绍如何使用ANTLR的，当你完成`configuration`你的项目结构大体如下：
```
Calculator
  |-- .idea
  |-- src
       |-- main
            |-- java
            |-- resources
       |-- test
            |-- java
```
&nbsp;&nbsp;&nbsp;&nbsp;在`main`目录下创建一个`antlr4`目录，`maven`构建插件会在这个目录下寻找`.g4`文件，同时在该目录下创建一个`Calculator.g4`文件，这里为了生成的java的文件带有`package`，需要建立相应的目录，最终配置如下：
```
Calculator
  |-- .idea
  |-- src
       |-- main
            |-- antlr4
                  |-- com
                       |-- demo
                            |-- parser
                                  |-- Calculator.g4
            |-- java
            |-- resources
       |-- test
            |-- java
```
&nbsp;&nbsp;&nbsp;&nbsp;`Calculator.g4`给出了一个简单的Calculator的Grammar定义：
```antlr4
grammar Calculator; // 必须与文件名字一致

// 这是注释

// grammar
compileUnit : expr EOF;    // 第一个默认为grammar的开始规则

expr : expr '*' expr
     | expr '/' expr
     | expr '+' expr
     | expr '-' expr
     | INT
     ;

// lexer
INT : [0-9]+;

SPACES : [ \r\t\n] -> skip;  // 空格， 用skip指示空格跳过
```
### 3. Test Grammar
&nbsp;&nbsp;&nbsp;&nbsp;这时候可以利用前面已经安装了IDEA的`ANTLR v4 Grammar Plugin`来测试刚才定义的grammar， 鼠标放在`compileUnit`，右键，选择`Test Rule compileUnit`, 当然也可以选择测试其他的rule:

<figure class="image">
  <img src="{{site.baseurl}}/images/antlr4plugin.png" alt="test rule compileUnit">
  <figcaption>test rule compileUnit</figcaption>
</figure>

&nbsp;&nbsp;&nbsp;&nbsp;`ANTLR v4 Grammar Plugin`提供方便的可视化，方便调试grammar。

### 4. Generate Java File
&nbsp;&nbsp;&nbsp;&nbsp;测试Grammar完成后，确定下功能，就需要根据`Calculator.g4`生成相应的java文件。这就需要依靠之前配置的`maven`的`build`的`plugin`, 只需要点击`antlr4:antlr4`即可，或者使用命令行`mvn antlr4:antlr4`:

<figure class="image">
  <img src="{{site.baseurl}}/images/antlr4antlr4.png" alt="mvn antlr4:antlr4">
  <figcaption>antlr4:antlr4</figcaption>
</figure>

&nbsp;&nbsp;&nbsp;&nbsp;生成的文件位于`target`目录下的`generated-sources`下的`antlr4`，你可以注意到目录的颜色，知道这个文件已经被标记成`Sources root`了。浏览下这个目录下的文件：
```
target
  |-- generated-sources
           |-- antlr4
                 |-- com/demo/parser
                           |-- CalculatorBaseListener.java
                           |-- CalculatorBaseVisitor.java
                           |-- CalculatorLexer.java
                           |-- CalculatorParser.java
                           |-- CalculatorListener.java
                           |-- CalculatorVisitor.java
                 |-- Calculator.tokens
                 |-- CalculatorLexer.tokens
  |-- ...
```
&nbsp;&nbsp;&nbsp;&nbsp;其中的`Listener`和`Visitor`是根据我们的配置的`pom.xml`决定是否生成都不是必须的，只有`Lexer`和`Parser`是必须的。并且可以注意到`package`层次和`.g4`文件的目录层次是一致的。

&nbsp;&nbsp;&nbsp;&nbsp;现在开始创建项目的入口函数main所在的文件，把这个Parser真正的跑起来！在`java`目录下创建`Main.java`文件：
```java
import com.demo.parser.CalculatorLexer;
import com.demo.parser.CalculatorParser;
import org.antlr.v4.runtime.ANTLRInputStream;
import org.antlr.v4.runtime.CommonTokenStream;

public class Main {
    public static void main(String[] args) {
        ANTLRInputStream charStream = new ANTLRInputStream("1 + 2");
        CalculatorLexer lexer = new CalculatorLexer(charStream); // 创建 lexer

        CommonTokenStream tokenStream = new CommonTokenStream(lexer);
        CalculatorParser parser = new CalculatorParser(tokenStream);  // 创建 grammar
        
        parser.compileUnit();  // 运行解析规则入口
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;启动这个`main`函数一个简单的Parser就完成了，可以随意的改变`ANTLRInputStream`的参数，体会不同的字符串的结果，同时这里也可以传递文件名，输入流等等。
### 5. Interpreter
&nbsp;&nbsp;&nbsp;&nbsp;`Calculator`的功能当然不仅仅只应该是识别而已，还要负责计算出表达式的结果。计算表达式的结果不过是一次树的遍历，仅此而已！这里使用Visitor模式实现，可以参考我前面的[Visitor Pattern]({{site.baseurl}}/gof23/java/2019/04/10/visitor-pattern.html)一文。只需要继承之前生成的`BaseVisitor`文件，在`java`目录下创建`Interpreter.java`文件，代码简单易懂，不再赘述：
```java
import com.demo.parser.CalculatorBaseVisitor;
import com.demo.parser.CalculatorParser;
import org.antlr.v4.runtime.tree.ParseTree;

public class Interpreter extends CalculatorBaseVisitor<Integer> {

    @Override
    public Integer visitCompileUnit(CalculatorParser.CompileUnitContext ctx) {
        ParseTree expr = ctx.getChild(0);
        return expr.accept(this);
    }

    @Override
    public Integer visitExpr(CalculatorParser.ExprContext ctx) {
        if (ctx.getChildCount() == 1) {
            // 只有1个child的expr规则只有一个
            // expr : INT    
            String i = ctx.getChild(0).getText();
            return Integer.valueOf(i);
        } else {
            // 其他expr规则child有3个，其中第二个是运算符
            // expr : expr (+ | - | * | /) expr
            ParseTree expr1 = ctx.getChild(0);
            String operator = ctx.getChild(1).getText();
            ParseTree expr2 = ctx.getChild(2);
            Integer value1 = expr1.accept(this);
            Integer value2 = expr2.accept(this);
            if ("+".equals(operator)) {
                return value1 + value2;
            } else if ("-".equals(operator)) {
                return value1 - value2;
            } else if ("*".equals(operator)) {
                return value1 * value2;
            } else if ("/".equals(operator)) {
                return value1 / value2;
            } else {
                throw new RuntimeException("");
            }
        }
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;改变`Main`文件的，把`Visitor`加入上去，运行`main`即可：
```java
import com.demo.parser.CalculatorLexer;
import com.demo.parser.CalculatorParser;
import org.antlr.v4.runtime.ANTLRInputStream;
import org.antlr.v4.runtime.CommonTokenStream;


public class Main {
    public static void main(String[] args) {
        ANTLRInputStream charStream = new ANTLRInputStream("1 + 2");
        CalculatorLexer lexer = new CalculatorLexer(charStream);

        CommonTokenStream tokenStream = new CommonTokenStream(lexer);
        CalculatorParser parser = new CalculatorParser(tokenStream);

        CalculatorParser.CompileUnitContext compileUnitContext = parser.compileUnit();

        Interpreter interpreter = new Interpreter();  // 创建 Visitor
        Integer result = interpreter.visitCompileUnit(compileUnitContext);   // 使用 Visitor
        System.out.println(result);
    }
}
```

---
## III. Complete Calculator with ANTLR4
---
&nbsp;&nbsp;&nbsp;&nbsp;ANTLR4的语法规则非常简单：小写字母开头定义都是规则(`rule`), 大写字母开头都是`token`, 而类似于`'+'`，`'-'`之类不过是没有命名的`token`，完全可以定义类似于`INT`一样的`PLUS:'+';`，`SUB:'-';`词法规则。

&nbsp;&nbsp;&nbsp;&nbsp;如果你实现过递归下降的Parser，一定对于左递归和二义性不陌生。观察`Calculator.g4`不难发现`expr`两者都有了，当你使用前面的Test Rule去检测"1 + 2 * 3"，可以发现可视化的树的结构一定程度上符合我们的预期，这是因为ANTLR可以自动处理左递归，对于二义性的问题采用的是自上而下优先匹配的原则，所以`*`比`+`的优先级要高，但是前面的`Calculator.g4`中文法优先级顺序是`'*' > '/' > '+' > '-'`，但是实际上我们需要的是`'*' = '/' > '+' = '-'`，所以还需要一点小小的改变：
```antlr4
grammar Calculator; // 必须与文件名字一致

// 这是注释

// grammar
compileUnit : expr EOF;    // 第一个默认为grammar的开始规则

expr : expr ('*' | '/') expr
     | expr ('+' | '-') expr
     | INT
     ;

// lexer
INT : [0-9]+;

SPACES : [ \r\t\n] -> skip;  // 配置空格， 用skip指示空格跳过
```
&nbsp;&nbsp;&nbsp;&nbsp;通过Test Rule测试，发现`*`和`/`的优先级一样了，重新运行`antlr4:antlr4`，重新生成的代码，这次完全符合我们的常识了。你或许还注意到了`8 / 2 / 2`的计算顺序是`(8 / 2) / 2`而不是`8 / (2 / 2)`，ANTLR4中运算符的默认是左结合的，如何想要实现右结合运算符，如幂运算符`^`，即`2 ^ 3 ^ 4`表示的是2<sup>3<sup>4</sup></sup>，它的计算顺序是`2 ^ (3 ^ 4)`，即2<sup>(3<sup>4</sup>)</sup>，而不是`(2 ^ 3) ^ 4`，即(2<sup>3</sup>)<sup><sup>4</sup></sup>，需要使用ANTLR4中特殊标记`<assoc=right>`:
```antlr4
expr : <assoc=right> expr '^' expr
     | expr ('*' | '/') expr
     | expr ('+' | '-') expr
     | INT
     ;
```
&nbsp;&nbsp;&nbsp;&nbsp;在生成的`Visitor`中使用`visitExpr`，通过获取语法树的第二个`child`的值来判断`expr`采用的具体是哪一条规则，这样的做法虽然正确但是不方便，ANTLR4提供别名的机制，在规则只有使用`#`来给规则重新定义一个名字：
```antlr4
expr : <assoc=right> expr '^' expr    #pow
     | expr ('*' | '/') expr          #mul
     | expr ('+' | '-') expr          #add
     | INT                            #int
     ;
```
&nbsp;&nbsp;&nbsp;&nbsp;这样就可以选择重载`visitPow`，`visitMul`替换前面的`if else`的判断：
```java
// Interpreter.java
import com.demo.parser.CalculatorBaseVisitor;
import com.demo.parser.CalculatorParser;
import org.antlr.v4.runtime.tree.ParseTree;

public class Interpreter extends CalculatorBaseVisitor<Integer> {

    @Override
    public Integer visitCompileUnit(CalculatorParser.CompileUnitContext ctx) {
        ParseTree expr = ctx.getChild(0);
        return expr.accept(this);
    }

    @Override
    public Integer visitPow(CalculatorParser.PowContext ctx) {
        ParseTree exprL = ctx.getChild(0);
        ParseTree exprR = ctx.getChild(2);
        Integer left = exprL.accept(this);
        Integer right = exprR.accept(this);
        return (int)Math.pow(left, right);
    }

    @Override
    public Integer visitMul(CalculatorParser.MulContext ctx) {
        ParseTree exprL = ctx.getChild(0);
        String op = ctx.getChild(1).getText();
        ParseTree exprR = ctx.getChild(2);
        Integer left = exprL.accept(this);
        Integer right = exprR.accept(this);
        if ("*".equals(op)) {
            return left * right;
        } else {
            return left / right;  // ignore some detail here 
        }
    }

    @Override
    public Integer visitAdd(CalculatorParser.AddContext ctx) {
        ParseTree exprL = ctx.getChild(0);
        String op = ctx.getChild(1).getText();
        ParseTree exprR = ctx.getChild(2);
        Integer left = exprL.accept(this);
        Integer right = exprR.accept(this);
        if ("+".equals(op)) {
            return left + right;
        } else {
            return left - right;
        }
    }

    @Override
    public Integer visitInt(CalculatorParser.IntContext ctx) {
        return Integer.valueOf(ctx.getText());
    }
}

```
&nbsp;&nbsp;&nbsp;&nbsp;完成`Calculator`所需要的全部ANTLR4知识已经足够了，剩下的工作就是完成文法的定义了。
```antlr4
grammar Calculator; // 必须与文件名字一致

// 这是注释

// grammar
compileUnit : expr EOF;    // 第一个默认为grammar的开始规则

expr : '(' expr ')'                #paren
     | ('+' | '-') expr            #unary
     | <assoc=right> expr '^'expr  #pow
     | expr ('*' | '/') expr       #mul
     | expr ('+' | '-') expr       #add
     | DOUBLE                      #double
     | INT                         #int
     ;

// lexer
INT : [0-9]+;
DOUBLE: INT '.' INT;

SPACES : [ \r\t\n] -> skip;  // 配置空格， 用skip指示空格跳过
```
&nbsp;&nbsp;&nbsp;&nbsp;以及最后的`Interpreter.java`:
```java
import com.demo.parser.CalculatorBaseVisitor;
import com.demo.parser.CalculatorParser;
import org.antlr.v4.runtime.tree.ParseTree;

public class Interpreter extends CalculatorBaseVisitor<Double> {

    @Override
    public Double visitCompileUnit(CalculatorParser.CompileUnitContext ctx) {
        ParseTree expr = ctx.getChild(0);
        return expr.accept(this);
    }

    @Override
    public Double visitPow(CalculatorParser.PowContext ctx) {
        ParseTree exprL = ctx.getChild(0);
        ParseTree exprR = ctx.getChild(2);
        Double left = exprL.accept(this);
        Double right = exprR.accept(this);
        return Math.pow(left, right);
    }

    @Override
    public Double visitMul(CalculatorParser.MulContext ctx) {
        ParseTree exprL = ctx.getChild(0);
        String op = ctx.getChild(1).getText();
        ParseTree exprR = ctx.getChild(2);
        Double left = exprL.accept(this);
        Double right = exprR.accept(this);
        if ("*".equals(op)) {
            return left * right;
        } else {
            return left / right;
        }
    }

    @Override
    public Double visitAdd(CalculatorParser.AddContext ctx) {
        ParseTree exprL = ctx.getChild(0);
        String op = ctx.getChild(1).getText();
        ParseTree exprR = ctx.getChild(2);
        Double left = exprL.accept(this);
        Double right = exprR.accept(this);
        if ("+".equals(op)) {
            return left + right;
        } else {
            return left - right;
        }
    }

    @Override
    public Double visitInt(CalculatorParser.IntContext ctx) {
        return Integer.valueOf(ctx.getText()).doubleValue();
    }

    @Override
    public Double visitDouble(CalculatorParser.DoubleContext ctx) {
        return Double.valueOf(ctx.getText());
    }

    @Override
    public Double visitParen(CalculatorParser.ParenContext ctx) {
        ParseTree expr = ctx.getChild(1);
        return expr.accept(this);
    }

    @Override
    public Double visitUnary(CalculatorParser.UnaryContext ctx) {
        String unaryOp = ctx.getChild(0).getText();
        ParseTree expr = ctx.getChild(1);
        Double val = expr.accept(this);
        if ("+".equals(unaryOp)) {
            return val;
        } else {
            return -val;
        }
    }
}
```

---
## IV. Conclusion
---
&nbsp;&nbsp;&nbsp;&nbsp;ANTLR的功能远非如此，想要了解更多可以参考ANTLR作者的两本书：《ANTLR4 权威指南》<sup>[3]</sup>以及《编程语言实现模式》<sup>[4]</sup>，不过《编程语言实现模式》使用的是ANTLR3，内容和ANTLR4有些出入。

&nbsp;&nbsp;&nbsp;&nbsp;实现一门语言的Parser并没有想象的难度那么大，类似于Antlr4之类的优秀的工具让这样的工作更加简单容易，只要掌握Parser背后原理，应当把重心放到对语义的分析，这或许就是绝大多数编译原理课程把Parser这项工作轻描淡写，一笔带过的理由吧。龙书(编译原理)中花费大量的篇幅(三到四章节左右)讲述了Parser相关的算法，实际上并非是教你如何实现Parser(实际上只有第二章节的介绍了递归下降的实现)，其他篇幅更多是在描述Parser Generator是如何实现的。但是绝大多数人往往倒在了Parser部分......

---
## V. Reference
---
[1] [https://en.wikipedia.org/wiki/Recursive_descent_parser](https://en.wikipedia.org/wiki/Recursive_descent_parser)

[2] [https://en.wikipedia.org/wiki/Pratt_parser](https://en.wikipedia.org/wiki/Pratt_parser)

[3] [https://www.amazon.com/Definitive-ANTLR-4-Reference/dp/1934356999](https://www.amazon.com/Definitive-ANTLR-4-Reference/dp/1934356999)

[4] [https://www.amazon.com/Language-Implementation-Patterns-Domain-Specific-Programming/dp/193435645X](https://www.amazon.com/Language-Implementation-Patterns-Domain-Specific-Programming/dp/193435645X)