In some cases the we might want to store generic model object, rather a
particular specific model as 'ForeignKey'. Here is scenario of such kind.
Suppose there are models like User, Project, Ticket and TimeLine as below.
class Ticket(models.Model): name = models.CharField(max_length=200,
verbose_name=_("name")) slug = models.SlugField(max_length=250, null=False,
blank=True, verbose_name=_("slug")) class Project(models.Model): name =
models.CharField(max_length=200, verbose_name=_("name")) slug =
models.SlugField(max_length=250, null=False, blank=True,
verbose_name=_("slug")) class User(models.Model): name =
models.CharField(max_length=200, verbose_name=_("name")) slug =
models.SlugField(max_length=250, null=False, blank=True,
verbose_name=_("slug")) class Timeline(models.Model): involved_object = *****
event_type = models.CharField(max_length=250, default="created") If we want to
store the user activities with these models like "created User, created
Project, created Task" in time line, we have to create all the three models(
User, Project, Task)as ' ForeignKey' fields. This is not a good programming
practice. To overcome this,  Django's contenttypes framework  provides a
special field type ( GenericForeignKey) which allows the relationship to be
with any model.  Using 'GenericForeignKey' : Here is the solution for the
above scenario. There are three parts to setting up a GenericForeignKey: Give
your model a ForeignKeyto ContentType. The usual name for this field is
“content_type”. Give your model a field that can store primary key values from
the models you’ll be relating to. For most models, this means a
PositiveIntegerField. The usual name for this field is “object_id”. Give your
model a GenericForeignKey, and pass it the names of the two fields described
above. If these fields are named “content_type” and “object_id”, you can omit
this – those are the default field names GenericForeignKey will look for. So
by following the above three steps the TimeLine will look as following. class
Timeline(models.Model): content_type = models.ForeignKey(ContentType,
related_name="content_type_timelines") object_id =
models.PositiveIntegerField() content_object =
GenericForeignKey('content_type', 'object_id') event_type =
models.CharField(max_length=250, default="created") For example if we want to
store the event "created Project" after the project is created, the following
snippet will do things for us. t1 = TimeLine(content_object=project_object)
t1.save() But due to the way 'GenericForeignKey' is implemented, you cannot
use such fields directly with filters (filter() and exclude(), for example)
via the database API. Because a GenericForeignKey isn’t a normal field object,
the following examples will not work:
TimeLine.objects.filter(content_object=project_object) # This will also fail
TimeLine.objects.get(content_object=project_object) To retrieve the TimeLine
object with the project object, we have to follow these steps. Get the
'ContentType' object with the follwoing code.  from
django.contrib.contenttypes.models import ContentType contenttype_obj =
ContentType.objects.get_for_model(project_object) "object_id" is stored with
project_object.id TimeLine.objects.filter(object_id=project_object.id,
content_type=contenttype_obj)

