## 一、pytest 简介
1. pytest是一个非常成熟的python的单元框架，比unittest更灵活，容易上手
2. pytest可以和selenium,requests,appium结合实现web自动化，接口自动化，app自动化
3. pytest可以实现测试用例的跳过以及reruns失败用例重试
4. pytest可以和alure生成非常美观的测试报告
5. prtest可以和Jenkins持续集成
6. pytest有很多功能非常强大的插件，并且这些插件能够实现很多的实用操作
    pytest常用插件：  
    &ensp;&ensp;pytest-html 生成html格式的自动化测试报告  
    &ensp;&ensp;pytest-xdist 测试用例分布式执行，多CPU分发  
    &ensp;&ensp;pytest-ordering 用于改变测试用例的执行顺序  
    &ensp;&ensp;pytest-rerunfailure 用例失败后重新执行  
    &ensp;&ensp;allure-pytest 用于生成美观的测试报告  
    pytest以及插件安装方式:  
    &ensp;&ensp;通过python pypi仓库安装 pip install pytest pytest-html ......  
    &ensp;&ensp;项目用一般使用requirements.txt维护  

## 二、使用pytest应遵循的规则以及基础应用
1. 模块名必须以test_开头，或者以_test结尾
2. 测试类必须以Test开头，并且不能有init方法
3. 测试方法必须以test开头
## 三、pytest运行方式
1. 主函数模式
    &ensp;&ensp;运行所有：pytest.main(["-vs"])  
    &ensp;&ensp;指定模块：pytest.main(["-vs", "test_module.py"])  
    &ensp;&ensp;指定目录：pytest.main(["-vs", "./test_dir"])  
    &ensp;&ensp;指定nodeid：pytest.main(["-vs", "./test_dir/test_file.py::TestObject::test_func"])  
2. 命令行模式
    &ensp;&ensp;运行所有：pytest  
    &ensp;&ensp;指定模块：pytest -vs test_module.py  
    &ensp;&ensp;指定目录：pytest -vs ./test_dir  
    &ensp;&ensp;指定nodeid：pytest -vs ./test_dir/test_file.py::TestObject::test_func  
3. 通过读取pytest.ini配置文件运行
    &ensp;&ensp;pytest.ini：是pytest单元测试的核心文件  
    &ensp;&ensp;位置：一般放在项目根目录  
    &ensp;&ensp;编码：必须是ANSI  
    &ensp;&ensp;作用：修改pytest默认行为  
    &ensp;&ensp;规则：主函数运行模式和命令行模式都会去读取这个配置文件  
4. 参数详解
    &ensp;&ensp;-s：表示输出调试信息，包括print打印信息  
    &ensp;&ensp;-v：显示详细的信息  
    &ensp;&ensp;-vs: 两个参数组合使用  
    &ensp;&ensp;-n：支持多线程或者分布式运行测试用例  
    &ensp;&ensp;--reruns $NUM 失败用例重跑  
    &ensp;&ensp;-x：只要有一个用例失败就停止测试  
    &ensp;&ensp;--maxfail=2 容许用例失败的最大次数，有两个用例失败就停止测试  
    &ensp;&ensp;-k 根据测试用例部分字符串关键字指定测试用例  
## 四、pytest执行顺序
1. unittest：使用ascll码的大小来执行
2. pytest：默认重上往下
3. pytest：修改执行顺序使用mark来标记 //@pytest.mark.run(order=$NUM)
## 五、pytest分组执行 （冒烟，分模块，web分类，接口分类）
（smoke：冒烟用例，分布在各个模块中）
# pytest常用的前后置（固件，夹具）的处理方法
## 一、setup/teardown,setup_class/teardown_class
1. 为什么需要这些功能
    &ensp;&ensp;在测试用例执行前可能需要初始化，执行后需要清理工作，例如：web自动化测试中每个测试用例可能在执行前都需要打开浏览器，执行后关闭浏览器。可能在整个测试前需要创建日志对象，数据库对象等；测试后需要销毁对应的日志对象，数据库对象。
    ```
    class TestObjectA():
        def setup_class(self):
            print("class setup")
        def setup(self):
            print("setup")
        def test_a_func(self):
            print("test A func")
        def test_a_func(self):
            print("test A func")
        def test_b_func(self):
            print("test B func")
        def teardown(self):
            print("teardown")
        def teardown_class(self):
            print("class teardown")
            
    ```
## 二、使用fixture装饰器来实现部分用例的前后置
1. 功能实现
    ```
    @pytest.fixture(scope="function", autouse=False, params=[], ids="", name="")
    ```
1. 参数：  
    &ensp;&ensp;scope: 功能作用域，function,class,session/package  
    &ensp;&ensp;&ensp;&ensp;function:作用于函数，类似setup/teardown功能  
    &ensp;&ensp;&ensp;&ensp;class:作用于类，类似setup_class/teardown_class功能，有多个类执行多次  
    &ensp;&ensp;&ensp;&ensp;module:作用于模块，多个类执行一次  
    &ensp;&ensp;&ensp;&ensp;session:作用于函数  
    &ensp;&ensp;&ensp;&ensp;function:作用于函数  
    &ensp;&ensp;autouse: 自动执行，如：在function作用域下，无效调用，会每个function自动执行  
    &ensp;&ensp;params: 参数化(支持列表，元组，字典列表[{},{},{}]，字典元组[(),(),()])  
    &ensp;&ensp;ids: 当使用params参数化时，给每一个值设置一个变量名。  
    &ensp;&ensp;name: 给表示的是被@pytest.fixture()标记的方法设置一个别名，当设置了别名后无 法使用原有的名称  
3. 示例
    ```
    @pytest.fixture(scope="function", autouse=False)
    def my_fixture():
        print("这是前置方法")
        yield
        print("这是后置方法")
    class TestObjectA():
        def test_a(self):
            print("test a")
        def test_b(self, my_fixture):
            print("test b")
    ```
    ```
    @pytest.fixture(scope="class", autouse=False)
    def my_fixture():
        print("这是前置方法")
        yield
        print("这是后置方法")
    class TestObjectA():
        def test_a(self):
            print("test a")
        def test_b(self, my_fixture):
            print("test b")
    ```
    ```
    @pytest.fixture(scope="function", params=["1","2","3"],ids=["one","two","thr"])
    def my_fixture(request):
        print("这是前置方法")
        yield request.param
        print("这是后置方法")
    class TestObjectA():
        def test_a(self):
            print("test a")
        def test_b(self, my_fixture):
            print("test b")
    ```
## 三、通过conftest.py和@pytest.fixture()结合使用实现全局的前置应用
1. conftest.py文件是单独存放的夹具配置文件，名称无法修改
2. 用处：可以在不同的py文件中使用同一个fixture
3. 原则上conftest.py需要和运行的用例放到同一层，并且无需导入
4. conftest.py可以在不同的module下定义不同的congtest.py文件
## 四、pytest结合allure-pytest插件生成allure测试报告
1. 安装
2. pytest执行加入参数生成json格式的临时报告
   ```
   --alluredir ./tmp
   ```
3. 生成allure报告
   ```
   os.system("allure generate ./tmp -o ./report --clean")
   ```

