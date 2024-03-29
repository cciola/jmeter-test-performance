## Automação de testes de performance com JMeter
Projeto de estudos para testes de performance/carga/stress com [JMeter](https://jmeter.apache.org/).

-----------------------------------------
### O que este script faz?
Este script efetua alguns testes de performance nas APIs de teste dos sites [ServeRest](https://front.serverest.dev/login) e [ViaCEP](https://viacep.com.br)

-----------------------------------------
### Arquitetura do projeto - Serverest.jmx

```
🧪 TestPlan
 └── 📊 View Results Tree
 └── 📊 Aggregate Report
       ├─ ⚙️ ServeRest (Thread Group)
           ├─ 🥍 Acesso (HTTP Request)
               └─ 🛠️ HTTP Header Manager
           ├─ 🥍 Criação de conta (HTTP Request)
               └─ 🛠️ HTTP Header Manager
           ├─ 🥍 Login (HTTP Request)
               └─ 🛠️ HTTP Header Manager
```

-----------------------------------------
### Instalação (Windows)
**JMeter**: acesse o [site do JMeter](https://jmeter.apache.org/), baixer o arquivo `.zip` binário na sessão "Binaries":

![image](https://user-images.githubusercontent.com/3456363/161556522-f16aa87d-0ab8-43c0-8a0d-8995ed9c4a44.png)

Descompacte em `C:\`, acessar `C:\apache-jmeter-5.3\apache-jmeter-5.3\bin` e execute **jmeter.bat**.

O JMeter é uma ferramenta desenvolvida em Java. Por esse motivo, é necessário que sua máquina tenha o **Java SE Development Kit 8 (JDK)** instalado. Baixe [neste link](https://www.oracle.com/java/technologies/downloads/).

**Plugins:** acesse o [site de plugins do JMeter](https://jmeter-plugins.org/),baixe o plugin desejado e coloque o arquivo `.jar` dentro da pasta `lib`, e também na pasta `lib > ext` do JMeter. Para que o novo plugin seja listado, feche o Jmeter e abra novamente (acesse no botão "cocar" no canto superior direito).

**Blazemeter:** instalar a extensão Blazementer no Chome, [acessar este site](https://auth.blazemeter.com/auth/realms/blazect/protocol/saml/clients/blazemeter) e se cadastrar/fazer login.

## Testes de performance
*“A eficiência da performance (ou simplesmente desempenho) é uma parte essencial de fornecer uma boa experiência aos usuários quando eles usam seus aplicativos em uma variedade de plataformas fixas e móveis.”* (CTFL-PT Performance Testing – 2018br)

É bom conhecer a arquitetura que estamos inseridos para sabermos onde interferir e onde pode ser o gargalo. Pode ser utilizado cache para melhora do tempo da página.

* O teste de **carga** avalia a estabilidade no sistema, quando é aplicado níveis crescentes de carga na aplicação.
* No teste de **estresse**, o número de usuários vai sendo aumentado gradativamente, não é tudo de uma vez.
* O teste de **pico** é mais utilizado quando há promoções repentinas, que geram picos de acesso repentinos (notificações push, e-mail marketing...).
* O teste de **resistência** avalia o uso prolongado do sistema. É utilizado para avaliar questões de uso de memória, disco. Executa com um percentual um pouco abaixo do previsto para o sistema, mas por um longo período (dias ou até semana), não é um teste tão comum.
* O teste de **escalabilidade** é executado junto com o teste de estresse, tem um aumento rápido da rampa, seu foco é observar as configurações de infra estrutura (percentual de uso da CPU).

## Cenário de teste
*botão direito em Test Plan > Add > Threads (Users) > Thread Group*

Dentro deste cenário vamos adicionar as requisições dos cenários a serem executados.
Aqui também temos condições de controle para a execução destas requests.

- **Number of Threads (users)**: é o número de VUs que você quer no teste.
- **Ramp-up period (seconds)**: é o período em segundos da rampa de usuários vai subindo. A rampa é o número de usuários que vão sendo inseridos no sistema ao longo do tempo. Exemplo: se tivermos 100 usuários e 10 segundos, a cada 5 segundos a rampa vai subir 50 usuários.
- **Loop Count**: quando acaba uma thread e sobe outra, se quero que minhas threads sejam executadas de forma infinita, marco a opção "Infinite" e deixa o campo em branco. A partir do momento que quero que entra uma thread e ela caia, eu deixo 1 no campo.
- **Same user on each interation**: indica se deseja utilizar o mesmo usuário em cada iteração.
- **Delay Thread creation until needed**: define um delay entre cada iteração.
- **Specify Thread lifetime**: especifica o tempo de vida de cada thread, ou seja, o tempo de execução/duração de cada cenário.

É possível selecionar opções no Thread Group para o que fazer quando ocorrer um erro:

*Action to be taken after a Sampler error: Continue, Start Next Thread Group, Stop Thread, Stop Test, Stop Test Now*

## Requisições
*botão direito em Thread Group > Add*

Opções:
- **Pre Processors**: é o que será executado antes da request. (ex. quando temos algo que precisa ser executado antes de a requisição ser chamada).
- **Post Processors**: é o que será executado após da request. (ex. quando precisamos executar uma request utilizando dados obtidos de uma outra request que foi executada antes).
- **Assertions**: são as asserções que queremos fazer na execução das requisições.
- **Timer**: permite adicionar um tempo específico de espera no teste.
- **Test Fragment**: seria para conseguirmos modularizar nossas requests. (ex. uma requisição de login será chamada por N usuários. Se esta request de login for criada de forma fragmentada, caso houver uma alteração no login, não será necessário alterar cenário por cenário; basta alterar no fragmento, e a alteração já será refletida para todos os cenários.
- **Config Element**: apresenta opções de limpeza de cache, cookies, dentre outros.

Vamos adicionar os elementos para limpeza de coockie e cache, "HTTP Cookie Manager" e "HTTP Cache manager".
No "HTTP Cache manager" temos um número máximo de elementos em cache (5000). Quando este número é atingido, o cache é limpo.

- **User Defined Variables**: aqui temos a opção de definir variáveis.
**Observação**: para que esta opção seja acessível em todos os cenários, ela precisa estar aninhado no último Thread Group (de baixo para cima).

## Opção de gravação do script de teste
*botão direto em Test Plan > Add > Non-Test Elements > HTTP(S) Test Script Recorder*

Esta opção permite gravar uma navegação no browser (geralmente Firefox), a partir da porta informada. No exemplo temos a porta `8888`. Basta configurar esta mesma porta nas configurações do browser, e após dar um start, o JMeter já grava a navegação.

## Visualizar os resultados da execução
*botão direto em Thread Group > Add > Listener > View Results Tree*

*botão direto em Thread Group > Add > Listener > Aggregate Report*

No gráfico agregado temos:

- **Label**: nome da requisição
- **Samples**: quantidade de vezes que a requisição foi executada
- **Average**: média (máximo e mínimo dividido ao meio)
- **Median**: a mediana (menor e maior de um conjunto)
- **90% Line**: significa que 90% dos meus casos executados estão nesta faixa de tempo.
- **95% Line**: significa que 95% dos meus casos executados estão nesta faixa de tempo.
- **99% Line**: significa que 99% dos meus casos executados estão nesta faixa de tempo.
- **Min**: mí
- **Maximum**:
- **Error %**: percentual de erros
- **Throughput**: vazão
- **Received kB/sec**: quantidade de kB recebidos por segundo
- **Sent kB/sec**: quantidade de kB enviados por segundo

## Aumentando a memória para execução do teste
Ao clicar com o botão direito sobre o `jmeter.bat` e editando o código, é possível aumentar o valor de memória da nossa máquina na linha `set HEAP=-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m`:
```
if not defined HEAP (
    rem See the unix startup file for the rationale of the following parameters,
    rem including some tuning recommendations
    set HEAP=-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m
)
```

## Testando o ServRest
1) No *Test Plan*, adicione:
```
BASE_URL_1 = front.serverest.dev
BASE_URL_2 = serverest.dev
```

2) Acesse o [ServeRest](https://front.serverest.dev/login)

3) Crie uma conta com perfil Admin.

4) Faça logoff, tecle `F12`, faça login

5) Na aba `Network`, acesse a request `usuarios`

![image](https://user-images.githubusercontent.com/3456363/161571254-66ca81ad-a0fc-4c81-ad4f-5ad8a9c6b08a.png)

6) Adicione em *TestPlan > Thread Group*, desmarcando a opção **Specify Thread lifetime**

7) Adicione em *Thread Group > Add > Samples > HTTP Request*, informando:
```
Name = Acesso
Protocolo = https
Server Name or IP = ${BASE_URL_2}
HTTP Request = OPTIONS
Path = usuarios
```

8) Adicione *Acesso > Add > Config Element > HTTP Header Manager*

9) Na página do ServRest, copie e cole os valores do bloco **Response Headers** do modo desenvolvedor da página, e cole na opção:

![image](https://user-images.githubusercontent.com/3456363/161592981-4378014d-04cd-4bbb-ae1f-61cdd543e53e.png)

É importante informar os valores do Header, pois o script pode não entender que estamos requisitando de uma página web, podendo não executar corretamente.

10) Em **Acesso**, marque a opção `Redirect Automatically` quando há dados de post para se seguir no próximo request.

11) Adicione *Thread Group > Add > Listener > View Results Tree*

12) Clique no play para executar o script.

Geralmente costuma-se verificar estas informações no resultado:

![image](https://user-images.githubusercontent.com/3456363/161598477-512ed833-6c47-4e8a-986a-2e6c31e5b8cb.png)

Podemos aninhar o *View Results Tree* no *Thread Group*, assim os testes ficam agrupados todos abaixo.

13) Vamos duplicar o *Acesso* renomeando para *Criação de conta*, alterar o argumento para `POST`, copiar os dados do Payload da request de *usuarios* e colar na aba *BodyData*:

![image](https://user-images.githubusercontent.com/3456363/161600195-9346b85a-e1b1-4220-a8e2-cdd75214321d.png)

![image](https://user-images.githubusercontent.com/3456363/161600676-7f8d2c98-39ab-47d2-930e-2295050b59e0.png)

Devemos também copiar o header da página, assim como fizemos para *Acesso*.

14) Criaremos outro teste, *Login*, informando o *Path* `login`, copiando o Payload e o header correspondentes da página web.

![image](https://user-images.githubusercontent.com/3456363/161607006-48270209-34f0-4437-a51c-3cbf73ee5ed7.png)

15) Ao executar, os testes passarão.

![image](https://user-images.githubusercontent.com/3456363/161607068-4e9d5f1f-5428-4137-a3af-9951c45277ff.png)

## Testando o ViaCEP

### Arquitetura do projeto - ViaCEP.jmx

```
🧪 TestPlan
 └── 📊 View Results Tree
 └── 📊 Aggregate Report
       ├─ ⚙️ ViaCEP (Thread Group)
           └─ 🥍 Consulta de CEP (HTTP Request)
```

-----------------------------------------

1) Crie um novo teste (Thread Group), informando *Number of Threads* = 20, e um *Ramp-up* = 10.

2) Adicione *Add > Sampler > HTTP Request*

3) Acesse o [ViaCEP](https://viacep.com.br/), copie o endpoint informado na página e preencha os campos:
```
Protocol = https
Server Name or IP = viacep.com.br
HTTP Request = GET
Path = ws/01001000/json/
```
![image](https://user-images.githubusercontent.com/3456363/161609883-8644d6cd-3060-4f7c-b609-3996179bccef.png)

4) Reposicione o *View Results Tree* abaixo de *Test Plan*, para que englobe o novo teste.

5) Adicione um listener em *Thread Group > Add > Listener > Aggregate Report*

6) Execute o teste.

![image](https://user-images.githubusercontent.com/3456363/162188819-9eb0337d-ddc8-4eca-8ffc-86ced7ad3b3c.png)



## Cálculo de valores para o JMeter

Os valores podem ser obtidos de duas formas:

1) Calculando a média dos acessos nos dias desejados, e simular para 1 hora de execução

2) Calculando a simulação do pico em 1h:

**Exemplos:**
```
(número máximo de requisições / 24h) / 60min = requisições por minuto (throughput)
8849 / 24 = 368 requisições/hora
368 / 60 = 6,14 requisições/minuto
```

```
600s = 10min

1 usuário virtual (VU) consegue fazer 3,72 requisições em 10min

100 usuários fazem 3,72 requisições, em 1h (3600 segundos)
100 * 3,72 * 3600 = hits/tempo de 1339200 hits/tempo
```

**Observação**: todos os valores tem sempre que estar na mesma base decimal para o cálculo dar certo.

## Blazemeter
Efetue login no plugin do Blazementer, pois somente é possível salvar o script logado. 

Após o login, o Blazemeter exibe o usuário na extensão (canto superior direito).

1) Informe o nome do teste e clique no botão para gravar.
2) Acesse um site e aguarde o Blazemeter abrir o modal.
3) Informe o nome do passo do teste **antes** de executar uma ação ou acessar alguma página, para sabermos depois em qual página estamos.
4) A cada página acessada/passo efetuado, vá informano o nome do passo no Blazemeter.

Após navegar para fazer o teste, dar um stop na gravação. Clique no botão "Save..." e selecione a opção `JMeter (JMX)`.

O Blazemeter lista todos os endpoints que conseguiu capturar. Selecione os endpoints desejados e clique em "Save (1)". O Blazemeter vai exportar o arquivo com a gravação do teste, para a pasta Downloads.

Abra no JMeter (File > Open) o arquivo *.jmx* gerado no Blazemeter, para ver o que foi gerado. É importante sempre informar os labels dos steps durante a gravação do teste, para nos localizarmos melhor quando abrir a gravação no JMeter.

## Importação de dados utilizando planilha CSV
Vamos consumir o valor do CEP de um arquivo .csv que contém uma massa de teste.

Inclua o item *CSV Data Set Config* clicando com o botão direito no teste (ADD > Config Element > CSV Data Set Config), e ordene-o para ficar como último item do teste.

No campo `Filename`, vamos selecionar um arquivo .csv que contém a massa de dados pronta, lembrando que é necessário remover todo o caminho do arquivo, deixando apenas o nome do arquivo+extensão. Assim, o arquivo pode ser executado em qualquer lugar que tiver o script e o arquivo .csv, sem a necessidade de informar o caminho específico.

**Observação:** importante que, antes do teste, o arquivo .csv seja salvo com as células dos dados do CEP com o tipo **Texto**, para poder interpretar os zeros corretamente no JMeter.

![image](https://user-images.githubusercontent.com/3456363/162196476-7e10639e-9cb6-48bf-a44a-8e6ac6cb424e.png)

Onde: prefixo,sufixo (00000,000)

Configurar conforme abaixo:

- **Filename**: deixar apenas o nome do arquivo, apagar o caminho
- **Variable Names (comma-delimited)**: prefixo,sufixo
- **Ignore first line (only used if Variable Names is not empty)**: False
- **Delimiter (use '\t' for tab)**: ,
- **Allow quoted data?**: False (apenas utilizado como true se dentro de algum dos campos/variáveis há algum caractere especial; é bem difícil setar como true)
- **Recycle on EOF?**: True (quando chegar no fim do arquivo, volta para a primeira linha e continua reutilizando a massa. Se tivermos problema em reutilizar a massa, aí marcamos como true). Exemplo: se temos uma massa de apenas 50 CEPs e queremos fazer um teste com 100 VUs, podemos marcar esta opção para reutilizar os 50 itens novamente.
- **Stop thread on EOF?**: False (não preciso que pare quando chegar ao final do arquivo)
- **Sharing mode**: All threads (disponibilidade da massa para todas as threads)

Indique no endpoint da chamada do CEP, os parâmetros `${sufixo}` e `${prefixo}`, em substituição aos valores fixos, para obter os valores do arquivo .CSV.

Vamos configurar para agrupar as execuções, marcando a opção *Generate parent sample* para as threads, para não ficar listando linha a linha em cada execução.

## Links úteis para testar performance

- https://reqres.in/
- https://epic.gsfc.nasa.gov/about/api
- https://swapi.dev/
- https://explore.postman.com/
- https://developer.marvel.com/
- https://pokeapi.co/
- https://rapidapi.com/
