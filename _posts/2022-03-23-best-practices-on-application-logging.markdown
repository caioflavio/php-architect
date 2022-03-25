---
layout: post
title:  "Melhorando a observabilidade: Boas práticas na escrita de LOGS"
date:   2022-03-23 11:34:51 -0300
categories: logs event-sourcing good-pratices observability
---

## Introdução

Esse artigo tem como objetivo dismitificar o uso de logs nas nossa aplicações. E para nos guiarmos nesse processo tentaremos responder algumas perguntas como: **O que são logs? Para que servem? Como escrever logs relevantes?**. Esse é um tema pode parecer simples mas grande parte dos engenheiros de software apesar de conhecerem não são familiarizados com o conceito, segundo  [esse artigo]([www.linkedin.com](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)) da equipe de engenharia do linkedin e nos iremos mudar isso hoje!

![Pokemons rolling on a log](https://c.tenor.com/8SwMrENO6AEAAAAM/pikachu-togepi.gif)

## Definição
Uma possível definição técnica que eu particularmente gosto é de que talvez os logs sejam a forma mais simples possível de abstração de armazenamento, eles são sequências de registros ordedenados pela data de criação e possuem duas propriedades, todo arquivo de log deve ser [_append-only_](https://en.wikipedia.org/wiki/Append-only) e [_totally-ordered_](https://mathworld.wolfram.com/TotallyOrderedSet.html). De modo simples, _append-only_ significa que o arquivo de log suporta apenas inserção de dados e os dados já existentes devem ser imutáveis e _totally-ordered_ é uma propriedade que nos grante que existe uma ordem em todos elementos do conjunto (nesse caso nosso arquivo de log) e que pegando quaisquer dois elementos desse conjunto conseguimos dizer qual veio primeiro e qual veio por último.

![Ordenando itens](https://cdn.dribbble.com/users/6319642/screenshots/14533255/sorting.gif)

## Principais finalidades
Existem diversos tipos e finalidades de logs como os [transaction logs](https://www.ibm.com/docs/en/db2/11.5?topic=strategies-database-logging) que são utilizados por diversos bancos de dados para realizar um fallback dos dados em caso de falha de uma transação. Existem também os logs utlizados por [serviços de mensageria para criação de tópicos e mensagens](https://www.rapid7.com/blog/post/2018/01/16/taking-a-message-based-approach-to-logging/) apesar de ser bom saber que um mesmo conceito pode ser aplicado em situações diferentes nosso objetivo aqui é focar na utilização de logs para monitoramento e auxilio na solução de possíveis problemas na nossa aplicação e esses logs são popularmente conhecidos como [application logs](https://www.xplg.com/application-logs-what-how).

## Observabilidade
Os logs são comumente utilizados para auxiliar no diágnostico de problemas e no processo de observação do comportamento de uma aplicação, principalmente em sistemas de [arquitetura monolitica](https://pt.wikipedia.org/wiki/Aplica%C3%A7%C3%A3o_monol%C3%ADtica). Porém com a popularização da [arquitetura de microsserviços](https://www.redhat.com/pt-br/topics/microservices) apesar de eles continuarem sendo utilizados para esse fim eles passaram a ser considerados como parte de um conceito mais amplo a [observabilidade](https://aws.amazon.com/pt/products/management-and-governance/use-cases/monitoring-and-observability/). Os pilares da observabilidade segundo [este artigo](https://www.linkedin.com/pulse/import%C3%A2ncia-da-observabilidade-em-aplica%C3%A7%C3%B5es-modernas-felipe-neves/?originalSubdomain=pt) são as métricas, eventos, traces e **logs**. Neste texto iremos abordar boas práticas na escrita de logs focando no objetivo de melhorar a observabilidade de nossas aplicações e os outros pilares serão abordados em posts futuros.

![logs não são coisa do passado](https://reev.co/wp-content/uploads/2016/07/gifs-contam-uma-historia.gif)

## Como escrever LOGs relavantes?
Tendo em mente o conceito de observabilidade e como os logs funcionam reuni algumas dicas de como escrever logs para que eles nos forneçam informações detalhadas sobre o que acontece na nossa e de fato nos ajudar no processo de monitoramento e correção de problemas no fluxo da nossa aplicação.

### 1. Padronize a escrita de logs
Os [PSRs (PHP Stardard Recommendation)](https://www.php-fig.org/psr/) tem uma [seção especifica](https://www.php-fig.org/psr/psr-3/) para como devemos escrever nossas classes de logs. Mas nem sempre precisamos reinventar a roda, existem diversos pacotes mantidos pela comunidade e amplamente utilizados que podemos utilizar em nossos projetos sem nos precuparmos em implementar essas diretrizes, mas é sempre importante entender o que estamos fazendo.

#### 1.1. Pacotes populares para logs em php
1. [Monolog](https://github.com/Seldaek/monolog) - É uma das bibliotecas mais utilizadas no mercado e é utilizada nativamente pelos principais frameworks PHP da atualidade como: Laravel, Symphony, Lumen, CakePHP, Slim, entre outros...
2. [Analog](https://github.com/jbroadway/analog) - Apesar de não ser tão popular essa biblioteca tem a premissa de ser otimizada para criação de logs de forma simples, configurável e extensível.
3. [KLogger](https://github.com/katzgrau/KLogger) - Essa biblioteca também mantida pela comunidade é utilizada em algumas universades dos EUA e assim como as outras segue as especificações da PSR-3.  

### 2. Sempre use os níveis de log
Os [PSRs](https://www.php-fig.org/psr/psr-3/) especificam oito níveis de mensagens de log que não são arbitrários, eles seguem o protocolo [RFC-5424](https://datatracker.ietf.org/doc/html/rfc5424) que faz parte de um conjunto de regras definidas para padrozinar o que é implementado na internet. E por esse motivo temos oito níveis de logs, que são eles em ordem de criticidade:
1. **emergency**: Deve ser utilizado quando o sistema está completamente não utilizavel.
2. **alert**: Deve ser utilizado quando uma ação precisa ser tomada imediatamente.
3. **critical**: Deve ser utilizado quando algum componente está indisponível, normalmente quando é uma exceção não esperada.
4. **error**: Deve ser utilizado para erros de execução que normalmente não precisam de uma ação no momento em que ocorrem mas precisam ser monitorados.
5. **warning**: Deve ser utilizado para comportamentos anormais no sistema que não necessáriamente sejam erros, como por exemplo, uso de API's desafadas ou mal uso de uma API.
6. **notice**: Deve ser utilizado para eventos que ocorrem normalmente mas que são siginificantes para o monitoramento.
7. **info**: Deve ser utilizado para eventos que são interessantes de serem observados no fluxo da aplicação, como por exemplo: "Usuário {id} logou no MYSQL".
8. **debug**: Deve ser utilizado para salvar informação detalhada de depuração de erros e/ou um determinado fluxo do sistema.

### 3. Escreva mensagens descritivas e contextualizadas

Os logs precisam nos ajudar a entender o fluxo da nossa aplicação então messagens como os exemplos abaixo dificilmente irão nos ajudar nisso:
```php
  LoggerLibrary::error("Algo deu errado");
  LoggerLibrary::error("Banco de dados não esta funcionando");
```
As mensagens acima não nos dão um caminho de o que e nem onde está acontecendo o problema Um exemplo de mensagem que nos ajuda a entender problemas no fluxo precisa espeficar 
```php
  LoggerLibrary::error("[Atualização Usuário] Não foi foi possível atualizar dados do usuário de id: {$id}");
  LoggerLibrary::info("[App\Services\ExternalApiService] Protolo número {$protocolNumber} atualizado com sucesso.");
  LoggerLibrary::warning("[App\Services\ExternalApiService] Não foi possível atualizar o protolo número {$protocolNumber}: API Indisponível.");
```

### 4. Não logue apenas exceções
Caso haja a necessidade de criar logs para determinadas exceções evite logar somente a stacktrace pois ela normalmente devolve apenas quais linhas de código geraram aquela exceção e díficilmente tem os dados geraram aquele problema, por isso evite fazer como no exemplo abaixo:
```php
try {
  $data = ['some' => 'data'];
  $this->apiService->getUserData($data);
} catch (\AnErrorException $e) {
  LoggerLibrary::debug("Caiu em uma excessão.", ['exception' => $e])
}
```

Lembre-se que os logs são uma parte importante da observabilidade e é preciso que ele forneça um contexto que possa ajudar a quem está monitorando a situação a entender aquele determinado fluxo para sanar eventuais problemas.
```php
try {
  $data = ['some' => 'data'];
  $this->apiService->getUserData($data);
} catch (\AnErrorException $e) {
  LoggerLibrary::debug("[Serviço de API XPTO] Falha ao receber dados de cliente da API XPTO. Os seguintes dados foram utilizados.", ['exception' => $e, 'data' => $data])
}
```

### 5. Isso é realmente necessário?
Quando vamos escrever mensagens de log para nossa aplicação precisamos lembrar que estamos descrevendo um fluxo do nosso sistema que precisa ser monitorado. Então antes de sair criando mensagens de log para cada trecho do código precisamos compreender o contexto e se a criação dessas mensagens irá colaborar de maneira positiva para o monitoramento do funcionamento da nossa aplicação.

### 6. Padronize, mas tenha em mente que as coisas podem mudar
Sabemos que os logs podem ser feitos para serem lidos por humanos e por máquinas e além disso conforme a nossa aplicação evolui novas necessidades surgem e utilizar [textos planos](https://en.wikipedia.org/wiki/Plain_text) pode deixar de ser uma boa opção. Por isso é importante a outras maneiras de realizar o parsing de nossos logs como por exemplo em JSON. A biblioteca [Monolog](https://github.com/Seldaek/monolog/blob/main/src/Monolog/Formatter/JsonFormatter.php) possui um parser pronto pra você salvar seus logs em JSON e neste [link](https://www.loggly.com/use-cases/json-logging-best-practices/) você encontra algumas dicas de como criar bons logs em JSON.

## Conclusão
Os logs são um dos pilares da observabilidade e quanto mais entendemos da nossa aplicação mais fácil se torna escrever fluxos que nos ajudem a entender a saúde de nossos serviços. Essas dicas são um caminho para a escrita de logs que podem contribuir para melhorar a observabilidade dos nossos sistemas. 