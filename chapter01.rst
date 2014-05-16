=================================
Capítulo 1: Introdução ao Django
=================================

Este livro é sobre Django, um framework de desenvolvimento web que otimiza seu tempo
e torna o desenvolvimento web divertido. Usando Django, você pode construir e manter 
aplicações web de alta qualidade com mínimo trabalho.

No seu melhor, o desenvolvimento web pode ser emocionante, criar coisas, fazer funcionar,
no pior, pode ser repetitivo, frustrantemente chato. Django deixa seu foco na coisa divertida,
o crucial em sua aplicação, enquanto as partes repetitivas são deixadas de lado.
Ao fazer isso, são providos abstrações de algo nível comuns no padrão de desenvolvimento Web,
atalhos para tarefas frequentes de programação, e conveções claras para resolução de problemas.
Ao mesmo tempo, Django tenta ficar fora do seu caminho,
deixando você trabalhar com liberdade de ação no framework como é necessário.
O objetivo deste livro é tornar você um especialista em Django. Temos um foco duplo.
Primeiro, nós explicamos, intimamente, o que o Django faz e como construir aplicações Web com ele.
Segundo, nós discutiremos conceitos de alto nível apropriado, respondendo a questão
"Como posso aplicar estas ferramentas efetivamente nos meus projetos?". Lendo este livro,
Você aprenderá as ferramentas e habilidades necessárias para desenvolver poderosas
aplicaçoes Web rapidamente, com código limpo e fácil de manter.

O que é um Framework?
=====================
Django é um proeminente membro de uma nova geração de "Frameworks Web", porém o que significa isso precisamente?

Para responder a essa questão, vamos considerar o design de uma aplicação web escrita em Python "sem" um framework.
Ao avançar este livro, mostraremos formas básicas de fazer todo o trabalho "sem" atalhos, na esperança que você reconheça
porque atalhos ajudam tanto.(Também é de extrema valia saber fazer o trabalho sem os atalhos, pois os atalhos nem sempre estão disponíveis.
E o mais importante, saber "porque" as coisas funcionam desta forma tornar você um melhor desenvolvedor Web.)

Uma das mais simples e mais diretas maneiras de criar uma Aplicação Web em Python do desenho é usar o padrão CGI(Common Gateway Interface),
que foi uma técnica muito popular em torno de 1998. Aqui está uma explicação em alto nível de como funciona: Crie um script Python que tem como saída
HTML, então salve o script em um Servidor Web com a extensão ".cgi" e visite a página em seu navegador, é isso.

Aqui está um exemplo de um script CGI que mostra os livros publicados recentemente a partir de um banco de dados.
Não ligue para os detalhes de sintaxe; atente somente sobre as coisas básicas que o script faz::

    #!/usr/bin/env python

    import MySQLdb

    print "Content-Type: text/html\n"
    print "<html><head><title>Books</title></head>"
    print "<body>"
    print "<h1>Books</h1>"
    print "<ul>"

    connection = MySQLdb.connect(user='me', passwd='letmein', db='my_db')
    cursor = connection.cursor()
    cursor.execute("SELECT name FROM books ORDER BY pub_date DESC LIMIT 10")

    for row in cursor.fetchall():
        print "<li>%s</li>" % row[0]

    print "</ul>"
    print "</body></html>"

    connection.close()

Primeiramente, para preencher os requisitos da CGI, este código imprime uma linha
"Content-Type", seguida de uma linha em branco. É impresso a abertura do arquivo HTML,
conecta a um banco de dados e executa uma query para obter os nomes dos últimos dez livros.
Passando pelo registro dos livros, ele gera uma lista em HTML dos títulos. Finalmente, ele
imprime o fechamento do HTML e fecha a conexão com o banco de dados.

Com uma página deste tipo, a abordagem de escrita desde o rascunho não é necessariamente ruim.
Por uma lado, este código é simples de compreender -- até mesmo um desenvolvedor não experiente
pode ler estas 16 linhas de Python e entender tudo o que faz, do início ao fim.
Não há nada a aprender, nada que não esteja explícito no código.
É também simples de se fazer o deploy: basta salvar este código em um arquivo que tenha a extensão
".cgi", fazer upload do mesmo em um Servidor Web e visitar esta página com um navegador.
Mas esqueça a simplicidade, esta abordagem tem um número de problemas e não segue boa práticas.
Faça a si mesmo as seguintes perguntas:

* O que aconteceria quando várias partes da sua aplicação precisassem se conectar ao banco de dados?
  Certamente a conexão de bancos de dados não deveriam estar duplicadas em cada script CGI.
  O modo pragmático de se fazer deveria ser refatorar em uma função compartilhada.

* Um desenvolvedor deveria realmente se preocupar em mostrar a linha "Content-Type" 
  e lembrar de fechar a conexão com o banco de dados?
  Este tipo de coisa, reduz a produtividade do programador e induz a oportunidade de erro.
  Estas tarefas e configurações deveriam ser tarefas melhores a serem cuidadas por uma infraestrutura em comum.
  
* O que aconteceria quando este código fosse reutilizado em vários ambientes, cada um com seu banco de dados separado
  ou outra senha? Neste ponto, algumas configurações de ambiente tornan-se essenciais.

* O que aconteceria se um Web Designer que não tem experiência em Python quisesse refazer a página?
  Um caracter errado poderia acabar com toda a aplicação. Como padrão ideal, a lógica da página --
  a resposta dos títulos de livros do banco de dados -- deveriam ser separadas do código HTML da página, então um 
  designer poderia editar toda a mesma sem afetar a lógica.

Estes problemas são precisamente o que um framework web intenciona resolver.
Um framework web provê a infraestrutura de programação para suas aplicações, então você pode
focar em escrever um código limpo, de fácil manutenção sem ter de reinventar a roda.
Na sua essência, é o que o Django faz.


The MVC Design Pattern
======================

Let's dive in with a quick example that demonstrates the difference between the
previous approach and a Web framework's approach. Here's how you might write
the previous CGI code using Django. The first thing to note is that that we
split it over four Python files (``models.py``, ``views.py``, ``urls.py``) and
an HTML template (``latest_books.html``)::

    # models.py (the database tables)

    from django.db import models

    class Book(models.Model):
        name = models.CharField(max_length=50)
        pub_date = models.DateField()


    # views.py (the business logic)

    from django.shortcuts import render
    from models import Book

    def latest_books(request):
        book_list = Book.objects.order_by('-pub_date')[:10]
        return render(request, 'latest_books.html', {'book_list': book_list})


    # urls.py (the URL configuration)

    from django.conf.urls.defaults import *
    import views

    urlpatterns = patterns('',
        (r'^latest/$', views.latest_books),
    )


    # latest_books.html (the template)

    <html><head><title>Books</title></head>
    <body>
    <h1>Books</h1>
    <ul>
    {% for book in book_list %}
    <li>{{ book.name }}</li>
    {% endfor %}
    </ul>
    </body></html>

Again, don't worry about the particulars of syntax; just get a feel for the
overall design. The main thing to note here is the *separation of concerns*:

* The ``models.py`` file contains a description of the database table,
  represented by a Python class. This class is called a *model*. Using it,
  you can create, retrieve, update and delete records in your database
  using simple Python code rather than writing repetitive SQL statements.

* The ``views.py`` file contains the business logic for the page. The
  ``latest_books()`` function is called a *view*.

* The ``urls.py`` file specifies which view is called for a given URL
  pattern. In this case, the URL ``/latest/`` will be handled by the
  ``latest_books()`` function. In other words, if your domain is
  example.com, any visit to the URL http://example.com/latest/ will call
  the ``latest_books()`` function.

* The ``latest_books.html`` file is an HTML template that describes the
  design of the page. It uses a template language with basic logic
  statements -- e.g., ``{% for book in book_list %}``.

Taken together, these pieces loosely follow a pattern called
Model-View-Controller (MVC). Simply put, MVC is way of developing software so
that the code for defining and accessing data (the model) is separate from
request-routing logic (the controller), which in turn is separate from the user
interface (the view). (We'll discuss MVC in more depth in Chapter 5.)

A key advantage of such an approach is that components are *loosely coupled*.
Each distinct piece of a Django-powered Web application has a single key
purpose and can be changed independently without affecting the other pieces.
For example, a developer can change the URL for a given part of the application
without affecting the underlying implementation. A designer can change a page's
HTML without having to touch the Python code that renders it. A database
administrator can rename a database table and specify the change in a single
place, rather than having to search and replace through a dozen files.

In this book, each component of MVC gets its own chapter. Chapter 3 covers
views, Chapter 4 covers templates, and Chapter 5 covers models.

Django's History
================

Before we dive into more code, we should take a moment to explain Django's
history. We noted above that we'll be showing you how to do things *without*
shortcuts so that you more fully understand the shortcuts. Similarly, it's
useful to understand *why* Django was created, because knowledge of the history
will put into context why Django works the way it does.

If you've been building Web applications for a while, you're probably familiar
with the problems in the CGI example we presented earlier. The classic Web
developer's path goes something like this:

1. Write a Web application from scratch.
2. Write another Web application from scratch.
3. Realize the application from step 1 shares much in common with the
   application from step 2.
4. Refactor the code so that application 1 shares code with application 2.
5. Repeat steps 2-4 several times.
6. Realize you've invented a framework.

This is precisely how Django itself was created!

Django grew organically from real-world applications written by a Web
development team in Lawrence, Kansas, USA. It was born in the fall of 2003,
when the Web programmers at the *Lawrence Journal-World* newspaper, Adrian
Holovaty and Simon Willison, began using Python to build applications.

The World Online team, responsible for the production and maintenance of
several local news sites, thrived in a development environment dictated by
journalism deadlines. For the sites -- including LJWorld.com, Lawrence.com and
KUsports.com -- journalists (and management) demanded that features be added
and entire applications be built on an intensely fast schedule, often with only
days' or hours' notice. Thus, Simon and Adrian developed a time-saving Web
development framework out of necessity -- it was the only way they could build
maintainable applications under the extreme deadlines.

In summer 2005, after having developed this framework to a point where it was
efficiently powering most of World Online's sites, the team, which now included
Jacob Kaplan-Moss, decided to release the framework as open source software.
They released it in July 2005 and named it Django, after the jazz guitarist
Django Reinhardt.

Now, several years later, Django is a well-established open source project with
tens of thousands of users and contributors spread across the planet. Two of
the original World Online developers (the "Benevolent Dictators for Life,"
Adrian and Jacob) still provide central guidance for the framework's growth,
but it's much more of a collaborative team effort.

This history is relevant because it helps explain two key things. The first is
Django's "sweet spot." Because Django was born in a news environment, it offers
several features (such as its admin site, covered in Chapter 6) that are
particularly well suited for "content" sites -- sites like Amazon.com,
craigslist.org, and washingtonpost.com that offer dynamic, database-driven
information. Don't let that turn you off, though -- although Django is
particularly good for developing those sorts of sites, that doesn't preclude it
from being an effective tool for building any sort of dynamic Web site.
(There's a difference between being *particularly effective* at something and
being *ineffective* at other things.)

The second matter to note is how Django's origins have shaped the culture of
its open source community. Because Django was extracted from real-world code,
rather than being an academic exercise or commercial product, it is acutely
focused on solving Web development problems that Django's developers themselves
have faced -- and continue to face. As a result, Django itself is actively
improved on an almost daily basis. The framework's maintainers have a vested
interest in making sure Django saves developers time, produces applications
that are easy to maintain and performs well under load. If nothing else, the
developers are motivated by their own selfish desires to save themselves time
and enjoy their jobs. (To put it bluntly, they eat their own dog food.)

.. AH The following sections are the type of content that typically appears
.. AH in a book's Introduction section, but we include it here because this
.. AH chapter serves as an introduction.

How to Read This Book
=====================

In writing this book, we tried to strike a balance between readability and
reference, with a bias toward readability. Our goal with this book, as stated
earlier, is to make you a Django expert, and we believe the best way to teach is
through prose and plenty of examples, rather than providing an exhaustive
but bland catalog of Django features. (As the saying goes, you can't expect to
teach somebody how to speak a language merely by teaching them the alphabet.)

With that in mind, we recommend that you read Chapters 1 through 12 in order.
They form the foundation of how to use Django; once you've read them, you'll be
able to build and deploy Django-powered Web sites. Specifically, Chapters 1
through 7 are the "core curriculum," Chapters 8 through 11 cover more advanced
Django usage, and Chapter 12 covers deployment. The remaining chapters, 13
through 20, focus on specific Django features and can be read in any order.

The appendixes are for reference. They, along with the free documentation at
http://www.djangoproject.com/, are probably what you'll flip back to occasionally to
recall syntax or find quick synopses of what certain parts of Django do.

Required Programming Knowledge
------------------------------

Readers of this book should understand the basics of procedural and
object-oriented programming: control structures (e.g., ``if``, ``while``,
``for``), data structures (lists, hashes/dictionaries), variables, classes and
objects.

Experience in Web development is, as you may expect, very helpful, but it's
not required to understand this book. Throughout the book, we try to promote
best practices in Web development for readers who lack this experience.

Required Python Knowledge
-------------------------

At its core, Django is simply a collection of libraries written in the Python
programming language. To develop a site using Django, you write Python code
that uses these libraries. Learning Django, then, is a matter of learning how
to program in Python and understanding how the Django libraries work.

If you have experience programming in Python, you should have no trouble diving
in. By and large, the Django code doesn't perform a lot of "magic" (i.e.,
programming trickery whose implementation is difficult to explain or
understand). For you, learning Django will be a matter of learning Django's
conventions and APIs.

If you don't have experience programming in Python, you're in for a treat.
It's easy to learn and a joy to use! Although this book doesn't include a full
Python tutorial, it highlights Python features and functionality where
appropriate, particularly when code doesn't immediately make sense. Still, we
recommend you read the official Python tutorial, available online at
http://docs.python.org/tut/. We also recommend Mark Pilgrim's free book
*Dive Into Python*, available at http://www.diveintopython.net/ and published in
print by Apress.

Required Django Version
-----------------------

This book covers Django 1.4.

Django's developers maintain backwards compatibility as much as possible, but
occasionally introduce some backwards incompatible changes.  The changes in each
release are always covered in the release notes, which you can find here:
https://docs.djangoproject.com/en/dev/releases/1.X


Getting Help
------------

One of the greatest benefits of Django is its kind and helpful user community.
For help with any aspect of Django -- from installation, to application design,
to database design, to deployment -- feel free to ask questions online.

* The django-users mailing list is where thousands of Django users hang out
  to ask and answer questions. Sign up for free at http://www.djangoproject.com/r/django-users.

* The Django IRC channel is where Django users hang out to chat and help
  each other in real time. Join the fun by logging on to #django on the
  Freenode IRC network.

What's Next
-----------

In :doc:`chapter02`, we'll get started with Django, covering installation and
initial setup.
