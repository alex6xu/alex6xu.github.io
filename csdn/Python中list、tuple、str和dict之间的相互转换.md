1、字典（dict） dict = {‘name’: ‘Zara’, ‘age’: 7, ‘class’: ‘First’} 1.1 字典——字符串 返回：
print type(str(dict)), str(dict)11 1.2 字典——元组 返回：(‘age’, ‘name’, ‘class’)
print tuple(dict)11 1.3 字典——元组 返回：(7, ‘Zara’, ‘First’) print
tuple(dict.values())11 1.4 字典——列表 返回：[‘age’, ‘name’, ‘class’] print
list(dict)11 1.5 字典——列表 print dict.values 2、元组 tup=(1, 2, 3, 4, 5) 2.1 元组——字符串
返回：(1, 2, 3, 4, 5) print tup.__str__()11 2.2 元组——列表 返回：[1, 2, 3, 4, 5] print
list(tup)11 2.3 元组不可以转为字典 3、列表 nums=[1, 3, 5, 7, 8, 13, 20]; 3.1 列表——字符串
返回：[1, 3, 5, 7, 8, 13, 20] print str(nums)11 3.2 列表——元组 返回：(1, 3, 5, 7, 8, 13,
20) print tuple(nums)11 3.3 列表不可以转为字典 4、字符串 4.1 字符串——元组 返回：(1, 2, 3) print
tuple(eval("(1,2,3)"))11 4.2 字符串——列表 返回：[1, 2, 3] print
list(eval("(1,2,3)"))11 4.3 字符串——字典 返回： print type(eval("{'name':'ljq',
'age':24}"))

