===============================================
Apêndice B: Referência da API de Banco de Dados
===============================================

A API de banco de dados do Django é a outra metade da API de modelos discutida no
Apêndice A. Uma vez que o modelo está definido, essa API será usada a qualquer
momento que for necessário acessar o banco de dados. Você viu exemplos de uso dela
por todo o livro. Este apêndice explica todas as várias opções em detalhes.

Apesar de as APIs de modelos e bancos de dados serem consideradas muito estáveis,
os desenvolvedores do framework constantemente as incrementam, adicionando novos
atalhos e conveniências. É uma boa ideia sempre checar a documentação mais recente,
disponível online no endereço http://docs.djangoproject.com/.

Nesta referência iremos nos referir aos seguintes modelos, que serão utilizados
para implementar uma aplicação simples de blog::

    from django.db import models

    class Blog(models.Model):
        name = models.CharField(max_length=100)
        tagline = models.TextField()

        def __unicode__(self):
            return self.name

    class Author(models.Model):
        name = models.CharField(max_length=50)
        email = models.EmailField()

        def __unicode__(self):
            return self.name

    class Entry(models.Model):
        blog = models.ForeignKey(Blog)
        headline = models.CharField(max_length=255)
        body_text = models.TextField()
        pub_date = models.DateTimeField()
        authors = models.ManyToManyField(Author)

        def __unicode__(self):
            return self.headline

Criando objetos
===============

Para criar um objeto, basta instanciá-lo usando argumentos chave-valor para o
modelo da classe e então chamar ``save()`` para salvá-lo no banco de dados::

    >>> from mysite.blog.models import Blog
    >>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
    >>> b.save()

Esse método executa um comando SQL ``INSERT`` por baixo dos panos. Django não
acessa o banco de dados até que ``save()`` seja explicitamente chamado.

O método ``save()`` não possui nenhum valor de retorno.

Para criar um objeto e savá-lo em um único passo, veja o método ``create``.

O que acontece quando um objeto é salvo?
----------------------------------------

Quando você salva um objeto, Django executa os seguintes passos:

#. **Emite um sinal pre_save.** Tal sinal notifica que um objeto está prestes a
   ser salvo. É possível registrar um ouvinte que será invocado sempre que esse
   sinal é emitido. Confira a documentação online para saber mais a respeito de
   sinais.

#. **Faz pré processamento dos dados.** A cada campo do objeto solicita-se que
   realize qualquer modificação automatizada de dados que o campo talvez precise
   realizar.

   A maior parte dos campos *não* realiza pré processamento -- os dados do campo
   são mantidos intocados. Pré processamento só é utilizado em campos que têm
   comportamento especial, a exemplo de campos de arquivo.

#. **Prepara os dados para o banco de dados.** A cada campo solicita-se que
   forneça seu valor atual em um tipo de dado que pode ser escrito ao banco de
   dados.

   A maior parte dos campos não precisam de preparação. Tipos simples de dados,
   como inteiros e strings, são "prontos para ser escritos" como um objeto Python.
   No entanto, tipos de dados mais complexos frequentemente requer alguma
   modificação. ``DateFields``, por exemplo, usa um objeto Python ``datetime``
   para armazenar dados. Bancos de dados não armazenam objetos ``datetime``,
   portanto o valor do campo precisa ser convertido para um string compatível
   com os padrões ISO antes de ser inserido no banco.

#. **Insere os dados no banco de dados.** Os dados pré processados e preparados
   são transformados em um comando SQL de inserção no banco de dados.

#. **Emite um sinal post_save.** Análogo ao sinal ``pre_save``, o post_save é
   usado para notificar que um objeto foi persistido com sucesso.

Autoincrementando chaves primárias
----------------------------------

Por motivos de conveniência, a cada modelo é atribuída uma chave primária
automaticamente incrementada chamada ``id`` -- a menos que o desenvolvedor
especifique explicitamente ``primary_key=True`` em um campo (veja a seção
intitulada "AutoField" no Apêndice A).

Se seu modelo tem um ``AutoField``, o valor que é automaticamente incrementado
será calculado e persistido como um atributo de seu objeto na primeira vez em que
o método ``save()`` é chamado::

    >>> b2 = Blog(name='Cheddar Talk', tagline='Thoughts on cheese.')
    >>> b2.id     # Retorna None, porque b ainda não tem um ID
    None

    >>> b2.save()
    >>> b2.id     # Retorna o ID do seu objeto.
    14

Não há como definir o valor que o ID assumirá antes de chamar ``save()`` por que
o valor é calculado pelo banco de dados e não pelo Django.

Se um modelo possui um ``AutoField`` mas você quer definir o ID de um novo objeto
explicitamente quando for salvá-lo, basta definí-lo explicitamente antes de salvar
em vez de confiar na atribuição automática do ID::

    >>> b3 = Blog(id=3, name='Cheddar Talk', tagline='Thoughts on cheese.')
    >>> b3.id
    3
    >>> b3.save()
    >>> b3.id
    3

Se for atribuir valores a chaves primárias manualmente, tenha certeza de não
utilizar uma que já existe! Caso uma chave primária já existente seja utilizada
para criar um novo objeto, Django irá assumir que se está querendo modificar o
objeto que já existe no banco -- e não criar um novo.

Dado o exemplo do blog anterior, ``'Cheddar Talk'``, as linhas a seguir iriam
sobrescrever o registro já existente no banco de dados::

    >>> b4 = Blog(id=3, name='Not Cheddar', tagline='Anything but cheese.')
    >>> b4.save()  # Sobrescreve o blog já existente com ID=3!

Especificações explícitas de valores para chaves primárias auto incrementadas são
utilizadas principalmente para inserção massiva de objetos quando existe a garantia
de que não haverá colisão entre as chaves primárias.

Salvando modificações a objetos
===============================

Modificações a objetos que já existem no banco de dados são efetivadas com o uso
de ``save()``.

Dada uma instância ``Blog`` chamada ``b5`` que já foi persistida, este exemplo
modifica seu nome e atualiza o registro no banco.

    >>> b5.name = 'New name'
    >>> b5.save()

O código acima executa um comando SQL ``UPDATE`` por baixo dos panos. Ratificando:
Django não acessa o banco de dados até que seja feita uma chamada explícita ao
método ``save()``.

.. admonition:: How Django Knows When to ``UPDATE`` and When to ``INSERT``

    Você deve ter notado que os objetos Django utilizam o mesmo método ``save()``
    tanto para criação quanto para atualização de objetos. Django abstrai a
    necessidade de usar comandos SQL ``INSERT`` ou ``UPDATE``. Mais especificamente,
    quando você chama ``save()``, o framework executa o seguinte algoritmo:

    * Se foi atribuído à chave primária do objeto um valor que é avaliado para
      ``True`` (i.e., um valor que não seja ``None`` ou string vazia), Django
      executa uma query ``SELECT`` para determinar se existe um registro com a
      chave primária fornecida.

    * Se o registro com a chave primária dada já existe, Django executa uma query
    ``UPDATE``.

    * Se *não* foi atribuído um valor à chave primária do objeto ou se foi atribuído
    mas não existe um registro com tal valor, Django executa um ``INSERT``.

    Por conta disso, é preciso ter cuidado para não especificar um valor de chave
    primária explicitamente ao salvar novos objetos se não é possível garantir
    que tal valor ainda não foi utilizado.

A atualização dos campos ``ForeignKey`` funciona exatamente da mesma maneira:
basta atribuir um objeto do tipo certo para o campo em questão::

    >>> joe = Author.objects.create(name="Joe")
    >>> entry.author = joe
    >>> entry.save()

Django irá protestar se você tentar atribuir um objeto de um tipo errado.

Buscando objetos
================

Por todo o livro foram mencionados objetos recuperados através de código como o
seguinte::

    >>> blogs = Blog.objects.filter(author__name__contains="Joe")

Há, aqui, alguns pontos menos óbvios de ser entendidos: quando são recuperados
objetos do banco de dados, o que acontece na realidade é a construção de um
``QuerySet`` que utiliza o ``Manager``do modelo. Esse ``QuerySet`` sabe como
executar SQL e retornar os objetos solicitados.

Ambos objetos foram abordados pelo Apêndice A do ponto de vista da definição de
modelos. Agora analisaremos como eles operam.

Um ``QuerySet`` representa uma coleção de objetos do seu banco de dados. Ele pode
ter nenhum, um ou muitos *filtros* -- isto é, critérios que restringem a coleção
baseados em parâmetros fornecidos. Em termos de SQL, um ``QuerySet é análogo a
um comando ``SELECT`` enquanto um filtro se equipararia a uma cláusula ``WHERE``.

Um ``QuerySet`` é obtido através da utilização do ``Manager``do modelo. Cada
modelo possui ao menos um ``Manager``, que é chamado ``objects`` por padrão.
Ele é acessado diretamente via classe do modelo, como segue::

    >>> Blog.objects
    <django.db.models.manager.Manager object at 0x137d00d>

``Manager``\s são acessíveis apenas via classes de modelo e não via instâncias
para obrigar uma separação entre operações a "nível de tabela" e operações a
"nível de registro"::

    >>> b = Blog(name='Foo', tagline='Bar')
    >>> b.objects
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: Manager isn't accessible via Blog instances.

O ``Manager``é a principal fonte de ``QuerySet``\s para um modelo. Ele age como
um ``QuerySet`` "root" que descreve todos os objetos na tabela do banco de dados
referente àquele modelo. ``Blog.objets``, por exemplo, é o ``QuerySet`` inicial
que contém todos os objetos ``Blog`` presentes no banco de dados.

Caching and QuerySets
=====================

Com vistas a minimizar o acesso ao banco de dados, cada ``QuerySet`` contém um
cache. É importante compreender como isso funciona para escrever códigos mais
eficientes.

Em um ``QuerySet`` recém-criado, o cache está vazio. Django salva os resultados
da query no cache do ``QuerySet`` e retorna os resultados que foram explicitamente
solicitados (e.g., o próximo elemento, no caso de uma iteração sobre o ``QuerySet``)
na primeira vez em que o ``QuerySet`` for avaliado -- isto é, quando ocorre uma
consulta ao banco de dados. As próximas avaliações do ``QuerySet`` reusam os
resultados mantidos em cache.

É importante manter esse comportamente de caching em mente por que ele pode ser
prejudicial ao desempenho da aplicação se seu ``QuerySet`` não for usado
corretamente. O código a seguir, por exemplo, criará dois ``QuerySet``\s, os
avaliará e os jogará fora::

    print [e.headline for e in Entry.objects.all()]
    print [e.pub_date for e in Entry.objects.all()]

Isso significa que a mesma consulta ao banco de dados será executada duas vezes,
efetivamente duplicando a demanda ao seu banco de dados. Existe, também, a
possibilidade de as duas listas não incluírem os mesmos registros do banco de
dados por que um ``Entry`` não foi adicionado ou removido na fração de segundo
entre as duas requisições.

Para evitar esse problema, simplesmente salve o ``QuerySet`` e o reutilize::

    queryset = Poll.objects.all()
    print [p.headline for p in queryset] # Evaluate the query set.
    print [p.pub_date for p in queryset] # Reuse the cache from the evaluation.

Filtrando Objetos
=================

A maneira mais simples de recuperar objetos de uma tabela é fazer uma consulta
que retorne todos os registros da tabela. Para tal, use o método ``all()`` de
um ``Manager``::

    >>> Entry.objects.all()

O método ``all()`` retorna um ``QuerySet`` de todos os objetos no banco de dados.

Apesar disso, normalmente você precisará selecionar apenas um subconjunto de todos
os objetos. Para conseguir tal subconjunto, melhore o ``QuerySet`` inicial através
da adição de condições de filtragem. Isso é feito geralmente com os métodos
``filter()`` e / ou ``exclude()``::

    >>> y2006 = Entry.objects.filter(pub_date__year=2006)
    >>> not2006 = Entry.objects.exclude(pub_date__year=2006)

``filter()`` and ``exclude()`` both take *field lookup* arguments, which are
discussed in detail shortly.

``filter()`` e ``exclude()`` recebem argumentos de *campo de busca*, que serão
discutidos detalhadamente em breve.

Encadeamento de Filtros
-----------------------

O resultado de refinar um ``QuerySet`` é um próprio ``QuerySet`` que pode, por
sua vez, receber novos filtros possibilitando, assim, o encadeamento desses
refinamentos, por exemplo::

    >>> qs = Entry.objects.filter(headline__startswith='What')
    >>> qs = qs.exclude(pub_date__gte=datetime.datetime.now())
    >>> qs = qs.filter(pub_date__gte=datetime.datetime(2005, 1, 1))

O código acima utiliza o ``QuerySet`` inicial, que contém todas as entradas do
banco de dados, adiciona um filtro, uma exclusão e, por fim, um outro filtro. O
resultado final é um ``QuerySet`` que contém todas as entradas com um headline
que começa em "What" e que foram publicados entre 1 de Janeiro de 2005 até o dia
atual.

Vale salientar que ``QuerySets`` são preguiçosos -- a criação de um ``QuerySet``
não acarreta nenhuma atividade do banco de dados. De fato, as três linhas de código
precedentes não executam *nenhuma* chamada ao banco de dados; você pode passar o
dia todo encadeando filtros que Django não irá executar a query até que o ``QuerySet``
seja *avaliado*.

Um ``QuerySet`` pode ser avaliado das seguintes maneiras:

* *Iterando*: Um ``QuerySet`` pode ser iterado e sua consulta ao banco de dados
  é executada na primeira vez em que a iteração é realizada. O ``QuerySet`` a
  seguir, por exemplo, não é avaliado até que uma iteração ocorra sobre ele no
  loop ``for``::

      qs = Entry.objects.filter(pub_date__year=2006)
      qs = qs.filter(headline__icontains="bill")
      for e in qs:
          print e.headline

  O código imprime todas as headlines de 2006 que contém "bill" executando um
  único acesso ao banco de dados.

* *Imprimindo*: Um ``QuerySet`` é avaliado quando ocorre uma chamado ao seu método
  ``repr()``. Isso ocorre por conveniência no interpretador interativo de Python,
  de maneira que é possível ver os resultados imediatamente ao utilizar a API
  interativamente.

* *Seccionando*: Como explicado na seção "Limitando QuerySets" a seguir, um ``QuerySet``
  pode ser seccionado através da sintaxe de secção de array de Python.
  Seccionar um ``QuerySet`` normalmente retorna outro ``QuerySet`` (não avaliado),
  mas Django executará a consulta ao banco de dados se você usar o parâmetro
  "step" da sintaxe de secção.

* *Convertendo para uma lista*: É possível forçar a avaliação de um ``QuerySet``
  através da chamada ao método ``list()``, por exemplo::

      >>> entry_list = list(Entry.objects.all())

  No entanto, é necessário cautela por que isso pode acarretar uma grande
  sobrecarga de memória, uma vez que Django carregará cada elemento da lista na
  memória. Por outro lado, iterar sobre um ``QuerySet`` tem a vantagem de fazer
  o carregamento de dados e instanciar objetos apenas quando eles são necessários.

.. admonition:: Filtered QuerySets Are Unique

    A cada vez que um ``QuerySet`` é refinado, é gerado um novo ``QuerySet`` que
    não tem qualquer ligação com o ``QuerySet`` anterior. Cada refinamento cria
    um ``QuerySet`` diferente e separado que pode ser armazenado, utilizado e
    reutilizado::

        q1 = Entry.objects.filter(headline__startswith="What")
        q2 = q1.exclude(pub_date__gte=datetime.now())
        q3 = q1.filter(pub_date__gte=datetime.now())

    Esses três ``QuerySets`` são separados. O primeiro é o ``QuerySet`` utilizado
    como base que contém todas as entradas que contém um headline começando com
    "What". O segundo é um subjeconjunto do primeiro, com o critério adicional
    que exclui registro cujo ``pub_date`` é maior que o datetime atual. O terceiro
    é um subconjunto do primeiro, com o critério adicional que seleciona apenas
    os registros cujo ``pub_date`` é maior que o datetime atual. O ``QuerySet``
    inicial (``q1``) não é afetado pelo processo de refinamento.

Limitando QuerySets
-------------------

Utilize a sintaxe de seccionamento de array de Python para limitar seu ``QuerySet``
a um determinado número de resultados. Isso é equivalente às cláusulas SQL ``LIMIT``
e ``OFFSET``.

A linha a seguir, por exemplo, retorna as primeiras cinco entradas (``LIMIT 5``)::

    >>> Entry.objects.all()[:5]

Já o código abaixo retorna da sexta à décima entradas (``OFFSET 5 LIMIT 5``)::

    >>> Entry.objects.all()[5:10]

Seccionar um ``QuerySet`` retorna, normalmente, um novo ``QuerySet`` -- e não
avalia a query. Uma exceção é o caso de se utilizar o parâmetro "step" da sintaxe
de seccionamento de array de Python. Por exemplo, a linha a seguir executaria a
consulta a fim de retornar uma lista de cada *segundo* objeto dos dez primeiros::

    >>> Entry.objects.all()[:10:2]

Para recuperar um *único* objeto em vez de uma lista (e.g., ``SELECT foo FROM bar
LIMIT 1``), utilize um índice simples em vez de uma secção. O código a seguir
retirba o primeiro ``Entry`` no banco de dados, depois de ordenar as entradas
alfabeticamente de acordo com seus headlines::

    >>> Entry.objects.order_by('headline')[0]

A grosso modo, isso é equivalente a isto::

    >>> Entry.objects.order_by('headline')[0:1].get()

Perceba, no entano, que, no primeiro caso, ``IndexError`` será lançada enquanto,
no segundo, ``DoesNotExist`` será lançada caso não existam objetos que satisfaçam
as condições de filtragem.

Métodos QuerySet que retornam novos QuerySets
------------------------------------------

Django fornece uma porção de métodos de refinamento de ``QuerySet`` que modificam
tanto os tipos dos resultados retornados pelo ``QuerySet`` quanto a maneira como
sua consulta SQL é executada. Esses métodos são descritos nas seções a seguir.
Alguns deles recebem argumentos de campo de busca, que serão discutidos em detalhes
um pouco mais adiante.

filter(\*\*lookup)
~~~~~~~~~~~~~~~~~~

Retorna um novo ``QuerySet`` que contém objetos que satisfazem os parâmetros de
busca fornecidos.

exclude(\*\*lookup)
~~~~~~~~~~~~~~~~~~~

Retorna um novo ``QuerySet`` que contém objetos que *não* satisfazem os parâmetros
de busca fornecidos.

order_by(\*fields)
~~~~~~~~~~~~~~~~~~

Por padrão, resultados retornados por um ``QuerySet`` são ordenados através da
ordenação da tupla dada pela opção de ``ordering`` presente nos metadados do
modelo (veja o Apêndice A). Você pode sobrescrever esse comportamento para uma
query em particular usando o método ``order_by()``::

    >> Entry.objects.filter(pub_date__year=2005).order_by('-pub_date', 'headline')

Esse resultado será ordenado pelo ``pub_date`` em ordem decrescente, depois pelo
``headline`` em ordem crescente. O sinal negativo na frente de ``"-pub_date"``
indica ordem *decrescente*. A ordem crescente é assumida se o ``-`` não estiver
presente. Para ordenar aleatoriamente, utilize ``"?"``, como em:

    >>> Entry.objects.order_by('?')

No entanto, ordenar aleatoriamente acarreta uma penalidade de performance, portanto
não é interessante utilizar esse comportamento para algo que tenha uma grande carga
de dados.

Se nenhuma ordenação for especificada em uma ``class Meta`` de um modelo e um
``QuerySet`` desse modelo não incluir a cláusula ``order_by()``, então a ordenação
ficará indefinida e pode ser diferente de uma query para outra.

distinct()
~~~~~~~~~~

Retorna um novo ``QuerySet`` que usa ``SELECT DISTINCT`` em sua query SQL. Isso
elimina dos resultados da query qualquer linha duplicada.

Por padrão, um ``QuerySet`` não eliminará linhas duplicadas. Na prática, isso
raramente é um problema, visto que queries simples como ``Blog.objects.all()``
não introduzem a possibilidade de linhas de resultado duplicadas.

No entanto, se sua query se aplica a muitas tabelas, é possível receber resultados
duplicados quando um ``QuerySet`` for avaliado. Esse é o caso em que ``distinct()``
deve ser utilizado.

values(\*fields)
~~~~~~~~~~~~~~~~

Retorna um ``QuerySet`` especial que é avaliado como uma lista de dicionários em
vez de objetos modelo-instância. Cada um desses dicionários representa um objeto,
com as chaves correspondendo aos nomes de atributos de objetos do modelo::

    # Esta lista contém um objeto Blog.
    >>> Blog.objects.filter(name__startswith='Beatles')
    [Beatles Blog]

    # Esta lista contém um dicionário.
    >>> Blog.objects.filter(name__startswith='Beatles').values()
    [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]

``values()`` pode receber argumentos de posicionamento, ``*fields``, que especificam
nomes de campos para os quais o ``SELECT`` deve ser limitado. Ao especificar os
campos, cada dicionário conterá somente o campo chaves/valores para os campos
especificados. Caso nenhum campo seja especificado, cada dicionário conterá uma
chave e um valor para cada campo na tabela do banco de dados::

    >>> Blog.objects.values()
    [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}],
    >>> Blog.objects.values('id', 'name')
    [{'id': 1, 'name': 'Beatles Blog'}]

Esse método é útil quando se sabe que apenas valores de uma pequena quantidade
dos campos disponíveis serão necessários e não haverá a necessidade de instanciar
um objeto do modelo. É mais eficiente selecionar somente os campos cuja utilização
se faz necessária.

dates(field, kind, order)
~~~~~~~~~~~~~~~~~~~~~~~~~

Retorna um ``QuerySet`` especial que é avaliado como uma lista de objetos
``datetime.datetime`` que representam todas as datas disponíveis de um tipo
específico dado pelo conteúdo do ``QuerySet``

O argumento ``field`` deve ser o nome de um ``DateField`` ou o ``DateTimeField``
de seu modelo. O argumento ``kind`` deve ser ``"year"``, ``"month"`` ou ``"day"``.
Cada objeto ``datetime.datetime`` na lista resultante é truncado de acordo com
seu tipo:

* ``"year"`` retorna uma lista de todos os valores de anos distintos para o campo.

* ``"month"`` retorna uma lista de todos os valores de ano/mês distintos para o campo.

* ``"day"`` retorna uma lista de todos os valores de ano/mês/dia distintos para
o campo.

``order``, cujo padrão é ``'ASC'``, deve ser ``'ASC'`` ou ``'DESC'``. Esse argumento
especifica como ordenar os resultados.

Seguem alguns exemplos::

    >>> Entry.objects.dates('pub_date', 'year')
    [datetime.datetime(2005, 1, 1)]

    >>> Entry.objects.dates('pub_date', 'month')
    [datetime.datetime(2005, 2, 1), datetime.datetime(2005, 3, 1)]

    >>> Entry.objects.dates('pub_date', 'day')
    [datetime.datetime(2005, 2, 20), datetime.datetime(2005, 3, 20)]

    >>> Entry.objects.dates('pub_date', 'day', order='DESC')
    [datetime.datetime(2005, 3, 20), datetime.datetime(2005, 2, 20)]

    >>> Entry.objects.filter(headline__contains='Lennon').dates('pub_date', 'day')
    [datetime.datetime(2005, 3, 20)]

select_related()
~~~~~~~~~~~~~~~~

Retorna um ``QuerySet` que irá, de maneira automática, "seguir" relacionamentos
de chave estrangeira, selecionando esses dados adicionais relacionados ao objeto
quando a query for executada. É uma forma de favorecer a perfomance, resultando
em (às vezes muitas) queries grandes mas significando que o uso posterior de
relacionamentos da chave estrangeira não acarretarão novas consultas ao banco
de dados.

Os casos a seguir exemplificam a diferença entre pesquisas simples e pesquisas
do tipo ``select_related()``. Aqui está a pesquisa padrão::

    # Vai ao banco de dados.
    >>> e = Entry.objects.get(id=5)

    # Vai ao banco de dados novamente para recuperar o objeto Blog relacionado.
    >>> b = e.blog

Aqui está a pesquisa ``select_related``:::

    # Vai ao banco de dados.
    >>> e = Entry.objects.select_related().get(id=5)

    # Não vai ao banco de dados por que e.blog já foi previamente populado na
    # query anterior.
    >>> b = e.blog

``select_related()`` segue chaves estrangeiras sempre que possível. Havendo os
seguintes modelos::

    class City(models.Model):
        # ...

    class Person(models.Model):
        # ...
        hometown = models.ForeignKey(City)

    class Book(models.Model):
        # ...
        author = models.ForeignKey(Person)

então uma chamada a ``Book.objects.select_related().get(id=4)`` irá colocar em
cache o ``Person`` relacionado *e* o ``City`` relacionado::

    >>> b = Book.objects.select_related().get(id=4)
    >>> p = b.author         # Doesn't hit the database.
    >>> c = p.hometown       # Doesn't hit the database.

    >>> b = Book.objects.get(id=4) # No select_related() in this example.
    >>> p = b.author         # Hits the database.
    >>> c = p.hometown       # Hits the database.

Observe que ``select_related()`` não segue chaves estrangeira que possuem
``null=True``.

O uso de ``select_related()`` pode, normalmente, melhorar substancialmente a
performance por que possibilita que a aplicação evite muitas consultas ao banco
de dados. No entanto, em situações com aninhamentos muito profundos de conjuntos
de relacionamentos, ``select_related()`` pode eventualmente acabar por seguir
relacionamentos demais e gerar queries tão grandes que demoram para executar.

Métodos QuerySet que não retornam QuerySets
-------------------------------------------

Os seguintes métodos ``QuerySet`` avaliam o ``QuerySet`` e retornam alguma coisa
*que não é* um ``QuerySet`` -- um único objeto, valor e assim por diante.

get(\*\*lookup)
~~~~~~~~~~~~~~~

Retorna um objeto que satisfaz os parâmetros da pesquisa, que deve estar de acordo
com o formato descrito na seção "Pesquisas de Campo". ``AssertionError`` é lançado
se mais de um objeto for encontrado.

``get()`` lança uma exceção ``DoesNotExist`` se um objeto não for encontrado para
os parâmetros dados. A exceção ``DoesNotExist`` é um atributo da classe do modelo,
por exemplo::

    >>> Entry.objects.get(id='foo') # lança Entry.DoesNotExist

A exceção ``DoesNotExist`` herda de ``django.core.exceptions.ObjectDoesNotExist``,
então é possível capturar múltiplas exceções ``DoesNotExist``::

    >>> from django.core.exceptions import ObjectDoesNotExist
    >>> try:
    ...     e = Entry.objects.get(id=3)
    ...     b = Blog.objects.get(id=1)
    ... except ObjectDoesNotExist:
    ...     print "Either the entry or blog doesn't exist."

create(\*\*kwargs)
~~~~~~~~~~~~~~~~~~

Método de conveniência para criar um objeto e salvá-lo em um único passo. Permite
que sejam condensados dois comandos comuns::

    >>> p = Person(first_name="Bruce", last_name="Springsteen")
    >>> p.save()

em uma única linha::

    >>> p = Person.objects.create(first_name="Bruce", last_name="Springsteen")

get_or_create(\*\*kwargs)
~~~~~~~~~~~~~~~~~~~~~~~~~

Método de conveniência para pesquisar por um objeto e criar um se o objeto pesquisado
não existir. Retorna uma tupla de ``(objeto, criado)`` em que ``objeto`` é o
objeto recuperado ou criado e ``criado`` é um Boolean especificando se um novo
objeto foi criado.

Este método é aplicado como um atalho padronização de código e é útil principalmente
para scripts de migração de dados. Por exemplo::

    try:
        obj = Person.objects.get(first_name='John', last_name='Lennon')
    except Person.DoesNotExist:
        obj = Person(first_name='John', last_name='Lennon', birthday=date(1940, 10, 9))
        obj.save()

Esse padrão se torna bastante pesado à medida em que o número de campos de um
modelo aumenta. O exemplo anterior pode ser reescrito com a utilização do método
``get_or_create()`` como segue::

    obj, created = Person.objects.get_or_create(
        first_name = 'John',
        last_name  = 'Lennon',
        defaults   = {'birthday': date(1940, 10, 9)}
    )

Quaisquer argumentos de palavra-passe passados para ``get_or_create()`` -- *com
exceção* de um opcional chamado ``defaults`` -- serão utilizados em uma chamada
a ``get()``. Se um objeto for encontrado, ``get_or_create()`` retorna uma tupla
daquele objeto e ``False``. Se um objeto *não* for encontrado, ``get_or_create()``
instanciará e salvará um novo objeto, retornando uma tupla do novo objeto e
``True``. O novo objeto será criado de acordo com o algoritmo a seguir::

    defaults = kwargs.pop('defaults', {})
    params = dict([(k, v) for k, v in kwargs.items() if '__' not in k])
    params.update(defaults)
    obj = self.model(**params)
    obj.save()

Em linguagem natural, isso significa começar com qualquer argumento de palavra-passe
não-``'defaults'`` que não contém um sublinhado duplo (que indicaria uma pesquisa
não exata). Depois, adicionar o conteúdo de ``defaults``, sobrescrevendo, se
necessário, qualquer chave, e utilizar o resultado como os argumentos de palavra-passe
para a classe do modelo.

Havendo um campo chamado ``defaults`` e desejando-se utilizá-lo como uma pesquisa
exata em ``get_or_create()``, basta informar ``defaults__exact``. Assim::

    Foo.objects.get_or_create(
        defaults__exact = 'bar',
        defaults={'defaults': 'bar'}
    )

.. note::

    Como mencionado anteriormente, ``get_or_create()`` é especialmente útil em
    scripts que precisam parsear dados e criar novos registros se não estão
    disponíveis registros existentes. Mas se você precisar usar ``get_or_create()``
    em uma view, certifique-se de utilizá-lo apenas em requisições ``POST`` a
    menos que você tenha uma boa razão para não fazer isso. Requisições ``GET``
    não deveriam ter nenhum efeito sobre os dados; utilize ``POST`` sempre que
    uma requisição para uma página tem algum efeito colateral sobre os dados.

count()
~~~~~~~

Retorna um inteiro que representa o número de objetos no banco de dados que
satisfazem o ``QuerySet``. ``count()`` nunca lança exceções. Eis um exemplo::

    # Retorna o número total de entradas no banco de dados.
    >>> Entry.objects.count()
    4

    # Retorna o número de entradas cujo headline contém 'Lennon'
    >>> Entry.objects.filter(headline__contains='Lennon').count()
    1

``count()`` executa um ``SELECT COUNT(*)`` nos bastidores, então deve-se sempre
utilizar ``count()`` em vez de carregar todos os registros em objetos Python e
chamar ``len()`` no resultado.

Dependendo do banco de dados que se estiver usando (e.g., PostgreSQL ou MySQL),
``count()`` pode retornar um long integer em vez de um integer Python comum. Essa
é uma peculiaridade da implementação subjacente que não deveria acarretar nenhum
problema do mundo real.

in_bulk(id_list)
~~~~~~~~~~~~~~~~

Recebe uma lista de valores de chave primária e retorna um dicionário que mapeia
cada valor de chave primária em uma instância do objeto com o dado ID. Por exemplo::

    >>> Blog.objects.in_bulk([1])
    {1: Beatles Blog}
    >>> Blog.objects.in_bulk([1, 2])
    {1: Beatles Blog, 2: Cheddar Talk}
    >>> Blog.objects.in_bulk([])
    {}

IDs de objetos que não existem são silenciosamente derrubados do dicionário
resultante. Passar uma lista vazia para ``in_bulk()`` resulta em um dicionário
vazio.

latest(field_name=None)
~~~~~~~~~~~~~~~~~~~~~~~

Retorna o último objeto na tabela, de acordo com a data, utilizando o ``field_name``
fornecido como o campo de data. O exemplo a seguir retorna o último ``Entry`` na
tabela de acordo com o campo ``pub_date``::

    >>> Entry.objects.latest('pub_date')

Se o ``Meta`` do modelo especifica ``get_latest_by``, o argumento ``field_name``
torna-se prescindível. Django utilizará o campo especificado no ``get_latest_by``
por padrão.

Assim como ``get()``, ``latest()`` lança ``DoesNotExist`` se não existe objeto
com os parâmetros fornecidos.

Pesquisas de Campo
==================

Pesquisas de campo são a maneira de se especificar o conteúdo de uma cláusula
SQL ``WHERE``. Elas são explicitadas por meio de argumentos tipo palavra-chave
para os métodos ``filter()``, ``exclude()`` e ``get()`` de ``QuerySet``.

Argumentos palavra-chave básicos possuem a forma ``campo__tipodepesquisa=valor``
(observer o sublinhado duplo). Por exemplo::

    >>> Entry.objects.filter(pub_date__lte='2006-01-01')

pode ser grosseiramente traduzido para a seguinte consulta SQL::

    SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';

Ao passar um valor inválido como argumento de palavra-chave, uma função de pesquisa
irá lançar ``TypeError``.

Os tipos de pesquisa suportados são exibidos a seguir.

exact
-----

Realiza uma busca que satisfaça exatamente os argumentos dados::

    >>> Entry.objects.get(headline__exact="Man bites dog")

A consulta acima retorna qualquer objeto que possua o headline exatamente igual
a "Man bites dog".

Não especificar um tipo de pesquisa -- isto é, se o argumento do método de pesquisa
não tiver um sublinhado duplo --, assume-se que o tipo de pesquisa é ``exact``.

As linhas seguintes, por exemplo, são equivalentes::

    >>> Blog.objects.get(id__exact=14) # Explicit form
    >>> Blog.objects.get(id=14) # __exact is implied

Isso é feito por conveniência por que pesquisas ``exact`` são o tipo mais comum.

iexact
------

Realiza uma busca case-insensitive (isto é, que não leva em consideração diferenças
entre letras maiúsculas e minúsculas)::

    >>> Blog.objects.get(name__iexact='beatles blog')

A consulta é satisfeita por ``'Beatles Blog'``, ``'beatles blog'``,
``'BeAtLes BLoG'`` e assim por diante.

contains
--------

Realiza uma busca case-sensitive (isto é, considerando diferenças entre letras
maiúsculas e minúsculas) para entradas que possuem o termo especificado como
argumento::

    Entry.objects.get(headline__contains='Lennon')

A pesquisa é satisfeita pelo headline ``'Today Lennon honored'`` mas não por
``'today lennon honored'``.

SQLite não dá suporte a comandos ``LIKE`` case-sensitive; ao utilizar SQLite,
``contains`` irá possuir o mesmo comportamento que ``icontains``.

.. admonition:: Escaping Percent Signs and Underscores in LIKE Statements

    As pesquisas de campo semelhantes ao comando SQL ``LIKE`` (``iexact``,
    ``contains``, ``icontains``, ``startswith``, ``istartswith``, ``endswith``
    e ``iendswith``) irão converter automaticamente os dois caracteres especiais
    utilizados em comandos ``LIKE`` -- o sinal de porcentagem e o sublinhado --
    para que funcionem adequadamente. (Em um comando ``LIKE``, o sinal de
    porcentagem indica um caractere curinga múltiplo -- isto é, pode ser qualquer
    caractere ou uma sequência de quaisquer caracteres -- e o sublinhado indica
    um caractere curinga único -- isto é, pode ser qualquer caractere).

    Isso significa que tudo deve funcionar intuitivamente de maneira tal que a
    abstração não falhe. Por exemplo, para recuperar todas as entradas que contém
    um sinal de porcentagem, simplesmente use o sinal de porcentagem como um
    caractere comum::

        Entry.objects.filter(headline__contains='%')

    Django se encarrega de converter a consulta. O SQL resultante parecerá algo
    como isto::

        SELECT ... WHERE headline LIKE '%\%%';

    O mesmo vale para sublinhados. Ambos sinais de porcentagem e sublinhado são
    manipulados de maneira transparente para o programador.

icontains
---------

Realiza uma busca case-insensitive para entradas que possuem o termo especificado
como argumento::

    >>> Entry.objects.get(headline__icontains='Lennon')

Diferentemente de ``contains``, ``icontains`` *aceitará* ``'today lennon honored'``.

gt, gte, lt, and lte
--------------------

Essas são representações de maior que, maior que ou igual a, menor que e menor
que ou igual a::

    >>> Entry.objects.filter(id__gt=4)
    >>> Entry.objects.filter(id__lt=15)
    >>> Entry.objects.filter(id__gte=0)

Essas queries, respectivamente, retornam quaisquer objetos com um ID maior que
possuam ID maior que 4, ID menor que 15 e um ID maior ou igual a 1.

Tais parâmetros são normalmente utilizados para pesquisa de campos numéricos. É
necessário tomar cuidado com relação aos campos de caracteres uma vez que sua
ordem nem sempre é a esperada (i.e., a string "4" é ordenada para *depois* da
string "10").

in
--

Filtros em que um valor é dado numa lista::

    Entry.objects.filter(id__in=[1, 3, 4])

Isso retorna todos objetos com o ID 1, 3 ou 4.

startswith
----------

Realiza uma consulta case-sensitive que checa o começo da entrada::

    >>> Entry.objects.filter(headline__startswith='Will')

Isso retornará os headlines "Will he run?" e "Willbur named judge" mas não
"Who is Will?" nem "will found in crypt".

istartswith
-----------

Realiza uma consulta case-insensitive que checa o começo da entrada::

    >>> Entry.objects.filter(headline__istartswith='will')

Isso retornará os headlines "Will he run?", "Willbur named judge" e
"will found in crypt" mas não "Who is Will?".

endswith and iendswith
----------------------

Realiza consultas case-sensitive e case-insensitive que checa o término da entrada::

    >>> Entry.objects.filter(headline__endswith='cats')
    >>> Entry.objects.filter(headline__iendswith='cats')

Similar a ``startswith`` e ``istartswith``.

range
-----

Realiza uma checagem inclusiva de intervalo::

    >>> start_date = datetime.date(2005, 1, 1)
    >>> end_date = datetime.date(2005, 3, 31)
    >>> Entry.objects.filter(pub_date__range=(start_date, end_date))

Pode-se utilizar ``range`` em qualquer lugar em que se pode utilizar ``BETWEEN``
em SQL -- para datas, números e até caracteres.

year, month, and day
--------------------

Para campos date/datetime, realiza comparação exata de ano, mês ou dia::

    # Retorna todas as entradas publicadas em 2005
    >>>Entry.objects.filter(pub_date__year=2005)

    # Retorna todas as entradas publicadas em dezembro
    >>> Entry.objects.filter(pub_date__month=12)

    # Retorna todas as entradas publicadas no terceiro dia do mês
    >>> Entry.objects.filter(pub_date__day=3)

    # Combinação: retorna todas as entradas do natal de qualquer ano
    >>> Entry.objects.filter(pub_date__month=12, pub_date_day=25)

isnull
------

Recebe ``True`` ou ``False``, correspondendo às queries SQL ``IS NULL`` e
``IS NOT NULL``, respectivamente::

    >>> Entry.objects.filter(pub_date__isnull=True)

search
------

Uma busca booleana textual que se vale do indexador de texto. Funciona como o
``contains`` mas é significativamente mais rápido devido à utilização do indexador.

Observe que este método está disponível apenas para MySQL e exige manipulação
direta do banco de dados para adicionar o indexador de texto.

O atalho de pesquisa pk
----------------------

Por conveniência, Django fornece um tipo de pesquisa ``pk``, que significa
"primary_key" (chave primária).

No modelo ``Blog``do exemplo, a chave primária é o campo ``id``, portanto as
três linhas de código a seguir são equivalentes::

    >>> Blog.objects.get(id__exact=14) # Explicitamente
    >>> Blog.objects.get(id=14) # __exact está implícito
    >>> Blog.objects.get(pk=14) # pk implica em id__exact

A utilização de ``pk`` não é restrita a queries do tipo ``__exact`` -- qualquer
termo de query pode ser combinado com ``pk`` para realizar uma consulta sobre a
chave primária de um modelo::

    # Recupera entradas de blog com id 1, 4 e 7
    >>> Blog.objects.filter(pk__in=[1,4,7])

    # Recupera todas as entradas de blog com id > 14
    >>> Blog.objects.filter(pk__gt=14)

Pesquisas ``pk`` também funcionam através de joins. Por exemplo, as três linhas
de código a seguir são equivalentes::

    >>> Entry.objects.filter(blog__id__exact=3) # Explicitamente
    >>> Entry.objects.filter(blog__id=3) # __exact está implícito
    >>> Entry.objects.filter(blog__pk=3) # __pk implica em __id__exact

A razão de ser do ``pk`` é fornecer uma maneira genérica de referenciar a chave
primária em casos em que não se tem certeza a respeito do nome da chave primária
de um modelo.

Pesquisas Complexas com Objetos Q
=================================

Consultas com argumentos palavra-chave -- ``filter()`` e daí em diante -- são
acopladas com o operador lógico ``AND``. Se for necessário executar consultas
mais complexas (e.g., consultas com o operador ``OR``), pode-se lançar mão de
objetos ``Q``.

Um objeto ``Q`` (``django.db.models.Q``) é um objeto usado para encapsular uma
coleção de argumentos palavra-chave. Esses argumentos palavra-chave são especificados
como na seção "Pesquisas de Campo".

Por exemplo, o objeto ``Q``a seguir encapsula uma única consulta ``LIKE``::

    Q(question__startswith='What')

Objetos ``Q`` podem ser combinados usando operadores ``&`` e ``|``. Quando um
operador é usado em dois objetos ``Q``, eles dão espaço a um novo objeto ``Q``.
Por exemplo, a linha a seguir origina um único objeto ``Q`` que representa o
OR de duas queries ``"question__startswith"``::

    Q(question__startswith='Who') | Q(question__startswith='What')

Isso é equivalente à seguinte cláusula SQL ``WHERE``::

    WHERE question LIKE 'Who%' OR question LIKE 'What%'

É possível compor consultas de complexidade arbitrária através da combinação de
objetos ``Q`` com os operadores ``&`` e ``|``. Também é possível utilizar
agrupamento através de parênteses.

Cada função de pesquisa que recebe argumentos palavra-chave (e.g., ``filter()``,
``exclude()``, ``get()``) pode também receber um ou mais objetos ``Q`` como
argumentos posicionais (não nomeados). Ao fornecer múltiplos objetos ``Q`` como
argumentos para uma função de pesquisa, os argumentos serão acoplados com o
operador AND. Por exemplo::

    Poll.objects.get(
        Q(question__startswith='Who'),
        Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
    )

pode ser traduzido grosseiramente para a seguinte consulta SQL::

    SELECT * from polls WHERE question LIKE 'Who%'
        AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')

Funções de pesquisa podem mesclar a utilização de objetos ``Q`` e argumentos
palavra-chave. Todos argumentos fornecidos para uma função de pesquisa (sejam
eles argumentos palavra-chave ou objetos ``Q``) são acoplados com o operador
AND. No entanto, se um objeto ``Q`` é fornecido, ele deve preceder a definição
de qualquer argumento palavra-chave. Por exemplo, o seguinte código::

    Poll.objects.get(
        Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
        question__startswith='Who')

seria uma query válida, equivalente ao exemplo anterior, mas esta::

    # QUERY INVÁLIDA
    Poll.objects.get(
        question__startswith='Who',
        Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)))

não seria válida.

Mais exemplos estão disponíveis online em http://www.djangoproject.com/documentation/models/or_lookups/.

Objetos Relacionados
====================

Quando se define um relacionamento em um modelo (i.e., ``ForeignKey``,
``OneToOneField`` ou ``ManyToManyField``), instâncias desse modelo possuirão uma
API conveniente para acessar o(s) objeto(s) relacionado(s).

Por exemplo, um objeto ``Entry`` ``e`` pode acessar seu objeto ``Blog`` relacionado
acessando o atributo ``blog`` ``e.blog``.

Django também cria uma API de acesso para o "outro" lado do relacionamento --
o link do modelo relacionado e o modelo que define o relacionamento. Por exemplo,
um objeto ``Blog`` ``b`` tem acesso a uma lista de todos os objetos ``Entry``
relacionados através do atributo ``entry_set``: ``b.entry_set.all()``.

Todos os exemplos nesta seção usa os modelos ``Blog``, ``Author`` e ``Entry``
do exemplo definidos no começo deste apêndice.

Pesquisas que espalham relacionamentos
--------------------------------------

Django oferece uma maneira intuitiva e poderosa para "seguir" relacionamentos
em pesquisas tomando conta de ``JOIN``\s SQL automaticamente por trás da cena.
Para espalhar um relacionamento, deve-se simplesmente utilizar o nome do campo
de campos relacionados entre modelos, separados por sublinhados duplos, até obter
o campo desejado.

A linha a seguir recupera todos os objetos ``Entry`` com um ``Blog`` cujo ``name``
é ``'Beatles Blog'``::

    >>> Entry.objects.filter(blog__name__exact='Beatles Blog')

Esse espalhamento pode ser tão profundo quanto se desejar.

Ele também funciona no sentido contrário. Para se referir a um relacionamento
"reverso", deve-se utilizar o nome do modelo em minúsculo.

Este exemplo recupera todos os objetos ``Blog`` que têm ao menos um ``Entry``
cujo ``headline`` contém ``'Lennon'``::

    >>> Blog.objects.filter(entry__headline__contains='Lennon')

Relacionamentos de Chave Estrangeira
------------------------------------

Se um modelo possui uma ``ForeignKey``, instâncias daquele modelo terão acesso
ao objeto (estrangeiro) relacionado através de um simples atributo de modelo.
Por exemplo::

    e = Entry.objects.get(id=2)
    e.blog # Retorna o objeto Blog relacionado

É possível recuperar e setar através de atributo de chave estrangeira. Como se pode
esperar, mudanças à chave estrangeira não são persistidas no banco de dados até
que se chame o método ``save()``, por exemplo::

    e = Entry.objects.get(id=2)
    e.blog = some_blog
    e.save()

Se um campo ``ForeignKey`` está configurado para ``null=True`` (i.e., ele permite
valores ``NULL``), pode-se configurar para ``NULL`` atribuindo ``None``e salvando::

    e = Entry.objects.get(id=2)
    e.blog = None
    e.save() # "UPDATE blog_entry SET blog_id = NULL ...;"

O acesso a relacionamentos um-para-muitos é guardado em cache na primeira vez em
que o objeto relacionado é acessado. Acessos subsequentes à chave estrangeira no
mesmo objeto são feitos através do cache. Por exemplo::

    e = Entry.objects.get(id=2)
    print e.blog  # Acessa o banco de dados para recuperar o Blog associado.
    print e.blog  # Não acessa o banco de dados; usa a versão em cache.

Observe que o método ``QuerySet`` ``select_related()`` previamente popula, de
forma recursiva, o cache de todos os relacionamentos um-para-muitos.

    e = Entry.objects.select_related().get(id=2)
    print e.blog  # Não acessa o banco de dados; usa a versão em cache.
    print e.blog  # Não acessa o banco de dados; usa a versão em cache.

``select_related()`` está documentado na seção "Métodos QuerySet que retornam
novos QuerySets".

Relacionamentos de Chave Estrangeira "Reversos"
-----------------------------------------------

Relacionamentos de chave estrangeira são automaticamente simétricos -- um
relacionamento reverso é inferido da presença de uma ``ForeignKey`` apontando
para outro modelo.

Se um modelo possui uma ``ForeignKey``, instâncias do modelo da chave estrangeira
terão acesso a um ``Manager`` que retorna todas as instâncias do primeiro modelo
que estão relacionados àquele objeto. Por padrão, esse ``Managed`` recebe o nome
``FOO_set``, em que ``FOO`` é o nome do modelo de origem em minúsculo. Esse
``Manager`` retorna ``QuerySets`` que podem ser filtrados e manipulados como
descrito na seção "Buscando objetos".

Eis um exemplo::

    b = Blog.objects.get(id=1)
    b.entry_set.all() # Retorna todos os objetos Entry relacionados ao blog.

    # b.entry_set é um Manager que retorna QuerySets.
    b.entry_set.filter(headline__contains='Lennon')
    b.entry_set.count()

É possível sobrescrever o nome ``FOO_set`` configurando o parâmetro ``related_name``
na definição de ``ForeignKey()``. Por exemplo, se o modelo ``Entry`` foi alterado
para ``blog = ForeignKey(Blog, related_name='entries')``, o exemplo anterior
se pareceria com o seguinte::

    b = Blog.objects.get(id=1)
    b.entries.all() # Retorna todos os objetos Entry relacionados ao blog.

    # b.entry_set é um Manager que retorna QuerySets.
    b.entries.filter(headline__contains='Lennon')
    b.entries.count()

``related_name`` é particularmente útil se um modelo possui duas chaves estrangeiras
para o mesmo segundo modelo.

Não é possível acessar um ``Manager`` ``ForeignKey`` reverso a partir da classe;
ele deve ser acessado a partir de uma instância::

    Blog.entry_set # Levanta AttributeError: "Manager must be accessed via instance".

Adicionalmente aos métodos ``QuerySet`` definidos na seção "Buscando objetos",
o ``Manager`` ``ForeignKey`` possui estes outros métodos:

* ``add(obj1, obj2, ...)``: Adiciona os objetos especificados ao conjunto de
  objetos relacionado, por exemplo::

      b = Blog.objects.get(id=1)
      e = Entry.objects.get(id=234)
      b.entry_set.add(e) # Associa Entry e com Blog b.

* ``create(**kwargs)``: Cria um novo objeto, o salva e o coloca no conjunto de
  objetos relacionado. Retorna o objeto recém-criado::

      b = Blog.objects.get(id=1)
      e = b.entry_set.create(headline='Hello', body_text='Hi', pub_date=datetime.date(2005, 1, 1))
      # Não é necessário chamar e.save() neste ponto -- o objeto já foi salvo.

  Isso é equivalente a (mas muito mais simples que) o seguinte::

      b = Blog.objects.get(id=1)
      e = Entry(blog=b, headline='Hello', body_text='Hi', pub_date=datetime.date(2005, 1, 1))
      e.save()

  Observe que não há a necessidade de especificar o argumento de palavra-chave
  do modelo que define o relacionamento. No exemplo anterior, o parâmetro ``blog``
  não é passado para ``create()``. Django percebe que o novo campo ``blog``
  do objeto ``Entry`` deveria ser configurado para ``b``.

* ``remove(obj1, obj2, ...)``: Remove os objetos especificados do conjunto de
  objetos relacionado::

      b = Blog.objects.get(id=1)
      e = Entry.objects.get(id=234)
      b.entry_set.remove(e) # Desassocia Entry e do Blog b.

  Com vistas a prevenir inconsistência do banco de dados, esse método só existe
  em objetos ``ForeignKey`` em que ``null=TRUE``. Se o campo relacionado não
  puder ser configurado para ``None`` (``NULL``), então um objeto não pode ser
  removido de um relacionamento sem que outro seja adicionado. No exemplo anterior,
  remover ``e`` do ``b.entry_set()`` é equivalente a fazer ``e.blog=None``
  e, pelo fato de ``ForeignKey`` ``blog`` não possuir ``null=True``, isso não é
  válido.

* ``clear()``: Remove todos objetos do conjunto de objetos relacionado::

      b = Blog.objects.get(id=1)
      b.entry_set.clear()

  Observe que isso não apaga os objetos relacionados -- simplesmente os
  desassocia.

  Assim como ``remove()``, ``clear()`` só está disponível em ``ForeignKey``s
  em que ``null=True``.

Para atribuir os membros de um conjunto relacionado de uma só vez, simplesmente
atribua a ele a partir de qualquer objeto iterável. Por exemplo::

    b = Blog.objects.get(id=1)
    b.entry_set = [e1, e2]

Se o método ``clear()`` estiver disponível, quaisquer objetos previamente
existentes serão removidos do ``entry_set`` antes que todos objetos no iterável
(nesse caso, uma lista) sejam adicionados ao conjunto. Se o método ``clear()``
*não* estiver disponível, todos objetos no iterável serão adicionados sem remover
nenhum dos elementos existentes.

Cada operação "reversa" descrita nesta seção possui um efeito imediato no banco
de dados. Toda adição, criação e remoção é imediatamente e automaticamente salva
no banco de dados.

Relacionamentos muitos-para-muitos
----------------------------------

Ambos lados de um relacionamento muitos-para-muitos ganham, automaticamente,
API de acesso para o outro lado. A API funciona assim como um relacionamento
um-para-muitos "reverso" (descrito na seção anterior).

A única diferença está na denominação do atributo: o modelo que define
``ManyToManyField`` utiliza o nome de atributo do próprio campo, ao passo em que
o modelo "reverso" utiliza o nome do modelo do modelo original em minúsculo
concatenado com ``'_set'`` (da mesma maneira que relacionamentos um-para-muitos
reversos).

Um exemplo pode facilitar a compreensão desse conceito::

    e = Entry.objects.get(id=3)
    e.authors.all() # Retorna todos objetos Author para este Entry.
    e.authors.count()
    e.authors.filter(name__contains='John')

    a = Author.objects.get(id=5)
    a.entry_set.all() # Retorna todos objetos Entry para este Author.

Assim como ``ForeignKey``, ``ManyToManyField`` pode especificar ``related_name``.
No exemplo anterior, se o ``ManyToManyField`` em ``Entry`` tivesse especificado
``related_name='entries'``, então cada instância de ``Author`` possuiria um
atributo ``entries`` em vez de ``entry_set``.

.. admonition:: How Are the Backward Relationships Possible?

    Outros mapeadores objeto-relacionamento exigem que sejam definidos relacionamentos
    em ambos lados. Os desenvolvedores de Django acreditam que isso é uma violação
    do princípio DRY (Don't Repeat Yourself -- Não Se Repita), portanto Django
    exige que se defina o relacionamento em apenas um lado. Mas como é possível,
    dado que uma classe de modelo não sabe quais outras classes de modelo estão
    relacionados a ela até que aquelas outras classes de modelo sejam carregadas?

    A resposta está na configuração de ``INSTALLED_APPS``. Na primeira vez em que
    qualquer modelo for carregado, Django itera sobre cada modelo em ``INSTALLED_APPS``
    e cria os relacionamentos no sentido contrário na memória sob demanda.
    Essencialmente, uma das funções de ``INSTALLED_APPS`` é informar ao Django
    todo o domínio de modelo.

Consultas sobre objetos relacionados
------------------------------------

Consultas que envolvém objetos relacionados seguem as mesmas regras que consultas
que envolvem campos de valores comuns. Ao especificar o parâmetro de satisfação
para uma consulta, deve-se utilizar uma instância de objeto ou o valor de chave
primária para o objeto.

Por exemplo, supondo que exista um objeto ``Blog`` ``b`` com ``id=5``, as três
queries seguintes seriam idênticas::

    Entry.objects.filter(blog=b) # Query using object instance
    Entry.objects.filter(blog=b.id) # Query using id from instance
    Entry.objects.filter(blog=5) # Query using id directly

Deletando Objetos
=================

O método para deletar objetos é, convenientemente, chamado ``delete()``. Esse
método imediatamente deleta o objeto e não tem nenhum valor de retorno::

    e.delete()

Também é possível deletar objetos massivamente. Todo ``QuerySet`` possui um
método ``delete()`` que deleta todos os membros do ``QuerySet``. Por exemplo,
o código seguinte deleta todos os objetos ``Entry`` com um ``pub_date`` do ano
2005::

    Entry.objects.filter(pub_date__year=2005).delete()

Quando Django deleta um objeto, ele emula o comportamento da restrição SQL
``ON DELETE CASCADE`` -- em outras palavras, quaisquer objetos que tiverem
chaves estrangeiras apontando para o objeto a ser deletado serão deletados
junto com ele, por exemplo::

    b = Blog.objects.get(pk=1)
    # Isso deletará o Blog e todo os seus objetos Entry.
    b.delete()

Observe que ``delete()`` é o único método ``QuerySet`` que não está disponível
em um ``Manager``. Esse é um mecanismo de segurança para prevenir chamadas
acidentais a ``Entry.objects.delete()`` e deletar *todas* as entradas. Se você
*quiser* de fato deletar todos os objetos, então terá de requisitar isso
explicitamente:

    Entry.objects.all().delete()

Atalhos
=======

À medida em que se desenvolvem views, descobre-se uma porção de expressões comuns
na utilização da API do banco de dados. Django codifica algumas dessas expressões
como atalhos que podem ser utilizados para simplificar o processo de escrever
views. Essas funções estão no módulo ``django.shortcuts``.

get_object_or_404()
-------------------

É bastante comum utilizar ``get()`` e levantar ``Http404`` se o objeto não existe.
Isso é capturado por ``get_object_or_404()``: uma função que recebe um modelo
Django como seu primeiro argumento e um número arbitrário de argumentos
palavra-chave que são passados para o gerente padrão da função ``get()``. Ela
levanta ``Http404`` se o objeto não existe, por exemplo::

    # Pega o Entry com uma chave primária 3
    e = get_object_or_404(Entry, pk=3)

Ao fornecer um modelo para essa função de atalho, o gerente padrão é utilizado
para executar a query ``get()`` subjacente. Se a utilização do gerente padrão
não for desejada ou no caso de se desejar buscar uma lista de objetos relacionados,
é possível fornecer um objeto ``Manager`` para ``get_object_or_404()``::

    # Pega o autor do blog e com um nome 'Fred'
    a = get_object_or_404(e.authors, name='Fred')

    # Utiliza um gerente padrão 'recent_entries' na busca por uma entrada com
    # uma chave primária 3
    e = get_object_or_404(Entry.recent_entries, pk=3)

get_list_or_404()
-----------------

``get_list_or_404`` se comporta de maneira semelhante a ``get_object_or_404()``,
a não ser pelo fato de utilizar ``filter()`` em vez de ``get()``. Levanta
``Http404`` se a lista estiver vazia.

Recuperando com SQL puro
========================

Se for necessário escrever uma query SQL que é complexa demais para ser feita
com o mapeador de banco de dados do Django, é possível utilizar comandos SQL
puros.

A melhor maneira de se fazer isso é fornecer ao modelo métodos personalizados
ou métodos de gerente personalizados para execução de consultas. No entanto,
não há nada em Django que *exija* que consultas a banco de dados residam na
camada de modelo -- essa abordagem mantém toda a lógica de acesso a dados em um
único lugar, o que é inteligente do ponto de vista de organização do código. Para
instruções, veja o Apêndice A.

Por fim, é importante perceber que a camada de bando de dados do Django não passa
de uma interface para o banco de dados. É possível acessar o banco de dados
através de outras ferramentas, linguagens de programação ou frameworks de banco
de dados -- não há nada que torne o banco de dados específico para Django.
