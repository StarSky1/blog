[pixiv: 034]: # "https://cdn.jsdelivr.net/gh/starsky1/poi/2020/34/34.png"

1.需求：项目中有一个类的重复代码太多，想要删除冗余代码，减少代码复杂度，让代码更易读和易用。
2.思路：此类的特点，有两个方法，方法体代码的头部和尾部相同，中间部分不同。思路是将中间变化的部分取出，头部和尾部相同的代码进行复用。
3.解决方案：使用 JDK8 的函数式编程特性和 Lambda 表达式,，定义函数类型的参数，将变化部分以回调函数的形式传入。JDK8提供了四种函数式编程工具类，Consumer、Supplier、Predicate 与 Function。
4.代码：

```java
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

/**
 * 函数式编程
 * @author 76355
 */
public class Clt {

    public static Double pow(double x) {
        return Math.pow(x, x);
    }

    public static void main(String[] args) {
        // 有参数无返回值
        new CltCon<Integer>().fun(System.out::println, 2);
        // 有参数有返回值
        System.out.println(new CltFun<Integer, Double>().fun(Clt::pow, 2));
        // 有参数，返回boolean值
        System.out.println(new CltPred<Integer>().fun(x -> x instanceof Number, 2));
        // 无参数返回一个结果
        System.out.println(new CltSup<Double>().fun(() -> Math.PI));
    }

    private static class CltCon<T> {
        public void fun(Consumer<? super T> action, T i) {
            action.accept(i);
            action.andThen((x) -> System.out.println("xxxxx"));
        }
    }

    private static class CltFun<T, R> {
        public R fun(Function<T, R> action, T i) {
            return action.apply(i);
        }
    }

    private static class CltPred<T> {
        public boolean fun(Predicate<T> action, T i) {
            return action.test(i);
        }
    }

    private static class CltSup<T> {
        public T fun(Supplier<T> action) {
            return action.get();
        }
    }
}

```