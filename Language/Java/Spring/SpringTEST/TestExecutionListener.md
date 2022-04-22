///////////////////////////////////////////////////////////////////////////
// --- beforeTestClass ---
// --- CONSTRUCTOR 1 ---
// --- prepareTestInstance ---
// --- beforeTestMethod ---
// --- RUN TEST ---
//...
// --- STOP TEST ---
// --- afterTestMethod ---
// --- afterTestClass ---
//...
// --- beforeTestClass ---
// --- CONSTRUCTOR 2 ---
// --- prepareTestInstance ---
// --- beforeTestMethod ---
// --- RUN TEST ---
//...
// --- NEXT TEST ---
// --- afterTestMethod ---
// --- CONSTRUCTOR 2 ---
// --- prepareTestInstance ---
// --- beforeTestMethod ---
// --- RUN NEXT TEST ---
//...
// --- STOP TEST ---
// --- afterTestMethod ---
// --- afterTestClass ---
///////////////////////////////////////////////////////////////////////////

```java
//Но вместо этого мы можем использовать аннотацию @Order .
//@Order(Integer.MAX_VALUE)
@Slf4j
public class CustomTestExecutionListener implements TestExecutionListener, Ordered {

    public void beforeTestClass(TestContext testContext) throws Exception {
        sendMassage("beforeTestClass");
        log.info("beforeTestClass : {}", testContext.getTestClass());
    };

    public void prepareTestInstance(TestContext testContext) throws Exception {
        sendMassage("prepareTestInstance");
        log.info("prepareTestInstance : {}", testContext.getTestClass());
    };

    public void beforeTestMethod(TestContext testContext) throws Exception {
        sendMassage("beforeTestMethod");
        log.info("beforeTestMethod : {}", testContext.getTestMethod());
    };

    public void afterTestMethod(TestContext testContext) throws Exception {
        sendMassage("afterTestMethod");
        log.info("afterTestMethod : {}", testContext.getTestMethod());
    };

    public void afterTestClass(TestContext testContext) throws Exception {
        sendMassage("afterTestClass");
        log.info("afterTestClass : {}", testContext.getTestClass());
    }

    private void sendMassage(String message) {
        System.out.println("--- " + message + " ---");
    }

    @Override
    public int getOrder() {
        return Integer.MAX_VALUE;
    };
}
```
