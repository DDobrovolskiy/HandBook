
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.task.TaskDecorator;
import org.springframework.core.task.TaskExecutor;
import org.springframework.scheduling.concurrent.ConcurrentTaskExecutor;

import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author dda
 * @since 21.04.2022
 */
public class Rest {
    public static void main (String[] args) {
        ApplicationContext context =
                new AnnotationConfigApplicationContext(MyConfig.class);
        MyBean bean = context.getBean(MyBean.class);
        bean.runTasks();
        ConcurrentTaskExecutor exec = context.getBean(ConcurrentTaskExecutor.class);
        ExecutorService service = (ExecutorService) exec.getConcurrentExecutor();
        service.shutdown();
    }

    @Configuration
    public static class MyConfig {

        @Bean
        MyBean myBean () {
            return new MyBean();
        }

        @Bean
        TaskExecutor taskExecutor () {
            ConcurrentTaskExecutor t = new ConcurrentTaskExecutor(
                    Executors.newFixedThreadPool(1));
            t.setTaskDecorator(new TaskDecorator() {
                @Override
                public Runnable decorate (Runnable runnable) {
                    return () -> {
                        long t = System.currentTimeMillis();
                        runnable.run();
                        System.out.printf("time taken for task: %s , %s%n",
                                ((MyTask) runnable).getI(),
                                (System.currentTimeMillis() - t));
                    };
                }
            });
            return t;
        }
    }

    private static class MyBean {
        @Autowired
        private TaskExecutor executor;

        public void runTasks () {
            for (int i = 0; i < 10; i++) {
                executor.execute(new MyTask(i));

            }
        }
    }

    private static class MyTask implements Runnable {

        private final int i;

        MyTask (int i) {
            this.i = i;
        }

        @Override
        public void run () {
            System.out.printf("running task %d. Thread: %s%n",
                    i,
                    Thread.currentThread().getName());
            try {
                Thread.sleep(Math.abs(new Random().nextInt(10000)));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }


        public int getI () {
            return i;
        }
    }
}
```
