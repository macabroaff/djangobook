======================
Capítulo 17: Middleware
======================

Em algumas ocasiões, você precisa executar algum pedaço de código para cada request que o Django trata.
Esse código pode ser uma alteração na request antes que a view a trate, ou pode ser
para registrar informações sobre a request para propósitos de debugging, e assim por diante.

Você pode fazer isso com o framework de *middleware* do Django, que é um conjunto de "ganchos" (utilitários)
dentro do processamento de request/response. É um sistema de "plug-in" leve e de baixo nível capaz
de alterar globalmente as entradas e saídas do Django.

Cada componente do middleware é responsável por fazer uma função específica.
Se você está lendo este livro sequencialmente, você já deve ter visto middleware uma série de vezes:

* Todas as ferramentas de sessão e usuários que nós vimos no Capítulo 14 são
  possíveis graças ao middleware (mais especificamente, o middleware faz
  ``request.session`` e ``request.user`` estarem disponíveis nas views).

* O cache do site, discutido no Capítulo 15 é na verdade somente um
  middleware que ignora a chamada para sua view, se sua resposta já
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
e você pode ler um pouco mais sobre neste capítulo.


Instalação do Middleware
=======================

Se você está lendo este livro sequencialmente, você já deve ter visto uma série
de exemplos de instalação de middleware; muitos dos exemplos nos capítulos
anteriores precisavam de um certo middleware. Para completar, aqui está
como instalar um middleware.

Para ativar um componente middleware, adicione-o na tupla ``MIDDLEWARE_CLASSES``
no settings de seu módulo. Em ``MIDDLEWARE_CLASSES``, cada componente middleware
é representado por uma string: sendo o caminho Python completo para o nome da
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

Se a classe middleware define um metódo ``__init__()``, o metódo não deve receber
nenhum argumento além ``self``.

Pré-processador de Solicitação: process_request(self, request)
----------------------------------------------------

Este metódo é chamado assim que a solicitação (request) é recebida -- antes
do Django ter analisado a URL para determinar qual função de visualização (view)
será executada. Ele recebe o objeto ``HttpRequest``, que você pode modificar à vontade.

``process_request()`` deve retornar ``None`` ou um objeto ``HttpResponse``.

* Se retornar ``None``, o Django irá continuar processando a solicitação,
  executando qualquer outro middleware e então a função de visualização apropriada.

* Se retornar um objeto ``HttpResponse``, o Django não irá chamar *nenhum*
  outro middleware (de nenhum tipo), nem a função de visualização apropriada.
  Ele irá retornar imediatamente o ``HttpResponse``.


Pré-processador de Visualização: process_view(self, request, view, args, kwargs)
------------------------------------------------------------------

Este metódo é chamado depois que pré-processador de solicitação é chamado e
o Django determinou qual função de visualização será executada, mas antes
que a ela seja executada.

Os argumentos passados para esse metódo são mostrados na Tabela 17-1.

.. table:: Tabela 17-1. Argumentos passados para o process_view()

    ==============  ==========================================================
    Argumento       Descrição
    ==============  ==========================================================
    ``request``     O objeto ``HttpRequest``.

    ``view``        Função Python que o Django irá chamar para tratar essa
                    solitação. Isto é, uma referência ao objeto da função
                    e não o nome ou a função em string.

    ``args``        Uma lista de argumentos posicionados, que serão passados
                    para a função de visualização, não incluindo o argumento
                    ``request`` (que é sempre o primeiro argumento para a função
                    de visualização).

    ``kwargs``      O dicionário de palavras-chave que será passado para a função de visualização.
    ==============  ==========================================================

Assim como ``process_request()``, ``process_view()`` deve retornar ``None`` ou
um objeto ``HttpResponse``.

* Se retornar ``None``, o Django irá continuar processando a request,
  executando qualquer outro middleware e então a view apropriada.

* Se retornar um objeto ``HttpResponse``, o Django não irá chamar *nenhum*
  outro middleware (de nenhum tipo), nem a view apropriada. Django irá
  retornar imediatamente o ``HttpResponse``.

Pós-processador de resposta: process_response(self, request, response)
-----------------------------------------------------------------

Este metódo é chamado depois que a função de visualização é executada
e a resposta é gerada. Aqui, o processador pode modificar o conteúdo da resposta.
Um caso óbvio de uso é compressão do conteúdo, como gzipping do pedido HTML.

Os parâmetros devem ser bastante auto-explicativos: ``request`` é o objeto
request, e ``response`` é o objeto response retornado pela função de visualização.

Diferente dos pré-processadores de solicitação e visualização, que podem retornar ``None``,
``process_response()`` *deve* retornar um objeto ``HttpResponse``.
A resposta pode ser o response original passado para a função (possivelmente
modificado) ou um novo.

Pós-processador de exceção: process_exception(self, request, exception)
--------------------------------------------------------------------

Este metódo é chamado somente se algo ocorreu errado e a função de visualização
gerou uma exceção não capturada. Você pode usar esse "gancho" para enviar notificações
de erro, despejar informações em um log, ou até mesmo tentar recuperar do
erro automaticamente.

Os parâmetros para essa função é o mesmo objeto ``request`` que estamos
lidando durante o tempo todo e ``exception``, que é o objeto ``Exception``
gerado pela função de visualização.

``process_exception()`` deve retornar ou ``None`` ou um objeto ``HttpResponse``.

* Se ele retornar ``None``, Django irá continuar processando a solicitação
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


Middleware de Suporte a Autenticação
---------------------------------

classe Middleware: ``django.contrib.auth.middleware.AuthenticationMiddleware``.

Este middleware ativa o suporte à autenticação. Ele adiciona o atributo
``request.user`` que representa o usuário atualmente logado para cada
objeto request recebido.

Veja o Capítulo 14 para detalhes completos.


"Common" Middleware
-------------------

classe Middleware: ``django.middleware.common.CommonMiddleware``.

Este middleware adiciona algumas conveniências para perfeccionistas:

* *Proíbe o acesso para user agents especificados na configuração ``DISALLOWED_USER_AGENTS``*:
  Se definida, essa configuração deve ser uma lista de expressões regulares compiladas,
  quais são verificados contra o cabeçalho user-agent para cada request recebida.
  Aqui está um exemplo de fragmento extraído de um arquivo de configuração::

      import re

      DISALLOWED_USER_AGENTS = (
          re.compile(r'^OmniExplorer_Bot'),
          re.compile(r'^Googlebot')
      )

  Perceba o ``import re``, porque o ``DISALLOWED_USER_AGENTS`` requer
  que seus valores sejam expressões regulares compiladas (i.e., o retorno de ``re.compile()``).
  O arquivo de configuração é um arquivo Python, então é perfeitamente normal incluir
  a declaração ``import`` Python nele.

* *Reescreve a URL baseando-se nas configurações ``APPEND_SLASH`` e ``PREPEND_WWW``*:
  Se `APPEND_SLASH`` estiver definido como ``True``, nas URLs que estiverem faltando
  a barra invertida serão redirecionadas para a mesma URL com a barra invertida,
  a não ser que o último componente do caminho possua uma extensão. Então ``foo.com/bar``
  é redirecionada para ``foo.com/bar/``, mas `foo.com/bar/file.txt`` é transmitida
  sem nenhuma mudança.

  Se ``PREPEND_WWW`` estiver definido como ``True``, URLs que estivem faltando
  o "www." no início serão redirecionadas para a mesma URL, iniciando com "www.".

  Ambas as opções pretendem normalizar as URLs. Esta filosofia é que cada
  URL deve existir em um -- e somente um -- lugar. Tecnicamente a URL
  ``example.com/bar`` é diferente de ``example.com/bar/``, o que a torna
  diferente também de ``www.example.com/bar/``. Um indexador de um mecanismo de
  busca irá tratar como URLs separadas, o que é prejudicial para seus site no
  rankings dos mecanismos de busca, então essa é a melhor prática para normalizar URLs.

* *Trata as ETags baseando-se na configuração ``USE_ETAGS``*: *ETags* são otimização
  no nível HTTP para cachear páginas condicionalmente. Se ``USE_ETAGS`` está definido
  como ``True``, Django irá calcular uma ETag para cada request fazendo um hash MD5 do
  conteúdo da página, e irá tomar cuidado em mandar respostas ``Not Modified`` (Não modificado),
  se necessário.

  Perceba que existe um middleware para ``GET`` condicional, que será discutido em breve,
  que lida com as ETags e faz um pouco mais.


Middleware de compressão
----------------------

classe Middleware: ``django.middleware.gzip.GZipMiddleware``.

Este middleware comprime automaticamente o conteúdo para navegadores que entendem
a compressão gzip (todos os navegadores modernos). Isso pode ser uma grande redução
na largura de banda que um servidor Web consome.

Nós costumamos preferir velocidade a largura de banda, mas se você prefere o contrário,
é só ativar esse middleware.


Middleware de GET condicional
--------------------------

classe Middleware: ``django.middleware.http.ConditionalGetMiddleware``.

Este middleware fornece suporte para operações de ``GET```condicional.
Se a resposta tiver um ``Last-Modified`` ou ``ETag`` ou cabeçalho,
e a solicitação tem um ``If-None-Match`` ou ``If-Modified-Since``, a resposta é
substuída por uma resposta 304 ("Not modified" -- não modificado --).
O suporte para ``ETag`` depende da configuração ``USE_ETAGS`` e espera que
o cabeçalho de resposta ``ETag`` esteja definido. Como discutido anteriormente,
o cabeçalho ``ETag`` é definido pelo "Common" middleware.

Além disso remove o conteúdo de qualquer resposta para um solicitação ``HEAD``
e define os cabeçalhos de resposta ``Date`` e ``Content-Length`` para todas as
solicitações.


Suporte a proxy reverso (X-Forwarded-For Middleware)
--------------------------------------------------

classe Middleware: ``django.middleware.http.SetRemoteAddrFromForwardedFor``.

Este é o exemplo que examinamos anteriormente na seção "O que é um Middleware?".
Ele define o ``request.META['REMOTE_ADDR']`` baseado em
``request.META['HTTP_X_FORWARDED_FOR']``, se o último for definido. Isto é útil
se você está atrás de um proxy reverso, o que faz com que o ``REMOTE_ADDR`` seja
definido como ``127.0.0.1`` em cada solicitação.


.. admonition:: Cuidado!

    Este middleware *não* valida o ``HTTP_X_FORWARDED_FOR``.

    Se você não está atrás de um proxy reverso que define automaticamente
    o ``HTTP_X_FORWARDED_FOR``, não use este middleware. Qualquer um pode
    falsificar o valor de ``HTTP_X_FORWARDED_FOR``, e porque o ``REMOTE_ADDR``
    define seu valor baseado no ``HTTP_X_FORWARDED_FOR``, isto significa
    que qualquer um pode falsificar seu endereço IP.

    Somente use este middleware quando você pode confiar absolutamente no
    valor de ``HTTP_X_FORWARDED_FOR``.


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
