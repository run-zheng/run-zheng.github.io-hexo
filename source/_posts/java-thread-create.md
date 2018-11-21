---


title: Java多线程的实现方式
date: 2018-11-21 23:04:56
tags: [并发编程,java多线程,Thread,FutureTask,Runnable,]
categories: 并发编程


---

Java中实现多线程的方式
-------------------------

## 一、继承Thread类 

继承Thread类，重写run方法，调用Thread的start()方法启动线程： 

```java
	/**
	 * 实现线程方式：1、继承Thread类
	 */
	@Slf4j
	public static class ThreadTarget extends Thread {
		@Override
		public void run() {
			while (true) {
				log.info("Thread extentions: " + System.currentTimeMillis());
				try {
					Thread.sleep(200);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
```

new一个Thread实例的时候，所有构造方法最终都是调用Thread的init方法， init方法中，设置线程属性，校验相关权限： 

```java
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc) {
		/**设置线程名字,匿名线程，名字自动生成：Thread-nextThreadNum()**/
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }
        this.name = name;
		/**设置线程组**/
        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            if (security != null) {
                g = security.getThreadGroup();
            }
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }
        g.checkAccess(); //检查当前线程(parent)是否有权限修改线程组g
        if (security != null) { //校验是否thread子类的getContextClassLoader被覆盖
            if (isCCLOverridden(getClass())) { //并且拥有enableContextClassLoaderOverride权限
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }
        g.addUnstarted(); //增加线程组的unstarted线程计数
        this.group = g; //设置线程组
		/**继承当前线程(parant)的是否后台线程、设置优先级的值**/
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
		/**设置contextClassLoader和继承accessControlContext**/
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
		/**设置Runnable接口的实例，作为target对象**/		
        this.target = target;
		/**这里才是真正设置优先级的地方**/
        setPriority(priority);
		/**继承当前线程(parent)的ThreadLocal.ThreadLocalMap**/
        if (parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        this.stackSize = stackSize;
		/**设置线程id, 这里设置的是tid，跟nextThreadNum()设置的threadInitNumber是两个值**/
        tid = nextThreadID();
    }
```

启动线程需要调用start方法，方法中调用native方法start0真正启动线程，
虚拟机会在线程启动过程中回调Thread的run方法：

```java
    public synchronized void start() {
        /**threadStatus不等于0, 状态异常，非NEW状态的线程*/
        if (threadStatus != 0)
            throw new IllegalThreadStateException();
        /**将线程加入到线程组，减线程组的unstarted计数器**/
        group.add(this);
        boolean started = false;
        try {
			/**native方法启动线程**/
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
					/**移出线程组，增加unstarted计数器**/
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /** do nothing. start0的异常会直接抛出 **/
            }
        }
    }
	/**启动线程的native方法**/
    private native void start0();

```

- Thread本身也实现了Runnable接口, 实现run方法，虚拟机启动时，调用的就是该实现方法。
- 单纯从target看，Thread和Runnable的关系看，这是简单的模板方法模式的实现。
- 这也引出了第二种实现方式，实现Runnable接口或者实现/继承Runnable接口的子接口/子类。 
- 所以继承Thread实现run和实现Runnable接口实现run方法是有本质区别的，Thread的run是被虚拟机调用的，Runnable的run是作为thread的target属性的模板方法被调用的。 

```java
    @Override
    public void run() {
        /**如果target不为空，调用target的run方法**/
        if (target != null) {
            target.run();
        }
    }
```

## 二、实现或继承Runnable接口的子接口或实现类

实现多线程的另外方式是实现或继承Runnable接口的子接口或实现类

#### 1、继承Runnable接口

直接实现Runnable接口的问题是，线程执行完成后，无法获取到线程执行结果。

```java 
        /**
		 * 实现线程方式： 2、实现Runnable接口
		 */
		Thread thread2 = new Thread(new Runnable() {
			@Override
			public void run() {
				while (true) {
					log.info("Runnable implements 1: " + System.currentTimeMillis());
					try {
						Thread.sleep(200);
					} catch (InterruptedException e) {
						log.error(e.getMessage(), e);
					}
				}
			}
		}); 
		thread2.start();
		/**
		 * 实现线程方式： jdk1.8之后，可以更简单的写法
		 */
		Thread thread3 = new Thread(() -> {
			while (true) {
				log.info("Runnable implements 2: " + System.currentTimeMillis());
				try {
					Thread.sleep(200);
				} catch (InterruptedException e) {
					log.error(e.getMessage(), e);
				}
			}
		}); 
		thread3.start();
```

#### 2、继承FutureTask接口 

如果需要获取线程执行结果， 可以实现FutureTask接口, 并通过FutureTask的get方法获取执行结果： 
```java
		/**
		 * 实现线程方式： 3、实现Callable接口，配合FutureTask，获取线程执行结果
		 */
		FutureTask<Integer> task = new FutureTask<>(new Fibonacci(10));
		Thread thread4 = new Thread(task);
		thread4.start();
		try {
			Integer result = task.get();
			log.info("FutureTask result: {}" , result);
		} catch (ExecutionException e) {
			log.error(e.getMessage(), e);
		}
```

> FutureTask相关的源码研究，后续分析。 

#### 3、继承TimerTask接口

如果需要在主线程外执行一些任务的话，可以使用TimerTask接口,配合定时器工具Timer实现，Timer内部维护一个TimerThread线程，用于执行调度任务：
```java
		/**
		 * 实现线程方式：4、调度执行任务
		 */
		Timer timer = new Timer(); 
		timer.schedule(new TimerTask() {
			@Override
			public void run() {
				log.info("TimerTask execute: {}", System.currentTimeMillis());
			}
		}, 5000, 3000);
```
简单的应用可以使用Timer，复杂需求应该引入框架。

## 三、Executor框架
JDK1.5引入Executor异步执行框架，灵活强大，支持多种任务执行策略，将任务提交和执行解耦，可以通过submit和execute提交任务给线程池执行。Executors提供一系列创建线程池的工厂方法。

> 后续对Executor框架进行分析

```java
		/**
		 * 实现多线程的方式： 5、java.util.concurrent包提供的Executor框架,创建线程池，执行任务
		 */
		ExecutorService executorService = Executors.newFixedThreadPool(5);
		//通过execute执行实现Runnable接口的任务
		for(int i = 0; i < 100; i++) {
			executorService.execute(new Runnable() {
				@Override
				public void run() {
					log.info("ThreadPool execute task: {} {}", 
							Thread.currentThread().getName(), 
							System.currentTimeMillis());
					try {
						Thread.sleep(100);
					} catch (InterruptedException e) {
						log.error(e.getMessage(), e);
					}
				}
			});
		}
		//也可以通过submit实现Callable接口的任务，以便获取线程执行结果
		Future<Integer> fiboResult = executorService.submit(new Fibonacci(10)); 
		try {
			log.info("ThreadPool submit task: {}",fiboResult.get());
		} catch (ExecutionException e) {
			log.error(e.getMessage(), e);
		}
```

## 四、ForkJoin框架
如果大任务可以分解成小任务并行计算，可以实现RecursiveTask接口，提交到ForJoin框架执行。 

```java
	public static void useForkJoinFramework() {
		long start = System.currentTimeMillis();
		ForkJoinPool pool = new ForkJoinPool(); 
		Long result = pool.invoke(createNewTask(0L, 10000000000L, 1000000000L));
		log.info("ForkJoinPool execute result: {} {}", result , (System.currentTimeMillis() - start));
		
		start = System.currentTimeMillis();
		long sum = 0; 
		for(long i = 0;i <= 10000000000L; i++) {
			sum += i; 
		}
		log.info("Simple sum: {} {}", sum, (System.currentTimeMillis() - start));
	}
	
	@SuppressWarnings("serial")
	public static RecursiveTask<Long> createNewTask(final Long start, final Long end, final Long critical){
		return new RecursiveTask<Long>() {
			@Override
			protected Long compute() {
				if(end - start <= critical) {
					long sum = 0L; 
					for(long l = start; l <= end; l++) {
						sum += l; 
					}
					return sum; 
				}else {
					Long middle = (end + start) / 2; 
					RecursiveTask<Long> left = createNewTask(start, middle, critical);
					RecursiveTask<Long> right = createNewTask(middle+1, end, critical);
					left.fork();
					right.fork();
					return left.join()+right.join(); 
				}
			}
		};
	}
```

RecursiveTask接口继承自ForkJoinTask接口， ForkJoinTask继承Future接口。 

## 五、总结

Java中多线程编程主要分两类：
1. 通过创建Thread实例创建线程和start()方法启动线程，自己管理线程，执行任务；
2. 通过Executor框架创建线程池，或者实现相关接口，通过线程池管理管理线程执行任务；
3. 通过ForkJoin框架并行执行任务。
不管是通过哪种方式， 都有支持Runnable接口提交任务和支持Callable接口的方式，jdk1.8之后，还可以通过lambda表达式提交任务，
编程上弱化了Runnable和Callable的区别，所以选用哪种方式去创建线程和管理线程，取决于：
- 场景是否足够简单，只需简单管理线程，还是需要强大的线程管理功能？
- 是否需要线程执行结果，可以Future相关的API？
- 是否可以并行执行？




