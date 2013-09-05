============================
Capítulo 3: Views e URLconfs
============================

No capítulo anterior foi explicado como montar um projeto Django e executar o
servidor de desenvolvimento do Django. Neste capítulo serão vistos os conceitos
básicos de como criar páginas dinâmicas com o Django.

Primeira página em Django: Hello World
======================================

Como primeiro objetivo iremos criar uma página que tem como saída a
famosa mensagem de exemplo: "Hello world."

Se estivesse publicando uma simples página Web sem um framework Web, bastaria digitar
"Hello World" em um arquivo texto, chamá-lo de ``hello.html`` e fazer upload do mesmo
para um diretório em um servidor Web em algum lugar. Note que nesse processo foram
especificadas duas peças chave de informação sobre a página: o conteúdo (a string 
``"Hello world"``) e sua URL (``http:://www.exemplo.com/hello.html``, ou talvez 
``http:://www.exemplo.com/files/hello.html`` caso tenha posto em um subdiretório)

Com Django, as mesmas coisas são especificadas, porém de modos diferentes. Os
conteúdos da página são produzidos por uma *função view (visão)*, e a URL é
especificada em um *URLconf*. Primeiro será escrita a função view "Hello World".

Primeira view
-------------

Dentro do diretório ``meusite`` que o ``django-admin.py startproject`` criou no
último capítulo, crie um arquivo vazio chamado ``views.py``. Esse módulo Python conterá
as views vistas neste capítulo. Note que não há nada especial sobre o nome ``views.py``
-- o Django não se preocupa como o arquivo é chamado, como será visto mais para
frente -- mas é uma boa ideia chamá-lo de ``views.py`` por ser uma convenção, para o
benefício de outros desenvolvedores que irão ler seu código.

A view "Hello world" é simples. Aqui está toda a função, incluindo os imports, que
deverá ser digitada no arquivo ``views.py``::

    from django.http import HttpResponse

    def hello(request):
        return HttpResponse("Hello world")

Vamos analisar esse código linha-por-linha:

* Primeiro, é importada a classe ``HttpResponse``, que está localizada no módulo
  ``django.http``. É necessário importá-la pois é usada no código.

* Depois é definida uma função chamada ``hello`` -- a função view.

  Cada função view tem pelo menos um parametro chamado ``request``, por
  convenção. Este é um objeto que contém informação sobre a requisição Web 
  atual que ativou a view, e é uma instância da classe 
  ``django.http.HttpRequest``. Nesse exemplo não é feito nada com ``request``,
  mas de qualquer modo deve ser o primeiro parametro da view.

  Note que o nome da função view não importa, não é necessário nomeá-la de
  uma maneira específica para que o django a reconheça. É chamanda de
  ``hello``, pois esse nome indica de forma clara a essência da view, mas nada
  impede de que fosse chamada de ``hello_wonderful_beautiful_world``, ou algo
  do tipo. Na próxima seção, "Primeiro URLconf", será explicado como o Django
  encontra essa função.

* A função só contém uma linha: simplesmente retorna um objeto ``HttpResponse``
  que foi instanciado com o texto ``Hello World``.

A lição principal aqui é a seguinte: a view é somente uma função Python que
recebe um ``HttpRequest`` como primeiro argumento e retorna uma instância de
``HttpRequest``. Para que uma função Python seja uma view Django é necessário
que faça essas duas coisas. (Existem exceções, mas chegaremos nelas depois.)

Seu Primeiro URLconf
--------------------

Se, neste ponto, for executado ``python manage.py runserver`` de novo, ainda será
exibida a mensagem "Welcome to Django", sem nenhum traço da view "Hello World".
Isso acontece, pois o projeto ``meusite`` ainda não sabe da view ``hello``; é
preciso mostrar explicitamente ao Django que essa view está sendo ativada em uma
URL particular. (Continuando a analogia anterior de publicar arquivos HTML estáticos,
neste ponto é criado o arquivo HTML, mas ainda não foi feito upload dele no servidor
web) Para ligar uma função view para uma URL com o Django, é utilizado um URLconf.

Um *URLconf* é como um índice remissivo para sua página Django. Basicamente é um
mapeamento entre URLs e funções view que devem ser chamadas para essas URLs. É como
dizer ao Django: "Para esta URL, chame este código, e para aquela URL, chame aquele
código". Por exemplo: "Quando alguém visita a URL ``/foo/``, a função view ``foo_view()``
é chamada, que está no arquivo Python ``views.py``."

Quando ``django-admin startproject`` foi executado no capítulo anterior, um URLconf 
foi criado automaticamente: o arquivo ``urls.py``. Por padrão ele parece com
algo assim::

    from django.conf.urls import patterns, include, url

    # Uncomment the next two lines to enable the admin:
    # from django.contrib import admin
    # admin.autodiscover()

    urlpatterns = patterns('',
        # Examples:
        # url(r'^$', 'mysite.views.home', name='home'),
        # url(r'^mysite/', include('mysite.foo.urls')),

        # Uncomment the admin/doc line below to enable admin documentation:
        # url(r'^admin/doc/', include('django.contrib.admindocs.urls')),

        # Uncomment the next line to enable the admin:
        # url(r'^admin/', include(admin.site.urls)),
    )

Este URLconf padrão inclui algumas funcionalidades comumente usadas no Django dentro
de comentários, para que ativá-las seja tão fácil quanto descomentar as linhas certas.
Se os códigos comentados forem ignorados, aqui está a essência de um URLconf::

    from django.conf.urls.defaults import patterns, include, url

    urlpatterns = patterns('',
    )

Vamos analisar linha-por-linha deste código:

* A primeira linha importa três funções do módulo ``django.conf.urls.defaults``,
  que são a infraestrutura do URLconf do Django: ``patterns``, ``include`` e
  ``urls``.

* A segunda linha chama a função ``patterns`` e salva o resultado em uma
  variável chamada ``urlpatterns``. A função ``patterns`` recebe um único
  argumento: uma string vazia. (A string pode ser usada para prover um
  prefixo comum para funções view, que será visto no :doc:`chapter08`.)

O que deve-se notar aqui é a váriavel ``urlpatterns``, que o Django espera encontrar
em seu módulo URLconf. Essa variável define o mapeamento entre URLs e o código que
manipula essas URLs. Por padrão, como pode-se ver, o URLconf está vazio -- sua
aplicação Django está vazia. (Como uma nota, é desse jeito que o Django mostra 
a página "Welcome to Django" vista no último capítulo. Se um URLconf está vazio,
o Django assume que um novo projeto foi iniciado e, portanto, mostra essa mensagem.)

Para adicionar uma URL e view ao URLconf, só é necessário adicionar um mapeamento
entre um URLpattern e a função view. Aqui mostra-se como fazer a ligação na nossa
view ``hello``::

    from django.conf.urls.defaults import patterns, include, url
    from mysite.views import hello

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
    )

(Note que os códigos comentados foram removidos para poupar espaço. É possível
deixar as linhas se quiser.)

Foram feitas duas mudanças aqui:

* Primeiro,  a view ``hello`` foi importada de seu módulo ``mysite/views.py``,
  que é acessado como ``mysite.views`` na sintaxe de importação do Python.
  (Isso assume que ``mysite/views.py`` está no seu diretório Python; veja a 
  barra lateral para detalhes.)

* Depois é adicionada a linha ``url(r'^ĥello/$'),`` à variável ``urlpatterns``. 
  Essa linha é denominada *URLpattern*. A função ``url()`` avisa ao Django como 
  lidar com a url que está sendo configurado. O primeiro argumento é uma string
  de correspondência de padrão (pattern-matching, uma expressão regular; será
  explicado melhor mais a frente) e o segundo argumento é a função view para 
  aquele padrão. ``url()`` pode levar outros argumentos, que serão vistos mais
  profundamente no :doc:`chapter08`.

.. nota::

  Um detalhe importante que foi introduzido é o caracter ``r`` no começo da string
  de expressão regular. Ele avisa o Python que a string é uma "raw string" -- seu
  conteúdo não deve interpretar contrabarras -- como na string ``'\n'``, que é uma
  string com um único caracter contendo uma nova linha. Quando ``r`` é adicionado,
  o Python não faz a substituição, então ``r'\n'`` é uma string composta de dois
  caracteres contendo uma contrabarra e um "n". Há uma colisão natural entre o uso
  de contrabarras pelo Python e as contrabarras utilizadas em expressões regulares,
  então é fortemente recomendado utilizar raw strings sempre que for definir
  expressões regulares em Python. Todos os URLpatterns neste livro serão raw strings.

Em poucas palavras, foi instruído ao Django que qualquer solicitação feita a URL
``/hello/`` deve ser manuseada pela função view ``hello``.

.. admonition:: Diretório Python

    O  *diretório Python* é a lista de diretório do sistema que o Python
    procura quando a instrução ``import`` é utilizada.

    Por exemplo, supondo que o diretório Python está em ``['', '
    '/usr/lib/python2.7/site-packages', '/home/username/djcode']``. Caso a instrução
    ``from foo import bar`` seja executada, o Python procurará um módulo chamado
    ``foo.py`` no diretório atual. (A primeira ocorrência no diretório Python, uma
    string vazia, significa "o diretório atual.") Se o arquivo não existir, o Python
    procurará pelo arquivo ``/usr/lib/python2.7/site-packages/foo.py``. Se esse
    arquivo também não existir, o Python tentará procurar pelo arquivo
    ``/home/username/djcode/foo.py``. Por fim, se *aquele* arquivo não existir, será
    levantado um ``ImportError``.

    Para descobrir seu diretório Python, inicie o interpretador interativo Python
    e digite isso::

        >>> import sys
        >>> print sys.path

    Geralmente não é necessário se preocupar com o diretório Python -- o Python e
    Django tomam conta disso automaticamente nos bastidores. (Configurar o diretório
    Python é uma das tarefas que o script ``manage.py`` faz.)

Vale a pena discutir a sintaxe desse URLpattern, pois pode não parecer óbvio a 
primeira vista. Apesar de querer encontrar a URL ``/hello/``, o padrão é um pouco
diferente disso. Aqui está o por quê:

* O Django remove a barra do começo de cada URL antes de checar os 
  URLpatterns. Isto significa que o URLpattern visto no capítulo não
  possui uma barra no começo em ``/hello/``. (A princípio isso pode
  parecer nada intuitivo, mas isso simplifica as coisas, como a inclusão
  de URLconfs dentro de outros URLconfs, que será visto no Capítulo 8.)

* O padrão inclui um acento circunflexo (``^``) e um sifrão (``$``). Esses
  são caracteres que possuem signficado especial em expressões regulares:
  o acento circunflexo signfica "o padrão deve casar com o começo da string"
  e o sifrão significa "o padrão deve casar com o fim da string."

  Esse conceito é melhor explicado através de um exemplo. Se ao invés de
  usar o padrão ``'^hello/'`` (sem o sifrão no final), então *qualquer* URL
  começada por ``/hello/`` seria casa, como ``/hello/foo`` e ``/hello/bar``,
  e não somente ``/hello/``. Semelhantemente, se o acento circunflexo fosse
  retirado (ou seja, ``'hello/$'``), o Django casaria *qualquer* URL
  terminada em ``hello/``, como ``/foo/bar/hello/``. Se fosse usado ``hello/``,
  sem acento circunflexo *ou* sifrão, então qualquer URL que contesse
  ``hello/`` seria casada, como ``/foo/hello/bar``. Desse modo usamos tanto
  o acento circunflexo quanto o sifão para garantir que somente a URL
  ``/hello/`` seja casada -- nada mais, nada menos.

  A maioria dos URLpatterns começarão com acentos circunflexos e terminarão
  com sifrão, mas é interessante ter a flexibilidade de poder casar de formas
  mais sofisticadas.

  Pode estar se perguntando o que acontece se a URL ``/hello`` (isto é, *sem*
  a barra no final) for requisitada. Essa URL *não* casaria, pois o URLpattern
  exige uma barra no fim. Entretanto, por padrão, qualquer requisição feita a
  uma URL que *não* casa com a URLpattern e *não* termina com uma barra é
  redirecionada para a mesma URL com uma barra no final. (Isso é controlado
  pela configuração ``APPEND_SLASH`` do Django, que é explicada no
  Apêndice D.)

  Se optar por utilizar URLs terminadas por barra, tudo o que precisa fazer
  é adicionar barras em cada URLpattern e atribuir ``True`` ao
  ``APPEND_SLASH``. Caso opte por *não* deixar barras no fim das URLs, ou 
  quiser escolher dependendo da URL, deixe o ``APEEND_SLASH`` como ``False``
  e colocar barras no fim de cada URLpattern que desejar.

Outro ponto importante sobre esse URLconf é que passamos a função view
``hello`` como um objeto sem chamar a função. Essa é uma das características
chaves do Python (e outras linguagens dinâmicas): funções são objetos de
primeira classe, o que quer dizer que é possível utilizá-las como quaisquer
outras variáveis. Maneiro, não?

Para testar as mudanças feitas no URLconf, inicie o servidor de desenvolvimento
do Django, como visto Capítulo 2, executando o comando ``python manage.py runserver``.
(Se já estiver rodando, não tem problema. O servidor detecta mudanças feitas no
código automaticamente e as recarrega, para não ser preciso reiniciar o servidor quando
houver mudança.) O servidor está rodando no endereço ``http://127.0.0.1:8000/``, então
abra o navegador e vá para ``http://127.0.0.1:8000/hello/``. Deve aparecer o texto
"Hello world" -- a saída da sua view Django.

Eba! Você fez sua primeira página Django.

.. admonition:: Expressões regulares

    *Expressões regulares* (ou *regexes*) são uma forma compacta de especificar
    padrões em um texto. Enquanto URLconfs do Django possibilitam o uso de
    qualquer regex para casas URLs, só serão utilizados alguns símbolos de
    regex na prática. Aqui estão alguns símbolos mais utilizados:

    ============  ==========================================================
    Símbolo       Casa com
    ============  ==========================================================
    ``.`` (dot)   Qualquer (um) caracter

    ``\d``        Qualquer (um) digito

    ``[A-Z]``     Qualquer caracter entre ``A`` e ``Z`` (caixa alta)

    ``[a-z]``     Qualquer caracter entre ``a`` e ``z`` (caixa baixa)

    ``[A-Za-z]``  Qualquer caracter entre ``a`` e ``z`` (caixa baixa e alta)

    ``+``         Um ou mais da expressão anterior (por ex., ``\d+`` casa um
                  ou mais digitos.

    ``[^/]+``     Um ou mais caracteres até (e não incluindo) uma barra

    ``?``         Zero ou um da expressão anterior (por ex., ``\d?`` casa
                  zero ou mais digitos

    ``*``         Zero or mais da expressão anterior (por ex., ``\d*`` casa
                  zero, um ou mais que um dígito

    ``{1,3}``     Entre um e três (incluso) da expressão anterior (por ex/,
                  ``\d{1,3}`` casa um, dois ou três digitos)
    ============  ==========================================================

    Para mais sobre expressões regulares, veja http://turing.com.br/material/regex/python_re.html

Uma nota rápida sobre erros 404
-------------------------------

Neste ponto, o URLconf define somente um URLpattern, o que lida com as
requisições da URL ``/hello/``. O que acontece quando é feito uma requisição
para outra URL?

Para descobrir, rode o servidor de desenvolvimento do Django e visite alguma
página como ``http://127.0.0.1:8000/goodbye/`` ou
``http://127.0.0.1:8000/hello/subdirectory/``, ou até ``http://127.0.0.1:8000/``
(a "raiz" do site). Deverá aparecer a mensagem "Page not found" (ver
Figura 3-1). O Django exibe essa mensagem, pois você requisitou uma URL que
não está especificada em seu URLconf.

.. figure:: graphics/chapter03/404.png
   :alt: Screenshot da página 404 do Django.

   Figura 3-1. Página 404 di Django

A função dessa página vai além de exibir a mensagem de erro 404. Ela também
mostra qual URLconf exatamente o Django usou para cada padrão nesse URLconf.
Com essa informação é possível saber por quê a URL requisitada lançou um
erro 404.

Naturalmente, essa informação sensível é só para você, o desenvolvedor Web.
Se esse fosse um site em produção disponível na Internet, não seria bom
exibir essa informação para o público. Por esse motivo, essa página "Page
not found" é exibida somente se seu projeto Django está no *modo debug*.
Mais para frente será explicado como desativar esse modo. Por enquanto, saiba
que cada projeto Django está no modo debug quando é criado pela primeira vez.
Se o projeto não esá no modo debug, o Django mostra outra mensagem 404.

Uma nota rápida sobre a raiz do site
------------------------------------

Como explicado na última seção, você verá uma mensagem de erro ao acessar a
raiz do site -- ``http://127.0.0.1:8000``. O Django não adiciona nada magicamente
à raiz do site; essa URL não é tratada de modo especial. O desenvolvedor deve
atribuir um URLpatternt a ela, como todo outro acesso em seu URLconf.

O URLpattern que casa com a raiz do site é um pouco não intuitivo, então será
explicado aqui. Quando for implementar a view da raiz do site, use a URLpattern
``'^$'``, que casa uma string vazia. Por exemplo::

    from mysite.views import hello, my_homepage_view

    urlpatterns = patterns('',
        url(r'^$', my_homepage_view),
        # ...
    )

Como o Django processa uma requisição
=====================================

Antes de continuar para nossa segunda função view, vamos parar e aprender um
pouco sobre como o Django funciona. Especificamente, quando é exibida a mensagem
"Hello world" ao visitar ``http://127.0.0.1:8000/hello/`` no seu navegador, o que
o Django faz por trás?

Tudo começa com o *arquivo de configurações*. Quando é executado ``python
manage.py runser``, o script procura por um arquivo chamado ``settings.py``,
dentro do site ``mysite``. Esse arquivo contem todo o tipo de configuração
para esse projeto Django, como (tudo em caixa alta): ``TEMPLATE_DIRS``,
``DATABASES`` e etc. A configuração mais importante é chamada de
``ROOT_URLCONF``. Essa configuração avisa ao Django qual módulo deve ser usado
como URLconf para esse site.

Lembra quando ``django-admin.py startproject`` criou os arquivos
``settings.py`` e ``urls.py``? O ``settings.py`` gerado automaticamente contém um
configuração ``ROOT_URLCONF`` que aponto para o ``urls.py`` gerado automaticamente.
Abra o arquivo ``settings.py`` e veja por si mesmo; deve parecer com algo assim::

    ROOT_URLCONF = 'mysite.urls'

Isso corresponde ao arquivo ``mysite/urls.py``.

Quando uma requisição é feita para uma URL em particular, por exemplo uma
requisição para ``/hello/``, o Django carrega o URLconf apontado pelo
``ROOT_URLCONF``. Então checa cada URLpattern naquele URLconf, a fim de comparar
a URL requisitada com os padrões, um por um, até encontrar um que case.
Quando um padrão é casado, é chamada a função view associada com esse padrão,
passando-a como um objeto ``HttpRequest`` como primeiro parametro. (O objeto
``HttpRequest`` será explicado melhor mais para a frente.)

Como visto no primeiro exemplo, uma função view deve retornar um
``HttpResponse``. Uma vez que isso é feito o Django lida com o resto, convertendo
o objeto Python para uma requisição web com os cabeçalhos HTTP e corpo (ou seja,
conteúdo da página) apropriados.

Resumindo:

1. Uma requisição chega a ``/hello/``.
2. Django determina o URLconf raiz analisando a configuração ``ROOT_URLCONF``.
3. Django examina todas as URLpatterns no URLconf até encontrar a primeira
   que case com ``/hello/``.
4. Se for casado, é chamado a função view associada.
5. A função view retorna um ``HttpResponse``.
6. Django converte o ``HttpResponse`` para uma resposta HTTP, que resulta em
   uma página Web.

Agora você sabe o básico de como fazer páginas em Django. É realmente simples,
só é necessário escrever funções view e mapeá-las a URLs através de URLconfs.

A segunda view: conteúdo dinânico
=================================

Nossa view "Hello world" serviu para demonstrar o básico de como o Django
funciona, mas não foi um exemplo de uma página *dinâmica*, pois o conteúdo
da página é sempre o mesmo. Cada vez que ``/hello`` é exibida, será mostrada
a mesma coisa; na verdade, poderia ser um arquivo HTML estático.

Para nossa segunda view, mas criar algo mais dinâmico -- uma página que exibe
a data e hora atuais. Esse exemplo é um bom e, simples, próximo passo, pois
não envolve banco de dados ou entrada de usuário, somente a saída do relógio
interno do servidor. Só é um pouco mais interessante que o "Hello world", mas
demonstrará alguns conceitos.

Esta view precisa fazer duas coisas: calcular a hora e data atuais, e retornar
um ``HttpResponse`` contendo esse valor. Se possui experiência com Python,
deve saber que há um módulo Python chamado ``datetime`` para calcular datas.
Aqui mostra como usá-lo::

    >>> import datetime
    >>> now = datetime.datetime.now()
    >>> now
    datetime.datetime(2008, 12, 13, 14, 9, 39, 2731)
    >>> print now
    2008-12-13 14:09:39.002731

Isso é simples o bastante e não tem nada a ver com o Django. É só código Python.
(Queremos enfatizar quais códigos são "só Python" vs código específico do Django.
Conforme for aprendendo Django, queremos ensiná-lo Python, para que possa usar
o mesmo para outros projetos que não envolvem Django).

Para o Django fazer com que a view mostre a data e hora atuais, só é necessário
colocar a instrução ``datetime.datetime.now()`` em uma view e retornar um
``HttpResponse``. Fica assim::

    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It's now %s.</body></html>" % now
        return HttpResponse(html)

Como na função view ``hello``, esse código deve estar em ``views.py``. Note que
não mostramos a função ``hello`` neste exemplo por brevidade, mas por completude,
aqui está como todo o ``views.py`` fica::

    from django.http import HttpResponse
    import datetime

    def hello(request):
        return HttpResponse("Hello world")

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It's now %s.</body></html>" % now
        return HttpResponse(html)

(De agora em diante não mostraremos código anterior em exemplos, exceto quando
necessário. Você deve saber diferenciar a partir do contexto quais partes do
exemplo são novas das antigas).

Vamos passar pelas mudanças que fizemos ao ``views.py`` para acomodar a view
``current_datetime``.

* Adicionamos um ``import datetime`` na parte superior do módulo, para
  calcular datas.

* O nova função ``current_datetime`` calcula a data e hora atuais, com um
  objeto ``datetime.datetime``, e a armazena como a variável local ``now``.

* A segunda linha de código dentro da view constrói uma resosta HTML usando
  funcionalidade do Python de formata strings. O ``%s`` na string é um
  placeholder, e o símbolo de porcentagem após a string significa
  "Substitua o ``%s`` da string com o valor da variável ``now``". A variável
  ``now`` é, tecnicamente, um objeto ``datetime.datetime``, não uma string,
  mas o caracter de formatação ``%s`` faz a conversão para sua representação
  em string, que é algo do tipo ``"2008-12-13 14:09:39.002731"``. Isso
  resultará em uma string HTML como
  ``"<html><body>It is now 2008-12-13 14:09:39.002731.</body></html>"``.

  (Sim, esse código HTML é inválido, pois tentamos manter os exemplos
  simples e curtos)

* Por fim, a view retorna um objeto ``HttpResponse`` que contém a resposta,
  assim como fizemos na view ``hello``.

Após essa adição ao ``views.py``, adicione o URLpatter ao ``urls.py`` para
avisar o Django qual URL deve acessar essa view. Algo como ``/time/`` é um
bom nome::

    from django.conf.urls.defaults import patterns, include, url
    from mysite.views import hello, current_datetime

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
    )

Foram feitas duas mudanças. Primeiro importamos a função ``current_datetime``
no começo do código. Depois, e mais importante, adicionamos a URLpattern que
mapeia a URL ``/time/`` àquela view. Está começando a entender como funciona?

Com a view escrita e o URLconf atualizado, execute o ``runserver`` e visite
``http://127.0.0.1:8000/time/`` no seu navegador. Deverá ser exibidas a data
e hora atuais.

.. admonition:: Fuso horário do Django

    Dependendo do seu computador, a data e hora atuais pode estar errado.
    Isso acontece pois o Django usa o fuso horário ``America/Chicago`` como
    padrão. (Algum tem de ser o padrão e esse é o fuso horário que os
    desenvolvedores originais do Django vivem) Se você vive em outro lugar,
    você pode mudá-lo no arquivo ``settings.py``. Leia os comentários nesse
    arquivo para um link atualizado contendo a lista de todas os fuso horário
    do globo.

URLconfs e acoplamento fraco
============================

Agora é uma boa hora para ressaltar a filosofia por trás dos URLconfs e do Django
em geral: o princípio de *acoplamento fraco*. De for simples, acoplamento fraco
é uma abordagem em desenvolvimento de software que valoriza a importância de fazer
partes intercambiáveis. Se duas partes de código são fracamente acopladas, então
mudanças feitas em uma das partes terá pouco ou nenhum efeito a outra.

URLconfs do Django são um bom exemplo desse princípio na prática. Em uma aplicação
web em Django, as definições de URLs e funções views são fracamente acopladas,
ou seja, a decisão de qual URL chamará qual função e a implementação em si da função,
ficam em lugares separados. Isso permite mudar uma parte sem afetar a outra.

Por exemplo, considere a função ``current_datetime`` feita recentemente. Para mudar
a URL da aplicação de ``/time/`` para ``current-time``, é necessário somente mudar
o URLconf, sem ter de se preocupar com a view em si. Do mesmo modo, para mudar
a função view, alterando sua lógico de algum jeito, poderíamos fazer isso sem
afetar a URL com que a função está ligada.

Além disso, para mostrar a funcionalidade de mostrar data e hora locais para
*várias* URLs, tudo o que levaria seria editar o URLconf, sem ter que alterar
o código da view. Neste exemplo, nosso ``current_datetime`` está disponível para
duas URLs. É um exemplo não muito real, mas essa técnica pode ser útil::

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
        url(r'^outra-pagina-de-hora/$', current_datetime),
    )

URLconfs e view são fracamente acopladas na práticas. Mais exemplos dessa
importante filosofia serão dados ao longo deste livro.

A terceira view: URLs dinâmicas
===============================

Na view ``current_datetime`` o conteúdo da página, a data e hora atuais,
eram dinâmicos, mas a URL (``/time``) era estática. Entretanto, na maioria das
aplicações Web dinâmicas, há parâmetros na URL que influenciam na saída da
página. Por exemplo, uma loja de livros online pode dar a cada livro uma URL,
como ``/livros/243/`` e ``/livros/81196/``.

Criaremos uma terceia view que exibe a data e hora atuais deslocados por um
certo número de horas. O objeto é fazer um site em que a página ``/hora/mais/1``
mostre a data/hora de uma hora no futuro, a página ``/hora/mais/2`` mostre a
data/hora de duas horas no futuro, a página ``/hora/mais/3`` mostre a data/hora
de três horas no futuro, e assim por diante.

Um novato pode pensar em criar uma função view para cada deslocamento de hora,
que resultaria em um URLconf parecido com este::

    urlpatterns = patterns('',
        url(r'^hora/$', current_datetime),
        url(r'^hora/mais/1/$', uma_hora_a_mais),
        url(r'^hora/mais/2/$', duas_horas_a_mais),
        url(r'^hora/mais/3/$', tres_horas_a_mais),
        url(r'^hora/mais/4/$', quatro_horas_a_mais),
    )

Claramente essa linha de pensamento é falha. Não somente isso resultaria em
funções view redundantes, mas a aplicação seria limitada por um número pré-definido
de horas -- um, dois, três ou quatro horas. Para criar uma página de *cinco* horas
no futuro, seria necessário criar uma view separada e adicionar uma linha no
URLconf, aumentando a duplicação. É necessário abstrair um pouco.

.. admonition:: Uma palavra sobre URLs bonitas

    Se você possui experiência em outra plataforma de desenvolvimento web,
    como PHP ou Java, você deve estar pensando: "Ei, vamos utilizar um query
    string parameter!", algo como ``/hora/mais?horas=3``, em que as horas seriam
    atribuídos a partir do parametro ``horas`` na query string da URL (a parte
    após o ``?``).

    É *possível* fazer isso com Django (é explicado como no Capitulo 7), mas
    uma das filosofias do Django é que a URL deve ser linda. A URL
    ``/time/plus/3/`` é muito mais limpa, simples, legível, fácil de dizer alto
    para alguém e... simplesmente mais bonita que a query string equivalente.
    URLs bonitas são uma carecterística de aplicações Web de qualidade.

    O sistema URLconf do Django encoraja o uso de URLs bonitas ao fazer mais
    simples usar URLbonitas do que *não* usar.

Então como projetamos nossa aplicação para lidar com deslocamentos de horas
arbitrários? A chave é utilizar *URLpatterns curingas*. Como citado
anteriormente, uma URLpatterns é uma expressão regular; por isso, podemos
utilizar o padrão de expressões regulares ``\d+`` para casar um ou mais digitos::

    urlpatterns = patterns('',
        # ...
        url(r'^hora/mais/\d+/$', horas_adiante),
        # ...
    )

(Estamos utilizando o ``# ...`` para sugerir que podem haver outros URLpatterns
que foram retirados desde exemplo).

Esse novo URLpattern casa qualquer URL parecida com ``/hora/mais/2``,
``/hora/mais/25``, ou até ``/time/mais/100000000000``. Pensando nisso, vamos
limitar o deslocamento máximo para 99 horas. Isso significa que permitiremos
números com um ou dois digitos, na sintaxe de expressões regulares isso se
traduz em ``\d{1,2}``::

    url(r'^hora/mais/\d{1,2}/$', horas_adiante),

.. nota::

    Ao construir aplicações Web, é sempre importante considerar as entradas
    mais estranhas e decidir se a aplicação deve suportar essa entrada.
    As entradas estranhas foram removidas ao limitar o deslocamento para 99
    horas.

Agora que designamos um curinga para a URL, precisamos passar o dado desse
curinga para qualquer deslocamento de hora. Isso é feito colocando o parênteses
em volta do dado no URLpattern que queremos salvar. No caso do nosso exemplo,
queremos salvar qualquer número digitado na URL, então colocamos os parênteses
em volta do ``\d{1,2}``, assim::

    url(r'^hora/mais/(\d{1,2})/$', horas_adiante),

Se você conhece expressões regulares, deve estar se sentindo em casa. Estamos
usando parênteses para *capturar* dados do texto casado.

A URLconf final, incluindo as duas views anteriores, fica assim::

    from django.conf.urls.defaults import *
    from mysite.views import hello, current_datetime, hours_ahead

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
        url(r'^hora/mais/(\d{1,2})/$', hours_ahead),
    )

Resolvido isso, vamos escrever a view ``hours_ahead``.

``hours_ahead`` é bastante parecido com a view ``current_datetime`` escrita
anteriormente, com uma diferença chave: recebe mais um argumento, o número de
horas do deslocamento. Aqui está o código da view::

    from django.http import Http404, HttpResponse
    import datetime

    def hours_ahead(request, offset):
        try:
            offset = int(offset)
        except ValueError:
            raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        html = "<html><body>Em %s hora(s), será %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Vamos analisar o código passo-a-passo:

* A função view ``hours_ahead`` possui *dois* parametros: ``request`` e
  ``offset``.

  * ``request`` é um objeto do tipo ``HttpRequest``, assim como em
    ``hello`` e ``current_datetime``. Repetindo: cada view *sempre* recebe
    um ``HttpRequest`` como primeiro parâmetro.

  * ``offset`` é a string capturada pelo parênteses no URLpattern. Por
    exemplo, se a URL requisitada for ``/hora/mais/3/``, então a string
    ``offset`` seria a string ``'3'``. Se a URL fosse ``/time/plus/21/``,
    ``offset`` seria a string ``'21'``. Note que todos os valores capturados
    sempre serão *strings* e não inteiros, mesmo se a string for composta
    somente de dígitos, como ``'21'``.

    (Tecnicamente, valores capturados sempre serão *objetos Unicode*,
    não strings puras do Python, mas não se preocupe com essa
    diferenciação neste momento.)

    Decidimos chamar a variável ``offset``, mas pode ser nomeada como
    prefirir, contanto que seja um identificador Python válido. O nome
    da variável não importa; o que importa é que seja o segundo argumento,
    após ``request``. (Também é possível utilizar argumento por
    palavra-chave, ao invés de posicional, em um URLconf. Isso é explicado
    no Capítulo 8).

* A primeira coisa que fazemos dentro da função é chamar ``int()`` na
  variável ``offset``. Isso converte o valor da string para um inteiro.

  Note que o Python levantará uma exceção ``ValueError`` se a função
  ``int()`` for chama em um valor que não possa ser converido para
  inteiro, como a string ``foo``. Neste exemplo, se for encontrado um
  ``ValueError``, nós levantamos a exceção ``django.http.Http404``, que,
  como deve estar imaginando, resulta em um erro 404 "Página não
  encontrada".

  Leitores astutos se perguntarão: como chegaríamos ao caso ``ValueError``,
  já que a expressão regular no nosso URLpattern, ``(\d{1,2})``, captura
  somente digitos e, portanto, ``offset`` só será uma string composta de
  digitos? A resposta é, não chegaríamos, pois o URLpattern dá um modesto,
  porém poderoso, nível de validação de entrada, *mas* ainda checamos pelo
  ``ValueError``, caso a função view seja chamada de outro modo. É uma boa
  prática implementar funções view que não façam suposições sobre seus
  parametros. Acoplamento fraco, lembra-se?

* Na próxima linha da função calculmaos a data e hora atuais e o número
  apropiado de horas. Já haviamos visto ``datetime.datetime.now()``, usado
  na view ``current_datetime``. O novo conceito aqui é o uso de operações
  aritméticas com data/hora, usando o objeto ``datetime.timedelta`` e somando
  ao objeto ``datetime.datetime``. O resultado é armazenado na variável
  ``dt``.

  Essa linha também mostra o por quê de chamarmos ``int()`` no ``offset``,
  a função ``datetime.timedelta`` requer que o parametro ``hours`` seja um
  inteiro.

* Depois construimos a saída HTML dessa função view, como feito em
  ``current_datetime``. A pequena diferença desta linha com a anterior é que
  e utilizado a funcionalidade do Python de formatar strings com *dois*
  valores, não somente um. Por isso, há dois símbolos ``%s`` na string e
  uma tupla de valores a serem inseridos: ``(offset, dt)``.

* Por fim retornamos um ``HttpResponse`` do código HTML. Por agora esse já
  é um velho conhecido.

Após escrever esse URLconf e função view, inicie o servidor de desenvolvimneto
do Django (caso ainda não esteja sendo executado) e visite
``http://127.0.0.1:8000/time/plus/3/`` para verificar que funcione. Então tente
``http://127.0.0.1:8000/time/plus/5``. Então ``http://127.0.0.1:8000/time/plus/24``.
Por fim, visite ``http://127.0.0.1:8000/time/plus/100/`` para verificar que o
padrão em seu URLconf só aceita números com um ou dois digitos; Django deverá
mostrar um erro "Página não encontrada" nesse caso, assim como visto anteriormente
na seção "Uma nota rápida sobre erros 404". A URL
``http://127.0.0.1:8000/time/plus/`` (*sem* deslocamento de hora) também deverá
lançar um erro 404.

.. admonition:: Ordem do código

    Neste exemplo escrevemos primeiro o URLpattern e depois a view, mas nos
    exemplos anteriores escrevemos primeiro a view e então o URLpattern. Qual
    técnica é melhor?

    Bem, cada desenvolvedor é diferente.

    Se você é uma pessoa com uma boa visão de todo o problema, pode fazer
    sentido escrever todos os URLpatterns de uma só vez, no começo do projeto
    para então codificar as views. Isso tem a vantagem de fornecer uma lista
    de afazeres, e essencialmente define os requerimentos de parâmetros para
    as funções views a serem escritas.

    Se você é o tipo de desenvolvedor que começa as coisas por baixo, talvez
    prefira escrever as views primeiro e depois ligá-las a URLs. Essa forma
    também está certa.

    No fim a técnica utilizada é aquela seu cérebro melhor se adapta. Ambos
    métodos são válidos.

As páginas bonitas de erro do Django
====================================

Leve um tempo para admirar as boas aplicações Web que criamos até agora...
Agora vamos quebrá-las! Vamos intencionalmente colocar um erro Python em
nosso arquivo ``views.py`` comentando as linhas ``offset = int(offset)``
na view ``hours_ahead``::

    def hours_ahead(request, offset):
        # try:
        #     offset = int(offset)
        # except ValueError:
        #     raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Carregue o servidor de desenvolvimento e navegue para ``/time/plus/3/``. Você
verá uma página de erro com bastante informações, encluindo uma mensagem
``TypeError`` no topo da p\agina: ``"unsupported type for timedelta hours
compontent: unicode"``.

O que aconteceu? Bem, a função ``datetime.timedelta`` expera que o parametro
``horas`` seja um inteiro, e comentamos a parte de código que converte
``offset`` para inteiro. Isso fez com que ``datetime.timedelta`` levantasse o
``TypeError``. É o típico tipo de bug pequeno que todo programador acaba
passando por em alguma hora.

A motivação desse exmplo era demonstrar as páginas de erro do Django. Leve
algum tempo explorando a página de erro e preste atenção nas informações que
a página fornece.

Aqui estão alguns pontos para se saber:

* No topo da página, há informações importantes sobre a exceção: o tipo,
  quaisquer parâmetros para a exceção (a mensagem ``"unsupported type"``,
  neste caso), o arquivo em que a exceção foi levantada, e o número da
  linha em que ocorreu.

* Abaixo disso, a página exibe o traceback Python completo para a exceção.
  O traceback é parecido com o padrão, que é exibido no interpetrador Python
  em linha de comando, exceto que é mais interativo. Para cada nível
  ("frame") na pilha, o Django mostra o nome do arquivo, o nome da função ou
  método, o número da linha e o código-fonte da linha.

  Ao clicar na linha do código-fonte (em cinza escuro) você verá várias
  linhas antes e depois da linha com erro, para dar contexto.

  Clique "Local vars" em qualquer "frame" na pilha para ver uma tabela de
  todas variáveis locais e seus valores, nesse "frame", na exata hora em
  que a exceção foi levantada. Essa informação de debug pode ser de grande
  ajuda.

* Observe o texto "Switch to copy-and-paste-view" embaixo do cabeçalho
  "Traceback". Clique nessa palavras e o traceback mudará para uma versão
  alternativa que pode ser facilmente copiada e colada. Use essa opção
  quando quiser compartilhar o traceback da exceção com outras pessoas
  para receber apoio técnico -- do pessoal da sala de chat do Django no
  IRC ou na lista de emails de usuários do Django.

  Embaixo, o botão "Share this traceback on a public Web site" fará esse
  trabalho em somente um clique. Clique para postar o traceback no
  http://www.dpaste.com/, aonde será gerada uma URL única que pode ser
  compartilhada com outras pessoas.

* Após isso, a seção "Request information" inclui uma gama de informação
  sbore a requisição Web que gerou o erro: informação GET e POST, valor dos
  cookies, e meta informação, como cabeçalhos CGI. O Apêndice G possui uma
  referência completa sobre toda a informação que um objeto de requisição
  contém.

  Abaixo da seção "Request information", a seção "Settings" lista todas as
  configurações dessa instalação particular do Django. (Já mencionamos o
  ``ROOT_URLCONF``, e mostraremos várias configurações do Django no decorrer
  do livro. Todas as configurações são cobertas em detalhes no Apêndice D).

A página de erro do Django é capaz de exibir mais informação em alguns casos
especiais, como no caso de ocorrer um erro com a sintaxe de template. Esses
serão explicados mais tarde, ao explicar o sistema de templates do Django.
Por agora, retire os comentários da linha ``offset = inf(offset)`` para que
a view volte a funcionar corretamente.

Você é do tipo de programar que gosta de fazer debug com a ajuda de instruções
``print`` bem localizadas? Você pode utilizar a página do Django para fazer isso,
mas sem as instruções ``print``. A qualquer ponto na sua view, insira
temporariamente um ``assert False`` para ativar a página de erro. Então será
possível visualizar as variáveis locais e o estado do programa. Aqui está um
exemplo usando a view ``hours_ahead``::

    def hours_ahead(request, offset):
        try:
            offset = int(offset)
        except ValueError:
            raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        assert False
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Por fim, é óbvio que muitas dessas informações são sensíveis, pois mostram o
conteúdo do código Python e configuração do Django, e seria burrice mostrar
essa informação a qualquer um na Internet. Uma pessoa maliciosa poderia usar
para fazer engenharia reversa da sua aplicação Web e fazer coisas más. Por
esse motivo a página de erro do Django só é exibida quando seu projeto está em
modo debug. Como desativar o modo debug é explicado no Capítulo 12. Por
enquanto, fique atento que todo projeto django está em modo debug quando
iniciado. (Soa familiar? Os erros "Página não encontrada", descritos no
começo do capítulo, funcionam da mesma maneira).

O que vem depois?
=================

Até agora estivemos escrevendo nossas funções view com HTML codificado
diretamento no código Python. Fizemos isso para manter tudo simples para
demonstrar os conceitos mais importantes, mas no mundo real essa é, quase
sempre, uma má ideia.

Django vem com uma simples, porém poderosa engine de template que permite
separar o design da página do código por debaixo. A engine de templates do
Django será explicada no :doc:`next chapter <chapter04>`.
