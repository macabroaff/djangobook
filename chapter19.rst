================================
Capítulo 19: Internacionalização
================================

Django foi originalmente desenvolvido na região central dos Estados Unidos -- 
mais precisamente, em Lawrence, Kansas, sendo menos do que 64km do centro
geográfico do continente norte americano. Como a maioria dos projetos de código aberto,
a comunidade também cresceu e incluiu pessoas de todo o globo. Como a comunidade do Django 
está cada vez mais diversificada, a "internacionalização" e a "localização" 
se tornou cada vez mais relevante. Como muitos desenvolvedores tem um conhecimento
impreciso desses termos, nós iremos definí-los rapidamente.


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

* Ele permite desenvolvedores e autores de templates especificar quais
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

Especifica uma string de tradução usando a função ``ugettext()``. 
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
        

(O embargo de usar variáveis ou valores computados, como nos últimos dois
exemplos, é que o utilitário de detecção de strings de tradução do Django, 
``django-admin.py makemessages``, não será capz de encontrar essas strings.
Mais informações sobre ``makemessages`` mais a frente.)

As strings passadas para ``_()`` ou ``ugettext()`` podem receber parâmetros,
especificados com o padrão sintático de interpolação nome-string.
Exemplo::

    def my_view(request, m, d):
        output = _('Hoje é %(dia)s de %(mes)s.') % {'mes': m, 'dia': d}
        return HttpResponse(output)
        
Esta técnica permite que as traduções reordenem parâmetros do texto.
Por exemplo, em uma tradução inglesa poderia ser ``"Today is November 26."``
enquanto uma espanhola seria ``"Hoy es 26 de Noviembre."`` -- com os parâmetros
(mês e dia) com posições trocadas.

Por esta razão, pode-se usar a interpolação nome-string (e.g., ``%(day)s``)
ao invés da interpolação posicional (e.g., ``%s`` ou ``%d``) onde houver
mais de um parâmetro. Se for usada a interpolação posicional, traduções
não serão possíveis para parâmetros reordenados.

Marcando strings como no-op
~~~~~~~~~~~~~~~~~~~~~~~~

Use a função ``django.utils.translation.ugettext_noop()`` para marcar uma string
como string de tradução sem traduzí-la. A string é posteriormente traduzida de uma
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
não a tradução em si. A tradução será  feita quando a string for usada em um contexto
de string, como na rederização de um template no site do admin de Django.

O resultado de uma chamada a ``ugettext_lazy()`` pode ser usada onde você
usaria uma string unicode (um objeto do tipo ``unicode``) em Python. Se você
tentar usar isso no lugar de uma bytestring (um objeto ``str``), ocorrerá um
comportamento inesperado, pois um objeto ``ugettext_lazy()`` não sabe como se
converter para um bytestring. Também não é possível usar uma string unicode dentro
de uma bytestring, então isso é consistente como comportamento de Python. Por exemplo::
    
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

    class MyThing(models.Model):
        nome = models.CharField(texto_de_ajuda=_('Este é um texto de ajuda'))
        
Sempre use traduções preguiçosa em modelos de Django. Nomes de campos e nomes
de tabelas devem ser marcadas para tradução; caso contrário, eles não serão
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
de tradução no plural e o número de objetos (que é passado para as linguas de tradução
como a variável ``contador``).


Em templates
----------------

Tradução de templates Django usam duas tags de template e um sintaxe ligeiramente
diferente da usada em código Python. Para dar ao template acesso à essas tags, ponha
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
bloco de tradução.

    {% blocktrans with valor|filter as minhavariavel %}Isto terá {{ minhavariavel }} dentro.{% endblocktrans %}

Se for necessário ligar mais de uma expressão dentro de uma tag ``blocktrans``,
separe os pedações com ``and``::

    {% blocktrans with livro|titulo as livro_t and autor|titulo as autor_t %}
    Isto é {{ livro_t }} by {{ autor_t }}
    {% endblocktrans %}
    
Para pluralizar, especifique tanto as formas singular e plural com a
tag ``{% plural %}``, a qual aparece dentro de ``{% blocktrans %}`` e
``{% endblocktrans %}``. Por exemplo::

    {% blocktrans count lista|tamanho as contador %}
    Existe apenas um {{ none }} objeto.
    {% plural %}
    Existem {{ contador }} {{ nome }} objetos.
    {% endblocktrans %}

Internamente, todos os blocos e traduções inline usam a chamada apropriada
de ``ugettext`` / ``ungettext``.

Cada ``RequestContext`` tem acesso a três variáveis para traduções específicas:

* ``LANGUAGES`` é uma lista de tuplas nas quais o primeiro elemento é o código
  da linguagem e o segundo é o nome da linguagem (traduzido para a língua local).

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

As concatenações padrão de Python (``''.join([...])``) não funcionarão em listas que contém
objetos de tradução preguiçosa. No lugar delas, deve-se usar ``django.utils.translation.string_concat()``, que cria
um objeto preguiçoso que concatena seu conteúdo *e* o converte em strings
apenas quando o resultado é incluído em uma string. Por exemplo::

    from django.utils.translation import string_concat
    # ...
    nome = ugettext_lazy(u'John Lennon')
    instrumento = ugettext_lazy(u'guitarra')
    resultado = string_concat([nome, ': ', instrumento])
    
Aqui, as traduções preguiçosas em ``resultado`` irão apenas ser convertidas
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
que esta função esteja seja usada fora de uma view (e, portanto, a configuração
de localidade da thread corrente não estará correta).

Nestes casos, deve-se usar o decorador ``django.utils.functional.allow_lazy()``. 
Ele modifica a funções de tal modo que *se* chamada com um objeto de tradução 
preguiçosa como primeiro argumento, a avaliação será postergada até que haja 
a necessidade de converter para uma string.

Por exemplo::

    from django.utils.functional import allow_lazy

    def funcao_utilitaria(s, ...):
        # Faz alguma conversão sobre a string 'x'
        # ...
    funcao_utilitaria = allow_lazy(funcao_utilitaria, unicode)
    
O decorador ``allow_lazy()`` recebe, além da função, um certo número de 
argumentos extras (``*args``) que especificam o(s) tipo(s) de retorno da
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
    o Django em si não tenha sido traduzido. Neste caso, os arquivos de tradução
    serão ignorados. Ao tentar-se fazer isto, inevitavelmente, haverá uma 
    mistura de strings traduzidas (vindas da aplicação) e strings em inglês
    (do Django em si). Para adicionar suporte a uma localidade ainda sem 
    tradução, é necessário traduzir uma parte mínima do núcleo do Django.

Arquivos de mensagem
-------------

O primeiro passo é criar um *arquivo de mensagem* para uma nova linguagem.
Um arquivo de mensagem é um arquivo-texto, representando uma única linguagem,
que contém todas as strings traduzidas disponíveis e como elas devem ser
representadas em uma dada linguagem. Arquivos de mensagem tem uma extensão
``.po``.

Django vem com uma ferramenta, ``django-admin.py makemessages``, 
que automatiza a criação e manutenção desses arquivos. Para criar ou atualizar
um arquivo de mensagem, execute o seguinte comando::

    django-admin.py makemessages -l pt_BR

...onde ``pt_BR`` é o código da linguagem do arquivo de mensagem a criar-se.
O código da linguagem, neste caso, está em formato de localidade. Por exemplo,
``de`` é para alemão e ``ru`` para russo.


O script deverá ser executado de um dos locais abaixo:

* A pasta raíz do projeto Django.

* A pasta raíz da aplicação Django.

* A pasta raíz ``django`` (Não a pasta do checkout de Subversion, mas a pasta ligada via ``$PYTHONPATH`` ou localizada em algum local no caminho). Isto só é relecante quando cria-se uma tradução para Django em si.

Esse script percorre a árvore do projeto, ou da aplicação, e extrai todas as s
trings marcadas para tradução. Ele cria (ou atualiza) um arquivo de mensagem no 
diretório ``locale/LANG/LC_MESSAGES``. No exemplo ``de``, o arquivo será ``locale/de/LC_MESSAGES/django.po``

Por default, ``django-admin.py makemessages`` examina todos os arquivo que possuem extensão ``.html``.
Nesse caso, para sobrescrever esse comportamento, deve-se usar as opções 
``--extension`` ou ``-e`` para especificar as extensões a serem examinadas::

    django-admin.py makemessages -l de -e txt
    
Separe multiplas extensões com vírgulas e/ou use ``-e`` ou ``--extension``
várias vezes::

    django-admin.py makemessages -l de -e html,txt -e xml

Quando forem usados catálogos de tradução de JavaScript (os quais vamos abordar
mais tarde neste capítulo), deve-se usar o domínio especial 'djangojs', **não**
``-e js``.

.. admonition:: Nenhum gettext?

    Se as utilidades de ``gettext`` não estiverem instaladas, ``django-admin.pt
    makemessages`` irá criar arquivos vazios. Nesse caso, deve-se instalar as utilidades
    ou apenas copiar o arquivo de mensagem de Inglês (``locale/en/LC_MESSAGES/django.po``)
    ,se ele estiver disponível, e usá-lo como um ponto inicial; ele é apenas um arquivo
    de tradução vazio.

.. admonition:: Trabalhando no Windows?

   Quando estiver trabalhando no Windows, deve-se instalar as utilidades
   gettext do GNU para ``django-admin makemessages`` funcionar, veja a seção
   "gettext on Windows" abaixo para mais informações.
   
O formato dos arquivos ``.po`` é simples. Cada ``.po`` contém uma pequena quantidade
de metadata, como as informações de contato do administrador da tradução, mas
o grosso do arquivo é uma lista de *mensagens* -- simplesmente um mapeamento
entre as strings de tradução e a tradução em si, para determinada língua.

Por exemplo, se a aplicação Django contém uma string de tradução para o texto
``"Bem-vindo ao meu site."``, como::

    _("Bem-vindo ao meu site.")
    
...então ``django-admin.py makemessages`` irá criar um arquivo ``.po`` contendo o seguinte fragmento --
uma mensagem::

    #: path/to/python/module.py:23
    msgid "Bem-vindo ao meu site.."
    msgstr ""
    
Uma rápida explicação:

* ``msgid`` é uma string de tradução, que aparece no código. Não mude isso.
* ``msgstr`` é onde deve-se colocar a tradução em si. Inicialmente, ela está
  vazia, então, deve-se mudar isto. Certifique-se que as aspas continuam
  na tradução.
* Por conveniência, cada mensagem inclue, na forma de um comentário acima
  de ``msgid``, o nome do arquivo e o número da linha a partir do qual a string 
  de tradução foi adquirida.
  
Mensagens longas são um caso especial. Aqui, a primeira string após ``msgstr``
(ou ``msgid``) é vazia. Então, o conteúdo em si irá ser escrito sobre as próximas
linhas como uma string por linha. Essas strings são concatenas diretamente.
Não esqueça dos espaços dentro das strings; caso contrário, eles irão aparecer 
todos juntos, sem espaço em branco!


Para reexaminar todo código-fonte e templates para novas strings de tradução
e atualizar todos os arquivos de mensagem para *todas* as línguas, execute::

    django-admin.py makemessages -a

Compilando arquivos de mensagem
-----------------------

Após a criação de um arquivo de mensagem -- e a cada alteração deles -- é necessário
compilá-lo em uma forma mais eficiente, para o uso de ``gettext``. Isso é feito usando-se
a utilidade ``django-admin.py compilemessages``.

Essa ferramenta percorre todos os arquivos ``.po`` disponíveis e cria arquivos
``.mo``, que são arquivos binários otimizados para o uso de ``gettext``. No mesmo
diretório onde ``django-admin.py makemessages`` foi executado, `
deve-se executar ``django-admin.py compilemessages``, como abaixo::

   django-admin.py compilemessages

Pronto. As traduções estão prontas para uso.

3. Como descobrir as preferências de língua de Django
===========================================

Com as traduções preparadas -- ou, se forem ser usadas apenas as traduções
que vem com Django -- deve-se ativar a tradução para a aplicação.

Por baixo dos panos, Django possue um modelo muito flexível para decidir
qual língua deve ser usada -- ou instalada, para um usuário em particular,
ou ambas.

Para definir uma preferência linguística, deve-se alterar ``LANGUAGE_CODE``.
Django usa esta língua como tradução padrão -- a última alternativa se 
nenhum outro tradutor encontrar uma tradução.

Se você quiser apenas que Django execute usando tua língua nativa, e um
arquivo de língua está disponível para essa linguagem, basta alterar ``LANGUAGE_CODE``.

Se a língua deve ser definida pelas preferências de cada usuário, deve-se usar
``LocaleMiddleware``. Isso habilita a seleção de língua baseada nos dados da
requisição. Isso customiza o conteúdo para cada usuário.

Para usar ``LocaleMiddleware``, deve-se adicionar ``'django.middleware.locale.LocaleMiddleware'`` na configuração 
``MIDDLEWARE_CLASSES``. Como a ordem do middleware é relevante, deve-se seguir
as recomendações:

* Certifique-se que este é o primeiro middleware instalado.
* Ele deve vir após ``SessionMiddleware``, pois ``LocaleMiddleware`` faz uso
  dos dados de seção.
* Se usar ``CacheMiddleware``, deve-se por ``LocaleMiddleware`` após isso.

Por exemplo, ``MIDDLEWARE_CLASSES`` deve ser semelhante a::


    MIDDLEWARE_CLASSES = (
       'django.contrib.sessions.middleware.SessionMiddleware',
       'django.middleware.locale.LocaleMiddleware',
       'django.middleware.common.CommonMiddleware',
    )

(Para mais sobre middlewares, veja o Capítulo 17.)

``LocaleMiddleware`` tenta determinar a preferência de língua do usuário
através do seguinte algoritmo:

* Primeiramente, procura-se chave ``django_language`` na atual seção.

* Caso falhe, procura-se um cookie.

* Caso falhe, procura-se o cabeçalho HTTP ``Accept-Language``. 
  Este cabeçalho é enviado para o browser e diz ao servidor qual é a 
  linguagem preferencial, em ordem de prioridade. Django tenta cada língua
  no cabeçalho até encontrar uma para qual as traduções estejam disponíveis.

* Caso falhe, usa-se a configuração global ``LANGUAGE_CODE``.  

Notas:

* Em cada espaço, a preferência linguística é esperada no formato padrão
  de línguas, como uma string. Por exemplo, para Portugês brasileiro, ``pt-br``.
  
* Se uma linguagem base está disponível, mas a "sub-língua" não, Django
  usa a linguagem base. Por exemplo, se a preferência for por ``de-at``
  (Alemão austríaco), mas Django apenas possuir ``de``, Django usará ``de``.
  
* Apenas línguas listadas em ``LANGUAGES`` podem ser selecionadas.
  Se deseja-se restringir a seleção de línguas para um sub-conjunto das línguas
  disponíveis (pois a aplicação não suporta todas essas línguas), deve-se alterar
  ``LANGUAGES`` para uma lista de línguas. Por exemplo::
  
      LANGUAGES = (
        ('de', _('German')),
        ('en', _('English')),
      )
      
  Esse exemplo restringe para alemão e inglês as línguas disponíveis 
  para seleção automática (ou qualquer sub-língua, como ``de-ch`` ou ``en-us``).
  
* Se a configuração ``LANGUAGES`` for customizada, como explicado, é correto marcar
  as línguagens como strings de tradução -- mas use uma função "falsa" ``ugettext()``,
  não a função de ``django.utils.translation``. *Nunca* deve-se importar ``django.utils.translation`` para 
  dentro do arquivo de configuração, pois esse módulo em si depende das configuração, 
  e isso criaria um import circular.
  
  A solução é usar uma função ``ugettext()`` falsa. Segue uma amostra do
  arquivo de configurações::

      ugettext = lambda s: s

      LANGUAGES = (
          ('de', ugettext('German')),
          ('en', ugettext('English')),
      )

  Com essa disposição, ``django-admin.py makemessages`` ainda encontrará e marcará essas strings para tradução,
  mas a tradução não acontecerá em tempo de execução -- então, deve-se lembrar 
  de cobrir as línguas no ``ugettext()`` *real* em qualquer código que 
  usar ``LANGUAGE`` em tempo de execução.
  
* ``LocaleMiddleware`` pode apenas selecionar línguas para as quais há uma
  tradução base suportada pelo Django. Se deseja-se adicionar línguas à
  árvore de código de Django, deve-se prover, ao menos, traduções básicas
  para essa linguagem. Por exemplo, Django usa identificadores de mensagens
  técnicas para traduzir datas e horas -- então, deve-se prover ao menos essas
  traduções para que o sistema funcione corretamente.
  
  Um bom ponto de começo é copiar o arquivo ``.po`` em inglês e traduzir
  ao menos as mensagens técnicas -- talvez as mensagens de validação, também.
  
  Identificadores de mensagens técnicas são facilmente reconhecíveis;
  estão todos em maiúsculo. Essas mensagens são traduzidas de forma
  especial; deve-se informar a correta variante local no valor em inglês
  provido. Por exemplo, com ``DATETIME_FORMAT`` (ou ``DATE_FORMAT`` ou 
  ``TIME_FORMAT``), informaria-se o formato da string que deve ser usada
  na linguagem. Esse formato é idêntico para o formato de strings usados 
  pela tag de template ``now``.

Quando ``LocaleMiddleware`` determina as preferências do usuário, elas tornam-se
disponíveis como ``request.LANGUAGE_CODE`` para cada ``HttpResquest``. Esteja
livre para ler este valor nas views. Segue um exemplo::

    def hello_world(request):
        if request.LANGUAGE_CODE == 'de-at':
            return HttpResponse("Você prefere ler alemão austríaco.")
        else:
            return HttpResponse("Você prefere ler outra língua.")
            
Vale nota que, com traduções estáticas (sem middleware), a linguagem fica em
``settings.LANGUAGE_CODE``, enquanto que com tradução dinâmicas (middleware), 
elas ficam em ``request.LANGUAGE_CODE``.


Usando traduções em seus próprios projetos
=======================================

Django procura por traduções seguindo o algoritmo:

* Primeiramente, procura-se por um diretório ``locale`` no diretório da aplicação
  da view que está sendo chamada. Se uma tradução for encontrada para a linguagem
  selecionada, a tradução é instalada.

* Em seguida, procura-se por ``locale`` no diretório do projeto. Se encontrar
  uma tradução, ela é instalada.
  
* Por fim, checa-se as traduções de base providas por Django, em ``django/conf/locale``.
  ``django/conf/locale``.
  
Deste modo, pode-se escrever aplicações que incluem suas próprias traduções,
e pode-se sobreescrever traduções de base no path do projeto. Ou pode-se apenas
construir um grande projeto com diversas aplicações e por todas elas em um grande
arquivo de mensagem. A escolha fica com o desenvolvedor.

Todos os repositórios de arquivos de mensagem são estruturados da mesma
forma. Assim eles são:

* ``$APPPATH/locale/<lingua>/LC_MESSAGES/django.(po|mo)``
* ``$PROJECTPATH/locale/<lingua>/LC_MESSAGES/django.(po|mo)``
* Todos os caminhos listados em ``LOCALE_PATHS`` no arquivo de configurações
  são procurados em ordem por ``<lingua>/LC_MESSAGES/django.(po|mo)``
* ``$PYTHONPATH/django/conf/locale/<lingua>/LC_MESSAGES/django.(po|mo)``

Para criar arquivos de mensagem, deve-se usar a mesma ferramenta 
``django-admin.py makemessages`` como em arquivos de mensagem de Django.
Deve-se apenas estar no local correto -- no diretório onde o ``conf/locale``
(no caso de árvore de código) ou o ``locale/`` (no caso de mensagens de aplicação
ou de projeto) estão localizados. Usa-se o mesmo para produzir os arquivos binários
``django.mo`` para serem usados por ``gettext``.

Podem também executar ``django-admin.py compilemessages --settings=path.to.settings`` 
para compilar todos os diretórios na configuração ``LOCALE_PATHS``.

Arquivos de mensagem de aplicação são um pouco mais complicados de descobrir -- 
eles precisam do ``LocaleMiddleware``. Sem o uso do middleware, apenas os arquivos
de mensagem de Django e de projeto serão processados.

Para terminar, deve-se pensar na estruturação dos arquivos de tradução.
Se a aplicação precisa ser entregue a outros usuários e será usada em outros
projetos, deve-se usar tradução específicas para cada aplicação. Mas, ao usar
esse tipo de tradução juntamente com tradução de projeto pode causar problemas
estranhos com ``makemessages``: ``makemessages`` irá percorrer todos os diretórios
abaixo do caminho corrente, então pode por identificadores de mensagem dentro do arquivo
de mensagem do projeto que já está nos arquivos de mensagem da aplicação.


A saída mais simples é guardar aplicações que não são parte do projeto (e que,
portanto, possuem suas próprias traduções) fora da árvore do projeto. Assim,
``django-admin.py makemessages`` no nível do projeto irá traduzir apenas as
strings que estão conectadas com o projeto explicitamente e não strings que 
estão distribuídas independentemente.

A  view de redirecionamento ``set_language`` 
============================================

Por conveniência, Django vem com uma view, ``django.views.i18n.set_language``,
que configura uma linguagem preferencial do usuário e o redirecciona de volta
para a página anterior.

Ative esta view com a adição da seguinte linha em URLconf::

    (r'^i18n/', include('django.conf.urls.i18n')),

(Note que esse exemplo torna a view disponível em ``/i18n/setlang/``.) 

A view espera ser chamada via um método ``POST``, com um pârametro ``language``,
na requisição. Se o suporte de sessão estiver ativado, a view salva a escolha
de linguagem da seção do usuário. Do contrário, ela salva a linguagem escolhida
em um cookie que é, por padrão, nomeado ``django_language``. (O nome pode ser 
alterado através da configuração ``LANGUAGE_COOKIE_NAME``.)

Após a configuração da linguagem de escolha, Django redireciona o usuário,
seguindo o seguite algoritmo:

* Django procura por um parâmetro ``next`` na requisição ``POST``.
* Se não existir, ou estiver vazio, Django tenta a URL no cabeçalho
  ``Referrer``.
* Se estiver vazio -- caso o browser do usuário tenha suprimido o cabeçalho, e.g. --
  então o usuário será redirecionado para ``\`` (raíz do site).

Segue um exemplo de template HTML::

    <form action="/i18n/setlang/" method="post">
    <input name="next" type="hidden" value="/next/page/" />
    <select name="language">
        {% for lang in LANGUAGES %}
        <option value="{{ lang.0 }}">{{ lang.1 }}</option>
        {% endfor %}
    </select>
    <input type="submit" value="Go" />
    </form>

Traduções e Javascript
===========================

Adicionar traduções para JavaScript traz alguns problemas:

* Código JavaScript não tem acesso a implementação de ``gettext``.

* Código JavaScript não tem acesso aos arquivos .po e .mo; eles precisam
  ser entregues pelo servidor. 
  
* Catálogos de tradução para JavaScript precisam ser mantidos no menor
  tamanho possível.

Django fornece uma solução integrada para esses problemas: Ele passa as 
tradução para JavaScript, de modo que possa-se chamar ``gettext``, etc., 
a partir do código JavaScript.

A view ``javascript_catalog``
-------------------------------

A principal solução para esses problemas é a view ``javascript_catalog``, que
envia uma biblioteca Javascript com funções que simulam a interface de ``gettext``,
além de um array de strings de tradução. Essas strings são tomadas da aplicação,
do projeto ou de Django, de acordo com que a especificação contida 
em info_dict ou na URL.

O JavaScript deve ser semelhante a isto::

    js_info_dict = {
        'packages': ('your.app.package',),
    }

    urlpatterns = patterns('',
        (r'^jsi18n/$', 'django.views.i18n.javascript_catalog', js_info_dict),
    )
    
Cada string em ``packages`` deve estar na sintaxe dotted-package de Python
(o mesmo formato das strings em ``INSTALLED_APPS``) e deve referenciar
um pacote contendo um diretório ``locale``. Se forem especificados múltiplos
pacotes, todos os catálogos devem ser fundidos em apenas um catálogo. Isso
é útil quando tem-se Javascript que usa strings de diferentes aplicações.

Pode-se fazer a view dinâmica pondo os pacotes em um padrão de URL::

    urlpatterns = patterns('',
        (r'^jsi18n/(?P<packages>\S+)/$', 'django.views.i18n.javascript_catalog'),
    )
    
Com isto, especifica-se os pacotes como uma lista de nomes de pacotes delimitados
por '+' na URL. Isso é especialmente útil quando o código para diferentes aplicações
e de alta frequência de alterações e quando não se quiser usar um grande arquivo
de catálogo. Como medida de segurança, esses valores somente podem estar em 
``django.conf`` ou em qualquer pacotes da configuração ``INSTALLED_APPS`.

Usando o catálogo de tradução JavaScript
----------------------------------------

Para usar o catálogo, apenas coloca-se ele no script gerado dinamicamente assim::

    <script type="text/javascript" src="/path/to/jsi18n/"></script>

É assim que o administrador busca o catálogo de tradução do servidor. Quando este
catálogo é carregado, o código Javascript pode ser usar a interface de ``gettext`` 
padrão para acessá-lo::

    document.write(gettext('isto será traduzido'));
    
Também é possível usar a interface de ``ngettext``::

    var object_cnt = 1 // or 0, or 2, or 3, ...
    s = ngettext('literal para o caso singular',
            'literal para o caso plural', object_cnt);

e até uma função de interpolação de strings::

A sintaxe de interpolação vem de Python, de modo que ``interpolate`` suporta
tanto interpolação posicionais como de nome:

* Interpolação posicional: ``obj`` contém um objeto array de JavaScript
  o qual os elementos são então, sequencialmente, interpolados em seus espaços
  reservados ``fmt`` correspondentes na mesma ordem em que aparecem.
  Por exemplo::
  
    fmts = ngettext('Existe %s objeto. Restando: %s',
            'Existem %s objetos. Restando: %s', 11);
    s = interpolate(fmts, [11, 20]);
    // s é 'Existem 11 objetos. Restando: 20'

* Interpolação de nome: Esse modo é selecionado quando se passa um
  booleano opcional, chamado ``named``, como true. ``obj`` contém um
  objeto JavaScript ou um array associativo. Por exemplo::
  
    d = {
        count: 10
        total: 50
    };

    fmts = ngettext('Total: %(total)s, existe %(count)s object',
    'Existe %(count)s de um total de %(total)s objetos', d.count);
    s = interpolate(fmts, d, true);

Não deve-se ir sobre o topo com uma string de inteporlação: Isso ainda
é JavaScript, portanto, o código deve fazer substituições repetidas de 
expressões regulares. Isso não é tão rápido quando interpolação de Python,
então deve-se reservar esses casos para quando for realmente necessário (por
exemplos, um conjução com ``ngettext`` para produzir plularizações propriamente).

Criando catálogos de tradução JavaScript
----------------------------------------

Cria-se e atualiza-se catálogos de tradução de uma maneira ou de outra.

Catálogos de tradução de Django -- com a ferramenta django-admin.py makemessages.
A única diferença é a necessidade de prover um parâmetro ``-d djangojs``, desse jeito::

    django-admin.py makemessages -d djangojs -l de
    
Isso criaria ou atualizaria o catálogos de tradução de alemão para JavaScript.
Após atualizar os catálogos de tradução, apenas roda-se ``django-admin.py compilemessages`` 
do mesmo modo como é feito com catálogos normais de Django.


Notas para usuários familiares com ``gettext``
=========================================

Se você conheçe ``gettext``, deve-se notar que esses detalhes no modo de
tradução de Django:

* O domínio de string é ``django`` ou ``djangojs``. Esse domínio é
  usado para diferenciar diferentes programas que armazenam dados em
  uma biblioteca comum de arquivos de mensagem (normalmente ``/usr/share/locale/``).
  O domínio ``django`` é usado por strings de tradução de python e templates e é
  carregado nos catálogos de tradução global. O domínio ``djangojs`` somente é
  usado por catálogos de tradução JavaScript para garantir que eles fiquem o
  menor possível.
* Django não usa ``xgettext`` isoladamente. Ele usa wrappers de Python 
  em torno de ``xgettext`` e ``msgfmt``. Isso é feito principalmente por conveniência.
  
``gettext`` no Windows
======================

Isso é só é necessário quem quer extrair identificadores de mensagens
ou compilar arquivos de mensagem (``.po``). O trabalho de tradução por si só envolve
apenas edição de arquivos deste tipo, mas, caso queira-se criar arquivos de mensagem
próprios, ou testar ou compilar um arquivo alterado, será necessário as utilidades de
``gettext``:

* Baixe os seguintes arquivos de
  http://sourceforge.net/projects/gettext

  * ``gettext-runtime-X.bin.woe32.zip``
  * ``gettext-tools-X.bin.woe32.zip``
  * ``libiconv-X.bin.woe32.zip``

* Extraia os 3 arquivos no mesmo diretório (i.e.` `C:\Program Files\gettext-utils``)

* Atualize o PATH do sistema:

  * ``Control Panel > System > Advanced > Environment Variables``
  * Na lista ``System variables``, clique em ``Path``, e clique em ``Edit``
  * Adicione ``;C:\Program Files\gettext-utils\bin`` ao final do campo
    ``Variable value``
    
Pode-se também usar os binários de ``gettext`` obtidos em outros lugares, desde
que o comando ``xgettext --version`` funcione corretamente. Alguma versão 0.14.4
de binários não suportam esse comando. Não tente usar utilidades de tradução
de Django com um pacote ``gettext`` se o comando ``xgettext --version`` produzir
uma janela de popup que dizendo "xgettext.exe has generated errors and 
will be closed by Windows".

O que vem a seguir?
============

O `capítulo final`_ foca em segurança -- como pode-se proteger sites e usuários
de ataques maliciosos.

.. _capítulo final: ../chapter20/
