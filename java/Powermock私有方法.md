如果方法中还有其他待测逻辑等原因不能直接mock掉的话，这里就需要去mock被调用的private方法。

```java
public class Calculator {
    private int sumXX(int a, int b) {
		return a + b;
	}
    
    public int callSumXX(int a, int b){
    	return sumXX(a, b);
    }
}
```

### 1.powermock private method的方法

spy被测类：

```java
Calculator powerMock = PowerMockito.spy(new Calculator());
```

构造返回：

```java
PowerMockito.when(powerMock, "sumXX", 1, 2).thenReturn(2);
```

断言：

```java
assertEquals(2, powerMock.callSumXX(1, 2));
```

完整代码如下：

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({Calculator.class})
public class CalculatorTest {
	
	private Calculator test;
	 
	@BeforeClass
	public static void setUpBeforeClass() throws Exception {
	}
	 
	@AfterClass
	public static void tearDownAfterClass() throws Exception {
	}
	 
	@Before
	public void setUp() throws Exception {
		test = new Calculator();
	}
	 
	@After
	public void tearDown() throws Exception {
	}
	 
	@Test
	public void testSumXX() throws Exception {
		Calculator powerMock = PowerMockito.spy(new Calculator());
		PowerMockito.when(powerMock, "sumXX", 1, 2).thenReturn(2);
		assertEquals(2, powerMock.callSumXX(1, 2));
	}
}
```

测试发现pass了,说明private方法sumXX被成功mock

