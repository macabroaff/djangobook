================================
Capítulo 19: Internacionalização
================================

Django foi originalmente desenvolvido na região central dos Estados Unidos -- 
mais precisamente, em Lawrence, Kansas, sendo menos do que 64km do centro
geográfico do continente norte americano. Como a maioria dos projetos de código aberto,
a comunidade também cresceu e incluiu pessoas de todo o globo. Como a comunidade do Django 
esta cada vez mais diversificada, a "internacionalização" e a "localização" 
se tornou cada vez mais relevante. Como muitos desenvolvedores tem um conhecimento
impreciso desses termos, nós iremos defini-los rapidamente.


*Internacionalização* refere-se ao processo de produzir programas com uso potencial
em qualquer localidade. Isto inclui marcar textos (como elementos de interface com o usuário
e mensagens de erro) para  futuras traduções, abstrair a exibição de datas e horários
de modo que diferentes padrões locais possam ser observados, prover suporte para 
diferentes fusos horários e certificar-se que o código não contém nenhuma suposições
sobre a localização dos usuários. Você verá frequentimente o termo "internacionalização" 
abreviado para *I18N*. (O "18" refere-se ao número de letras omitidas entre a letra inicial
"I" e a letra final "N", no termo em inglês "internationalization")

*Localização* refere-se ao processo de traduzir, propriamente, um programa
internacionalizado para uso em uma determinada localidade. Você frequentimente
verá o termo "localização" abreviado para *L10N*.

Django em si é totalmente internacionalizado; todas as strings são marcadas
para traduzição e as configurações controlam a exibição de valores que 
dependem de localidade, como horas e datas. Django também vem com mais de 
50 arquivos de localização. Se você não é um falante nativo de inglês,
há uma boa chance que o Django já está traduzido para sua língua-mãe.

O mesmo framework de internacionalização usado para estas localizações
está disponível para você usar em seus próprios códigos e templates.

Para usar este framework, você precisará adicionar um número mínimo de 
ganchos no seu código Python e templates. Esses ganchos são chamados de 
*strings de tradução*. Eles dizem ao Django que o texto deve ser traduzido
à língua do usuário final, se essa tradução estiver disponível.

Django lida com esses ganchos para traduzido aplicações web, em tempo de 
execução, de acordo com as preferências linguísticas do usuário.

Essencialmente, Django faz duas coisas:

* Ele permite desenvolvedores e autores de templates espeficar quais
  partes de suas aplicações devem ser traduzidas.

* Ele usa estas informações para traduzir aplicações web para determinados
  usuários de acordo com suas preferências linguísticas.

.. nota::

    O maquinário de tradução do Django usa a GNU ``gettext`` 
    (http://www.gnu.org/software/gettext/) através do módulo 
    padrão ``gettext`` que vem com o Python.

.. advertência:: Se Você Não Precisar de Internacionalização:

    Os ganhos de internacionalização do Django são habilitados por padrão,
    o que ocorre com um pouco de overhead. Se você não usar internacionalização,
    você deverá configurar ``USE_I18N = False`` no seu 
    arquivo de configuração (settings.py). Se ``USE_I18N`` estiver em ``False``, 
    então Django irá fazer otimizações   para não carregar o 
    maquinário de internacionalização.
    
    Você provavelmente também quererá remover 
    ``'django.core.context_processors.i18n'`` da sua configuração 
    ``TEMPLATE_CONTEXT_PROCESSORS``.
    
Os três passos para internacionalização de sua aplicação Django são:


1. Incorporar strings de tradução em seu código Python e templates.

2. Obter traduções para estas strings, em qualquer idioma que deseja suportar.

3. Ativar o middleware de localização nas suas configuração do Django.

Cobriremos cada um desses passos detalhadamente.

1. Como especificar strings de tradução
=====================================

Strings de tradução especificam que um texto deve ser traduzido. Estas
strings podem aparecer no código Python ou em templates. É de responsabilidade
do desenvolvedor marcar estas strings; o sistema só pode traduzir strings marcadas 
manualmente.

No código Python
--------------

Tradução padrão
~~~~~~~~~~~~~~~~~~~~

Especifica uma string de traução usando a função ``ugettext()``. 
Uma convenção existente é importar esta função usando o encurtador, 
``_``, para diminuir digitação.

Neste exemplo, o texto ``"Bem-vindo ao meu site."`` é marcado como uma
string de tradução::

    from django.utils.translation import ugettext as _

    def my_view(request):
        output = _("Bem-vindo ao meu site.")
        return HttpResponse(output)
        
Obviamente, você poderia codificar isso sem usar o encurtador. Este exemplo
é idêntico ao anterior::

    from django.utils.translation import ugettext

    def my_view(request):
        output = ugettext("Bem-vindo ao meu site.")
        return HttpResponse(output)
        
Tradução trabalha sobre valores computados. Este exemplo é idêntico aos anteriores::

    def my_view(request):
        words = ['Bem-vindo', 'ao', 'meu', 'site.']
        output = _(' '.join(words))
        return HttpResponse(output)

Tradução trabalha sobre variáveis. Segue outro exemplo idêntico::

    def my_view(request):
        sentence = 'Bem-vindo ao meu site.'
        output = _(sentence)
        return HttpResponse(output)
        

(O embargo de usar variáveis or valores computados, como nos últimos dois
exemplos, é que o utilitário de detecção de strings de tradução do Django, 
``django-admin.py makemessages``, não será capz de encontrar essas strings.
Mais informações sobre ``makemessages`` mais a frente.)

As strings passadas para ``_()`` ou ``ugettext()`` pode pegar parâmetros,
especificados com o padrão sintático de interpolação nome-string.
Exemplo::

    def my_view(request, m, d):
        output = _('Hoje é %(dia)s de %(mes)s.') % {'mes': m, 'dia': d}
        return HttpResponse(output)
        
Esta técnica permite que as traduções reordenem parâmetros do texto.
Por exemplo, em uma tradução inglesa poderia ser ``"Today is November 26."``
enquanto uma espanhola seria ``"Hoy es 26 de Noviembre."`` -- com os parâmetros
(mes e dia) com posições trocadas.

Por esta razão, pode-se usar a interpolação nome-string (e.g., ``%(day)s``)
ao invés da interpolação posicional (e.g., ``%s`` ou ``%d``) onde houver
mais de um parâmetro. Se for usada a interpolação posicional, traduções
não serão possíveis para parâmetros reordenados.

Marcando strings como no-op
~~~~~~~~~~~~~~~~~~~~~~~~

Use a função ``django.utils.translation.ugettext_noop()`` para marcar uma string
como string de tradução sem traduzí-la. A string é posteriormente traduzido de uma
variável.

Use isso se você tiver strings constantes que devem ser armazenadas na 
linguagem de origem, porque elas são trocadas pelo sistema ou usuários -- 
como strings em um banco de dados -- mas devem ser traduzidas no último
momento possível, e.g. quando a string é exibida para o usuário.

Tradução preguiçosa
~~~~~~~~~~~~~~~~

Use a função ``django.utils.translation.ugettext_lazy()`` para traduzir strings
preguiçosamente (lazy evaluation) -- quando o valor é acessado, ao invés
do momento em que ``ugettext_lazy()`` é chamada.

Por exemplo, para traduzir um modelo de ``help_text``, faça o seguinte::

    from django.utils.translation import ugettext_lazy

    class MyThing(models.Model):
        name = models.CharField(help_text=ugettext_lazy('Este é um texto de ajuda'))
        
Neste exemplo, ``ugettext_lazy()`` guarda uma referência preguiçosa para a string--
não a tradução em si. A tradução serpa feita quando a string for usada em um contexto
de string, como na rederização de um template no site do admin de Django.

O resultado de uma chamada a ``ugettext_lazy()`` pode ser usada onde você
usaria uma string unicode (um objeto do tipo ``unicode``) em Python. Se você
tentar usar isso no lugar de uma bytestring (um objeto ``str``), ocorrerá um
comportamento inesperado, pois um objeto ``ugettext_lazy()`` não sabe como se
converter para um bytestring. Também não é possível usar uma string unicode dentro
de uma bytestring, então isso é consistente como comportamente de Python. Por exemplo::

The result of a ``ugettext_lazy()`` call can be used wherever you would use a
unicode string (an object with type ``unicode``) in Python. If you try to use
it where a bytestring (a ``str`` object) is expected, things will not work as
expected, since a ``ugettext_lazy()`` object doesn't know how to convert
itself to a bytestring.  You can't use a unicode string inside a bytestring,
either, so this is consistent with normal Python behavior. For example::
    
    # Isto está certo: por um proxy unicode em uma string unicode
    u"Olá %s" % ugettext_lazy("pessoas")
    
    # Isto não funcionará, já que não pode-se inserir um objeto unicode
    # em um bytestring (nem pode-se inserir nosso proxy unicode aqui)
    "Olá %s" % ugettext_lazy("pessoas")

Uma saída desta forma ``"olá <django.utils.functional...>"``, 
indica que você tentou inserir o resultado de ``ugettext_lazy()``
em um bytestring. Isso é um bug no seu código.


Se ``ugettext_lazy`` parece muito verborrágico, é possível usar um nome-atalho,
como ``_`` (underline), como em::

    from django.utils.translation import ugettext_lazy as _

    class Minha_coisa(models.Model):
        nome = models.CharField(texto_de_ajuda=_('Este é um texto de ajuda'))
        
Sempre use traduções preguiçosa em modelos de Django. Nomes de campos e nomes
de tabelas devem ser marcadas para trandução; caso contrário, eles não serão
traduzidos na interface do administrador. Isto significa escrever explicitamente
``nome_verboso`` e ``nome_verboso_no_plural`` nas opções da classe``Meta``,
ao invés de contar com a determinação padrão de Django de ``nome_verboso``
e ``nome_verboso_no_plural``, que olha o nome da classe do modelo::

    from django.utils.translation import ugettext_lazy as _

    class MyThing(models.Model):
        nome = models.CharField(_('nome'), texto_de_ajuda=_('Este é um texto de ajuda'))
        class Meta:
            nome_verboso = _('minha coisa')
            nome_verboso_no_plural = _('minhas coisas')

Pluralização
~~~~~~~~~~~~~

Use a função ``django.utils.translation.ungettext()`` para especificar mensagens
pluralizadas. Por exemplo::

    from django.utils.translation import ungettext

    def hello_world(request, contador):
        pagina = ungettext('Há apenas %(contador)d objeto',
            'Existem %(contador)d objetos', contador) % {
                'contador': contador,
            }
        return HttpResponse(pagina)

``ungettext`` recebe três argumentos: a string de tradução no singular, a string
de tradução no plural e o número de objetos (que é passada para as linguas de tradução
como a variável ``contador``).


Em templates
----------------

Tradução de templates Djanho usam duas tags de template e um sintaxe ligeiramente
diferente da usada em código Python. Para dar ao template acesso a essas tags, ponha
``{% load i18n %}`` no topo do template.

A tag de template ``{% trans %}`` traduz tanto strings constantes (dentro de 
aspas simples ou duplas) e conteúdo de variáveis::

    <title>{% trans "Isto é um título." %}</title>
    <title>{% trans minhaVariavel %}</title>

Se a opção ``noop`` está presente, o lookup de variavéis ainda atua, porém
a tradução é pulada. Isso é útil quando estiver "apagando" conteúdo que exigirá
tradução no futuro::

    <title>{% trans "minhaVariavel" noop %}</title>
    
Não é possível misturar uma varivável de template dentro de uma string, como em
``{% trans %}``. Se strings exigirem variáveis (espaços reservados), use 
``{% blocktrans %}``::

    {% blocktrans %}Esta string terá {{ valor }} dentro.{% endblocktrans %}
    
Para traduzir uma expressão de template -- digamos, usando filtros de template --
você precisará ligar a expressão à uma variável local para usá-la dentro do
bloco de tradução

    {% blocktrans with valor|filter as minhavariavel %}
    Isto terá {{ minhavariavel }} dentro.
    {% endblocktrans %}

Se for necessário ligar mais de uma expressão dentro de uma tag ``blocktrans``,
separe os pedações com ``and``::

    {% blocktrans with livro|titulo as livro_t and autor|titulo as autor_t %}
    Isto é {{ livro_t }} by {{ autor_t }}
    {% endblocktrans %}
    
Para pluralizar, especifique tanto as formas singular e plural com a
tag ``{% plural %}``, a qual aparece dentro de ``{% blocktrans %}`` e
``{% endblocktrans %}``. Por exemplo::

    {% blocktrans count lista|tamanho as contador %}
    Há apenas um {{ none }} objeto.
    {% plural %}
    Hão {{ contador }} {{ nome }} objetos.
    {% endblocktrans %}

Internamente, todos os blocos e traduções inline usam a chamada apropriada
de ``ugettext`` / ``ungettext``.

Cada ``RequestContext`` tem acesso a três variáveis para traduções específicas:

* ``LANGUAGES`` é uma lista de tuplas nas quais o primeiro elemento é o código
  da linguagem e o segundo é o nome da linguagem (traduzido para a lingua local).

* ``LANGUAGE_CODE`` é a linguagem preferencial do usuário, em formato string.
  Exemplo: ``pt-br``. (Veja "Como Django descobre a linguagem preferencial", abaixo.)
  
* ``LANGUAGE_BIDI`` é a direção local. Se é True, a linguagem que vai
  da direita para esquerda, e.g.:: Hebreu, Árabe. Se é False, é 
  uma linguagem que vai  da esquerda para direita, e.g.:: Inglês, Alemão,
  Francês etc.
  
Se a extensão ``RequestContext`` não for usada, esses valores podem ser obtidos
com três tags::

    {% get_linguagem_local as LANGUAGE_CODE %}
    {% get_linguagens_disponiveis as LANGUAGES %}
    {% get_linguagem_local_bidi as LANGUAGE_BIDI %}
    
Essas tags também requerem uma ``{% load i18n %}``.

Ganchos de tradução também estão disponíves dentro de qualquer tag de bloco
de template que aceite strings constantes. Nestes casos, apenas use a sintaxe 
``_()`` para especificar uma string de tradução::

    {% alguma_tag_especial _("Pagina não encontrada") valor|yesno:_("yes,no") %}

Nesse caso, tanto a tag quanto o filtro verão a string já traduzida, então
eles não precisam estar conscientes de traduções.

.. nota::
    Nesse exemplo, a infra-estrutura de tradução passará a string ``"yes,no"``,
    não as strings ``yes`` e ``no`` individualmente. A string traduzida precisará
    conter a vírgula para que o filtro que analisa o código saiba como dividir os
    argumento. Por exemplo, um tradutor alemão precisará traduzir a string
    ``"yes,no"`` como ``"ja,nein"`` (deixando a vírgula intacta).

Trabalhando com tradução preguiçosa de objetos
-------------------------------------

Usar ``ugettext_lazy()`` e ``ungettext_lazy()`` para marcar strings em modelos e 
funções utilitárias é uma operação comum. Quando trabalha-se com esses objetos
em outros pontos do código, deve-se garantir que eles não sejam convertidos em
strings, pois eles devem ser convertidos os mais tardiamente possível (de modo 
que o contexto local seja usado). Para tanto, deve-se usar algumas funções auxiliares.

Concatenando strings: string_concat()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As concatenações padrão de Python (``''.join([...])`) não funcionaram em listas que contém
objetos de tradução preguiçosa. No lugar delas, deve-se usar y, que cria
um objeto preguiçoso que concatena seu conteúdo *e* o converte em strings
apenas quando o resultado é incluído em uma string. Por exemplo::

    from django.utils.translation import string_concat
    # ...
    nome = ugettext_lazy(u'John Lennon')
    instrumento = ugettext_lazy(u'guitarra')
    resultado = string_concat([nome, ': ', instrumento])
    
Aqui, as traduções preguiçosa em ``resultado`` irão apenas ser convertidas
em string quando ``resultado`` é usado em uma string (geralmente durante a
renderização de um template).

O decorador allow_lazy()
~~~~~~~~~~~~~~~~~~~~~~~~~~

Django oferece muita funções utilitárias (em geral no módulo ``django.utils``)
que recebem uma string como primeiro argumento e fazem algo com esta string. 
Estas funções são usadas tanto por filtros de template, como de forma direta, em outras
partes do código.

Se você escrever suas próprias funções, similares a estas, e lidar com traduções,
pode surgir um problema quando o primeiro argumento é um objeto de tradução preguiçosa.
Em geral, este objeto não será convertido em uma string imediatamente, pois pode ser
que esta funções esteja seja usada fora de uma view (em, portanto, a configuração
de localidade da thread corrente não estará correta).

Nestes casos, deve-se usar o decorador ``django.utils.functional.allow_lazy()``. 
Ele modifica a funções de tal modo que*se* chamada com um objeto de tradução 
preguiçosa como primeiro argumento, a avaliação será postergada até que haja 
a necessidade de converter para uma string.

Por exemplo::

    from django.utils.functional import allow_lazy

    def funcao_utilitaria(s, ...):
        # Faz alguma conversão sobre a string 'x'
        # ...
    funcao_utilitaria = allow_lazy(funcao_utilitaria, unicode)
    
O decorador ``allow_lazy()`` recebe, além da função, um certo número de 
argumentos extras (``*args``)que especificam o(s) tipo(s) de retorno da
função original. É suficiente incluir ``unicode`` aqui e garantir que a
função retorna apenas strings Unicode.

Usar este decorador indica que a função escrita assume como input uma string simples,
porém, adiciona-se o suporte a objetos de tradução preguiçosa ao final.

2. Como criar arquivos de linguagem
===============================

Após marcar as strings para posterior tradução, deve-se escrever (ou obter)
as traduções em si. Segue como isto funciona.

.. admonition:: Restrições de localidade

    Django não consegue localizar uma aplicação em uma localidade para a qual
    o Django em si não tenha sido traduzida. Neste caso, os arquivos de tradução
    serão ignorados. Ao tentar-se fazer isto, inevitavelmente, haverá uma 
    mistura de strings traduzidas (vindas da aplicação) e strings em inglês
    (do Django em si). Para adicionar suporta a uma localidade ainda sem 
    tradução, é necessário traduzir uma parte mínima do core do Django.

Arquivos de mensagem
-------------

O primeiro passo é criar um *arquivo de mensagem* para uma nova linguagem.
Um arquivo de mensagem é um arquivo-texto, representando uma única linguage,
que contém todas as strings traduzidas disponíveis e como elas devem ser
representadas em uma dada linguagem. Arquivos de mensagem tem uma extensão
``.po``.

Django vem com uma ferramente, ``django-admin.py makemessages``, 
que automatiza a criação e manutenção desses arquivos. Para criar ou atualizar
um arquivo de mensagem, execute o seguinte comando::

    django-admin.py makemessages -l pt_BR

...onde ``pt_BR`` é o código da linguagem do arquivo de mensagem a cria-se.
O código da linaugem, neste caso, está em formato de localidade. Por exemplo,
``de`` é para alemão e ``ru`` para russo.


O script deverá ser executado de um dos locais abaixo:

*A pasta raíz do projeto Django
*A pasta raíz da aplicação Djanho
*A pasta raíz ``django`` (Não a pasta do checkout de Subversion, mas 
a pasta ligada via ``$PYTHONPATH`` ou localizada em algum local no caminho).
Isto só é relecante quando cria-se uma tradução para Django em si.

This script runs over your project source tree or your application source tree and
pulls out all strings marked for translation. It creates (or updates) a message
file in the directory ``locale/LANG/LC_MESSAGES``. In the ``de`` example, the
file will be ``locale/de/LC_MESSAGES/django.po``.

By default ``django-admin.py makemessages`` examines every file that has the
``.html`` file extension. In case you want to override that default, use the
``--extension`` or ``-e`` option to specify the file extensions to examine::

    django-admin.py makemessages -l de -e txt

Separate multiple extensions with commas and/or use ``-e`` or ``--extension``
multiple times::

    django-admin.py makemessages -l de -e html,txt -e xml

When creating JavaScript translation catalogs (which we'll cover later in this
chapter,) you need to use the special 'djangojs' domain, **not** ``-e js``.

.. admonition:: No gettext?

    If you don't have the ``gettext`` utilities installed, ``django-admin.py
    makemessages`` will create empty files. If that's the case, either install
    the ``gettext`` utilities or just copy the English message file
    (``locale/en/LC_MESSAGES/django.po``) if available and use it as a starting
    point; it's just an empty translation file.

.. admonition:: Working on Windows?

   If you're using Windows and need to install the GNU gettext utilities so
   ``django-admin makemessages`` works, see the "gettext on Windows" section
   below for more information.

The format of ``.po`` files is straightforward. Each ``.po`` file contains a
small bit of metadata, such as the translation maintainer's contact
information, but the bulk of the file is a list of *messages* -- simple
mappings between translation strings and the actual translated text for the
particular language.

For example, if your Django app contained a translation string for the text
``"Welcome to my site."``, like so::

    _("Welcome to my site.")

...then ``django-admin.py makemessages`` will have created a ``.po`` file
containing the following snippet -- a message::

    #: path/to/python/module.py:23
    msgid "Welcome to my site."
    msgstr ""

A quick explanation:

* ``msgid`` is the translation string, which appears in the source. Don't
  change it.
* ``msgstr`` is where you put the language-specific translation. It starts
  out empty, so it's your responsibility to change it. Make sure you keep
  the quotes around your translation.
* As a convenience, each message includes, in the form of a comment line
  prefixed with ``#`` and located above the ``msgid`` line, the filename and
  line number from which the translation string was gleaned.

Long messages are a special case. There, the first string directly after the
``msgstr`` (or ``msgid``) is an empty string. Then the content itself will be
written over the next few lines as one string per line. Those strings are
directly concatenated. Don't forget trailing spaces within the strings;
otherwise, they'll be tacked together without whitespace!

To reexamine all source code and templates for new translation strings and
update all message files for *all* languages, run this::

    django-admin.py makemessages -a

Compiling Message Files
-----------------------

After you create your message file -- and each time you make changes to it --
you'll need to compile it into a more efficient form, for use by ``gettext``.
Do this with the ``django-admin.py compilemessages`` utility.

This tool runs over all available ``.po`` files and creates ``.mo`` files, which
are binary files optimized for use by ``gettext``. In the same directory from
which you ran ``django-admin.py makemessages``, run ``django-admin.py
compilemessages`` like this::

   django-admin.py compilemessages

That's it. Your translations are ready for use.

3. How Django Discovers Language Preference
===========================================

Once you've prepared your translations -- or, if you just want to use the
translations that come with Django -- you'll just need to activate translation
for your app.

Behind the scenes, Django has a very flexible model of deciding which language
should be used -- installation-wide, for a particular user, or both.

To set an installation-wide language preference, set ``LANGUAGE_CODE``.
Django uses this language as the default translation -- the final attempt if no
other translator finds a translation.

If all you want to do is run Django with your native language, and a language
file is available for your language, all you need to do is set
``LANGUAGE_CODE``.

If you want to let each individual user specify which language he or she
prefers, use ``LocaleMiddleware``. ``LocaleMiddleware`` enables language
selection based on data from the request. It customizes content for each user.

To use ``LocaleMiddleware``, add ``'django.middleware.locale.LocaleMiddleware'``
to your ``MIDDLEWARE_CLASSES`` setting. Because middleware order matters, you
should follow these guidelines:

* Make sure it's one of the first middlewares installed.
* It should come after ``SessionMiddleware``, because ``LocaleMiddleware``
  makes use of session data.
* If you use ``CacheMiddleware``, put ``LocaleMiddleware`` after it.

For example, your ``MIDDLEWARE_CLASSES`` might look like this::

    MIDDLEWARE_CLASSES = (
       'django.contrib.sessions.middleware.SessionMiddleware',
       'django.middleware.locale.LocaleMiddleware',
       'django.middleware.common.CommonMiddleware',
    )

(For more on middleware, see Chapter 17.)

``LocaleMiddleware`` tries to determine the user's language preference by
following this algorithm:

* First, it looks for a ``django_language`` key in the current user's
  session.

* Failing that, it looks for a cookie.

* Failing that, it looks at the ``Accept-Language`` HTTP header. This
  header is sent by your browser and tells the server which language(s) you
  prefer, in order by priority. Django tries each language in the header
  until it finds one with available translations.

* Failing that, it uses the global ``LANGUAGE_CODE`` setting.

Notes:

* In each of these places, the language preference is expected to be in the
  standard language format, as a string. For example, Brazilian Portuguese
  is ``pt-br``.

* If a base language is available but the sublanguage specified is not,
  Django uses the base language. For example, if a user specifies ``de-at``
  (Austrian German) but Django only has ``de`` available, Django uses
  ``de``.

* Only languages listed in the ``LANGUAGES`` setting can be selected.
  If you want to restrict the language selection to a subset of provided
  languages (because your application doesn't provide all those languages),
  set ``LANGUAGES`` to a list of languages. For example::

      LANGUAGES = (
        ('de', _('German')),
        ('en', _('English')),
      )

  This example restricts languages that are available for automatic
  selection to German and English (and any sublanguage, like ``de-ch`` or
  ``en-us``).

* If you define a custom ``LANGUAGES`` setting, as explained in the
  previous bullet, it's OK to mark the languages as translation strings
  -- but use a "dummy" ``ugettext()`` function, not the one in
  ``django.utils.translation``. You should *never* import
  ``django.utils.translation`` from within your settings file, because that
  module in itself depends on the settings, and that would cause a circular
  import.

  The solution is to use a "dummy" ``ugettext()`` function. Here's a sample
  settings file::

      ugettext = lambda s: s

      LANGUAGES = (
          ('de', ugettext('German')),
          ('en', ugettext('English')),
      )

  With this arrangement, ``django-admin.py makemessages`` will still find
  and mark these strings for translation, but the translation won't happen
  at runtime -- so you'll have to remember to wrap the languages in the
  *real* ``ugettext()`` in any code that uses ``LANGUAGES`` at runtime.

* The ``LocaleMiddleware`` can only select languages for which there is a
  Django-provided base translation. If you want to provide translations
  for your application that aren't already in the set of translations
  in Django's source tree, you'll want to provide at least basic
  translations for that language. For example, Django uses technical
  message IDs to translate date formats and time formats -- so you will
  need at least those translations for the system to work correctly.

  A good starting point is to copy the English ``.po`` file and to
  translate at least the technical messages -- maybe the validation
  messages, too.

  Technical message IDs are easily recognized; they're all upper case. You
  don't translate the message ID as with other messages, you provide the
  correct local variant on the provided English value. For example, with
  ``DATETIME_FORMAT`` (or ``DATE_FORMAT`` or ``TIME_FORMAT``), this would
  be the format string that you want to use in your language. The format
  is identical to the format strings used by the ``now`` template tag.

Once ``LocaleMiddleware`` determines the user's preference, it makes this
preference available as ``request.LANGUAGE_CODE`` for each
``HttpRequest``. Feel free to read this value in your view
code. Here's a simple example::

    def hello_world(request):
        if request.LANGUAGE_CODE == 'de-at':
            return HttpResponse("You prefer to read Austrian German.")
        else:
            return HttpResponse("You prefer to read another language.")

Note that, with static (middleware-less) translation, the language is in
``settings.LANGUAGE_CODE``, while with dynamic (middleware) translation, it's
in ``request.LANGUAGE_CODE``.

Using Translations in Your Own Projects
=======================================

Django looks for translations by following this algorithm:

* First, it looks for a ``locale`` directory in the application directory
  of the view that's being called. If it finds a translation for the
  selected language, the translation will be installed.
* Next, it looks for a ``locale`` directory in the project directory. If it
  finds a translation, the translation will be installed.
* Finally, it checks the Django-provided base translation in
  ``django/conf/locale``.

This way, you can write applications that include their own translations, and
you can override base translations in your project path. Or, you can just build
a big project out of several apps and put all translations into one big project
message file. The choice is yours.

All message file repositories are structured the same way. They are:

* ``$APPPATH/locale/<language>/LC_MESSAGES/django.(po|mo)``
* ``$PROJECTPATH/locale/<language>/LC_MESSAGES/django.(po|mo)``
* All paths listed in ``LOCALE_PATHS`` in your settings file are
  searched in that order for ``<language>/LC_MESSAGES/django.(po|mo)``
* ``$PYTHONPATH/django/conf/locale/<language>/LC_MESSAGES/django.(po|mo)``

To create message files, you use the same ``django-admin.py makemessages``
tool as with the Django message files. You only need to be in the right place
-- in the directory where either the ``conf/locale`` (in case of the source
tree) or the ``locale/`` (in case of app messages or project messages)
directory are located. And you use the same ``django-admin.py compilemessages``
to produce the binary ``django.mo`` files that are used by ``gettext``.

You can also run ``django-admin.py compilemessages --settings=path.to.settings``
to make the compiler process all the directories in your ``LOCALE_PATHS``
setting.

Application message files are a bit complicated to discover -- they need the
``LocaleMiddleware``. If you don't use the middleware, only the Django message
files and project message files will be processed.

Finally, you should give some thought to the structure of your translation
files. If your applications need to be delivered to other users and will
be used in other projects, you might want to use app-specific translations.
But using app-specific translations and project translations could produce
weird problems with ``makemessages``: ``makemessages`` will traverse all
directories below the current path and so might put message IDs into the
project message file that are already in application message files.

The easiest way out is to store applications that are not part of the project
(and so carry their own translations) outside the project tree. That way,
``django-admin.py makemessages`` on the project level will only translate
strings that are connected to your explicit project and not strings that are
distributed independently.

The ``set_language`` Redirect View
==================================

As a convenience, Django comes with a view, ``django.views.i18n.set_language``,
that sets a user's language preference and redirects back to the previous page.

Activate this view by adding the following line to your URLconf::

    (r'^i18n/', include('django.conf.urls.i18n')),

(Note that this example makes the view available at ``/i18n/setlang/``.)

The view expects to be called via the ``POST`` method, with a ``language``
parameter set in request. If session support is enabled, the view
saves the language choice in the user's session. Otherwise, it saves the
language choice in a cookie that is by default named ``django_language``.
(The name can be changed through the ``LANGUAGE_COOKIE_NAME`` setting.)

After setting the language choice, Django redirects the user, following this
algorithm:

* Django looks for a ``next`` parameter in the ``POST`` data.
* If that doesn't exist, or is empty, Django tries the URL in the
  ``Referrer`` header.
* If that's empty -- say, if a user's browser suppresses that header --
  then the user will be redirected to ``/`` (the site root) as a fallback.

Here's example HTML template code::

    <form action="/i18n/setlang/" method="post">
    <input name="next" type="hidden" value="/next/page/" />
    <select name="language">
        {% for lang in LANGUAGES %}
        <option value="{{ lang.0 }}">{{ lang.1 }}</option>
        {% endfor %}
    </select>
    <input type="submit" value="Go" />
    </form>

Translations and JavaScript
===========================

Adding translations to JavaScript poses some problems:

* JavaScript code doesn't have access to a ``gettext`` implementation.

* JavaScript code doesn't have access to .po or .mo files; they need to be
  delivered by the server.

* The translation catalogs for JavaScript should be kept as small as
  possible.

Django provides an integrated solution for these problems: It passes the
translations into JavaScript, so you can call ``gettext``, etc., from within
JavaScript.

The ``javascript_catalog`` View
-------------------------------

The main solution to these problems is the ``javascript_catalog`` view, which
sends out a JavaScript code library with functions that mimic the ``gettext``
interface, plus an array of translation strings. Those translation strings are
taken from the application, project or Django core, according to what you
specify in either the info_dict or the URL.

You hook it up like this::

    js_info_dict = {
        'packages': ('your.app.package',),
    }

    urlpatterns = patterns('',
        (r'^jsi18n/$', 'django.views.i18n.javascript_catalog', js_info_dict),
    )

Each string in ``packages`` should be in Python dotted-package syntax (the
same format as the strings in ``INSTALLED_APPS``) and should refer to a package
that contains a ``locale`` directory. If you specify multiple packages, all
those catalogs are merged into one catalog. This is useful if you have
JavaScript that uses strings from different applications.

You can make the view dynamic by putting the packages into the URL pattern::

    urlpatterns = patterns('',
        (r'^jsi18n/(?P<packages>\S+)/$', 'django.views.i18n.javascript_catalog'),
    )

With this, you specify the packages as a list of package names delimited by '+'
signs in the URL. This is especially useful if your pages use code from
different apps and this changes often and you don't want to pull in one big
catalog file. As a security measure, these values can only be either
``django.conf`` or any package from the ``INSTALLED_APPS`` setting.

Using the JavaScript Translation Catalog
----------------------------------------

To use the catalog, just pull in the dynamically generated script like this::

    <script type="text/javascript" src="/path/to/jsi18n/"></script>

This is how the admin fetches the translation catalog from the server. When the
catalog is loaded, your JavaScript code can use the standard ``gettext``
interface to access it::

    document.write(gettext('this is to be translated'));

There is also an ``ngettext`` interface::

    var object_cnt = 1 // or 0, or 2, or 3, ...
    s = ngettext('literal for the singular case',
            'literal for the plural case', object_cnt);

and even a string interpolation function::

    function interpolate(fmt, obj, named);

The interpolation syntax is borrowed from Python, so the ``interpolate``
function supports both positional and named interpolation:

* Positional interpolation: ``obj`` contains a JavaScript Array object
  whose elements values are then sequentially interpolated in their
  corresponding ``fmt`` placeholders in the same order they appear.
  For example::

    fmts = ngettext('There is %s object. Remaining: %s',
            'There are %s objects. Remaining: %s', 11);
    s = interpolate(fmts, [11, 20]);
    // s is 'There are 11 objects. Remaining: 20'

* Named interpolation: This mode is selected by passing the optional
  boolean ``named`` parameter as true. ``obj`` contains a JavaScript
  object or associative array. For example::

    d = {
        count: 10
        total: 50
    };

    fmts = ngettext('Total: %(total)s, there is %(count)s object',
    'there are %(count)s of a total of %(total)s objects', d.count);
    s = interpolate(fmts, d, true);

You shouldn't go over the top with string interpolation, though: this is still
JavaScript, so the code has to make repeated regular-expression substitutions.
This isn't as fast as string interpolation in Python, so keep it to those
cases where you really need it (for example, in conjunction with ``ngettext``
to produce proper pluralizations).

Creating JavaScript Translation Catalogs
----------------------------------------

You create and update the translation catalogs the same way as the other

Django translation catalogs -- with the django-admin.py makemessages tool. The
only difference is you need to provide a ``-d djangojs`` parameter, like this::

    django-admin.py makemessages -d djangojs -l de

This would create or update the translation catalog for JavaScript for German.
After updating translation catalogs, just run ``django-admin.py compilemessages``
the same way as you do with normal Django translation catalogs.

Notes for Users Familiar with ``gettext``
=========================================

If you know ``gettext``, you might note these specialties in the way Django
does translation:

* The string domain is ``django`` or ``djangojs``. This string domain is
  used to differentiate between different programs that store their data
  in a common message-file library (usually ``/usr/share/locale/``). The
  ``django`` domain is used for python and template translation strings
  and is loaded into the global translation catalogs. The ``djangojs``
  domain is only used for JavaScript translation catalogs to make sure
  that those are as small as possible.
* Django doesn't use ``xgettext`` alone. It uses Python wrappers around
  ``xgettext`` and ``msgfmt``. This is mostly for convenience.

``gettext`` on Windows
======================

This is only needed for people who either want to extract message IDs or compile
message files (``.po``). Translation work itself just involves editing existing
files of this type, but if you want to create your own message files, or want to
test or compile a changed message file, you will need the ``gettext`` utilities:

* Download the following zip files from
  http://sourceforge.net/projects/gettext

  * ``gettext-runtime-X.bin.woe32.zip``
  * ``gettext-tools-X.bin.woe32.zip``
  * ``libiconv-X.bin.woe32.zip``

* Extract the 3 files in the same folder (i.e. ``C:\Program
  Files\gettext-utils``)

* Update the system PATH:

  * ``Control Panel > System > Advanced > Environment Variables``
  * In the ``System variables`` list, click ``Path``, click ``Edit``
  * Add ``;C:\Program Files\gettext-utils\bin`` at the end of the
    ``Variable value`` field

You may also use ``gettext`` binaries you have obtained elsewhere, so long as
the ``xgettext --version`` command works properly. Some version 0.14.4 binaries
have been found to not support this command. Do not attempt to use Django
translation utilities with a ``gettext`` package if the command ``xgettext
--version`` entered at a Windows command prompt causes a popup window saying
"xgettext.exe has generated errors and will be closed by Windows".

What's Next?
============

The `final chapter`_ focuses on security -- how you can help secure your sites and
your users from malicious attackers.

.. _final chapter: ../chapter20/
