=========================
Apêndice D: Configurações
=========================

Seu arquivo de configurações do Django contém toda a configuração de sua instalação do Django. Esse apêndice descreve como as configurações funcionam e quais estão disponíveis.

O que é um arquivo de configuração?
===================================

Um *arquivo de configuração* é apenas um módulo Python com variáveis no nível de módulo.

Eis aqui alguns exemplos de configurações::

    DEBUG = False
    DEFAULT_FROM_EMAIL = 'webmaster@example.com'
    TEMPLATE_DIRS = ('/home/templates/mike', '/home/templates/john')

Porque um arquivo de configuração é um módulo Python, o seguinte se aplica:

* O código Python deve ser válido; erros de sintaxe não são permitidos.

* Configurações podem ser dinamicamente declaradas usando a sintaxe normal do Python, por exemplo::

      MY_SETTING = [str(i) for i in range(30)]

* Valores de outros arquivos de configurações podem ser importados.

Configurações padrão
--------------------

Não é preciso definir nenhuma configuração em um arquivo de configurações do Django se ela não for necessária. Cada configuração tem um valor padrão. Essas predefinições estão no arquivo 
``django/conf/global_settings.py``.

Eis o algoritmo que o Django utiliza na compilação das configurações:

* Carregue as configurações de ``global_settings.py``.
* Carregue as configurações do arquivo de configurações específico, sobrescrevendo as configurações globais conforme necessário.

Note que um arquivo de configurações *não* deve ter um import from ``global_settings``, porque isso é redundante.

Vendo quais configurações você alterou
--------------------------------------

Há uma forma fácil de ver quais das suas configurações desviam das configurações padrão. O comando ``manage.py diffsettings`` mostra as diferenças entre o arquivo de configurações atual e o arquivo de configurações padrão do Django.

``manage.py`` é descrito com mais detalhes no Apêndice F.

Usando as configurações no código Python
----------------------------------------

Nas suas aplicações Django, use as configurações importando o objeto ``django.conf.settings``, por exemplo::

    from django.conf import settings

    if settings.DEBUG:
        # Faça algo

Note que ``django.conf.settings`` não é um módulo -- é um objeto. Então não é possível importar configurações individuais::

    from django.conf.settings import DEBUG  # Isso não vai funcionar.

Também note que seu código *não* deve importar de ``global_settings`` ou mesmo de seu próprio arquivo de configurações. ``django.conf.settings`` abstrai os conceitos das configurações padrão e das configurações específicas do projeto; apresenta uma interface única. Também dissocia o código que usa as configurações do local das suas configurações.

Alterando configurações em tempo de execução
--------------------------------------------

Você não deve alterar as configurações em suas aplicações em tempo de execução. Por exemplo, não faça isto em uma view::

    from django.conf import settings

    settings.DEBUG = True   # Não faça isso!

O único lugar que as configurações devem ser definidas é em um arquivo de configuração.

Segurança
---------

Como um arquivo de configurações contém informações sensíveis, tal como a senha do banco de dados, você deve tentar de qualquer forma limitar o acesso a ele. Por exemplo, mude as permissões do arquivo para que somente você e seu usuário do servidor web possam lê-lo. Isso é especialmente importante em um ambiente de servidor de hospedagem compartilhado.

Criando suas próprias configurações
-----------------------------------

Não há nada que lhe impeça de criar suas próprias configurações, para suas aplicações Django. Basta seguir estas convenções:

* Use todas as letras maiúsculas para definir os nomes das configurações.

* Para configurações que são sequências, use tuplas ao invés de listas. Configurações devem ser consideradas imutáveis e não devem ser alteradas uma vez que foram definidas. O uso de tuplas reflete essa semântica.

* Não reinvente uma configuração que já existe.

Designando as configurações: DJANGO_SETTINGS_MODULE
===================================================

Quando você usa Django, você tem de dizer a ele quais configurações você está usando. Faça isto por usar a variável de ambiente ``DJANGO_SETTINGS_MODULE``.

O valor de ``DJANGO_SETTINGS_MODULE`` deve ser na sintaxe de caminhos do Python (por exemplo, ``mysite.settings``). Note que o módulo de configurações deve estar no caminho de pesquisa de importação do Python (``PYTHONPATH``).

.. admonition:: Dica:

    Um bom guia para ``PYTHONPATH`` pode ser encontrada em 
    http://diveintopython.org/getting_to_know_python/everything_is_an_object.html.

O utilitário django-admin.py
----------------------------

Ao usar o ``django-admin.py`` (veja o Apêndice F), você pode definir a variável de ambiente uma vez ou mesmo pode passar explicitamente no módulo de configurações cada vez que você executar o utilitário.

Eis um exemplo usando o shell do Unix::

    export DJANGO_SETTINGS_MODULE=mysite.settings
    django-admin.py runserver

Eis um exemplo usando o shell do Windows::

    set DJANGO_SETTINGS_MODULE=mysite.settings
    django-admin.py runserver

Use o argumento de linha de comando ``--settings`` para especificar a configuração manualmente::

    django-admin.py runserver --settings=mysite.settings

O utilitário ``manage.py`` criado pelo ``startproject`` como parte do esqueleto do projeto define o ``DJANGO_SETTINGS_MODULE`` automaticamente; veja o Apêndice F para mais informações sobre o ``manage.py``.

No servidor (mod_python)
------------------------

No seu ambiente de produção, você vai precisar dizer ao Apache/mod_python qual arquivo de configuração usar. Faça isso com ``SetEnv``::

    <Location "/mysite/">
        SetHandler python-program
        PythonHandler django.core.handlers.modpython
        SetEnv DJANGO_SETTINGS_MODULE mysite.settings
    </Location>

Para mais informações, leia a documentação online do Django mod_python em http://docs.djangoproject.com/en/dev/howto/deployment/modpython/.

Usando as configurações sem definir o DJANGO_SETTINGS_MODULE
============================================================

Em alguns casos, talvez você queira ignorar a variável de ambiente ``DJANGO_SETTINGS_MODULE``. Por exemplo, se você está usando somente o sistema de templates, provavelmente não vai querer ter de configurarar uma variável de ambiente apontando para o módulo de configuração.

Nesses casos, você pode setar as configurações do Django manualmente. Faça isto chamando ``django.conf.settings.configure()``. Eis aqui um exemplo::

    from django.conf import settings

    settings.configure(
        DEBUG = True,
        TEMPLATE_DEBUG = True,
        TEMPLATE_DIRS = [
            '/home/web-apps/myapp',
            '/home/web-apps/base',
        ]
    )

Informe no ``configure()`` quantos argumentos/palavras-chave você quiser, com cada argumento representando uma configuração e seu valor. Cada nome do argumento (palavra-chave que representa o nome da variável) deve estar em maiúsculas, com o mesmo nome das configurações descritas anteriormente. Se uma configuração em particular não for informada em ``configure()`` e for necessária em algum momento no futuro, o Django irá usar o valor padrão da configuração.

Configurar o Django desta forma é, na maior parte das vezes, -- e, de fato, recomendado -- quando você está usando uma parte do framework dentro de uma aplicação maior.

Consequentemente, quando configurando via ``settings.configure()``, o Django não irá fazer nenhuma modificação nas variáveis de ambiente do processo. (Veja a explicação sobre ``TIME_ZONE`` mais tarde neste apêndice para o porquê de isso normalmente ocorrer.)
Se presume que você já possui total controle sobre seu ambiente, nesses casos.

Configurações padrão personalizadas
-----------------------------------

Se você quer que os valores padrão venham de outro lugar que não seja de ``django.conf.global_settings``, você pode passar em um módulo ou classe que fornece as configurações padrão como o argumento ``default_settings`` (ou na primeira posição como argumento) na chamada para o ``configure()``.

Neste exemplo, as configurações padrão são obtidas de ``myapp_defaults``, e a cofiguração ``DEBUG`` está setada como ``True``, independentemente de seu valor em ``myapp_defaults``::

    from django.conf import settings
    from myapp import myapp_defaults

    settings.configure(default_settings=myapp_defaults, DEBUG=True)

O exemplo a seguir, que usa ``myapp_defaults`` como um argumento posicional é equivalente a::

    settings.configure(myapp_defaults, DEBUG = True)

Normalmente, você não vai precisar sobrescrever os padrões dessa forma. Os padrões do Django são suficientemente inofencivos que você pode usá-los tranquilamente. Esteja ciente de que se você passar um novo módulo padrão, ele *substitui* inteiramente os padrões do Django, então você precisa especificar valores para cada configuração possível que possa ser utilizada no código que você está importando. Dê uma olhada em ``django.conf.settings.global_settings`` para lista compelta.

Ou o configure() ou o DJANGO_SETTINGS_MODULE é necessário
---------------------------------------------------------

Se você não configurar a variável de ambiente ``DJANGO_SETTINGS_MODULE``, você *precisa* executar o ``configure()`` em algum lugar antes de usar qualquer código que leia o arquivo de configurações.

Se você não definir o ``DJANGO_SETTINGS_MODULE`` e não executar o ``configure()``, o Django irá gerar uma exceção do tipo ``EnvironmentError`` na hora que o arquivo de configurações for acessado.

Se você definir o ``DJANGO_SETTINGS_MODULE``, acessar as configurações de alguma forma, e *então* executar o ``configure()``, o Django irá gerar uma exceção do tipo ``EnvironmentError`` afirmando que as configurações já foram definidas.

Também, é um erro executar o ``configure()`` mais de uma vez, ou executar o ``configure()`` após o arquivo de configurações ter sido acessado.

Em resumo faça assim: use somente um, ou o ``configure()`` ou o ``DJANGO_SETTINGS_MODULE``, e somente uma vez.

Configurações Disponíveis
=========================

As seções a seguir consistem em uma lista das principais configurações disponíveis, em ordem alfabética, e seus respectivos valores.

ABSOLUTE_URL_OVERRIDES
----------------------

*Valor padrão*: ``{}`` (dicionário vazio)

Esse é um dicionário de mapeamento de strings ``"app_label.model_name"`` para funções que assume um model object e retorna sua URL. Essa é uma forma de sobrescrever o método ``get_absolute_url()`` em cada uma de seus módulos instalados. Eis um exemplo::

    ABSOLUTE_URL_OVERRIDES = {
        'blogs.weblog': lambda o: "/blogs/%s/" % o.slug,
        'news.story': lambda o: "/stories/%s/%s/" % (o.pub_year, o.slug),
    }

Note que o nome do model usado nessa configuração deve ser todo em minúsculo, independentemente do real nome da classe.

ADMIN_MEDIA_PREFIX
------------------

*Valor padrão*: ``'/media/'``

Essa configuração é o prefixo da URL para os arquívos de mídia do admin: CSS, Javascript e imagens. Certifique-se de usar a barra invertida.

ADMINS
------

*Valor padrão*: ``()`` (tupla vazia)

Essa é a tupla que lista as pessoas que receberão notificações de erro. Quando ``DEBUG=False`` e a view lançar uma exceção, o Django irá enviar um email para essas pessoas com as informações sobre a exceção gerada. Cada membro da tupla deve ser uma tupla com (Nome completo, endereço de email), por exemplo::

    (('John', 'john@example.com'), ('Mary', 'mary@example.com'))

Observe que o Django irá enviar email para todas essas pessoas sempre que os erros aparecerem.

ALLOWED_INCLUDE_ROOTS
---------------------

*Valor padrão*: ``()`` (tupla vazia)

Essa é uma tupla de strings representando os prefixos autorizados para a template tag ``{% ssi %}``. Isso é uma medida de segurança, para que os autores de templates não acessem arquivos que eles não deveriam estar acessando.

Por exemplo, se ``ALLOWED_INCLUDE_ROOTS`` é ``('/home/html', '/var/www')`` então ``{% ssi /home/html/foo.txt %}`` funcionaria, mas ``{% ssi /etc/passwd %}`` não.

APPEND_SLASH
------------

*Valor padrão*: ``True``

Esta configuração indica se será adicionado barras ao final das URLs. Isso é usado somente se o ``CommonMiddleware`` estiver instalado (ver Capítulo 17). Veja também ``PREPEND_WWW``.

CACHE_BACKEND
-------------

*Valor padrão*: ``'locmem://'``

Esse é o back-end de cache para usar (ver Capítulo 15).

CACHE_MIDDLEWARE_KEY_PREFIX
---------------------------

*Valor padrão*: ``''`` (string vazia)

Esse é o prefixo de chave do cache que o middleware de cache deverá usar (ver Capítulo 15).

DATABASE_ENGINE
---------------

*Valor padrão*: ``''`` (string vazia)

Essa configuração indica qual back-end de banco de dados usar, por exemplo ``'postgresql_psycopg2'``, ou ``'mysql'``.

DATABASE_HOST
-------------

*Valor padrão*: ``''`` (string vazia)

Essa configuração indica qual endereço de host usar ao conectar com o banco de dados. String vazia significa ``localhost``. Isso não é usado com SQLite.

Se o valor começar com a barra (``'/'``) e você estiver usando MySQL, irá conectar via Unix socket para o socket especificado:

    DATABASE_HOST = '/var/run/mysql'

Se você estiver usando MySQL e esse valor *não* começar com uma barra, então é assumido que esse valor seja o host.

DATABASE_NAME
-------------

*Valor padrão*: ``''`` (string vazia)

Esse é o nome da base de dados para uso. Para SQLite, é o caminho completo para o arquivo da base de dados.

DATABASE_OPTIONS
----------------

*Valor padrão*: ``{}`` (dicionário vazio)

São parâmetros extras para usar ao conectar à base de dados. Consulte a documentação do back-end utilizado para as possíveis palavras-chave.

DATABASE_PASSWORD
-----------------

*Valor padrão*: ``''`` (string vazia)

Essa configuração é a senha para usar ao conectar à base de dados. Não é usado com SQLite.

DATABASE_PORT
-------------

*Valor padrão*: ``''`` (string vazia)

Essa é a porta para usar ao conectar à base de dados. String vazia significa a porta padrão. Não é usado com SQLite.

DATABASE_USER
-------------

*Valor padrão*: ``''`` (string vazia)

Essa configuração é o nome de usuário usado ao conectar à base de dados. Não é usado com SQLite.

DATE_FORMAT
-----------

*Valor padrão*: ``'N j, Y'`` (e.g., ``Feb. 4, 2003``)

Essa é a formatação padrão para usar em campos do tipo date nas páginas de listagem (change-list) do Django admin -- e, possivelmente, por outras partes do sistema. Os formatos aceitos são os mesmos da tag ``now`` (ver Apêndice E, Tabela E-2).

Veja também ``DATETIME_FORMAT``, ``TIME_FORMAT``, ``YEAR_MONTH_FORMAT``, e
``MONTH_DAY_FORMAT``.

DATETIME_FORMAT
---------------

*Valor padrão*: ``'N j, Y, P'`` (e.g., ``Feb. 4, 2003, 4 p.m.``)

Essa é a formatação padrão para usar em campos do tipo datetime nas páginas de listagem (change-list) do Django admin -- e, possivelmente, por outras partes do sistema. Os formatos aceitos são os mesmos da tag ``now`` (ver Apêndice E, Tabela E-2).

Veja também ``DATE_FORMAT``, ``DATETIME_FORMAT``, ``TIME_FORMAT``,
``YEAR_MONTH_FORMAT``, e ``MONTH_DAY_FORMAT``.

DEBUG
-----

*Valor padrão*: ``False``

Essa configuração é um Boolean que liga e desliga o modo de debug.

Se você definir configurações personalizadas, ``django/views/debug.py`` tem uma expressão regular ``HIDDEN_SETTINGS`` que irá esconder da ``DEBUG`` view qualquer coisa que conter ``'SECRET``, ``PASSWORD``, or ``PROFANITIES'``. Isso permite usuários não confiáveis sejam capazes de dar backtraces sem ver configurações sensíveis ou ofensivas.

Mesmo assim, note que sempre haverão seções de saída do seu debug que são inapropriadas para o público. Caminhos de arquivos, opções de configurações, bem como dar informações extra sobre seu servidor para invasores.

Nunca faça o deploy de um site com ``DEBUG`` ligado.


DEFAULT_CHARSET
---------------

*Valor padrão*: ``'utf-8'``

Esse é o conjunto de caracteres padrão para usar em todos os objetos ``HttpResponse``, se um MIME type não é manualmente especificado. É usado com ``DEFAULT_CONTENT_TYPE`` para construir o ``Content-Type`` do cabeçalho. Veja o Apêndice G para mais sobre objetos ``HttpResponse``.

DEFAULT_CONTENT_TYPE
--------------------

*Valor padrão*: ``'text/html'``

Esse é o tipo de conteúdo padrão usado para todos os objetos ``HttpResponse``, se um MIME type não é manualmente especificado. É usado com ``DEFAULT_CHARSET`` para construção do ``Content-Type`` do cabeçalho. Veja o Apêndice G para mais sobre objetos ``HttpResponse``.

DEFAULT_FROM_EMAIL
------------------

*Valor padrão*: ``'webmaster@localhost'``

Esse é o email padrão usado para várias correspondências automáticas dos administradores do site.

DISALLOWED_USER_AGENTS
----------------------

*Valor padrão*: ``()`` (tupla vazia)

Essa é uma lista de objetos de expressões regulares representando strings de User-Agent que não têm permissão para visitar qualquer página, em todo o sistema. Use isto para robôs/rastreadores mal intencionados. Isso é somente utilizado se ``CommonMiddleware`` estiver instalado (ver Capítulo 17).

EMAIL_HOST
----------

*Valor padrão*: ``'localhost'``

Isso é o host utilizado para enviar email. Veja támbém ``EMAIL_PORT``.

EMAIL_HOST_PASSWORD
-------------------

*Valor padrão*: ``''`` (string vazia)

Essa é a senha utilizada para o servidor SMTP definido em ``EMAIL_HOST``. Essa configuração é usada em conjunto com ``EMAIL_HOST_USER`` ao autenticar no servidor SMTP. Se qualquer uma dessas configurações estiver vazia, o Django não tentará autenticar.

Veja também ``EMAIL_HOST_USER``.

EMAIL_HOST_USER
---------------

*Valor padrão*: ``''`` (string vazia)

Esse é o nome de usuário a ser utilizado para o servidor SMTP definido em ``EMAIL_HOST``. Se estiver vazio, o Django o Django não tentará autenticar. 

Veja também ``EMAIL_HOST_PASSWORD``.

EMAIL_PORT
----------

*Valor padrão*: ``25``

Essa é a porta utilizada para o servidor SMTP definido em ``EMAIL_HOST``.

EMAIL_SUBJECT_PREFIX
--------------------

*Valor padrão*: ``'[Django] '``

Esse é o prefixo da linha assunto para emails enviados com ``django.core.mail.mail_admins`` ou ``django.core.mail.mail_managers``. Você, provavelmente, vai querer incluir o espaço à direita.

FIXTURE_DIRS
-------------

*Valor padrão*: ``()`` (tupla vazia)

Essa é a lista de localizações dos arquivos fixture, em ordem de busca. Note que estes caminhos devem usar barras Unix-style, mesmo no Windows. É utilizado pelo framework de testes do Django, que é documentado online em http://docs.djangoproject.com/en/dev/topics/testing/.

IGNORABLE_404_ENDS
------------------

*Valor padrão*: ``('mail.pl', 'mailform.pl', 'mail.cgi', 'mailform.cgi', 'favicon.ico',
'.php')``

Essa é a tupla de strings que especifica inícios de URLs que devem ser ignoradas pelo 404 e-mailer. (Ver Capítulo 12 para mais sobre 404 e-mailer.)

Nenhum erro será enviado para as URLs terminadas com as strings dessa sequência.

Veja também ``IGNORABLE_404_STARTS`` e ``SEND_BROKEN_LINK_EMAILS``.

IGNORABLE_404_STARTS
--------------------

*Valor padrão*: ``('/cgi-bin/', '/_vti_bin', '/_vti_inf')``

Veja também ``SEND_BROKEN_LINK_EMAILS`` e ``IGNORABLE_404_ENDS``.

INSTALLED_APPS
--------------

*Valor padrão*: ``()`` (tupla vazia)

Uma tupla de strings designando todas as aplicações que serão habilitadas nessa instalação do Django. Cada string deve ser um caminho completo de módulo Python para um pacote Python que contenha a aplicação Django. Ver Capítulo 5 para mais sobre aplicações.

LANGUAGE_CODE
-------------

*Valor padrão*: ``'en-us'``

Essa é a string representando o código de idioma para esta instalação. E deve estar no padrão de formato de linguagem -- for example, U.S. English é ``"en-us"``. Ver Capítulo 19.

LANGUAGES
---------

*Valor padrão*: Uma tupla com todas os idiomas disponíveis. Essa lista está continuamente crescendo e qualquer cópia incluída aqui irá, inevitavelmente, estar desatualizada. Você pode ver a lista atualizada de idiomas traduzidos em ``django/conf/global_settings.py``.

A lista é uma tupla com uma tupla dupla no formato (código do idioma, nome do idioma) -- por exemplo, ``('ja', 'Japanese')``. Isso especifica quais idiomas estão disponíveis para 
The list is a tuple of two-tuples in the format (language code, language name)
-- for example, ``('ja', 'Japanese')``. Isto especifica quais idiomas estão disponíveis para seleção de idioma. Ver Capítulo 19 para mais sobre seleção de idiomas.

Geralmente, o valor padrão deve ser suficiente. Somente redefina essa configuração se você quer restringir a seleção de idiomas para um subconjunto de idiomas oferecidos pelo Django.

Se você definir a configuração ``LANGUAGES`` personalizadamente, está tudo bem em marcar os idiomas como strings de tradução, mas você *nunca* deve importar ``django.utils.translation`` no seu arquivo de configuração, porque esse módulo por si depende do arquivo de configuração, e isto irá causar uma importação circular.

A solução é usar uma função "dummy" ``gettext()``. Aqui está um exemplo de arquivo de configuração::

    gettext = lambda s: s

    LANGUAGES = (
        ('de', gettext('German')),
        ('en', gettext('English')),
    )

Com essa disposição, ``make-messages.py`` ainda irá encontrar e marcar essa string para tradução, mas a tradução não acontecerá em tempo de execução -- então você tem de lembrar de envolver os idiomas em um *real* ``gettext()`` em qualquer código que usar ``LANGUAGES`` em tempo de execução.

MANAGERS
--------

*Valor padrão*: ``()`` (tupla vazia)

Essa tupla está no mesmo formato de ``ADMINS`` que especifica quem irá obter notificações de links quebrados quando ``SEND_BROKEN_LINK_EMAILS=True``.

MEDIA_ROOT
----------

*Valor padrão*: ``''`` (string vazia)

Isso é um caminho absoluto para o diretório que contém mídias para esta instalação (por exemplo, ``"/home/media/media.lawrence.com/"``) Veja também ``MEDIA_URL``.

MEDIA_URL
---------

*Valor padrão*: ``''`` (string vazia)

Essa URL lida com as mídias servidas a partir de ``MEDIA_ROOT`` (por exemplo,
``"http://media.lawrence.com"``).

Note que isso deve possuir uma barra à direita, se possuir um caminho de componente:

* *Correto*: ``"http://www.example.com/static/"``
* *Incorreto*: ``"http://www.example.com/static"``

Ver Capítulo 12 para mais sobre deployment e servir mídias.

MIDDLEWARE_CLASSES
------------------

*Valor padrão*::

    ("django.contrib.sessions.middleware.SessionMiddleware",
     "django.contrib.auth.middleware.AuthenticationMiddleware",
     "django.middleware.common.CommonMiddleware",
     "django.middleware.doc.XViewMiddleware")

Essa é uma tupla de classes middleware para uso. Ver Capítulo 17.

MONTH_DAY_FORMAT
----------------

*Valor padrão*: ``'F j'``

Essa é a formatação padrão para usar em campos do tipo date nas páginas de listagem (change-list) do Django admin -- e, possivelmente, por outras partes do sistema -- em casos quando somente o mês e o dia são mostrados. Os formatos aceitos são os mesmos da tag ``now`` (ver Apêndice E, Tabela E-2).

Por exemplo, quando uma change-list no Django admin está filtrando por data, o cabeçalho para um determinado dia mostra o dia e o mês; Diferentes localidades têm diferentes formatos. Por exemplo, em U.S. English seria "January 1," onde em Spanish poderia ser "1 Enero."

Veja também ``DATE_FORMAT``, ``DATETIME_FORMAT``, ``TIME_FORMAT``, e
``YEAR_MONTH_FORMAT``.

PREPEND_WWW
-----------

*Valor padrão*: ``False``

Essa configuração indica se será precedido o subdomínio "www." em URLs que não o possui. Isso é somente utilizado se ``CommonMiddleware`` estiver instalado (ver Capítulo 17).

Veja também ``APPEND_SLASH``.

ROOT_URLCONF
------------

*Valor padrão*: Not defined

This is a string representing the full Python import path to your root URLconf (e.g.,
``"mydjangoapps.urls"``). Ver Capítulo 3.

SECRET_KEY
----------

*Valor padrão*: (Generated automatically when you start a project)

This is a secret key for this particular Django installation. It is used to provide a seed in
secret-key hashing algorithms. Set this to a random string -- the longer, the
better. ``django-admin.py startproject`` creates one automatically and most
of the time you won't need to change it

SEND_BROKEN_LINK_EMAILS
-----------------------

*Valor padrão*: ``False``

This setting indicates whether to send an email to the ``MANAGERS`` each time somebody visits a
Django-powered page that is 404-ed with a nonempty referer (i.e., a broken
link). This is only used if ``CommonMiddleware`` is installed (see Chapter 17).
Veja também ``IGNORABLE_404_STARTS`` and ``IGNORABLE_404_ENDS``.

SERIALIZATION_MODULES
---------------------

*Valor padrão*: Not defined.

Serialization is a feature still under heavy development. Refer to the online
documentation at http://docs.djangoproject.com/en/dev/topics/serialization/
for more information.

SERVER_EMAIL
------------

*Valor padrão*: ``'root@localhost'``

This is the email address that error messages come from, such as those sent to
``ADMINS`` and ``MANAGERS``.

SESSION_COOKIE_AGE
------------------

*Valor padrão*: ``1209600`` (two weeks, in seconds)

This is the age of session cookies, in seconds. Ver Capítulo 14.

SESSION_COOKIE_DOMAIN
---------------------

*Valor padrão*: ``None``

This is the domain to use for session cookies. Set this to a string such as
``".lawrence.com"`` for cross-domain cookies, or use ``None`` for a standard
domain cookie. Ver Capítulo 14.

SESSION_COOKIE_NAME
-------------------

*Valor padrão*: ``'sessionid'``

This is the name of the cookie to use for sessions; it can be whatever you want.
Ver Capítulo 14.

SESSION_COOKIE_SECURE
---------------------

*Valor padrão*: ``False``

This setting indicates whether to use a secure cookie for the session cookie.
If this is set to ``True``, the cookie will be marked as "secure,"
which means browsers may ensure that the cookie is only sent under an HTTPS connection.
Ver Capítulo 14.

SESSION_EXPIRE_AT_BROWSER_CLOSE
-------------------------------

*Valor padrão*: ``False``

This setting indicates whether to expire the session when the user closes
his browser. Ver Capítulo 14.

SESSION_SAVE_EVERY_REQUEST
--------------------------

*Valor padrão*: ``False``

This setting indicates whether to save the session data on every request. Ver Capítulo 14.

SITE_ID
-------

*Valor padrão*: Not defined

This is the ID, as an integer, of the current site in the ``django_site`` database
table. It is used so that application data can hook into specific site(s)
and a single database can manage content for multiple sites. Ver Capítulo 16.

TEMPLATE_CONTEXT_PROCESSORS
---------------------------

*Valor padrão*::

    ("django.core.context_processors.auth",
    "django.core.context_processors.debug",
    "django.core.context_processors.i18n",
    "django.core.context_processors.media")

This is a tuple of callables that are used to populate the context in ``RequestContext``.
These callables take a request object as their argument and return a dictionary
of items to be merged into the context. Ver Capítulo 9.

TEMPLATE_DEBUG
--------------

*Valor padrão*: ``False``

This Boolean turns template debug mode on and off. If it is ``True``, the fancy
error page will display a detailed report for any ``TemplateSyntaxError``. This
report contains the relevant snippet of the template, with the appropriate line
highlighted.

Note that Django only displays fancy error pages if ``DEBUG`` is ``True``, so
you'll want to set that to take advantage of this setting.

Veja também ``DEBUG``.

TEMPLATE_DIRS
-------------

*Valor padrão*: ``()`` (tupla vazia)

This is a list of locations of the template source files, in search order. Note that these
paths should use Unix-style forward slashes, even on Windows. See Chapters 4 and
9.

TEMPLATE_LOADERS
----------------

*Valor padrão*::

    ('django.template.loaders.filesystem.load_template_source',
    'django.template.loaders.app_directories.load_template_source')

This is a tuple of callables (as strings) that know how to import templates from
various sources. Ver Capítulo 9.

TEMPLATE_STRING_IF_INVALID
--------------------------

*Valor padrão*: ``''`` (string vazia)

This is output, as a string, that the template system should use for invalid (e.g.,
misspelled) variables. Ver Capítulo 9.

TEST_RUNNER
-----------

*Valor padrão*: ``'django.test.simple.run_tests'``

This is the name of the method to use for starting the test suite. It is used by Django's
testing framework, which is covered online at
http://docs.djangoproject.com/en/dev/topics/testing/.

TEST_DATABASE_NAME
------------------

*Valor padrão*: ``None``

This is the name of database to use when running the test suite. If a value of ``None``
is specified, the test database will use the name ``'test_' +
settings.DATABASE_NAME``. See the documentation for Django's testing framework,
which is covered online at http://docs.djangoproject.com/en/dev/topics/testing/.

TIME_FORMAT
-----------

*Valor padrão*: ``'P'`` (e.g., ``4 p.m.``)

This is the default formatting to use for time fields on Django admin change-list pages
-- and, possibly, by other parts of the system. It accepts the same format as the
``now`` tag (see Appendix E, Table E-2).

Veja também ``DATE_FORMAT``, ``DATETIME_FORMAT``, ``TIME_FORMAT``,
``YEAR_MONTH_FORMAT``, and ``MONTH_DAY_FORMAT``.

TIME_ZONE
---------

*Valor padrão*: ``'America/Chicago'``

This is a string representing the time zone for this installation. Time zones are in the
Unix-standard ``zic`` format. One relatively complete list of time zone strings
can be found at
http://www.postgresql.org/docs/8.1/static/datetime-keywords.html#DATETIME-TIMEZONE-SET-TABLE.

This is the time zone to which Django will convert all dates/times --
not necessarily the time zone of the server. For example, one server may serve
multiple Django-powered sites, each with a separate time-zone setting.

Normally, Django sets the ``os.environ['TZ']`` variable to the time zone you
specify in the ``TIME_ZONE`` setting. Thus, all your views and models will
automatically operate in the correct time zone. However, if you're using the
manually configuring settings (described above in the section titled "Using
Settings Without Setting DJANGO_SETTINGS_MODULE"), Django will *not* touch the
``TZ`` environment variable, and it will be up to you to ensure your processes
are running in the correct environment.

.. note::
    Django cannot reliably use alternate time zones in a Windows environment. If
    you're running Django on Windows, this variable must be set to match the
    system time zone.

URL_VALIDATOR_USER_AGENT
------------------------

*Valor padrão*: ``Django/<version> (http://www.djangoproject.com/)``

This is the string to use as the ``User-Agent`` header when checking to see if URLs
exist (see the ``verify_exists`` option on ``URLField``; see Appendix A).

USE_ETAGS
---------

*Valor padrão*: ``False``

This Boolean specifies whether to output the ETag header. It saves
bandwidth but slows down performance. This is only used if ``CommonMiddleware``
is installed (see Chapter 17).

USE_I18N
--------

*Valor padrão*: ``True``

This Boolean specifies whether Django's internationalization system (see
Chapter 19) should be enabled. It provides an easy way to turn off internationalization, for
performance. If this is set to ``False``, Django will make some optimizations so
as not to load the internationalization machinery.

YEAR_MONTH_FORMAT
-----------------

*Valor padrão*: ``'F Y'``

This is the default formatting to use for date fields on Django admin change-list pages
-- and, possibly, by other parts of the system -- in cases when only the year
and month are displayed. It accepts the same format as the ``now`` tag (see
Appendix E).

For example, when a Django admin change-list page is being filtered by a date
drill-down, the header for a given month displays the month and the year.
Different locales have different formats. For example, U.S. English would use
"January 2006," whereas another locale might use "2006/January."

Veja também ``DATE_FORMAT``, ``DATETIME_FORMAT``, ``TIME_FORMAT``, and
``MONTH_DAY_FORMAT``.
