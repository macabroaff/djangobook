==============================================
Apêndice E: Template Tags embutidos e Filtros
==============================================

O capítulo 4 apresenta uma lista das mais úteis template tags e filtros. Entretanto, Django traz muito mais tags e filtros embutidos. Este apêndice fala sobre eles.

Referência de Tag incorporada
==============================

autoescape
----------

Controla o comportamento atual do auto-escape. Essa tag recebe ``on`` ou ``off`` como argumento e determina se o auto-escape terá efeito no interior do bloco.

Quando o auto-escape é ativado, todo conteúdo da variável tem um escape HTML aplicado à ela antes da saída do resultado (mas depois de quaisquer filtros terem sido aplicados). Isso é equivalente a aplicar manualmente o filtro ``escape`` para cada variável.

A única excessão são as variáveis que já estão marcadas como "seguras" do auto-escape, tanto pelo código que popula a variável, ou porque já está com os filtros ``safe`` ou ``escape`` aplicados.

block
-----

Define um bloco que pode ser sobrescrito por um template filho. Consulte o capítulo 4 para mais informações sobre herança de modelos.

comment
-------

Ignora tudo entre ``{% comment %}`` e ``{% endcomment %}``.

cycle
-----


Cycle among the given strings or variables each time this tag is encountered.

Within a loop, cycles among the given strings each time through the
loop::

    {% for o in some_list %}
        <tr class="{% cycle 'row1' 'row2' %}">
            ...
        </tr>
    {% endfor %}

You can use variables, too. For example, if you have two template variables,
``rowvalue1`` and ``rowvalue2``, you can cycle between their values like this::

    {% for o in some_list %}
        <tr class="{% cycle rowvalue1 rowvalue2 %}">
            ...
        </tr>
    {% endfor %}

Yes, you can mix variables and strings::

    {% for o in some_list %}
        <tr class="{% cycle 'row1' rowvalue2 'row3' %}">
            ...
        </tr>
    {% endfor %}

In some cases you might want to refer to the next value of a cycle from
outside of a loop. To do this, just give the ``{% cycle %}`` tag a name, using
"as", like this::

    {% cycle 'row1' 'row2' as rowcolors %}

From then on, you can insert the current value of the cycle wherever you'd like
in your template::

    <tr class="{% cycle rowcolors %}">...</tr>
    <tr class="{% cycle rowcolors %}">...</tr>

You can use any number of values in a ``{% cycle %}`` tag, separated by spaces.
Values enclosed in single (``'``) or double quotes (``"``) are treated as
string literals, while values without quotes are treated as template variables.

For backwards compatibility, the ``{% cycle %}`` tag supports the much inferior
old syntax from previous Django versions. You shouldn't use this in any new
projects, but for the sake of the people who are still using it, here's what it
looks like::

    {% cycle row1,row2,row3 %}

In this syntax, each value gets interpreted as a literal string, and there's no
way to specify variable values. Or literal commas. Or spaces. Did we mention
you shouldn't use this syntax in any new projects?

debug
-----

Output a whole load of debugging information, including the current context and
imported modules.

extends
-------

Signal that this template extends a parent template.

This tag can be used in two ways:

* ``{% extends "base.html" %}`` (with quotes) uses the literal value
  ``"base.html"`` as the name of the parent template to extend.

* ``{% extends variable %}`` uses the value of ``variable``. If the variable
  evaluates to a string, Django will use that string as the name of the
  parent template. If the variable evaluates to a ``Template`` object,
  Django will use that object as the parent template.

See Chapter 4 for more information on template inheritance.

filter
------

Filter the contents of the variable through variable filters.

Filters can also be piped through each other, and they can have arguments --
just like in variable syntax.

Sample usage::

    {% filter force_escape|lower %}
        This text will be HTML-escaped, and will appear in all lowercase.
    {% endfilter %}

firstof
-------

Outputs the first variable passed that is not False.  Outputs nothing if all the
passed variables are False.

Sample usage::

    {% firstof var1 var2 var3 %}

This is equivalent to::

    {% if var1 %}
        {{ var1 }}
    {% else %}{% if var2 %}
        {{ var2 }}
    {% else %}{% if var3 %}
        {{ var3 }}
    {% endif %}{% endif %}{% endif %}

You can also use a literal string as a fallback value in case all
passed variables are False::

    {% firstof var1 var2 var3 "fallback value" %}

for
---

Loop over each item in an array.  For example, to display a list of athletes
provided in ``athlete_list``::

    <ul>
    {% for athlete in athlete_list %}
        <li>{{ athlete.name }}</li>
    {% endfor %}
    </ul>

You can loop over a list in reverse by using ``{% for obj in list reversed %}``.

If you need to loop over a list of lists, you can unpack the values
in each sub-list into individual variables. For example, if your context
contains a list of (x,y) coordinates called ``points``, you could use the
following to output the list of points::

    {% for x, y in points %}
        There is a point at {{ x }},{{ y }}
    {% endfor %}

This can also be useful if you need to access the items in a dictionary.
For example, if your context contained a dictionary ``data``, the following
would display the keys and values of the dictionary::

    {% for key, value in data.items %}
        {{ key }}: {{ value }}
    {% endfor %}

The ``for`` loop sets a number of variables available within the loop (see
Table E-1).

.. table:: Table E-1. Variables Available Inside {% for %} Loops

    ==========================  ================================================
    Variable                    Description
    ==========================  ================================================
    ``forloop.counter``         The current iteration of the loop (1-indexed)
    ``forloop.counter0``        The current iteration of the loop (0-indexed)
    ``forloop.revcounter``      The number of iterations from the end of the
                                loop (1-indexed)
    ``forloop.revcounter0``     The number of iterations from the end of the
                                loop (0-indexed)
    ``forloop.first``           True if this is the first time through the loop
    ``forloop.last``            True if this is the last time through the loop
    ``forloop.parentloop``      For nested loops, this is the loop "above" the
                                current one
    ==========================  ================================================

The ``for`` tag can take an optional ``{% empty %}`` clause that will be
displayed if the given array is empty or could not be found::

    <ul>
    {% for athlete in athlete_list %}
        <li>{{ athlete.name }}</li>
    {% empty %}
        <li>Sorry, no athlete in this list!</li>
    {% endfor %}
    <ul>

The above is equivalent to -- but shorter, cleaner, and possibly faster
than -- the following::

    <ul>
      {% if athlete_list %}
        {% for athlete in athlete_list %}
          <li>{{ athlete.name }}</li>
        {% endfor %}
      {% else %}
        <li>Sorry, no athletes in this list.</li>
      {% endif %}
    </ul>

if
--

The ``{% if %}`` tag evaluates a variable, and if that variable is "true" (i.e.
exists, is not empty, and is not a false boolean value) the contents of the
block are output::

    {% if athlete_list %}
        Number of athletes: {{ athlete_list|length }}
    {% else %}
        No athletes.
    {% endif %}

In the above, if ``athlete_list`` is not empty, the number of athletes will be
displayed by the ``{{ athlete_list|length }}`` variable.

As you can see, the ``if`` tag can take an optional ``{% else %}`` clause that
will be displayed if the test fails.

``if`` tags may use ``and``, ``or`` or ``not`` to test a number of variables or
to negate a given variable::

    {% if athlete_list and coach_list %}
        Both athletes and coaches are available.
    {% endif %}

    {% if not athlete_list %}
        There are no athletes.
    {% endif %}

    {% if athlete_list or coach_list %}
        There are some athletes or some coaches.
    {% endif %}

    {% if not athlete_list or coach_list %}
        There are no athletes or there are some coaches (OK, so
        writing English translations of boolean logic sounds
        stupid; it's not our fault).
    {% endif %}

    {% if athlete_list and not coach_list %}
        There are some athletes and absolutely no coaches.
    {% endif %}

``if`` tags don't allow ``and`` and ``or`` clauses within the same tag, because
the order of logic would be ambiguous. For example, this is invalid::

    {% if athlete_list and coach_list or cheerleader_list %}

If you need to combine ``and`` and ``or`` to do advanced logic, just use nested
``if`` tags. Por exemplo::

    {% if athlete_list %}
        {% if coach_list or cheerleader_list %}
            We have athletes, and either coaches or cheerleaders!
        {% endif %}
    {% endif %}

Multiple uses of the same logical operator are fine, as long as you use the
same operator. For example, this is valid::

    {% if athlete_list or coach_list or parent_list or teacher_list %}

ifchanged
---------

Check if a value has changed from the last iteration of a loop.

The ``ifchanged`` tag is used within a loop. It has two possible uses.

1. Checks its own rendered contents against its previous state and only
   displays the content if it has changed. For example, this displays a list of
   days, only displaying the month if it changes::

        <h1>Archive for {{ year }}</h1>

        {% for date in days %}
            {% ifchanged %}<h3>{{ date|date:"F" }}</h3>{% endifchanged %}
            <a href="{{ date|date:"M/d"|lower }}/">{{ date|date:"j" }}</a>
        {% endfor %}

2. If given a variable, check whether that variable has changed. For
   example, the following shows the date every time it changes, but
   only shows the hour if both the hour and the date has changed::

        {% for date in days %}
            {% ifchanged date.date %} {{ date.date }} {% endifchanged %}
            {% ifchanged date.hour date.date %}
                {{ date.hour }}
            {% endifchanged %}
        {% endfor %}

The ``ifchanged`` tag can also take an optional ``{% else %}`` clause that
will be displayed if the value has not changed::

        {% for match in matches %}
            <div style="background-color:
                {% ifchanged match.ballot_id %}
                    {% cycle red,blue %}
                {% else %}
                    grey
                {% endifchanged %}
            ">{{ match }}</div>
        {% endfor %}

ifequal
-------

Output the contents of the block if the two arguments equal each other.

Example::

    {% ifequal user.id comment.user_id %}
        ...
    {% endifequal %}

As in the ``{% if %}`` tag, an ``{% else %}`` clause is optional.

The arguments can be hard-coded strings, so the following is valid::

    {% ifequal user.username "adrian" %}
        ...
    {% endifequal %}

It is only possible to compare an argument to template variables or strings.
You cannot check for equality with Python objects such as ``True`` or
``False``.  If you need to test if something is true or false, use the ``if``
tag instead.

ifnotequal
----------

Just like ``ifequal``, except it tests that the two arguments are not equal.

include
-------

Loads a template and renders it with the current context. This is a way of
"including" other templates within a template.

The template name can either be a variable or a hard-coded (quoted) string,
in either single or double quotes.

This example includes the contents of the template ``"foo/bar.html"``::

    {% include "foo/bar.html" %}

This example includes the contents of the template whose name is contained in
the variable ``template_name``::

    {% include template_name %}

An included template is rendered with the context of the template that's
including it. This example produces the output ``"Hello, John"``:

* Context: variable ``person`` is set to ``"john"``.
* Template::

    {% include "name_snippet.html" %}

* The ``name_snippet.html`` template::

    Hello, {{ person }}

See also: ``{% ssi %}``.

load
----

Load a custom template tag set. See Chapter 9 for more information on custom
template libraries.

now
---

Display the date, formatted according to the given string.

Uses the same format as PHP's ``date()`` function (http://php.net/date)
with some custom extensions.

Table E-2 shows the available format strings.

.. table:: Table E-2. Available Date Format Strings

    ================  ========================================  =====================
    Format character  Description                               Example output
    ================  ========================================  =====================
    a                 ``'a.m.'`` or ``'p.m.'`` (Note that       ``'a.m.'``
                      this is slightly different than PHP's
                      output, because this includes periods
                      to match Associated Press style.)
    A                 ``'AM'`` or ``'PM'``.                     ``'AM'``
    b                 Month, textual, 3 letters, lowercase.     ``'jan'``
    B                 Not implemented.
    d                 Day of the month, 2 digits with           ``'01'`` to ``'31'``
                      leading zeros.
    D                 Day of the week, textual, 3 letters.      ``'Fri'``
    f                 Time, in 12-hour hours and minutes,       ``'1'``, ``'1:30'``
                      with minutes left off if they're zero.
                      Proprietary extension.
    F                 Month, textual, long.                     ``'January'``
    g                 Hour, 12-hour format without leading      ``'1'`` to ``'12'``
                      zeros.
    G                 Hour, 24-hour format without leading      ``'0'`` to ``'23'``
                      zeros.
    h                 Hour, 12-hour format.                     ``'01'`` to ``'12'``
    H                 Hour, 24-hour format.                     ``'00'`` to ``'23'``
    i                 Minutes.                                  ``'00'`` to ``'59'``
    I                 Not implemented.
    j                 Day of the month without leading          ``'1'`` to ``'31'``
                      zeros.
    l                 Day of the week, textual, long.           ``'Friday'``
    L                 Boolean for whether it's a leap year.     ``True`` or ``False``
    m                 Month, 2 digits with leading zeros.       ``'01'`` to ``'12'``
    M                 Month, textual, 3 letters.                ``'Jan'``
    n                 Month without leading zeros.              ``'1'`` to ``'12'``
    N                 Month abbreviation in Associated Press    ``'Jan.'``, ``'Feb.'``, ``'March'``, ``'May'``
                      style. Proprietary extension.
    O                 Difference to Greenwich time in hours.    ``'+0200'``
    P                 Time, in 12-hour hours, minutes and       ``'1 a.m.'``, ``'1:30 p.m.'``, ``'midnight'``, ``'noon'``, ``'12:30 p.m.'``
                      'a.m.'/'p.m.', with minutes left off
                      if they're zero and the special-case
                      strings 'midnight' and 'noon' if
                      appropriate. Proprietary extension.
    r                 RFC 2822 formatted date.                  ``'Thu, 21 Dec 2000 16:01:07 +0200'``
    s                 Seconds, 2 digits with leading zeros.     ``'00'`` to ``'59'``
    S                 English ordinal suffix for day of the     ``'st'``, ``'nd'``, ``'rd'`` or ``'th'``
                      month, 2 characters.
    t                 Number of days in the given month.        ``28`` to ``31``
    T                 Time zone of this machine.                ``'EST'``, ``'MDT'``
    U                 Not implemented.
    w                 Day of the week, digits without           ``'0'`` (Sunday) to ``'6'`` (Saturday)
                      leading zeros.
    W                 ISO-8601 week number of year, with        ``1``, ``53``
                      weeks starting on Monday.
    y                 Year, 2 digits.                           ``'99'``
    Y                 Year, 4 digits.                           ``'1999'``
    z                 Day of the year.                          ``0`` to ``365``
    Z                 Time zone offset in seconds. The          ``-43200`` to ``43200``
                      offset for timezones west of UTC is
                      always negative, and for those east of
                      UTC is always positive.
    ================  ========================================  =====================

Example::

    It is {% now "jS F Y H:i" %}

Note that you can backslash-escape a format string if you want to use the
"raw" value. In this example, "f" is backslash-escaped, because otherwise
"f" is a format string that displays the time. The "o" doesn't need to be
escaped, because it's not a format character::

    It is the {% now "jS o\f F" %}

This would display as "It is the 4th of September".

regroup
-------

Regroup a list of alike objects by a common attribute.

This complex tag is best illustrated by use of an example: say that ``people``
is a list of people represented by dictionaries with ``first_name``,
``last_name``, and ``gender`` keys:

.. code-block:: python

    people = [
        {'first_name': 'George', 'last_name': 'Bush', 'gender': 'Male'},
        {'first_name': 'Bill', 'last_name': 'Clinton', 'gender': 'Male'},
        {'first_name': 'Margaret', 'last_name': 'Thatcher', 'gender': 'Female'},
        {'first_name': 'Condoleezza', 'last_name': 'Rice', 'gender': 'Female'},
        {'first_name': 'Pat', 'last_name': 'Smith', 'gender': 'Unknown'},
    ]

...and you'd like to display a hierarchical list that is ordered by gender,
like this::

    * Male:
        * George Bush
        * Bill Clinton
    * Female:
        * Margaret Thatcher
        * Condoleezza Rice
    * Unknown:
        * Pat Smith

You can use the ``{% regroup %}`` tag to group the list of people by gender.
The following snippet of template code would accomplish this::

    {% regroup people by gender as gender_list %}

    <ul>
    {% for gender in gender_list %}
        <li>{{ gender.grouper }}
        <ul>
            {% for item in gender.list %}
            <li>{{ item.first_name }} {{ item.last_name }}</li>
            {% endfor %}
        </ul>
        </li>
    {% endfor %}
    </ul>

Let's walk through this example. ``{% regroup %}`` takes three arguments: the
list you want to regroup, the attribute to group by, and the name of the
resulting list. Here, we're regrouping the ``people`` list by the ``gender``
attribute and calling the result ``gender_list``.

``{% regroup %}`` produces a list (in this case, ``gender_list``) of
**group objects**. Each group object has two attributes:

* ``grouper`` -- the item that was grouped by (e.g., the string "Male" or
  "Female").
* ``list`` -- a list of all items in this group (e.g., a list of all people
  with gender='Male').

Note that ``{% regroup %}`` does not order its input! Our example relies on
the fact that the ``people`` list was ordered by ``gender`` in the first place.
If the ``people`` list did *not* order its members by ``gender``, the regrouping
would naively display more than one group for a single gender. For example,
say the ``people`` list was set to this (note that the males are not grouped
together):

.. code-block:: python

    people = [
        {'first_name': 'Bill', 'last_name': 'Clinton', 'gender': 'Male'},
        {'first_name': 'Pat', 'last_name': 'Smith', 'gender': 'Unknown'},
        {'first_name': 'Margaret', 'last_name': 'Thatcher', 'gender': 'Female'},
        {'first_name': 'George', 'last_name': 'Bush', 'gender': 'Male'},
        {'first_name': 'Condoleezza', 'last_name': 'Rice', 'gender': 'Female'},
    ]

With this input for ``people``, the example ``{% regroup %}`` template code
above would result in the following output::

    * Male:
        * Bill Clinton
    * Unknown:
        * Pat Smith
    * Female:
        * Margaret Thatcher
    * Male:
        * George Bush
    * Female:
        * Condoleezza Rice

The easiest solution to this gotcha is to make sure in your view code that the
data is ordered according to how you want to display it.

Another solution is to sort the data in the template using the ``dictsort``
filter, if your data is in a list of dictionaries::

    {% regroup people|dictsort:"gender" by gender as gender_list %}

spaceless
---------

Remove o espaço em branco entre as tags HTML. Isso inclui tabs e quebra de linhas.

Exemplo de uso::

    {% spaceless %}
        <p>
            <a href="foo/">Foo</a>
        </p>
    {% endspaceless %}

Esse exemplo retorna o HTML seguinte::

    <p><a href="foo/">Foo</a></p>

Só os espaços entre *tags* será removido -- não se aplica aos espaços entre tags e o texto. Neste exemplo, o espaço em torno do ``Hello`` não será retirado::

    {% spaceless %}
        <strong>
            Hello
        </strong>
    {% endspaceless %}

ssi
---

Output the contents of a given file into the page.

Like a simple "include" tag, ``{% ssi %}`` includes the contents of another
file -- which must be specified using an absolute path -- in the current
page::

    {% ssi /home/html/ljworld.com/includes/right_generic.html %}

If the optional "parsed" parameter is given, the contents of the included
file are evaluated as template code, within the current context::

    {% ssi /home/html/ljworld.com/includes/right_generic.html parsed %}

Note that if you use ``{% ssi %}``, you'll need to define
``ALLOWED_INCLUDE_ROOTS`` in your Django settings, as a security measure.

See also: ``{% include %}``.

templatetag
-----------

Output one of the syntax characters used to compose template tags.

Since the template system has no concept of "escaping", to display one of the
bits used in template tags, you must use the ``{% templatetag %}`` tag.

See Table E-3 for the available arguments.

.. table:: Table E-3. Available Arguments for templatetag Filter

    ==================  =======
    Argument            Outputs
    ==================  =======
    ``openblock``       ``{%``
    ``closeblock``      ``%}``
    ``openvariable``    ``{{``
    ``closevariable``   ``}}``
    ``openbrace``       ``{``
    ``closebrace``      ``}``
    ``opencomment``     ``{#``
    ``closecomment``    ``#}``
    ==================  =======

url
---

Returns an absolute URL (i.e., a URL without the domain name) matching a given
view function and optional parameters. This is a way to output links without
violating the DRY principle by having to hard-code URLs in your templates::

    {% url path.to.some_view arg1,arg2,name1=value1 %}

The first argument is a path to a view function in the format
``package.package.module.function``. Additional arguments are optional and
should be comma-separated values that will be used as positional and keyword
arguments in the URL. All arguments required by the URLconf should be present.

For example, suppose you have a view, ``app_views.client``, whose URLconf
takes a client ID (here, ``client()`` is a method inside the views file
``app_views.py``). The URLconf line might look like this::

    ('^client/(\d+)/$', 'app_views.client')

If this app's URLconf is included into the project's URLconf under a path
such as this::

    ('^clients/', include('project_name.app_name.urls'))

...then, in a template, you can create a link to this view like this::

    {% url app_views.client client.id %}

The template tag will output the string ``/clients/client/123/``.

widthratio
----------

For creating bar charts and such, this tag calculates the ratio of a given value
to a maximum value, and then applies that ratio to a constant.

Por exemplo::

    <img src="bar.gif" height="10" width="{% widthratio this_value max_value 100 %}" />

Above, if ``this_value`` is 175 and ``max_value`` is 200, the image in the
above example will be 88 pixels wide (because 175/200 = .875; .875 * 100 = 87.5
which is rounded up to 88).

with
----

Caches a complex variable under a simpler name. This is useful when accessing
an "expensive" method (e.g., one that hits the database) multiple times.

Por exemplo::

    {% with business.employees.count as total %}
        {{ total }} employee{{ total|pluralize }}
    {% endwith %}

The populated variable (in the example above, ``total``) is only available
between the ``{% with %}`` and ``{% endwith %}`` tags.

Built-in Filter Reference
=========================

add
---

Adiciona o argumento ao valor.

Por exemplo::

    {{ valor|add:"2" }}

Se ``valor`` for ``4``, então a saída será ``6``.

addslashes
----------

Adiciona barras antes das aspas. Útil como caractere de escape em CSV, por exemplo.

capfirst
--------

Coloca em maiúsculo o primeiro caractere do valor.

center
------

Centraliza o valor num campo de acordo com a largura.

cut
---

Remove todos os valores de uma string de acordo com seu argumento.

Por exemplo::

    {{ valor|cut:" "}}

Se ``valor`` for ``"String com espaços"``, a saída será ``"Stringcomespaços"``.

date
----

Formata a data de acordo com o formato dado (o mesmo da tag ``{% now %}``).

Por exemplo::

    {{ valor|date:"D d M Y" }}

Se ``valor`` for um objeto ``datetime`` (ex., o resultado de ``datetime.datetime.now()``), a saída será uma string ``"Qui 25 Jul 2013"``.

Quando usado sem o formato::

    {{ valor|date }}

...a sequência de formatação definida em ``DATE_FORMAT`` será utilizada.

default
-------

Se o valor for avaliado como ``False``, use o padrão determinado. Caso contrário, use o valor.

Por exemplo::

    {{ valor|default:"nada" }}

Se ``valor`` for ``""`` (uma string vazia), a saída será ``nada``.

default_if_none
---------------

Se (e somente se) o valor for ``None``, use o padrão determinado. Caso contrário, use o valor.

Note que se for dada uma string vazia, o valor padrão *não* será usado.
Use o filtro ``default`` se você quiser uma alternativa para strings vazias.

Por exemplo::

    {{ valor|default_if_none:"nada" }}

Se ``valor`` for ``None``, a saída será a string ``"nada"``

dictsort
--------

Recebe uma lista de dicionários e retorna uma lista ordenada pela chave dada no argumento.

Por exemplo::

    {{ valor|dictsort:"nome" }}

Se ``valor`` for::

    [
        {'nome': 'zed', 'idade': 19},
        {'nome': 'amy', 'idade': 22},
        {'nome': 'joe', 'idade': 31},
    ]

então a saída será::

    [
        {'nome': 'amy', 'idade': 22},
        {'nome': 'joe', 'idade': 31},
        {'nome': 'zed', 'idade': 19},
    ]

dictsortreversed
----------------

Recebe uma lista de dicionários e retorna uma lista ordenada de modo reverso pela chave dada no argumento. Esse filtro funciona exatamente como o filtro acima, porém, o valor retornado terá uma ordem inversa.

divisibleby
-----------

Retorna ``True`` se o valor for divisível pelo argumento.

Por exemplo::

    {{ valor|divisibleby:"3" }}

Se ``valor`` for ``21``, a saída será ``True``.

escape
------

Escapa uma string HTML. Especificamente, ele faz estas substituições:

* ``<`` é convertido para ``&lt;``
* ``>`` é convertido para ``&gt;``
* ``'`` (aspas simples) é convertido para ``&#39;``
* ``"`` (aspas duplas) é convertido para ``&quot;``
* ``&`` é convertido para ``&amp;``

O escape somente é aplicado quando a string é a saída, então não importa a sequência encadeada de filtros que você colocou no ``escape``: ele sempre será aplicado como se fosse o último filtro. Se você quiser que o escape seja usado imediatamente, use o filtro ``force_escape``.

Aplicando o ``escape`` em uma variável que normalmente já contém o auto-escape, o resultado vai produzir um escape sendo feito. Por isso é seguro usar esta função em ambientes com auto-escape. Se você quiser múltiplos escapes para serem aplicados, use o filtro ``force_escape``.

escapejs
--------

Caracteres de escape para em strings JavaScript. Este *não* o torna a string segura para o uso em HTML, mas proteje você de um erro de sintaxe quando usado em templates para gerar JavaScript/JSON.

filesizeformat
--------------

Formata de forma legível o valor do tamanho de um arquivo (ex., ``"13 KB"``, ``"4.1 MB"``, ``"102 bytes"``, etc)

Por exemplo::

    {{ valor|filesizeformat }}

Se ``valor`` for 123456789, a saída será ``117.7 MB``.

first
-----

Retorna o primeiro item de uma lista.

Por exemplo::

    {{ valor|first }}

Se ``valor`` for a lista ``['a', 'b', 'c']``, a saída será ``'a'``.

fix_ampersands
--------------

Substitue o E comercial (&) por ``&amp;``.

Por exemplo::

    {{ valor|fix_ampersands }}

Se ``valor`` for ``Tom & Jerry``, a saída será ``Tom &amp; Jerry``.

floatformat
-----------

Quando usado sem um argumento, arredonda um número ponto flutuante para uma casa decimal -- mas somente se houver uma parte decimal para ser exibida. Por exemplo:

============  ===========================  ========
``valor``     Template                     Saída
============  ===========================  ========
``34.23234``  ``{{ valor|floatformat }}``  ``34.2``
``34.00000``  ``{{ valor|floatformat }}``  ``34``
``34.26000``  ``{{ valor|floatformat }}``  ``34.3``
============  ===========================  ========

Se usado um número inteiro como argumento, ``floatformat`` arredonda o número de acordo com o argumento passado. Por exemplo:

============  =============================  ==========
``valor``     Template                       Saída
============  =============================  ==========
``34.23234``  ``{{ valor|floatformat:3 }}``  ``34.232``
``34.00000``  ``{{ valor|floatformat:3 }}``  ``34.000``
``34.26000``  ``{{ valor|floatformat:3 }}``  ``34.260``
============  =============================  ==========

Se o argumento passado em ``floatformat`` for negativo, ele irá arredondar o número de acordo com o argumento passado -- mas somente se houver uma parte decimal para ser exibida. Por exemplo:

============  ================================  ==========
``valor``     Template                          Saída
============  ================================  ==========
``34.23234``  ``{{ valor|floatformat:"-3" }}``  ``34.232``
``34.00000``  ``{{ valor|floatformat:"-3" }}``  ``34``
``34.26000``  ``{{ valor|floatformat:"-3" }}``  ``34.260``
============  ================================  ==========

Usando ``floatformat`` sem argumento é o mesmo que usar ``floatformat`` com argumento ``-1``

force_escape
------------

Aplica um escape HTML em uma string (veja o filtro ``escape`` para mais detalhes).
Esse filtro é aplicado *imediatamente* e retorna uma nova string "escapada". Isto é útil em casos raros onde você precisa de múltiplos escapes ou precisa aplicar outros filtros para resultados "escapados". Normalmente, se usa o filtro ``escape``.

get_digit
---------

Dado um número, retorna o dígito requisitado, onde 1 é o dígito mais à direita, 2 é o dígito mais à esquerda, e etc. Retorna o valor original de uma entrada inválida (se a entrada ou argumento não for um inteiro, ou se o argumento for menor que 1). Caso contrário, a saída sempre será um inteiro.

Por exemplo::

    {{ valor|get_digit:"2" }}

Se ``valor`` for ``123456789``, the output will be ``8``.

iriencode
---------

Converte um IRI (Internationalized Resource Identifier, ou Identificador de recursos internacionalizados) para uma string adequada para incluir em uma URL. Isto é necessário se você está tentando usar strings contendo caracteres não-ASCII em uma URL

É seguro usar esse filtro em uma string que já foi passada pelo filtro ``urlencode``.

join
----

Une uma lista com uma string, como a função em Python ``str.join(list)``

Por exemplo::

    {{ valor|join:" // " }}

Se ``valor`` for a lista ``['a', 'b', 'c']``, a saída será a string ``"a // b // c"``.

last
----

Retorna o último item de uma lista.

Por exemplo::

    {{ valor|last }}

Se ``valor`` for a lista ``['a', 'b', 'c', 'd']``, a saída será a string ``"d"``.

length
------

Retorna o tamanho da variável. Isto funciona tanto para strings e listas.

Por exemplo::

    {{ valor|length }}

Se ``valor`` for ``['a', 'b', 'c', 'd']``, a saída será ``4``.

length_is
---------

Retorna ``True`` se o valor do argumento for igual ao tamanho, ou ``False`` caso contrário

Por exemplo::

    {{ valor|length_is:"4" }}

Se ``valor`` for ``['a', 'b', 'c', 'd']``, a saída será ``True``.

linebreaks
----------

Substitue a quebra de linha em um texto puro com o HTML apropriado; uma nova linha torna-se uma quebra de linha no padrão HTML (``<br />``) e uma nova linha seguida por uma linha em branco torna-se uma quebra de parágrafo (``</p>``).

Por exemplo::

    {{ valor|linebreaks }}

Se ``valor`` for ``João\né um rapaz``, a saída será ``<p>João<br />é um rapaz</p>``.

linebreaksbr
~~~~~~~~~~~~

Converte todas as quebras de linhas em uma parte de texto em uma quebra de linha HTML (``<br />``).

linenumbers
-----------

Exibe o texto com números de linha.

ljust
-----

Alinha à esquerda o valor em um campo de uma dada largura.

**Argumento:** tamanho do campo

lower
-----

Converte toda string para minúsculo.

Por exemplo::

    {{ valor|lower }}

Se ``valor`` for ``Continuo COM RAIVA da Yoko``, a saída será ``continuo com raiva da yoko``.

make_list
---------

Retorna o valor transformado em uma lista. Para um inteiro, uma lista de dígitos. Para uma string, uma lista de caracteres.

Por exemplo::

    {{ valor|make_list }}

Se ``valor`` for a string ``"João"``, ocasionará a lista ``[u'J', u'o', u'ã', u'o']``. Se ``valor`` for ``123``, a saída será a lista ``[1, 2, 3]``.

phone2numeric
-------------

Converte um número de telefone (possivelmente contento letras) para o seu equivalente numérico. Por exemplo, ``"0800-COLLECT"`` será convertido para ``"0800-2655328"``.

A entrada não tem que ser um telefone válido. Essa função converte qualquer string.

pluralize (revisar)
---------

Retorna um sufixo plural se o valor não for 1. Por padrão, esse sufixo é ``"s"``.

Exemplo::

    Você tem {{ num_mensagens }} mensagen{{ num_messagens|pluralize }}.

Para palavras que exigem um outro sufixo que ``"s"``, você pode providenciar um sufixo alternativo como um parâmetro para o filtro.

Exemplo::

    Você tem {{ num_morsas }} mors{{ num_morsas|pluralize:"as" }}.

Para palavras que não são pluralizadas por sufixos simples, você pode especificar o seu sufixos, singular e plural, separados por uma vírgula.

Exemplo::
    
    Você tem {{ num_grampeadores }} grampeado{{ num_grampeadores|pluralize:"r, res" }}.

pprint (revisar)
------

Um invólucro em torno da biblioteca padrão em Python da função ``pprint.pprint`` -- para debug, apenas.

random
------

Retorna um item aleatório de uma dada lista.

Por exemplo::

    {{ valor|random }}

Se ``valor`` for a lista ``['a', 'b', 'c', 'd']``, a saída poderá ser ``"b"``.

removetags
----------

Remove os espaços de separação de uma lista de tags [X]HTML.

Por exemplo::

    {{ valor|removetags:"b span"|safe }}

Se ``valor`` for ``"<b>João</b> <button>é</button> um <span>rapaz</span>"`` a saída será ``"João <button>é</button> um rapaz"``.

rjust
-----

Alinha à direita o valor do campo de uma dada largura.

**Argumento:** tamanho do campo

safe
----

Marca uma string como não necessitando de um escape HTML, antes de sua saída. Quando o auto-escape não está ativado, esse filtro não tem efeito algum.

safeseq
-------

Aplica o filtro ``safe`` para cada elemento de uma sequência. Útil em conjunto com outros filtros que operam em sequência, como ``join``.

 Por exemplo::

    {{ lista|safeseq|join:", " }}

Você não pode usar o filtro ``safe`` diretamente nesse caso, uma vez que primeiro converteria a variável em uma string, ao invés de trabalhar com os elementos individuais da sequência.

slice (revisar)
-----

Retorna uma parte de uma lista

Usa a mesma sintaxe Python que utiliza um mecanismo para criar "fatias" de uma lista.

Exemplo::

    {{ lista|slice:":2" }}

slugify
-------

Converte para minúsculo, remove os caracteres que não são palavras (apenas alfanuméricos e underlines são mantidos) e transforma espaços em hífens. Também remove os espaços em branco da direita e da esquerda.

Por exemplo::

    {{ valor|slugify }}

Se ``valor`` for ``"João é um rapaz"``, a saída será ``"joao-e-um-rapaz"``.

stringformat
------------

Formata a variável de acordo com o argumento, um formatador de string específico.
Este formatador utiliza a sintaxe Python de formatação de strings, com excessão do "%" que foi descartado.


Por exemplo::

    {{ valor|stringformat:"s" }}

Se ``valor`` for ``"João é um rapaz"``, a saída será ``"João é um rapaz"``.

striptags
---------

Remove todas as tags [X]HTML.

Por exemplo::

    {{ valor|striptags }}

Se ``valor`` for ``"<b>João</b> <button>é</button> um <span>rapaz</span>"``, a saída será ``"João é um rapaz"``.

time
----

Define o tempo de acordo com um formato dado (o mesmo que a tag `now`_).
O filtro ``time`` só aceitará parâmetros com o formato string que relacionem com a hora do dia, e não a data (por razões óbvias). Se você quiser um formato de data, utilize o filtro ``data``_.

Por exemplo::

    {{ valor|time:"H:i" }}

Se ``valor`` for equivalente a ``datetime.datetime.now()``, a saída será a string ``"01:23"``.

Quando usado sem uma formatação string::

    {{ valor|time }}

...será usado o formato definido em ``TIME_FORMAT``.

timesince (revisar)
---------

Formata a data como tempo desde essa date (ex., "4 dias, 6 horas").

Recebe um argumento opcional que é uma variável contendo a data a ser usada como ponto de comparação (sem argumento, o ponto de comparação é *now*). Por exemplo, se ``blog_data`` é uma instância de uma data representando meia-noite de 25 de Julho de 2013, e ``comentario_data`` for uma instância de 08:00 de 25 de Julho de 2013, então ``{{ blog_data|timesince:comentario_data }}`` retornaria "8 horas".

Comparing offset-naive and offset-aware datetimes will return an empty string.

Minutos são as menores unidades usadas, "0 minutos" serão tranformados em qualquer data que seja no futuro em relação ao ponto de comparação.

timeuntil (revisar)
---------

Similar ao ``timesince``, exceto pelo fato de que ele mete o tempo atual até uma dada data ou hora. Por exemplo, se hoje é 1 de Junho de 2013 e ``data_conferencia`` é uma instância de data marcando 26 de Junho de 2013, então ``{{ data_conferencia|timeuntil }}`` irá retornar "4 semanas".

Recebe um argumento opcional que é uma variável contendo uma data usada como ponto de comparação (em vez de *now*). Se ``desde_a_data`` for 22 de Junho de 2006, então ``{{ data_conferencia|timeuntil:desde_a_data }}`` irá retornar "1 semana".

Comparing offset-naive and offset-aware datetimes will return an empty string.

Minutos são as menores unidades usadas, "0 minutos" serão tranformados em qualquer data que está no passado em relação ao ponto de comparação.


title
-----

Converte a string em maiúscula (no formato de "Título")

truncatewords
-------------

Trunca uma string depois de um certo número de palavras.

**Argumento:** Número de palavras para serem truncadas 

Por exemplo::

    {{ valor|truncatewords:2 }}

Se ``valor`` for ``"João é um rapaz"``, a saída será ``"João é ..."``.

truncatewords_html
------------------

Similar ao ``truncatewords``, exceto pelo fato de ser usada em tags HTML. Qualquer tag que for aberta na string e não fechado antes do ponto de truncagem, serão fechadas imediatamente após o truncamento.

Essa função é menos eficiente que ``truncatewords``, então deve ser usada somente quando for passado um texto HTML.

unordered_list
--------------

Recursively takes a self-nested list and returns an HTML unordered list --
WITHOUT opening and closing <ul> tags.

The list is assumed to be in the proper format. For example, if ``var`` contains
``['States', ['Kansas', ['Lawrence', 'Topeka'], 'Illinois']]``, then
``{{ var|unordered_list }}`` would return::

    <li>States
    <ul>
            <li>Kansas
            <ul>
                    <li>Lawrence</li>
                    <li>Topeka</li>
            </ul>
            </li>
            <li>Illinois</li>
    </ul>
    </li>

upper
-----

Converte toda uma string em maiúscula.

Por exemplo::

    {{ valor|upper }}

Se ``valor`` for ``"João é um rapaz"``, a saída será ``"JOÃO É UM RAPAZ"``.

urlencode
---------

Escapa um valor para ser usado na URL.

urlize
------

Converte as URLs do texto em links clicáveis.

Note que se a função ``urlize`` é aplicada em um texto que já contém a marcação HTML, o filtro não funcionará como esperado. Aplique este filtro somente nos textos *simples*

Por exemplo::

    {{ valor|urlize }}

Se ``valor`` for ``"Saiba mais em www.djangoproject.com"``, a saída será ``"Saiba mais em <a href="http://www.djangoproject.com">www.djangoproject.com</a>"``.

urlizetrunc
-----------

Converts URLs into clickable links, truncating URLs longer than the given
character limit.

As with urlize_, this filter should only be applied to *plain* text.

**Argument:** Length to truncate URLs to

Por exemplo::

    {{ valor|urlizetrunc:15 }}

Se ``valor`` is ``"Check out www.djangoproject.com"``, the output would be
``'Check out <a
href="http://www.djangoproject.com">www.djangopr...</a>'``.

wordcount
---------

Retorna o número de palavras.

wordwrap
--------

Quebra palavras com um comprimento específico.

**Argumento:** número de caracteres no qual o texto será quebrado

Por exemplo::

    {{ valor|wordwrap:5 }}

Se ``valor`` for ``João é um rapaz``, a saída será::

    João
    é um
    rapaz

yesno
-----

Dado um mapeamento de strings com valores ``True``, ``False``, e (opcionalmente) ``None``, retorna uma destas sequências de acordo com o valor (veja Tabela F-4).

.. table:: Tabela E-4. Exemplos de filtro yesno

    ==========  ======================  ==================================
    Valor       Argumento               Saída
    ==========  ======================  ==================================
    ``True``    ``"yeah,no,maybe"``     ``yeah``
    ``False``   ``"yeah,no,maybe"``     ``no``
    ``None``    ``"yeah,no,maybe"``     ``maybe``
    ``None``    ``"yeah,no"``           ``"no"`` (converts None to False
                                        if no mapping for None is given)
    ==========  ======================  ==================================
