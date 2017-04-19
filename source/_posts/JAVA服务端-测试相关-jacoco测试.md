title: jacoco覆盖率测试
date: 2016/12/06 12:06:06
categories:
- JAVA服务端
tags:
- 测试
- jacoco
---

# 1两种方式

## 1.1 eclipse直接安装插件测试
![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco1.png)

**install new soft 安装插件 eclemma java code coverage**

运行
![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco2.png)


结果
![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco3.png)

插件导出单元测试报告
![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco4.png)

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco5.png)




## 1.2 通过maven加入插件，打印单元测试报告

**pom.xml文件中加入对应的部分，plugin部分**


```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.whx</groupId>
    <artifactId>jacoco-test</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
    <build>
          <plugins>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.7.1.201405082137</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```


**maven clean package 即可有报告 在target文件中 site index.html**

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco6.png)

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco7.png)






结果

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco8.png)

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco9.png)


**注：有时候你会发现会爆出这个问题Skipping JaCoCo execution due to missing execution data file，大多数原因是因为没有编译测试类，可以使用 clean package -Dmaven.test.skip=false**



# 2.关于检测指标
（可大概理解其意思，英文网址附上）
[http://www.eclemma.org/jacoco/trunk/doc/counters.html](http://note.youdao.com/)

**Instructions (C0 Coverage)**

- 主要是计算字节码文件的覆盖率。

**Branches (C1 Coverage)**

- JaCoCo也计算分支覆盖所有if和 switch语句。主要是计算分支的。
- 没有覆盖：在该行没有分支机构已执行（红钻）
- 部分覆盖：只有在该行分支机构的一部分已经被执行（黄钻）
- 全覆盖：在该行各分支机构已执行（绿钻）

**Cyclomatic Complexity **

- 圈复杂度(Cyclomatic Complexity)是一种代码复杂度的衡量标准。它可以用来衡量一个模块判定结构的复杂程度，数量上表现为独立现行路径条数，也可理解为覆盖所有的可能情况最少使用的测试用例数。圈复杂度大说明程序代码的判断逻辑复杂，可能质量低且难于测试和维护。程序的可能错误和高的圈复杂度有着很大关系。请注意，JaCoCo不考虑异常处理的分支机构try-catch块也不会增加复杂性。总体和分支正相关。实际上，过去几年的各种研究已经确定：一个方法的圈复杂度（或 CC）大于 10 的方法存在很大的出错风险。
- 关于圈复杂度的理解，可以看以下链接。

- [http://blog.csdn.net/lg707415323/article/details/7790660 ](http://note.youdao.com/)
- [http://www.ibm.com/developerworks/cn/java/j-cq03316/](http://note.youdao.com/)

- 以及一个图

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco10.png)

**Lines**

- 主要计算基于覆盖的实际源代码行类和源文件行覆盖。通常会标识三种状态。
- 没有覆盖：在该行任何指令执行（红色背景）
- 部分覆盖：只有在该行的指示的一部分已经被执行（黄色背景）
- 全覆盖：在该行的所有指令已执行（绿色背景）


**Methods**

- 每个非抽象方法包含至少一个指令。构造函数和静态初始化都算作方法。




# 4.完整例子

**先贴上代码
被测试类：**

```
package utils;

import java.math.BigDecimal;

/**
 * 安全转换钱的单位
 * 
 * @author rutine
 * @time Apr 28, 2015 3:52:18 PM
 */
public class MoneyUtil {


    /**
     * <pre>
     * 功能说明 : 安全转换double类型, 将单位元的钱转为分
     *     如: 19.9(元), 最终结果: 1990(分)
     * </pre>
     * 
     * @param money 金额(元)
     * @return
     */
    public static int getFenMoney(String money) {
        BigDecimal hundred = new BigDecimal(100);
        BigDecimal decimalMoney = new BigDecimal(money);
        return decimalMoney.multiply(hundred).intValue();
    }


    /**
     * 功能说明 : 获取单价, 保留小数点两位数
     * 
     * @param money 总金额(元)
     * @param quantity 数量
     * @return
     */
    public static double getUnitMoney(String money, int quantity) {
        if ("50".equals(money)) {
            System.out.println("branch1");
        }else if ("60".equals(money)) {
            System.out.println("branch2");
        }else if ("70".equals(money)) {
            System.out.println("branch3");
        }else {
            System.out.println("other branch");
        }
        BigDecimal decimalMoney = new BigDecimal(money);
        BigDecimal unitMoney = decimalMoney.divide(new BigDecimal(quantity), 2, BigDecimal.ROUND_HALF_UP);

        return unitMoney.doubleValue();
    }
}
```



**测试类：**

```
package test;

import org.junit.Test;

import utils.MoneyUtil;

public class TestMoney {

    @Test
    public void testGetUnitMoney(){
        MoneyUtil.getUnitMoney("50", 2);
        MoneyUtil.getUnitMoney("60", 2);
        MoneyUtil.getUnitMoney("70", 2);
        MoneyUtil.getUnitMoney("75", 2);
    }
}
```




**测试结果，指标分析**

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco11.png)


这一项指标instructions，指的是字节码文件的行数。18/62的意思是  18为未执行的指令行数，62为总指令行数。

 ![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco12.png)
 
 ![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco13.png)



这一项指标branches，指的是分支的覆盖情况。0/6中0为未执行的分支行数，6为总分支行数。这里else不会计入到分支行数中。但是如果你不写else,则覆盖率不会为100%。

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco14.png)

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco15.png)


这一项指标，为圈复杂度。missed 为未测试的数量，cxty为总数。

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco16.png)

我们来看看这个圈复杂度为啥是4

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco17.png)

简单的画个控制流图

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco18.png)

v=e-n+2   8条边 - 6个节点 + 2 = 4


这一项指标，为行的覆盖情况。missed 为未测试的数量，lines为总数。

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco19.png)

同样的，看看这个例子

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco20.png)

正好是10行。未检测的是0行。

这一项指标，为方法的覆盖情况。missed 为未测试的数量，methods为总数。

![image](http://ohoyqlwj0.bkt.clouddn.com/jacoco21.png)

最后，请注意，覆盖率达到100%，不代表你的程序就ok了！
