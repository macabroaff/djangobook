==================================
Apêndice C: Generic View Reference
==================================

No capítulo 11 fomos introduzidos ao Genetic View, porém de forma superficial e
sem maiores detalhes. Neste apêndice veremos todos os pontos importantes não
abordados anteriormente, porém é altamente recomendado que você leia o capítulo
11 antes de tentar entender as referências abaixo. Talvez você queira rever os 
objetos ``Livro``, ``Editor``, e ``Autor`` definido nesse capítulo, os exemplos 
a seguir usam esses modelos.


Argumentos comuns nas Generic Views
===================================

A maioria dessas views possuem uma grande quantidade de argumentos que
possibilitam a modificação do comportamento padrão das generic views. Muitos
desse argumentos fumcionam da mesma forma na maioria das views. A tabela C-1
descreve os argumentos mais comuns; sempre que você encontrar um desses
argumentos, ele funcionará da forma descrita abaixo

.. table:: Tabela C-1. Argumentos comuns nas Generic Views

    ==========================  ===============================================
    Argumento                   Descrição
    ==========================  ===============================================
    ``allow_empty``             Um booleano que especifica se, para carregar a
                                página, existem ou não objetos disponíveis.
                                Se este for ``False`` e nenhum objeto estiver
                                disponível, a view lançará um 404 ao invés de
                                mostrar uma página vazia. O valor padrão é 
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

    ``mimetype``                O MIME type a ser usado para o documento
                                resultante. Caso nenhum valor seja fornecido,
                                será utilizado o valor de
                                ``DEFAULT_CONTENT_TYPE`` do arquivo de
                                settings.py que é ``text/html``.

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
                                seu parâmetro é um ``object``. Views que listam
                                mais que um objeto (exemplo views ``object_list``)
                                vão adicionar ``_list`` no valor do parâmetro.
    ==========================  ===============================================

"Simple" Generic Views
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

Dado o objeto ``Autor`` do capítulo 5, podemos usar a view ``object_list``
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
  por página. A view espera por uma página que possua
  uma query string (enviada via ``GET``) com indice zero ou uma página variável
  especificada na URLconf. Veja a seção "Notas de paginação".

Além destes, essa view pode utilizar qualquer um desses argumentos descritos na
Tabela C-1.

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
``<app_label>/<model_name>_list.html`` por padrão. Tanto a label da aplicação
quanto o nome do modelo são derivados do parâmetro ``queryset``. A label da
aplicação é o nome do aplicativo que o medelo está definito, e o nome do modelo
é a versão minúscula do nome do modelo de classe.

No exemplo anterior usando ``Author.objects.all()`` como uma ``queryset``, a
label da aplicação seria ``livros`` e o nome do módulo seria ``autor``. Assim o 
nome padrão do template seria ``livros/autor_list.html``.

Template Context
````````````````

Além do ``extra_context``, o template context irá conter o seguinte:

* ``object_list``: Lista de objetos. O nome dessa variável depende do parametro
  ``template_object_name``, que é ``'object'`` por padrão. Se
  ``template_object_name`` é ``foo``, o nome dessa variável será ``foo_list``.

* ``is_paginated``: Um booleano que indica se o resultado é paginado.
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
    Você pode especificar o número da página na URL de duas maneiras:

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


* ``template_name_field``: O nome de um campo no objeto cujo valor é o usado 
  pelo template name.

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
  ``template_object_name``, que é ``'object'`` por padrão. Se o
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
um determinado mês.

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

* ``month_format``: Um formato que regula o formato do parâmetro ``month`` utilizar.
  Esta deve ser na sintaxe aceita pelo Python ``time.strftime``. (Veja sobre 
  strftime em http://docs.python.org/library/time.html#time.strftime.)
  Por padrão ``"%b"`` é definido, que é uma abreviação de mês de três letras
  (i.e., "jan", "fev", etc.). Para usar números, use ``"%m"``.

* ``allow_future``: Um booleano que especifica se existem objetos do "futuro"
  na página, mostrato anteriormente

Esta view também possui os seguintes argumentos (veja na tabela C-1):

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

Arquivos semana
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

* ``year``: O ano de quatro dígitos para que o arquivo serve (uma string).

* ``week``: A semana do ano para o qual o arquivo serve (uma string).

* ``queryset``: Um ``QuerySet`` de objetos para os quais o arquivo serve.

* ``date_field``: O nome do ``DateField`` ou ``DateTimeField`` na ``QuerySet``
  que o arquivo data-base deve usar para determinar os objetos na página.

Argumentos opcionais
````````````````````

* ``allow_future``: Um booleano que especifica se incluem "futuros" objetos
   nessa página, conforme descrito anteriormente.

Esta view também possui os seguintes argumentos (veja na tabela C-1):

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
``<app_label>/<model_name>_archive_week.html`` por padrão.

Template Context
````````````````

Além do ``extra_context``, o Template Context irá seguir o seguinte:

* ``week``: Um objeto ``datetime.date`` que representa o primeiro dia de uma
  dada semana.

* ``object_list``: Uma lista de objetos disponíveis para uma dasa semana.
  O nome dessa variável depende do parâmetro ``template_object_name``, que é
  um ``'object'`` por padrão. Se o ``template_object_name`` for ``'foo'``,
  o nome dessa variável será ``foo_list``.

Arquivos dia
------------

*View function*: ``django.views.generic.date_based.archive_day``

Essa view gera gera todos os objetos em um determinado dia.

Exemplo
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


Argumentos obrigatórios
```````````````````````

* ``year``: O ano de quatro dígitos para que o arquivo serve (uma string).

* ``month``: O mês em que o arquivo serve, formatado de acordo com o argumento
  ``month_format``.

* ``day``: O dia em que o arquivo serve, formatado de acordo com o argumento
  ``day_format``.

* ``queryset``: Uma ``QuerySet`` de objetos para os quais o arquivo serve.

* ``date_field``: O nome do ``DateField`` ou ``DateTimeField`` na ``QuerySet``
  que o arquivo data-base deve usar para determinar os objetos na página.

Argumentos opcionais
````````````````````

* ``month_format``: Um formato que regula o que o formato do parâmetro
  ``month`` usa. Veja a explicação detalhada na seção "Arquivos Mês".

* ``day_format``: Como ``month_format``, mas para o parâmetro ``day``.
  ele é padrão para ``"%d"`` (o dia do mês como um número decimal, 01-31).

* ``allow_future``: Um booleano que especifica se incluem "futuros" objetos
   nessa página, conforme descrito anteriormente.

Esta view também possui os seguintes argumentos (veja na tabela C-1):

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
``<app_label>/<model_name>_archive_day.html`` por padrão.

Template Context
````````````````

Além fo ``extra_context``, o Template Context seguirá o seguinte:

* ``day``: Um objeto ``datetime.date`` representando um dado dia.

* ``next_day``: Um objeto ``datetime.date`` representando o próximo dia. Se o
  próximo dia for um dia futuro, ele retornará ``None``.

* ``previous_day``: Um objeto ``datetime.date`` representando o dia anterior.
  Diferente do ``next_day``, ele nunca retornará ``None``.

* ``object_list``: Uma lista de objetos disponíveis para o dia. O nome da
  variável depende do parâmetro ``template_object_name``, que é um ``'object'``
  por padrão. Se o ``template_object_name`` for ``'foo'``, essa variável será
  ``foo_list``.

Arquivos para hoje
------------------

A view ``django.views.generic.date_based.archive_today`` mostra todos os
objetos de *hoje*. É exatamente igual a ``archive_day``, exeto os argumentos
``year``/``month``/``day`` que não são utilizados, em vez disso será usada a
data de hoje.

Exemplo
```````

.. parsed-literal::

    urlpatterns = patterns('',
        # ...
        **(r'^books/today/$', date_based.archive_today, book_info),**
    )


Página de detalhes Date-Based
-----------------------------

*View function*: ``django.views.generic.date_based.object_detail``

Utilize esta view para uma página que representa um objeto individual.

Isto tem um URL diferente da view ``object_detail``; a view ``object_detail``
usa URLs como ``/entries/<slug>/``, enquanto essa usa URLs como
``/entries/2006/aug/27/<slug>/``.

.. nota::

    Se você estiver usando páginas date-based de detalhe com slugs nas URLs,
    você provavelmente terá que usar como opção a ``unique_for_date`` no slug
    para validar essa slugs não são duplicadas em um único dia. Veja o apêndice
    A para maioresd informações sobre ``unique_for_date``.

Exemplo
```````

Esse é (ligeiramente) diferente de todos os outros exemplos de date-based 
que precisamos fornecer o ID de um objeto ou uma slug para que o
Django tenha formas de encontrar o objeto em questão.

Uma vez que o objeto que estamos usando não tem um campo de slug, usaremos ID
baseado em URLs. É considerada uma boa prática usar um campo de slug, mas por
questões de exemplo, vamos deixar sem.

.. parsed-literal::

    urlpatterns = patterns('',
        # ...
        **(**
            **r'^(?P<year>\d{4})/(?P<month>[a-z]{3})/(?P<day>\d{2})/(?P<object_id>[\w-]+)/$',**
            **date_based.object_detail,**
            **book_info**
        **),**
    )

Argumentos obrigatórios
```````````````````````

* ``year``: Um objeto ano de quatro dígitos (uma string).

* ``month``: Um objeto mês, formatado de acordo com o argumento
  ``month_format``

* ``day``: Um objeto dia, formatado de acordo com o argumento ``day_format``.

* ``queryset``: Uma ``QuerySet`` que contém o objeto.

* ``date_field``: O nome do ``DateField`` ou ``DateTimeField`` na ``QuerySet``
  que essa generic view utilizar para procurar o objeto conforme o ``ano``,
  ``mês`` e ``dia``.

Você também precisa de:

* ``object_id``: O valor do campo primary-key para o objecto.

ou:

* ``slug``: O slug de um determinado objeto. Se você informar este campo, o
  argumento ``slug_field`` (descrito na próxima seção) será obrigatório.

Argumentos opcionais
````````````````````

* ``allow_future``: Um booleano que especifica se incluem "futuros" objetos
   nessa página, conforme descrito anteriormente.

* ``day_format``: Igual ao ``month_format``, mas para o parâmetro ``day``. Ele
  é padrão ``"%d"`` (o dia do mês como um número decimal, 01-31).

* ``month_format``: Um formato que regula o formato ``mês``. Veja a explicação
  detalhada na seção "Arquivos Mês".

* ``slug_field``: O nome do campo que contenha o slug. Será obrigatório se você
  usar o argumento ``slug``, mas não será se usar o parâmetro ``object_id``.

* ``template_name_field``: O nome de um campo no objeto cujo valor é o nome que 
  o template irá usar. Isto permite armazenar nomes de modelo nos dados. Em 
  outras palavras, se o seu objeto tiver o campo ``'the_template'``,  este conterá 
  a string ``'foo.html'``, e se você definir o ``template_name_field``, a generic 
  view para esse objeto irá usar o template ``'foo.html'``.

Esta view também possui os seguintes argumentos (veja na tabela C-1):

* ``context_processors``
* ``extra_context``
* ``mimetype``
* ``template_loader``
* ``template_name``
* ``template_object_name``

Template Name
`````````````

Se o ``template_name`` e o ``template_name_field`` não forem especificados,
essa view irá usar o template ``<app_label>/<model_name>_detail.html`` por
padrão

Template Context
````````````````

Além do ``extra_context``, o template context irá seguir:

* ``object``: Objeto. O nome dessa variável depende do parâmetro 
  ``template_object_name``, que é ``'object'`` por padrão. Se o 
  ``template_object_name`` for ``'foo'``, o nome dessa variável será ``foo``.
