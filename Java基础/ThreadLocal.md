# ThreadLocal

![](https://github.com/skittlekx/JAVA_NOTE/blob/master/img/ThreadLocal.png?raw=true)

ThreadLocal类用来设置线程私有变量,**本身不储存值**,主要提供自身引用和操作hreadLocalMap 属性值得方法，使用ThreadLocal会通过ThreadLocal的引用定位到到堆中Thread的类ThreadLocalMap里散列表里的值,从而达到线程私有。

**测试代码**
```java
public class Main {
    public static ThreadLocal<Object> threadTest = new ThreadLocal<>();

    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable(){
            @Override
            public void run() {
                for (int i = 0; i < 3; i++) {
                    if(null == Main.threadTest.get()){
                        Main.threadTest.set("t1 set");
                    }
                    else {
                        String str = (String)Main.threadTest.get();
                        Main.threadTest.set(str += "1");
                        System.out.println(str);
                    }
                }
            }
        });

        Thread t2 = new Thread(new Runnable(){
            @Override
            public void run() {
                for (int i = 0; i < 3; i++) {
                    if(null == Main.threadTest.get()){
                        Main.threadTest.set(0);
                    }
                    else {
                        Integer num = (Integer)Main.threadTest.get();
                        Main.threadTest.set(num += 1);
                        System.out.println("t2" + num);
                    }
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

运行结果：
```
t1 set1  
t1 set11  
t21  
t22
```


- Thread

Thread 中聚合两个ThreadLocalMap对象，用于储存当前线程的本地ThreadLocal实例，inheritableThreadLocals用于将threadLocals传递给子线程。

```java
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

**InheritableThreadLocal 类结构**
```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    /**
     * Creates an inheritable thread local variable.
     */
    public InheritableThreadLocal() {}

    /**
     * Computes the child's initial value for this inheritable thread-local
     * variable as a function of the parent's value at the time the child
     * thread is created.  This method is called from within the parent
     * thread before the child is started.
     * <p>
     * This method merely returns its input argument, and should be overridden
     * if a different behavior is desired.
     *
     * @param parentValue the parent thread's value
     * @return the child thread's initial value
     */
    protected T childValue(T parentValue) {
        return parentValue;
    }

    /**
     * Get the map associated with a ThreadLocal.
     *
     * @param t the current thread
     */
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    /**
     * Create the map associated with a ThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the table.
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}

```


- **ThreadLocal getMap 方法**

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

- **ThreadLocal set 方法**

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        createMap(t, value);
    }
}
```

- **ThreadLocal get 方法**

在线程中调用，首先获取当前线程，然后以this(ThreadLocal示例)为key值获取value
```Java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

- ThreadLocalMap

通过ThreadLocal实例的弱引用储存键值结构，当线程关闭的时候自动回收。
处理hash碰撞的方式为开放寻址法，线性探查

```java
static class ThreadLocalMap {

    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    private static final int INITIAL_CAPACITY = 16;

    private Entry[] table;

    private int size = 0;

    private int threshold;

    //线性探查寻找下一个位置
    private static int nextIndex(int i, int len) {
        return ((i + 1 < len) ? i + 1 : 0);
    }
}
```

