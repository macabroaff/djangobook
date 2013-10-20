======================
Capítulo 20: Segurança
======================

A Internet pode ser um lugar assustador.

Atualmente, gafes de segurança de alto nível parecem surgir diariamente. Temos visto
vírus sendo espalhados em uma velocidade incrível, enxames de computadores comprometidos exercido como
armas, a corrida armamentista incessante contra spammers, e muitos, muitos relatos de
roubo de identidade de sites hackeados.

Como desenvolvedores Web, temos o dever de fazer o que for necessário para combater essas forças
das trevas. Todo desenvolvedor Web precisa encarar a segurança como um aspecto fundamental
da programação web. Infelizmente, verifica-se que implementar a segurança é *difícil*
- Atacantes precisam achar apenas uma vulnerabilidade, mas os defensores têm que
proteger cada uma delas.

O Django tenta atenuar esta dificuldade. Ele foi projetado para automaticamente
protegê-lo de muitos dos erros comuns de segurança que novos (e mesmo
experientes) desenvolvedores Web cometem. Mesmo assim, é importante entender quais
são esses problemas, como o Django protege você, e - mais importante - os
passos que você pode tomar para tornar o código mais seguro.

Primeiro, porém, um aviso importante: Não temos a intenção de apresentar um
guia definitivo para todos os exploits de segurança Web conhecidos, e por isso não vamos tentar
explicar cada vulnerabilidade de forma abrangente. Em vez disso, vamos dar um 
breve resumo de como os problemas de segurança se aplicam ao Django.

O Tema da Segurança Web
=======================

Se você aprender apenas uma coisa desse capítulo, será isso:

Nunca -- sob quaisquer circunstância -- confie nos dados de um navegador.

Você *nunca* sabe quem está do outro lado da conexão HTTP. Seria um de seus usuários,
mas poderia facilmente ser um cracker nefasto procurando por uma abertura.

Quaisquer dados de qualquer natureza vindo do navegador precisa ser tratado
com uma saudável dose de paranóia. Isso inclue dados que sãp tanto "na banda" (ou seja, 
submetido pelos formulários Web) e "fora da banda" (ou seja, cabeçalhos HTTP, cookies,
e outras informações requisitadas). É trivial falsificar requisições metadata que
navegadores geralmente adicionam automaticamente.

Cada uma das vulnerabilidades abordadas nesse capítulo decorrem de confiar diretamente
em dados vindos da rede e depois a falta da limpeza dos dados antes de usá-los. Você 
deveria fazer a prática geral de se questionar continuamente, "De onde os dados vieram?"

SQL Injection
=============

*Injeção de SQL* é um exploit comum na qual um atacante altera os parâmetros
da página Web (tal como dados ``GET``/``POST`` ou URLs) para inserir trechos
arbitrários de SQL que uma ingênua aplicação Web executa diratamente em seu
banco de dados. É provavelmente a mais perigosa -- e, infelizmente, a mais
comum -- vulnerabilidade lá fora.

Essa vulnerabilidade geralmente surge ao construir o SQL "na mão" pela entrada do
usuário. Por exemplo, imagine escrever uma função para reunir uma lista de informações
de contato de uma página de busca de contatos. Para prevenir spammers de ler cada
um dos e-mails de seu sistema, nós forçamos o usário a digitar o nome do usuário de 
alguém antes de fornecer seu endereço de email::

    def user_contacts(request):
        user = request.GET['username']
        sql = "SELECT * FROM user_contacts WHERE username = '%s';" % username
        # executa o SQL aqui...

.. nota::

    Nesse exemplo, e em todos os exemplos "não faça isso" similadres a seguir,
    nós deliberadamente deixamos de fora a maioria do código necessário para
    fazer as funções realmente funcionarem. Nós não queremos que o código funcione,
    se alguém acidentalmente levá-lo fora do contexto.

Embora a princípio isso não pareça perigoso, ele realmente é.

Primeiramente, a nossa tentativa de proteger nossa lista inteira de e-mail falhará 
com uma query habilmente construída. Pense sobre o que acontece se o atacante digitar
``"' OR 'a'='a"`` na caixa de consulta. Nesse caso, a query que a interpolação da string
irá construir será::

    SELECT * FROM user_contacts WHERE username = '' OR 'a' = 'a';

Por permitimos um SQL inseguro na string, o atancante adicionou a cláusula ``OR``
garantindo que cara linha seja retornada.

No entanto, esse é o ataque *menos* assustador. Imagine o que acontece se cada
atacante enviar ``"'; DELETE FROM user_contacts WHERE 'a' = 'a"``. Vamos acabar
com essa query completa::

    SELECT * FROM user_contacts WHERE username = ''; DELETE FROM user_contacts WHERE 'a' = 'a';

Caramba! Nossa lista de contatos inteira será deletada imediatamente.

A Solução
---------

Embora esse problema é insidioso e as vezes difícil de detectar, a solução
é simples: *nunca* confie nos dados submitidos pelo usuário, e *sempre* escape 
ao passar para o SQL.

O banco de dados do Django API faz isso para você. Ele automaticamente escapa
todos os parâmetros especiais SQL, de acordo com convenções de citação do
servidor do banco de dados que você está usando (e.x., PostgreSQL ou MySQL).

Por exemplo, nessa chamada API::

    foo.get_list(bar__exact="' OR 1=1")

O Django irá escapar a entrada nesse sentido, resultando em uma declaração como esta::

    SELECT * FROM foos WHERE bar = '\' OR 1=1'

Completamente inofensivo.    

Isso se aplica a todos os bancos de dados do Django API, com algumas excessões:

* O método ``where`` argumenta para o ``extra()``. (Olhe o Apêndice C.)
  Esse parâmetro aceita SQL puro por design.

* Queries feitas "na mão" usando o banco de dados de baixo nível da API. (Veja Capítulo 10.)

Em cada um dos casos, é fácil manter-se protegido. Em cada caso, evite a interpolação 
de string para favorecer a passagem de *bind parameters*. Ou seja, o exemplo que iniciou
esse seção deve ser escrita seguinte forma::

    from django.db import connection

    def user_contacts(request):
        user = request.GET['username']
        sql = "SELECT * FROM user_contacts WHERE username = %s"
        cursor = connection.cursor()
        cursor.execute(sql, [user])
        # ... do something with the results

O método baixo-nível ` execute`` pega a string SQL com o espaço reservado ``%s`` 
e automaticamente escapa e insere parâmetros da lista passada como segundo argumento.
Você deve *sempre* construir SQL personalizados dessa forma.
        
Infelizmente, você não pode usar parâmetros de vinculação em todos os lugares no SQL;
eles não são permitidos como identificadores (ou seja, tabelas ou nomes de colunas).
Assim, se você precisar, dizer, construir dinamicamente uma lista de tabelas de uma
variável ``POST``, você precisa escapar o nome no seu código. O Django fornece uma função,
``django.db.connection.ops.quote_name``, que vai escapar o identificador de acordo
com o esquema de citação atual do banco de dados.

Cross-Site Scripting (XSS)
==========================

*Cross-site scripting* (XSS), é encontrado em aplicações Web que falham ao
escapar conteúdos submetidos corretamente pelo usuário antes de renderizar no
HTML. Isso permite ao atacante inserir HTML arbitrário na sua página Web, 
geralmente na forma da tag ``<script>``.

Attackers often use XSS attacks to steal cookie and session information, or to trick
users into giving private information to the wrong person (aka *phishing*).

This type of attack can take a number of different forms and has almost
infinite permutations, so we'll just look at a typical example. Consider this
extremely simple "Hello, World" view::

    from django.http import HttpResponse

    def say_hello(request):
        name = request.GET.get('name', 'world')
        return HttpResponse('<h1>Hello, %s!</h1>' % name)

This view simply reads a name from a ``GET`` parameter and passes that name
into the generated HTML. So, if we accessed
``http://example.com/hello/?name=Jacob``, the page would contain this::

    <h1>Hello, Jacob!</h1>

But wait -- what happens if we access
``http://example.com/hello/?name=<i>Jacob</i>``? Then we get this::

    <h1>Hello, <i>Jacob</i>!</h1>

Of course, an attacker wouldn't use something as benign as ``<i>`` tags; he
could include a whole set of HTML that hijacked your page with arbitrary
content. This type of attack has been used to trick users into entering data
into what looks like their bank's Web site, but in fact is an XSS-hijacked form
that submits their back account information to an attacker.

The problem gets worse if you store this data in the database and later display it
it on your site. For example, MySpace was once found to be vulnerable to an XSS
attack of this nature. A user inserted JavaScript into his profile that automatically
added him as your friend when you visited his profile page. Within a few days, he had
millions of friends.

Now, this may sound relatively benign, but keep in mind that this attacker
managed to get *his* code -- not MySpace's -- running on *your* computer. This
violates the assumed trust that all the code on MySpace is actually written
by MySpace.

MySpace was extremely lucky that this malicious code didn't automatically
delete viewers' accounts, change their passwords, flood the site with spam, or
any of the other nightmare scenarios this vulnerability unleashes.

The Solution
------------

The solution is simple: *always* escape *any* content that might have come
from a user before inserting it into HTML.

To guard against this, Django's template system automatically escapes all
variable values. Let's see what happens if we rewrite our example using the
template system::

    # views.py

    from django.shortcuts import render

    def say_hello(request):
        name = request.GET.get('name', 'world')
        return render(request, 'hello.html', {'name': name})

    # hello.html

    <h1>Hello, {{ name }}!</h1>

With this in place, a request to ``http://example.com/hello/name=<i>Jacob</i>``
will result in the following page::

    <h1>Hello, &lt;i&gt;Jacob&lt;/i&gt;!</h1>

We covered Django's auto-escaping back in Chapter 4, along with ways to turn
it off. But even if you're using this feature, you should *still* get in the
habit of asking yourself, at all times, "Where does this data come from?" No
automatic solution will ever protect your site from XSS attacks 100% of the
time.

Cross-Site Request Forgery
==========================

Cross-site request forgery (CSRF) happens when a malicious Web site tricks users
into unknowingly loading a URL from a site at which they're already authenticated --
hence taking advantage of their authenticated status.

Django has built-in tools to protect from this kind of attack. Both the attack
itself and those tools are covered in great detail in `Chapter 16`_.

Session Forging/Hijacking
=========================

This isn't a specific attack, but rather a general class of attacks on a
user's session data. It can take a number of different forms:

* A *man-in-the-middle* attack, where an attacker snoops on session data
  as it travels over the wire (or wireless) network.

* *Session forging*, where an attacker uses a session ID
  (perhaps obtained through a man-in-the-middle attack) to pretend to be
  another user.

  An example of these first two would be an attacker in a coffee shop using
  the shop's wireless network to capture a session cookie. She could then use that
  cookie to impersonate the original user.

* A *cookie-forging* attack, where an attacker overrides the supposedly
  read-only data stored in a cookie. `Chapter 14`_ explains in detail how
  cookies work, and one of the salient points is that it's trivial for
  browsers and malicious users to change cookies without your knowledge.

  There's a long history of Web sites that have stored a cookie like
  ``IsLoggedIn=1`` or even ``LoggedInAsUser=jacob``. It's dead simple to
  exploit these types of cookies.

  On a more subtle level, though, it's never a good idea to trust anything
  stored in cookies. You never know who's been poking at them.

* *Session fixation*, where an attacker tricks a user into setting or
  reseting the user's session ID.

  For example, PHP allows session identifiers to be passed in the URL
  (e.g.,
  ``http://example.com/?PHPSESSID=fa90197ca25f6ab40bb1374c510d7a32``). An
  attacker who tricks a user into clicking a link with a hard-coded
  session ID will cause the user to pick up that session.

  Session fixation has been used in phishing attacks to trick users into entering
  personal information into an account the attacker owns. He can
  later log into that account and retrieve the data.

* *Session poisoning*, where an attacker injects potentially dangerous
  data into a user's session -- usually through a Web form that the user
  submits to set session data.

  A canonical example is a site that stores a simple user preference (like
  a page's background color) in a cookie. An attacker could trick a user
  into clicking a link to submit a "color" that actually contains an
  XSS attack. If that color isn't escaped, the user could again
  inject malicious code into the user's environment.

The Solution
------------

There are a number of general principles that can protect you from these attacks:

* Never allow session information to be contained in the URL.

  Django's session framework (see `Chapter 14`_) simply doesn't allow
  sessions to be contained in the URL.

* Don't store data in cookies directly. Instead, store a session ID
  that maps to session data stored on the backend.

  If you use Django's built-in session framework (i.e.,
  ``request.session``), this is handled automatically for you. The only
  cookie that the session framework uses is a single session ID; all the
  session data is stored in the database.

* Remember to escape session data if you display it in the template. See
  the earlier XSS section, and remember that it applies to any user-created
  content as well as any data from the browser. You should treat session
  information as being user created.

* Prevent attackers from spoofing session IDs whenever possible.

  Although it's nearly impossible to detect someone who's hijacked a
  session ID, Django does have built-in protection against a brute-force
  session attack. Session IDs are stored as hashes (instead of sequential
  numbers), which prevents a brute-force attack, and a user will always get
  a new session ID if she tries a nonexistent one, which prevents session
  fixation.

Notice that none of those principles and tools prevents man-in-the-middle
attacks. These types of attacks are nearly impossible to detect. If your site
allows logged-in users to see any sort of sensitive data, you should *always*
serve that site over HTTPS. Additionally, if you have an SSL-enabled site,
you should set the ``SESSION_COOKIE_SECURE`` setting to ``True``; this will
make Django only send session cookies over HTTPS.

E-mail Header Injection
=======================

SQL injection's less well-known sibling, *e-mail header injection*, hijacks
Web forms that send e-mail. An attacker can use this technique to send spam via
your mail server. Any form that constructs e-mail headers from Web form data is
vulnerable to this kind of attack.

Let's look at the canonical contact form found on many sites. Usually this
sends a message to a hard-coded e-mail address and, hence, doesn't appear
vulnerable to spam abuse at first glance.

However, most of these forms also allow the user to type in his own subject
for the e-mail (along with a "from" address, body, and sometimes a few other
fields). This subject field is used to construct the "subject" header of the
e-mail message.

If that header is unescaped when building the e-mail message, an attacker could
submit something like ``"hello\ncc:spamvictim@example.com"`` (where ``"\n``" is
a newline character). That would make the constructed e-mail headers turn into::

    To: hardcoded@example.com
    Subject: hello
    cc: spamvictim@example.com

Like SQL injection, if we trust the subject line given by the user, we'll
allow him to construct a malicious set of headers, and he can use our
contact form to send spam.

The Solution
------------

We can prevent this attack in the same way we prevent SQL injection: always
escape or validate user-submitted content.

Django's built-in mail functions (in ``django.core.mail``) simply do not allow
newlines in any fields used to construct headers (the from and to addresses,
plus the subject). If you try to use ``django.core.mail.send_mail`` with a
subject that contains newlines, Django will raise a ``BadHeaderError``
exception.

If you do not use Django's built-in mail functions to send e-mail, you'll need
to make sure that newlines in headers either cause an error or are stripped.
You may want to examine the ``SafeMIMEText`` class in ``django.core.mail`` to
see how Django does this.

Directory Traversal
===================

*Directory traversal* is another injection-style attack, wherein a malicious
user tricks filesystem code into reading and/or writing files that the Web
server shouldn't have access to.

An example might be a view that reads files from the disk without carefully
sanitizing the file name::

    def dump_file(request):
        filename = request.GET["filename"]
        filename = os.path.join(BASE_PATH, filename)
        content = open(filename).read()

        # ...

Though it looks like that view restricts file access to files beneath
``BASE_PATH`` (by using ``os.path.join``), if the attacker passes in a
``filename`` containing ``..`` (that's two periods, a shorthand for
"the parent directory"), she can access files "above" ``BASE_PATH``. It's only
a matter of time before she can discover the correct number of dots to
successfully access, say, ``../../../../../etc/passwd``.

Anything that reads files without proper escaping is vulnerable to this
problem. Views that *write* files are just as vulnerable, but the consequences
are doubly dire.

Another permutation of this problem lies in code that dynamically loads
modules based on the URL or other request information. A well-publicized
example came from the world of Ruby on Rails.  Prior to mid-2006,
Rails used URLs like ``http://example.com/person/poke/1`` directly to
load modules and call methods. The result was that a
carefully constructed URL could automatically load arbitrary code,
including a database reset script!

The Solution
------------

If your code ever needs to read or write files based on user input, you need
to sanitize the requested path very carefully to ensure that an attacker isn't
able to escape from the base directory you're restricting access to.

.. note::

    Needless to say, you should *never* write code that can read from any
    area of the disk!

A good example of how to do this escaping lies in Django's built-in static
content-serving view (in ``django.views.static``). Here's the relevant code::

    import os
    import posixpath

    # ...

    path = posixpath.normpath(urllib.unquote(path))
    newpath = ''
    for part in path.split('/'):
        if not part:
            # strip empty path components
            continue

        drive, part = os.path.splitdrive(part)
        head, part = os.path.split(part)
        if part in (os.curdir, os.pardir):
            # strip '.' and '..' in path
            continue

        newpath = os.path.join(newpath, part).replace('\\', '/')

Django doesn't read files (unless you use the ``static.serve``
function, but that's protected with the code just shown), so this
vulnerability doesn't affect the core code much.

In addition, the use of the URLconf abstraction means that Django will *never*
load code you've not explicitly told it to load. There's no way to create a
URL that causes Django to load something not mentioned in a URLconf.

Exposed Error Messages
======================

During development, being able to see tracebacks and errors live in your
browser is extremely useful. Django has "pretty" and informative debug
messages specifically to make debugging easier.

However, if these errors get displayed once the site goes live, they can
reveal aspects of your code or configuration that could aid an attacker.

Furthermore, errors and tracebacks aren't at all useful to end users. Django's
philosophy is that site visitors should never see application-related error
messages. If your code raises an unhandled exception, a site visitor should
not see the full traceback -- or *any* hint of code snippets or Python
(programmer-oriented) error messages. Instead, the visitor should see a
friendly "This page is unavailable" message.

Naturally, of course, developers need to see tracebacks to debug problems in
their code. So the framework should hide all error messages from the public,
but it should display them to the trusted site developers.

The Solution
------------

As we covered in Chapter 12, Django's ``DEBUG`` setting controls the display of
these error messages. Make sure to set this to ``False`` when you're ready to
deploy.

Users deploying under Apache and mod_python (also see Chapter 12) should also
make sure they have ``PythonDebug Off`` in their Apache conf files; this will
suppress any errors that occur before Django has had a chance to load.

A Final Word on Security
========================

We hope all this talk of security problems isn't too intimidating. It's true
that the Web can be a wild world, but with a little bit of foresight,
you can have a secure Web site.

Keep in mind that Web security is a constantly changing field; if you're
reading the dead-tree version of this book, be sure to check more up to date
security resources for any new vulnerabilities that have been discovered. In
fact, it's always a good idea to spend some time each week or month
researching and keeping current on the state of Web application security. It's
a small investment to make, but the protection you'll get for your site and
your users is priceless.

What's Next?
============

You've reached the end of our regularly scheduled program. The following
appendixes all contain reference material that you might need as you work on
your Django projects.

We wish you the best of luck in running your Django site, whether it's a little
toy for you and a few friends, or the next Google.

.. _Chapter 14: ../chapter14/
.. _Chapter 16: ../chapter16/
