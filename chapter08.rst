======================================
Capítulo 8: Views e URLconfs avançados
======================================

.. SL Note: as the code examples in this chapter are expository, and
.. SL are not complete runnable examples, I'm proof-reading by eye
.. SL and running an automated syntax check on them to test them.

No `Chapter 3`_ foi explicado o básico sobre funções views e URLconfs.
Este capítulo explicaá mais a fundo funcionalidades avançadas nessas duas
partes do framework.

.. _Chapter 3: ../chapter03/

Truques do URLconf
==================

Não há nada "especial" sobre os URLconfs. Como tudo em Django são somente
código Python. Você pode tomar proveito disso de vários modos, como
descrito nas seções seguintes.

Racionalizando importações de funções
-------------------------------------

Considere este URLconf, baseado em um exemplo do Capítulo 3::

    from django.conf.urls.defaults import *
    from mysite.views import hello, current_datetime, hours_ahead

    urlpatterns = patterns('',
        (r'^hello/$', hello),
        (r'^time/$', current_datetime),
        (r'^time/plus/(\d{1,2})/$', hours_ahead),
    )

Como explicado no capítulo 3, cada entrada no URLconf contém a função view
associada, que é passada diretamenet como um objeto de função. Isso
significa que é necessário importar as funções view no topo do módulo.

Mas conforme a aplicação Django cresce em complexidade seu URLconf também
cresce. Isso faz com que as importações se tornem tediosas para se gerenciar.
(Para cada nova função view é preciso lembrar de importá-la, e a instrução
de importação acaba ficando grande demais utilizando essa abordagem).
É possível evitar esse tédio importando o próprio módulo ``views``, o exemplo
a seguir é equivalente ao anterior:

.. parsed-literal::

    from django.conf.urls.defaults import *
    **from mysite import views**

    urlpatterns = patterns('',
        (r'^hello/$', **views.hello**),
        (r'^time/$', **views.current_datetime**),
        (r'^time/plus/(\d{1,2})/$', **views.hours_ahead**),
    )

Django oferece outra maneira de especificar a função view para um padrão
particular no URLconf: você pode passar a string contendo o nome do módulo
e da função ao invés do próprio objeto. Continuando o exemplo em andamento:

.. parsed-literal::

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^hello/$', **'mysite.views.hello'**),
        (r'^time/$', **'mysite.views.current_datetime'**),
        (r'^time/plus/(\d{1,2})/$', **'mysite.views.hours_ahead'**),
    )

(Preste atenção nas aspas em volta dos nomes das views. Estamos usando
``'mysite.views.current_datetime'`` -- com aspas -- ao invés de
``mysite.views.current_datetime``.)

Usando esta técnica não é mais necessário importar as funções view. O Django
automaticamente importa a view correta da primeira vez que for necessário,
de acordo com a string que descreve o nome e o caminho da função view.

Um outro atalho que é possível utilizando a técnica da string é fatorar um
"prefixo de view" comum. No URLconf de exemplo, cada string de view começa
com ``'mysite.views'``, que não precisa ser digitado para cada linha.
Podemos fatorar o prefixo comum e passá-lo como primeiro arumento para
``patterns()``, assim:

.. parsed-literal::

    from django.conf.urls.defaults import *

    urlpatterns = patterns(**'mysite.views'**,
        (r'^hello/$', **'hello'**),
        (r'^time/$', **'current_datetime'**),
        (r'^time/plus/(\d{1,2})/$', **'hours_ahead'**),
    )

Note que não é posto um ponto (``"."``) no fim do prefixo, nem no começo da
string da view. O Django coloca os pontos automaticamente.

Com essas duas abordagens em mente, qual é a melhor? Isso depende do seu
estilo e necessidades de programação.

As vantagens da abordagem de string são:


* É mais compacta, pois não exige a importação das funções view.

* Resulta em URLconfs mais legíveis e melhor configuráveis se suas
  funções views estão espalhadas em diferentes módulos Python.

As vantages da abordagem de objeto de função são:

* It allows for easy "wrapping" of view functions. See the section "Wrapping View
  Functions" later in this chapter.

* É mais "Pythonica", ou seja, está alinhada com as tradições do Python,
  como passar funções como objetos.

Ambas abordagens são válidas, sendo possível até combiná-las em um mesmo
URLconf. A escolha é sua.

Usando múltiplos prefixos de views
----------------------------------

Na prática, se você usa a técnica de string, acabará misturando views até
o ponto que as views no URLconf não possuam um prefixo em comum. Entretanto,
é possível tomar proveito do atalho de prefixo de views para remover
repetições. Para isso é só adicionar múltiplos objetos ``patterns()``,
assim:

Antigo::

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^hello/$', 'mysite.views.hello'),
        (r'^time/$', 'mysite.views.current_datetime'),
        (r'^time/plus/(\d{1,2})/$', 'mysite.views.hours_ahead'),
        (r'^tag/(\w+)/$', 'weblog.views.tag'),
    )

Novo::

    from django.conf.urls.defaults import *

    urlpatterns = patterns('mysite.views',
        (r'^hello/$', 'hello'),
        (r'^time/$', 'current_datetime'),
        (r'^time/plus/(\d{1,2})/$', 'hours_ahead'),
    )

    urlpatterns += patterns('weblog.views',
        (r'^tag/(\w+)/$', 'tag'),
    )

Tudo o que o framework se importa é que haja uma variável a nível de módulo
chamada ``urlpatterns``. Essa variável pode ser construída dinamicamente,
como é feito nesse exemplo. Devemos ressaltar que objetos retornados pelo
``patterns()`` podem ser somados, pois é algo não intuitivo.

URLs acessáveis somente no modo Debug
-------------------------------------

Falando sobre construir ``urlpatterns`` dinamicamente, pode ser de sua
vontade tomar proveito dessa técnica para alterar o comportamente de seu
URLconf enquanto o Django está no modo Debug. Para fazer isso só mude o valor
de ``DEBUG`` tem tempo de execução, como a seguir::

    from django.conf import settings
    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^$', views.homepage),
        (r'^(\d{4})/([a-z]{3})/$', views.archive_month),
    )

    if settings.DEBUG:
        urlpatterns += patterns('',
            (r'^debuginfo/$', views.debug),
        )

Neste exemplo, a URL ``/debuginfo`` só estará disponível se sua
configuração ``DEBUG`` estiver como ``True``.

Usando grupos nomeados
----------------------

Em todos os exemplos de URLconf, foram usadas simples, *não nomeados* grupos
de expressão regular, ou seja, ao por parêntesis em volta de partes da URL que
queremos capturar, o Django passa o texto capturado para a função view como um
argumento posicional. Em um uso mais avançado é possível usar grupos de
expressão regular *nomeados* para captuar partes de URL e passá-los como
argumentos de *keyword* para a view.

.. admonition:: Argumentos de Keyword vs. Argumentos de posição

    Uma função Python pode ser chamada utilizando argumentos de keyword
    ou de posição e, em alguns casos, ambos ao mesmo tempo. Em uma
    chamada de argumento por keyword, são especificados os nomes dos
    argumentos em conjunto com os valores que estão sendo passados. Em uma
    chamada de argumento por posição só são passados os argumentos sem
    espeficicar explicitamente quais argumentos vão com quais valores. A
    associação fica implicita na ordem dos argumentos.

    Por exemplo, considere essa função simples::

        def sell(item, price, quantity):
            print "Selling %s unit(s) of %s at %s" % (quantity, item, price)

    Para chamá-la com argumentos posicionais, os argumentos são especificados
    na ordem em que são listados na definição da função::

        sell('Socks', '$2.50', 6)

    Para chamá-lo com argumentos kewwor, é especificado os nomes dos argumentos
    em conjunto com os valores. As seguintes instruções são equivalentes::

        sell(item='Socks', price='$2.50', quantity=6)
        sell(item='Socks', quantity=6, price='$2.50')
        sell(price='$2.50', item='Socks', quantity=6)
        sell(price='$2.50', quantity=6, item='Socks')
        sell(quantity=6, item='Socks', price='$2.50')
        sell(quantity=6, price='$2.50', item='Socks')

    Por fim, é possível misturar argumentos posicionais e kewyword, contanto
    que todos os argumentos posicionais apareçam antes dos argumentos keyword.
    As seguintes instruções são equivalentes aos exemplos anteriores::

        sell('Socks', '$2.50', quantity=6)
        sell('Socks', price='$2.50', quantity=6)
        sell('Socks', quantity=6, price='$2.50')

A sintaxe para grupos de expressões regulares, nas expressões regulares do
Python, é ``(?P<name>pattern)``, aonde ``name`` é o nome do grupo e
``pattern`` o padrão para casar.

Segue um exemplo de URLconf que usa grupos não-nomeados::

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^articles/(\d{4})/$', views.year_archive),
        (r'^articles/(\d{4})/(\d{2})/$', views.month_archive),
    )

Aqui está o mesmo URLconf, reescrito para usar grupos nomeados::

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^articles/(?P<year>\d{4})/$', views.year_archive),
        (r'^articles/(?P<year>\d{4})/(?P<month>\d{2})/$', views.month_archive),
    )

Isso faz exatamente o mesmo que o exemplo anterior, com uma pequena diferença:
os valores capturados são passados para as funções view como argumentos keyword
ao invés de argumentos posicionais.

Por exemplo, com grupos não nomeados, uma requisição a ``/articles/2006/03/``
resultaria em uma invocoção de função equivalente a isso::

    month_archive(request, '2006', '03')

Com grupos nomeados a mesma requisição resultaria na seguinte invocação de
função::

    month_archive(request, year='2006', month='03')

Na prática, usar grupos nomeados faz com que seus URLconfs sejam mais
explícitos e menos inclinados a bugs relacionados a ordem dos argumentos,
além de ser possível reordenar os arumentos nas definições das funções view.
Seguindo o exemplo anterior, se quiséssemos mudar as URLs para incluir o mês
*antes* do ano, e estivéssemos usando grupos não-nomeados, seria necessário
mudar a ordem dos argumentos da view ``month_archive``. Já, se fossem usados
grupos nomeados, mudar a ordem dos parametros capturados pela view na URL
não teria efeito na view.

É claro que os benefícios de grupos nomeados vem ao preço da brevidade,
alguns desenvolvedores acham a sintaxe de grupos nomeados feias e verbosa
demais. Ainda, outra vantagem de grupos nomeados é a legibilidade,
especialmente para aqueles que não são muito íntimos com expressões
regulares ou sua aplicação Django em específico. É mais fácil ver o que
está acontecendo, com só uma olhada, em um URLconf que usa grupos nomeados.

Entendendo o algoritmo de Casar/Agrupar
---------------------------------------

Um cuidado ao usar grupos nomeados em um URLconf é que um único padrão de
URLconf não pode conter ambos grupos nomeados e não-nomeados. Se isso for
feito, o Django não lançará erros, mas é provável que algumas URLs não
estão sendo casadas como esperado. Especificamente, há um algoritmo que o
analisador de URLconf segue, respeitando grupos nomeados vs. não-nomeados
em uma expressão regular:

* Se há argumentos nomeados, esses serão usados, ignorando argumentos
  não-nomeados.

* Caso contrário, passará todos os não-nomeados como argumentos de
  posição.

* Em ambos os casos serão passadas quaisquer opções extras como argumentos
  keyword. Veja a próxima seção para mais informações.

Passando Opções Extras Para as Funções View
-------------------------------------------

As vezes você perceberá que está escrevendo funções que são muito parecidas,
contendo apenas algumas pequenas diferenças. Por exemplo, digamos que há duas
views cujo conteúdo são iguais, com exceção do template que usam::

    # urls.py

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^foo/$', views.foo_view),
        (r'^bar/$', views.bar_view),
    )

    # views.py

    from django.shortcuts import render
    from mysite.models import MyModel

    def foo_view(request):
        m_list = MyModel.objects.filter(is_new=True)
        return render(request, 'template1.html', {'m_list': m_list})

    def bar_view(request):
        m_list = MyModel.objects.filter(is_new=True)
        return render(request, 'template2.html', {'m_list': m_list})

O código anterior contém alumas repetições e isso é deselegante. Primeiro
é preciso remover a redundância usando a mesma view para ambas as URLs. Isso
pode ser feito colocando parênteses em volta da URL para capturá-la e checar
a URL dentro da view para decidir o template, como mostrado a seguir::

    # urls.py

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^(foo)/$', views.foobar_view),
        (r'^(bar)/$', views.foobar_view),
    )

    # views.py

    from django.shortcuts import render
    from mysite.models import MyModel

    def foobar_view(request, url):
        m_list = MyModel.objects.filter(is_new=True)
        if url == 'foo':
            template_name = 'template1.html'
        elif url == 'bar':
            template_name = 'template2.html'
        return render(request, template_name, {'m_list': m_list})

O problema com essa solução é que une suas URLs ao seu código. Caso a URL
``/foo/`` mude para ``/fooey/``, será necessário lembrar de mudar o código
da view.

A solução elegante envolve um parametro opcional do URLconf. Cada padrão em
um URLconf pode receber um terceiro item: um dicionário de argumentos nomeados
a serem passados para a função view.

Com isso em mente, podemos reescrever o exemplo atual desse modo::

    # urls.py

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^foo/$', views.foobar_view, {'template_name': 'template1.html'}),
        (r'^bar/$', views.foobar_view, {'template_name': 'template2.html'}),
    )

    # views.py

    from django.shortcuts import render
    from mysite.models import MyModel

    def foobar_view(request, template_name):
        m_list = MyModel.objects.filter(is_new=True)
        return render(request, template_name, {'m_list': m_list})

Como demonstrado acima, o URLconf no exemplo especifica o ``template_name`` no
URLconf. A função view o trata simplesmente como outro parametro.

Essa técnica extra das opções do URLconf é uma boa maneira de enviar
informações adicionais para suas funções views com pouca agitação. Como tal,
é utilizada por algumas das aplicações do Django, mais notavelmente seu
sistema de views genéricas, cobertas no `Chapter 11`_.

As próximas seções mostram algumas ideias de como usar a técnica de opções 
extra do URLconf em seus projetos.

Falsificando valores URLconf capturados
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Consideramos que há um conjunto de views que casam com um padrão, juntamente
com outra URL que não se encaixa no padrão, mas a lógica da view é a mesma.
Nesse caso, é possível "falsificar" os valores capturados pela URL usando
as opções extra do URLconf para lidar com a URL extra da mesma view.

Por exemplo, pode-se ter uma aplicação que exibe dados para um dia em
particular, com URLs como essas::

    /mydata/jan/01/
    /mydata/jan/02/
    /mydata/jan/03/
    # ...
    /mydata/dec/30/
    /mydata/dec/31/

Este exemplo é simples o bastante -- é possível capturá-los em um URLconf
dessa maneira (usando a sintaxe de grupos nomeados)::

    urlpatterns = patterns('',
        (r'^mydata/(?P<month>\w{3})/(?P<day>\d\d)/$', views.my_view),
    )

E o cabeçalho da função view ficaria assim::

    def my_view(request, month, day):
        # ....

Essa abordagem é direta -- não é nada de novo. O artifício aparece quando
é preciso adicionar outra URL que usa ``my_view`` mas que a URL não inclui
um ``month`` e/ou ``day``.

Por exemplo, pode ser necessário adicionar outra URL, ``/mydata/birthday/``,
que seria equivalente a ``/mydata/jan/06/``. Para esse caso é possível utilizar
das opções extras do URLconf::

    urlpatterns = patterns('',
        (r'^mydata/birthday/$', views.my_view, {'month': 'jan', 'day': '06'}),
        (r'^mydata/(?P<month>\w{3})/(?P<day>\d\d)/$', views.my_view),
    )

A parte legal daqui é que não foi preciso mudar a função view. A função só é
encarregada de *receber* os parametros ``month`` e ``day``, não importa se
eles são capturados da URL ou vem dos parametros extra.

Criando uma Generic View
~~~~~~~~~~~~~~~~~~~~~~~~

É uma boa prática de programação fatorar repetições no código. Por
exemplo, temos essas duas funções Python::

    def say_hello(person_name):
        print 'Hello, %s' % person_name

    def say_goodbye(person_name):
        print 'Goodbye, %s' % person_name

podemos fatorar o código tornando o cumprimento um parâmetro::

    def greet(person_name, greeting):
        print '%s, %s' % (greeting, person_name)

Essa mesma filosofia poed ser aplicada às views Django ao se utilizar os
parametros extra do URLconf.

Tendo isso em mente, é possível começar a fazer abstrações de alto nível
das views. Ao invés de pensar "Esta view exibe uma lista de objetos ``Event``"
e "aquela view exibe uma lista de objetos ``BlogEntry``", perceba que ambos
são casos específicos de "Uma view que exibe uma listra de objetos, onde
o tipo do objeto pode variar".

Observe esse código, por exemplo::

    # urls.py

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^events/$', views.event_list),
        (r'^blog/entries/$', views.entry_list),
    )

    # views.py

    from django.shortcuts import render
    from mysite.models import Event, BlogEntry

    def event_list(request):
        obj_list = Event.objects.all()
        return render(request, 'mysite/event_list.html', {'event_list': obj_list})

    def entry_list(request):
        obj_list = BlogEntry.objects.all()
        return render(request, 'mysite/blogentry_list.html', {'entry_list': obj_list})

As duas views fazem essencialmente a mesma coisa: elas mostram uma lista de
objetos. Pode-se fatorar o tipo do objeto que elas estão exibindo::

    # urls.py

    from django.conf.urls.defaults import *
    from mysite import models, views

    urlpatterns = patterns('',
        (r'^events/$', views.object_list, {'model': models.Event}),
        (r'^blog/entries/$', views.object_list, {'model': models.BlogEntry}),
    )

    # views.py

    from django.shortcuts import render

    def object_list(request, model):
        obj_list = model.objects.all()
        template_name = 'mysite/%s_list.html' % model.__name__.lower()
        return render(request, template_name, {'object_list': obj_list})

Com todas essas pequenas mudanças foi obtida uma view reusável que não
depende do modelo. A partir de agora, toda vez que for necessária uma view
que lista um conjunto de objetos, podemos reutilizar a ``object_list`` em
detrimento de escrever o código da view. A seguir são mostradas algumas
notas sobre o que foi feito aqui.

* Estamos passando as classes dos modelos diretamento como o parametro
  ``model``. O dict de opções extras do URLconf pode passar qualquer
  tipo de objeto Python -- não somente strings.

* A linha ``model.objects.all()`` é um exemplo de *duck typing*: "Se anda
  como um pato e fala como um pato, pode-se tratá-lo como um pato". Note
  que o código não diz qual tipo de objeto ``model`` é. A única exigência
  é que ``model`` tenha um atributo ``objects`` que, por usa vez, tenha
  um método ``all()``.

* É utilizado ``model.__name__.lower()`` para determinar o nome do template.
  Cada class Python possui um atributo ``__name__`` que retorna o nome da
  classe. Essa funcionalidade é útil em horas como essa, quando não sabemos
  o nome da classe até a execução do código. Por exemplo, o ``__name___``
  da classe ``BlogEntry`` é a string ``'BlogEntry'``.

* Em uma pequena diferença entre este exemplo e o anterior, estamos passando
  para o nome variável genérico ``object_list`` para o template. Poderiamos
  facilmente mudar esse nome variável para ``blogentry_list`` ou
  ``event_list``, porém deixamos isso como exercício para o leitor.

Como websites orientados a banco de dados possui diversos padrões comuns,
Django vem com uma série de "views genéricas" que usam essa mesma técnica
para economizar tempo. Views gnéricas são cobertas no Capítulo 11.

Dando a view opções de configuração
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Se a aplicação Django é distribuída, há grande chances dos usuários quererem
algum grau de configuração. Nesse caso é uma boa ideia adicionar ganchos
à suas views para quaisquer opções de configuração que as pessoas possam
querer mudar. É possível utilizar os parametros extra do URLconf para isso.

Uma parte comum de uma aplicação para se fazer configurável é o nome do
template::

    def my_view(request, template_name):
        var = do_something()
        return render(request, template_name, {'var': var})

Entendendo a Precedência de Valor Capturados vs. Opções Extras
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Quando há conflito os parametros extras do URLconfs são precedidos sobre
os parâmetros capturados. Em outras palavras, se o URLconf captura uma
variável de grupo nomeado e um parametro extra de URLconf inclui uma variável
com o mesmo nome, o parâmetro extra do URLconf será utilizado.

Por exemplo, considere o seguinte URLConf::

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^mydata/(?P<id>\d+)/$', views.my_view, {'id': 3}),
    )

Aqui, ambas expressões regluares e o dict extra incluem um ``id``. O ``id``
hard-coded dá precedência. Isso quer dizer que qualquer requisição (por
exemplo ``/mydata/2/`` ou ``/mydata/432432``) será tratado como se o ``id``
fosse ``3``, independente do valor capturado na URL.

Leitores astutos perceberão que nesse caso é uma perda de tempo e digitação
capturar o ``id`` na expressão regular, pois o valor será sempre sobrescrito
pelo valor do dict. Essa afirmação está correta; esse exemplo é trazido apenas
para auxiliar o leitor evitar a cometer esse tipo de erro.

Using Default View Arguments
----------------------------

Another convenient trick is to specify default parameters for a view's
arguments. This tells the view which value to use for a parameter by default if
none is specified.

An example::

    # urls.py

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^blog/$', views.page),
        (r'^blog/page(?P<num>\d+)/$', views.page),
    )

    # views.py

    def page(request, num='1'):
        # Output the appropriate page of blog entries, according to num.
        # ...

Here, both URL patterns point to the same view -- ``views.page`` -- but the
first pattern doesn't capture anything from the URL. If the first pattern
matches, the ``page()`` function will use its default argument for ``num``,
``'1'``. If the second pattern matches, ``page()`` will use whatever ``num``
value was captured by the regular expression.

(Note that we've been careful to set the default argument's value to the
*string* ``'1'``, not the integer ``1``. That's for consistency, because
any captured value for ``num`` will always be a string.)

It's common to use this technique in conjunction with configuration options,
as explained earlier. This example makes a slight improvement to the example in
the "Giving a View Configuration Options" section by providing a default
value for ``template_name``::

    def my_view(request, template_name='mysite/my_view.html'):
        var = do_something()
        return render(request, template_name, {'var': var})

.. SL Again wonder whether default should be unicode?

Special-Casing Views
--------------------

Sometimes you'll have a pattern in your URLconf that handles a large set of
URLs, but you'll need to special-case one of them. In this case, take advantage
of the linear way a URLconf is processed and put the special case first.

For example, you can think of the "add an object" pages in Django's admin site
as represented by a URLpattern like this::

    urlpatterns = patterns('',
        # ...
        ('^([^/]+)/([^/]+)/add/$', views.add_stage),
        # ...
    )

This matches URLs such as ``/myblog/entries/add/`` and ``/auth/groups/add/``.
However, the "add" page for a user object (``/auth/user/add/``) is a special
case -- it doesn't display all of the form fields, it displays two password
fields, and so forth. We *could* solve this problem by special-casing in the
view, like so::

    def add_stage(request, app_label, model_name):
        if app_label == 'auth' and model_name == 'user':
            # do special-case code
        else:
            # do normal code

.. SL It's not strictly relevant to the point you're making here, but
.. SL the view func in the admin is now called 'add_view'

but that's inelegant for a reason we've touched on multiple times in this
chapter: it puts URL logic in the view. As a more elegant solution, we can take
advantage of the fact that URLconfs are processed in order from top to bottom::

    urlpatterns = patterns('',
        # ...
        ('^auth/user/add/$', views.user_add_stage),
        ('^([^/]+)/([^/]+)/add/$', views.add_stage),
        # ...
    )

With this in place, a request to ``/auth/user/add/`` will be handled by the
``user_add_stage`` view. Although that URL matches the second pattern, it
matches the top one first. (This is short-circuit logic.)

Capturing Text in URLs
----------------------

Each captured argument is sent to the view as a plain Python Unicode string,
regardless of what sort of match the regular expression makes. For example, in
this URLconf line::

    (r'^articles/(?P<year>\d{4})/$', views.year_archive),

the ``year`` argument to ``views.year_archive()`` will be a string, not
an integer, even though ``\d{4}`` will only match integer strings.

This is important to keep in mind when you're writing view code. Many built-in
Python functions are fussy (and rightfully so) about accepting only objects of
a certain type. A common error is to attempt to create a ``datetime.date``
object with string values instead of integer values::

    >>> import datetime
    >>> datetime.date('1993', '7', '9')
    Traceback (most recent call last):
        ...
    TypeError: an integer is required
    >>> datetime.date(1993, 7, 9)
    datetime.date(1993, 7, 9)

Translated to a URLconf and view, the error looks like this::

    # urls.py

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        (r'^articles/(\d{4})/(\d{2})/(\d{2})/$', views.day_archive),
    )

    # views.py

    import datetime

    def day_archive(request, year, month, day):
        # The following statement raises a TypeError!
        date = datetime.date(year, month, day)

Instead, ``day_archive()`` can be written correctly like this::

    def day_archive(request, year, month, day):
        date = datetime.date(int(year), int(month), int(day))

Note that ``int()`` itself raises a ``ValueError`` when you pass it a string
that is not composed solely of digits, but we're avoiding that error in this
case because the regular expression in our URLconf has ensured that only
strings containing digits are passed to the view function.

Determining What the URLconf Searches Against
---------------------------------------------

When a request comes in, Django tries to match the URLconf patterns against the
requested URL, as a Python string. This does not include ``GET`` or ``POST``
parameters, or the domain name. It also does not include the leading slash,
because every URL has a leading slash.

For example, in a request to ``http://www.example.com/myapp/``, Django will try
to match ``myapp/``. In a request to ``http://www.example.com/myapp/?page=3``,
Django will try to match ``myapp/``.

The request method (e.g., ``POST``, ``GET``) is *not* taken into account when
traversing the URLconf. In other words, all request methods will be routed to
the same function for the same URL. It's the responsibility of a view function
to perform branching based on request method.

Higher-Level Abstractions of View Functions
-------------------------------------------

And speaking of branching based on request method, let's take a look at how we
might build a nice way of doing that. Consider this URLconf/view layout::

    # urls.py

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        # ...
        (r'^somepage/$', views.some_page),
        # ...
    )

    # views.py

    from django.http import Http404, HttpResponseRedirect
    from django.shortcuts import render

    def some_page(request):
        if request.method == 'POST':
            do_something_for_post()
            return HttpResponseRedirect('/someurl/')
        elif request.method == 'GET':
            do_something_for_get()
            return render(request, 'page.html')
        else:
            raise Http404()

In this example, the ``some_page()`` view's handling of ``POST`` vs. ``GET``
requests is quite different. The only thing they have in common is a shared
URL: ``/somepage/``. As such, it's kind of inelegant to deal with both ``POST``
and ``GET`` in the same view function. It would be nice if we could have two
separate view functions -- one handling ``GET`` requests and the other handling
``POST`` -- and ensuring each one was only called when appropriate.

We can do that by writing a view function that delegates to other views,
either before or after executing some custom logic. Here's an example of how
this technique could help simplify our ``some_page()`` view::

    # views.py

    from django.http import Http404, HttpResponseRedirect
    from django.shortcuts import render

    def method_splitter(request, GET=None, POST=None):
        if request.method == 'GET' and GET is not None:
            return GET(request)
        elif request.method == 'POST' and POST is not None:
            return POST(request)
        raise Http404

    def some_page_get(request):
        assert request.method == 'GET'
        do_something_for_get()
        return render(request, 'page.html')

    def some_page_post(request):
        assert request.method == 'POST'
        do_something_for_post()
        return HttpResponseRedirect('/someurl/')

    # urls.py

    from django.conf.urls.defaults import *
    from mysite import views

    urlpatterns = patterns('',
        # ...
        (r'^somepage/$', views.method_splitter, {'GET': views.some_page_get, 'POST': views.some_page_post}),
        # ...
    )

Let's go through what this does:

* We've written a new view, ``method_splitter()``, that delegates to other
  views based on ``request.method``. It looks for two keyword arguments,
  ``GET`` and ``POST``, which should be *view functions*. If
  ``request.method`` is ``'GET'``, then it calls the ``GET`` view. If
  ``request.method`` is ``'POST'``, then it calls the ``POST`` view. If
  ``request.method`` is something else (``HEAD``, etc.), or if ``GET`` or
  ``POST`` were not supplied to the function, then it raises an
  ``Http404``.

* In the URLconf, we point ``/somepage/`` at ``method_splitter()`` and pass
  it extra arguments -- the view functions to use for ``GET`` and ``POST``,
  respectively.

* Finally, we've split the ``some_page()`` view into two view functions --
  ``some_page_get()`` and ``some_page_post()``. This is much nicer than
  shoving all of that logic into a single view.

  Note that these view functions technically no longer have to check
  ``request.method``, because ``method_splitter()`` does that. (By the time
  ``some_page_post()`` is called, for example, we can be confident
  ``request.method`` is ``'post'``.) Still, just to be safe, and also to
  serve as documentation, we stuck in an ``assert`` that makes sure
  ``request.method`` is what we expect it to be.

Now we have ourselves a nice, generic view function that encapsulates the logic
of delegating a view by ``request.method``. Nothing about ``method_splitter()``
is tied to our specific application, of course, so we can reuse it in other
projects.

But, while we're at it, there's one way to improve on ``method_splitter()``.
As it's written, it assumes that the ``GET`` and ``POST`` views take no
arguments other than ``request``. What if we wanted to use
``method_splitter()`` with views that, for example, capture text from URLs,
or take optional keyword arguments themselves?

To do that, we can use a nice Python feature: variable arguments with
asterisks. We'll show the example first, then explain it::

    def method_splitter(request, *args, **kwargs):
        get_view = kwargs.pop('GET', None)
        post_view = kwargs.pop('POST', None)
        if request.method == 'GET' and get_view is not None:
            return get_view(request, *args, **kwargs)
        elif request.method == 'POST' and post_view is not None:
            return post_view(request, *args, **kwargs)
        raise Http404

Here, we've refactored ``method_splitter()`` to remove the ``GET`` and ``POST``
keyword arguments, in favor of ``*args`` and ``**kwargs`` (note the asterisks).
This is a Python feature that allows a function to accept a dynamic, arbitrary
number of arguments whose names aren't known until runtime. If you put a single
asterisk in front of a parameter in a function definition, any *positional*
arguments to that function will be rolled up into a single tuple. If you put
two asterisks in front of a parameter in a function definition, any *keyword*
arguments to that function will be rolled up into a single dictionary.

For example, with this function::

    def foo(*args, **kwargs):
        print "Positional arguments are:"
        print args
        print "Keyword arguments are:"
        print kwargs

Here's how it would work::

    >>> foo(1, 2, 3)
    Positional arguments are:
    (1, 2, 3)
    Keyword arguments are:
    {}
    >>> foo(1, 2, name='Adrian', framework='Django')
    Positional arguments are:
    (1, 2)
    Keyword arguments are:
    {'framework': 'Django', 'name': 'Adrian'}

.. SL Tested ok

Bringing this back to ``method_splitter()``, you can see we're using ``*args``
and ``**kwargs`` to accept *any* arguments to the function and pass them along
to the appropriate view. But before we do that, we make two calls to
``kwargs.pop()`` to get the ``GET`` and ``POST`` arguments, if they're
available. (We're using ``pop()`` with a default value of ``None`` to avoid
``KeyError`` if one or the other isn't defined.)

Wrapping View Functions
-----------------------

Our final view trick takes advantage of an advanced Python technique. Say you
find yourself repeating a bunch of code throughout various views, as in this
example::

    def my_view1(request):
        if not request.user.is_authenticated():
            return HttpResponseRedirect('/accounts/login/')
        # ...
        return render(request, 'template1.html')

    def my_view2(request):
        if not request.user.is_authenticated():
            return HttpResponseRedirect('/accounts/login/')
        # ...
        return render(request, 'template2.html')

    def my_view3(request):
        if not request.user.is_authenticated():
            return HttpResponseRedirect('/accounts/login/')
        # ...
        return render(request, 'template3.html')

Here, each view starts by checking that ``request.user`` is authenticated
-- that is, the current user has successfully logged into the site -- and
redirects to ``/accounts/login/`` if not. (Note that we haven't yet covered
``request.user`` -- Chapter 14 does -- but, as you might imagine,
``request.user`` represents the current user, either logged-in or anonymous.)

It would be nice if we could remove that bit of repetitive code from each of
these views and just mark them as requiring authentication. We can do that by
making a view wrapper. Take a moment to study this::

    def requires_login(view):
        def new_view(request, *args, **kwargs):
            if not request.user.is_authenticated():
                return HttpResponseRedirect('/accounts/login/')
            return view(request, *args, **kwargs)
        return new_view

This function, ``requires_login``, takes a view function (``view``) and returns
a new view function (``new_view``). The new function, ``new_view`` is defined
*within* ``requires_login`` and handles the logic of checking
``request.user.is_authenticated()`` and delegating to the original view
(``view``).

Now, we can remove the ``if not request.user.is_authenticated()`` checks from
our views and simply wrap them with ``requires_login`` in our URLconf::

    from django.conf.urls.defaults import *
    from mysite.views import requires_login, my_view1, my_view2, my_view3

    urlpatterns = patterns('',
        (r'^view1/$', requires_login(my_view1)),
        (r'^view2/$', requires_login(my_view2)),
        (r'^view3/$', requires_login(my_view3)),
    )

This has the same effect as before, but with less code redundancy. Now we've
created a nice, generic function -- ``requires_login()`` that we can wrap
around any view in order to make it require login.

Including Other URLconfs
========================

If you intend your code to be used on multiple Django-based sites, you should
consider arranging your URLconfs in such a way that allows for "including."

At any point, your URLconf can "include" other URLconf modules. This
essentially "roots" a set of URLs below other ones. For example, this
URLconf includes other URLconfs::

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^weblog/', include('mysite.blog.urls')),
        (r'^photos/', include('mysite.photos.urls')),
        (r'^about/$', 'mysite.views.about'),
    )

(We saw this before in Chapter 6, when we introduced the Django admin site. The
admin site has its own URLconf that you merely ``include()`` within yours.)

There's an important gotcha here: the regular expressions in this example that
point to an ``include()`` do *not* have a ``$`` (end-of-string match character)
but *do* include a trailing slash. Whenever Django encounters ``include()``, it
chops off whatever part of the URL matched up to that point and sends the
remaining string to the included URLconf for further processing.

Continuing this example, here's the URLconf ``mysite.blog.urls``::

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^(\d\d\d\d)/$', 'mysite.blog.views.year_detail'),
        (r'^(\d\d\d\d)/(\d\d)/$', 'mysite.blog.views.month_detail'),
    )

With these two URLconfs, here's how a few sample requests would be handled:

* ``/weblog/2007/``: In the first URLconf, the pattern ``r'^weblog/'``
  matches. Because it is an ``include()``, Django strips all the matching
  text, which is ``'weblog/'`` in this case. The remaining part of the URL
  is ``2007/``, which matches the first line in the ``mysite.blog.urls``
  URLconf.

* ``/weblog//2007/`` (with two slashes): In the first URLconf, the pattern
  ``r'^weblog/'`` matches. Because it is an ``include()``, Django strips
  all the matching text, which is ``'weblog/'`` in this case. The remaining
  part of the URL is ``/2007/`` (with a leading slash), which does not
  match any of the lines in the ``mysite.blog.urls`` URLconf.

* ``/about/``: This matches the view ``mysite.views.about`` in the first
  URLconf, demonstrating that you can mix ``include()`` patterns with
  non-``include()`` patterns.

How Captured Parameters Work with include()
-------------------------------------------

An included URLconf receives any captured parameters from parent URLconfs, for
example::

    # root urls.py

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^(?P<username>\w+)/blog/', include('foo.urls.blog')),
    )

    # foo/urls/blog.py

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^$', 'foo.views.blog_index'),
        (r'^archive/$', 'foo.views.blog_archive'),
    )

In this example, the captured ``username`` variable is passed to the
included URLconf and, hence, to *every* view function within that URLconf.

Note that the captured parameters will *always* be passed to *every* line in
the included URLconf, regardless of whether the line's view actually accepts
those parameters as valid. For this reason, this technique is useful only if
you're certain that every view in the included URLconf accepts the
parameters you're passing.

How Extra URLconf Options Work with include()
---------------------------------------------

Similarly, you can pass extra URLconf options to ``include()``, just as you can
pass extra URLconf options to a normal view -- as a dictionary. When you do
this, *each* line in the included URLconf will be passed the extra options.

For example, the following two URLconf sets are functionally identical.

Set one::

    # urls.py

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^blog/', include('inner'), {'blogid': 3}),
    )

    # inner.py

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^archive/$', 'mysite.views.archive'),
        (r'^about/$', 'mysite.views.about'),
        (r'^rss/$', 'mysite.views.rss'),
    )

Set two::

    # urls.py

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^blog/', include('inner')),
    )

    # inner.py

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        (r'^archive/$', 'mysite.views.archive', {'blogid': 3}),
        (r'^about/$', 'mysite.views.about', {'blogid': 3}),
        (r'^rss/$', 'mysite.views.rss', {'blogid': 3}),
    )

As is the case with captured parameters (explained in the previous section),
extra options will *always* be passed to *every* line in the included
URLconf, regardless of whether the line's view actually accepts those options
as valid. For this reason, this technique is useful only if you're certain that
every view in the included URLconf accepts the extra options you're passing.

What's Next?
============

This chapter has provided many advanced tips and tricks for views and URLconfs.
Next, in `Chapter 9`_, we'll give this advanced treatment to Django's template
system.

.. _Chapter 9: ../chapter09/
