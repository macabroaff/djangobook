==================================
Apêndice C: Generic View Reference
==================================

No capítulo 11 fomos introduzidos ao Genetic View, porém de forma superficial e
sem maiores detalhes. Neste capítulo veremos todos os pontos importantes não
abordados anteriormente, porém é altamente recomendado que você leia o capítulo
11 antes de tentar entender a referência material que se segue. Você pode
querer referir-se ao ``Livro``, ``Editor``, e objetos de ``Autor`` definido
neste capítulo, os exemplos a seguir usam esses modelos.


Argumentos comuns nas Generic Views
===================================

A maioria dessas views possuem uma grande quantidade de argumentos que
possibilitam a modificação do comportamento padrão das generic views. Muitos
desse argumentos fumcionam da mesma forma na maioria das views. A tabela C-1
descreve os argumentos mais comuns; sempre que você encontrar um desse
argumentos, ele funcionará da forma descrita abaixo

.. table:: Tabela C-1. Argumentos comuns nas Generic Views

    ==========================  ===============================================
    Argumentos                  Descrição
    ==========================  ===============================================
    ``allow_empty``             Um booleano que especifica se, para carregar a
                                página, existem ou não objetos disponíveis.
                                Se este for ``False`` e nenhum objeto estiver
                                disponível, a view lançará um 404 ao invés de
                                mostrar uma página vazia. Por padrão, isso é
                                ``True``.

    ``context_processors``      Uma lista adicional de ``template-context
                                processors`` (além dos padrões) que podem ser
                                aplicados nos templates das views. Para maiores
                                informações sobre ``template-context
                                processors``, consulte o capítulo 9.

    ``extra_context``           Possibilita adicionar um dicionário de valores
                                extras no contexto do template. Por padrão, é
                                um dicionário vazio. Se for definido um valor
                                no dicionário, a genetic view irá chamá-lo
                                antes de renderizar o template.

    ``mimetype``                O tipo MIME a ser usado para o documento
                                resultante. Caso nenhum valor seja fornecido,
                                será utilizado o valor de
                                ``DEFAULT_CONTENT_TYPE`` do arquivo de
                                settings.py.

    ``queryset``                Um ``QuerySet`` (exemplo ``Author.objects.all()``)
                                que serve para ler a partir de objetos.
                                Consulte o Apêndice B para mais detalhes sobre
                                ``QuerySet``. As generic views exigem este
                                argumento.

    ``template_loader``         O template loarder é usado ao carregar um
                                template. É usado ``django.template.loader``
                                por padrão. Consulte o capítulo 9 para maiores
                                informações.

    ``template_name``           O nome completo de um template para renderizar uma
                                página. Isso permite que você substitua o nome
                                padrão do template derivado do ``QuerySet``.

    ``template_object_name``    Designa o nome da variável do template para
                                utilizar no contexto do template. Por padrão
                                seu parâmetro é um ``objeto``.
    ==========================  ===============================================

"simples" Generic Views
=======================

O módulo ``django.views.generic.simple`` possui views simples que podem
resolver os casos mais comuns: renderiza um template e, quando existe a
necessidade, executar um redirecionamento.

Renderizando Template
----------------------

*View function*: ``django.views.generic.simple.direct_to_template``

Está view renderiza o template indicado, passando ``{{ params }}`` na variável
do template, criando assim um dicionário contendo os parâmetros obtidos na URL.

Exemplo
```````

Dada a seguinte URLconf::

    from django.conf.urls.defaults import *
    from django.views.generic.simple import direct_to_template

    urlpatterns = patterns('',
        (r'^foo/$',             direct_to_template, {'template': 'foo_index.html'}),
        (r'^foo/(?P<id>\d+)/$', direct_to_template, {'template': 'foo_detail.html'}),
    )

Veremos que a requisição feita para ``/foo/`` renderizará o template
``foo_index.html``, e a requisição feita para ``/foo/15/`` renderizará
``foo_detail.html`` no contexto da variável ``{{ params.id }}`` o valor
``15`` será atribuido.

Argumentos obrigatórios
```````````````````````

* ``template``: Nome completo de um template

Redirecionando para outra URL
------------------------------

*View function*: ``django.views.generic.simple.redirect_to``

Está view redireciona para outra URL. A URL indicada pode conter uma string
formatada como um dictionary-style, que será interpolado com os parâmetros
capturados na URL.

Se a URL dada for ``None``, o Django irá retornar o código HTTP 410 ("Gone").

Exemplo
```````
Esta URLconf redireciona de ``/foo/<id>/`` para ``/bar/<id>/``::

    from django.conf.urls.defaults import *
    from django.views.generic.simple import redirect_to

    urlpatterns = patterns('django.views.generic.simple',
        ('^foo/(?p<id>\d+)/$', redirect_to, {'url': '/bar/%(id)s/'}),
    )

Este exemplo retorna "Gone" como resposta a solicitação enviada a ``/bar/``::

    from django.views.generic.simple import redirect_to

    urlpatterns = patterns('django.views.generic.simple',
        ('^bar/$', redirect_to, {'url': None}),
    )

Argumentos obrigatórios
```````````````````````

* ``url``: URL que redireciona ``de`` para ``para``. Ou ``None``, que irá
retornar 410 "Gone".

Lista/Detalhe Generic Views
===========================

A lista/detalhe da generic views (no módulo ``django.views.generic.list_detail``)
lida com o caso comum de exibição de uma lista de itens em uma view e views
de "detalhe" indivudual desses itens em outra.

Lista de Objetos
----------------

*View function*: ``django.views.generic.list_detail.object_list``

Utilize esta view para exibir uma página que representa uma lista de objetos.

Exemplo
```````

Dado o ``Autor`` objeto do capítulo 5, podemos usar o ``object_list`` view
para mostrar uma lista simples de todos os autores::

    from mysite.books.models import Author
    from django.conf.urls.defaults import *
    from django.views.generic import list_detail

    author_list_info = {
        'queryset':   Author.objects.all(),
    }

    urlpatterns = patterns('',
        (r'authors/$', list_detail.object_list, author_list_info)
    )

Argumentos obrigatórios
```````````````````````

* ``queryset``: A ``QuerySet`` da lista de objetos (veja na Tabela C-1).

Argumentos opcionais
````````````````````

* ``paginate_by``: Um inteiro que especifica quantos objetos devem ser mostrados
  por página
  ``paginate_by`` objetos por página. A view espera por uma página que possua
  uma query string (enviada via ``GET``) com indice zero ou uma página variável
  especificada na URLconf. Veja a seção "Notas de paginação".

Além destes, essa view pode utilizar qualquer um desses argumentos descritos na
descritos na Tabela C-1.

* ``allow_empty``
* ``context_processors``
* ``extra_context``
* ``mimetype``
* ``template_loader``
* ``template_name``
* ``template_object_name``

Template Name
`````````````

Se ``template_name`` não for especificado, a view irá usar o template
``<app_label>/<model_name>_list.html`` por padrão. Tanto o rótulo da aplicação
quanto o nome do modelo são derivados do parâmetro ``queryset``. O rótulo da
aplicação é o nome do aplicativo que o medelo está definito, e o nome do modelo
é a versão minúsculas do nome do modelo de classe.

No exemplo anterior usando `Author.objects.all()`` como uma ``queryset``, o
rótulo da aplicação setia ``livros```e olivros/autor_list.html``.

Template Context
````````````````

Além do ``extra_context``, o template context irá conter o seguinte:

* ``object_list``: Lista de objetos. O nome dessa variável depende do parametro
  ``template_object_name``, que é ``'object'`` por padrão. Se
  ``template_object_name`` é ``foo``, o nome dessa variável será ``foo_list``.

* ``is_paginated``: Um booleano indica se o resultado é paginado.
  Especificamente, é atribuido ``False`` se o número de objetos é menor ou
  igual ao ``paginate_by``.

Se os resultados forem paginados, possuirá essas variáveis adicionais:

* ``results_per_page``: O número de objetos por página (Isso é igual ao
  parâmetro ``paginate_by``)

* ``has_next``: Um booleano é apresentado se houver uma próxima página.

* ``has_previous``: Um booleano é apresentado se houver uma página anterior

* ``page``: Número da página atual, representado por um inteiro.

* ``next``: Número da próxima página, representado por um inteiro. Se não houver
  uma próxima página, este ainda será representado por um inteiro.

* ``previous``: Número da próxima página, representado por um inteiro.

* ``pages``: Número total de páginas, representado por um inteiro

* ``hits``: O número total de objetos em *todas* as páginas, não apenas nesta
  página

.. admonition:: Uma nota sobre a paginação

    Se o ``paginate_by`` for especificado, o Djando irá paginar o resultado.
    VocÊ pode especificar o número da página na URL de duas maneiras:

    * Use o parâmetro ``page`` dentro da URLcong. Sua URLconf ficará parecida
      com isso, por exemplo::

        (r'^objects/page(?P<page>[0-9]+)/$', 'object_list', dict(info_dict))

    * Passe o número da página pelo parâmetro ``page`` query-string. Assim,
      sua URL ficará desta forma::

        /objects/?page=3

    Em ambos os casos, ``page`` será base 1 e não base 0, sendo assim a primeira
    página seria representada como página ``1``.

Detail Views
------------

*View function*: ``django.views.generic.list_detail.object_detail``

Essa view fornece uma visão detalhada de um único objeto

Exemplo
```````

Continuando o exemplo anterior, utilizado em ``object_list``, podemos adicionar
uma visão detalhada modificando a URLconf:

.. parsed-literal::

    from mysite.books.models import Author
    from django.conf.urls.defaults import *
    from django.views.generic import list_detail

    author_list_info = {
        'queryset' :   Author.objects.all(),
    }
    **author_detail_info = {**
        **"queryset" : Author.objects.all(),**
        **"template_object_name" : "author",**
    **}**

    urlpatterns = patterns('',
        (r'authors/$', list_detail.object_list, author_list_info),
        **(r'^authors/(?P<object_id>\d+)/$', list_detail.object_detail, author_detail_info),**
    )

Argumentos obrigatórios
```````````````````````

* ``queryset``: A ``QuerySet`` será pesquisada para o objeto (veja a Tabela C-1)

ou

* ``object_id``: O valor do campo primary-key para o objecto.

ou

* ``slug``: O slug do objeto. Se você informar este campo, o argumento
  ``slug_field`` (veja na seção seguinte) será obrigatório.

Argumentos opcionais
````````````````````

* ``slug_field``: Nome do campo no objeto contendo slug. Será obrigatório se
  você usar o argumento ``slug``. Caso você utilize o argumento ``object_id``
  esse argumento deverá ser ignorado.


* ``template_name_field``: O nome de um campo no objeto cujo valor é
   o usado pelo template name.

  Em outras palavras, se seu objeto possui o campo ``'the_template'`` que
  contenha a string ``'foo.html'``, e você informa ``template_name_field`` para
  ``'the_template'``, a generic view para esse objeto irá usar o template
  ``'foo.html'``.

  Se o template nomeado por ``template_name_field`` não existir, um nome dado
  ao ``template_name`` será usado. Isso é um pouco de ``'brain-bender'``,
  mas é útil em alguns casos.

Essa view também poderá usar esses argumentos (veja a tabela C-1):

* ``context_processors``
* ``extra_context``
* ``mimetype``
* ``template_loader``
* ``template_name``
* ``template_object_name``

Template Name
`````````````

Se o ``template_name`` e ``template_name_field`` não forem especificados,
essa view irá usar o template ``<app_label>/<model_name>_detail.html`` por
padrão.

Template Context
````````````````

Adicionalmente ao ``extra_context``, o template context será como:

* ``object``: O objeto. O nome dessa variável depende do parametro
  ``template_object_name``, que é ``'object'`` por padrão. se o
  ``template_object_name`` for ``'foo'``, o nome dessa variável será ``foo``.

Date-Based Generic Views
========================

Date-based generic views geralmente são usados ​​para fornecer um conjunto de
"arquivo" páginas para material datado. Pense em arquivos de ano/mês/dia para
um jornal, ou uma arquivo do blog típico.

.. admonition:: Tip:

    Por padrão, essa view ignora objetos com datas futuras.

    Isso significa que se você tentar visitar arquivos futuros, o Django irá
    mostrar automáticamente o erro 404 ("Page not found"), mesmo que exista
    uma publicação para esta data.

    Assim, você pode publicar os objetos futuros que não aparecem publicamente
    até sua desejada data de publicação.

    Entretanto, para diferentes tipos de objetos date-based, isso não é
    apropriado (e.g., um calendário de próximos eventos). Para estas views,
    atribua ``True`` a opção ``allow_future``, que irá apresentar os objetos
    futuros ( e permitir que o usuário visite "futuros" arquivos).

Arquivo Index
-------------

*View function*: ``django.views.generic.date_based.archive_index``

Essa view fornece uma página de índice de nível superior mostrando
"mais recente" (ou seja, objetos mais recentes) por data.

Exemplo
```````

Digamos que uma editora de livros quer uma página de livros publicados
recentemente. Dado algum objeto ``Book`` com o campo ``publication_date``,
nós podemos usar a view ``archive_index`` para algumas tarefas comuns:

.. parsed-literal::

    from mysite.books.models import Book
    from django.conf.urls.defaults import *
    from django.views.generic import date_based

    book_info = {
        "queryset"   : Book.objects.all(),
        "date_field" : "publication_date"
    }

    urlpatterns = patterns('',
        (r'^books/$', date_based.archive_index, book_info),
    )

Argumentos obrigatórios
```````````````````````

* ``date_field``: É o nome da ``DateField`` ou ``DateTimeField`` na model
  ``QuerySet`` que o arquivo date-based deve usar para determinar os objetos
  na página.
* ``queryset``: A ``QuerySet`` dos objetos do arquivo.

Argumentos opcionais
````````````````````

* ``allow_future``: Um booleano que especifica se incluem "futuros" objetos
   nessa página, conforme descrito anteriormente.

* ``num_latest``: O número de novos objetos para enviar para ao template
  context. São 15 por padrão.

Essa view também pode ter os seguintes argumentos (veja a tabela C-1):

* ``allow_empty``
* ``context_processors``
* ``extra_context``
* ``mimetype``
* ``template_loader``
* ``template_name``

Template Name
`````````````

Se o ``template_name`` não for especificado, essa view ira usar o template
``<app_label>/<model_name>_archive.html`` por padrão.


Template Context
````````````````

Além do ``extra_context``, o template context será como:

* ``date_list``: Uma lista de objetos ``datetime.date`` representando todos
  os anos que têm objetos disponíveis de acordo com a ``queryset``. Estas são
  ordenados em sentido inverso.

  Por exemplo, se você tiver publicações de um blog de 2003 a 2006, essa lista
  conterá quatro objetos ``datetime.date``: um para cada um dos anos.

* ``latest``: São os objetos ``num_latest`` do sistema, em ordem decrescente
  por ``date_field``. Por exemplo, se ``num_latest`` é ``10``, o ``latest``
  será uma lista com os últimos 10 objetos na ``queryset``.

Arquivos ano
------------

*View function*: ``django.views.generic.date_based.archive_year``

Use esta view para páginas anuais de arquivos. As paginas possuiem uma lista
de meses nos objetos existentes, e eles podem mostar opcionalmente todos os
objetos publicados em um determinado ano.

Exemplo
```````

Aproveitando o exemplo ``archive_index`` utilizado anteriormente, vamos
adicionar uma forma de ver todos os livros publicados em um determinado ano:

.. parsed-literal::

    from mysite.books.models import Book
    from django.conf.urls.defaults import *
    from django.views.generic import date_based

    book_info = {
        "queryset"   : Book.objects.all(),
        "date_field" : "publication_date"
    }

    urlpatterns = patterns('',
        (r'^books/$', date_based.archive_index, book_info),
        **(r'^books/(?P<year>\d{4})/?$', date_based.archive_year, book_info),**
    )

Argumentos obrigatórios
```````````````````````

* ``date_field``: É como ``archive_index`` (veja na seção anterior).

* ``queryset``:  ``QuerySet`` dos objetos para os quais o arquivo serve.

* ``year``: O ano de quatro dígitos para os quais o arquivo serve (como em
  nosso exemplo, esta é geralmente tomada a partir de um parâmetro de URL).

Argumentos opcionais
````````````````````

* ``make_object_list``: Um booleano para especificar a lista de objetos para
  esse ano. Se for ``True``, essa lista de objetos será colocada à disposição
  do  template conforme o ``object_list``. (O nome ``object_list`` pode ser
  diferente; veja mais sobre ``object_list`` na seção "Template Context"). Seu
  valor padrão é ``False``.

* ``allow_future``: Um booleano que especifica se incluem "futuros" objetos
   nesta página.

Esta view também pode tomar os seguintes argumentos (veja a tabela C-1):

* ``allow_empty``
* ``context_processors``
* ``extra_context``
* ``mimetype``
* ``template_loader``
* ``template_name``
* ``template_object_name``

Template Name
`````````````

If ``template_name`` isn't specified, this view will use the template
``<app_label>/<model_name>_archive_year.html`` by default.

Template Context
````````````````

Além do ``extra_context``, contexto do modelo será como se segue:

* ``date_list``: Uma lista de objetos ``datetime.date`` representando todos
  os meses que têm objetos disponíves no ano, de acordo com a ``queryset``,
  ordenada em ordem crescente.

* ``year``: O ano, com uma sequência de quatro caracteres.

* ``object_list``: Se o parâmetro ``make_object_list`` for ``True``, ele irá
  atribuir uma lista de objetos disponíveis no ano em questão, ordenado pelo
  campo data. O nome dessa variável depende do parâmetro
  ``template_object_name``, que é ``'object'`` por padrão. Se o
  ``template_object_name`` for ``'foo'``, o nome da variável será ``foo_list``.

  Se o ``make_object_list`` for ``False``, o ``object_list`` irá passar uma
  para lista vazia para o template.

Arquivos mês
------------

*View function*: ``django.views.generic.date_based.archive_month``

Essa view fornece páginas de arquivos mensais mostrando todos os objetos para
,um determinado mês.

Exemplo
```````

Continuando o nosso exemplo, vamos adicionar na view os meses:

.. parsed-literal::

    urlpatterns = patterns('',
        (r'^books/$', date_based.archive_index, book_info),
        (r'^books/(?P<year>\d{4})/?$', date_based.archive_year, book_info),
        **(**
            **r'^(?P<year>\d{4})/(?P<month>[a-z]{3})/$',**
            **date_based.archive_month,**
            **book_info**
        **),**
    )

Argumentos obrigatórios
```````````````````````

* ``year``: O ano de quatro dígitos para que o arquivo serve (uma string).

* ``month``: O mês em que o arquivo serve, formatado de acordo com
   o argumento ``month_format``.

* ``queryset``: A ``QuerySet`` de objetos para os quais o arquivo serve.

* ``date_field``: É o nome do ``DateField`` ou ``DateTimeField`` na
  ``QuerySet`` que o arquivo date-based deve usar para determinar
   os objetos na página.

Argumentos opcionais
````````````````````

* ``month_format``: Um formato que regula o formato do parâmetro ``month`` é
  usado. Esta deve ser na sintaxe aceita pelo Python ``time.strftime``.
  (Veja sobre strftime em http://docs.python.org/library/time.html#time.strftime.)
  Por padrão ``"%b"`` é definido, que é uma abreviação de mês de três letras
  (i.e., "jan", "fev", etc.). Para usar números, use ``"%m"``.

* ``allow_future``: Um booleano que especifica se existem objetos do "futuro"
  na página, mostrato anteriormente

Está view também possui os seguintes argumentos (veja na tabela C-1):

* ``allow_empty``
* ``context_processors``
* ``extra_context``
* ``mimetype``
* ``template_loader``
* ``template_name``
* ``template_object_name``

Template Name
`````````````

Se o ``template_name`` não for especificado, essa view irá usar o template
``<app_label>/<model_name>_archive_month.html`` por padrão.

Template Context
````````````````

Além do ``extra_context``, o template context seguirá o seguinte:

* ``month``: Um objeto ``datetime.date`` que represente um dado mês.

* ``next_month``: Um objeto ``datetime.date`` que representa o primeiro dia
  do poróximo mês. Se o mês seguinte for no futuro, esta será ``None``.

* ``previous_month``: Um objeto ``datetime.date`` representando o primeiro dia
  do mês anterior. Ao contrátio do ``next_month``, ele nunca retornará
  ``None``.

* ``object_list``: Uma lista de objetos disponíveis para um dado mês. O nome
  dessa variável depende do parâmetro ``template_object_name``, que é
  ``'object'`` por padrão. Se o ``template_object_name`` for ``'foo'``, essa
  o nome dessa variável será ``foo_list``.

Ârquivos semana
---------------

*View function*: ``django.views.generic.date_based.archive_week``

Essa view mostrará todos os objetos em uma determinada semana.

.. nota::

    Por uma questão de coerência com o built-in date/time do Python, o Django
    assume que o primeiro dia da semana é o domingo.

Exemplo
```````

.. parsed-literal::

    urlpatterns = patterns('',
        # ...
        **(**
            **r'^(?P<year>\d{4})/(?P<week>\d{2})/$',**
            **date_based.archive_week,**
            **book_info**
        **),**
    )


Argumentos obrigatórios
```````````````````````

* ``year``: The four-digit year for which the archive serves (a string).

* ``week``: The week of the year for which the archive serves (a string).

* ``queryset``: A ``QuerySet`` of objects for which the archive serves.

* ``date_field``: The name of the ``DateField`` or ``DateTimeField`` in the
  ``QuerySet``'s model that the date-based archive should use to determine
  the objects on the page.

Optional Arguments
``````````````````

* ``allow_future``: A Boolean specifying whether to include "future" objects
  on this page, as described in the previous note.

This view may also take these common arguments (see Table C-1):

* ``allow_empty``
* ``context_processors``
* ``extra_context``
* ``mimetype``
* ``template_loader``
* ``template_name``
* ``template_object_name``

Template Name
`````````````

If ``template_name`` isn't specified, this view will use the template
``<app_label>/<model_name>_archive_week.html`` by default.

Template Context
````````````````

In addition to ``extra_context``, the template's context will be as follows:

* ``week``: A ``datetime.date`` object representing the first day of the
  given week.

* ``object_list``: A list of objects available for the given week. This
  variable's name depends on the ``template_object_name`` parameter, which
  is ``'object'`` by default. If ``template_object_name`` is ``'foo'``, this
  variable's name will be ``foo_list``.

Day Archives
------------

*View function*: ``django.views.generic.date_based.archive_day``

This view generates all objects in a given day.

Example
```````

.. parsed-literal::

    urlpatterns = patterns('',
        # ...
        **(**
            **r'^(?P<year>\d{4})/(?P<month>[a-z]{3})/(?P<day>\d{2})/$',**
            **date_based.archive_day,**
            **book_info**
        **),**
    )


Required Arguments
``````````````````

* ``year``: The four-digit year for which the archive serves (a string).

* ``month``: The month for which the archive serves, formatted according to the
  ``month_format`` argument.

* ``day``: The day for which the archive serves, formatted according to the
  ``day_format`` argument.

* ``queryset``: A ``QuerySet`` of objects for which the archive serves.

* ``date_field``: The name of the ``DateField`` or ``DateTimeField`` in the
  ``QuerySet``'s model that the date-based archive should use to determine
  the objects on the page.

Optional Arguments
``````````````````

* ``month_format``: A format string that regulates what format the ``month``
  parameter uses. See the detailed explanation in the "Month Archives"
  section, above.

* ``day_format``: Like ``month_format``, but for the ``day`` parameter. It
  defaults to ``"%d"`` (the day of the month as a decimal number, 01-31).

* ``allow_future``: A Boolean specifying whether to include "future" objects
  on this page, as described in the previous note.

This view may also take these common arguments (see Table C-1):

* ``allow_empty``
* ``context_processors``
* ``extra_context``
* ``mimetype``
* ``template_loader``
* ``template_name``
* ``template_object_name``

Template Name
`````````````

If ``template_name`` isn't specified, this view will use the template
``<app_label>/<model_name>_archive_day.html`` by default.

Template Context
````````````````

In addition to ``extra_context``, the template's context will be as follows:

* ``day``: A ``datetime.date`` object representing the given day.

* ``next_day``: A ``datetime.date`` object representing the next day. If the
  next day is in the future, this will be ``None``.

* ``previous_day``: A ``datetime.date`` object representing the previous day.
  Unlike ``next_day``, this will never be ``None``.

* ``object_list``: A list of objects available for the given day. This
  variable's name depends on the ``template_object_name`` parameter, which
  is ``'object'`` by default. If ``template_object_name`` is ``'foo'``, this
  variable's name will be ``foo_list``.

Archive for Today
-----------------

The ``django.views.generic.date_based.archive_today`` view shows all objects for
*today*. This is exactly the same as ``archive_day``, except the
``year``/``month``/``day`` arguments are not used, and today's date is used
instead.

Example
```````

.. parsed-literal::

    urlpatterns = patterns('',
        # ...
        **(r'^books/today/$', date_based.archive_today, book_info),**
    )


Date-Based Detail Pages
-----------------------

*View function*: ``django.views.generic.date_based.object_detail``

Use this view for a page representing an individual object.

This has a different URL from the ``object_detail`` view; the ``object_detail``
view uses URLs like ``/entries/<slug>/``, while this one uses URLs like
``/entries/2006/aug/27/<slug>/``.

.. note::

    If you're using date-based detail pages with slugs in the URLs, you probably
    also want to use the ``unique_for_date`` option on the slug field to
    validate that slugs aren't duplicated in a single day. See Appendix A for
    details on ``unique_for_date``.

Example
```````

This one differs (slightly) from all the other date-based examples in that we
need to provide either an object ID or a slug so that Django can look up the
object in question.

Since the object we're using doesn't have a slug field, we'll use ID-based URLs.
It's considered a best practice to use a slug field, but in the interest of
simplicity we'll let it go.

.. parsed-literal::

    urlpatterns = patterns('',
        # ...
        **(**
            **r'^(?P<year>\d{4})/(?P<month>[a-z]{3})/(?P<day>\d{2})/(?P<object_id>[\w-]+)/$',**
            **date_based.object_detail,**
            **book_info**
        **),**
    )

Required Arguments
``````````````````

* ``year``: The object's four-digit year (a string).

* ``month``: The object's month, formatted according to the ``month_format``
  argument.

* ``day``: The object's day, formatted according to the ``day_format`` argument.

* ``queryset``: A ``QuerySet`` that contains the object.

* ``date_field``: The name of the ``DateField`` or ``DateTimeField`` in the
  ``QuerySet``'s model that the generic view should use to look up the
  object according to ``year``, ``month``, and ``day``.

You'll also need either:

* ``object_id``: The value of the primary-key field for the object.

or:

* ``slug``: The slug of the given object. If you pass this field, then the
  ``slug_field`` argument (described in the following section) is also
  required.

Optional Arguments
``````````````````

* ``allow_future``: A Boolean specifying whether to include "future" objects
  on this page, as described in the previous note.

* ``day_format``: Like ``month_format``, but for the ``day`` parameter. It
  defaults to ``"%d"`` (the day of the month as a decimal number, 01-31).

* ``month_format``: A format string that regulates what format the ``month``
  parameter uses. See the detailed explanation in the "Month Archives"
  section, above.

* ``slug_field``: The name of the field on the object containing the slug.
  This is required if you are using the ``slug`` argument, but it must be
  absent if you're using the ``object_id`` argument.

* ``template_name_field``: The name of a field on the object whose value is
  the template name to use. This lets you store template names in the data.
  In other words, if your object has a field ``'the_template'`` that
  contains a string ``'foo.html'``, and you set ``template_name_field`` to
  ``'the_template'``, then the generic view for this object will use the
  template ``'foo.html'``.

This view may also take these common arguments (see Table C-1):

* ``context_processors``
* ``extra_context``
* ``mimetype``
* ``template_loader``
* ``template_name``
* ``template_object_name``

Template Name
`````````````

If ``template_name`` and ``template_name_field`` aren't specified, this view
will use the template ``<app_label>/<model_name>_detail.html`` by default.

Template Context
````````````````

In addition to ``extra_context``, the template's context will be as follows:

* ``object``: The object. This variable's name depends on the
  ``template_object_name`` parameter, which is ``'object'`` by default. If
  ``template_object_name`` is ``'foo'``, this variable's name will be
  ``foo``.
