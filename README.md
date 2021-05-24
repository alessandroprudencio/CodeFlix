# CodeFlix

* [Sobre o projeto](#sobre-o-projeto)
* [Decisões de projeto e de arquitetura](#decisões-de-projeto-e-de-arquitetura)
* [Microsserviços](#microsserviços)
* [Testes](#testes)
* [CI/CD](#cicd)
* [Kubernetes](#kubernetes)
* [Service Mesh](#service-mesh)
* [Cloud Providers](#cloud-providers)
* [Observabilidade](#observabilidade)
* [Glossário](#glossário)

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
   * Tema customizado das telas de auth com React ou Vuejs
   * Compartilhamento de chave pública com os serviços para verificação de autenticidade dos tokens
   * Diversas categorias de ACL

## Microsserviços
 * Admin Catálogo de Vídeos (Backend com laravel)
 * Admin Catálogo de Vídeos (Frontend com React ou Vuejs)
 * Encoder e Video com Golang ou Node.js
 * API do Catálogo (Backend com Node.js)
 * Aplicação do Catálogo (Frontend com React.js ou Vuejs)
 * Assinatura do CodeFlix pelo cliente (Python com Django)

## Testes
 * Unidade
 * Integração
 * End-to-end
 * Selenium / frontend
 * Upload

## CI/CD
 * Para cada pull request gerada em uma aplicação, iniciaremos o processo  de CI
 * GitHub Actions
 * O Processo de CI será capaz de:
   * Subir a aplicação usando Docker 
   * Executar os testes
   * Utilizaremos o Sonarqube
 * No caso de acontecer o "merge" do Pull Request no "main" , o processo de CD acontece :
   * Fara a geração da imagem Docker
   * Realizara o upload da imagem em um container registry
   * Executará o deploy no Kubernetes

## Kubernetes
 * Cluster Kubernetes gerenciado
 * Deploy da aplicação
 * Startup, Readiness Liveness Probe para self healing
 * Horizontal Pod Autoscaler (HPA) para escalar horizontalmente a aplicação
 * TLS/SSl com cert-manager
 * Kubelens para ter uma visão geral do cluster

## Service Mesh
 * Istio
 *  Gerenciamento de  tráfego
    * Virtual Service
    * Ingress Gateway
    * Envoy Sidecar Proxy
    * Circuit Breaking
    * Requests Timeout
 * TLS/SSL com cert-manager

## Cloud Providers
 Utilizaremos os recursos necessários para podermos realizar o deploy das aplicações
 * AWS
 * GCP
 * Azure

## Observabilidade
 * Métricas
   * Prometheus/Grafana 
   * Elastic Stack
 * Logs
   * Elastic Stack
     * Elastic Search
     * Beats
     * Kibana
 * Tracing / APM
   * Elastic Stack
     * Elastic Search
     * Beats
     * Kibana
     * APM
 * Kiali

## Glossário
 * Catalog = Catálogo de vídeos
 * Category = Define o tipo de categorização de um video (ex: filme, documentário, serie,  infantil)
 * Genre = Define o gênero do video (ex: terror, comédia, ação)
 * Cast members = atores, diretores e outras pessoas responsáveis pela criação do video
 * Featured video = Video em destaque na plataforma
 * My Videos = Lista de vídeos favoritos do usuário
 * Subscription = Assinatura que o cliente realiza no sistema para acessar a plataforma
 * Plan = Plano que o usuário poderá contratar em uma assinatura para acessar a plataforma
 * Client = Cliente que contrata o acesso na plataforma de vídeos
 * User = Qualquer pessoa que tenha acesso ao sistema, independente de permissão ou função
