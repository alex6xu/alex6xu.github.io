其实就是动态的获取类的属性和方法 class Person: def __init__(self): self.name = "zjgtan" def
getName(self): return self.name 反射的简单含义： 　　通过类名获得类的实例对象 　　通过方法名得到方法，实现调用
反射方法一： from person import Person theObj = globals()["Person"]() print
theObj.getName() 反射方法二： module = __import__("person") theObj = getattr(module,
"Person")() print theObj.getName()

