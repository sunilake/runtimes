---

copyright:
  years: 2015, 2017
lastupdated: "2017-06-01"

---

{:new_window: target="_blank"}
{:codeblock: .codeblock}


# Configurar serviços de limite
{: #auto_config}

É possível ligar vários serviços ao seu aplicativo Liberty. Os serviços podem ser gerenciados por contêiner, por aplicativo ou por ambos, dependendo do que o desenvolvedor
deseja.

Um serviço gerenciado por aplicativo é um serviço que é inteiramente gerenciado pelo aplicativo,
sem qualquer assistência do Liberty. O aplicativo geralmente lê VCAP_SERVICES para obter informações sobre o serviço ligado
e acessa o serviço diretamente. O aplicativo fornece todo o código necessário de acesso ao cliente. Não há nenhuma dependência dos recursos do Liberty ou da configuração do arquivo server.xml. A configuração automática do buildpack
do Liberty não se aplica aos serviços deste tipo.

Um serviço gerenciado por contêiner é um serviço que é gerenciado pelo tempo de execução do Liberty. Em alguns casos, o aplicativo pode consultar o serviço ligado em JNDI, enquanto em outros o serviço é usado diretamente pelo próprio  Liberty. O buildpack do Liberty lerá VCAP_SERVICES para obter informações sobre os serviços ligados. Para cada serviço gerenciado por contêiner, o buildpack executa três funções.

* Gera [variáveis de nuvem](optionsForPushing.html#accessing_info_of_bound_services) para o serviço limite.
* Instala os recursos do Liberty e o código de acesso ao cliente necessário para
acessar o serviço ligado.
* Gera ou atualiza sub-rotinas do arquivo server.xml que são requeridas pelo serviço.

Este processo é referido como uma configuração automática.
O buildpack Liberty fornece configuração automática para os tipos de serviço a seguir:

* [ClearDB MySQL Database ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](http://www.cleardb.com/developers)
* [MySQL](/docs/services/MySQL/index.html#MySQL)
* [ElephantSQL](docs/services/ElephantSQL/index.html)
* [PostgreSQL](/docs/services/PostgreSQL/index.html#PostgreSQL)
* [Cloudant NoSQL Database](/docs/services/Cloudant/index.html#Cloudant)
* [dashDB](/docs/services/dashDB/index.html#dashDB)
* [Data Cache](/docs/services/DataCache/index.html#data_cache)
* [Session Cache](/docs/services/SessionCache/index.html#session_cache)
* [MQ Light](/docs/services/MQLight/index.html#mqlight010)
* [Monitoring and Analytics](/docs/services/monana/index.html#gettingstartedtemplate)
* [Auto-Scaling](/docs/services/Auto-Scaling/index.html#autoscaling)
* [Single Sign On](/docs/services/SingleSignOn/index.html#sso_gettingstarted)
* [New Relic](newRelic.html)
* [Dynatrace](dynatrace.html)
* [Compose for PostgreSQL](/docs/services/ComposeForPostgreSQL/index.html)
* [Compose for MongoDB](/docs/services/ComposeForMongoDB/index.html) (Atualmente disponível apenas com o tempo de execução mensal do Liberty).

Conforme observado, alguns serviços podem ser gerenciados por aplicativo ou por contêiner. Mongo
e SQLDB são exemplos desses serviços. Por padrão, o buildpack do Liberty assume que esses serviços são gerenciados por
contêiner e os configura automaticamente. Se desejar que o aplicativo gerencie o serviço, será possível fazer opt-out da configuração automática para o serviço, configurando a variável de ambiente services_autoconfig_excludes. Para obter mais informações, veja [Fazendo opt-out da configuração automática de serviço](autoConfig.html#opting_out).

## Instalação do código de acesso do cliente e dos recursos do Liberty
{: #installation_of_liberty_features}

Ao ligar a um serviço gerenciado por contêiner, o serviço pode requerer que os recursos do Liberty sejam configurados na sub-rotina featureManager no arquivo server.xml. O buildpack do  Liberty atualiza a sub-rotina featureManager e instala os binários de apoio necessários. Se o serviço precisar de jars do driver cliente, os jars serão transferidos por download para um local bem conhecido
na instalação do Liberty.

Veja a seção [Fazendo opt-out da configuração automática de serviço](#opting_out) para obter mais detalhes sobre os tipos de serviço de limite.

## Gerando ou atualizando sub-rotinas de configuração do server.xml
{: #generating_or_updating_serverxml}

Ao enviar por push um aplicativo independente, o buildpack do Liberty gera a sub-rotina server.xml, conforme descrito em [Opções para enviar por push os aplicativos do Liberty](optionsForPushing.html#options_for_pushing), para Bluemix. Ao enviar por push um aplicativo independente e ligar aos serviços gerenciados por contêiner, o buildpack do Liberty gera as sub-rotinas necessárias do server.xml para os serviços ligados.

Ao fornecer um arquivo server.xml e ligar aos serviços gerenciados por contêiner, o buildpack do Liberty:

* gera a configuração para os serviços ligados, se o arquivo server.xml fornecido não contiver sub-rotinas de configuração para os serviços ligados.
* atualiza a configuração para os serviços ligados, se o arquivo server.xml fornecido contiver a sub-rotina de configuração para os serviços ligados.

Consulte a documentação para o tipo de serviço ligado para obter
detalhes adicionais.

## Executando opt-out da configuração automática de serviço
{: #opting_out}

Em alguns
casos, talvez você não queira que o buildpack do Liberty configure automaticamente os serviços que foram
ligados. Considere os cenários a seguir:

* Meu aplicativo usa MongoDB, mas desejo que o aplicativo gerencie diretamente a conexão ao banco de dados. O  aplicativo contém o jar do driver cliente necessário. Eu não deseja que o buildpack do Liberty configure
automaticamente o serviço Mongo.
* Estou fornecendo um arquivo server.xml e forneci as sub-rotinas de configuração para a instância SQLDB porque preciso de uma configuração de origem de dados não padrão. Eu não desejo que o buildpack do Liberty atualize meu arquivo server.xml, mas ainda preciso que o buildpack do Liberty assegure que o software de apoio apropriado esteja instalado.

Para executar opt-out da configuração automática de serviço, use a variável de ambiente
services_autoconfig_excludes. É possível incluir esta variável de ambiente em um
manifest.yml ou configurá-la usando o cliente cf.

É possível executar opt-out da configuração automática de serviços em uma base por tipo de serviço. É possível escolher fazer opt-out completamente (como no cenário Mongo) ou fazer opt-out somente de atualizações de configuração do arquivo server.xml (como no cenário SQLDB). O valor que você especifica para a variável de ambiente services_autoconfig_excludes é uma Sequência conforme mostrado a seguir.

* A Sequência pode conter especificações de opt-out para um ou mais serviços.
* A especificação de opt-out para um serviço específico é service_type=option, em que:
  * O service_type é o rótulo para o serviço conforme exibido em VCAP_SERVICES.
  * A opção é all ou config.
* Se a Sequência contiver uma especificação de opt-out para mais de um serviço,
as especificações individuais de opt-out deverão ser separadas por um único caractere
de espaço em branco.

Mais formalmente, a gramática da Sequência é conforme a seguir.

```
    Opt_out_string :: <service_type_specification[<delimiter>service_type_specification]*
    <service_type_specification> :: <service_type>=<option>
    <service_type> :: service type (service label as it appears in VCAP_SERVICES)
    <option> :: all | config
    <delimiter> :: one white space character
```
{: codeblock}

**Importante**: O tipo de serviço que você especifica deve corresponder ao rótulo de serviços como ele aparece na variável de ambiente VCAP_SERVICES. O espaço em branco não é permitido.
**Importante**: Nenhum espaço em branco é permitido em um ```<service_type_specification>```. O único uso permitido de espaço em branco é
separar diversas instâncias de ```<service_type_specification>```.

Use a opção "all" para fazer opt-out de todas as ações de configuração automática para um serviço, como no cenário Mongo acima. Use a opção "config" para fazer opt-out somente das ações de atualização da configuração como no cenário SQLDB acima.

Aqui estão especificações de opt-out de amostra em um arquivo manifest.yml para os cenários Mongo e SQLDB.

```
    env:
      services_autoconfig_excludes: mongodb-2.2=all

    env:
      services_autoconfig_excludes: sqldb=config

    env:
      services_autoconfig_excludes: sqldb=config mongodb-2.2=all
```
{: codeblock}

Aqui estão exemplos de como
configurar a variável de ambiente services_autoconfig_excludes para o aplicativo myapp usando a interface de linha de comandos.

```
    $ cf set-env myapp services_autoconfig_excludes sqldb=config
    $ cf set-env myapp services_autoconfig_excludes "sqldb=config mongodb-2.2=all"
```
{: codeblock}

## Substituindo a configuração de serviço
{: #override_service_config}

Em alguns casos, pode ser desejável substituir a configuração padrão para um serviço gerado pela
configuração automática.
Isso pode ser feito usando a variável de ambiente
**LBP_SERVICE_CONFIG_xxxx**, em que "xxxx" é o nome do serviço em letras maiúsculas.  Por exemplo, para substituir a versão padrão do serviço *mysql* e configurá-la como
a versão 1.4.+, emita um comando semelhante a:

```
    $ cf set-env myapp LBP_SERVICE_CONFIG_MYSQL "{driver: { version: 1.4.+ }}"
```
{: codeblock}

A tabela a seguir mostra a sintaxe para substituir algumas opções de configuração de serviço:

<table>
<tr>
<th align="left">Nome da Variável de Ambiente</th>
<th align="left">Sintaxe de configuração</th>
</tr>

<tr>
<td>LBP_SERVICE_CONFIG_MYSQL</td>
<td>"{driver: { version: x.y.z }, connection_pool_size: 15}"</td>
</tr>

<tr>
<td>LBP_SERVICE_CONFIG_POSTGRESQL</td>
<td>"{driver: { version: x.y.z }}"</td>
</tr>
</table>



# rellinks
{: #rellinks notoc}
## geral
{: #general notoc}
* [Tempo de execução do Liberty](index.html)
* [Visão geral do perfil Liberty](http://www-01.ibm.com/support/knowledgecenter/SSAW57_8.5.5/com.ibm.websphere.wlp.nd.doc/ae/cwlp_about.html)
