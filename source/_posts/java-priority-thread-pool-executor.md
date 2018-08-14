---
title: Java priority thread pool executor
date: 2018-08-14 13:47:17
tags:
 - java
categories:
 - Java
lede: "Java 使用 ThreadPoolExecutor 和 PriorityBlockingQueue 实现优先级线程池"
thumbnail: /img/thread-pool.png
---

首先要了解的是线程池（`ThreadPoolExecutor`）的设计
![](/img/thread-pool-execotor.jpg)

线程池最重要的一个属性 `workQueue`，没有提交运行的任务，都放入了这个队列当中，所以，只要控住了这个队列即可。常用的队列有： 

* `SynchronousQueue`
* `LinkedBlockingDeque`
* `ArrayBlockingQueue`

默认没有基于 `PriorityBlockingQueue` 实现的 ThreadPool. 下面描述一下自己开发一个基于 `PriorityBlockingQueue` 队列的优先级线程池步骤。

定义 `PriorityThreadPoolExecutor` 

```java 
public class PriorityThreadPoolExecutor extends ThreadPoolExecutor {

    //省略 

    @Override
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new PriorityFutureTask(runnable, value);
    }

    @Override
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new PriorityFutureTask(callable);
    }

}
```

为什么需要这样的一个类，来继承 `ThreadPoolExecutor` ?  可以看看这篇文章[ThreadPoolExecutor的PriorityBlockingQueue类型转化问题](https://blog.csdn.net/jekxi/article/details/53142202)

在执行器 submit 的时候，会通过 newTaskFor 来创建 RunnableFuture. 覆写这两个方法，是为了用上自定义的 FutureTask.

```java  
public class PriorityFutureTask<V>
        extends FutureTask<V> implements Comparable<PriorityFutureTask<V>> {

    private Object object;

    public PriorityFutureTask(Callable<V> callable) {
        super(callable);
        object = callable;
    }

    public PriorityFutureTask(Runnable runnable, V result) {
        super(runnable, result);
        object = runnable;
    }

    @Override
    public int compareTo(PriorityFutureTask<V> o) {
        if (this == o) {
            return 0;
        }
        if (o == null) {
            return -1; // high priority
        }
        if (object != null && o.object != null) {
            if ((object instanceof Comparable)) {
                return ((Comparable) object).compareTo(o.object);
            }
        }
        return 0;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == null) {
            return false;
        } else if (!(obj instanceof PriorityFutureTask)) {
            return false;
        } else {
            PriorityFutureTask<V> c = (PriorityFutureTask<V>) obj;
            return object.equals(c.object);
        }
    }

    @Override
    public int hashCode() {
        return object.hashCode();
    }

    public Object getObject() {
        return object;
    }

    public void setObject(Object object) {
        this.object = object;
    }
}
```

该方法主要是保存了构造方法传入的 Runnable 并实现了排序器。因为 workQueue 存的就是这个对象，如果想要调用执行器的 remove 方法从执行器中的 workQueue 移除待提交任务，就需要对传入的这个 Runnable 对象做一些处理 

```java  
public class MiniTask extends AbstractTask<Integer> implements Runnable, Comparable {

    private Mini mini;

    public MiniTask(Mini mini) {
        this.mini = mini;
        this.priority = 1;
    }
    public MiniTask(Mini mini, Integer priority) {
        this.mini = mini;
        this.priority = priority;
    }

    public void run() {
        //to do
        System.out.println("Running - [" + mini.getMessage() + "], priority is [" + priority + "]" + " --- " + System.currentTimeMillis());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public int hashCode() {
        return mini.hashCode();
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == null) {
            return false;
        } else if (obj instanceof PriorityFutureTask) {
            PriorityFutureTask p = (PriorityFutureTask) obj;
            return this.equals(p.getObject());
        } else if (obj instanceof MiniTask) {
            MiniTask t = (MiniTask) obj;
            return mini.equals(t.mini);
        }
        return false;
    }

    public int compareTo(Object o) {
        if (o == null) {
            return -1;
        } else if (!(o instanceof MiniTask)) {
            return -1;
        }
        MiniTask m = (MiniTask) o;
        return Integer.compare(priority, m.priority);
    }
}
```

这个类的重点在 `equal` 方法，这里面判断了 `MiniTask` 与 `PriorityFutureTask` 的 equal 关系。


点击查看[项目源码](https://github.com/jianle/priority-executor-pool) 通过运行 `App` 来看最终效果, 日志如下 

```log 
Running - [A02], priority is [5] --- 1534225081301
Remove v6: true
Running - [A01], priority is [1] --- 1534225081301
Executor core pool size: 1
Running - [A04], priority is [8] --- 1534225083308
Running - [A12], priority is [8] --- 1534225085309
Running - [A10], priority is [8] --- 1534225087309
Running - [A08], priority is [5] --- 1534225089310
Running - [A11], priority is [4] --- 1534225091310
Running - [A13], priority is [4] --- 1534225093310
Running - [A05], priority is [4] --- 1534225095310
Running - [A09], priority is [3] --- 1534225097311
Running - [A03], priority is [3] --- 1534225099311
Running - [A07], priority is [1] --- 1534225101311
```


