=============================================
Capítulo 14: Sessões, Usuários, e Inscrição
=============================================

É hora de uma confissão: nós temos deliberadamente ignorado um aspecto importante de desenvolvimento web até esse ponto. Até agora, nós pensamos no tráfego visitando nossos sites como uma massa sem rosto e anônima se arremessando contra nossas páginas cuidadosamente projetadas.

Isso não é verdade, claro. Os browsers acertando nossos sites têm humanos reais por trás deles(na maioria das vezes, pelo menos). Isso é uma grande coisa a ignorar: a Internet está em seu melhor quando serve para conectar pessoas, não máquinas. Se nós vamos desenvolver sites realmente chamativos, eventualmente nós vamos ter que lidar com corpos atrás dos browsers.

Infelizmente, não é assim tão fácil. HTTP é projetado para ser *stateless* -- isto é, cada e toda requisição acontece em um vácuo. Não há persistência entre uma requisição e a próxima, e não podemos contar em nenhum aspecto da requisição(endereço IP, user agent, etc.) para consistentemente indicar requisições sucessivas da mesma pessoa.

Nesse capítulo você vai aprender a manusear essa falta de state. Nós vamos começar pelo level mais baixo(*cookies*), e trabalhar daí até ferramentas de nível mais alto para manusear sessões, usuários e inscrição.

Cookies
=======

Desenvolvedores de browser reconheceram que a falta de state do HTTP demonstra um grande problema para desenvolvedores Web há muito tempo, então *cookies* nasceram. Um cookie é um pequeno pedaço de informação que browsers guardam em favor dos servidores Web. Toda vez que o browser requisita uma página de um certo servidor, ele devolve um cookie que é inicialmente recebido.

Vamos dar uma olhada como isso pode funcionar. Quando você abre o browser and escreve ``google.com``, seu browser manda uma requisição HTTP para o Google que começa assim::
  
    GET / HTTP/1.1
    Host: google.com
    ...
    
Quando o Google responde, a resposta HTTP parece com assim::

    HTTP/1.1 200 OK
    Content-Type: text/html
    Set-Cookie: PREF=ID=5b14f22bdaf1e81c:TM=1167000671:LM=1167000671;
                expires=Sun, 17-Jan-2038 19:14:07 GMT;
                path=/; domain=.google.com
    Server: GWS/2.1
    ...
    
Note o header ``Set-Cookie``. Seu browser guarda aquele valor de cookie (``PREF=ID=5b14f22bdaf1e81c:TM=1167000671:LM=1167000671``) e serve de volta para o Google toda vez que acessa o site. Então próxima vez que você acessa o Google, seu browser vai mandar uma requisição assim::

    GET / HTTP/1.1
    Host: google.com
    Cookie: PREF=ID=5b14f22bdaf1e81c:TM=1167000671:LM=1167000671
    ...
    
Google então usa esse valor do Cookie para saber que você é a mesma pessoa que acessou o site previamente. Esse valor pode, por exemplo, ser a chave para entrar no banco de dados que armazena essa informação. Google poderia(e faz) usar isso para mostrar seu nome de usuário na página.

Pegando e configurando Cookies
---------------------------

Quando lidando com persistência em Django, boa parte do tempo você vai querer usar a sessão de level alto e/ou frameworks de usuário discutidos posteriormente neste capítulo. No entanto, uma primeira olhada em como ler e escrever cookies em um level baixo. Isso deve ajudar você a entender como o resto das ferramentas discutidas neste capítulo realmente funciona, e vai ser útil se você já tiver precisado brincar com cookies diretamente.

Ler cookies que já estão configurados é simples. Todo objeto ``HttpRequest`` tem um um objeto ``COOKIE`` que funciona como um dicionário; você pode usá-lo para ler qualquer cookie que o browser mandou para a view::

    def show_color(request):
        if "favorite_color" in request.COOKIES:
            return HttpResponse("Your favorite color is %s" % \
                request.COOKIES["favorite_color"])
        else:
            return HttpResponse("You don't have a favorite color.")
            
Escrever cookies é um pouco mais complicado. Você precisa usar o método "set_cookie()" em um objeto "HttpResponse". Aqui está um exemplo do cookie "favorite_color" baseado em um parâmetro "GET"::

    def set_color(request):
        if "favorite_color" in request.GET:

            # Create an HttpResponse object...
            response = HttpResponse("Your favorite color is now %s" % \
                request.GET["favorite_color"])

            # ... and set a cookie on the response
            response.set_cookie("favorite_color",
                                request.GET["favorite_color"])

            return response

        else:
            return HttpResponse("You didn't give a favorite color.")

.. SL Tested ok

Você também pode passar um número opcional de argumentos para "response.set_cookie()" que controla aspectos do cookie, como mostrado na Tabela 14-1

.. table:: Tabela 14-1: Opções do Cookie

    ==============  ==========  =====================================================
    Parameter       Default     Description
    ==============  ==========  =====================================================
    ``max_age``     ``None``    Idade (em segundos) que um cookie deve durar.
                                Se este parâmetro for ````None```` o cookie vai durar
                                só até quando o browser for fechado.

    ``expires``     ``None``    O dia/tempo real que o cookie deve expirar.
                                Precisa estar no formato ``"Wdy, DD-Mth-YY 
                                HH:MM:SS GMT"``. se dado, este parâmetro sobrescreve,
                                o parâmetro ``max_age``.

    ``path``        ``"/"``     O prefixo do caminho para o qual este cookie é
                                válido. Browsers só vão passar os cookies de 
                                volta para as páginas abaixo desse prefixo de 
                                caminho, então você pode usar isso para prevenir
                                que cookies sejam mandados para outras sessões desse
                                site.

                                Isso é especialmente útil quando você não controla
                                o topo do domínio do seu site.

    ``domain``      ``None``    O domínio para o qual este cookie é válido. Você 
                                pode usar este parâmetro para configurar um cookie
                                cross-domain. Por exemplo, ``domínio=".example.com'``
                                can use this parameter to set a cross-domain
                                cookie. For example, ``domain=".example.com"``
                                vai configurar um cookie que é legível pelo domínio
                                ``www.example.com``, ``www2.example.com``, e
                                ``an.other.sub.domain.example.com``.

                                Se este parâmetro for ``None``, o cookie só vai ser
                                legível para o domínio que o criou.

    ``secure``      ``False``   Se for configurado para ``True``, este parâmetro
                                instrúi o browser a só retornar este cookie para 
                                páginas acessadas por HTTPS.
    ==============  ==========  =====================================================

As Benções Misturadas dos Cookies
-----------------------------

Talvez você perceber um número de problemas em potencial com a maneira que os cookies funcionam.
Vamos dar uma olhada em algum dos mais importantes:

* Armazenamento dos cookies é voluntário; um cliente não tem que aceitar ou 
  armazenar cookies. Na verdade, todos os browsers permitem que os usuários controlem a 
  política para aceitação de cookies. Se você quer ver o quão vitais os cookies são para a 
  Web, tente ligar a opção "pergunte para aceitar cada cookie" do seu browser.

  Apesar de seu uso universal, cookies ainda são a definição de de não
  confiabilidade. Isso quer dizer que desenvolvedores devem checar que um 
  usuário de fato aceita cookies antes de depender deles.

* Cookies(especialmente aqueles que não são mandados por HTTPS) não são seguros. Porque
  dados HTTP são mandados em texto claro, cookies são extremamente vulneráveis a ataques 
  snooping. Isto é, um atacante snoopie conectado pode interceptar um cookie e lê-lo. Isso 
  quer dizer que dados sensíveis nunca devem ser armazenados em um cookie.

  Existe um ataque ainda mais insíduo, conhecido como ataque *man-in-the-middle*, 
  que é quando um atacante intercepta um cookie e o usa para parecer outro usuário. O
  capítulo 20 discute ataques dessa natureza com profundidade, assim como maneiras
  de prevení-los.

* Cookies não são seguros mesmo dos receptores esperados. A maioria dos browsers proveem 
  maneiras fácies de editar o conteúdo de cookies individuais, e usuários com recursos 
  sempre podem usar ferramentas como mechanize(http://wwwsearch.sourceforge.net/mechanize/) 
  para manualmente construir requisições HTTP.

Então você não pode armazenar dados que possam ser sensíveis a modificaçOes em cookies. O erro canônico neste cenário é armazenar algo como ``EstaLogado=1`` em um cookie quando um usuário loga. Você ficaria impressionado com o número de sites que cometem erros dessa natureza; é necessário apenas um segundo para enganar o sistema "seguro" desses sites.

Framework de Sessão do Django
==========================

Com todas essa limitações e buracos de segurança em potencial, é óbvio que cookies e sessões persistentes  são exemplos desses "pontos de dor" no desenvolvimento Web. Mas é claro que o objetivo do Django é ser um anestésico efetivo, então ele vem com um framework de sessão feito para superar essas dificuldades.

Esse framework de sessão permite armazenar e carregar dados arbitrários a base de visitantes por site. Ele armazena dados do lado do servidor e abstrai o envio e recebimento de cookies. Cookies usam um ID de sessão hashed -- não os dados mesmo -- ou seja, protejendo você da maioria dos problemas de cookie.

Vamos olhar como permitir sessões e usá-las em views.

Permitindo Sessões
-----------------

Sessões são implementadas via um pedaço de middleware (ver capítulo 17) e um modelo Django. Para permitir sessões, você vai precisar seguir os seguintes passos: 

#. Edite as configurações das suss ``MIDDLEWARE_CLASSES`` e certifique-se que 
   ``MIDDLEWARE_CLASSES`` contém 
   ``'django.contrib.sessions.middleware.SessionMiddleware'``.

#. Certifique-se que ``'django.contrib.sessions'`` está na configuração dos seus ``INSTALLED_APPS`` (e rode ``manage.py syncdb`` se tiver que adicioná-lo).

A configuração esqueleto default criado por ``startproject`` tem ambos esses bits instalados, então a menos que você os tenha removido, você provavelmente não vai precisar mudar nada para permitir sessões.

Se você não quiser usar sessões, você pode querer remover a linha ``SessionMiddleware`` do ``MIDDLEWARE_CLASSES`` e ``'django.contrib.sessions'`` dos seus ``INSTALLED_APPS``. Só vai lhe salvar uma pequena quantidade de overhead, mas cada pequeno bit conta.

Usando Sessões em Views
-----------------------

Quando ``SessionsMiddleware`` é ativado, cada objeto ``HttpRequest`` -- o primeiro argumento de qualquer função view Django -- vai ter um atributo ``sessão``, que é um objeto parecido com um dicionário. Você pode ler e escrever nele da mesma maneira que um dicionário normal. Por exemplo, em uma view você pode fazer coisas como::

    # Configurar o valor da sessão:
    request.session["fav_color"] = "blue"

    # Pegar um valor de sessão -- isso pose set chamado em uma view diferente,
    # ou muitas requisições depois(ou ambos):
    fav_color = request.session["fav_color"]

    # Limpar um item da sessão:
    del request.session["fav_color"]

    # Checar se a sessão tem uma dada chave:
    if "fav_color" in request.session:
        ...

Você também pode usar outros métodos de dicionário como ``keys()`` e ``items()`` no ``request.session``.

Existem algumas regras simples para usar sessões Django efetivamente:

* Use strings Python normal como chaves de dicionários em ``request.session`` 
  (em contraponto a integers, objects, etc.).

*  Chaves de dicionários de sessão que começam com underscore são reservados para uso 
   interno do Django. Na prática, o framework usa apenas poucas variáves de sessão 
   prefixadas com underscore, mas a menos que você saiba o que eles são (e está disposto a 
   se manter atuaizado com qualquer mudança no próprio Django), se manter longe do prefixo 
   underscore vai manter Django de interferir com sua aplicação.

   Por exemplo, não use a chave de sessão chamada ``fav_color``, assim:
      request.session['_fav_color'] = 'blue' # Não faça isso!

*  Não substitua ``request.session`` com um novo objeto, e não acesse ou configure seues 
   atributos. Use ele como um dicionário Python. Exemplos::

      request.session = some_other_object # Não faça isso!

      request.session.foo = 'bar' # Não faça isso!

Vamos dar uma olhada em alguns exemplos rápidos. Essa view simples configura uma  variável ``has_commentend`` como ``True`` depois do usuário postar um comentário. É uma maneira simples (se não particularmente segura) de previnir o usuário de postar mais de um comentário::

    def post_comment(request):
        if request.method != 'POST':
            raise Http404('Only POSTs are allowed')

        if 'comment' not in request.POST:
            raise Http404('Comment not submitted')

        if request.session.get('has_commented', False):
            return HttpResponse("You've already commented.")

        c = comments.Comment(comment=request.POST['comment'])
        c.save()
        request.session['has_commented'] = True
        return HttpResponse('Thanks for your comment!')

Essa view simples tem saída em um "member" do site::

    def login(request):
        if request.method != 'POST':
            raise Http404('Only POSTs are allowed')
        try:
            m = Member.objects.get(username=request.POST['username'])
            if m.password == request.POST['password']:
                request.session['member_id'] = m.id
                return HttpResponseRedirect('/you-are-logged-in/')
        except Member.DoesNotExist:
            return HttpResponse("Your username and password didn't match.")

E este aqui tem como saída um membro que logou via ``login()`` acima::

    def logout(request):
        try:
            del request.session['member_id']
        except KeyError:
            pass
        return HttpResponse("You're logged out.")

.. note::

    Na prática, essa é uma maneira feia de logar usuários. O framework
    autenticação discutido brevemente lida com essa tarefa para você de uma
    maneira muito mais robusta e útil. Esses exemplos são deliberadamente
    simples para que você possa facilmente ver o que está acontecendo.

Configurando Testes de Cookies
------------------------------

Como mencionado acima, você não pode depender de cada browser aceitar cookies. Então, como conveniência, Django provem uma maneira fácil de testar se o browser do usuário aceita cookies. Basta chamar ``request.session.set_test_cookie()`` em uma view, e cheque ``request.session.test_cookie_worked()`` em uma view subsequente -- não em uma mesma chamada de view.

Essa divisão estranha entre ``set_test_cookie`` e ``test_cookie_worked()`` é necessário graças a maneira que cookies funcionam. Quando um cookie é configurado, você não pode dizer se um browser o aceitou até a próxima requisição do browser.

É boa prática usar ``delete_test_cookie()`` para limpar depois de você mesmo. 
Faça isso depois de verificar que o cookie de teste funcionou.

Aqui está um exemplo de uso típico::

    def login(request):

        # If we submitted the form...
        if request.method == 'POST':

            # Check that the test cookie worked (we set it below):
            if request.session.test_cookie_worked():

                # The test cookie worked, so delete it.
                request.session.delete_test_cookie()

                # In practice, we'd need some logic to check username/password
                # here, but since this is an example...
                return HttpResponse("You're logged in.")

            # The test cookie failed, so display an error message. If this
            # were a real site, we'd want to display a friendlier message.
            else:
                return HttpResponse("Please enable cookies and try again.")

        # If we didn't post, send the test cookie along with the login form.
        request.session.set_test_cookie()
        return render(request, 'foo/login_form.html')

.. note::

	Novamente, as funções de autenticação prontas lidam com essa checagem para você.

Usando Sessões fora de Views
----------------------------

Internamente, cada sessão é apenas um modelo Django normal definido em ``django.contrib.sessions.models``. Cada sessão é identificado por um hash randômico de 32 caracteres armazenado em um cookie. Já que é um modelo normal, você pode acessar sessões usando a API de banco de dados normal do Django::

    >>> from django.contrib.sessions.models import Session
    >>> s = Session.objects.get(pk='2b1189a188b44ad18c35e113ac6ceead')
    >>> s.expire_date
    datetime.datetime(2005, 8, 20, 13, 35, 12)

Você precisa chamar "get_decoded()" para pegar os dados da sessão. Isto é necessário porque o dicionário é armazenado em um formado codificado::

    >>> s.session_data
    'KGRwMQpTJ19hdXRoX3VzZXJfaWQnCnAyCkkxCnMuMTExY2ZjODI2Yj...'
    >>> s.get_decoded()
    {'user_id': 42}

Quando Sessões são Salvos
-------------------------

Por default, Django só salva no banco de dados se a sessão tiver sido modificada -- isto é, se algum dos valores de dicionários tiverem sido atribuído ou deletado::


    # Session is modified.
    request.session['foo'] = 'bar'

    # Session is modified.
    del request.session['foo']

    # Session is modified.
    request.session['foo'] = {}

    # Gotcha: Session is NOT modified, because this alters
    # request.session['foo'] instead of request.session.
    request.session['foo']['bar'] = 'baz'

Para mudar esse comportamente default, configure ``SESSION_SAVE_EVERY_REQUEST`` para ``True``. Se ``SESSION_SAVE_EVERY_REQUEST`` for ``True``, Django vai salvar a sessão no banco de dados em cada requisição, mesmo que não sido modificado.

Note que o cookie da sessão só é mandado quando a sessão tiver sido criado ou modificada. Se ``SESSION_SAVE_EVERY_REQUEST`` for ``True``, o cookie da sessão vai ser mandado em cada requisição. Similarmente, a parte ``expires`` de um cookie de sessão é atualizado cada vez que o cookie de sessão é mandado.

Sessões de Tamanho de Browser(browser-length) vs. Sessões Persistentes
----------------------------------------------------------------------

Você deve ter percebido que o cookie mandado pelo Google para nós no começo desse capítulo continha ``expires=Sun, 17-Jan-2038 19:14:07 GMT;``. Cookies podem conter opcionalmente uma data de expiração que aconselha o browser em quando ele deve remover o cookie. Se um cookie não contem uma valor de expiração, o browser vai expirá-lo quando o usuário fechar a sua janela de browser. Você pode controlar o comportamento do framework de sessão nesse sentido com a configuração ``SESSION_EXPIRE_AT_BROWSER_CLOSE``.

Por default, ``SESSION_EXPIRE_AT_BROWSER_CLOSE`` é configurando para ``False``, o que quer dizer que cookies de sessão vão ser armazenados em browsers de usuários por "SESSION_COOKIE_AGE" segundos (que por default é duas semanas, ou 1,209,600 segundos). Use isso se você não quiser que pessoas tenham acesso cada vez que abrem o browser.

Se ``SESSION_EXPIRE_AT_BROWSER_CLOSE`` é configurado para ``True``, Django vai usar cookies de tamanho de browser(browser-length).

Outras Configurações de Sessão
------------------------------

Além de configurações já mencionadas, outras poucas configurações influenciam como o framework de sessão do Django usa cookies, como mostrado na Tabela 14-2.

.. table:: Table 14-2. Settings that influence cookie behavior

    ==========================  =============================  ==============
    Configuração                Descrição                      Default
    ==========================  =============================  ==============
    ``SESSION_COOKIE_DOMAIN``   O domínio para usar cookies    ``None``
                                de sessão. Configure isso para
 						  uma string como ``"example.com"``
                                para cookies cross-domain,
                                ou use ``"None"``para um cookie 
						  básico.

    ``SESSION_COOKIE_NAME``     O nome do cookie para user     ``"sessionid"``
						  sessões. Isto pose set qualquer
						  string. 

    ``SESSION_COOKIE_SECURE``   Usar ou não um cookie "seguro" ``False``
                                para o cookie de sessão. Se
                                esse cookie for configurado 
                                para ``True``, o cookie vai
                                set marcado como "seguro"                                						  isso significa que o browser                                				 		  vai guarantir que o cookie só 
                                será envied via HTTPS.
    ==========================  =============================  ==============

.. admonition:: Detalhes técnicos

    Para os curiosos, aqui estão algumas notas técnicas sobre os trabalhos internos do
    framework de sessão:

    * O dicionário de sessão aceita qualquer objeto Python ccapaz de ser
      "picado"(``pickled`` em inglês). Veja a documentação feita pelo módulo 
      ``pickle`` feito no Python para informação sobre como isso funciona.

    * Os dados de sessão são armazenados em um banco de dados numa tabela com o 
      nome ``django_session``.

    * Dados de sessão são retornados de acordo com a demanda. Se você nunca 
      acessou ``request.session``, Django não vai acertar aquela tabela do
      banco de dados.

    * Django só manda um cookie se precisar fazê-lo. Se você não configurar
      qualquer dado de sessão, ele não vai mandar nenhum cookie de sessão (a menos que
      ``SESSION_SAVE_EVERY_REQUEST`` esteja configurado para
      ``True``).

    * O framework de sessão é inteiramente, e apenas, baseado em cookie.
      Ele não dá um passo para trás e colocar IDs e URLs da sessão como último
      recurso, como outras ferramentas(PHP, JSP) fazem.

      Essa é uma decisão de design intencional. Colocar sessões em URLs não só
      faz a URL ficar feia, mas também faz o seu site ficar vulnerável a
      certos tipos de formulários de roubo de ID via o cabeçalho ``Referer``.

    Se vicê ainda está curioso, a fonte é bem direta; olhe em 	
    ``django.contrib.sessions`` para mais detalhes.

Usuários e Autenticação
=======================

Sessões nos dão uma maneira de persistir dados atráves de requisições múltiplas de browsers; a segunda parte da equação é usar essas sessões para login do usuário. Claro, não podemoes confiar que usuários são quem dizem ser, então precisamos autenticá-los no meio do caminho.

Naturalmente, Django prover ferramentas para lidar com essa tarefa comum (e muitas outras). O sistema de autenticação de usuário do Django lida com contas, grupos, permissões de usuários, assim como sessões de usuários baseadas em cookies. Esse sistema é comumemnte referido como um sistema *auth/auth* (autenticação e autorização). Esse nome reconhece que lidar com usuários é, comumente, um processo de dois passos. Nós precisamos

#. Verificar (*autenticar*) que um usuário é que ele ou ela diz ser
   (normalmente ao checar o nome de usuário e o password contra um banco de dados de usuários)
#. Verificar que o usuário é *autorizado* a fazer algumas operações
   (normalmente ao checar em uma tabela de permissões)

Continuando com essas necessidades, o sistema de auth/auth do Django consiste de um número de partes:

* *Users*: Pessoas registradas com seu site

* *Permissions*: Flags binárias(sim/não) informando se o usuário pode ou não fazer certas tarefas

* *Groups*: Uma maneira genérica de aplicar rótulos e permissões para mais de um usuário

* *Messages*: Uma maneira simples de enfileirar e mostrar mensagens do sistema para usuários

Se você usou a ferramenta de administrado (discutida no Capítulo 6), você já viu muitas dessas ferramentas, e se você editou usuários ou grupos na ferramentas de administrador, você tem editado dados nas tabelas do banco de dados do sistema auth.

Permitindo Suporte de Autenticação
==================================

Como as ferramentas de sessão, suporte de autenticação é empacotado como uma aplicação Django em ``django.contrib`` que precisa ser instalado. Assim como as ferramentas de sessão, também é instalado por default, mas se você o removeu, você vai querer seguir os seguintes passos para instalar:

#. Verifique que o framework de sessão está instalado como descrito mais cedo
   nesse capítulo. Para acompanhar os usuários, é óbvio que são necessários
   cookies, então são feitos no framework de sessão.

#. Coloque ``'django.contrib.auth'`` nas suas configurações do
   ``INSTALLED_APPS`` e rode ``manage.py syncdb`` para instalar as tabelas do
   banco de dados apropriados.

#. Certifique-se que     
   ``'django.contrib.auth.middleware.AuthenticationMiddleware'`` está na sua
   configuração do ``MIDDLEWARE_CLASSES`` -- *depois*
   do ``SessionMiddleware``.

Com essa instalação fora do caminho, estamos prontos para lidar com usuários em funções de view. A interface principal que você irá usar para acessar usuários em uma view é ``request.user``; isto é um objeto que represta os usuários que estão logados atualmente. Caso o usuário não esteja logado, isto será um objeto ``AnonymousUser`` (veja abaixo para mais detalhes).

Você pode facilmente dizer se um usuário está logado com o método ``is_authenticated()``::

    if request.user.is_authenticated():
        # Fazer algo para usuários autenticados
    else:
        # Fazer algo para usuários anônimos.

Usando Usuários
---------------

Quando você tiver um ``User`` -- normalmente vindo de ``request.user``, mas possivelmente através de outros métodos discutidos em breve -- você terá um número de campos e métodos disponíveis neste objeto. O objeto ``AnonymousUser`` emula *algumas* dessas interfaces disponíveis, mas não todas, então você deve sempre checar ``user.is_authenticated()`` antes de assumir que você está lidando com um objeto usuário bona fide. As Tabelas 14-3 e 14-4 listam os campos e métodos, respecitavemente, em objetos ``User``.

.. table:: Tabela 14-3. Campos em objetos ``User``

    ==================  ======================================================
    Campo               Descrição
    ==================  ======================================================
    ``username``        Obrigatório; 30 caracters ou menos. Carácteres 
                        alfanuméricos apenas (letras, dígitos e underscores).
                        characters only (letters, digits, and underscores).
    
    ``first_name``      Opcional; 30 caracters ou menos.

    ``last_name``       Opcional; 30 caracters ou menos.

    ``email``           Opcional. Endereço de e-mail.

    ``password``        Obrigatório. Um hash de, e metadados sobre, o password
                        (Django não armazena o password cru). Ver sessão  
                        "Passwords" section para mais sobre este valor.

    ``is_staff``        Booleano. Designa se o usuário pode acessar o site de 
                        administrador.

    ``is_active``       Booleano. Designa se esta conta pode ser usada para
                        logar. Configure para ``False`` ao invés de deletar
                        contas.

    ``is_superuser``    Boolean. Designa que este usuário tem todas as
                        permissões sem explicitamente atribuí-las 
                        explicitamente.

    ``last_login``      Uma datetime de quando o usuário logou pela última
                        vez. Isto é configurado para date/time por default.

    ``date_joined``     Um datetime designado quando a conta foi criada.
                        Isto é configurado para date/time por default quando
                        a conta é criada.
    ==================  ======================================================

.. table:: Tabela 14-4. Métodos no objeto ``User``

    ================================  ==========================================
    Método                            Descrição
    ================================  ==========================================
    ``is_authenticated()``            Sempre retorna ``True`` para objetos 
                                      ``User`` "reais". Esta é uma maneira de
                                      dizer que o usuário foi autenticado. Isto
                                      não implica em qualquer permissão, e não
                                      checa se o usuário está ativo. Só indica
                                      que o usuário foi autenticado com sucesso.

    ``is_anonymous()``                Retorna ``True`` somente para objetos
                                      ``AnonymousUser`` (e ``Falso`` para 
                                      objetos ``User`` "reais". Geralmente,
                                      você deve preferir usar 
                                      ``is_authenticated()`` a esse método.

    ``get_full_name()``               Retorna o ``first_name`` mais o 
                                      ``last_name``, com um espaço no meio.

    ``set_password(passwd)``          Atribui o password de usuário para a 
                                      string crua dada, tendo cuidado com o
                                      hashing password. Isto não salva realmente
                                      o objeto ``User``.

    ``check_password(passwd)``        Retorna ``True`` se a string crua dada
                                      é o password correto do usuário. Isto 
                                      toma conta do hashing do password ao
                                      fazer a comparação.

    ``get_group_permissions()``       Retorna uma lista de strings de permissões 
                                      que o usuário tem nos grupos aos quais
                                      ele pertence.

    ``get_all_permissions()``         Retorna uma lista de strings de permissões
                                      que o usuário tem, tanto de grupo
                                      quanto permissão de usuário.

    ``has_perm(perm)``                Retorna ``True`` se o usuário tem 
                                      permissão especificada, onde ``perm`` está
                                      no formato ``"package.codename"``. Se
                                      o usuário está inativo, este método sempre
                                      vai retornar ``False``.

    ``has_perms(perm_list)``          Retorna ``True`` se o usuário tiver 
                                      *todos* as permissões especificadas.
                                      Se o usuário estiver inativo, este 
                                      método vai sempre retornar ``False``.

    ``has_module_perms(app_label)``   Retorna ``True`` se o usuário tiver dada
                                      permissão na ``app_label``.
                                      Se o usuário estiver inativo, este método
                                      retorna ``False``.

    ``get_and_delete_messages()``     Retorna a lista de objetos ``Message``
                                      na lista do usuário e deleta as mensagens
                                      da lista.

    ``email_user(subj, msg)``         Manda um email para o usuário. Este email
                                      é mandado da configuração 
                                      ``DEFAULT_FROM_EMAIL``. Você também pode
                                      passar um terceiro argumento, 
                                      ``from_email``, para sobre escrever o 
                                      endereço De do email.          
   ================================  ==========================================

Finalmente, objetos ``User`` tem dois campos n-para-n: ``groups`` e ``permissões``. Objetos ``User`` podem acessar seus objetos relacionados da mesma forma como qualquer outro campo n-para-n::

        # Configura um grupo de usuário:
        myuser.groups = group_list

        # Adiciona um usuário a algum grupo:
        myuser.groups.add(group1, group2,...)

        # Remove algum usuário de algum grupo:
        myuser.groups.remove(group1, group2,...)

        # Remove um usuário de todos os grupos:
        myuser.groups.clear()

        # Permissões funcionam do mesmo jeito
        myuser.permissions = permission_list
        myuser.permissions.add(permission1, permission2, ...)
        myuser.permissions.remove(permission1, permission2, ...)
        myuser.permissions.clear()

Logando e deslogando
--------------------

Django provê funções de view feitas para lidar com login e deslogin (e outros truques estilosos), mas antes de chegarmos nesses, vamos dar uma olhada em como logar e deslogar o usuário "na mão". Django provê duas funções para realizar essas ações em ``django.contrib.auth``: ``authenticate()`` e ``login()``.

Para autenticar um dado nome de usuário e password, use ``authenticate()``. Isso precisa de dois argumentos chaves, ``username`` e ``password``, e retorna um objeto ``User`` se o password for válido para dado usuário. Se o password for inválido, ``authenticate()`` retorna ``None``::

    >>> from django.contrib import auth
    >>> user = auth.authenticate(username='john', password='secret')
    >>> if user is not None:
    ...     print "Correct!"
    ... else:
    ...     print "Invalid password."

``authenticate()`` só verifica a credencial de um usuário. Para logar o usuário, use ``login()``. É preciso um objeto ``HttpRequest`` e um objeto ``User`` e salva o ID do usuário na sessão, usando o framework de sessão do Django.

Este exemplo mostra como você pode usar tanto ``authenticate()`` e ``login()`` dentro de uma função de view::

    from django.contrib import auth

    def login_view(request):
        username = request.POST.get('username', '')
        password = request.POST.get('password', '')
        user = auth.authenticate(username=username, password=password)
        if user is not None and user.is_active:
            # Password correto, e o usuário é marcado como "ativo"
            auth.login(request, user)
            # Redireciona para uma página de sucesso.
            return HttpResponseRedirect("/account/loggedin/")
        else:
            # Mostra uma página de erro
            return HttpResponseRedirect("/account/invalid/")

Para deslogar o usuário, use ``django.contrib.auth.logout()`` dentro da view. É preciso um objeto ``HttpRequest`` e não tem um valor de retorno::

    from django.contrib import auth

    def logout_view(request):
        auth.logout(request)
        # Redireciona para uma página de sucesso.
        return HttpResponseRedirect("/account/loggedout/")

Note que ``auth.logout()`` não joga nenhum erro se o usuário não estivesse logado.

Na prática, você não vai precisar escrever suas próprias funcções login/logout; o sistema de autenticação vem com um conjunto de views para lidar genericamente login e logout. O primeiro passo em usar essas views de autenticação é ligá-las em URLconf. Você vai precisar adicionar esse fragmento::

    from django.contrib.auth.views import login, logout

    urlpatterns = patterns('',
        # existing patterns here...
        (r'^accounts/login/$',  login),
        (r'^accounts/logout/$', logout),
    )

``/accounts/login/`` e ``/accounts/logout/`` são as URLs default que Django usa para essas views.

Por default, a view ``login`` renderiza um template na ``registration/login.html`` (você pode mudar este nome de template passando um argumento de view extra, ``template_name``). Este formulário precisa conter ``username`` e um campo ``password``. Um simples template pode parecer assim::

   {% extends "base.html" %}

    {% block content %}

      {% if form.errors %}
        <p class="error">Sorry, that's not a valid username or password</p>
      {% endif %}

      <form action="" method="post">
        <label for="username">User name:</label>
        <input type="text" name="username" value="" id="username">
        <label for="password">Password:</label>
        <input type="password" name="password" value="" id="password">

        <input type="submit" value="login" />
        <input type="hidden" name="next" value="{{ next|escape }}" />
      </form>

    {% endblock %}

Se o usuário logar com sucesso, ele ou ela vai ser redirecionado para ``/accounts/profile/`` por default. Você pode sobre escrever isto provendo um campo um escondido chamando ``next`` com a URL para onde redirecionar depois de logar. Você também pode passar este valor como um parâmetro ``GET`` para a view de login e vai automaticamente adicionar para o contexto como uma variável chamada ``next`` que você pode inserir no campo escondido.

A view de logout funciona de uma maneira um pouco diferente. Por defaul ela renderiza o template em ``registration/logged_out.html`` (que normalmente contém uma mensagem "Você deslogou com sucesso"). No entanto, você pode chamar a view com um argumento extra, ``next_page``, que vai instruir a view para redirecionar depois do logout.

Limitando Acesso para Usuários Logados
--------------------------------------

Claro, a razão que estamos passando por todo esse trabalho é para que possamos limitar o acesso para partes do nosso site.

A maneira simples, crua de limitar acesso a páginas é checar ``request.user.is_authenticated()`` e redirecionado para uma página de login::

    from django.http import HttpResponseRedirect

    def my_view(request):
        if not request.user.is_authenticated():
            return HttpResponseRedirect('/accounts/login/?next=%s' % request.path)
        # ...

ou talvez mostrar uma mensagem de erro::

    def my_view(request):
        if not request.user.is_authenticated():
            return render(request, 'myapp/login_error.html')
        # ...

Como atalho, você pode usar o decorador conveniente ``login_required``::

    from django.contrib.auth.decorators import login_required

    @login_required
    def my_view(request):
        # ...

``login_required`` faz o seguinte:

* Se o usuário não estiver logado, redirecione para ``/accounts/login/``, passando
  o caminho de URL atual na string de query como ``next``, por exemplo:
  ``/accounts/login/?next=/polls/3/``.

* Se o usuário estiver logado, execute a view normalmente. O código de view pode então
  assume que o usuário está logado.

Limitando Acesso para Usuário que Passam o Teste
------------------------------------------------

Limitando acesso baseado em certas permissões ou alguns outros testes, ou provendo um local diferente para a view de login funciona essencialmente da mesma maneira.

A maneira crua é de rodar seus testes em ``request.user`` na view diretamente. Por exemplo, esta view checa para ter certeza que o usuário está logado e tem a permissão ``polls.can_vote`` (mais em como permissões funcionam a seguir)::

    def vote(request):
        if request.user.is_authenticated() and request.user.has_perm('polls.can_vote')):
            # vote here
        else:
            return HttpResponse("You can't vote in this poll.")

Novamente, Django provê um atalhado chamado ``user_passes_test``. Ele aceita argumentos e gera um decorator especializado para sua situação em particular::

    def user_can_vote(user):
        return user.is_authenticated() and user.has_perm("polls.can_vote")

    @user_passes_test(user_can_vote, login_url="/login/")
    def vote(request):
        # Code here can assume a logged-in user with the correct permission.
        ...

``user_passes_test`` recebe um argumento obrigatório: um callable que aceita um objeto ``User`` e retorna ``True`` se o usuário puder ver a página. Note que ``user_passes_test`` não checa automaticamente que ``User`` está autenticado; você de fazer isso.

Neste exemplo nós também estamos mostrando o segundo argumento (opcional), ``login_url``, o qual deixa especificar a URL para sua página de login de usuário (``/accounts/login/`` por default). Se o usuário não passar o teste, então o decorator ``user_passes_test`` vai redirecionar o usuário para ``login_url``.

Já que é um tarefa relativamente comum checar se o um usuário tem uma permissão em particular, Django provê um atalho para este caso: o decorator ``permission_require()``. Usando este decorator, o exemplo anterior pode ser escrito assim::

    from django.contrib.auth.decorators import permission_required

    @permission_required('polls.can_vote', login_url="/login/")
    def vote(request):
        # ...

Note que ``permission_required()`` também recebe um parâmetro ``login_url`` opcional, que também tem defaul ``'/accounts/login'``.

.. admonition:: Limitando acesso para views genéricas

     Uma das perguntas mais frenquentemente perguntas na lista de usuários Django lida
     com limitando acesso a views genéricas. Para fazer isso, você precisará escrever um
     pequeno wrapper ao redor da view e apontar sua URLconf para seu wrapper
     ao invés da view genérica::

        from django.contrib.auth.decorators import login_required
        from django.views.generic.date_based import object_detail

        @login_required
        def limited_object_detail(*args, **kwargs):
            return object_detail(*args, **kwargs)

     Você pode, claro, repor ``login_required`` com qualquer outro decorators limitadores.

Gerenciando Usuários, Permissões, e Grupos
------------------------------------------

A maneira de longe mais fácil de gerenciar o sistema auth é atráves da interface de usuário. O capítulo 6 discute como usar o site admin do Django para editar usuários e controlar as permissões e o acesso deles, e boa parte do tempo você só vai usar essa interface.

No entanto, existem APIs de nível baixo que você pode mergulhar quando precisar de controle absoluto, e vamos discutir essas nas sessões a seguir.

Criando Usuários
````````````````

Crie usuários com a função de ajuda ``create_user``::

    >>> from django.contrib.auth.models import User
    >>> user = User.objects.create_user(username='john',
    ...                                 email='jlennon@beatles.com',
    ...                                 password='glass onion')

Neste ponto, ``user`` é uma instância ``User`` pronta para ser salva no banco de dados (``create_user()`` não pode realmente chamar ``save()``). Você pode continuar a mudar este atributos antes de salvar também:: 

    >>> user.is_staff = True
    >>> user.save()

.. SL Tested ok

Mudando Passwords
`````````````````

Você pode mudar um password com ``set_password()``::

    >>> user = User.objects.get(username='john')
    >>> user.set_password('goo goo goo joob')
    >>> user.save()

.. SL Tested ok

Não atribua o atributo ``password`` diretamente a menos que você saiba o que você está fazendo. O password é armazenado como um *salted hash* e por isso não pode ser editado diretamente.

Mais formalmente, o atributo ``password`` de um objeto ``User`` é uma string nesse formato::

    hashtype$salt$hash

Isto é um tipo de hash, o salt, e o hash mesmo, separados pelo caractere cifrão.

``hashtype`` é ``shal`` (default( ou ``md5``, o algoritmo usado para realizar um one-way hash de um password. ``salt`` é uma string randômica usada para salgar o password cru para criar o hash, por exemplo::

    sha1$a1976$a36cc8cbf81742a8fb52e221aaeab48ed7f58ab4

As funções ``User.set_password()`` e ``User.check_password()`` lidam com a atribuição e checagem desses valores por trás das cenas.

.. admonition:: Hashes salgados

   Um *hash* é uma função criptográfica one-way -- isto é, você pode facilmente computar
   um hash de um dado valor, mas é quase impossível de pegar um hash e reconstruir
   o valor original.

   Se armazenássemos o password como um texto normal, qualquer que conseguisse pôr as mãos
   no banco de dados dos passwords poderia instantaneamente saber o password de todo mundo.
   Armazenar os hashes de password reduzem o valor de um banco de dados comprometido.

   No entanto, um atacante com o banco de passwords ainda pode rodar um ataque de 
   *força bruta*, fazendo o hash de milhões de passwords e comparando esses hashes com os
   valores armazenados. Isto leva um tempo, mas menos do que você pensa.

   Pior, existem *tabelas arco-íris* publicamente disponíels, ou banco de dados de hashes
   pre computados de milhões de passwords. Com uma tabela arco-íris, um atacante experiente
   pode quebrar a maioria dos passwords em segundos.

   Adicionando um *salt* -- basicamente um valor inicial randômico -- para armazenar hash
   adicioan outra camada de dificuldade para quebrar passwords. Já que salts diferem de 
   password para password, eles também podem prevenir o uso de tabelas arco-íris, então 
   forçando atacantes a voltar para um ataque de força bruta, que por si só é mais difícil
   graças a entropia extra adicionada ao hash pelo salt.

   Enquanto salted hashes não são absolutamente a maneira mais segura de armazenar 
   passwords, eles são um bom meio termo entre segurança e conveniência.

Lidando com Registros
`````````````````````

Podemos usar as ferramentas de nível baixo para criar views que permite usuários registrar para novas contas. Desenvolvedores diferentes implementam registro de maneiras diferentes, então Django deixa a escrita de view de registro com você. Por sorte, é bem fácil

No mais fácil, poderiamos prover uma pequena view que pede as informações de usuários obrigatórias e criar esses usuários. Django provê um formulário feito que você pode usar para este propóstio, o qual vamos usar neste exemplo::

    from django import forms
    from django.contrib.auth.forms import UserCreationForm
    from django.http import HttpResponseRedirect
    from django.shortcuts import render

    def register(request):
        if request.method == 'POST':
            form = UserCreationForm(request.POST)
            if form.is_valid():
                new_user = form.save()
                return HttpResponseRedirect("/books/")
        else:
            form = UserCreationForm()
        return render(request, "registration/register.html", {
            'form': form,
        })

Este formulário assume um template nomeado ``registration/register.html``. Aqui está um exemplo com o que este template pode parecer::

  {% extends "base.html" %}

  {% block title %}Create an account{% endblock %}

  {% block content %}
    <h1>Create an account</h1>

    <form action="" method="post">
        {{ form.as_p }}
        <input type="submit" value="Create the account">
    </form>
  {% endblock %}

.. SL Tested ok

Usando Dados de Autenticação em Templates
-----------------------------------------

O usuário logado atual e suas permissões são feitos disponíveis no contexto de templates quando você usa o atalho  ``render()`` ou explicitamente usa um ``RequestContext``(ver Capítulo 9).

.. note::


   Tecnicamente, essas variáves só são feitas disponíves no template
   de contexto se você usar ``RequestContext`` *e* suas configurações
   ``TEMPLATE_CONTEXT_PROCESSORS`` contêm ``"django.core.context_processors.auth"``,
   que é o default. Novamente, ver o Capítulo 9 para mais informação.

Quando usando ``RequestContext``, o usuário atual (seja uma instância de ``User`` ou de ``AnonymousUser``) é armazenado em uma variável de template ``{{ user }}``::

    {% if user.is_authenticated %}
      <p>Welcome, {{ user.username }}. Thanks for logging in.</p>
    {% else %}
      <p>Welcome, new user. Please log in.</p>
    {% endif %}

Essas permissões de usuário são armazenadas na variável de template ``{{ pems }}``. Isto é um proxy amigável para templates para alguns métodos de permissão descritos em breve,

Existem duas maneiras que você pode usar esse objeto ``perms``. Você pode usar algo como ``{% if perms.polls %}`` para checar se o usuário tem *qualquer* permissão para uma dada aplicação, ou você pode usar algo como ``{% if perms.polls.can_vote %}`` para checar se o usuário tem uma permissão específica.

Entao, você pode checar permissões em declarações template ``{% if %}``::

    {% if perms.polls %}
      <p>You have permission to do something in the polls app.</p>
      {% if perms.polls.can_vote %}
        <p>You can vote!</p>
      {% endif %}
    {% else %}
      <p>You don't have permission to do anything in the polls app.</p>
    {% endif %}

Permissões, Grupos e Mensagens
==============================

Existem alguns outros bits do framework de autenticação que nós só lidamos em passagem. Vamos dar uma olhada neles nas seções a seguir.

Permissions
-----------

Permissões são uma maneira simples de "marcar" usuários e grupos como sendo capazes de realizar uma determinada ação. Elas são normalmente usadas por pelo site admin do Django, mas você pode facilmente usá-las no seu próprio código.

O site admin do Django usa permissões da seguinte forma:

* Acesso para ver o formulário "adicionar", e adicionar um objeto é limitado a usuários
  com a permissão *add* para aquele tipo de objeto.

* Acesso para ver a lista de mudança, ver o formulário de "mudança", e mudar um objeto 
  é limitado a usuários com a permissão *change* para o tipo do objeto.

* Acesso para deletar um objeto é limitado a usuários com a permissão *delete* para 
  aquele tipo de objeto.

Permissões são atribuidas globalmente por tipo de objeto, não por instância de objeto específica. Por exemplo, é possível dizer "Mary pode mudar novas histórias," mas permissões não permitem dizer "Mary pode mudar novas histórias, mas só aquelas que ela mesmo criou" ou "Mary só pode mudar novas histórias que tem um certo status, data de publicação, ou ID."

Essas três permissões básicas -- add, change, e delete -- são automaticamente criadas para cada modelo Django. Por trás das cenas, essas permissões são adicionadas a tabela do banco de dados ``auth_permission`` quando você roda ``manage.pu syncdb``.

Essas permissões vão estar no formato ``"<app>.<action>_<object_name>"``. Isto é, se você tem uma aplicação ``polls` com um modelo ``Choice``, você vai conseguir a permissão com o nome ``"polls.add_choice"``,  ``"polls.change_choice"``, and
``"polls.delete_choice"``.

Assim como usuários, permissões são implementadas no modelo Django que vive em ``django.contrib.auth.models``. Isto quer dizer que você pode usar a API de banco de dados do Django para interagir diretamente com permissões que você quiser.

Grupos
------

Grupos são uma maneira genérica de categorizar usuários para que você possa aplicar permissões, ou alguns outros rótulos, para aquels usuários. Um usuário pode pertencer a qualquer número de grupos.

Um usuários em um grupo automaticamente tem as permissões dadas àquele grupo. Por exemplo, seu o grupo ``Site editors`` tem a permissão ``can_edit_home_page``, qualquer usuário naquele grupo vai ter aquela permissão.

Grupos também são uma maneira conveniente de categorizar usuários para lhes dar rótulos, ou funcionalidades extendidas. Por exemplo, você poderia criar um grupo ``'Special users'``, e você poderia escrever código que poderia, digamos, dar àqueles usuários acesso a porções só de membros do site, ou mandar eles para mensagens de e-mail que são só para membros.

Como usuários, a maneira mais fácil de gerenciar grupos é através de interface de admin.
No entanto, grupos são só modelos Django que vivem em ``django.contrib.auth.models``, então mais uma vez você pode sempre usar as APIs do banco de dados do Django para lidar com grupos num nível baixo.

Mensagens
---------

O sistema de mensagem é uma maneira leve de enfileirar mensagens para dados usuários. Uma mensagem é associada com um ``User``. Não existe conceito de expiração ou timestamps.

Mensagens são usadas pela interface adming Django depois de ações bem sucedidas. Por exemplo, quando você cria um objeto, você vai perceber uma mensagem "O objeto foi criado com sucesso"("The object was created successfully" em inglês) no topo da página de admin.

Você pode usar a mesma API para enfileirar e mostrar mensagens em sua própria aplicação.
A API é simples:

* Para criar uma nova mensagem, use ``user.message_set.create(message='message_text')``.

* Para pegar/deletar uma mensagem, use ``user.get_and_delete_messages()``,
  a qual retorna uma lista de objetos ``Message`` na fila de usuários (se houver)
  e deleta esta mensagem da lista.

Nesta view de exemplo, O sistema salva uma mensagem para o usuário depois de criar uma playlist::

    def create_playlist(request, songs):
        # Create the playlist with the given songs.
        # ...
        request.user.message_set.create(
            message="Your playlist was added successfully."
        )
        return render(request, "playlists/create.html")

Quando você usa o atalho ``render()`` ou renderiza um template com um ``RequestContext``, o usuário atualmente logado e suas mensagens são disponibilizadas no contexto de template como variável de template ``{{ messages  }}``. Aqui está um exemplo de um código template que mostra mensagens::

    {% if messages %}
    <ul>
        {% for message in messages %}
        <li>{{ message }}</li>
        {% endfor %}
    </ul>
    {% endif %}

Note que ``RequestContext`` chama ``get_and_delete_messages`` por trás das cenas, então qualquisquer mensagens vão ser deletadas mesmo que você não as mostre.

Finalmente, note que este framework de mensagens só funciona com usuários no banco de dados de usuários. Para mandar mensagens para usuários anônimos, use o framework de sessão diretamente.

A seguir
========

O sistema de sessão e autorização é muita coisa para absorver. Boa parte do tempo,
você não vai precisar todas essas características descritas neste capítulo, mas quando você precisar permitir interações complexas entre usuários, é bom ter todo este poder disponível.

No `next chapter`_, vamos dar uma olhada numa infraestrutura de cache do Django, que é uma maneira conveniente de melhorar a performace da sua aplicação.

.. _next chapter: ../chapter15/