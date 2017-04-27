[python] view plain copy from haystack.query import SearchQuerySet
all_results = SearchQuerySet().all()   hello_results =
SearchQuerySet().filter(content='hello')   hello_world_results =
SearchQuerySet().filter(content='hello world')   unfriendly_results =
SearchQuerySet().exclude(content='hello').filter(content='world')
recent_results = SearchQuerySet().order_by('-pub_date')[:5]

