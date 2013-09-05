==============================================================
Capítulo 18: Integrando Banco de Dados Legados e Aplicações
==============================================================

Django é mais adequado para o chamado desenvolvimento "green-field" - isto é, começando
projetos a partir do zero, como se você estivesse construindo um edifício em um campo fresco
da grama verde. Mas, apesar do fato do Django favorecer projetos a partir do zero,
é possível integrar o framework à bancos de dados legados e
aplicações. Este capítulo explica a algumas estratégias de integração.


Integrando com banco de dados Legados
==================================

A camada de banco de dados do Django gera esquemas SQL a partir do código python -- mas com 
banco de dados legado, já exite os esquemas SQL. Nestes casos, você precisa criar modelos 
para as tabelas existes no banco de dados. Para este casos, Django vem com ferramentas que podem gerar o código 
do modelo lendo o layout das tabelas do banco de dados. Esta ferramenta é a "inspectdb", e pode ser utilizada executando
o comando "manage.py inspectdb".


Usando ``inspectdb``
-------------------

O utilitário "inspectdb"  examinará o banco de dados apontado pelo arquivo de configurações
,que determina uma representação do modelo de Django para cada uma das tabelas, e
imprime o modelo de código Python para a saída padrão.

Uma passagem através de um típico processo de integração com o de banco de dados legado.
As hipóteses são de que apenas o Django está instalado e que você tem um
banco de dados legado.

1. Criar um projeto Django executando ``django-admin.py startproject mysite`` (onde ``mysite`` é o nome do
    seu projeto). Utilizaremos ``mysite`` como o nome do projeto neste exemplo.


2. Editar o arquivo de configuração do projeto, ``mysite/settings.py``,
   para dizer ao Django quais os parâmetros de conexão e o nome do banco de dados.
   Especificamente, fornecer as seguintes configurações 
   ``DATABASE_NAME``, ``DATABASE_ENGINE``, ``DATABASE_USER``,
   ``DATABASE_PASSWORD``, ``DATABASE_HOST``, and ``DATABASE_PORT``.(Note que algumas configurações são opcionais. Leia o capítulo 5 para maiores informações)
   


3. Criar uma aplicação Django no seu projeto executando ``python mysite/manage.py startapp myapp``
   (onde ``myapp`` é o nome da aplicaçõa). Utilizaremos ``myapp`` como o nome da aplicação neste exemplo.
   

4. Executar o comando ``python mysite/manage.py inspectdb``. Ele examinará 
   as tabelas do banco de dados ``DATABASE_NAME`` e e imprimir a classe modelo gerado para cada tabela.
   Dê uma olhada na saída para ter uma idéia do que `` inspectdb `` pode fazer.
   

5. Salvar a saída no arquivo ``models.py`` da sua aplicação utilizando o redirecionamento de 
saída padrão do shell ::

       python mysite/manage.py inspectdb > mysite/myapp/models.py
       

6. Editar o arquivo ``mysite/myapp/models.py`` para limpar os modelos gerados e fazer
   as customizações necessárias. Daremos algumas dicas sobre isso na próxima seção.
   

Limpando os Modelos gerados
----------------------------

Como você deve esperar, a instrospecção no banoc de dados não é perfeita, e é necessário fazer alguma limpeza no código 
resultante do modelo. Aqui estão algumas dicas para lidar com os modelos gerados:

1. Cada tabela do bando de dados é convertida em uma classe do modelo. (isto é, existe um mapeamento um-para-um entre
   as tabelas do banco de dados e a classe do modelo. Isto quer dizer que será necessário refazer
   os modelos muitos-para-muitos das tabelas para objetos ``ManyToManyField`` 
   

2. Cada modelo gerado tem um atributo para cada campo, incluindo um campo de chave primária
   ``id``. No entanto, lembre-se que o Django adiciona automaticamente campo de chave primária
   ``id``  se um modelo não tiver uma chave primária.
    Assim, você vai querer remover todas as linhas que se parecem com isso ::
   
       id = models.IntegerField(primary_key=True)

   Não apenas estas linhas redundantes, mas também podem causar problemas se sua 
   aplicação adicionar *novos* registros a estas tabelas.
   

3. Cada tipo (por exemplo, ``CharField`` , ``DateField`` ) é determinado 
   analisando o tipo da coluna no banco de dados (por exemplo, ``VARCHAR`` , ``DATE`` ). Se o
   ``inspectdb`` não puder mapear o tipo da coluna para um tipo de campo no modelo, então será usado
   ``TextField`` e será inserido um comentário Python ``O tipo deste campo é uma sugestão.`` ,próximo ao campo do modelo gerado.
   . Fique atento a isso e mude o campo de acordo com suas necessidades.

   Se o campo no seu banco de dados não tiver nenhum equivalente no Django
   você pode deixa-lo desabilitado. Não é obrigatório incluir todos os campos da sua tabela
   na camada do modelo do Django.

4. Se o nome da coluna no banco de dados é uma palavra reservada em Python (como ``pass``,
   ``class``, ou ``for``), o ``inspectdb`` irá inserir ``'_field'`` ao nome do atributo e irá colocar
   ``db_coluna`` no nome do campo (por exemplo, ``pass``, ``class``, or ``for``).

   Por exemplo, se a tabela tem uma coluna chamada do tipo ``INT`` chamada ``for``,o modelo gerado terá um
   campo desta maneira::

       for_field = models.IntegerField(db_column='for')

   ``inspectdb`` irá inserir um comentário em Python
   ``'Campo renomeado pois essa é uma palavra reservada em Python.'`` próximo ao campo.

5. Se o banco de dados tiver tabelas que referenciam outras tabelas (como a maioria
   dos banco de dados fazem), talvez seja necessário reorganizar a ordem dos modelos gerados,
   de modo que os modelos que remetem a outros modelos sejam ordenados corretamente.
   Por exemplo, se o modelo ``Book`` tem uma ``ForeignKey`` para o modelo ``Author``, o
   modelo ``Author`` deve ser definido antes do modelo ``Book``. Se você precisa 
   criar um relacionamento em um modelo que ainda não foi definido, você pode usar um texto contendo 
   o nome do modelo ao invés do objeto modelo propriamente dito.

6. ``inspectdb`` detecta as chaves primárias do PostgreSQL, MySQL, and SQLite.
   Ou seja, ele insere ``primary_key=True`` onde é apropriado. Para os outros banco de dados
   ,você precisa inserir ``primary_key=True`` para pelo menos um campo em cada modelo
   , porque no modelo do Django são obrigatório ter campos ``primary_key=True``.

7. Detecção de chave estrangeira "Foreign-key" só funciona com o PostgreSQL e com certos tipos de tabelas do MySQL.
   Em outros casos, a chave estrangeira será gerada como ``IntegerField``s, assumindo a coluna da chave estrangeira como uma coluna ``INT``.

Integrando com Sistema de Auteticação
=========================================

É possível integrar o Django com um sistema de autenticação existente --
outros reposiórios de usuarios e senhas ou métodos de autenticação.

Por exemplo, sua compania pode ter uma instalação LDAP que armazena o usuário
e a senha para cada empregado. Isto pode ser um aborrecimento tanto para o administrador de 
rede e para os usuários, se os mesmes tiverem contas separadas no LDAP
e na aplicação do Django.

Para lidar com situações como esta, o sistema de autenticação do Django permite que você 
se conecte a outros repositórios de autenticação. Você pode substituir o banco de dados de autenticação
padrão do Django, ou você pode usar o sistema padrão em conjunto com outros sistemas.

Especificando backends de autenticação
----------------------------------

Por trás dos panos, o Django mantém uma lista de "backends de autenticação" que checa
a autenticação. Quando alguém chama
``django.contrib.auth.authenticate()`` (como descrito no capítulo 14), o Django
tenta se autenticar através de todas as backends de autenticação. Se o primeiro 
método de autenticação falhar, o Django tenta o segundo, e assim por diante, até todos os 
backends terem sidos testados.

A lista de backends de autenticação a ser usada é especificada nas configuração 
``AUTHENTICATION_BACKENDS``. Esta deve ser uma tupla de Python path,
nomes que apontam para classes Python que sabem como autenticar. Essas classes
pode estar em qualquer lugar em seu Python path.

Por padrão , ``AUTHENTICATION_BACKENDS`` é está definido desta maneira:

    ('django.contrib.auth.backends.ModelBackend',)

Esse é o esquema de autenticação básica que verifica o banco de dados de usuários do Django..

A ordem do ``AUTHENTICATION_BACKENDS`` importa , portanto se o mesmo usuário e
senha é válido em diversos backends, Django irá parar de processar no primeiro caso onde coincidir 
o usuário e senha.

Escrevendo um Backend de Autenticação
---------------------------------

Um backend de autenticação é uma classe que implementa dois métodos:
``get_user(id)`` e ``authenticate(**credentials)``.

O método``get_user`` pega um ``id`` -- que pode ser um usuário, banco de dados
ID, ou o que quer que seja -- e retorna um objeto ``User``.

O método ``authenticate`` pega credenciais como argumentos. Na maioria das vezes
será assim::

    class MyBackend(object):
        def authenticate(self, username=None, password=None):
            # Checa o usuário/senha e retorna um User.

Mas também pode autenticar um token, tipo::

    class MyBackend(object):
        def authenticate(self, token=None):
            # Checa o token e retorna um User.

De qualquer maneira, ``authenticate`` deve verificar as credenciais que receber, e que
deve retornar um object ``User`` que coincide com essas credenciais, se as
credenciais são válidas. Se elas não forem válidas , devem retornar ``None``.

O sistema de administração do Django está intimamente ligado ao próprio banco de dados do Django-backend
o obejeto ``User`` descrito no capítulo 14. A melhor maneira de lidar com isso é 
criar um objeto Django ``User`` para cada usuário que existe no backend
(por exemplo, em seu diretório LDAP, seu banco de dados SQL externo, etc). Ou você pode
escrever um script para fazer isso com antecedência ou o seu método ``authenticate`` pode fazer isso
a primeira vez que o susário logar no sistema.

Aqui está um exemplo do backend que autentica uma variável de um nome de usuário e senha
definida no arquivo ``settings.py`` e cria um objeto ``User`` do Django
na primeira vez que um usuário se autentica::

    from django.conf import settings
    from django.contrib.auth.models import User, check_password

    class SettingsBackend(object):
        """
        Autenticar nas configurações admin_login e admin_password.

         Usar o nome de login, e um hash da senha. por exemplo:

        ADMIN_LOGIN = 'admin'
        ADMIN_PASSWORD = 'sha1$4e987$afbcf42e21bd417fb71db8c66b321e9fc33051de'
        """
        def authenticate(self, username=None, password=None):
            login_valid = (settings.ADMIN_LOGIN == username)
            pwd_valid = check_password(password, settings.ADMIN_PASSWORD)
            if login_valid and pwd_valid:
                try:
                    user = User.objects.get(username=username)
                except User.DoesNotExist:
                    # Create a new user. Note that we can set password
                    # to anything, because it won't be checked; the password
                    # from settings.py will.
                    user = User(username=username, password='get from settings.py')
                    user.is_staff = True
                    user.is_superuser = True
                    user.save()
                return user
            return None

        def get_user(self, user_id):
            try:
                return User.objects.get(pk=user_id)
            except User.DoesNotExist:
                return None

Para saber mais sobre backends de autenticação, consulte a documentação oficial do Django..

Integrando com Aplicações Web Legadas
========================================

É possivel rodar uma aplicação do Django no mesmo servidor Web com uma
aplicação servida por outra tecnologia. A maneira mais simples de
fazer isso é usar o arquivo de configuração do Apache, ``httpd.conf``, para delegar 
diferentes padrões de URL para diferentes tecnologias. (Note-se que o Capítulo 12 cobre
a implantação do Django no Apache / mod_python, então pode valer a pena a leitura desse
primeiro capítulo antes de tentar essa integração.)

O principal é que o Django irá ser ativado por um padrão de URL em partiular somente se
o seu `` httpd.conf `` disser isso. A implantação padrão explicado no Capítulo
12 assume que você deseja que o Django sirva cada página em um determinado domínio ::

    <Location "/">
        SetHandler python-program
        PythonHandler django.core.handlers.modpython
        SetEnv DJANGO_SETTINGS_MODULE mysite.settings
        PythonDebug On
    </Location>

Aqui , a linha  ``<Location "/">`` quer dizer "lidar com cada URL, a partir da
raiz ", com o Django.

É perfeitamente possível limitar essa diretiva ``<Location>`` para uma certa 
árvore de diretórios. Por exemplo, digamos que você tem uma aplicação legada em PHP que serve 
a maioria das páginas em um domínio e você quer instalar o site administrativo do Djangoem em 
``/admin/`` sem quebrar o codigo do PHP. Para fazer isso , apenas configurea diretiva
``<Location>`` para ``/admin/``::

    <Location "/admin/">
        SetHandler python-program
        PythonHandler django.core.handlers.modpython
        SetEnv DJANGO_SETTINGS_MODULE mysite.settings
        PythonDebug On
    </Location>

Com isso no lugar, apenas as URLs que começam com `` admin / / `` serão ativadas no
Django. Qualquer outra página irá utilizar qualquer infra-estrutura já existente.


Note que anexar uma URL válida (como ``/admin/`` nesta seção de exemplo)
não afeta a análise da URL do Django. Django trabalha com as 
URL absoluta (por exemplo., ``/admin/people/person/add/``), e não uma versão "enxuta" da
URL (por exemplo, ``/people/person/add/``). Isso quer dizer que a raiz do URLconf
deve incluir o principal ``/admin/``.

Qual é o próximo?
============

Se você é nativo em Inglês, você pode não ter notado um dos
características mais legais do site de administração do Django: Está disponível em mais de 50
línguas diferentes! Isso é possível graças a internacionalização do Django 
framework (e do trabalho duro dos tradutores voluntários do Django). O próximo capítulo
explica como usar essa estrutura para fornecer sites com Django na língua local.

