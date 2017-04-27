Python from django.db import models class Province(models.Model): name =
models.CharField(max_length=10) def __unicode__(self): return self.name class
City(models.Model): name = models.CharField(max_length=5) province =
models.ForeignKey(Province) def __unicode__(self): return self.name class
Person(models.Model): firstname = models.CharField(max_length=10) lastname =
models.CharField(max_length=10) visitation = models.ManyToManyField(City,
related_name = "visitor") hometown = models.ForeignKey(City, related_name =
"birth") living = models.ForeignKey(City, related_name = "citizen") def
__unicode__(self): return self.firstname + self.lastname 1 2 3 4 5 6 7 8 9 10
11 12 13 14 15 16 17 18 19 20 21 from django.db import models   class
Province(models.Model):     name = models.CharField(max_length=10)     def
__unicode__(self):         return self.name   class City(models.Model):
name = models.CharField(max_length=5)     province =
models.ForeignKey(Province)     def __unicode__(self):         return
self.name   class Person(models.Model):     firstname  =
models.CharField(max_length=10)     lastname   =
models.CharField(max_length=10)     visitation = models.ManyToManyField(City,
related_name = "visitor")     hometown   = models.ForeignKey(City,
related_name = "birth")     living     = models.ForeignKey(City, related_name
= "citizen")     def __unicode__(self):         return self.firstname \+
self.lastname

