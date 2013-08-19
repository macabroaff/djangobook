======================
Capítulo 17: Middleware
======================

Em algumas ocasiões, você precisa executar algum de código para cada request que o Django trata.
Esse código pode ser uma alteração na request antes que a view a trate, pode ser
para registrar informações sobre a request para propósitos de debugging, e assim por diante.

Você pode fazer isso com framework de *middleware* do Django, que é um conjunto de "ganchos" (utilitários)
dentro do processamento de request/response. É um sistema de "plug-in" leve e de baixo nível capaz
de alterar globalmente as entradas e saídas do Django.

Cada componente do middleware é responsável por fazer alguma função específica.
Se você está lendo este livro sequencialmente, você já deve ter middleware uma série de vezes:

* Todas as ferramentas de sessão e usuários que nós vimos no Capítulo 14 são
  possívels graças a alguns pedaços de middleware (mais
  especificamente, o middleware faz ``request.session`` e
  ``request.user`` estarem disponíveis nas views).

* O cache do site, discutido no Capítulo 15 é na verdade somente um pedaço
  de middleware que ignora a chamada para sua view, se sua resposta já
  estiver registrada no cache.

* As aplicações ``flatpages``, ``redirects`` e ``csrf`` do Capítulo 16,
  todas fazem sua magia através dos componentes de middleware.

Este capítulo mergulha exatamente em o que é o middleware e como ele funciona,
e explica como você pode escrever o seu próprio middleware.


O que é um Middleware?
==================

Vamos começar com um exemplo bem simples.

Sites com alto tráfego, muitas vezes precisam implantar o Django atrás de um proxy
de balanceamento de carga (veja o Capítulo 12). Isto pode causar algumas pequenas
complicações, uma delas é que cada IP remoto (``request.META["REMOTE_IP"]``)
será o do proxy de balanceamento, e não o IP atual que está realizando a request.
Balanceadores de carga lidam com isto, definindo um cabeçalho especial, ``X-Forwardded-For``,
para o endereço IP que está fazendo a solicitação.


Então aqui está um pequeno pedaço de middleware que permite sites executarem atrás
de um proxy e ainda visualizarem o endereço de IP correto em ``request.META["REMOTE_ADDR"]``::

    class SetRemoteAddrFromForwardedFor(object):
        def process_request(self, request):
            try:
                real_ip = request.META['HTTP_X_FORWARDED_FOR']
            except KeyError:
                pass
            else:
                # HTTP_X_FORWARDED_FOR pode ser uma lista de IPs separados por vírgula.
                # Pegue o somente o primeiro.
                real_ip = real_ip.split(",")[0]
                request.META['REMOTE_ADDR'] = real_ip

(Nota: Embora o cabeçalho HTTP é chamado ``X-Forwarded-For``, Django o deixa
disponível como ``request.META['HTTP_X_FORWARDED_FOR']``. Com a exceção de
``content-length`` e ``content-type``, qualquer cabeçalho HTTP na request
são convertidos para chaves no ``request.META``, convertendo todos seus caracteres
para maiúsculo, substituindo os hífens por underscores e adicionando o prefixo ``HTTP_``
em seu nome.)

Se este middleware está instalado (veja a próxima seção), em toda request
o valor de ``X-Forwarded-For`` será inserido automaticamente dentro de
``request.META['REMOTE_ADDR']``. Isto significa que suas aplicações Django
não precisam se preocupar se estão atrás de um proxy de balanceamento de carga ou não;
elas podem simplesmente acessar ``request.META['REMOTE_ADDR']``, e isso irá funcionar
se estiverem ou não utilizando um proxy.

Na verdade, é uma necessidade bastante comum que essa parte de middleware
seja embutida no Django. Ela vive em ``django.middleware.http``,
e você pode ler um pouco mais sobre isso depois neste capítulo.


Instalação do Middleware
=======================

Se você está lendo este livro sequencialmente, você já deve ter visto uma série
de exemplos de instalação de middleware; muitos dos exemplos nos capítulos
anteriores precisavam de um certo middleware. Para completar, aqui está
como instalar um middleware.

Para ativar um componente middleware, adicione-o na tupla ``MIDDLEWARE_CLASSES``
no settings de seu módulo. Em ``MIDDLEWARE_CLASSES``, cada componente middleware
é representado por uma string: sendo caminho Python completo para o nome da
classe do middleware. Como exemplo, aqui está o ``MIDDLEWARE_CLASSES`` padrão
criado por ``django-admin.py startproject``::

    MIDDLEWARE_CLASSES = (
        'django.middleware.common.CommonMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
    )

Uma instalação Django não requer nenhum middleware -- ``MIDDLEWARE_CLASSES``
pode ser vazio, se você quiser -- mas recomendamos que você ative ``CommonMiddleware``,
qual iremos explicar em breve.

A ordem é importante. Na fase de request e view (visualização), Django executa
o middleware na ordem definida em ``MIDDLEWARE_CLASSES``, e na fase de
response (resposta) e exception (exceção ou erros) eles são executadas na ordem inversa.
Isto é, Django trata ``MIDDLEWARE_CLASSES`` como uma espécie de "wrapper" em volta da view:
na request ele caminha de cima para baixo até a view, e no response ele faz o caminho de volta.

Metódos do Middleware
==================

Agora que você sabe o que é um middleware e como instalá-lo, vamos dar uma olhada
em todos os metódos disponíveis que a classe do middleware pode definir.

Inicializador: __init__(self)
---------------------------

Use ``__init__()`` para executar a configuração de todo o sistema para uma determinada
classe middleware.

Por razões de perfomance, cada classe de middleware ativado é instanciada
somente *uma* vez por processo no servidor. Isto significa que ``__init__()``
é chamado somente uma vez -- ao iniciar o servidor -- e não para requests individuais.

Uma razão comum para implementar um metódo ``__init__()`` é para checar se o
middleware é realmente necessário. Se o ``__init__()`` gerar ``django.core.exceptions.MiddlewareNotUsed``,
então o Django irá remover o middleware da fila de execução. Você pode usar esse recurso
para checar se algum pedaço do software que a classe do middleware requer, ou checar
se o servidor está rodando em modo de debug, ou qualquer outra situação de ambiente.

Se a classe middleware define um metódo ``__init__()``, o metódo deve deve receber
nenhum argumento além do por padrão ``self``.

Pré-processador de Request: process_request(self, request)
----------------------------------------------------

Este metódo é chamado assim que a request é recebida -- antes do Django ter
analisado a URL para determinar qual view será executada. Ele recebe o
objeto ``HttpRequest``, que você pode modificar à vontade.

``process_request()`` deve retornar ``None`` ou um objeto ``HttpResponse``.

* Se retornar ``None``, o Django irá continuar processando a request,
  executando qualquer outro middleware e então a view apropriada.

* Se retornar um objeto ``HttpResponse``, o Django não irá chamar *nenhum*
  outro middleware (de nenhum tipo), nem a view apropriada. Django irá
  retornar imediatamente o ``HttpResponse``.


Pré-processador de View: process_view(self, request, view, args, kwargs)
------------------------------------------------------------------

Este metódo é chamado depois que pré-processador de request é chamado e
o Django determinou qual view será executada, mas antes que a view seja
executada.

Os argumentos passados para esse metódo são mostrados na Tabela 17-1.

.. table:: Tabela 17-1. Argumentos passados para o process_view()

    ==============  ==========================================================
    Argumento       Descrição
    ==============  ==========================================================
    ``request``     O objeto ``HttpRequest``.

    ``view``        Função Python que o Django irá chamar para tratar essa
                    request. Isto é, uma referência ao objeto da função
                    e não o nome ou a função em string.

    ``args``        Uma lista de argumentos posicionados, que serão passados
                    para a view, não incluindo o argumento ``request`` (que
                    é sempre o primeiro argumento para a view).

    ``kwargs``      O dicionário de palavras-chave que será passado para a view.
    ==============  ==========================================================

Assim como ``process_request()``, ``process_view()`` deve retornar ``None`` ou
um objeto ``HttpResponse``.

* Se retornar ``None``, o Django irá continuar processando a request,
  executando qualquer outro middleware e então a view apropriada.

* Se retornar um objeto ``HttpResponse``, o Django não irá chamar *nenhum*
  outro middleware (de nenhum tipo), nem a view apropriada. Django irá
  retornar imediatamente o ``HttpResponse``.

Pós-processador de Response: process_response(self, request, response)
-----------------------------------------------------------------

Este metódo é chamado depois que a view é executada e resposta é gerada.
Aqui, o processador pode modificar o conteúdo da resposta. Um caso óbvio de uso
é compressão do conteúdo, como gzipping do pedido HTML.

Os parâmetros devem ser bastante auto-explicativos: ``request`` é o objeto
request, e ``response`` é o objeto response retornado pela view.

Diferente do request e view pré-processadores, que podem retornar ``None``,
``process_response()`` *deve* retornar um objeto ``HttpResponse``.
A resposta pode ser o response original passado para a função (possivelmente
modificado) ou um novo.

Pós-processador de Exception: process_exception(self, request, exception)
--------------------------------------------------------------------

Este metódo é chamado somente se algo ocorreu errado e a view gerou uma
exceção não capturada. Você pode usar esse "gancho" para enviar notificações
de erro, despejar informações em um log, ou até mesmo tentar recuperar do
erro automaticamente.

Os parâmetros para essa função é o mesmo objeto ``request`` que estamos
lidando durante o tempo todo e ``exception``, que é o objeto ``Exception``
gerado pela view.


``process_exception()`` deve retornar ou ``None`` ou um objeto ``HttpResponse``.

* Se ele retornar ``None``, Django irá continuar processando a request
  com seu tratamento de exceção padrão.

* Se ele retornar um objeto ``HttpResponse``, Django irá usar essa resposta
  ao invés do tratamento de exceção padrão.

.. note::
    Django vem com um número de classes de middlwares (discutido na seção seguinte)
    que são bons exemplos. Lendo o código deles devem lhe dar uma boa noção do
    poder de um middleware.

    Você pode também encontrar um grande número de exemplos que a comunidade escreveu
    na wiki do Django: http://code.djangoproject.com/wiki/ContributedMiddleware


Middlewares embutidos
===================

Django vem com alguns middlewares embutidos para lidar com problemas comuns, que
descutiremos nas seções seguintes.


Authentication Support Middleware
---------------------------------

Middleware class: ``django.contrib.auth.middleware.AuthenticationMiddleware``.

This middleware enables authentication support. It adds the ``request.user``
attribute, representing the currently logged-in user, to every incoming
``HttpRequest`` object.

See Chapter 14 for complete details.

"Common" Middleware
-------------------

Middleware class: ``django.middleware.common.CommonMiddleware``.

This middleware adds a few conveniences for perfectionists:

* *Forbids access to user agents in the ``DISALLOWED_USER_AGENTS`` setting*:
  If provided, this setting should be a list of compiled regular expression
  objects that are matched against the user-agent header for each incoming
  request. Here's an example snippet from a settings file::

      import re

      DISALLOWED_USER_AGENTS = (
          re.compile(r'^OmniExplorer_Bot'),
          re.compile(r'^Googlebot')
      )

  Note the ``import re``, because ``DISALLOWED_USER_AGENTS`` requires its
  values to be compiled regexes (i.e., the output of ``re.compile()``).
  The settings file is regular Python, so it's perfectly OK to include
  Python ``import`` statements in it.

* *Performs URL rewriting based on the ``APPEND_SLASH`` and ``PREPEND_WWW``
  settings*: If ``APPEND_SLASH`` is ``True``, URLs that lack a trailing
  slash will be redirected to the same URL with a trailing slash, unless
  the last component in the path contains a period. So ``foo.com/bar`` is
  redirected to ``foo.com/bar/``, but ``foo.com/bar/file.txt`` is passed
  through unchanged.

  If ``PREPEND_WWW`` is ``True``, URLs that lack a leading "www." will be
  redirected to the same URL with a leading "www.".

  Both of these options are meant to normalize URLs. The philosophy is
  that each URL should exist in one -- and only one -- place. Technically the
  URL ``example.com/bar`` is distinct from ``example.com/bar/``, which in
  turn is distinct from ``www.example.com/bar/``. A search-engine indexer
  would treat these as separate URLs, which is detrimental to your site's
  search-engine rankings, so it's a best practice to normalize URLs.

* *Handles ETags based on the ``USE_ETAGS`` setting*: *ETags* are an HTTP-level
  optimization for caching pages conditionally. If ``USE_ETAGS`` is
  set to ``True``, Django will calculate an ETag for each request by
  MD5-hashing the page content, and it will take care of sending ``Not
  Modified`` responses, if appropriate.

  Note there is also a conditional ``GET`` middleware, covered shortly, which
  handles ETags and does a bit more.

Compression Middleware
----------------------

Middleware class: ``django.middleware.gzip.GZipMiddleware``.

This middleware automatically compresses content for browsers that understand gzip
compression (all modern browsers). This can greatly reduce the amount of bandwidth
a Web server consumes. The tradeoff is that it takes a bit of processing time to
compress pages.

We usually prefer speed over bandwidth, but if you prefer the reverse, just
enable this middleware.

Conditional GET Middleware
--------------------------

Middleware class: ``django.middleware.http.ConditionalGetMiddleware``.

This middleware provides support for conditional ``GET`` operations. If the response
has an ``Last-Modified`` or ``ETag`` or header, and the request has ``If-None-Match``
or ``If-Modified-Since``, the response is replaced by an 304 ("Not modified")
response. ``ETag`` support depends on on the ``USE_ETAGS`` setting and expects
the ``ETag`` response header to already be set. As discussed above, the ``ETag``
header is set by the Common middleware.

It also removes the content from any response to a ``HEAD`` request and sets the
``Date`` and ``Content-Length`` response headers for all requests.

Reverse Proxy Support (X-Forwarded-For Middleware)
--------------------------------------------------

Middleware class: ``django.middleware.http.SetRemoteAddrFromForwardedFor``.

This is the example we examined in the "What's Middleware?" section earlier. It
sets ``request.META['REMOTE_ADDR']`` based on
``request.META['HTTP_X_FORWARDED_FOR']``, if the latter is set. This is useful
if you're sitting behind a reverse proxy that causes each request's
``REMOTE_ADDR`` to be set to ``127.0.0.1``.

.. admonition:: Danger!

    This middleware does *not* validate ``HTTP_X_FORWARDED_FOR``.

    If you're not behind a reverse proxy that sets ``HTTP_X_FORWARDED_FOR``
    automatically, do not use this middleware. Anybody can spoof the value of
    ``HTTP_X_FORWARDED_FOR``, and because this sets ``REMOTE_ADDR`` based on
    ``HTTP_X_FORWARDED_FOR``, that means anybody can fake his IP address.

    Only use this middleware when you can absolutely trust the value of
    ``HTTP_X_FORWARDED_FOR``.

Middleware de Sessão
--------------------------

Middleware class: ``django.contrib.sessions.middleware.SessionMiddleware``.

Este middleware ativa suporte a sessão. Veja o capítulo 14 para mais detalhes.


Middleware de Cache do site
-------------------------

Middleware classes: ``django.middleware.cache.UpdateCacheMiddleware`` e
``django.middleware.cache.FetchFromCacheMiddleware``.

Estes middlewares trabalham juntos para fazer o cache de todas as páginas construídas com Django.
Isso foi discutido em detalhes no capítulo 15.


Middleware de Transação
----------------------

Middleware class: ``django.middleware.transaction.TransactionMiddleware``.

Este middleware monitora o ``COMMIT`` ou ``ROLLBACK`` no banco de dados na fase de request/response.
Se a view executar com sucesso, um ``COMMIT`` é emitido.
Se a view executar com erro e gerar uma exceção, um ``ROLLBACK`` é emitido.

A ordem que este middleware é inserido é importante. Módulos de middleware executando antes
desse (inseridos antes na listagem de middlewares), executam com commit-on-save -- compartamento padrão do Django.
Módulos de middleware executando depois desse (inseridos após na listagem de middlewares) estarão sob
o mesmo controle de transação assim como as funções de visualizações (views).

Veja o Apêndice B para mais informações sobre as transações em banco de dados.


O que vem em seguida?
============

Desenvolvedores web e projetistas de banco de dados, nem sempre têm o luxo de começar do zero.
No `próximo`_ capítulo, abordaremos como fazer integração com sistemas legados,
assim como esquemas de banco de dados herdados da decáda de 80.

.. _próximo: /chapter18.rst
