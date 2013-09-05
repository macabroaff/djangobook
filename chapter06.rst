=================================
Chapter 6: O Site Admin do Django
=================================

Para alguns tipos de Web sites, uma *interface admin* é parte essencial da
infraestrutura. Trata-se de uma interface baseada na Web e autorizada a 
apenas alguns administradores que permite acrescentar, remover e editar conteúdo.
Alguns exemplos mais comuns pode ser aquela interface que você usa para postar
conteúdo no seu blog, ou o backend que os administradores do site fazem uso
quando moderam comentários gerados por usuários.

Mas existe um problema com as interfaces admin: são muito chatas de construir.
O desenvolvimento Web é mais interessante quando você implementa funcionalidades
pensadas no público, enquanto que construir interfaces admin é sempre a mesma coisa.
Você precisa autenticar usuários, exibir e manipular formulários, validar dados de 
entrada, e por aí vai. É maçante e repetitivo.

E como o Django resolve essas tarefas monótonas para mim? Bem, basicamente ele
faz tudo por você em apenas algumas linhas de código. Com o Django, construir uma
interface admin é problema resolvido.

Este capítulo é sobre a interface admin automática do Django. Ela trabalha lendo
metadados de um modelo visando fornecer uma interface poderosa que permite um
administrador de Web site começar a usá-la instantaneamente. Neste capítulo,
iremos discutir como ativar, utilizar e customizar essa interface.

NOTA: Recomendamos a leitura integral deste capítulo mesmo se você não pretende utilizar o 
site de administrador do Django, porque alguns conceitos-chave apresentados aqui
se aplicam a todo o Django, independentemente da sua utilização.

Os pacotes django.contrib 
==========================

A administração automática do Django é parte de um conjunto de funcionalidades
chamado ``django.contrib`` --  a parte do código base do Django que contém
vários add-ons úteios ao framework. Você pode pensar no ``django.contrib`` 
como o equivalente à biblioteca padrão do Python standard library. O Django traz
esses pacotes embutidos nele para que você não precise reinventar a roda em suas
próprias aplicações.

O site admin é a primeira parte do ``django.contrib`` que iremos cobrir neste 
livro; tecnicamente o nome correto é ``django.contrib.admin``. Além dele, outras
características disponíveis em ``django.contrib`` incluem um sistema de autenticação
de usuários (``django.contrib.auth``), suporte para sessões anônimas,
(``django.contrib.sessions``) e até mesmo um sistema para commentários de usuários
(``django.contrib.comments``). À medida que você se torna um expert em Django, você
conhecerá muitas outras capacidades do ``django.contrib``, e depois iremos discutir 
mais sobre elas no Capítulo 16. Por enquanto, lembre-se que apenas que o Django traz 
embutido nele muitos add-ons incríveis e ``django.contrib`` é o lar onde vivem.

Ativando a Interface Admin
===========================

O admin site do Django é inteiramente opcional, porque apenas alguns tipos de sites
precisam dessa funcionalidade. Isso significa que você deverá seguir alguns passos
para ativá-lo no seu projeto.

Primeiro, faça algumas mudanças no seu arquivo de configuração (setting):


1. Acrescente ``'django.contrib.admin'`` à configuração ``INSTALLED_APPS``. (A
   ordem de ``INSTALLED_APPS`` não importa, mas gostamos da ordem alfabética
   porque é fácil para um ser humano compreender.)

2. Certifique-se de que ``INSTALLED_APPS`` contém ``'django.contrib.auth'``,
   ``'django.contrib.contenttypes'`` e ``'django.contrib.sessions'``. O site
   admin do Django precisa desses três pacotes. (Se você está acompanhando 
   nosso projeto``mysite``, observe que as linhas de código das três entradas 
   `INSTALLED_APPS`` aparecem como comentários no Capítulo 5, desfaça isso agora.)

3. Certifique-se de que ``MIDDLEWARE_CLASSES`` contém
   ``'django.middleware.common.CommonMiddleware'``,
   ``'django.contrib.sessions.middleware.SessionMiddleware'`` e
   ``'django.contrib.auth.middleware.AuthenticationMiddleware'``. (Novamente,
   no Capítulo 5, aparecem como comentários, desfaça isso agora.)

Depois, execute ``python manage.py syncdb``. Isso irá instalar as tabelas
de dados extras que a interface admin usa. Na primeira vez que você executa
``syncdb`` com ``'django.contrib.auth'`` em ``INSTALLED_APPS``, você será
solicitado a criar um superusuário. Se você não fizer isso, será necessário
executar ``python manage.py createsuperuser`` separadamente para criar uma 
conta de usuário admin; senão, você não será capaz de fazer log in no site admin. 
(Pegadinha em potencial: o comando ``python manage.py createsuperuser`` está
disponível apenas se``'django.contrib.auth'`` estiver em sua ``INSTALLED_APPS``.)

Por fim, acrescente o site admin para o seu URLconf (em ``urls.py``, lembre-se). Por
default, o ``urls.py`` gerado por ``django-admin.py startproject`` contém
código em comentário para o admin do Django, e tudo que você precisa fazer é retirá-lo
do comentário. Abaixo seguem as partes que você precisa garantir estarem lá:

    # Incluir estas declarações...
    from django.contrib import admin
    admin.autodiscover()

    # E inclua este URLpattern...
    urlpatterns = patterns('',
        # ...
        (r'^admin/', include(admin.site.urls)),
        # ...
    )

Agora você pode ver o site admin do Django em ação. Simplesmente execute o servidor 
de desenvolvimento (``python manage.py runserver``, como nos capítulos anteriores) e visite
``http://127.0.0.1:8000/admin/`` no seu Web browser.

Usando o Site Admin
====================

O site admin foi desenhado para ser utilizado por usuários não-técnicos, e como tal
é bastante auto-explicativo. Mesmo assim, iremos dar uma rápida olhada pelas características
básicas.

A primeira coisa que você verá é uma tela de login, conforme mostra a Figura 6-1.

.. figura:: graphics/chapter06/login.png
   :alt: Screenshot da tela de login do Django.

   Figura 6-1. Django's tela de login

Faça o log in com o nome de usuário e senha que você definiu quando você adicionou seu
superusuário. Se você não conseguir fazer o log in, certifique-se que você realmente
criou um superusuário -- tente exectuar ``python manage.py createsuperuser``.

Uma vez logado, a primeira coisa que você verá será a home page do administrador.
Esta página lista todos os tipos de dados disponíveis que podem ser editados no
site admin; Neste ponto, como ainda não ativamos nenhum dos nossos próprios 
modelos, a lista é pequena: ela inclui apenas Grupos e Usuários, que são os 
dois modelos de admin padrão.

.. DWP A screenshot contém livros etc.

.. figura:: graphics/chapter06/admin_index.png
   :alt: Screenshot da home page admin no Django.

   Figura 6-2. A home page admin do Django

Cada tipo de dados no admin site do Django possui uma *change list* e um
*edit form*. Change lists exibe todos os objetos disponíveis no banco de dados,
e edit forms permite adicionar, alterar ou deletar registros em seu banco de dados.

.. advertência:: Outros idiomas

    Caso sua língua materna não seja o Inglês e o seu Web browser estiver
    configurado para outro idioma, você pode fazer uma mudança rápida para
    ver se o site admin do Django foi traduzido para a sua língua.
    Adicione ``'django.middleware.locale.LocaleMiddleware'`` à sua
    ``MIDDLEWARE_CLASSES``, certificando-se que apareça *depois* de
    ``'django.contrib.sessions.middleware.SessionMiddleware'``.

    Apos fazer isso, recarregue a index page do admin. Se alguma tradução para
    seu idioma estiver disponível, várias partes da interface -- desde os links
    "Change password" e "Log out" no topo da página, até os links
    "Groups" e "Users" no meio -- aparecerão em seu idioma ao invés do inglês.
     O Django já vem com traduções para dezenas de idiomas.

    Para saber mais sobre internacionalização no Django, veja o Capítulo 19.

Clique no link "Change" na coluna "Users" para carregar a página change list para usuários.

.. figura:: graphics/chapter06/user_changelist.png
   :alt: Screenshot da página change list de usuários.

   Figura 6-3. A página change list

.. DWP Esta screenshot é na verdade uma lista para dois livros.

Esta página exibe todos os usuários no banco de dados; você pode pensá-la como 
uma versão Web embelezada de um query SQL ``SELECT * FROM auth_user;``. Se você
está acompanhando nosso exemplo, você verá apenas um usuário aqui, considerando que
você adicionou apenas um, mas tendo mais usuários as opções de filtrar, ordenar e
pesquisar passam a ser mais úteis. As opções de filtragem estão à direita, ordenar 
torna-se disponível clicando-se no cabeçalho de uma coluna e a caixa de pesquisa no 
topo permite a você fazer uma busca por nome de usuário.

Clique no nome de usuário do usuário que você criou, e você verá o edit form (formulário de 
edição) para aquele usuário.

.. figura:: graphics/chapter06/user_editform.png
   :alt: Screenshot do formulário de edição de usuários

   Figura 6-4.O formulário de edição de usuários

.. DWP A screenshot do formulário de edição para um livro.

Esta página permite a você alterar os atributos do usuário, como o
nome/sobrenome e várias permissões. (Observe que para alterar a senha de 
um usuário,você deve clicar em "change password form" sob o campo password
ao invés de editar o código hash.) Outra observação aqui é que campos de tipos
diferentes têm widgets diferentes -- por exemplo, campos date/time possuem 
controle de calendário, campos booleanos têm checkboxes, campos de caracteres têm
campos de entrada de texto.

Você pode deletar um registro clicando no botão delete na parte inferior esquerda
do seu formulário de edição. Ao fazer isso uma página de confirmação será exbidia, 
a qual, em alguns casos, exibirá quaisquer objetos dependentes que serão deletados
também. (Por exemplo, se você deletar uma editora, qualquer livro vinculado a ela
será deletado também!)

Você pode adicionar um registro clicando em "Add" na coluna apropriada da home page admin.
Isto lhe dará uma versão vazia da página de edição, pronta para que você a preencha.

Você também irá notar que a interface admin manipula dados de entrada por você. Tente 
deixar um campo obrigatório em branco ou colocar uma data inválida em um campo de datas e
você verá estes erros quando for tentar salvar, conforme mostra a Figura 6-5.

.. figura:: graphics/chapter06/user_editform_errors.png
   :alt: Screenshot de um formulário de edição exibindo erros.

   Figura 6-5. Um formulário de edição exibindo erros

.. DWP As screenshots ainda acompanham o exemplo de um livro.

Quando você edita um objeto existente, você irá perceber um link de Histórico
no canto superior direito da janela. Qualquer alteração feita através da interface
admin é registrada e você pode examinar esse registro clicando no link Histórico
(ver Figura 6-6)

.. figura:: graphics/chapter06/user_history.png
   :alt: Screenshot da página de histórico de um objeto.

   Figura 6-6. Uma página de histórico de um objeto

.. DWP Ainda usando um livro nas figuras.

Adicionando Seus Modelos ao Site Admin
======================================

Há uma parte crucial que ainda não vimos. Vamos adicionar nossos próprios modelos
ao site admin, para que possamos adicionar, alterar e deletar objetos nas tabelas
de nossa base de dados customizada usando esta bela interface. Iremos continuar
com o exemplo ``books`` do Capítulo 5, onde nós definimos três modelos: ``Publisher``, 
``Author`` e ``Book``.

Dentro do diretório ``books`` (``mysite/books``), crie um arquivo chamado
``admin.py``, e  digite as seguintes linhas de código::

    from django.contrib import admin
    from mysite.books.models import Publisher, Author, Book

    admin.site.register(Publisher)
    admin.site.register(Author)
    admin.site.register(Book)

Este código dirá ao admin site do Django para oferecer uma interface para cada um
desses modelos.

Feito isso, vá para a home page admin no seu Web browser
(``http://127.0.0.1:8000/admin/``), e  você deverá ver uma seção "Books" com links
para Authors, Books e Publishers. (Talvez seja necessário parar e reiniciar o
``runserver`` para que as mudanças surtam efeito.)

.. SL Tested ok

Agora você tem uma interface admin inteiramente funcional para cada um dos três
modelos.Foi bem fácil!

Fique à vontade para adicionar e alterar registros, popular sua base de dados
com dados. Se você seguiu os exemplos de criar objetos ``Publisher`` no Capítulo 5
(e desde que não tenha deletado-os), você verá esses registros na página publisher
de alterações.

Uma característica digna de ser mencionada é a capacidade do site admin de manusear
chavers estrangeiras e relações muitos-para-muitos, ambas as quais aparecem no modelo
``Book``. Como lembrete, aqui segue como o modelo ``Book`` se parece:

    class Book(models.Model):
        title = models.CharField(max_length=100)
        authors = models.ManyToManyField(Author)
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField()

        def __unicode__(self):
            return self.title

Na página "Add book" do site admin do Django
(``http://127.0.0.1:8000/admin/books/book/add/``), o publisher (uma
``ForeignKey``) é representada por uma caixa de seleção, e o campo authors 
(um ``ManyToManyField``) é representado por uma caixa de seleção múltipla. 
Ambos os campos ficam perto de um sinal de mais verde que permite a você adicionar
registros relacionados daquele tipo. Por exemplo, se você clicar no sinal de mais
próximo do campo "Publisher", aparecerá uma janela pop-up que permite adicionar um
publisher. Depois de criar o publisher na janela pop-up, o formulário "Add book" será
atualizado com o recém-criado publisher. Moleza.

Como o Site Admin Funciona
==========================

Como o site admin funciona? É bastante simples e direto.

When Django loads your URLconf from ``urls.py`` at server startup, it executes
the ``admin.autodiscover()`` statement that we added as part of activating the
admin. This function iterates over your ``INSTALLED_APPS`` setting and looks
for a file called ``admin.py`` in each installed app. If an ``admin.py``
exists in a given app, it executes the code in that file.

In the ``admin.py`` in our ``books`` app, each call to
``admin.site.register()`` simply registers the given model with the admin. The
admin site will only display an edit/change interface for models that have been
explicitly registered.

The app ``django.contrib.auth`` includes its own ``admin.py``, which is why
Users and Groups showed up automatically in the admin. Other ``django.contrib``
apps, such as ``django.contrib.redirects``, also add themselves to the admin,
as do many third-party Django applications you might download from the Web.

Beyond that, the Django admin site is just a Django application, with its own
models, templates, views and URLpatterns. You add it to your application by
hooking it into your URLconf, just as you hook in your own views. You can
inspect its templates, views and URLpatterns by poking around in
``django/contrib/admin`` in your copy of the Django codebase -- but don't be
tempted to change anything directly in there, as there are plenty of hooks for
you to customize the way the admin site works. (If you do decide to poke around
the Django admin application, keep in mind it does some rather complicated
things in reading metadata about models, so it would probably take a good
amount of time to read and understand the code.)

Making Fields Optional
======================

After you play around with the admin site for a while, you'll probably notice a
limitation -- the edit forms require every field to be filled out, whereas in
many cases you'd want certain fields to be optional. Let's say, for example,
that we want our ``Author`` model's ``email`` field to be optional -- that is,
a blank string should be allowed. In the real world, you might not have an
e-mail address on file for every author.

To specify that the ``email`` field is optional, edit the ``Author`` model
(which, as you'll recall from Chapter 5, lives in ``mysite/books/models.py``).
Simply add ``blank=True`` to the ``email`` field, like so:

.. parsed-literal::

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField(**blank=True**)

.. SL Tested ok

This tells Django that a blank value is indeed allowed for authors' e-mail
addresses. By default, all fields have ``blank=False``, which means blank
values are not allowed.

There's something interesting happening here. Until now, with the exception of
the ``__unicode__()`` method, our models have served as definitions of our
database tables -- Pythonic expressions of SQL ``CREATE TABLE`` statements,
essentially. In adding ``blank=True``, we have begun expanding our model beyond
a simple definition of what the database table looks like. Now, our model class
is starting to become a richer collection of knowledge about what ``Author``
objects are and what they can do. Not only is the ``email`` field represented
by a ``VARCHAR`` column in the database; it's also an optional field in
contexts such as the Django admin site.

Once you've added that ``blank=True``, reload the "Add author" edit form
(``http://127.0.0.1:8000/admin/books/author/add/``), and you'll notice the
field's label -- "Email" -- is no longer bolded. This signifies it's not a
required field. You can now add authors without needing to provide
e-mail addresses; you won't get the loud red "This field is required" message
anymore, if the field is submitted empty.

Making Date and Numeric Fields Optional
---------------------------------------

A common gotcha related to ``blank=True`` has to do with date and numeric
fields, but it requires a fair amount of background explanation.

SQL has its own way of specifying blank values -- a special value called
``NULL``. ``NULL`` could mean "unknown," or "invalid," or some other
application-specific meaning.

In SQL, a value of ``NULL`` is different than an empty string, just as the
special Python object ``None`` is different than an empty Python string
(``""``). This means it's possible for a particular character field (e.g., a
``VARCHAR`` column) to contain both ``NULL`` values and empty string values.

This can cause unwanted ambiguity and confusion: "Why does this record have a
``NULL`` but this other one has an empty string? Is there a difference, or was
the data just entered inconsistently?" And: "How do I get all the records that
have a blank value -- should I look for both ``NULL`` records and empty
strings, or do I only select the ones with empty strings?"

To help avoid such ambiguity, Django's automatically generated ``CREATE TABLE``
statements (which were covered in Chapter 5) add an explicit ``NOT NULL`` to
each column definition. For example, here's the generated statement for our
``Author`` model, from Chapter 5::

    CREATE TABLE "books_author" (
        "id" serial NOT NULL PRIMARY KEY,
        "first_name" varchar(30) NOT NULL,
        "last_name" varchar(40) NOT NULL,
        "email" varchar(75) NOT NULL
    )
    ;

In most cases, this default behavior is optimal for your application and will
save you from data-inconsistency headaches. And it works nicely with the rest
of Django, such as the Django admin site, which inserts an empty string (*not*
a ``NULL`` value) when you leave a character field blank.

But there's an exception with database column types that do not accept empty
strings as valid values -- such as dates, times and numbers. If you try to
insert an empty string into a date or integer column, you'll likely get a
database error, depending on which database you're using. (PostgreSQL, which is
strict, will raise an exception here; MySQL might accept it or might not,
depending on the version you're using, the time of day and the phase of the
moon.) In this case, ``NULL`` is the only way to specify an empty
value. In Django models, you can specify that ``NULL`` is allowed by adding
``null=True`` to a field.

So that's a long way of saying this: if you want to allow blank values in a
date field (e.g., ``DateField``, ``TimeField``, ``DateTimeField``) or numeric
field (e.g., ``IntegerField``, ``DecimalField``, ``FloatField``), you'll need
to use both ``null=True`` *and* ``blank=True``.

For sake of example, let's change our ``Book`` model to allow a blank
``publication_date``. Here's the revised code:

.. parsed-literal::

    class Book(models.Model):
        title = models.CharField(max_length=100)
        authors = models.ManyToManyField(Author)
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField(**blank=True, null=True**)

.. SL Tested ok

Adding ``null=True`` is more complicated than adding ``blank=True``, because
``null=True`` changes the semantics of the database -- that is, it changes the
``CREATE TABLE`` statement to remove the ``NOT NULL`` from the
``publication_date`` field. To complete this change, we'll need to update the
database.

For a number of reasons, Django does not attempt to automate changes to
database schemas, so it's your own responsibility to execute the appropriate
``ALTER TABLE`` statement whenever you make such a change to a model. Recall
that you can use ``manage.py dbshell`` to enter your database server's shell.
Here's how to remove the ``NOT NULL`` in this particular case::

    ALTER TABLE books_book ALTER COLUMN publication_date DROP NOT NULL;

(Note that this SQL syntax is specific to PostgreSQL.)

We'll cover schema changes in more depth in Chapter 10.

Bringing this back to the admin site, now the "Add book" edit form should allow
for empty publication date values.

Customizing Field Labels
========================

On the admin site's edit forms, each field's label is generated from its model
field name. The algorithm is simple: Django just replaces underscores with
spaces and capitalizes the first character, so, for example, the ``Book``
model's ``publication_date`` field has the label "Publication date."

However, field names don't always lend themselves to nice admin field labels,
so in some cases you might want to customize a label. You can do this by
specifying ``verbose_name`` in the appropriate model field.

For example, here's how we can change the label of the ``Author.email`` field
to "e-mail," with a hyphen:

.. parsed-literal::

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField(blank=True, **verbose_name='e-mail'**)

Make that change and reload the server, and you should see the field's new
label on the author edit form.

Note that you shouldn't capitalize the first letter of a ``verbose_name``
unless it should *always* be capitalized (e.g., ``"USA state"``). Django will
automatically capitalize it when it needs to, and it will use the exact
``verbose_name`` value in other places that don't require capitalization.

Finally, note that you can pass the ``verbose_name`` as a positional argument,
for a slightly more compact syntax. This example is equivalent to the previous
one:

.. parsed-literal::

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField(**'e-mail',** blank=True)

.. SL Tested ok

This won't work with ``ManyToManyField`` or ``ForeignKey`` fields, though,
because they require the first argument to be a model class. In those cases,
specifying ``verbose_name`` explicitly is the way to go.

Custom ModelAdmin classes
=========================

The changes we've made so far -- ``blank=True``, ``null=True`` and
``verbose_name`` -- are really model-level changes, not admin-level changes.
That is, these changes are fundamentally a part of the model and just so happen
to be used by the admin site; there's nothing admin-specific about them.

Beyond these, the Django admin site offers a wealth of options that let you
customize how the admin site works for a particular model. Such options live in
*ModelAdmin classes*, which are classes that contain configuration for a
specific model in a specific admin site instance.

Customizing change lists
------------------------

Let's dive into admin customization by specifying the fields that are
displayed on the change list for our ``Author`` model. By default, the change
list displays the result of ``__unicode__()`` for each object. In Chapter 5, we
defined the ``__unicode__()`` method for ``Author`` objects to display the
first name and last name together:

.. parsed-literal::

    class Author(models.Model):
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField(blank=True, verbose_name='e-mail')

        **def __unicode__(self):**
            **return u'%s %s' % (self.first_name, self.last_name)**

As a result, the change list for ``Author`` objects displays each other's
first name and last name together, as you can see in Figure 6-7.

.. DWP The image is of the change list for a book, not an author.

.. figure:: graphics/chapter06/author_changelist1.png
   :alt: Screenshot of the author change list page.

   Figure 6-7. The author change list page

We can improve on this default behavior by adding a few other fields to the
change list display. It'd be handy, for example, to see each author's e-mail
address in this list, and it'd be nice to be able to sort by first and last
name.

To make this happen, we'll define a ``ModelAdmin`` class for the ``Author``
model. This class is the key to customizing the admin, and one of the most
basic things it lets you do is specify the list of fields to display on change
list pages. Edit ``admin.py`` to make these changes:

.. parsed-literal::

    from django.contrib import admin
    from mysite.books.models import Publisher, Author, Book

    **class AuthorAdmin(admin.ModelAdmin):**
        **list_display = ('first_name', 'last_name', 'email')**

    admin.site.register(Publisher)
    **admin.site.register(Author, AuthorAdmin)**
    admin.site.register(Book)

.. SL Tested ok

Here's what we've done:

* We created the class ``AuthorAdmin``. This class, which subclasses
  ``django.contrib.admin.ModelAdmin``, holds custom configuration
  for a specific admin model. We've only specified one customization --
  ``list_display``, which is set to a tuple of field names to display on
  the change list page. These field names must exist in the model, of
  course.

* We altered the ``admin.site.register()`` call to add ``AuthorAdmin`` after
  ``Author``. You can read this as: "Register the ``Author`` model with the
  ``AuthorAdmin`` options."

  The ``admin.site.register()`` function takes a ``ModelAdmin`` subclass as
  an optional second argument. If you don't specify a second argument (as
  is the case for ``Publisher`` and ``Book``), Django will use the default
  admin options for that model.

With that tweak made, reload the author change list page, and you'll see it's
now displaying three columns -- the first name, last name and e-mail address.
In addition, each of those columns is sortable by clicking on the column
header. (See Figure 6-8.)

.. figure:: graphics/chapter06/author_changelist2.png
   :alt: Screenshot of the author change list page after list_display.

   Figure 6-8. The author change list page after list_display

.. DWP This figure has the same filename as the last one.

Next, let's add a simple search bar. Add ``search_fields`` to the
``AuthorAdmin``, like so:

.. parsed-literal::

    class AuthorAdmin(admin.ModelAdmin):
        list_display = ('first_name', 'last_name', 'email')
        **search_fields = ('first_name', 'last_name')**

.. SL Tested ok

Reload the page in your browser, and you should see a search bar at the top.
(See Figure 6-9.) We've just told the admin change list page to include a
search bar that searches against the ``first_name`` and ``last_name`` fields.
As a user might expect, this is case-insensitive and searches both fields, so
searching for the string ``"bar"`` would find both an author with the first
name Barney and an author with the last name Hobarson.

.. DWP Again, same screenshot.

.. figure:: graphics/chapter06/author_changelist3.png
   :alt: Screenshot of the author change list page after search_fields.

   Figure 6-9. The author change list page after search_fields

Next, let's add some date filters to our ``Book`` model's change list page:

.. parsed-literal::

    from django.contrib import admin
    from mysite.books.models import Publisher, Author, Book

    class AuthorAdmin(admin.ModelAdmin):
        list_display = ('first_name', 'last_name', 'email')
        search_fields = ('first_name', 'last_name')

    **class BookAdmin(admin.ModelAdmin):**
        **list_display = ('title', 'publisher', 'publication_date')**
        **list_filter = ('publication_date',)**

    admin.site.register(Publisher)
    admin.site.register(Author, AuthorAdmin)
    **admin.site.register(Book, BookAdmin)**

.. SL Tested ok

Here, because we're dealing with a different set of options, we created a
separate ``ModelAdmin`` class -- ``BookAdmin``. First, we defined a
``list_display`` just to make the change list look a bit nicer. Then, we
used ``list_filter``, which is set to a tuple of fields to use to create
filters along the right side of the change list page. For date fields, Django
provides shortcuts to filter the list to "Today," "Past 7 days," "This month"
and "This year" -- shortcuts that Django's developers have found hit the
common cases for filtering by date. Figure 6-10 shows what that looks like.

.. DWP Screenshot needs changing.

.. figure:: graphics/chapter06/book_changelist1.png
   :alt: Screenshot of the book change list page after list_filter.

   Figure 6-10. The book change list page after list_filter

``list_filter`` also works on fields of other types, not just ``DateField``.
(Try it with ``BooleanField`` and ``ForeignKey`` fields, for example.) The
filters show up as long as there are at least 2 values to choose from.

Another way to offer date filters is to use the ``date_hierarchy`` admin
option, like this:

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        **date_hierarchy = 'publication_date'**

.. SL Tested ok

With this in place, the change list page gets a date drill-down navigation bar
at the top of the list, as shown in Figure 6-11. It starts with a list of
available years, then drills down into months and individual days.

.. DWP Screenshot again.

.. figure:: graphics/chapter06/book_changelist2.png
   :alt: Screenshot of the book change list page after date_hierarchy.

   Figure 6-11. The book change list page after date_hierarchy

Note that ``date_hierarchy`` takes a *string*, not a tuple, because only one
date field can be used to make the hierarchy.

Finally, let's change the default ordering so that books on the change list
page are always ordered descending by their publication date. By default,
the change list orders objects according to their model's ``ordering`` within
``class Meta`` (which we covered in Chapter 5) -- but you haven't specified
this ``ordering`` value, then the ordering is undefined.

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        **ordering = ('-publication_date',)**

.. SL Tested ok

This admin ``ordering`` option works exactly as the ``ordering`` in models'
``class Meta``, except that it only uses the first field name in the list. Just
pass a list or tuple of field names, and add a minus sign to a field to use
descending sort order.

Reload the book change list to see this in action. Note that the
"Publication date" header now includes a small arrow that indicates which way
the records are sorted. (See Figure 6-12.)

.. DWP Different screenshot needed.

.. figure:: graphics/chapter06/book_changelist3.png
   :alt: Screenshot of the book change list page after ordering.

   Figure 6-12. The book change list page after ordering

We've covered the main change list options here. Using these options, you can
make a very powerful, production-ready data-editing interface with only a few
lines of code.

Customizing edit forms
----------------------

Just as the change list can be customized, edit forms can be customized in many
ways.

First, let's customize the way fields are ordered. By default, the order of
fields in an edit form corresponds to the order they're defined in the model.
We can change that using the ``fields`` option in our ``ModelAdmin`` subclass:

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        ordering = ('-publication_date',)
        **fields = ('title', 'authors', 'publisher', 'publication_date')**

.. SL Tested ok

After this change, the edit form for books will use the given ordering for
fields. It's slightly more natural to have the authors after the book title.
Of course, the field order should depend on your data-entry workflow. Every
form is different.

Another useful thing the ``fields`` option lets you do is to *exclude* certain
fields from being edited entirely. Just leave out the field(s) you want to
exclude. You might use this if your admin users are only trusted to edit a
certain segment of your data, or if part of your fields are changed by some
outside, automated process. For example, in our book database, we could
hide the ``publication_date`` field from being editable:

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        ordering = ('-publication_date',)
        **fields = ('title', 'authors', 'publisher')**

.. SL Tested ok

As a result, the edit form for books doesn't offer a way to specify the
publication date. This could be useful, say, if you're an editor who prefers
that his authors not push back publication dates. (This is purely a
hypothetical example, of course.)

When a user uses this incomplete form to add a new book, Django will simply set
the ``publication_date`` to ``None`` -- so make sure that field has
``null=True``.

Another commonly used edit-form customization has to do with many-to-many
fields. As we've seen on the edit form for books, the admin site represents each
``ManyToManyField`` as a multiple-select boxes, which is the most logical
HTML input widget to use -- but multiple-select boxes can be difficult to use.
If you want to select multiple items, you have to hold down the control key,
or command on a Mac, to do so. The admin site helpfully inserts a bit of text
that explains this, but, still, it gets unwieldy when your field contains
hundreds of options.

The admin site's solution is ``filter_horizontal``. Let's add that to
``BookAdmin`` and see what it does.

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        ordering = ('-publication_date',)
        **filter_horizontal = ('authors',)**

.. SL Tested ok

(If you're following along, note that we've also removed the ``fields`` option
to restore all the fields in the edit form.)

Reload the edit form for books, and you'll see that the "Authors" section now
uses a fancy JavaScript filter interface that lets you search through the
options dynamically and move specific authors from "Available authors" to
the "Chosen authors" box, and vice versa.

.. DWP screenshot!

.. figure:: graphics/chapter06/book_editform1.png
   :alt: Screenshot of the book edit form after adding filter_horizontal.

   Figure 6-13. The book edit form after adding filter_horizontal

We'd highly recommend using ``filter_horizontal`` for any ``ManyToManyField``
that has more than 10 items. It's far easier to use than a simple
multiple-select widget. Also, note you can use ``filter_horizontal``
for multiple fields -- just specify each name in the tuple.

``ModelAdmin`` classes also support a ``filter_vertical`` option. This works
exactly as ``filter_horizontal``, but the resulting JavaScript interface stacks
the two boxes vertically instead of horizontally. It's a matter of personal
taste.

``filter_horizontal`` and ``filter_vertical`` only work on ``ManyToManyField``
fields, not ``ForeignKey`` fields. By default, the admin site uses simple
``<select>`` boxes for ``ForeignKey`` fields, but, as for ``ManyToManyField``,
sometimes you don't want to incur the overhead of having to select all the
related objects to display in the drop-down. For example, if our book database
grows to include thousands of publishers, the "Add book" form could take a
while to load, because it would have to load every publisher for display in the
``<select>`` box.

The way to fix this is to use an option called ``raw_id_fields``. Set this to
a tuple of ``ForeignKey`` field names, and those fields will be displayed in
the admin with a simple text input box (``<input type="text">``) instead of a
``<select>``. See Figure 6-14.

.. parsed-literal::

    class BookAdmin(admin.ModelAdmin):
        list_display = ('title', 'publisher', 'publication_date')
        list_filter = ('publication_date',)
        date_hierarchy = 'publication_date'
        ordering = ('-publication_date',)
        filter_horizontal = ('authors',)
        **raw_id_fields = ('publisher',)**

.. SL Tested ok

.. DWP Screenshot!

.. figure:: graphics/chapter06/book_editform2.png
   :alt: Screenshot of edit form after raw_id_fields.

   Figure 6-14. The book edit form after adding raw_id_fields

What do you enter in this input box? The database ID of the publisher. Given
that humans don't normally memorize database IDs, there's also a
magnifying-glass icon that you can click to pull up a pop-up window, from which
you can select the publisher to add.

Users, Groups, and Permissions
==============================

Because you're logged in as a superuser, you have access to create, edit, and
delete any object. Naturally, different environments require different
permission systems -- not everybody can or should be a superuser. Django's
admin site uses a permissions system that you can use to give specific users
access only to the portions of the interface that they need.

These user accounts are meant to be generic enough to be used outside of the
admin interface, but we'll just treat them as admin user accounts for now. In
Chapter 14, we'll cover how to integrate user accounts with the rest of your
site (i.e., not just the admin site).

You can edit users and permissions through the admin interface just like any
other object. We saw this earlier in this chapter, when we played around with
the User and Group sections of the admin. User objects have the standard
username, password, e-mail and real name fields you might expect, along with a
set of fields that define what the user is allowed to do in the admin
interface. First, there's a set of three boolean flags:

* The "active" flag controls whether the user is active at all.
  If this flag is off and the user tries to log in, he won't be allowed in,
  even with a valid password.

* The "staff" flag controls whether the user is allowed to log in to the
  admin interface (i.e., whether that user is considered a "staff member" in
  your organization). Since this same user system can be used to control
  access to public (i.e., non-admin) sites (see Chapter 14), this flag
  differentiates between public users and administrators.

* The "superuser" flag gives the user full access to add, create and
  delete any item in the admin interface. If a user has this flag set, then
  all regular permissions (or lack thereof) are ignored for that user.

"Normal" admin users -- that is, active, non-superuser staff members -- are
granted admin access through assigned permissions. Each object editable through
the admin interface (e.g., books, authors, publishers) has three permissions: a
*create* permission, an *edit* permission and a *delete* permission. Assigning
permissions to a user grants the user access to do what is described by those
permissions.

When you create a user, that user has no permissions, and it's up to you to
give the user specific permissions. For example, you can give a user permission
to add and change publishers, but not permission to delete them. Note that
these permissions are defined per-model, not per-object -- so they let you say
"John can make changes to any book," but they don't let you say "John can make
changes to any book published by Apress." The latter functionality, per-object
permissions, is a bit more complicated and is outside the scope of this book
but is covered in the Django documentation.

.. note::

    Access to edit users and permissions is also controlled by this permission
    system. If you give someone permission to edit users, she will be able to
    edit her own permissions, which might not be what you want! Giving a user
    permission to edit users is essentially turning a user into a superuser.

You can also assign users to groups. A *group* is simply a set of permissions to
apply to all members of that group. Groups are useful for granting identical
permissions to a subset of users.

When and Why to Use the Admin Interface -- And When Not to
==========================================================

After having worked through this chapter, you should have a good idea of how to
use Django's admin site. But we want to make a point of covering *when* and
*why* you might want to use it -- and when *not* to use it.

Django's admin site especially shines when nontechnical users need to be able
to enter data; that's the purpose behind the feature, after all. At the
newspaper where Django was first developed, development of a typical online
feature -- say, a special report on water quality in the municipal supply --
would go something like this:

* The reporter responsible for the project meets with one of the developers
  and describes the available data.

* The developer designs Django models to fit this data and then opens up
  the admin site to the reporter.

* The reporter inspects the admin site to point out any missing or
  extraneous fields -- better now than later. The developer changes the
  models iteratively.

* When the models are agreed upon, the reporter begins entering data using
  the admin site. At the same time, the programmer can focus on developing
  the publicly accessible views/templates (the fun part!).

In other words, the raison d'être of Django's admin interface is facilitating
the simultaneous work of content producers and programmers.

However, beyond these obvious data entry tasks, the admin site is useful in a
few other cases:

* *Inspecting data models*: Once you've defined a few models, it can be
  quite useful to call them up in the admin interface and enter some dummy
  data. In some cases, this might reveal data-modeling mistakes or other
  problems with your models.

* *Managing acquired data*: For applications that rely on data coming from
  external sources (e.g., users or Web crawlers), the admin site gives you
  an easy way to inspect or edit this data. You can think of it as a less
  powerful, but more convenient, version of your database's command-line
  utility.

* *Quick and dirty data-management apps*: You can use the admin site to
  build yourself a very lightweight data management app -- say, to keep
  track of expenses. If you're just building something for your own needs,
  not for public consumption, the admin site can take you a long way. In
  this sense, you can think of it as a beefed up, relational version of a
  spreadsheet.

One final point we want to make clear is: the admin site is not an
end-all-be-all. Over the years, we've seen it hacked and chopped up to serve a
variety of functions it wasn't intended to serve. It's not intended to be a
*public* interface to data, nor is it intended to allow for sophisticated
sorting and searching of your data. As we said early in this chapter, it's for
trusted site administrators. Keeping this sweet spot in mind is the key to
effective admin-site usage.

What's Next?
============

So far we've created a few models and configured a top-notch interface for
editing data. In the :doc:`next chapter <chapter07>`, we'll move on to the real "meat and potatoes"
of Web development: form creation and processing.
