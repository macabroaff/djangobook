======================================
Apêndice G: Objetos Request e Response
======================================

O Django usa os objetos request e response para passar o estado do sistema. 

Quando uma página é requisitada, o Django cria um objeto ``HttpRequest``, que
contêm um metadado sobre a requisição. Então o Django executa a view apropriada, 
passado o ``HttpRequest`` como primeiro argumento para a função view. Cada
view é responsável por retornar um objeto ``HttpResponse``.

Nós usaremos esses objetos com maior freqüência durante o livro; esse apêndice explica 
todos as APIs dos objetos ``HttpRequest`` e ``HttpResponse``.

HttpRequest
===========

``HttpRequest`` representa uma simples requisição HTTP de algum agente de usuário.

Muitas informações importantes sobre o request está disponível como atributo na
instância ``HttpRequest`` (olhe a Tabela G-1). Todos os atributos exceto o 
``session`` devem ser considerados somente leitura.

.. table:: Tabela G-1. Atributos dos Objetos HttpRequest

    ==================  =======================================================
    Atributo            Descrição
    ==================  =======================================================
    ``path``            Uma string representando o caminho completo da página 
                        requisitada, sem incluir o domínio -- por exemplo,
                        ``"/musica/bandas/the_beatles/"``.

    ``method``          Uma string representando o método HTTP usado pelo request.
                        Precisa ser garantido que esteja em letras maiúsculas. Por
                        exemplo::

                            if request.method == 'GET':
                                faz_alguma_coisa()
                            elif request.method == 'POST':
                                faz_outra_coisa()

    ``encoding``        Uma string representando a codificação atual usada para
                        decodificar uma submissão de dados do formulário. (ou ``None``, 
                        significando que o setting ``DEFAULT_CHARSET`` está
                        sendo usado).

                        Você pode escrever esse atributo para mudar a codificação
                        usada durante o acesso aos dados do formulário. Qualquer acesso   
                        subseqüente ao atributo (assim como a leitura do ``GET`` ou
                        ``POST``) serão usados no novo valor do ``encoding``. Útil
                        se você souber se os dados do formulário não estão na codificação
                        ``DEFAULT_CHARSET``.

    ``GET``             Um objeto do tipo dicionário que contêm todos os parâmetros
                        HTTP GET dados. Veja o ``QueryDict`` na documentação posterior.

    ``POST``            Um objeto do tipo dicionário que contêm todos os parâmetros 
                        HTTP POST. Veja o ``QueryDict`` na documentação posterior.

                        É possível que uma requisição possa vir pelo POST com
                        um dicionário vazio ``POST`` -- se, digamos, o formulário foi
                        foi requisitado pelo método POST HTTP mas não inclue os 
                        dados do formulário. Portanto, não é possível usar o ``if
                        request.POST`` para checar o método POST;
                        no lugar, use ``if request.method == "POST"`` (veja
                        o item ``method`` dessa tabela).

                        Nota: ``POST`` *não* inclui informações de envio de 
                        arquivos. Veja em ``FILES``.

    ``REQUEST``         Por conveniência, um objeto do tipo dicionário busca
                        primeiro pelo ``POST``, e então pelo ``GET``. Inspirado
                        no ``$_REQUEST`` do PHP.

                        Por exemplo, se ``GET = {"nome": "john"}`` e ``POST
                        = {"idade": '34'}``, ``REQUEST["nome"]`` seria
                        ``"john"``, e ``REQUEST["idade"]`` seria ``"34"``.

                        É fortemente recomendado que você use ``GET`` e
                        ``POST`` em vez de ``REQUEST``, porque o padrão
                        é mais explícito.

    ``COOKIES``         Um dicionário padrão Python contendo todos os cookies.
                        Chaves e valores são strings. Veja o Capítulo 14 para
                        maiores informações sobre o uso dos cookies.

    ``FILES``           Um objeto do tipo dicionário que mapeia nomes de 
                        arquivos para os objetos ``UploadedFile``. Veja a
                        documentação do Django para maiores informações.

    ``META``            Um dicionário padrão Python contendo todos os cabeçalhos
                        HTTP disponíveis. Cabeçalhos disponíveis dependem do cliente
                        e dos servidor, mas aqui estão alguns exemplos:

                        * ``CONTENT_LENGTH``
                        * ``CONTENT_TYPE``
                        * ``QUERY_STRING``: A query string não analisada
                        * ``REMOTE_ADDR``: O endereço IP do cliente
                        * ``REMOTE_HOST``: O hostname do cliente
                        * ``SERVER_NAME``: O hostname do servidor
                        * ``SERVER_PORT``: A porta do servidor

                        Os cabeçalhos HTTP estão disponíveis no ``META`` como
                        chaves prefixadas com o ``HTTP_``, convertidos em
                        letras maiúsculas substituindo sublinhados por hífens.
                        Por exemplo:

                        * ``HTTP_ACCEPT_ENCODING``
                        * ``HTTP_ACCEPT_LANGUAGE``
                        * ``HTTP_HOST``: O cabeçalho HTTP ``Host`` enviado pelo cliente
                        * ``HTTP_REFERER``: A página referida, se existente
                        * ``HTTP_USER_AGENT``: A string do user-agente do cliente
                        * ``HTTP_X_BENDER``: O valor do cabeçalho ``X-Bender``, se definido

    ``user``            O objeto ``django.contrib.auth.models.User`` representando
                        o atual usuário logado. Se o usuário não estiver atualmente
                        logado, o ``user`` vai ser definido como instância do
                        ``django.contrib.auth.models.AnonymousUser``. Você pode
                        chama-los à parte com o ``is_authenticated()``, assim::

                            if request.user.is_authenticated():
                                # Faça alguma coisa com os usuários logados.
                            else:
                                # Faça alguma coisa com os usuários anônimos.

                        ``user`` está disponível apenas se sua instalação do
                        Django tiver o ``AuthenticationMiddleware`` ativado.

                        Para maiores detalhes sobre autenticação de usuários,
                        veja o Capítulo 14.

    ``session``         Um objeto do tipo dicionário de leitura e escrita,
                        representa a sessão atual. Está disponível apenas
                        se a sua instalação do Django ter a sessão support ativada.
                        Veja no Capítulo 14.

    ``raw_post_data``   O dado bruto do HTTP POST. É útil para processamento avançado.

    ==================  =======================================================

Objetos Request também possuem alguns métodos úteis, como mostra a Tabela G-2.

.. table:: Tabela G-2. Métodos HttpRequest

    ======================  ===================================================
    Método                  Descrição
    ======================  ===================================================
    ``__getitem__(key)``    Retorna o valor GET/POST para a chave dada, checando
                            primeiro o POST e depois o GET. Retorna o 
                            ``KeyError`` se a chave não existir.

                            Isso permite que você use a sintaxe de acesso ao 
                            dicionário, uma instância ``HttpRequest``.

                            Por exemplo, ``request["foo"]`` é o mesmo que
                            checar o ``request.POST["foo"]`` e então o 
                            ``request.GET["foo"]``.

    ``has_key()``           Retorna ``True`` ou ``False``, designando se o 
                            ``request.GET`` ou ``request.POST`` possuem a chave
                            dada.

    ``get_host()``          Retorna o host originado do request usando a informação
                            dos cabeçalhos ``HTTP_X_FORWARDED_HOST`` e ``HTTP_HOST``
                            (nessa ordem). Se eles não fornecerem um valor, o método 
                            usa a combinação do ``SERVER_NAME`` e ``SERVER_PORT``.

    ``get_full_path()``     Retorna o ``path``, mais o anexo da query string, se
                            aplicável. Por exemplo, ``"/musica/bandas/the_beatles/?print=true"``

    ``is_secure()``         Retorna ``True`` se o request for seguro; ou seja, se
                            ele foi feito pelo HTTPS.
    ======================  ===================================================

Objetos QueryDict
-----------------

Em um objeto ``HttpRequest``, os atributos ``GET`` e ``POST`` são
instâncias do objeto ``django.http.QueryDict``. ``QueryDict`` é uma classe
do tipo dicionário personalidazada para lidar com vários valores para a mesma
chave. Isso é necessário porque muitos elementos do formulário HTML, designado
``<select multiple="multiple">``, passa múltiplos valores para a mesma chave.

Instâncias ``QueryDict`` são imutáveis, a menos que você crie uma ``copy()`` deles.
Isso significa que você não pode mudar os atributos do ``request.POST`` e 
``request.GET`` diretamente.

O ``QueryDict`` implementa todos os métodos padrões de dicionários, porque ele é
uma subclasse do dicionário. Exceções são delineados na Tabela G-3.

.. table:: Tabela G-3. Como os QueryDicts Diferem dos Dicionários Padrões.

    ==================  =======================================================                        
    Método              Diferenças de Implementação do dicionário Padrão
    ==================  =======================================================
    ``__getitem__``     Trabalha apenas como um dicionário. Porém, se a chave
                        tiver mais de um valor, o ``__getitem()__`` retorna o 
                        último valor.

    ``__setitem__``     Define a chave dada para ``[valor]`` (uma lista Python 
                        cujo único elemento é o ``valor``). Note que isso, como
                        as outras funções de dicionários possuem o mesmo efeito,
                        pode ser chamado apenas um ``QueryDict`` imutável (um 
                        que foi criado via ``copy()``).

    ``get()``           Se a chave tiver mais de um valor, o ``get()`` retorna
                        o último valor assim como o ``__getitem__``.

    ``update()``        Toma ou um ``QueryDict`` ou um dicionário padrão. Ao
                        contrário do médoto de dicinário padrão ``update``,
                        esse método *anexa* itens no dicionário atual ao invés
                        de substituí-los::

                            >>> q = QueryDict('a=1')
                            >>> q = q.copy() # para torná-lo mutável
                            >>> q.update({'a': '2'})
                            >>> q.getlist('a')
                            ['1', '2']
                            >>> q['a'] # retorna a última
                            ['2']

    ``items()``         Tal como o método do dicionário padrão ``items()``,
                        exceto por usar o mesmo último valor lógico como o 
                        ``__getitem()__``::

                             >>> q = QueryDict('a=1&a=2&a=3')
                             >>> q.items()
                             [('a', '3')]

    ``values()``        Tal como o método do dicionário padrão ``values()``,
                        exceto por usar o mesmo último valor lógico como o
                        ``__getitem()__``.
    ==================  =======================================================

Fora isso, o ``QueryDict`` possui os métodos mostrados na Tabela G-4.

.. table:: G-4. Extra Métodos (Nondictionary) QueryDict

    ==========================  ===============================================
    Método                      Descrição
    ==========================  ===============================================
    ``copy()``                  Retorna uma cópia do objeto, usando 
                                ``copy.deepcopy()`` da biblioteca padrão Python.
                                O copy vai ser mutável -- ou seja, você pode
                                mudar os seus valores.

    ``getlist(key)``            Retorna os dados com a chave requisitada, como
                                uma lista Python. Retorna uma lista vazia se a
                                chave não existir. É garantido de retornar uma
                                lista de algum tipo.

    ``setlist(key, list_)``     Define a chave dada para ``list_`` (ao contrário
                                do ``__setitem__()``).

    ``appendlist(key, item)``   Acrescenta um item para a lista interna associada 
                                com a ``key``.

    ``setlistdefault(key, a)``  Tal como o ``setdefault``, exceto por tomar uma
                                lista de valores ao invés de um único valor.

    ``lists()``                 Tal como ``items()``, exceto por incluir todos os
                                valores, como uma lista, para cada membro do 
                                dicionário. Por exemplo::

                                    >>> q = QueryDict('a=1&a=2&a=3')
                                    >>> q.lists()
                                    [('a', ['1', '2', '3'])]

    ``urlencode()``             Retorna uma string dos dados no formato da
                                query-string (ex., ``"a=2&b=3&b=5"``).
    ==========================  ===============================================

Um Exemplo Completo
-------------------

Por exemplo, dado este formulário HTML::

    <form action="/foo/bar/" method="post">
    <input type="text" name="seu_nome" />
    <select multiple="multiple" name="bandas">
        <option value="beatles">The Beatles</option>
        <option value="who">The Who</option>
        <option value="zombies">The Zombies</option>
    </select>
    <input type="submit" />
    </form>

se o usuário entrar com ``John Smith`` no campo ``seu_nome`` e selecionar
ambos "The Beatles" e "The Zombies" na caixa de seleção múltipla, aqui está 
o que o objeto request do Django deveria ter::

    >>> request.GET
    {}
    >>> request.POST
    {'seu_nome': ['John Smith'], 'bandas': ['beatles', 'zombies']}
    >>> request.POST['seu_nome']
    'John Smith'
    >>> request.POST['bandas']
    'zombies'
    >>> request.POST.getlist('bandas')
    ['beatles', 'zombies']
    >>> request.POST.get('seu_name', 'Adrian')
    'John Smith'
    >>> request.POST.get('nonexistent_field', 'Nowhere Man')
    'Nowhere Man'

.. admonition:: Nota de Implementação:

    Os atributos ``GET``, ``POST``, ``COOKIES``, ``FILES``, ``META``, ``REQUEST``,
    ``raw_post_data``, e ``user`` são todos carregados lentamente. Isso significa
    que o Django não gasta recursos calculando os valores desses atributos até
    seu código solicitá-los.

HttpResponse
============

Em contraste com os objetos ``HttpRequest``, que são criados automaticamente 
pelo Django, os objetos ``HttpResponse`` são de sua responsabilidade. Cada
view que você escreve, é responsável por instanciar, preencher e devolver 
um ``HttpResponse``.

A classe ``HttpResponse`` fica no ``django.http.HttpResponse``.

Construção dos HttpResponses
----------------------------

Tipicamente, você irá construir um ``HttpResponse`` para passar os conteúdos
da página, como uma string, para o construtor ``HttpResponse``::

    >>> response = HttpResponse("Aqui está o texto da página Web.")
    >>> response = HttpResponse("Só textos, por favor.", mimetype="text/plain")

Mas se você quizer adicionar conteúdo de forma incremental, você pode usar o 
``response`` como um objeto filelike::

    >>> response = HttpResponse()
    >>> response.write("<p>"Aqui está o texto da página Web.</p>")
    >>> response.write("<p>Aqui está outro parágrafo.</p>")

Você pode passar o ``HttpResponse`` como um iterador, ao invés de
passar como strings codificadas. Se você usar essa técnica, siga
as orientações::

* O iterador deve retornar strings.

* Se um ``HttpResponse`` for inicializado com um iterador como seu
conteúdo, você não pode usar a instância ``HttpResponse`` como um
objeto filelike. Fazendo isso irá levantar uma ``Exception``.

Finalmente, note que o ``HttpResponse`` implementa um método ``write()``,
que o torna adequado para usar em qualquer lugar onde o Python espera um
objeto filelike. Veja o Capítulo 8 para alguns exemplos do uso dessa técnica.

Definindo Cabeçalhos
--------------------

Você pode adicionar ou deletar cabeçalhos usando a sintaxe de dicionário::

    >>> response = HttpResponse()
    >>> response['X-DJANGO'] = "É o melhor."
    >>> del response['X-PHP']
    >>> response['X-DJANGO']
    "É o melhor."

Você pode também usar o ``has_header(header)`` para checar a existência de um
cabeçalho.

Evite definir cabeçalhos ``Cookie`` manualmente; ao invés disso, olhe o 
Capítulo 14 para instruções de como os cookies funcionam no Django.

HttpResponse Subclasses
-----------------------

O Django inclue um número de subclasses ``HttpResponse`` que lidam com diferentes
tipos de respostas HTTP (olhe a Tabela G-5). Tal como ``HttpResponse``, essas
subclasses ficam no ``django.http``.

.. table:: Tabela G-5. HttpResponse Subclasses

    ==================================  =======================================
    Classe                              Descrição
    ==================================  =======================================
    ``HttpResponseRedirect``            O construtor toma um único argumento:
                                        o caminho para ser redirecionado. Isso
                                        pode ser uma URL totalmente qualificada
                                        (ex., ``'http://search.yahoo.com/'``) ou 
                                        uma URL absoluta com o domínio (ex.,
                                        ``'/search/'``). Note que isso retorna
                                        um código de status HTTP 302.

    ``HttpResponsePermanentRedirect``   Tal como ``HttpResponseRedirect``, mas 
                                        ele retorna um redirecionamento permantente
                                        (código de status HTTP 301) no lugar de
                                        redireção "found" (código de status 302).

    ``HttpResponseNotModified``         O construtor não toma nenhum argumento.
                                        Use isso para designar que a página não
                                        tem sido modificada desde a última 
                                        requisição do usuário.

    ``HttpResponseBadRequest``          Age como o ``HttpResponse`` mas usa o
                                        código de status 400.

    ``HttpResponseNotFound``            Age como o ``HttpResponse`` as usa o 
                                        código de status 404.

    ``HttpResponseForbidden``           Age como o ``HttpResponse`` mas usa o 
                                        código de status 403.

    ``HttpResponseNotAllowed``          Tal como o ``HttpResponse``, mas usa o 
                                        código de status 405. Ele pega um único
                                        argumento requerido: uma lista de métodos
                                        permitidos (ex., ``['GET', 'POST']``).

    ``HttpResponseGone``                Age como o ``HttpResponse`` mas usa o 
                                        código de status 410.    

    ``HttpResponseServerError``         Age como o ``HttpResponse`` mas usa o 
                                        código de status 500.
    ==================================  =======================================

Você pode, é claro, definir sua própria subclasse ``HttpResponse`` para
suportar diferentes tipos de respostas não suportadas fora da caixa.

Retornando Erros
----------------

Retornar códigos de erros HTTP no Django é fácil. Nós já mencionamos o 
``HttpResponseNotFound``, ``HttpResponseForbidden``,
``HttpResponseServerError``, e outras subclasses. Apenas retorna uma
instância de uma das subclasses ao invés de um ``HttpResponse`` normal
para representar um erro, por exemplo::

    def meu_view(request):
        # ...
        if foo:
            return HttpResponseNotFound('<h1>Página não encontrada.</h1>')
        else:
            return HttpResponse('<h1>Página encontrada.</h1>')

Porque o erro 404 é de longe o erro HTTP mais comum, há uma maneira mais fácil
de lidar com isso.

Quando você retornar um erro como o ``HttpResponseNotFound``, você será responsável
por definir o HTML da página de erro resultante::

    return HttpResponseNotFound('<h1>Página não encontrada</h1>')

Por conveniência, e porque é uma boa idéia ter uma página de erro 404 em seu site,
o Django fornece uma excessão ``Http404``. Se você levantar o ``Http404`` em qualquer
ponto da sua função view, o Django irá pegá-lo e retornar uma página de erro para sua
aplicação, juntamente com o código de erro 404.

Aqui está um exemplo::

    from django.http import Http404

    def detalhe(request, pesquisa_id):
        try:
            p = Pesquisa.objects.get(pk=pesquisa_id)
        except Pesquisa.DoesNotExist:
            raise Http404
        return render(request, 'pesquisas/detalhes.html', {'pesquisa': p})

Para usar a exceção ``Http404`` em sua totalidade, você deve criar um template
que é exibido quando um erro 404 for levantado. Esse template deve ser chamado
``404.html``, e não pode ficar no nível mais alto do template tree.

Customizando o 404 (Not Found) View
------------------------------------

Quando você levanta uma exceção ``Http404``, o Django carrega uma view especial
dedicado para lidar erros 404. Por padrão, é o view
``django.views.defaults.page_not_found``, que carrega e renderiza o template
``404.html``.

Isso significa que você precisa definir o template ``404.html`` no diretório 
de template raíz. O template será usado para todos os erros 404.

Essa view ``page_not_found`` deve ser suficiente para 99% das aplicações Web, mas
se você quizer substituir a view 404, você pode especificar o ``handler404`` no
seu URLconf, assim::

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        ...
    )

    handler404 = 'meusite.views.minha_view_404_personalizada'

Nos bastidores, o Django determina a view 404 procurando pelo
``handler404``. Por padrão, o URLconfs contêm a seguinte linha::

    from django.conf.urls.defaults import *

Que cuida da criação do ``handler404`` no módulo atual. Como você pode ver
no ``django/conf/urls/defaults.py``, ``handler404`` está definido para
``'django.views.defaults.page_not_found'`` por padrão.

Há três coisas a notar sobre as views 404:

* A view 404 é também chamada se o Django não achar uma correspondência depois
de checar todas as expressões regulares no URLconf.

* Se você não definir sua própria view 404 -- e simplesmente usar o padrão,
que é recomendado -- você ainda tem uma obrigação: criar um template 
``404.html`` no seu diretório de template raíz. Por padrão a view 404 usará 
esse template para todos os erros 404.

* Se o ``DEBUG`` está definido como ``True`` (no seu módulo settings), então
a sua view 404 nunca será usada, em vez disso, o traceback será exibido.

Customizando o 500 (Server Error) View
---------------------------------------

Da mesma forma, o Django executa um caso especial de comportamente no caso de 
um erro runtime no código view. Se a view resultar em uma exceção, o Django irá,
por padrão, chamar a view ``django.views.defaults.server_error``, que carrega e
renderiza o template ``500.html``.

Isso significa que você precisa definir o template ``500.html`` no seu diretório
de template raíz. Esse template será usado para todos os erros do servidor.

Essa view ``server_error`` deve ser suficiente para 99% das aplicações Web, mas
se você quizer substituir a view, você pode especificar o ``handler500`` no seu
URLconf, assim::

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        ...
    )

    handler500 = 'meusite.views.minha_view_personalizada_de_erro'
