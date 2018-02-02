#  面向对象suite

Python 的面向对象设计特性允许我们在设计和构建 [suite definition](https://software.ecmwf.int/wiki/display/ECFLOW/Glossary#term-suite-definition) 时考虑灵活性。
设计每个 suite 有不同的准则。让我们考虑如何以一种更面向对象的方式设计本教程的例子。我们以一些设计准则开始。

* 默认变量（例如 `ECF_HOME` 等）必须设置，并与 suite 独立
* 新的 suite 必须开启自动检测 job 生成。
* 需要将 definition 保存到单独的文件。
* 新的 suite 应该能够复用为上面要求而设计的代码

下面是一种可行的设计，使用单例模式和模板模式。

```python
import os
import ecflow  

class DefaultVariables(object):
    """Provide the setup variables for each suite"""
    def add_to(self, node):
        """Adds ECF_INCLUDE,ECF_HOME to the input node"""
        node.add_variable("ECF_INCLUDE", os.path.join(os.getenv("HOME"),  "course"))
        node.add_variable("ECF_HOME", os.path.join(os.getenv("HOME"),  "course"))


class BaseSuiteBuilder(object):
    """Abstract class. Add default variables to suite and enable job creation 
 checking for any derived suite
 """
    def __init__(self, default_variables):
        self.defs = ecflow.Defs()
        # use derived class name as suite name
        self.suite = self.defs.add_suite(type(self).__name__) 
        default_variables.add_to(self.suite)
        
    def _build_definition_hook(self):
        """Derived suite should override this function to build the suite
 Should not be called explicitly. How could we enforce this ?
 """
        pass
    
    def setup(self):
        """Template/skeleton function. 
 Provides common algorithm for *all* derivatives
 Uses Hollywood principle.
 """
        
        # Build the suite
        self._build_definition_hook()
        
        # check job creation. Could use an assert
        job_check_result = self.defs.check_job_creation()
        if len(job_check_result) != 0:
            print "Job creation failed\n" + job_check_result
 
        # Check trigger expressions and limit references"
        print self.defs.check()
                   
        # Allows definition creation to be separated from the load
        # Use the class name as the name of the definition file
        self.defs.save_as_defs( type(self).__name__ + ".def")
        
        
class SuiteBuilder(BaseSuiteBuilder):
    """My example suite. Generates SuiteBuilder.def"""
    def __init__(self, default_variables):
        BaseSuiteBuilder.__init__(self, default_variables)
        
    def _build_definition_hook(self):
        f1 = self.suite.add_family("family")
        f1.add_task("task1")           
        f1.add_task("task2") 
        

class TestSuite(BaseSuiteBuilder):
    """My test suite. Generates TestSuite.def"""
    def __init__(self, default_variables):
        BaseSuiteBuilder.__init__(self, default_variables)
        
    def _build_definition_hook(self):
        self.suite.add_task("task1")


if __name__ == "__main__":

    my_suite = SuiteBuilder(DefaultVariables())
    my_suite.setup()

    my_test_suite = TestSuite(DefaultVariables())
    my_test_suite.setup()
```
