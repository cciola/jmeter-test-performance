## Automação de testes de performance com JMeter
Projeto de estudos para testes de performance/carga/stress com [JMeter](https://jmeter.apache.org/).

-----------------------------------------
### O que este script faz?
Este script efetua alguns testes de performance nas APIs de teste do site do [ServeRest](https://front.serverest.dev/login).

-----------------------------------------
### Ferramentas e uso da arquitetura
- [JMeter](https://jmeter.apache.org/)
- [Plugins do JMeter](https://jmeter-plugins.org/)

-----------------------------------------
### Arquitetura do projeto

```
⌛ TestPlan
 └── ⚙️ ThreadGroup
      ├─ 🔧 HTTP Cookie Manager
      ├─ 🔧 HTTP Cache Manager
      ├─ 📊 View Results Tree
      └──📊 Aggregate Report
  ├─ ⚙️ ThreadGroup
      └── 🔧 User Defined Variables
  └──🏢 HTTP(S) Test Script Recorder
```

-----------------------------------------
### Instalação (Windows)
**JMeter**: Acessar o [site](https://jmeter.apache.org/), baixar o arquivo `.zip` binário na sessão "Binaries":

![image](https://user-images.githubusercontent.com/3456363/161556522-f16aa87d-0ab8-43c0-8a0d-8995ed9c4a44.png)

Descompactar em `C:\`, acessar `C:\apache-jmeter-5.3\apache-jmeter-5.3\lib` e executar **jmeter.bat**:

**Plugins:** baixe o plugin desejado e coloque o arquivo `.jar` dentro da pasta `lib`, e também na pasta `lib > ext` do JMeter. Feche o Jmeter e abra novamente, para listar o plugin (acesse no botão "cocar" no canto superior direito).

## Cenário de teste
*botão direito em Test Plan > Add > Threads (Users) > Thread Group*

Dentro deste cenário vamos adicionar as requiições dos cenários a serem executados.
Aqui também temos condições de controle para a execução destas requests.

- **Number of Threads (users)**: é o número de VUs que você quer no teste.
- **Ramp-up period (seconds)**: é o período em segundos da rampa de usuários vai subindo. Exemplo: se tivermos 100 usuários e 10 segndos, a cada 5 segundos a rampa vai subir 50 usuários.
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
- **Test Fragment**: seria para conseguirmos modularizar nossas requests. (ex. uma requisição de login será chamada por N usuários. Se esta request de login for criada de forma fragmentada, caso houver uma alteração no login, não será necessáio alterar cenário por cenário; basta alterar no fragmento, e a alteração já será refletida para todos os cenários.
- **Config Element**: apresenta opções de limpeza de cache, cookies, dentre outros.

Vamos adicionar os elementos para limpeza de coockie e cache, "HTTP Cookie Manager" e "HTTP Cache manager".
No "HTTP Cache manager" temos um número máximo de elementos em cache (5000). Quando este número é atingido, o chache é limpo.

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

![image](https://user-images.githubusercontent.com/3456363/161556128-13025c11-1d4c-4d8e-811e-dc65d7dbd236.png)

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
No Test Plan, adicione:

- `BASE_URL_1` *front.serverest.dev*
- `BASE_URL_2` *serverest.dev*

Acesse o [ServeRest](https://front.serverest.dev/login).
Crie uma conta com perfil Admin
Faça logoff
Acesse novamente a página de login
Tecle `F12`, faça login
Acesse a request `usuarios` (aba Network)

![image](https://user-images.githubusercontent.com/3456363/161571254-66ca81ad-a0fc-4c81-ad4f-5ad8a9c6b08a.png)

Vamos adicionar no nosso *Thread Group > Add > Samples > HTTP Request*, informando:
- Name `Acesso`
- Protocolo `https`
- Server Name or IP `${BASE_URL_2}`
- HTTP Request `OPTIONS`
- Path `usuarios`

Vamos adicionar em *Acesso > Add > Config Element > HTTP Header Manager*


- 
