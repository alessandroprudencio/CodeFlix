# CodeFlix

* [Sobre o projeto](#sobre-o-projeto)
* [Decisões de projeto e de arquitetura](#decisões-de-projeto-e-de-arquitetura)
* [Microsserviços](#microsserviços)

## Sobre o projeto

Front
* Uma espécie de netflix
* Assinatura do serviço pelo cliente
* Catálogo de vídeos para navegação
* Playback de vídeos
* Busca full text no catálogo (ElasticSearch)

Back
* Processamento e encoding dos vídeos
* Administração do catálogo de vídeos
* Administração do serviço de assinatura
* Autenticação (Keycloak)

## Decisões de projeto e de arquitetura
 
* Arquitetura baseada em microsserviços
* Tecnologia adequada para cada contexto (ex: Go para processar vídeos)
* Cada microsserviço terá seu próprio processo de CI/CD

* API Gateway
  * Acesso externo ao microsserviços através do Ingress do Kubernetes / Istio como API Gateway
  * Controle de tráfego
  * Rate limit
  * Politicas de Retry
 
* Service Discovery
  * O projeto utilizara o Kubernetes para orquestrar os containers, logo Service Discovery já faz parte do processo
 
* Escala horizontal
  * O processo de escala poderá ser configurado ao nível de microsserviço
  * Todos os microsserviços trabalharam de forma "Stateless"
  * Quando utilizado upload de qualquer categoria de asset, o mesmo será armazenado em Cloud Storage
  * O processo de escala se dará no aumento de PODs do Kubernetes
  * O processo de autoscaling também será utilizado, através de um recurso chamado HPA(Horizontal Pod Autoscaling)
  * Todos os logs gerados serão persistidos em sistemas externos como Prometheus e Elastichsearch

* Consistência eventual
  * Grande parte da comunicação entre os microsserviços será assíncrona
  * Cada microsserviço possuirá sua própria base de dados
  * Eventualmente os dados poderão ficar inconsistentes, desde que não haja prejuízo direto ao negócio 
 
* Mensageria
  * Como grande parte da comunicação entre os microsserviços é assíncrona, um sistema de mensagem é necessário
  * O RabbitMQ foi escolhido para esse caso

* Resiliência e Self Healing
  * Para garantir a Resiliência caso um ou mais microsserviços fiquem fora do ar, as filas serão essenciais
 
* Autenticação 
  * Serviço centralizado de identidade opensource: Keycloak
  * OpenID Connect
  * Tema customizado das telas de auth com react
  * Compartilhamento de chave pública com os serviços para verificação de autenticidade dos tokens
  * Diversas categorias de ACL

* Microsserviços
  * Admin Catálogo de Videos (Backend com laravel)
  * Admin Catálogo de Videos (Frontend com React)
  * Encoder e Video com Golang ou Node.js
  * API de Catalógo (Backend com Node.js)
