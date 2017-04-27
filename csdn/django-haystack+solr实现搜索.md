1.首先下载solr。http://mirror.bjtu.edu.cn/apache/lucene/solr/3.6.1/apache-
solr-3.6.1.zip下载解压即可 cd example/ java -jar start.jar
即可启动solr了，可以进入http://localhost:8983/solr/admin/看看效果。 ps.
这里有个地方要注意的，将solr/conf/stopwords.txt改名为stopwords_en.txt，不然会出错的 2.安装pysolr。sudo
pip install pysolr haystack就不介绍了，看上篇就知道了。 3.修改settings.py
HAYSTACK_CONNECTIONS = { 'default': { 'ENGINE':
'haystack.backends.solr_backend.SolrEngine', 'URL':
'http://127.0.0.1:8983/solr' }, } 4.新建models如下：  class Blog(models.Model):
title = models.CharField(max_length=30) content =
models.CharField(max_length=140) created_time =
models.DateTimeField(auto_now=True) def __unicode__(self): return self.title
5.在blog目录下新建search_indexes.py：  import datetime from haystack import indexes
from blog.models import Blog class BlogIndex(indexes.SearchIndex,
indexes.Indexable): text = indexes.CharField(document=True, use_template=True)
title = indexes.CharField(model_attr='title') content =
indexes.CharField(model_attr='content') def get_model(self): return Blog def
index_queryset(self): return self.get_model().objects.all()
6.新建templates/search/indexes/blog/blog_text.txt：  {{ object.title }} {{
object.content }} 7.在urls.py中加入(r’^$’, include(‘haystack.urls’)),
8.新建templates/search/search.html：

## Search

{{ form.as_table }}



{% if query %}

### Results

{% for result in page.object_list %}

Title: {{ result.object.title }}

Content: {{ result.object.content }}

{% empty %}

No results found.

{% endfor %} {% if page.has_previous or page.has_next %}

{% if page.has_previous %}[{% endif %}« Previous{% if page.has_previous
%}](?q={{ query }}&page={{ page.previous_page_number }}){% endif %} | {% if
page.has_next %}[{% endif %}Next »{% if page.has_next %}](?q={{ query
}}&page={{ page.next_page_number }}){% endif %}

{% endif %} {% else %} {# Show some example queries to run, maybe query
syntax, something else? #} {% endif %}  9.将schema.xml导入：python manage.py
build_solr_schema > /path_to_solr/example/solr/conf/ 10.重建索引：python manage.py
rebuild_index 大致流程就这样的。

