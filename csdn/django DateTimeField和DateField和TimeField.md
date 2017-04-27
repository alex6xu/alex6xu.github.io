需要了解跟时间相关的三个modelField， DateTimeField和DateField和TimeField
存储的内容分别对应着datetime(),date(),time()三个对象。
对于auto_now和auto_now_add。两者默认值都为False。     auto_now=Ture，字段保存时会自动保存当前时间，但要注意每次对
其实例执行save()的时候都会将当前时间保存，也就是不能再手动给它存非当前时间的值。     auto_now_add=True，字段在实例第一次保存的时
候会保存当前时间，不管你在这里是否对其赋值。但是之后的save()是可以手动赋值的。也就是新实例化一个model，想手动存其他时间，就需要对该实例save(
)之后赋值然后再save()。

