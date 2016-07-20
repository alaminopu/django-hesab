# Django Haystack Elasticsearch Autocomplete Backend

Custom Elasticsearch backend for django-haystack which makes it easier to customize 
Elasticsearch backend settings.
 
## Why?
I find myself having to subclass and customize the Elasticsearch backend in Haystack for
almost every single project I'm using it in. Specially, the autocomplete functionality bothers
me, and I struggled to figure out why it didn't "feel" like autocomplete. Well, it's because it really
isn't. The EdgeNGram analyzer does what it should, but to make it "feel" a bit more natural 
there's some tweaking that has to be done.

1. Create a filter that triggers at the first character
2. Create an analyzer that use the filter
3. Create a field mappings that use the new analyzer to index, and the standard analyzer to search.

So I have simply packaged up what I've got for others to use...

## How?

Simply use the default tweaks shipped in this package, or write your own.
Declare it in `settings.py` under the `ES_BACKEND_SETTINGS` namespace like this:

```python
ES_BACKEND_SETTINGS = {
    'INDEX_SETTINGS: {
        'settings: {
            'number_of_shards': 2,
            analysis: {
                'analyzer': {
                    'my_analyzer': {
                        'type': 'custom',
                        'filter': ['my_filter'],
                        ...
                    }
                },
                'filter: {
                    'my_filter': {
                        'type': 'edgeNgram',
                        ...
                    }
                }
            }
        }
    },
    'FIELD_MAPPINGS': {
        'my_field_mapping': {
            'type': 'string',
            'index_analyzer': 'my_analyzer',
            ...
        }
    }
}
```

### Start using it

```python
#
# search_indexes.py
#

from haystack import indexes
from hesab.fields import AutocompleteEdgeNgramField

class MySearchIndex(indexes.SearchIndex, indexes.Indexable):

    text = indexes.Charfield(document=True, use_template=True)
    autocomplete = AutocompleteEdgeNgramField()
    
    def get_model(self):
        return Address
        
    @staticmethod
    def prepare_autocomplete(obj):
        autocomplete " ".join((obj.street_address, obj.city, obj.zip_code))
        return " ".join(set(autocomplete.split()))


```
