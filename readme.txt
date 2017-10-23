django-haystack全文检索详细教程
标签： DjangohaystackpythonWeb前端
2016-10-20 21:28 3703人阅读 评论(5) 收藏 举报
 分类： web前端（4）   django（1）  
版权声明：本文为博主原创文章，未经博主允许不得转载。
目录(?)[+]
前几天要用Django-haystack来实现搜索功能，网上一搜中文资源少之又少，虽说有官方文档，但相信对于我们这些英语差的同学来说要看懂真的是一件难事。特别是关于高级部分，特地找了个英语专业的来翻译，也没能看出个名堂来，专业性实在是太强了，导致完全看不懂。。。
但是，对于一些小站点的开发来说，下面我要给大家讲的完全足够用了，只不过有时候确实麻烦点。好了，言归正传。为了节约时间，简单设置部分从网上找了一篇博客，但是这边文章没有解释配置的作用，而我会给你们详细解释并拓展。
转载部分来自：点击打开链接
一：使用的工具
haystack是django的开源搜索框架，该框架支持Solr,Elasticsearch,Whoosh, *Xapian*搜索引擎，不用更改代码，直接切换引擎，减少代码量。
搜索引擎使用Whoosh，这是一个由纯Python实现的全文搜索引擎，没有二进制文件等，比较小巧，配置比较简单，当然性能自然略低。
中文分词Jieba，由于Whoosh自带的是英文分词，对中文的分词支持不是太好，故用jieba替换whoosh的分词组件。
其他：Python 2.7 or 3.4.4, Django 1.8.3或者以上，Debian 4.2.6_3
二：配置说明
现在假设我们的项目叫做Project,有一个myapp的app，简略的目录结构如下。
Project
Project
settings.py
blog
models.py
此models.py的内容假设如下：
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
1. 首先安装各工具
pip install whoosh django-haystack jieba
2. 添加 Haystack 到Django的 INSTALLED_APPS
配置Django项目的settings.py里面的INSTALLED_APPS添加Haystack,例子：

[python] view plain copy print?
INSTALLED_APPS = [   
        'django.contrib.admin',  
        'django.contrib.auth',   
        'django.contrib.contenttypes',   
        'django.contrib.sessions',   
        'django.contrib.sites',   
  
          # Added. haystack先添加，  
          'haystack',   
          # Then your usual apps... 自己的app要写在haystakc后面  
          'blog',  
]  

3. 修改 你的 settings.py，以配置引擎
本教程使用的是Whoosh，故配置如下：

[python] view plain copy print?
import os  
HAYSTACK_CONNECTIONS = {  
    'default': {  
        'ENGINE': 'haystack.backends.whoosh_backend.WhooshEngine',  
        'PATH': os.path.join(os.path.dirname(__file__), 'whoosh_index'),  
    },  
}  
其中顾名思义，ENGINE为使用的引擎必须要有，如果引擎是Whoosh，则PATH必须要填写，其为Whoosh 索引文件的存放文件夹。
其他引擎的配置见官方文档
4.创建索引
如果你想针对某个app例如mainapp做全文检索，则必须在mainapp的目录下面建立search_indexes.py文件，文件名不能修改。内容如下：


[python] view plain copy print?
import datetime  
from haystack import indexes  
from myapp.models import Note  
  
class NoteIndex(indexes.SearchIndex, indexes.Indexable):     #类名必须为需要检索的Model_name+Index，这里需要检索Note，所以创建NoteIndex  
    text = indexes.CharField(document=True, use_template=True)  #创建一个text字段  
  
    author = indexes.CharField(model_attr='user')   #创建一个author字段  
  
    pub_date = indexes.DateTimeField(model_attr='pub_date')  #创建一个pub_date字段  
  
    def get_model(self):          #重载get_model方法，必须要有！  
        return Note  
  
    def index_queryset(self, using=None):   #重载index_..函数  
        """Used when the entire index for model is updated."""  
        return self.get_model().objects.filter(pub_date__lte=datetime.datetime.now())  

为什么要创建索引？索引就像是一本书的目录，可以为读者提供更快速的导航与查找。在这里也是同样的道理，当数据量非常大的时候，若要从这些数据里找出所有的满足搜索条件的几乎是不太可能的，将会给服务器带来极大的负担。所以我们需要为指定的数据添加一个索引（目录），在这里是为Note创建一个索引，索引的实现细节是我们不需要关心的，至于为它的哪些字段创建索引，怎么指定，正是我要给大家讲的，也是网上所不曾提到的。

每个索引里面必须有且只能有一个字段为 document=True，这代表haystack 和搜索引擎将使用此字段的内容作为索引进行检索(primary field)。其他的字段只是附属的属性，方便调用，并不作为检索数据。直到我自己完成一个搜索器，也没有用到这些附属属性，所以我索性就都删掉了，大家学习的时候也可以先注释掉不管。具体作用我也不明白，反正我没用上。
注意：如果使用一个字段设置了document=True，则一般约定此字段名为text，这是在SearchIndex类里面一贯的命名，以防止后台混乱，当然名字你也可以随便改，不过不建议改。
并且，haystack提供了use_template=True在text字段，这样就允许我们使用数据模板去建立搜索引擎索引的文件，说得通俗点就是索引里面需要存放一些什么东西，例如 Note 的 title 字段，这样我们可以通过 title 内容来检索 Note 数据了，举个例子，假如你搜索 python ，那么就可以检索出含有title含有 python 的Note了，怎么样是不是很简单？数据模板的路径为templates/search/indexes/yourapp/note_text.txt（推荐在项目根目录创建一个templates，并在settings.py里为其引入，使得django会从这个templates里寻找模板，当然，只要放在任何一个你的Django能搜索到的tempaltes下面就好，关于这点我想不属于我们讨论的范畴），templates/search/indexes/blog/note_text.txt文件名必须为要索引的类名_text.txt,其内容为

[python] view plain copy print?
{{ object.title }}  
{{ object.user.get_full_name }}  
{{ object.body }}  

这个数据模板的作用是对Note.title, Note.user.get_full_name,Note.body这三个字段建立索引，当检索的时候会对这三个字段做全文检索匹配。上面已经解释清楚了。

5.在URL配置中添加SearchView，并配置模板
在urls.py中配置如下url信息，当然url路由可以随意写。

[python] view plain copy print?
(r'^search/', include('haystack.urls')),  

其实haystack.urls的内容为

[python] view plain copy print?
from django.conf.urls import url  
from haystack.views import SearchView  
  
urlpatterns = [  
    url(r'^$', SearchView(), name='haystack_search'),  
]  

SearchView()视图函数默认使用的html模板路径为templates/search/search.html（再说一次推荐在根目录创建templates，并在settings.py里设置好）
所以需要在templates/search/下添加search.html文件，内容为

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

很明显，它自带了分页。
然后为大家解释一下这个文件。首先可以看到模板里使用了的变量有 form,query,page 。下面一个个的说一下。

form，很明显，它和django里的form类是差不多的，可以渲染出一个搜索的表单，相信用过Django的Form都知道，所以也不多说了，不明白的可以去看Django文档，当然其实我倒最后也没用上，最后是自己写了个<form></form>，提供正确的参数如name="seach"，method="get"以及你的action地址就OK了。。。如果需要用到更多的搜索功能如过滤的话可能就要自定义Form类了（而且通过上面的例子可以看到，默认的form也是提供一个简单的过滤器的，可以供你选择哪些model是需要检索的，如果一个都不勾的话默认全部搜索，当然我们也是可以自己利用html来模拟这个form的，所以想要实现model过滤还是很简单的，只要模拟一下这个Form的内容就好了），只有这样haystack才能够构造出相应的Form对象来进行检索，其实和django的Form是一样的，Form有一个自我检查数据是否合法的功能，haystack也一样，关于这个此篇文章不做多说，因为我也不太明白（2333）。具体细节去看文档，而且文档上关于View&Form那一节还是比较通俗易懂的，词汇量要求也不是很高，反正就连我都看懂了一些。。。

query嘛，就是我们搜索的字符串。

关于page，可以看到page有object_list属性，它是一个list，里面包含了第一页所要展示的model对象集合，那么list里面到底有多少个呢？我们想要自己控制个数怎么办呢？不用担心，haystack为我们提供了一个接口。我们只要在settings.py里设置：


[python] view plain copy print?
#设置每页显示的数目，默认为20，可以自己修改  
HAYSTACK_SEARCH_RESULTS_PER_PAGE  =  8  

然后关于分页的部分，大家看名字应该也能看懂吧。
如果想要知道更多的默认context带的变量，可以自己看看源码views.py里的SearchView类视图，相信都能看懂。

那么问题来了。对于一个search页面来说，我们肯定会需要用到更多自定义的 context 内容，那么这下该怎么办呢？最初我想到的办法便是修改haystack源码，为其添加上更多的 context 内容，你们是不是也有过和我一样的想法呢？但是这样做即笨拙又愚蠢，我们不仅需要注意各种环境，依赖关系，而且当服务器主机发生变化时，难道我们还要把 haystack 也复制过去不成？这样太愚蠢了！突然，我想到既然我不能修改源码，难道我还不能复用源码吗？之后，我用看了一下官方文档，正如我所想的，通过继承SeachView来实现重载 context 的内容。官方文档提供了2个版本的SearchView，我最开始用的是新版的，最后出错了，也懒得去找错误是什么引起的了，直接使用的了旧版本的SearchView，只要你下了haystack，2个版本都是给你安装好了的。于是我们在myapp目录下再创建一个search_views.py 文件，位置名字可以自己定，用于写自己的搜索视图，代码实例如下：


[python] view plain copy print?
from haystack.views import SearchView  
from .models import *  
  
class MySeachView(SearchView):  
    def extra_context(self):       #重载extra_context来添加额外的context内容  
        context = super(MySeachView,self).extra_context()  
        side_list = Topic.objects.filter(kind='major').order_by('add_date')[:8]  
        context['side_list'] = side_list  
        return context  


然后再修改urls.py将search请求映射到MySearchView：

[python] view plain copy print?
url(r'^search/', search_views.MySeachView(), name='haystack_search'),  

讲完了上下文变量，再让我们来讲一下模板标签，haystack为我们提供了 {% highlight %}和 {% more_like_this %} 2个标签，这里我只为大家详细讲解下 highlight的使用。
你是否也想让自己的检索和百度搜索一样，将匹配到的文字也高亮显示呢？ {% highlight %} 为我们提供了这个功能（当然不仅是这个标签，貌似还有一个HighLight类，这个自己看文档去吧，我英语差，看不明白）。

Syntax：

[python] view plain copy print?
{% highlight <text_block> with <query> [css_class "class_name"] [html_tag "span"] [max_length 200] %}  
大概意思是为 text_block 里的 query 部分添加css_class，html_tag，而max_length 为最终返回长度，相当于 cut ，我看了一下此标签实现源码，默认的html_tag 值为 span ，css_class 值为 highlighted，max_length 值为 200，然后就可以通过CSS来添加效果。如默认时：


[css] view plain copy print?
span.highlighted {  
        color: red;  
}  


Example：


[python] view plain copy print?
# 使用默认值  
{% highlight result.summary with query %}  
  
# 这里我们为 {{ result.summary }}里所有的 {{ query }} 指定了一个<div></div>标签，并且将class设置为highlight_me_please，这样就可以自己通过CSS为{{ query }}添加高亮效果了，怎么样，是不是很科学呢  
{% highlight result.summary with query html_tag "div" css_class "highlight_me_please" %}  
  
# 这里可以限制最终{{ result.summary }}被高亮处理后的长度  
{% highlight result.summary with query max_length 40 %}  


好了，到目前为止，如果你掌握了上面的知识的话，你已经会制作一个比较令人满意的搜索器了，接下来就是创建index文件了。

6.最后一步，重建索引文件
使用python manage.py rebuild_index或者使用update_index命令。

好，下面运行项目，进入该url搜索一下试试吧。

每次数据库更新后都需要更新索引，所以haystack为大家提供了一个接口，只要在settings.py里设置：

[python] view plain copy print?
#自动更新索引  
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'  


三：下面要做的，使用jieba分词

1 将文件whoosh_backend.py（该文件路径为python路径/lib/python2.7.5/site-packages/haystack/backends/whoosh_backend.py）拷贝到app下面，并重命名为whoosh_cn_backend.py，例如blog/whoosh_cn_backend.py。

修改为如下

[python] view plain copy print?
from jieba.analyse import ChineseAnalyzer   #在顶部添加  
  
schema_fields[field_class.index_fieldname] = TEXT(stored=True, analyzer=ChineseAnalyzer(),field_boost=field_class.boost, sortable=True)   #注意先找到这个再修改，而不是直接添加  

2 在settings.py中修改引擎，如下

[python] view plain copy print?
import os  
HAYSTACK_CONNECTIONS = {  
    'default': {  
        'ENGINE': 'blog.whoosh_cn_backend.WhooshEngine',      #blog.whoosh_cn_backend便是你刚刚添加的文件  
        'PATH': os.path.join(BASE_DIR, 'whoosh_index'  
    },  
}  

3 重建索引，在进行搜索中文试试吧。



四.highlight补充
终于写完了！！！泪奔！！！最后给大家看看我的

怎么样，还行吧？眼尖的人会发现，为什么标题里的高等没有被替换成...，而段落里的数学之前的内容却被替换成了...，标题本来就很短，想象一下，若是高等数学被显示成了数学，是不是丢失了最重要的信息呢？高等这么重要的字眼都被省略了，很显然是不行的，毕竟我是个高等生。那么怎么办呢？我没有选择去看文档，可能文档的HighLight类就是用来干这个的吧，但是我选择了读highlight 标签的源码，最终还是让我实现了。

我们需要做的是复制粘贴源码，然后进行修改，而不是选择直接改源码，创建一个自己的标签。为大家奉上。添加myapp/templatetags/my_filters_and_tags.py 文件和 myapp/templatetags/highlighting.py 文件，内容如下（源码分别位于haystack/templatetags/lighlight.py 和 haystack/utils/lighlighting.py 中）：

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
    #默认值  
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
        #text_block为内容，start_offset,end_offset分别为第一个匹配query开始和按长度截断位置  
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
        #matched_so_far最终求得为text中最后一个匹配query的结尾  
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
  
                #分别添上每个query+其后面的一部分（下一个query的前一个位置）  
                highlighted_chunk += text[prev + len(prev_str):cur] + hl_start + actual_term + hl_end  
                prev = cur  
                prev_str = cur_str  
  
                # Keep track of how far we've copied so far, for the last step  
                matched_so_far = cur + len(actual_term)  
  
        # Don't forget the chunk after the last term  
        #加上最后一个匹配的query后面的部分  
        highlighted_chunk += text[matched_so_far:]  
  
        #如果不要开头not start_head才加点  
        if start_offset > 0 and not self.start_head:  
            highlighted_chunk = '...%s' % highlighted_chunk  
  
        if end_offset < len(self.text_block):  
            highlighted_chunk = '%s...' % highlighted_chunk  
  
        #可见到目前为止还不包含start_offset前面的，即第一个匹配的前面的部分（text_block[:start_offset]），如需展示(当start_head为True时)便加上  
        if self.start_head:  
            highlighted_chunk = self.text_block[:start_offset] + highlighted_chunk  
        return highlighted_chunk  


添加上这2个文件之后，便可以使用自己的标签 {% mylighlight %}了，使用时记得Load哦！

Syntax：

[python] view plain copy print?
{% myhighlight <text_block> with <query> [css_class "class_name"] [html_tag "span"] [max_length 200] [start_head True] %}  

可见我只是多添加了一个选项 start_head ，默认为False，如果设置为True 则不会省略。