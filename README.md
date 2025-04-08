# Introdução à Segurança no Desenvolvimento Web
Neste breve artigo, vamos elaborar algumas vulnerabilidades e procedimentos comuns relacionados a segurança da informação em sistemas web.

Antes de abordarmos as vulnerabilidades e os procedimentos para mitigá-las, é essencial entender que um software pode conter inúmeras brechas de segurança, algumas são mais óbvias ou conhecidas, e podem ser amplamente exploradas, enquanto outras podem ser extremamente complexas e pouco viáveis para um atacante comum.

No nosso caso, vamos explorar algumas das mais comuns, para que possamos reconhecê-las e, mais importante, adotarmos práticas que as evitem desde o início. O objetivo é garantir que nossas aplicações sejam construídas com uma base mais segura, reduzindo a exposição a ataques e fortalecendo a confiabilidade do sistema.

## 1. XSS (Cross-Site Scripting)
O XSS (Cross-Site Scripting) é uma das vulnerabilidades mais comuns em aplicações web, que ocorre quando um atacante consegue injetar scripts em um sistema, de forma que ele seja executado pelo servidor, ou pelo navegador de outros usuários que acessam o sistema.

Esse tipo de ataque geralmente acontece por meio de entradas de dados não validadas, ou não tratadas adequadamente para serem utilizadas em contextos onde são interpretadas como código — como em trechos de HTML, comandos SQL, scripts JavaScript ou até comandos de terminal (Bash).

Quando o input do usuário é inserido diretamente nesses contextos sem passar por uma filtragem apropriada, o sistema pode acabar executando instruções maliciosas em vez de tratá-las como simples texto.

Por exemplo, imagine um campo de comentários em um site, onde o conteúdo enviado pelos usuários é exibido diretamente para os outros. Caso o desenvolvedor simplesmente renderize o texto dos comentários como se fosse código, por exemplo, através da atribuição da propriedade 'innerHTML' de um elemento HTML, sem remover os caracteres especiais, um atacante pode inserir código HTML e JavaScript no lugar de um comentário. Assim, quando outro usuário acessar a página, o script será executado em seu navegador, podendo executar qualquer instrução comumente acessível pelo próprio desenvolvedor do site (potencialmente efetuando ataques como roubo de cookies de sessão, redirecionamento para outros sites falsos, ou até execução de ações do site em nome do usuário).

Exemplo de código vulnerável:
```html
<div id="comentario"></div>

<script>
  const comentario = window.prompt("Faça um comentário");
  // um atacante poderia escrever código HTML/JavaScript, por exemplo:
    // <script>alert('XSS')</ script>

  // o qual seria executado ao ser renderizado na página.
  document.getElementById('comentario').innerHTML = comentario;
</script>
```

No geral, a injeção de código é uma das vulnerabilidades mais perigosas para uma aplicação. Além do exemplo de injeção no frontend que citamos, também são possíveis injeções de código SQL (que afetam o banco de dados), Bash (possibilitando o envio de instruções ao sistema operacional do servidor), e da própria linguagem de programação em uso no backend.

## 2. CSRF (Cross-Site Request Forgery):
Podemos encarar o CSRF basicamente como uma "falsificação de requisições", ele é um ataque que explora a confiança do sistema no navegador do usuário. O ataque ocorre quando um usuário autenticado em um site é induzido a executar ações indesejadas em um site, sem perceber, através de outra página maliciosa.

Esse tipo de ataque só é possível quando os cookies de autenticação são enviados pelo navegador nas requisições para o site original, mesmo que elas tenham se originado de outro domínio. Em muitos casos, é algo que o navegador faz por padrão, especialmente com cookies sem o atributo SameSite configurado corretamente.

Se outro site malicioso conseguir fazer o navegador enviar uma requisição ao site onde você está logado (ex: clicando em um link, carregando uma imagem ou formulário oculto), o cookie de sessão é enviado automaticamente junto, validando a requisição como se fosse sua.

O site atacado não tem como distinguir se a requisição foi iniciada pela interface legítima ou por outro site — a não ser que utilize tokens CSRF, verificação de origem ou cookies com SameSite.

Como mitigar ataques CSRF:
- Definir os cookies com SameSite=Strict ou SameSite=Lax, impedindo que sejam enviados em requisições de terceiros.

- Exigir tokens CSRF únicos por sessão ou requisição.

- Validar os cabeçalhos Origin ou Referer para verificar o domínio do qual a requisição foi enviada.

Existem casos onde essas táticas de mitigação não poderão ser empregadas, por exemplo, quando a própria funcionalidade de um sistema depende que requisições possam ser feitas a partir de outros sites. Um exemplo clássico são as redes sociais que permitem que usuários curtam conteúdos diretamente de sites externos, por meio de botões de "Like" embutidos em blogs, notícias, etc.

Nesses cenários, utilizar o atributo SameSite=Strict nos cookies impediria que o botão funcionasse fora do domínio da rede social, o que quebraria a funcionalidade desejada. Por isso, essas aplicações precisam lidar com a segurança de forma mais refinada — utilizando tokens, verificação de domínio e regras específicas por tipo de ação, para equilibrar a usabilidade e a proteção contra ataques CSRF.

## 3. Exposição de Informações Sensíveis:
A exposição de informações sensíveis é uma das falhas mais comuns em aplicações web. Essas informações, quando acessíveis por terceiros, podem permitir o controle de serviços internos, bancos de dados, contas de usuário e até de partes da infraestrutura de uma aplicação.

Um erro recorrente é deixar variáveis de ambiente contendo chaves secretas no código-fonte ou em repositórios públicos, como em arquivos .env publicados, ou hard code das credenciais direto no código.

Mesmo que por acidente, o upload dessas informações sensíveis para sistemas de versionamento pode comprometer seriamente a segurança de uma aplicação.

Outro problema crítico é armazenar senhas de usuários sem nenhum tipo de hash ou criptografia, salvando-as diretamente no banco de dados em texto puro (plaintext). Isso faz com que, em caso de vazamento, todas as contas fiquem imediatamente comprometidas.

Além disso, fazer requisições HTTP (sem o "S") em vez de HTTPS também representa risco, especialmente ao trafegar tokens ou credenciais. Isso permite que dados sejam interceptados facilmente por atacantes em redes Wi-Fi, através de técnicas como man-in-the-middle (MITM).

Como evitar o vazamento de informações sensíveis:
- Nunca suba arquivos .env ou similares para repositórios publicados.
  - Por exemplo, use .gitignore para garantir que esses arquivos não sejam inclusos.

- Nunca use credenciais hardcoded (diretamente no código fonte):
  - Não inclua chaves de API, tokens e senhas diretamente em arquivos de código fonte. Prefira sempre chamar essas informações através de variáveis de ambiente (.env).

- Use armazenamento seguro de senhas:
  - Sempre aplique hashing com algoritmos fortes como bcrypt ou argon2, com salt.

- Evite expor informações desnecessárias do backend:
  - Em vez de retornar todos os dados de um recurso (ex: todos os campos de um usuário), retorne apenas o que é necessário para a operação ou tela em questão. Isso reduz a superfície de exposição e evita o vazamento de dados sensíveis acidentalmente (além de aumentar a segurança, também pode melhorar a performance da aplicação).

- Sempre envie requisições por HTTPS:
  - Certifique-se de que todas as requisições de login e APIs utilizem HTTPS com certificados válidos.

- Configure os tokens e chaves de API com escopo e expiração.
  - Evite utilizar tokens com muitas permissões amplas (permissões desnecessárias para a operação ao qual ele foi convencionado) e sem tempo de expiração/quantidade de usos.

Esses pequenos descuidos podem levar a falhas catastróficas para um sistema. Portanto, proteger as credenciais e canais de comunicação é um dos pilares mais importantes da segurança desde as fases iniciais do desenvolvimento.

## 4. Upload Inseguro de Arquivos:
Quando permitimos o upload de arquivos em uma aplicação, podemos abrir portas para diversos tipos de ataques, caso não tenhamos o devido cuidado com validações.

Um dos erros mais comuns é confiar apenas na extensão do arquivo (ex: .jpg, .pdf, .png) para determinar sua segurança. Um atacante pode facilmente renomear um arquivo .php para .jpg, por exemplo, e tentar executá-lo no servidor caso ele seja salvo em uma pasta pública.

Além disso, arquivos muito grandes podem causar floods de memória ou armazenamento, e o envio de tipos inadequados pode resultar em execuções de código malicioso no servidor.

A regra de ouro aqui é: nunca confie em arquivos enviados por usuários, por mais inofensivos que pareçam. O upload seguro exige atenção redobrada em cada etapa — da validação à armazenagem — para evitar que uma funcionalidade simples se torne uma falha crítica.

Como ter uploads mais seguros:
- Valide a extensão e o MIME type do arquivo:
  - Não confie apenas na extensão. Verifique o MIME type (ex: image/jpeg, application/pdf) e analise o conteúdo do arquivo, se possível.

- Defina limites de tamanho de upload:
  - Impeça o envio de arquivos muito grandes para evitar sobrecarga do servidor e ataques de DoS (negação de serviço).

- Atribua nomes aleatórios ou hash ao salvar o arquivo:
  - Não use o nome original do arquivo enviado pelo usuário. Gere nomes únicos usando UUIDs, hashes ou timestamps, o que dificulta tentativas de acesso direto por URL e evita conflitos entre arquivos com nomes repetidos. Exemplo: em vez de salvar curriculo.pdf, salve como 3f2a9d7e-cv.pdf ou 1672449832981.pdf.

- Quando for adequado, armazene arquivos fora de rotas públicas:
  - Evite salvar arquivos que deveriam ser privados diretamente em rotas/diretórios públicos acessíveis sem autenticação.

- Desabilite a execução de arquivos no diretório de upload:
  - Configure o servidor (Apache, Nginx, etc.) para não executar nenhum tipo de script no diretório onde os arquivos são armazenados.

- Use listas de arquivos permitidos (whitelist):
  - Em vez de tentar bloquear extensões perigosas (blacklist), aceite apenas os formatos realmente adequados para aquela funcionalidade (ex: .jpg, .png, .pdf).

## 5. Rotas Inseguras:
Rotas mal protegidas são uma das portas de entrada mais comuns para ataques em endpoints de APIs e páginas web. A falta de uma configuração segura pode permitir desde acessos não autorizados até a exploração automatizada por bots, web scrapers e scripts maliciosos.

Problemas comuns:
- Falta de autenticação/autorização:
  - Não proteger rotas internas, administrativas ou de API. Sem efetuar algum tipo de autenticação, qualquer pessoa pode acessar/utilizar os recursos da API conhecendo apenas o endpoint.

- Ausência de rate limiting:
  - Sem limites de requisições por IP ou por token, endpoints públicos (como login, cadastro ou envio de e-mail) podem ser alvo fácil de brute force, spam e ataques de negação de serviço (DoS).

- Mensagens de erro muito detalhadas:
  - Retornar stack traces, mensagens de banco de dados ou códigos internos pode entregar informações sensíveis a um atacante, como estruturas de tabelas, nomes de funções e até trechos do código fonte.

- Má configuração de CORS (Cross-Origin Resource Sharing):
  - Permitir requisições de qualquer origem (Access-Control-Allow-Origin: *) sem restrição pode expor APIs a uso indevido por outros domínios maliciosos. Sempre que possível (e quando for adequado), é importante limitar as origens que de fato tem permissão, e limitar os métodos e headers permitidos.

Como garantir uma boa configuração de rota:
- Implemente autenticação e autorização adequadas quando necessário:
  - Certifique-se de que cada endpoint que tem a intenção de ser privado verifica quem está fazendo a requisição e se tem permissão para tal. Uma prática comum é usar middlewares de autenticação e níveis de acesso por cargo.

- Valide dados e permissões tanto no frontend quanto no backend:
  - O frontend pode impedir muitos erros e abusos, mas nunca deve ser o único filtro do sistema. As verificações de permissão, integridade de dados e formato devem sempre ser feitas também no backend. Isso garante que mesmo requisições manipuladas ou feitas via ferramentas como Postman ou scripts externos, passem pelos mesmos critérios de segurança.

- Use rate limiting:
  - Aplique limites de requisição por IP ou por token usando bibliotecas como express-rate-limit (Node.js), throttle (Laravel), entre outras. Isso ajuda a bloquear tentativas automatizadas e tentativas de exploração.

- Retorne mensagens de erro genéricas para os usuários finais:
  - Na produção, evite mostrar mensagens de erro detalhadas (exibindo código, stack trace, etc). Prefira mensagens como "Algo deu errado, tente novamente". Envie os logs detalhados diretamente para o servidor, mas não os exponha ao cliente.

- Não exponha endpoints privados publicamente:
  - Rotas de manutenção, depuração, backups, dashboards administrativos, etc. todas devem passar por autenticação.

- Configure o CORS com critérios adequados:
  - Permita apenas origens confiáveis, defina os métodos e headers necessários, e evite usar ('Access-Control-Allow-Credentials': '*') de forma inconsciente. Isso impede que domínios externos indesejados interajam com as ruas rotas de forma indevida.
