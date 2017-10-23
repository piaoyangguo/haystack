django-haystackȫ�ļ�����ϸ�̳�
��ǩ�� DjangohaystackpythonWebǰ��
2016-10-20 21:28 3703���Ķ� ����(5) �ղ� �ٱ�
 ���ࣺ webǰ�ˣ�4��   django��1��  
��Ȩ����������Ϊ����ԭ�����£�δ������������ת�ء�
Ŀ¼(?)[+]
ǰ����Ҫ��Django-haystack��ʵ���������ܣ�����һ��������Դ��֮���٣���˵�йٷ��ĵ��������Ŷ���������ЩӢ����ͬѧ��˵Ҫ���������һ�����¡��ر��ǹ��ڸ߼����֣��ص����˸�Ӣ��רҵ�������룬Ҳû�ܿ�������������רҵ��ʵ����̫ǿ�ˣ�������ȫ������������
���ǣ�����һЩСվ��Ŀ�����˵��������Ҫ����ҽ�����ȫ�㹻���ˣ�ֻ������ʱ��ȷʵ�鷳�㡣���ˣ��Թ�������Ϊ�˽�Լʱ�䣬�����ò��ִ���������һƪ���ͣ������������û�н������õ����ã����һ��������ϸ���Ͳ���չ��
ת�ز������ԣ����������
һ��ʹ�õĹ���
haystack��django�Ŀ�Դ������ܣ��ÿ��֧��Solr,Elasticsearch,Whoosh, *Xapian*�������棬���ø��Ĵ��룬ֱ���л����棬���ٴ�������
��������ʹ��Whoosh������һ���ɴ�Pythonʵ�ֵ�ȫ���������棬û�ж������ļ��ȣ��Ƚ�С�ɣ����ñȽϼ򵥣���Ȼ������Ȼ�Ե͡�
���ķִ�Jieba������Whoosh�Դ�����Ӣ�ķִʣ������ĵķִ�֧�ֲ���̫�ã�����jieba�滻whoosh�ķִ������
������Python 2.7 or 3.4.4, Django 1.8.3�������ϣ�Debian 4.2.6_3
��������˵��
���ڼ������ǵ���Ŀ����Project,��һ��myapp��app�����Ե�Ŀ¼�ṹ���¡�
Project
Project
settings.py
blog
models.py
��models.py�����ݼ������£�
[python] view plain copy print?
from django.db import models  
from django.contrib.auth.models import User  
  
  
class Note(models.Model):  
    user = models.ForeignKey(User)  
    pub_date = models.DateTimeField()  
    title = models.CharField(max_length=200)  
    body = models.TextField()  
  
    def __str__(self):  
        return self.title  
1. ���Ȱ�װ������
pip install whoosh django-haystack jieba
2. ��� Haystack ��Django�� INSTALLED_APPS
����Django��Ŀ��settings.py�����INSTALLED_APPS���Haystack,���ӣ�

[python] view plain copy print?
INSTALLED_APPS = [   
        'django.contrib.admin',  
        'django.contrib.auth',   
        'django.contrib.contenttypes',   
        'django.contrib.sessions',   
        'django.contrib.sites',   
  
          # Added. haystack����ӣ�  
          'haystack',   
          # Then your usual apps... �Լ���appҪд��haystakc����  
          'blog',  
]  

3. �޸� ��� settings.py������������
���̳�ʹ�õ���Whoosh�����������£�

[python] view plain copy print?
import os  
HAYSTACK_CONNECTIONS = {  
    'default': {  
        'ENGINE': 'haystack.backends.whoosh_backend.WhooshEngine',  
        'PATH': os.path.join(os.path.dirname(__file__), 'whoosh_index'),  
    },  
}  
���й���˼�壬ENGINEΪʹ�õ��������Ҫ�У����������Whoosh����PATH����Ҫ��д����ΪWhoosh �����ļ��Ĵ���ļ��С�
������������ü��ٷ��ĵ�
4.��������
����������ĳ��app����mainapp��ȫ�ļ������������mainapp��Ŀ¼���潨��search_indexes.py�ļ����ļ��������޸ġ��������£�


[python] view plain copy print?
import datetime  
from haystack import indexes  
from myapp.models import Note  
  
class NoteIndex(indexes.SearchIndex, indexes.Indexable):     #��������Ϊ��Ҫ������Model_name+Index��������Ҫ����Note�����Դ���NoteIndex  
    text = indexes.CharField(document=True, use_template=True)  #����һ��text�ֶ�  
  
    author = indexes.CharField(model_attr='user')   #����һ��author�ֶ�  
  
    pub_date = indexes.DateTimeField(model_attr='pub_date')  #����һ��pub_date�ֶ�  
  
    def get_model(self):          #����get_model����������Ҫ�У�  
        return Note  
  
    def index_queryset(self, using=None):   #����index_..����  
        """Used when the entire index for model is updated."""  
        return self.get_model().objects.filter(pub_date__lte=datetime.datetime.now())  

ΪʲôҪ��������������������һ�����Ŀ¼������Ϊ�����ṩ�����ٵĵ�������ҡ�������Ҳ��ͬ���ĵ������������ǳ����ʱ����Ҫ����Щ�������ҳ����е��������������ļ����ǲ�̫���ܵģ��������������������ĸ���������������ҪΪָ�����������һ��������Ŀ¼������������ΪNote����һ��������������ʵ��ϸ�������ǲ���Ҫ���ĵģ�����Ϊ������Щ�ֶδ�����������ôָ����������Ҫ����ҽ��ģ�Ҳ�������������ᵽ�ġ�

ÿ�����������������ֻ����һ���ֶ�Ϊ document=True�������haystack ���������潫ʹ�ô��ֶε�������Ϊ�������м���(primary field)���������ֶ�ֻ�Ǹ��������ԣ�������ã�������Ϊ�������ݡ�ֱ�����Լ����һ����������Ҳû���õ���Щ�������ԣ����������ԾͶ�ɾ���ˣ����ѧϰ��ʱ��Ҳ������ע�͵����ܡ�����������Ҳ�����ף�������û���ϡ�
ע�⣺���ʹ��һ���ֶ�������document=True����һ��Լ�����ֶ���Ϊtext��������SearchIndex������һ����������Է�ֹ��̨���ң���Ȼ������Ҳ�������ģ�����������ġ�
���ң�haystack�ṩ��use_template=True��text�ֶΣ���������������ʹ������ģ��ȥ�������������������ļ���˵��ͨ�׵��������������Ҫ���һЩʲô���������� Note �� title �ֶΣ��������ǿ���ͨ�� title ���������� Note �����ˣ��ٸ����ӣ����������� python ����ô�Ϳ��Լ���������title���� python ��Note�ˣ���ô���ǲ��Ǻܼ򵥣�����ģ���·��Ϊtemplates/search/indexes/yourapp/note_text.txt���Ƽ�����Ŀ��Ŀ¼����һ��templates������settings.py��Ϊ�����룬ʹ��django������templates��Ѱ��ģ�壬��Ȼ��ֻҪ�����κ�һ�����Django����������tempaltes����ͺã�����������벻�����������۵ķ��룩��templates/search/indexes/blog/note_text.txt�ļ�������ΪҪ����������_text.txt,������Ϊ

[python] view plain copy print?
{{ object.title }}  
{{ object.user.get_full_name }}  
{{ object.body }}  

�������ģ��������Ƕ�Note.title, Note.user.get_full_name,Note.body�������ֶν�����������������ʱ�����������ֶ���ȫ�ļ���ƥ�䡣�����Ѿ���������ˡ�

5.��URL���������SearchView��������ģ��
��urls.py����������url��Ϣ����Ȼurl·�ɿ�������д��

[python] view plain copy print?
(r'^search/', include('haystack.urls')),  

��ʵhaystack.urls������Ϊ

[python] view plain copy print?
from django.conf.urls import url  
from haystack.views import SearchView  
  
urlpatterns = [  
    url(r'^$', SearchView(), name='haystack_search'),  
]  

SearchView()��ͼ����Ĭ��ʹ�õ�htmlģ��·��Ϊtemplates/search/search.html����˵һ���Ƽ��ڸ�Ŀ¼����templates������settings.py�����úã�
������Ҫ��templates/search/�����search.html�ļ�������Ϊ

[html] view plain copy print?
<h2>Search</h2>  
  
<form method="get" action=".">  
    <table>  
        {{ form.as_table }}  
        <tr>  
            <td> </td>  
            <td>  
                <input type="submit" value="Search">  
            </td>  
        </tr>  
    </table>  
  
    {% if query %}  
        <h3>Results</h3>  
  
        {% for result in page.object_list %}  
            <p>  
                <a href="{{ result.object.get_absolute_url }}">{{ result.object.title }}</a>  
            </p>  
        {% empty %}  
            <p>No results found.</p>  
        {% endfor %}  
  
        {% if page.has_previous or page.has_next %}  
            <div>  
                {% if page.has_previous %}<a href="?q={{ query }}&page={{ page.previous_page_number }}">{% endif %}? Previous{% if page.has_previous %}</a>{% endif %}  
                |  
                {% if page.has_next %}<a href="?q={{ query }}&page={{ page.next_page_number }}">{% endif %}Next ?{% if page.has_next %}</a>{% endif %}  
            </div>  
        {% endif %}  
    {% else %}  
        {# Show some example queries to run, maybe query syntax, something else? #}  
    {% endif %}  
</form>  

�����ԣ����Դ��˷�ҳ��
Ȼ��Ϊ��ҽ���һ������ļ������ȿ��Կ���ģ����ʹ���˵ı����� form,query,page ������һ������˵һ�¡�

form�������ԣ�����django���form���ǲ��ģ�������Ⱦ��һ�������ı��������ù�Django��Form��֪��������Ҳ����˵�ˣ������׵Ŀ���ȥ��Django�ĵ�����Ȼ��ʵ�ҵ����Ҳû���ϣ�������Լ�д�˸�<form></form>���ṩ��ȷ�Ĳ�����name="seach"��method="get"�Լ����action��ַ��OK�ˡ����������Ҫ�õ������������������˵Ļ����ܾ�Ҫ�Զ���Form���ˣ�����ͨ����������ӿ��Կ�����Ĭ�ϵ�formҲ���ṩһ���򵥵Ĺ������ģ����Թ���ѡ����Щmodel����Ҫ�����ģ����һ���������Ļ�Ĭ��ȫ����������Ȼ����Ҳ�ǿ����Լ�����html��ģ�����form�ģ�������Ҫʵ��model���˻��Ǻܼ򵥵ģ�ֻҪģ��һ�����Form�����ݾͺ��ˣ���ֻ������haystack���ܹ��������Ӧ��Form���������м�������ʵ��django��Form��һ���ģ�Form��һ�����Ҽ�������Ƿ�Ϸ��Ĺ��ܣ�haystackҲһ�������������ƪ���²�����˵����Ϊ��Ҳ��̫���ף�2333��������ϸ��ȥ���ĵ��������ĵ��Ϲ���View&Form��һ�ڻ��ǱȽ�ͨ���׶��ģ��ʻ���Ҫ��Ҳ���Ǻܸߣ����������Ҷ�������һЩ������

query����������������ַ�����

����page�����Կ���page��object_list���ԣ�����һ��list����������˵�һҳ��Ҫչʾ��model���󼯺ϣ���ôlist���浽���ж��ٸ��أ�������Ҫ�Լ����Ƹ�����ô���أ����õ��ģ�haystackΪ�����ṩ��һ���ӿڡ�����ֻҪ��settings.py�����ã�


[python] view plain copy print?
#����ÿҳ��ʾ����Ŀ��Ĭ��Ϊ20�������Լ��޸�  
HAYSTACK_SEARCH_RESULTS_PER_PAGE  =  8  

Ȼ����ڷ�ҳ�Ĳ��֣���ҿ�����Ӧ��Ҳ�ܿ����ɡ�
�����Ҫ֪�������Ĭ��context���ı����������Լ�����Դ��views.py���SearchView����ͼ�����Ŷ��ܿ�����

��ô�������ˡ�����һ��searchҳ����˵�����ǿ϶�����Ҫ�õ������Զ���� context ���ݣ���ô���¸���ô���أ�������뵽�İ취�����޸�haystackԴ�룬Ϊ������ϸ���� context ���ݣ������ǲ���Ҳ�й�����һ�����뷨�أ���������������׾���޴������ǲ�����Ҫע����ֻ�����������ϵ�����ҵ����������������仯ʱ���ѵ����ǻ�Ҫ�� haystack Ҳ���ƹ�ȥ���ɣ�����̫�޴��ˣ�ͻȻ�����뵽��Ȼ�Ҳ����޸�Դ�룬�ѵ��һ����ܸ���Դ����֮�����ÿ���һ�¹ٷ��ĵ�������������ģ�ͨ���̳�SeachView��ʵ������ context �����ݡ��ٷ��ĵ��ṩ��2���汾��SearchView�����ʼ�õ����°�ģ��������ˣ�Ҳ����ȥ�Ҵ�����ʲô������ˣ�ֱ��ʹ�õ��˾ɰ汾��SearchView��ֻҪ������haystack��2���汾���Ǹ��㰲װ���˵ġ�����������myappĿ¼���ٴ���һ��search_views.py �ļ���λ�����ֿ����Լ���������д�Լ���������ͼ������ʵ�����£�


[python] view plain copy print?
from haystack.views import SearchView  
from .models import *  
  
class MySeachView(SearchView):  
    def extra_context(self):       #����extra_context����Ӷ����context����  
        context = super(MySeachView,self).extra_context()  
        side_list = Topic.objects.filter(kind='major').order_by('add_date')[:8]  
        context['side_list'] = side_list  
        return context  


Ȼ�����޸�urls.py��search����ӳ�䵽MySearchView��

[python] view plain copy print?
url(r'^search/', search_views.MySeachView(), name='haystack_search'),  

�����������ı�����������������һ��ģ���ǩ��haystackΪ�����ṩ�� {% highlight %}�� {% more_like_this %} 2����ǩ��������ֻΪ�����ϸ������ highlight��ʹ�á�
���Ƿ�Ҳ�����Լ��ļ����Ͱٶ�����һ������ƥ�䵽������Ҳ������ʾ�أ� {% highlight %} Ϊ�����ṩ��������ܣ���Ȼ�����������ǩ��ò�ƻ���һ��HighLight�࣬����Լ����ĵ�ȥ�ɣ���Ӣ���������ף���

Syntax��

[python] view plain copy print?
{% highlight <text_block> with <query> [css_class "class_name"] [html_tag "span"] [max_length 200] %}  
�����˼��Ϊ text_block ��� query �������css_class��html_tag����max_length Ϊ���շ��س��ȣ��൱�� cut ���ҿ���һ�´˱�ǩʵ��Դ�룬Ĭ�ϵ�html_tag ֵΪ span ��css_class ֵΪ highlighted��max_length ֵΪ 200��Ȼ��Ϳ���ͨ��CSS�����Ч������Ĭ��ʱ��


[css] view plain copy print?
span.highlighted {  
        color: red;  
}  


Example��


[python] view plain copy print?
# ʹ��Ĭ��ֵ  
{% highlight result.summary with query %}  
  
# ��������Ϊ {{ result.summary }}�����е� {{ query }} ָ����һ��<div></div>��ǩ�����ҽ�class����Ϊhighlight_me_please�������Ϳ����Լ�ͨ��CSSΪ{{ query }}��Ӹ���Ч���ˣ���ô�����ǲ��Ǻܿ�ѧ��  
{% highlight result.summary with query html_tag "div" css_class "highlight_me_please" %}  
  
# ���������������{{ result.summary }}�����������ĳ���  
{% highlight result.summary with query max_length 40 %}  


���ˣ���ĿǰΪֹ������������������֪ʶ�Ļ������Ѿ�������һ���Ƚ�����������������ˣ����������Ǵ���index�ļ��ˡ�

6.���һ�����ؽ������ļ�
ʹ��python manage.py rebuild_index����ʹ��update_index���

�ã�����������Ŀ�������url����һ�����԰ɡ�

ÿ�����ݿ���º���Ҫ��������������haystackΪ����ṩ��һ���ӿڣ�ֻҪ��settings.py�����ã�

[python] view plain copy print?
#�Զ���������  
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'  


��������Ҫ���ģ�ʹ��jieba�ִ�

1 ���ļ�whoosh_backend.py�����ļ�·��Ϊpython·��/lib/python2.7.5/site-packages/haystack/backends/whoosh_backend.py��������app���棬��������Ϊwhoosh_cn_backend.py������blog/whoosh_cn_backend.py��

�޸�Ϊ����

[python] view plain copy print?
from jieba.analyse import ChineseAnalyzer   #�ڶ������  
  
schema_fields[field_class.index_fieldname] = TEXT(stored=True, analyzer=ChineseAnalyzer(),field_boost=field_class.boost, sortable=True)   #ע�����ҵ�������޸ģ�������ֱ�����  

2 ��settings.py���޸����棬����

[python] view plain copy print?
import os  
HAYSTACK_CONNECTIONS = {  
    'default': {  
        'ENGINE': 'blog.whoosh_cn_backend.WhooshEngine',      #blog.whoosh_cn_backend������ո���ӵ��ļ�  
        'PATH': os.path.join(BASE_DIR, 'whoosh_index'  
    },  
}  

3 �ؽ��������ڽ��������������԰ɡ�



��.highlight����
����д���ˣ������ᱼ������������ҿ����ҵ�

��ô�������аɣ��ۼ���˻ᷢ�֣�Ϊʲô������ĸߵ�û�б��滻��...�������������ѧ֮ǰ������ȴ���滻����...�����Ȿ���ͺ̣ܶ�����һ�£����Ǹߵ���ѧ����ʾ������ѧ���ǲ��Ƕ�ʧ������Ҫ����Ϣ�أ��ߵ���ô��Ҫ�����۶���ʡ���ˣ�����Ȼ�ǲ��еģ��Ͼ����Ǹ��ߵ�������ô��ô���أ���û��ѡ��ȥ���ĵ��������ĵ���HighLight���������������İɣ�������ѡ���˶�highlight ��ǩ��Դ�룬���ջ�������ʵ���ˡ�

������Ҫ�����Ǹ���ճ��Դ�룬Ȼ������޸ģ�������ѡ��ֱ�Ӹ�Դ�룬����һ���Լ��ı�ǩ��Ϊ��ҷ��ϡ����myapp/templatetags/my_filters_and_tags.py �ļ��� myapp/templatetags/highlighting.py �ļ����������£�Դ��ֱ�λ��haystack/templatetags/lighlight.py �� haystack/utils/lighlighting.py �У���

my_filter_and_tags.py:
[python] view plain copy print?
# encoding: utf-8  
from __future__ import absolute_import, division, print_function, unicode_literals  
  
from django import template  
from django.conf import settings  
from django.core.exceptions import ImproperlyConfigured  
from django.utils import six  
  
from haystack.utils import importlib  
  
register = template.Library()  
  
class HighlightNode(template.Node):  
    def __init__(self, text_block, query, html_tag=None, css_class=None, max_length=None, start_head=None):  
        self.text_block = template.Variable(text_block)  
        self.query = template.Variable(query)  
        self.html_tag = html_tag  
        self.css_class = css_class  
        self.max_length = max_length  
        self.start_head = start_head  
  
        if html_tag is not None:  
            self.html_tag = template.Variable(html_tag)  
  
        if css_class is not None:  
            self.css_class = template.Variable(css_class)  
  
        if max_length is not None:  
            self.max_length = template.Variable(max_length)  
  
        if start_head is not None:  
            self.start_head = template.Variable(start_head)  
  
    def render(self, context):  
        text_block = self.text_block.resolve(context)  
        query = self.query.resolve(context)  
        kwargs = {}  
  
        if self.html_tag is not None:  
            kwargs['html_tag'] = self.html_tag.resolve(context)  
  
        if self.css_class is not None:  
            kwargs['css_class'] = self.css_class.resolve(context)  
  
        if self.max_length is not None:  
            kwargs['max_length'] = self.max_length.resolve(context)  
  
        if self.start_head is not None:  
            kwargs['start_head'] = self.start_head.resolve(context)  
  
        # Handle a user-defined highlighting function.  
        if hasattr(settings, 'HAYSTACK_CUSTOM_HIGHLIGHTER') and settings.HAYSTACK_CUSTOM_HIGHLIGHTER:  
            # Do the import dance.  
            try:  
                path_bits = settings.HAYSTACK_CUSTOM_HIGHLIGHTER.split('.')  
                highlighter_path, highlighter_classname = '.'.join(path_bits[:-1]), path_bits[-1]  
                highlighter_module = importlib.import_module(highlighter_path)  
                highlighter_class = getattr(highlighter_module, highlighter_classname)  
            except (ImportError, AttributeError) as e:  
                raise ImproperlyConfigured("The highlighter '%s' could not be imported: %s" % (settings.HAYSTACK_CUSTOM_HIGHLIGHTER, e))  
        else:  
            from .highlighting import Highlighter  
            highlighter_class = Highlighter  
  
        highlighter = highlighter_class(query, **kwargs)  
        highlighted_text = highlighter.highlight(text_block)  
        return highlighted_text  
 
 
@register.tag  
def myhighlight(parser, token):  
    """ 
    Takes a block of text and highlights words from a provided query within that 
    block of text. Optionally accepts arguments to provide the HTML tag to wrap 
    highlighted word in, a CSS class to use with the tag and a maximum length of 
    the blurb in characters. 
 
    Syntax:: 
 
        {% highlight <text_block> with <query> [css_class "class_name"] [html_tag "span"] [max_length 200] %} 
 
    Example:: 
 
        # Highlight summary with default behavior. 
        {% highlight result.summary with request.query %} 
 
        # Highlight summary but wrap highlighted words with a div and the 
        # following CSS class. 
        {% highlight result.summary with request.query html_tag "div" css_class "highlight_me_please" %} 
 
        # Highlight summary but only show 40 characters. 
        {% highlight result.summary with request.query max_length 40 %} 
    """  
    bits = token.split_contents()  
    tag_name = bits[0]  
  
    if not len(bits) % 2 == 0:  
        raise template.TemplateSyntaxError(u"'%s' tag requires valid pairings arguments." % tag_name)  
  
    text_block = bits[1]  
  
    if len(bits) < 4:  
        raise template.TemplateSyntaxError(u"'%s' tag requires an object and a query provided by 'with'." % tag_name)  
  
    if bits[2] != 'with':  
        raise template.TemplateSyntaxError(u"'%s' tag's second argument should be 'with'." % tag_name)  
  
    query = bits[3]  
  
    arg_bits = iter(bits[4:])  
    kwargs = {}  
  
    for bit in arg_bits:  
        if bit == 'css_class':  
            kwargs['css_class'] = six.next(arg_bits)  
  
        if bit == 'html_tag':  
            kwargs['html_tag'] = six.next(arg_bits)  
  
        if bit == 'max_length':  
            kwargs['max_length'] = six.next(arg_bits)  
  
        if bit == 'start_head':  
            kwargs['start_head'] = six.next(arg_bits)  
  
    return HighlightNode(text_block, query, **kwargs)  
lighlighting.py:
[python] view plain copy print?
# encoding: utf-8  
  
from __future__ import absolute_import, division, print_function, unicode_literals  
  
from django.utils.html import strip_tags  
  
  
class Highlighter(object):  
    #Ĭ��ֵ  
    css_class = 'highlighted'  
    html_tag = 'span'  
    max_length = 200  
    start_head = False  
    text_block = ''  
  
    def __init__(self, query, **kwargs):  
        self.query = query  
  
        if 'max_length' in kwargs:  
            self.max_length = int(kwargs['max_length'])  
  
        if 'html_tag' in kwargs:  
            self.html_tag = kwargs['html_tag']  
  
        if 'css_class' in kwargs:  
            self.css_class = kwargs['css_class']  
  
        if 'start_head' in kwargs:  
            self.start_head = kwargs['start_head']  
  
        self.query_words = set([word.lower() for word in self.query.split() if not word.startswith('-')])  
  
    def highlight(self, text_block):  
        self.text_block = strip_tags(text_block)  
        highlight_locations = self.find_highlightable_words()  
        start_offset, end_offset = self.find_window(highlight_locations)  
        return self.render_html(highlight_locations, start_offset, end_offset)  
  
    def find_highlightable_words(self):  
        # Use a set so we only do this once per unique word.  
        word_positions = {}  
  
        # Pre-compute the length.  
        end_offset = len(self.text_block)  
        lower_text_block = self.text_block.lower()  
  
        for word in self.query_words:  
            if not word in word_positions:  
                word_positions[word] = []  
  
            start_offset = 0  
  
            while start_offset < end_offset:  
                next_offset = lower_text_block.find(word, start_offset, end_offset)  
  
                # If we get a -1 out of find, it wasn't found. Bomb out and  
                # start the next word.  
                if next_offset == -1:  
                    break  
  
                word_positions[word].append(next_offset)  
                start_offset = next_offset + len(word)  
  
        return word_positions  
  
    def find_window(self, highlight_locations):  
        best_start = 0  
        best_end = self.max_length  
  
        # First, make sure we have words.  
        if not len(highlight_locations):  
            return (best_start, best_end)  
  
        words_found = []  
  
        # Next, make sure we found any words at all.  
        for word, offset_list in highlight_locations.items():  
            if len(offset_list):  
                # Add all of the locations to the list.  
                words_found.extend(offset_list)  
  
        if not len(words_found):  
            return (best_start, best_end)  
  
        if len(words_found) == 1:  
            return (words_found[0], words_found[0] + self.max_length)  
  
        # Sort the list so it's in ascending order.  
        words_found = sorted(words_found)  
  
        # We now have a denormalized list of all positions were a word was  
        # found. We'll iterate through and find the densest window we can by  
        # counting the number of found offsets (-1 to fit in the window).  
        highest_density = 0  
  
        if words_found[:-1][0] > self.max_length:  
            best_start = words_found[:-1][0]  
            best_end = best_start + self.max_length  
  
        for count, start in enumerate(words_found[:-1]):  
            current_density = 1  
  
            for end in words_found[count + 1:]:  
                if end - start < self.max_length:  
                    current_density += 1  
                else:  
                    current_density = 0  
  
                # Only replace if we have a bigger (not equal density) so we  
                # give deference to windows earlier in the document.  
                if current_density > highest_density:  
                    best_start = start  
                    best_end = start + self.max_length  
                    highest_density = current_density  
  
        return (best_start, best_end)  
  
    def render_html(self, highlight_locations=None, start_offset=None, end_offset=None):  
        # Start by chopping the block down to the proper window.  
        #text_blockΪ���ݣ�start_offset,end_offset�ֱ�Ϊ��һ��ƥ��query��ʼ�Ͱ����Ƚض�λ��  
        text = self.text_block[start_offset:end_offset]  
  
        # Invert highlight_locations to a location -> term list  
        term_list = []  
  
        for term, locations in highlight_locations.items():  
            term_list += [(loc - start_offset, term) for loc in locations]  
  
        loc_to_term = sorted(term_list)  
  
        # Prepare the highlight template  
        if self.css_class:  
            hl_start = '<%s class="%s">' % (self.html_tag, self.css_class)  
        else:  
            hl_start = '<%s>' % (self.html_tag)  
  
        hl_end = '</%s>' % self.html_tag  
  
        # Copy the part from the start of the string to the first match,  
        # and there replace the match with a highlighted version.  
        #matched_so_far�������Ϊtext�����һ��ƥ��query�Ľ�β  
        highlighted_chunk = ""  
        matched_so_far = 0  
        prev = 0  
        prev_str = ""  
  
        for cur, cur_str in loc_to_term:  
            # This can be in a different case than cur_str  
            actual_term = text[cur:cur + len(cur_str)]  
  
            # Handle incorrect highlight_locations by first checking for the term  
            if actual_term.lower() == cur_str:  
                if cur < prev + len(prev_str):  
                    continue  
  
                #�ֱ�����ÿ��query+������һ���֣���һ��query��ǰһ��λ�ã�  
                highlighted_chunk += text[prev + len(prev_str):cur] + hl_start + actual_term + hl_end  
                prev = cur  
                prev_str = cur_str  
  
                # Keep track of how far we've copied so far, for the last step  
                matched_so_far = cur + len(actual_term)  
  
        # Don't forget the chunk after the last term  
        #�������һ��ƥ���query����Ĳ���  
        highlighted_chunk += text[matched_so_far:]  
  
        #�����Ҫ��ͷnot start_head�żӵ�  
        if start_offset > 0 and not self.start_head:  
            highlighted_chunk = '...%s' % highlighted_chunk  
  
        if end_offset < len(self.text_block):  
            highlighted_chunk = '%s...' % highlighted_chunk  
  
        #�ɼ���ĿǰΪֹ��������start_offsetǰ��ģ�����һ��ƥ���ǰ��Ĳ��֣�text_block[:start_offset]��������չʾ(��start_headΪTrueʱ)�����  
        if self.start_head:  
            highlighted_chunk = self.text_block[:start_offset] + highlighted_chunk  
        return highlighted_chunk  


�������2���ļ�֮�󣬱����ʹ���Լ��ı�ǩ {% mylighlight %}�ˣ�ʹ��ʱ�ǵ�LoadŶ��

Syntax��

[python] view plain copy print?
{% myhighlight <text_block> with <query> [css_class "class_name"] [html_tag "span"] [max_length 200] [start_head True] %}  

�ɼ���ֻ�Ƕ������һ��ѡ�� start_head ��Ĭ��ΪFalse���������ΪTrue �򲻻�ʡ�ԡ�