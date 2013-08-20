====================
Capítulo 4: Templates
====================

No capítulo anterior, você deve ter notado algo peculiar em como nós retornamos
o texto em nossos exemplos de views. Isto é, o HTML foi codificado diretamente
em nosso código Python, tal como::

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It is now %s.</body></html>" % now
        return HttpResponse(html)

Embora essa técnica seja conveniente para o propósito de demonstrar como as views
trabalham, não é uma boa idéia codificar HTML diretamente em suas views.
Aqui está o por que:

* Qualquer modificação no design da página requer modificação no código Python.
  O design de um site tende a mudar com mais freqüência do que o código Python
  subjacente, por isso seria coveniente se o design pude-se ser modificado sem
  precisar modificar o código Python.

* Escrever código Python e design HTML são duas diciplinas diferentes,
  e a maioria dos ambientes de desenvolvimento Web professional dividem essas
  responsabilidades entre pessoas distintas (ou departamentos distintos).
  Designers e codificadores HTML/CSS não devem necessitar editar código Python
  para fazer o seu trabalho.

* É mais eficiente se programadores trabalharem em código Python e designers
  em templates, ao mesmo tempo, ao em vez de uma pessoa esperar a outra
  terminar a edição de um simples arquivo que contém código Python e HTML.

Por essas razões, é muito mais limpo e sustentável separar o design da página
do próprio código Python. Nós podemos fazer isso com o *sistema de template* do Django,
que discutiremos neste capítulo.

Sistema básico de template
==========================

Um template Django é uma seqüência de texto que se destina a separar a
apresentação de um documento a partir dos seus dados. Um template define espaços
reservados e vários pedaços básicos de lógica (template tags) que determinam a forma
como o documento deve ser exibido. Normalmente, templates são usados para gerar HTML,
mas os templates do Django são capazes de gerar qualquer formato baseado em texto.

Vamos iniciar com um exemplo simples de template. Esse template Django descreve uma
página HTML que agradece a uma pessoa por realizar um pedido de uma empresa. Imagine
isso como uma carta formulário::

    <html>
    <head><title>Ordering notice</title></head>

    <body>

    <h1>Ordering notice</h1>

    <p>Dear {{ person_name }},</p>

    <p>Thanks for placing an order from {{ company }}. It's scheduled to
    ship on {{ ship_date|date:"F j, Y" }}.</p>

    <p>Here are the items you've ordered:</p>

    <ul>
    {% for item in item_list %}
        <li>{{ item }}</li>
    {% endfor %}
    </ul>

    {% if ordered_warranty %}
        <p>Your warranty information will be included in the packaging.</p>
    {% else %}
        <p>You didn't order a warranty, so you're on your own when
        the products inevitably stop working.</p>
    {% endif %}

    <p>Sincerely,<br />{{ company }}</p>

    </body>
    </html>

Este template é um HTML básico com algumas variáveis e seus template tags jogado os
valores para dentro. Vamos análisa-lo:

* Qualquer texto cercado por um par de chaves (e.g., ``{{ person_name }}``) é
  uma *váriavel*. Isto significa "insira o valor da váriavel com o nome dado."
  (Como podemos especificar os valores das váriaveis? Nós vamos chegar nisso em breve.)

* Qualquer texto contido entre chaves e sinais de porcento (e.g., ``{% if
  ordered_warranty %}``) é um *template tag*. A definição de tag é bastante
  amplo: uma tag apenas diz ao sistema de template para "fazer alguma coisa".

  Este exemplo de template contem uma tag ``for`` (``{% for item in item_list %}``)
  e uma tag ``if`` (``{% if ordered_warranty %}``).

  Uma tag ``for`` trabalha de forma semelhante a declaração ``for`` em Python,
  permitindo você fazer um laço sobre cada item em uma seqüência. Uma tag ``if``,
  como você pode esperar, age como uma declaração lógica "if". Neste caso
  particular, a tag verifica se o valor da váriavel ``ordered_warranty`` está
  ``True``. Se sim, o sistema de template exibirá tudo que está entre ``{% if ordered_warranty %}``
  e ``{% else %}``. Se não, o sistema de template exibirá
  tudo que está entre ``{% else %}`` e ``{% endif %}``. Perceba que o ``{% else
  %}`` é opcional.

* Finalmente, o segundo parágrafo deste template contém um exemplo de *filtro*,
  sendo a forma mais conveniente de alterar a formatação de uma váriavel.
  Neste exemplo, ``{{ ship_date|date:"F j, Y" }}``, nós estamos passando a váriavel
  ``ship_date`` para o filtro ``date``, dando ao filtro ``date`` os argumentos
  "F j, Y". O filtro ``date`` formata datas no formato passado, como especificado
  pelo argumento. Os filtros são anexados usando o character pipe (``|``), como
  referência aos pipes do Unix.

Cada template Django tem acesso a vários tags e filtros embutidos, muitos dos
quais são discutidos nas sessões que seguem. No apêndice F contém a lista completa
de tags e filtros, e é uma boa idéia você se familiarizar com essa lista, assim
saberá quais as possíbilidades. Também é possível criar os seus próprios filtros
e tags; nós vamos cobrir isso no capítulo 9.


Usando o sistema de templates
=============================

Agora vamos mergulhar no sistema de templates do Django para que você veja como
funciona - mas nós ainda não vamos integrar com as views criadas no capítulo
anterior. Nosso objetivo aqui é mostrar para você como o sistema funciona de
forma idependente do restante do Django. (Dito de outra forma: geralmente você
usará o sistema de template dentro de uma view do Django, mas nós queremos deixar
claro que o sistema de template é somente uma biblioteca Python que você pode usar
em *qualquer lugar*, não somente nas views do Django).

Aqui está a maneira mais básica que você pode usar o sistema de templates do
Django em código Python:

1. Crie um objeto ``Template`` fornecendo  *******the raw template code*******
   como uma string.

2. Chame o método ``render()`` do objeto ``Template`` com um determinado
   conjunto de váriaveis (o *contexto*). Isto retorna  o template completamente
   renderizado como uma string, com todas as váriaveis e template tags
   avaliadas de acordo com o contexto.

Em código, é assim que se parece::

    >>> from django import template
    >>> t = template.Template('My name is {{ name }}.')
    >>> c = template.Context({'name': 'Adrian'})
    >>> print t.render(c)
    My name is Adrian.
    >>> c = template.Context({'name': 'Fred'})
    >>> print t.render(c)
    My name is Fred.

As sessões seguintes descrevem cada etapa com muito mais detalhe.

Criando objetos Template
-------------------------

O caminho mais fácil para criar um objeto ``Template`` é instância-lo diretamente.
A classe ``Template`` está no módulo ``django.template``, e o construtor tem um
argumento, o raw template code. Vamos mergulhar no interpretador interativo do Python
para ver como isto funciona no código.

Apartir do diretorio ``mysite`` criado por ``django-admin.py startproject`` (como
descrito no capítulo 2), digite ``python manage.py shell`` para iniciar o interpretador
interativo.

.. admonition::  Um prompt Python especial

    Se você anteriormente usou Python, você pode estar se perguntando porque
    estamos executando ``python manage.py shell`` ao invés de apenas ``python``.
    Ambos os comandos iniciam o interpretador interativo, mas o comando ``manage.py shell``
    possui uma diferença chave: antes de iniciar o interpretador, ele informa ao Django
    qual arquivo de configuração usar. Muitas partes do Django, incluindo o sistema de
    template, dependem de suas configurações, e você não conseguirá usá-los, a menos
    que o framework saiba quais configurações usar.

    Se você está curioso, aqui está como funciona por detrás das cenas. O Django
    procura por uma variável de ambiente chamada ``DJANGO_SETTINGS_MODULE``, que deve
    ser definido para o caminho de importação do seu ``settings.py``. Por exemplo,
    ``DJANGO_SETTINGS_MODULE`` deve ser definido como ``'mysite.settings'``, assumindo
    que ``mysite`` está no seu caminho Python.

    Quando você executa ``python manage.py shell``, o comando se preocupa em definir
    a variável ``DJANGO_SETTINGS_MODULE`` para você. Nós estamos encorajando você a usar
    ``python manage.py shell`` nestes exemplos, de modo que minimize a quantidade de ajustes
    e configurações que você deva fazer.

Vamos passar por alguns princípios básicos do sistema de template::

    >>> from django.template import Template
    >>> t = Template('My name is {{ name }}.')
    >>> print t

Se você está seguindo a forma interativa, você vai ver algo como isso::

    <django.template.Template object at 0xb7d5f24c>

O ``0xb7d5f24c`` será diferente toda vez, e isso não é relevante; é algo do
Python (a "identidade" Python do objeto ``Template``, se você precisar saber).

Quando você cria um objeto ``Template``, o sistema de template compila o código
do template cru em uma forma otimizada, pronta para renderização. Mas se o código
do seu template possuir qualquer erro de sintaxe, a chamada de ``Template()`` irá
causar uma exceção ``TemplateSyntaxError``::

    >>> from django.template import Template
    >>> t = Template('{% notatag %}')
    Traceback (most recent call last):
      File "<stdin>", line 1, in ?
      ...
    django.template.TemplateSyntaxError: Invalid block tag: 'notatag'

O termo "block tag" aqui se refere a ``{% notatag %}``. "Block tag" e
"template tag" são sinônimos.

O sistema gera uma exceção ``TemplateSyntaxError`` para qualquer um dos seguintes
casos:

* Tags inválidas
* Argumentos inválidos para tags válidas
* Filtros inválidos
* Argumentos inválidos para filtros válidos
* Sintaxe de template inválido
* Tags não fechadas (para tags que requerem fechamento)

Processando um template
--------------------

Uma vez que você tenha um objeto de ``Template``, você pode passar os
dados, dando-lhe um *contexto*. Um contexto é uma simples definição de
nomes de váriaveis e seus valores associados. Um template usa isto para
popular as váriaveis e avaliar as tags.

Um contexto é representado no Django pela classe ``Context``, a qual está
no módulo ``django.template``. Seu construtor tem um argumento optional:
***a dictionary mapping variable names to variable values***. Chame o método
``render()`` do objeto ``Template`` com o contexto para "preencher" o template::

    >>> from django.template import Context, Template
    >>> t = Template('My name is {{ name }}.')
    >>> c = Context({'name': 'Stephane'})
    >>> t.render(c)
    u'My name is Stephane.'

Uma coisa que devemos salientar, é que o valor de retorno de ``t.render(c)``
é um objeto Unicode -- não uma string normal Python. Você pode tratar isto
pelo uso do ``u`` em frente a string. Django usa objetos Unicode ao invés de
strings normais em seu framework. Se você entende a repercurssão disso, seja
grato pelas coisas sofisticadas que o Django faz nos bastidores para isto funcionar.
Se você não entende a repercussão disso, não se preocupe agora; apenas entenda que
o Unicode do Django torna simples que os seus aplicativos tenham suporte a uma grande
variedade de conjuntos de caracteres além do básico "A-Z" da língua Inglesa.

.. admonition:: Dicionários e contextos

   Um dicionário Python é um mapeamento entre chaves conhecidas
   e valores váriaveis. Um ``Context`` é similar ao dicionário, mas
   o ``Context`` possui uma funcionalidade adicional, como descrito
   no capítulo 9.

Nomes de váriaveis devem iniciar com letras (A-Z or a-z)  podem contem
mais letras, digitos, sublinhados e pontos (Pontos são um caso especial, vamos ver em breve).
Nomes de váriaves são case sensitive.

Aqui está um exemplo de modelo de compilação e renderização, usando um template
semelhante ao exemplo no início deste capítulo::

    >>> from django.template import Template, Context
    >>> raw_template = """<p>Dear {{ person_name }},</p>
    ...
    ... <p>Thanks for placing an order from {{ company }}. It's scheduled to
    ... ship on {{ ship_date|date:"F j, Y" }}.</p>
    ...
    ... {% if ordered_warranty %}
    ... <p>Your warranty information will be included in the packaging.</p>
    ... {% else %}
    ... <p>You didn't order a warranty, so you're on your own when
    ... the products inevitably stop working.</p>
    ... {% endif %}
    ...
    ... <p>Sincerely,<br />{{ company }}</p>"""
    >>> t = Template(raw_template)
    >>> import datetime
    >>> c = Context({'person_name': 'John Smith',
    ...     'company': 'Outdoor Equipment',
    ...     'ship_date': datetime.date(2009, 4, 2),
    ...     'ordered_warranty': False})
    >>> t.render(c)
    u"<p>Dear John Smith,</p>\n\n<p>Thanks for placing an order from Outdoor
    Equipment. It's scheduled to\nship on April 2, 2009.</p>\n\n\n<p>You
    didn't order a warranty, so you're on your own when\nthe products
    inevitably stop working.</p>\n\n\n<p>Sincerely,<br />Outdoor Equipment
    </p>"

Vamos passar as instruções de código uma por vez:

* Primeiro, nós importamos as classes ``Template`` e ``Context``, ambas
  ficam nó módulo ``django.template``.

* Nós salvamos o texto bruto do nosso template na váriavel
  ``raw_template``. Perceba que usamos aspas triplas para definir a string,
  porque envolve várias linhas; em contraste, strings com aspas simples não
  podem ser usadas em multiplas linhas.

* Em seguida, nós criamos o objeto template, ``t``, passando ``raw_template``
  para o construtor da classe ``Template`` .

* Nós importamos o módulo ``datetime`` da biblioteca padrão do Python,
  porque vamos precisar dele na declaração seguinte.

* Depois, criamos um objeto ``Context``, ``c``. O construtor ``Context``
  recebe um dicionário Python, que mapeia os nomes das váriaveis para valores.
  Aqui, por exemplo, nós especificamos que ``person_name`` é  ``'John Smith'``,
  ``company`` é ``'Outdoor Equipment'``, e assim por diante.

* Finalmente, chamamos o método ``render()`` em seu objeto template, passando
  o contexto. Este retorna o template renderizado, ou seja, ele substitui
  as váriaveis do template com os valores reais das váriaveis, e executa
  as tags de template.

  Note que o páragrafro "Você não solicitou garantia" é exibido porque
  a váriavel ``ordered_warranty`` tem seu valor como ``False``. Além
  disso, observer a data, ``April 2, 2009``, que é exibido de acordo com
  o formato da string ``'F j, Y'``. (Vamos explicar a formatação de strings
  para os filtros ``date`` em breve).

  Se você é novo com Python, você deve estar se perguntado porque incluir
  caracteres de nova linha(``'\n'``) ao invés de exibir as quebras de linhas.
  Isso está acontecendo por causa de uma detalhe no interpretador interativo
  do Python: a chamada para ``t.render(c)``, retorna uma string, e por padrão
  o interpretador interativo exibe a *representação* da string, ao invés do
  valor impresso na string. Se deseja ver a string com quebras de linha
  verdadeiramente, ao invés de dos caracteres ``'\n'`` , use a declaração
  ``print`` : ``print t.render(c)``.

Esses são os fundamentos para usar o sistema de templates do Django: basta
escrever um template string, criar um objeto ``Template``, criar um ``Context``,
e chamar o método ``render()``.

Múltiplos contextos, mesmo template
--------------------------------

Uma vez que você tem um objeto ``Template``, você pode processar múltiplos
contextos por ele. Por exemplo::

    >>> from django.template import Template, Context
    >>> t = Template('Hello, {{ name }}')
    >>> print t.render(Context({'name': 'John'}))
    Hello, John
    >>> print t.render(Context({'name': 'Julie'}))
    Hello, Julie
    >>> print t.render(Context({'name': 'Pat'}))
    Hello, Pat

Sempre que você está usando o mesmo código de template para renderizar
multiplos contextos, como isso, é mais eficiente criar o objeto
``Template`` *uma vez*, e depois chamar o ``render()`` por várias vezes::

    # Ruim
    for name in ('John', 'Julie', 'Pat'):
        t = Template('Olá, {{ name }}')
        print t.render(Context({'name': name}))

    # Bom
    t = Template('Olá, {{ name }}')
    for name in ('John', 'Julie', 'Pat'):
        print t.render(Context({'name': name}))

A análise de templates do Django é bastante rápida. Nos bastidores, a maior
parte da análise acontece através da chamada a uma única expressão regular.
Isso é um contraste gritante com as engines de template baseadas em XML, o qual
provoca uma sobrecarga ao parser XML e tendem a ser na ordem de magnitude mais
lentos que a engine de renderização de template do Django.

Pesquisa váriavel de contexto
-----------------------------

Nos exemplos até agora, passamos valores simples nos contextos -- na maior parte
strings, álem de um exemplo com ```datetime.date``. No entanto, o sistema de
template manipula de forma elegante estruturas de dados mais complexas, como
listas, dicionários e objetos personalizados.

A chave para percorer estruturas complexas de dados nos templates Django é
o caracter ponto (``.``). Use o ponto para acessar as chaves do dicionário,
atributos, métodos ou índices em um objeto.

Isso é melhor ilustrado com alguns exemplos. Por exemplo, suponha que
você está passando um dicionário Python a um template. Para acessar o
valor desse dicionário por chave de dicionário, use o ponto::

    >>> from django.template import Template, Context
    >>> person = {'name': 'Sally', 'age': '43'}
    >>> t = Template('{{ person.name }} is {{ person.age }} years old.')
    >>> c = Context({'person': person})
    >>> t.render(c)
    u'Sally is 43 years old.'

Da mesma forma, pontos também permitem o acesso a atributos de objetos. Por
exemplo, um objeto Python ``datetime.date`` possui atributos ``year``, ``month``
e ``day``, e você pode usar o ponto para acessar esses atributos em um template
Django::

    >>> from django.template import Template, Context
    >>> import datetime
    >>> d = datetime.date(1993, 5, 2)
    >>> d.year
    1993
    >>> d.month
    5
    >>> d.day
    2
    >>> t = Template('The month is {{ date.month }} and the year is {{ date.year }}.')
    >>> c = Context({'date': d})
    >>> t.render(c)
    u'The month is 5 and the year is 1993.'

Esse exemplo usa uma classe customizada, demonstrando que pontos váriaveis
também permitem o acesso a objetos arbitrários::

    >>> from django.template import Template, Context
    >>> class Person(object):
    ...     def __init__(self, first_name, last_name):
    ...         self.first_name, self.last_name = first_name, last_name
    >>> t = Template('Hello, {{ person.first_name }} {{ person.last_name }}.')
    >>> c = Context({'person': Person('John', 'Smith')})
    >>> t.render(c)
    u'Hello, John Smith.'

Pontos também podem remeter a *métodos* em objetos. Por exemplo, cada string
Python tem os métodos ``upper()`` e ``isdigit()``, e você pode chama-los
nos templates Django usando a mesma sintaxe do ponto::

    >>> from django.template import Template, Context
    >>> t = Template('{{ var }} -- {{ var.upper }} -- {{ var.isdigit }}')
    >>> t.render(Context({'var': 'hello'}))
    u'hello -- HELLO -- False'
    >>> t.render(Context({'var': '123'}))
    u'123 -- 123 -- True'

Perceba que você *não* incluiu parenteses na chamada do método. Além disso,
não é possível passar argumentos para os métodos, você só pode chamar
métodos que não tem argumentos requeridos (Nós explicáremos essa filosofia
adiante nesse cápitulo).

Finalizando, pontos são usados também para acessar índices de listas, por exemplo::

    >>> from django.template import Template, Context
    >>> t = Template('Item 2 is {{ items.2 }}.')
    >>> c = Context({'items': ['apples', 'bananas', 'carrots']})
    >>> t.render(c)
    u'Item 2 is carrots.'

Índices negativos em listas não são permitidos. Por exemplo, a váriavel
de template ``{{ items.-1 }}`` causará um ``TemplateSyntaxError``.

.. admonition:: Listas Python

   Um lembrete: listas Python possuem índices baseados em 0. O primeiro item é
   o índice 0, o segundo é o índice 1 e assim por diante.

Pesquisa por ponto pode ser resumida assim: quando o sistema de template
encontra um ponto em nome de váriavel, ele tenta as pesquisas a seguir, nesta
ordem:

* Pesquisa de dicionário (ex. ``foo["bar"]``)
* Pesquisa de atributo (ex. ``foo.bar``)
* Chamada de método  (ex. ``foo.bar()``)
* Pesquisa em índice de lista (ex. ``foo[2]``)

O sistema usa o primeiro tipo de pesquisa que funcionar. É um circuito lógico
curto.

Pesquisa por ponto podem ser aninhados em vários níveis de profundidade. Por
exemplo, o exemplo a seguir usa ``{{ person.name.upper }}``, que se traduz
em uma pesquisa de dicionário (``person['name']``) e depois em uma chamada
de método (``upper()``)::

    >>> from django.template import Template, Context
    >>> person = {'name': 'Sally', 'age': '43'}
    >>> t = Template('{{ person.name.upper }} is {{ person.age }} years old.')
    >>> c = Context({'person': person})
    >>> t.render(c)
    u'SALLY is 43 years old.'

Comportamento para chamada de método
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Chamada de métodos são levemente mais complexa do que outros tipos de pesquisa.
Aqui estão algumas coisas que devemos ter em mente:

* Se, durante a pesquisa de método, o método escapar uma exceção, a exceção
  será propagada, a não ser que a exceção tenha um atributo ``silent_variable_failure``
  cujo o valor seja ``True``. Se a exceção naõ tem um atributo ``silent_variable_failure``,
  a váriavel vai renderizar uma string vazia, por exemplo::

        >>> t = Template("My name is {{ person.first_name }}.")
        >>> class PersonClass3:
        ...     def first_name(self):
        ...         raise AssertionError, "foo"
        >>> p = PersonClass3()
        >>> t.render(Context({"person": p}))
        Traceback (most recent call last):
        ...
        AssertionError: foo

        >>> class SilentAssertionError(AssertionError):
        ...     silent_variable_failure = True
        >>> class PersonClass4:
        ...     def first_name(self):
        ...         raise SilentAssertionError
        >>> p = PersonClass4()
        >>> t.render(Context({"person": p}))
        u'My name is .'

* Uma chamada de métodos funcionará se o método não tenha argumentos
  requeridos. Caso contrário, o sistema irá para o próximo tipo de pesquisa
  (pesquisa em índice de lista).

* Obviamente, alguns métodos tem efeitos colaterais, e seria insensato e
  uma possível falha de segurança, permitir que o sistema de template pudesse
  acessá-los.

  Digamos, por exemplo, você tem um objeto ``BankAccount`` que tem um método
  ``delete()``. Se o template inclui algo como ``{{ account.delete }}``,
  onde ``account`` é um objeto ``BankAccount``, o objeto seria excluído
  quando o template for renderizado!

  Para previnir isso, defina o atributo ``alters_data`` no método::

      def delete(self):
          # Excluí um conta
      delete.alters_data = True

  O sistema de template não irá executar metodos marcados dessa maneira.
  Continuando exemplo acima, se o template incluir ``{{ account.delete }}``
  e o método ``delete()`` tem o ``alters_data=True``, então o método
  ``delete()` não será executado quando o template é renderizado. Ao invés
  disso, ele irá falhar silenciosamente.

Como váriaveis inválidas são tratadas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Por padrão, se uma váriavel não existe, o sistema de templates mostra ela como
uma string vazia, falhando silenciosamente. Por exemplo::

    >>> from django.template import Template, Context
    >>> t = Template('Your name is {{ name }}.')
    >>> t.render(Context())
    u'Your name is .'
    >>> t.render(Context({'var': 'hello'}))
    u'Your name is .'
    >>> t.render(Context({'NAME': 'hello'}))
    u'Your name is .'
    >>> t.render(Context({'Name': 'hello'}))
    u'Your name is .'

O sistema falha silenciosamente, ao invés de levantar uma exceção porque
ele entende ser resiliente a um erro humano. Nesse caso, todas as pesquisas
falharam porque os nomes das váriaveis foram escritas com o tamanho ou nome
na forma errada. No mundo real, é inaceitaǘel para um web site tornar-se
inacessível devido a um pequeno erro de sintaxe em template.

Jogando com objetos de contexto
-------------------------------

Na maioria das vezes, você vai instanciar objetos ``Context`` passando um
dicionário totalmente preenchido para ``Context()``. Mas você pode adicionar
e excluir items de um objeto ``Context`` uma vez que estanciado, também, usando
a sintaxe padrão de dicionários Python::

    >>> from django.template import Context
    >>> c = Context({"foo": "bar"})
    >>> c['foo']
    'bar'
    >>> del c['foo']
    >>> c['foo']
    Traceback (most recent call last):
      ...
    KeyError: 'foo'
    >>> c['newvariable'] = 'hello'
    >>> c['newvariable']
    'hello'

Básico de Template Tags e Filtros
=================================

Como já mencionado, the template system ships with built-in tags and
filters. As seções seguintes fornecem um resumo das tags e filtros mais
comuns.

Tags
----

if/else
~~~~~~~

A tag ``{% if %}`` avalia uma váriavel e se a váriavel é "True" (ou seja,
ela existe, não está vazia e não é um valor booleano falso), o sistema
irá exibir tudo entre ``{% if %}`` e ``{% endif %}``, por example::

    {% if today_is_weekend %}
        <p>Welcome to the weekend!</p>
    {% endif %}

E a tag ``{% else %}`` é opcional::

    {% if today_is_weekend %}
        <p>Welcome to the weekend!</p>
    {% else %}
        <p>Get back to work.</p>
    {% endif %}

.. admonition:: Python "Truthiness"

   Em Python e no sistema de template do Django, estes objetos apresentam
   valor ``False`` em um contexto booleano::

   * Uma lista vazia (``[]``)
   * Uma tupla vazia (``()``)
   * Um dicionário vazio (``{}``)
   * Uma string vazia (``''``)
   * Zero (``0``)
   * O objeto especial ``None``
   * O objeto ``False`` (obviamente)
   * Objetos customizados que definem seu próprio contexto de comportamento booleano (isso é um uso avançado do Python)

   Todo o resto é avaliado com ``True``.

A tag ``{% if %}`` aceita ``and``, ``or`` ou ``not`` para testar multiplas
váriaveis ou para negar uma determinada váriavel. Por exemplo::

    {% if athlete_list and coach_list %}
        Ambos os atletas e treinadores estão disponíveis.
    {% endif %}

    {% if not athlete_list %}
        Não existem atletas.
    {% endif %}

    {% if athlete_list or coach_list %}
        Existem alguns atletas ou treinadores.
    {% endif %}

    {% if not athlete_list or coach_list %}
        Não existem atletas ou existem alguns treinadores.
    {% endif %}

    {% if athlete_list and not coach_list %}
        Existem alguns atletas e absulutamente nenhum treinador.
    {% endif %}

Tags ``{% if %}`` não permitem cláusulas ``and`` e ``or`` juntas,
porque a ordem da lógica pode ser ambigua. Por exemplo, isso é inválido::

    {% if athlete_list and coach_list or cheerleader_list %}

O uso de parênteses para controlar a ordem das operações não é suportado. Se
você achar que precisa de parênteses, considere a realização da lógica fora do
template e passe o resultado disso em uma variável de template dedicada. Ou,
apenas use tags ``{% if %}`` aninhadas, como isso::

    {% if athlete_list %}
        {% if coach_list or cheerleader_list %}
            Nós temos atletas, e treinadores ou líderes de torcida!
        {% endif %}
    {% endif %}

Multiplo uso de mesmo operador lógica é bom, mas você não pode combinar
diferentes operadores. Por exemplo, isso é válido::

    {% if athlete_list or coach_list or parent_list or teacher_list %}

Não há tag ``{% elif %}``. Use tags aninhadas ``{% if %}`` para realizar a
mesma coisa::

    {% if athlete_list %}
        <p>Here are the athletes: {{ athlete_list }}.</p>
    {% else %}
        <p>No athletes are available.</p>
        {% if coach_list %}
            <p>Here are the coaches: {{ coach_list }}.</p>
        {% endif %}
    {% endif %}

Certifique-se de que fechou cada ``{% if %}`` com um ``{% endif %}``. Senão,
o Django irá lançar um ``TemplateSyntaxError``.

for
~~~

A tag ``{% for %}`` permite você fazer loop sobre cada item em uma sequência.
Como na declaração ``for`` em Python, a sintaxe é ``for X in Y``, onde ``Y`` é
a sequência para ser passada pelo loop e ``X`` é o nome da variável a ser usada para
um ciclo particular do loop. Cada vez que passar pelo loop, o sistema de template irá
exibir tudo entre ``{% for %}`` e ``{% endfor %}``.

Por exemplo, você pode usar o seguinte para exibir um lista de atletas dada a
variável ``athlete_list``::

    <ul>
    {% for athlete in athlete_list %}
        <li>{{ athlete.name }}</li>
    {% endfor %}
    </ul>

E ``reversed`` para marcar o loop sobre a lista no sentido inverso::

    {% for athlete in athlete_list reversed %}
    ...
    {% endfor %}

É possível aninhar tags ``{% for %}``::

    {% for athlete in athlete_list %}
        <h1>{{ athlete.name }}</h1>
        <ul>
        {% for sport in athlete.sports_played %}
            <li>{{ sport }}</li>
        {% endfor %}
        </ul>
    {% endfor %}

Um padrão comum é verificar o tamanho da lista antes de fazer o looping
sobre ela e produzir um texto especial, se a lista é vazia::

    {% if athlete_list %}
        {% for athlete in athlete_list %}
            <p>{{ athlete.name }}</p>
        {% endfor %}
    {% else %}
        <p>There are no athletes. Only computer programmers.</p>
    {% endif %}

Devido esse padrão ser bastante comum, a tag ``for`` suporta uma cláusula
opcional ``{% empty %}``, que permite você definir o que será exibido se
a lista é vazia. Este exemplo é equivalente ao anterior::

    {% for athlete in athlete_list %}
        <p>{{ athlete.name }}</p>
    {% empty %}
        <p>There are no athletes. Only computer programmers.</p>
    {% endfor %}

Não existe suporte para "sair (breaking out)" em um laço antes do laço ser concluído.
Se você quer fazer isso, altere a variável que está em looping de forma que
contenha apenas os valores que você deseja varrer. Da mesma forma, não há
suporte para a declaração "continue", que instrue o processo de laço voltar
imediatamente para para o laço (Veja a seção "Filosofia e limitações" mais
tarde nesse capítulo para compreender o raciocínio por trás dessa decisão
de design).

Dentro de cada laço ``{% for %}``, você tem acesso a variável de template chamada
``forloop``. Essa variável tem atributos que lhe dão informações sobre o progresso
do laço:

* ``forloop.counter`` é sempre definido como um inteiro que representa
  o número de vezes que loop foi inserido. Este é indexado como um,
  então a primeira passada através do laço, ``forloop.counter`` será
  setado como ``1``. Aqui está um exemplo::

    {% for item in todo_list %}
        <p>{{ forloop.counter }}: {{ item }}</p>
    {% endfor %}

* ``forloop.counter0`` é como ``forloop.counter``, exceto que é indexado
  como zero. Seu valor será  setado como ``0`` na primeira vez que o laço
  passar.

* ``forloop.revcounter`` é sempre definido como um inteiro representando
  o número restante de itens no laço. A primeira vez através do laço,
  ``forloop.revcounter`` será definido o número totoal de itens na
  sequência que você está atravessando. A ultima iteração do laço,
  ``forloop.revcounter`` será definido como ``1``.

* ``forloop.revcounter0`` é como ``forloop.revcounter``, exceto que é
  indexado como zero. A primeira interação do loop, ``forloop.revcounter0``
  será setado o número de elementos da sequência menos 1. A ultima iteração
  do laço, será definido como ``0``.

* ``forloop.first`` é um valor booleano definido como ``True`` se está é a
  primeira iteração do laço. Isso é conveniente para casos especiais::

    {% for object in objects %}
        {% if forloop.first %}<li class="first">{% else %}<li>{% endif %}
        {{ object }}
        </li>
    {% endfor %}

* ``forloop.last`` é um valor booleano definido como ``True`` se está for a
  ultima iteração do laço. Um uso comum para isso, é colocar caracteres de
  tabulação entre uma lista de links::

    {% for link in links %}{{ link }}{% if not forloop.last %} | {% endif %}{% endfor %}

  O código do template acima pode imprimir algo assim::

    Link1 | Link2 | Link3 | Link4

  Outro uso comum para isso é colocar vírgula entre palavras em uma lista::

    Favorite places:
    {% for p in places %}{{ p }}{% if not forloop.last %}, {% endif %}{% endfor %}

* ``forloop.parentloop`` é uma referência ao objeto ``forloop`` para o
  laço *pai*, em caso de laços aninhados. Abaixo um exemplo::

    {% for country in countries %}
        <table>
        {% for city in country.city_list %}
            <tr>
                <td>Country #{{ forloop.parentloop.counter }}</td>
                <td>City #{{ forloop.counter }}</td>
                <td>{{ city }}</td>
            </tr>
        {% endfor %}
        </table>
    {% endfor %}

A magia da variável ``forloop`` está disponível dentro do laço. Depois de
o analizador de templates atingir ``{% endfor %}``, ``forloop`` desaparece.

.. admonition:: Contexto e a variável forloop

    Dentro do bloco ``{% for %}``, as variáveis existentes são
    movidas para fora do caminho evitando sobrescrever a magia
    da váriavel ``forloop``. Django expõe este contexto movido
    em ``forloop.parentloop``. Você geralmente não precisa se
    preocupar com isso, mas se você fornecer uma variável de
    template chamada ``forloop`` (embora nós tenhamos aconselhado
    contra), ele vai ser nomeado ``forloop.parentloop`` enquanto
    dentro do bloco ``{% for %}``.

ifequal/ifnotequal
~~~~~~~~~~~~~~~~~~

O sistema de template do Django deliberadamente  não é uma linguagem de
programação completa e portanto não é permite que vocẽ execute declarações
arbitrárias Python (Mais informações sobre esta idéia na seção "Filosofias
e limitações"). No entanto, é muito comum requisitar que o template compare
dois valores e exiba algo se eles forem iguais -- e o Django fornece uma tag
``{% ifequal %}`` para esse fim.

A tag ``{% ifequal %}`` compara dois valores e exibe tudo entre ``{% ifequal %}``
e ``{% endifequal %}`` se os valores são iguais.

Esse exemplo compara as variáveis de template ``user`` e ``currentuser``::

    {% ifequal user currentuser %}
        <h1>Welcome!</h1>
    {% endifequal %}

Os argumentos podem ser strings em código fixo, com aspas simples ou duplas,
então o seguinte é válido::

    {% ifequal section 'sitenews' %}
        <h1>Site News</h1>
    {% endifequal %}

    {% ifequal section "community" %}
        <h1>Community</h1>
    {% endifequal %}

Assim como ``{% if %}``, a tag ``{% ifequal %}`` tem suporte opcional a tag
``{% else %}``::

    {% ifequal section 'sitenews' %}
        <h1>Site News</h1>
    {% else %}
        <h1>No News Here</h1>
    {% endifequal %}

Apenas variáveis de template, strings, números inteiros e decimais são permitidos
como argumetos para ``{% ifequal %}``. Estes são exemplos válidos::

    {% ifequal variable 1 %}
    {% ifequal variable 1.23 %}
    {% ifequal variable 'foo' %}
    {% ifequal variable "foo" %}

Quaisquer outros tipos de variáveis, tais como dicionários Python, listas ou
booleanos, não pode ser codificados em ``{% ifequal %}``. Estes são exemplos válidos::

    {% ifequal variable True %}
    {% ifequal variable [1, 2, 3] %}
    {% ifequal variable {'key': 'value'} %}

Se você precisa testar se algo é verdadeiro ou falso, use a tag ``{% if %}``
em vez de ``{% ifequal %}``.

Comentários
~~~~~~~~

Assim como em HTML ou Python, a linguagem de template do Django permite
comentários. Para designar um comentário, use ``{# #}``::

    {# This is a comment #}

O comentário não será emitido quando o modelo é processado.

Comentários usando essa sintaxe não podem ocupar várias linhas. Esta limitação
melhora o desempenho análise do template. No template a seguir, a saída processada
será exatamente igual ao template, ou seja, a tag de comentário não será analizada
como um comentário::

    This is a {# this is not
    a comment #}
    test.

Se você quiser usar comentários em várias linhas, use o template tag ``{% comment %}``,
dessa forma::

    {% comment %}
    This is a
    multi-line comment.
    {% endcomment %}

Filtros
-------

Como explicado anteriormente nesse capítulo, filtros de template são caminhos
simples para alterar os valores de variáveis antes que sejam exibidas. Filtros
usam o caracter pipe, dessa forma::

    {{ name|lower }}

Isso exibe o valor da variável ``{{ name }}`` depois de ser filtrada através
do filtro ``lower``, que converte o texto para letras minúsculas.

Filtros podem ser *acorrentados*, ou seja, eles podem ser usados em conjunto
de tal modo que a saída de um filtro é aplicado ao seguinte. Aqui um exemplo
que pega o primeiro elemento em uma lista e converte para letras minúsculas::

    {{ my_list|first|upper }}

Alguns filtros devem ter argumentos. O argumento para o filtros deve vir
após dois pontos e estar sempre entre aspas duplas. Por exemplo::

    {{ bio|truncatewords:"30" }}

Isso exibe as 30 primeiras palavras da váriavel ``bio``.

A seguir estão alguns dos filtros mais importantes. Apêndice E cobre o resto.

* ``addslashes``: Adiciona contrabarra antes de alguma contrabarra, aspas
  simples ou aspas duplas. Isso é útil se o texto produzido é incluído em
  um string Javascript.

* ``date``: Formata objeto ``date`` ou ``datetime`` de acordo com a string
  de formatação passada no parâmetro, por exemplo::

      {{ pub_date|date:"F j, Y" }}

  Formatação de strings são definidas no Apêndice E.

* ``length``: Retorna o comprimento do valor. Para lista, este retorna o número
  de elementos. Para string, este retorna o número de caracteres (Expecialistas em
  Python, lembrem-se de que isso funciona em qualque objeto Python que saiba como
  determinar o seu comprimento -- ex. qualquer objeto que tenha o
  método ``__len__()``).

Filosofia e limitações
============================

Agora que você ja tem uma idéia sobre a linguagem de template do Django, devemos
destacar algumas de suas limitações intencionais, juntamente com algumas filosofias
sobre porque funciona da maneira que funciona.

Mais do que qualquer outro componente de aplicação Web, sintaxe de template é
muito subjetiva e as opiniões do programadores variam muito. Fato é que o Python
possui dezenas, se não centenas, de implementações de linguagem de templates em
código aberto dando suporte a isso. Cada uma que foi criada deve-se ao fato de que
o desenvolvedor cosiderava as linguagens existentes inadequadas (Na verdade, diz-se
ser um rito de passagem desenvolvedores Python escrever a sua própria linguagem de
template! Se você não tiver feito isso ainda, considere. É um exercicio divertido).

Com isso em mente, você pode estar interessado em saber que o Django não requer que
você utilize a sua linguagem de template. Como o Django se destina a ser o Web
framework full-stack que fornece todas as partes necessárias para desenvolvedores
Web serem produtivos, muitas vezes é *mais conveniente* usar o sistema de template
do Django do que outra biblioteca de templates Python, mas não é uma obrigação
restrita em qualquer sentido. Como vocẽ verá na próxima seção "Usando templates na
visão", é muito fácil usar outra linguagem de templates com o Django.

Assim, é claro que temos uma forte preferência pela forma como a linguagem de
templates do Django funciona. O sistema de templates possui raizes na forma como
o desenvolvimento Web é feito no mundo online e combina a experiência dos criadores
do Django. Aqui estão algumas dessas filosofias:

* *Lógica de negócios deve ser separada da lógica de apresentação*.
  Desenvolvedores Django enchergam o sistema de templates como uma ferramenta
  de controle da apresentação e apresentação relacionada com lógica -- e é isso.
  O sistema de templates não deve suportar funcionalidades que vão além dos
  seus objetivos básicos.

  Por essa razão, é impossível chamar código Python diretamente dentro
  de templates Django. Toda a "programação" é limitada fundamentalmente no
  escopo do que as tags de template podem fazer. Isso *é* possível escrevendo
  template tags personalizadas que fazem coisas arbitrárias, mas o out-of-the-box
  template tags do Django não permite a execução de código arbitrário Python.

* *Sintaxe deve ser desacoplada de HTML/XML*. Embora o sistema de templates
  do Django é usado para produzir principalmente HTML, ele tem a intenção
  de ser útil em formatos não HTML, como texto simples. Algumas outras
  linguagens de templates são baseadas em XML, colocam todas á lógica de
  template dento de tags XML ou atributos, mas o Django evita essa limitação
  deliberadamente. Exigir XML válido para escrever templates introduz um
  mundo de erros humanos e mensagens de erro difíceis de entender, e usando
  uma engine XML para analisar templates incorre em um nível inaceitável
  de sobrecarga no processamento do template.

* *Designers são assumidamente mais confortáveis com código HTML*. O sistema
  de templates não foi projetado para ser necessáriamente exibindo de maneira
  agradável em editores WYSIWYG como o Dreamweaver. Isso é uma limitação muito
  grave e não permite que a sintaxe seja amigável como ela é. Django expera que
  os autores de templates estejam confortáveis editando diretamento HTML.

* *Designers são assumidamente não programadores Python*. Os autores do sistema
  de templates reconhecem que templates de páginas web são mais frequentemente
  escritas por *designers*, não *programadores* e portanto não devem assumir
  conhecimento em Python.

  No entanto, o sistema também tem a intenção de acomodar pequenas equipes
  em que os templates *são* criados por programadores Python. Ele oferece
  uma maneira de extender a sintaxe do sistema, escrevendo em código Python puro
  (Mais sobre isso no capítulo 9).

* *O objetivo é não inventar uma linguagem de programação*. O objetivo é de
  oferecer apenas o suficiente de funcionalidades de programação, como branching e
  looping, que é essencial para tomada de decisões relacionada a apresentação.


Usando templates na visão
=========================

Você já aprendeu o básico do uso do sistema de template, agora vamos usar esse
conhecimento para criar uma view. Lembre-se da view ``current_datetime`` em
``mysite.views``, que nós começamos no capítulo anterior. Aqui está o que se
parece::

    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>It is now %s.</body></html>" % now
        return HttpResponse(html)

Vamos modificar essa view para usar o sistema de template do Django. De primeira,
você poderia pensar em fazer algo assim::

    from django.template import Template, Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        t = Template("<html><body>It is now {{ current_date }}.</body></html>")
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

Certo, isso usa o sistema de template, mas não resolver os problemas que nós
apontamos no introdução deste capítulo. Ou seja, o template está ainda incorporado
ao código Python, então a separação real de dados e apresentação não é alcaçada.
Vamos corrigir isso, colocando o template em um *arquivo separado*, que essa
view irá carregar.

Você pode considerar primeiro salvar o seu template em algum lugar do
seu sistema de arquivos e usar o Python built-in com a funcionalidade de abertura
de arquivos para ler o conteúdo do template. Aqui está como deve parecer, assumindo
que o template está salvo no arquivo ``/home/djangouser/templates/mytemplate.html``::

    from django.template import Template, Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        # Simple way of using templates from the filesystem.
        # This is BAD because it doesn't account for missing files!
        fp = open('/home/djangouser/templates/mytemplate.html')
        t = Template(fp.read())
        fp.close()
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

Esta abordagem, no entanto, é deselegante pelas seguintes razões::

* Ela não lida com o caso de um arquivo que está faltando. Se o arquivo
  ``mytemplate.html`` não existe ou não pode ser lido, a chamada ``open()``
  irá levantar uma excessão ``IOError``.

* Ela usa códigos fixos na localização do seu template. Se você estivesse usando
  essa técnica para cada view function, você estaria duplicando a localização
  do template. Sem mencionar que envolve bastante digitação!

* Ela incluí um série de código clichê chato. Você tem coisa melhor para
  fazer do que escrever chamadas para ``open()``, ``fp.read()`` e ``fp.close()``
  cada vez que você carrega um template.

Para resolver estas questões, nós vamos usar *carregamento de template* e
*diretório de template*.

Carregando Template
===================

Django fornece uma API conveniente e poderosa para carregamento de templates
apartir do sistema de arquivos, com o objetivo de remover a redundância, tanto na
chamada do carregamento de template e bem como em seus templates propriamente
ditos.

Para poder utilizar esse API de carregamento de template, primeiro você
precisa dizer ao framework onde você armazena seus templates. O local para
fazer isso é em seu arquivo de configurações -- o arquivo ``settings.py``
que mencionamos no último capítulo, quando nós introduzimos a configuração
``ROOT_URLCONF``.

Se voce está acompanhando, abra seu ``settings.py`` e encontre a
configuração ``TEMPLATE_DIRS``. Por padrão, é uma tupla vazia, provavelmente
contendo alguns comentários gerados automaticamente::

    TEMPLATE_DIRS = (
        # Put strings here, like "/home/html/django_templates" or "C:/www/django/templates".
        # Always use forward slashes, even on Windows.
        # Don't forget to use absolute paths, not relative paths.
    )

Essa configuração informa o mecanismo de carregamento de template do Django onde
procurar por templates. Escolha o diretório onde você gostaria de armazenar seus templates
e adicione ele ao ``TEMPLATE_DIRS``, dessa forma::

    TEMPLATE_DIRS = (
        '/home/django/mysite/templates',
    )

Existem algumas coisas para serem observadas:

* Você pode especificar qualquer diretório que você deseja, contanto que
  o diretório e os templates dentro desse diretório possam ser lidos pela
  conta de usuário sob o qual o servidor web é executado. Se você não pensa
  em um local apropriado para colocar seus templates, nós recomendamos a
  criação de um diretório ``templates`` dentro de seu projeto, ou seja,
  dentro do diretório ``mysite`` criador por você no capítulo 2.

* Se o seu ``TEMPLATE_DIRS`` contém somente um diretório, não esqueça da
  vírgula ao final da string do diretório!

  Ruim::

      # Faltando vírgula!
      TEMPLATE_DIRS = (
          '/home/django/mysite/templates'
      )

  Bom::

      # Vírgula corretamente no lugar.
      TEMPLATE_DIRS = (
          '/home/django/mysite/templates',
      )

  O motivo para isso é que o Python exige vírgula dentro de tuplas com
  elemento unico para remover ambigüidade da tupla de uma expressão entre
  parênteses. Isso é uma pegadinha para novatos muito comum.

* Se você está no Windows, inclua a letra da unidade e use o estilo
  Unix de barras ao invés de barras invertidas, como segue::

      TEMPLATE_DIRS = (
          'C:/www/django/templates',
      )

* É mais simples usar caminhos absolutos, ou seja, caminhos de diretório
  que começam na raiz do sistema de arquivos. Se você quiser ser mais
  flexível, dissociado, você pode tirar vantagem do fato de que os arquivos
  de configuração do Django são apenas código Python empregados para contrução
  dos conteúdos de ``TEMPLATE_DIRS`` dinâmicamente. Por exemplo::

      import os.path

      TEMPLATE_DIRS = (
          os.path.join(os.path.dirname(__file__), 'templates').replace('\\','/'),
      )

  Esse exemplo usa a variável mágica Python ``__file__``, que é automaticamente
  definida para o nome do arquivo do módulo Python em que o código vive. Ela
  recebe o nome do diretório que contém``settings.py`` (``os.path.dirname``),
  depois junta com ``templates`` em um caminho cross-platform (``os.path.join``),
  então assegura que tudo usa barras ao invés de barras invertidas (no
  caso do Windows).

  Enquanto estamos no tópico de código Python dinâmico em arquivos de
  configuraçôes, devemos salientar que é muito importante evitar erros
  Python em seu arquivo de configuração. Se você introduzir erro de sintaxe,
  ou erro de execução, seu site em Django provavelmente vai falhar.

Com a definição de ``TEMPLATE_DIRS``, o próximo passo é modificar o código
da view para usar a funcionalidade de carregamento de template do Django,
em vez de embutir códigos fixos nos caminhos de template. Voltando à nossa
view ``current_datetime``, vamos modificar para algo assim::

    from django.template.loader import get_template
    from django.template import Context
    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        t = get_template('current_datetime.html')
        html = t.render(Context({'current_date': now}))
        return HttpResponse(html)

Neste exemplo, estamos usando a função
``django.template.loader.get_template()`` em vez de carregar o template do
sistema de arquivo manualmente. A função ``get_template()`` recebe o nome do
template como argumento, descobre onde o template vive no sistema de arquivo,
abre o arquivo e devolve um objeto ``Template`` compilado.

Nosso template nesse exemplo é  ``current_datetime.html``, mas não há nada
especial sobre a extensão ``.html``. Você pode passar em seus templates qualquer
extensão que faça sentido para sua aplicação, ou você pode deixar de fora as
extensões completamente.

Para determinar a localização do template em seu sistema de arquivo,
``get_template()`` combina seus diretórios de template apartir de ``TEMPLATE_DIRS``
com o nome do template que você passou para ``get_template()``. Por exemplo, se
seu ``TEMPLATE_DIRS`` está definido para ``'/home/django/mysite/templates'``, a
chamada acima ``get_template()`` ficaria para o template de
``/home/django/mysite/templates/current_datetime.html``.

Se ``get_template()`` não encontrar o template com o nome informado, ele levantará
uma exceção ``TemplateDoesNotExist``. Para ver como se parece, dispare o servidor
de desenvolvimento do Django novamente rodando ``python manage.py runserver``
dentro do diretório do seu projeto Django. Depois, aponte seu browser para a página
que ativa a view ``current_datetime`` (por exemplo, ``http://127.0.0.1:8000/time/``).
Assumindo que a sua configuração ``DEBUG`` está definida como ``True`` e você
ainda não tenha criado um template ``current_datetime.html``, você deverá ver uma
página de erro Django destacando o erro ``TemplateDoesNotExist``.

.. figure:: graphics/chapter04/missing_template.png
   :alt: Screenshot of a "TemplateDoesNotExist" error.

   Figura 4-1: A página de erro é mostrada quando um template não pode ser encontrado.

Esta página de erro é semelhante ao que explanamos no Capítulo 3, com um pedaço
adicional de informação de debugging: uma seção "Template-loader postmortem".
Esta seção informa você quais templates Django tentou-se carregar, juntamente
com o motivo de cada tentativa fracassada (por exemplo, "File does not exist").
Esta informação é inestimável quando você está tentando depurar erros no carregamento
de template.

Continuando, crie o arquivo ``current_datetime.html`` dentro seu diretório
de templates usando o seguinte código de template::

    <html><body>It is now {{ current_date }}.</body></html>

Atualize a página em seu web browser e você deverá ver a renderização completa
da página.

render()
--------

Nós mostramos para você como carregar um template, preencher um ``Context``
e retornar um objeto ``HttpResponse`` com os resultados da renderização do
template. Nós otimizamos para usar ``get_template()`` ao invés de códigos
fixos em templates e caminhos de templates. Mas ainda requer uma boa quantidade
de codificação para fazer essas coisas. Devido ser um idioma comum, Django
fornece um atalho que permite você carregar um template, renderizar e retornar
um ``HttpResponse`` -- tudo em uma linha de código.

Esse atalho é uma função chamada ``render()``, que vive no módulo
``django.shortcuts``. Na maioria das vezes, você estará usando
``render()`` em vez de carregar templates e criar objetos ``Context``
e ``HttpResponse`` manualmente -- a não ser que seu chefe julgue o seu trabalho
pelo total de linhas de código escritas, então tudo bem.

Aqui está a continuação do exemplo ``current_datetime``, reescrito para usar
``render()``::

    from django.shortcuts import render
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        return render(request, 'current_datetime.html', {'current_date': now})

Que diferença! Vamos percorrer as alterações de código:

* Não temos mais que importar ``get_template``, ``Template``, ``Context``,
  ou ``HttpResponse``. Em vez disso, importamos
  ``django.shortcuts.render``. O ``import datetime`` permanece.

* Dentro da função ``current_datetime``, nós ainda calculamos ``now``, mas
  o carregamento de template, criação do contexto, renderização do template e
  a criação ``HttpResponse`` são todos realizados pela chamada ``render()``.
  Pois ``render()`` retorna um objeto ``HttpResponse``, podemos simplesmente
  ``retornar`` este valor na view.

O primeiro argumento de ``render()`` é a requisição, o segundo é o nome do
template que será usado. O terceiro argumento, se informado, deve ser um
dicionário para ser usado na criação de um um ``Context`` para este template.
Se você não prover um terceiro argumento, ``render()``usará-se um dicionário vazio.

Subdiretórios em get_template()
-------------------------------

Ele pode ficar pesado para armazenar todos os seus templates em um unico diretório.
Você pode gostar de armazenar os templates em subdiretórios em seu diretório de
template, e isso é bom. Na verdade, recomendamos faze-lo; alguns recursos mais
avançados do Django (tais como o sistema de views genéricas, que cobrimos no
Capítulo 11) expera esse layout de template como uma convenção padrão.

Armazerar templates em subdiretórios de seu diretório de templates é fácil.
Em suas chamadas para ``get_template()``, apenas inclua
o nome do subdiretório e uma barra antes do nome do template, assim::

    t = get_template('dateapp/current_datetime.html')

Devido ``render()`` ser um pequeno invólucro em torno de ``get_template()``,
você pode fazer a mesma coisa com o segundo argumento para ``render()``,
assim::

    return render(request, 'dateapp/current_datetime.html', {'current_date': now})

Não há limites para a profundidade da sua árvore de diretórios. Sinta-se a
vontade para usar a quantidade de subdiretórios que desejar.

.. note::

    Usuários Windows, certifiquem-se de utilizar barras ao invés de barras
    invertidas. ``get_template()`` assume o estilo Unix de designação
    do nome de arquivo.

A template tag ``include``
--------------------------

Agora que nós cobrimos o mecanismo de carregamento de templates, podemos
introduzir uma tag integrada ao template para tirar vantagem disso: ``{% include %}``.
Essa tag permite você incluir conteúdo de outro template. O argumento para a
tag deve ser o nome do template a ser incluído e o nome do template pode ser tanto
uma váriavel ou uma string fixa (entre aspas), podendo ser aspas simples ou duplas.
Quando você tem o mesmo código em vários templates, considere
o uso de um ``{% include %}`` para remover a duplicação.

Estes dois exemplos incluem o conteúdo do template ``nav.html``. Os
exemplos são equivalentes e ilustram que tanto aspas simples ou duplas são
permitidas:

    {% include 'nav.html' %}
    {% include "nav.html" %}

Esse exemplo incluí o conteúdo do template ``includes/nav.html``:

    {% include 'includes/nav.html' %}

Esse exemplo incluí o conteúdo do template cujo o nome está contido na
váriavel ``template_name``::

    {% include template_name %}

Como em ``get_template()``, o nome do arquivo do template é determinado pela
adição do diretório de template do ``TEMPLATE_DIRS`` para o nome de template
solicitado.

Templates incluídos são avaliados com o contexto dos templates que os incluí.
Por exemplo, considere estes dois templates::

    # mypage.html

    <html>
    <body>
    {% include "includes/nav.html" %}
    <h1>{{ title }}</h1>
    </body>
    </html>

    # includes/nav.html

    <div id="nav">
        You are in: {{ current_section }}
    </div>

Se você exibir ``mypage.html`` com um contexto contendo ``current_section``,
então a variável estará disponível no template "incluído", como seria de
esperar.

Se, em uma tag ``{% include %}``, o template com o nome passado não é encontrado,
o Django fará umda das duas coisas:

* Se ``DEBUG`` está definido como ``True``, você verá a exceção
  ``TemplateDoesNotExist`` em uma página de erro do Django.

* Se ``DEBUG`` está definido como ``False``, a tag falhará
  silenciosamente, exibindo nada no lugar da tag.


Herança de Template
==================

Nosso template exemplo até agora tem sido pequenos snippets HTML, mas no
mundo real, você usará o sistema de template do Django para criar páginas
HTML completas. Isso leva a um problema comum de desenvolvimento web: em
um web site, como reduzir a duplicação e a redundância de áreas em páginas
que são comuns, bem como a navegação no site inteiro?

Uma forma clássica de resolução desse problema é usar *server-sides
includes*, diretivas que você pode colocar junto a suas páginas HTML
para "incluir" uma página web dentro de outra. De fato, Django suporta
essa possibilidade, com a tag template ``{% include %}`` que acabamos de
descrever. Mas a forma preferida de resolução desse problema com Django
é usar uma estratégia mais elegante chamada *template inheritance*.

Na essência, herança de template permite você construir um "esqueleto" inicial
de template, que contém todas as partes comuns de seu site e define "blocos" que
templates filhos podem ser substituir.

Vejamos um exemplo disso com a criação de um template mais completo para
nossa view ``current_datetime``, pela edição do arquivo ``current_datetime.html``::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>The current time</title>
    </head>
    <body>
        <h1>My helpful timestamp site</h1>
        <p>It is now {{ current_date }}.</p>

        <hr>
        <p>Thanks for visiting my site.</p>
    </body>
    </html>


Isso parece bom, mas o que acontece quando queremos criar um template para
outra view, por exemplo, a view ``hours_ahead`` do capítulo 3? Se queremos
mais uma vez fazer uma bom, válido e completo template HTML, criaremos algo
como::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>Future time</title>
    </head>
    <body>
        <h1>My helpful timestamp site</h1>
        <p>In {{ hour_offset }} hour(s), it will be {{ next_time }}.</p>

        <hr>
        <p>Thanks for visiting my site.</p>
    </body>
    </html>

Claramente, simplesmente duplicamos um monte de HTML. Imagine se tivessemos um
site típico, incluíndo uma barra de navegação, algumas folhas de estilo,
talvés um pouco de Javascript, que nos acabaria colocando todos os tipos de HTML
redundantes em cada template.

A solução do lado servidor para esse problema é fatorar os pedaços comuns em
ambos os templates e salvar eles em trechos de templates separados, que então
são incluídos em cada template. Possivelmente você armazene o pedaço superior do
template em um arquivo chamado ``header.html``::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>

E possivelmente armazene o pedaço inferior em um arquivo chamado ``footer.html``::

        <hr>
        <p>Thanks for visiting my site.</p>
    </body>
    </html>

Com um estratégoa baseada em inclusão, o cabeçalho e rodapé são fáceis de inserir.
Há o meio termo que confunde. Nesse exemplo, ambas as páginas tem um título --
``<h1>My helpful timestamp site</h1>`` -- mas esse título não pode caber em
``header.html`` porque o ``<title>`` em ambas as páginas é diferente. Se
incluírmos o ``<h1>`` no cabeçalho, teríamos de incluir o ``<title>``, que
nos permite um personalização por página. Está vendo onde isso vai dar?

O sistema de herança de template do Django soluciona esse problema. Você pode
imaginar isso como sendo uma versão "inside-out" do server-side includes. Ao
invés de definir os trechos que são *comuns*, você define os trechos que
são *diferentes*.

O primeiro passo é definir um *template base* -- um esqueleto de sua página
que *templates filhos* irão mais tarde preencher. Aqui está um template base
para o nosso exemplo em curso::

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>{% block title %}{% endblock %}</title>
    </head>
    <body>
        <h1>My helpful timestamp site</h1>
        {% block content %}{% endblock %}
        {% block footer %}
        <hr>
        <p>Thanks for visiting my site.</p>
        {% endblock %}
    </body>
    </html>

Esse template, que vamos chamar de ``base.html``, define um simples esqueleto
HTML, que usaremos para todas as páginas do site. É o trabalho de templates
filhos para substituir, adicionar ou isolar o conteúdo dos blocos
(Se estiver acompanhando, salve esse arquivo em seu diretório de templates
como ``base.html``). // Revisar

Estamos usando uma template tag aqui que você não viu anteriormente: a
tag ``{% block %}``. O que todas as tags ``{% block %}`` fazem é falar ao
template engine que um template filho deve pode ser substituido naquelas
partes do template.

Agora que temos este template base, podemos modificar o nosso template
existente ``current_datetime.html`` para usá-lo::

    {% extends "base.html" %}

    {% block title %}The current time{% endblock %}

    {% block content %}
    <p>It is now {{ current_date }}.</p>
    {% endblock %}

Já que estamos no assunto, vamos criar um template para a view ``hours_ahead`` 
do Capítulo 3 (Se você está acompanhando com código, vamos deixar para que 
você altere ``hours_ahead`` para usar o sistema de template, ao invés de código 
fixo em HTML). Aqui está como deve se parecer::

    {% extends "base.html" %}

    {% block title %}Future time{% endblock %}

    {% block content %}
    <p>In {{ hour_offset }} hour(s), it will be {{ next_time }}.</p>
    {% endblock %}

Não é lindo? Cada template contém apenas o código que é *unico* para o 
template. Sem necessidade de redundância. Se você precisa fazer um site ter o 
design modificado, apenas faça a alteração em ``base.html``, e todos os outros 
templates imediatamente terão a modificação refletida.

Aqui está como isso funciona. Quando você carrega o template 
``current_datetime.html``, o mecanismo de template reconhece a tag 
``{% extends %}``, lembrando que esse template é um template filho. O mecanismo 
carrega imediatamente o template pai, neste caso, ``base.html``.

Nesse ponto, o mecanismo de template reconhece as três tags ``{% block %}`` 
no ``base.html`` e substitui esses blocos com o conteúdo do template 
filho. Assim, o título que havíamos definido em ``{% block title %}`` 
será usado, assim como ``{% block content %}``.

Perceba, desde que o template filho não defina o bloco ``footer``, o 
sistema de template usa em seu lugar o valor vindo do template pai. 
Conteúdo dentro de uma tag ``{% block %}`` em um template pai é sempre 
usado como um retorno.

Herança não afeta o contexto do template. Em outras palavras, qualquer template 
na árvore de herança devera ter acesso a cada uma de suas variáveis de 
templates a partir do contexto.

Você pode usar muitos níveis de herança, como julgar necessário. Uma maneira comum 
de usar herança é a abordagem em três níveis descrito a baixo:

1. Create a ``base.html`` template that holds the main look and feel of
   your site. This is the stuff that rarely, if ever, changes.

2. Create a ``base_SECTION.html`` template for each "section" of your site
   (e.g., ``base_photos.html`` and ``base_forum.html``). These templates
   extend ``base.html`` and include section-specific styles/design.

3. Create individual templates for each type of page, such as a forum page
   or a photo gallery. These templates extend the appropriate section
   template.

This approach maximizes code reuse and makes it easy to add items to shared
areas, such as section-wide navigation.

Here are some guidelines for working with template inheritance:

* If you use ``{% extends %}`` in a template, it must be the first
  template tag in that template. Otherwise, template inheritance won't
  work.

* Generally, the more ``{% block %}`` tags in your base templates, the
  better. Remember, child templates don't have to define all parent blocks,
  so you can fill in reasonable defaults in a number of blocks, and then
  define only the ones you need in the child templates. It's better to have
  more hooks than fewer hooks.

* If you find yourself duplicating code in a number of templates, it
  probably means you should move that code to a ``{% block %}`` in a
  parent template.

* If you need to get the content of the block from the parent template,
  use ``{{ block.super }}``, which is a "magic" variable providing the
  rendered text of the parent template. This is useful if you want to add
  to the contents of a parent block instead of completely overriding it.

* You may not define multiple ``{% block %}`` tags with the same name in
  the same template. This limitation exists because a block tag works in
  "both" directions. That is, a block tag doesn't just provide a hole to
  fill, it also defines the content that fills the hole in the *parent*.
  If there were two similarly named ``{% block %}`` tags in a template,
  that template's parent wouldn't know which one of the blocks' content to
  use.

* The template name you pass to ``{% extends %}`` is loaded using the same
  method that ``get_template()`` uses. That is, the template name is
  appended to your ``TEMPLATE_DIRS`` setting.

* In most cases, the argument to ``{% extends %}`` will be a string, but it
  can also be a variable, if you don't know the name of the parent template
  until runtime. This lets you do some cool, dynamic stuff.

What's next?
============

You now have the basics of Django's template system under your belt. What's next?

Many modern Web sites are *database-driven*: the content of the Web site is
stored in a relational database. This allows a clean separation of data and logic
(in the same way views and templates allow the separation of logic and display.)

The :doc:`next chapter <chapter05>` covers the tools Django gives you to
interact with a database.
