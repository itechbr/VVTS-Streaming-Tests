# Laboratório 02 — Estratégias e Níveis de Teste na Prática
**Domínio Escolhido:** Plataforma de Streaming de Vídeo sob Demanda (StreamFlix)  
**Disciplina:** Verificação, Validação e Testes de Software  
**Autor:** Ícaro Pontes  

---

## 🏗️ 0. Arquitetura Base do Sistema (StreamFlix)

Para garantir consistência técnica e rastreabilidade nos 13 cenários de teste, este relatório utiliza uma arquitetura padronizada composta pelos seguintes módulos e interfaces:

* **Cliente & Aplicação:** `VideoPlayerClient` (Player no app mobile/web).
* **Serviços de Domínio (Backend):**
  * `DRMTokenValidator` (Validação de tokens de direitos digitais).
  * `StreamingSessionService` (Gerenciamento de sessões ativas de reprodução).
  * `VideoCatalogService` (Catálogo e metadados de títulos).
  * `AuthService` (Autenticação e controle de acesso).
  * `VideoRepository` (Camada de persistência de dados de vídeo).
* **Interfaces & Gateway Externos:**
  * `ICDNProvider` (Interface para distribuidoras de conteúdo / CDN).
  * `IPaymentGateway` (Interface de processamento de assinaturas).
* **Infraestrutura:** `UserDatabase` (Banco de dados de usuários), `MediaStorageS3` (Armazenamento de mídia bruta).

<img width="1591" height="828" alt="image" src="https://github.com/user-attachments/assets/9af1db2f-ff92-4f2b-900d-e25020815a65" />


---

## Sumário de Testes

* [1. Teste de Unidade (Unit Testing)](#1-teste-de-unidade-unit-testing)
  * [1.1 Verificação de Lógica Atômica em Componente Isolado](#11-verificação-de-lógica-atômica-em-componente-isolado)
* [2. Teste de Integração (Integration Testing)](#2-teste-de-integração-integration-testing)
  * [2.1 Integração Não Incremental (Big Bang)](#21-integração-não-incremental-big-bang)
  * [2.2 Integração Incremental Top-Down (Descendente) com Uso de Stubs](#22-integração-incremental-top-down-descendente-com-uso-de-stubs)
  * [2.3 Integração Incremental Bottom-Up (Ascendente) com Uso de Drivers](#23-integração-incremental-bottom-up-ascendente-com-uso-de-drivers)
  * [2.4 Teste de Fumaça (Smoke Testing)](#24-teste-de-fumaça-smoke-testing)
  * [2.5 Teste de Regressão](#25-teste-de-regressão)
* [3. Teste de Validação (Validation Testing)](#3-teste-de-validação-validation-testing)
  * [3.1 Teste Baseado em Casos de Uso (Use Case Testing)](#31-teste-baseado-em-casos-de-uso-use-case-testing)
  * [3.2 Teste Transversal de Desempenho (Performance Testing)](#32-teste-transversal-de-desempenho-performance-testing)
  * [3.3 Teste Transversal de Segurança (Security Testing)](#33-teste-transversal-de-segurança-security-testing)
* [4. Teste de Sistema e Aceitação (System & Acceptance Testing)](#4-teste-de-sistema-e-aceitação-system--acceptance-testing)
  * [4.1 Teste de Sistema Ponta a Ponta (End-to-End - E2E)](#41-teste-de-sistema-ponta-a-ponta-end-to-end---e2e)
  * [4.2 Teste Alpha (Alpha Testing)](#42-teste-alpha-alpha-testing)
  * [4.3 Teste Beta (Beta Testing)](#43-teste-beta-beta-testing)
  * [4.4 Teste de Aceitação do Usuário (User Acceptance Testing - UAT)](#44-teste-de-aceitação-do-usuário-user-acceptance-testing---uat)

---

## 1. Teste de Unidade (Unit Testing)

### 1.1 Verificação de Lógica Atômica em Componente Isolado

#### Diagrama de Classes UML

<pre class="mermaid">
classDiagram
    class DRMTokenValidator {
        -String secretKey
        -Long maxTokenAgeMs
        +DRMTokenValidator(secretKey: String, maxTokenAgeMs: Long)
        +validateToken(token: String, currentTimestamp: Long) Boolean
        -isSignatureValid(token: String) Boolean
        -isExpired(issuedAt: Long, currentTimestamp: Long) Boolean
    }

    class TokenValidationResult {
        <<enumeration>>
        VALID
        EXPIRED
        INVALID_SIGNATURE
        MALFORMED
    }

    DRMTokenValidator ..> TokenValidationResult : retorna status interno
</pre>
<img width="1725" height="942" alt="image" src="https://github.com/user-attachments/assets/d974d4e7-478c-4bad-9efc-1ea4a8a31214" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Classe:** `DRMTokenValidator`
* **Responsabilidade:** Validar atomicamente a autenticidade e a validade temporal de tokens de autorização de mídia (DRM) antes de permitir a liberação do fluxo de vídeo.

##### Operação do Teste
* **Método Testado:** `validateToken(token, currentTimestamp)` (método público)
* **Modo de Execução:** Teste unitário exercitado de forma 100% pura na memória.
* **Isolamento:** Sem dependências externas como banco de dados, rede ou servidores externos de licença.

##### Detecção de Defeitos
O teste visa identificar:
1. **Falhas na lógica temporal:** Erros matemáticos/booleanos no cálculo de expiração (`isExpired`).
2. **Falhas de autenticidade:** Erros na verificação de hash HMAC da assinatura (`isSignatureValid`).
3. **Tratamento de exceções:** Respostas incorretas ao receber strings nulas ou malformatadas.

---

## 2. Teste de Integração (Integration Testing)

### 2.1 Integração Não Incremental (Big Bang)

#### Diagrama de Componentes UML

<pre class="mermaid">
graph TD
    subgraph "Suíte de Testes Big Bang"
        TestRunner[TestRunner / SuiteIntegrationTest]
    end

    subgraph "Módulos Integrados Simultaneamente"
        Auth[AuthService]
        Catalog[VideoCatalogService]
        Session[StreamingSessionService]
        CDN[AkamaiCDNProvider]
    end

    TestRunner --> Auth
    TestRunner --> Catalog
    TestRunner --> Session
    TestRunner --> CDN

    Session --> Auth : valida sessão
    Session --> Catalog : busca URL do vídeo
    Session --> CDN : requisita manifesto HLS
</pre>
<img width="1181" height="715" alt="image" src="https://github.com/user-attachments/assets/338f5c99-3af3-4870-a963-1e33d83c6e0f" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Módulos Integrados:** `AuthService`, `VideoCatalogService`, `StreamingSessionService` e `AkamaiCDNProvider`.
* **Responsabilidade:** Validar a interoperabilidade global imediata entre os serviços de autenticação, catálogo, sessão e entrega de conteúdo.

##### Operação do Teste
* **Executor:** `TestRunner / SuiteIntegrationTest`.
* **Modo de Execução:** Abordagem Não Incremental (Big Bang) exercitada com todos os módulos acoplados e executados simultaneamente.
* **Isolamento:** Nenhum. Não há uso de Stubs ou Drivers; todas as dependências reais operam em conjunto.

##### Detecção de Defeitos
O teste visa identificar:
1. **Incompatibilidades de Interface:** Erros em contratos de API ou comunicação entre módulos de diferentes camadas.
2. **Falhas de Protocolo de Comunicação:** Falhas ao repassar tokens e manifestos HLS entre `StreamingSessionService` e `AkamaiCDNProvider`.
3. **Complexidade de Diagnóstico:** Evidencia a dificuldade de isolar a causa raiz quando um erro ocorre, dado que múltiplos módulos interagem ao mesmo tempo.

---

### 2.2 Integração Incremental Top-Down (Descendente) com Uso de Stubs

#### Diagrama de Classes UML

<pre class="mermaid">
classDiagram
    class StreamingSessionService {
        -ICDNProvider cdnProvider
        +iniciarSessao(userId: String, contentId: String) SessionResponse
    }

    class ICDNProvider {
        <<interface>>
        +obterURLManifesto(contentId: String) String
        +verificarDisponibilidadeChunk(chunkId: String) Boolean
    }

    class CDNProviderStub {
        <<service>>
        -String urlFicticia
        -Boolean simularInstabilidade
        +obterURLManifesto(contentId: String) String
        +verificarDisponibilidadeChunk(chunkId: String) Boolean
        +configurarFalhaSimulada(status: Boolean) void
    }

    StreamingSessionService --> ICDNProvider : utiliza
    CDNProviderStub ..|> ICDNProvider : implementa
</pre>
<img width="1795" height="857" alt="image" src="https://github.com/user-attachments/assets/cfb9e2aa-e7c1-4597-b150-2316512e60ac" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Classe sob Teste (Alto Nível):** `StreamingSessionService`
* **Substituto (Stub):** `CDNProviderStub` (implementa a interface `ICDNProvider`)
* **Responsabilidade:** Validar a regra de negócio do gerenciador de sessões de streaming de forma descendente, sem depender do servidor de CDN real.

##### Operação do Teste
* **Método Testado:** `iniciarSessao(userId, contentId)`
* **Modo de Execução:** Teste incremental Top-Down com injeção de dependência do `CDNProviderStub`.
* **Isolamento:** A CDN real é substituída pelo Stub, que retorna respostas pré-programadas em memória (URLs simuladas ou gatilhos de falha via `configurarFalhaSimulada`).

##### Detecção de Defeitos
O teste visa identificar:
1. **Falhas no Fluxo de Controle:** Tratamento incorreto no `StreamingSessionService` quando a CDN falha ou retorna indisponibilidade de mídia.
2. **Violações de Contrato:** Desconformidades entre a interface `ICDNProvider` e o consumo feito pela classe superior.
3. **Erros de Regra de Negócio:** Liberação indevida de sessão sem a obtenção válida do manifesto HLS.

---

### 2.3 Integração Incremental Bottom-Up (Ascendente) com Uso de Drivers

#### Diagrama de Classes UML

<pre class="mermaid">
classDiagram
    class VideoRepositoryDriver {
        -VideoRepository repository
        +testarPersistenciaEBusca() void
        +testarTratamentoDeErroChaveDuplicada() void
    }

    class VideoRepository {
        -MediaStorageS3 storage
        +salvarMetadados(video: Video) Boolean
        +buscarPorId(videoId: String) Video
    }

    class MediaStorageS3 {
        +uploadArquivo(byteStream: Byte[]) String
        +checarExiste(fileId: String) Boolean
    }

    VideoRepositoryDriver --> VideoRepository : exercita (Driver)
    VideoRepository --> MediaStorageS3 : consome
</pre>
<img width="961" height="945" alt="image" src="https://github.com/user-attachments/assets/551151c7-b790-4ada-9055-bb8888a4ff70" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Classes de Base sob Teste:** `VideoRepository` e `MediaStorageS3`
* **Acionador (Driver):** `VideoRepositoryDriver`
* **Responsabilidade:** Validar a integração da camada de persistência e armazenamento de objetos no nível mais baixo da arquitetura.

##### Operação do Teste
* **Métodos Testados:** `salvarMetadados(video)` e `buscarPorId(videoId)`
* **Modo de Execução:** Teste incremental Bottom-Up. Como os serviços superiores (ex.: `VideoCatalogService`) ainda não foram construídos, o `VideoRepositoryDriver` é desenvolvido para simular as chamadas.
* **Isolamento:** As camadas de base interagem diretamente entre si, acionadas exclusivamente pelo Driver de teste.

##### Detecção de Defeitos
O teste visa identificar:
1. **Erros de Persistência:** Incompatibilidades no salvamento e recuperação de dados entre `VideoRepository` e a infraestrutura S3.
2. **Falhas de Mapeamento de Dados:** Tipagem incorreta ou corrupção de atributos de metadados durante a escrita.
3. **Exceções da Camada de Base:** Erros de chave duplicada ou arquivos não encontrados que não são capturados adequadamente pelo repositório.

---

### 2.4 Teste de Fumaça (Smoke Testing)

#### Diagrama de Sequência UML

<pre class="mermaid">
sequenceDiagram
    autonumber
    actor Cliente as VideoPlayerClient
    participant Auth as AuthService
    participant Catalog as VideoCatalogService
    participant CDN as ICDNProvider

    Note over Cliente, CDN: Suíte de Fumaça (Health Check Crítico do Build)
    Cliente->>Auth: autenticarUsuario("user@test.com", "pass123")
    Auth-->>Cliente: 200 OK (Token JWT)
    
    Cliente->>Catalog: obterMetadadosVideo("filme-456")
    Catalog-->>Cliente: 200 OK (Título, Duracao, DRM_Id)
    
    Cliente->>CDN: requisitarManifestoHLS("filme-456", Token)
    CDN-->>Cliente: 200 OK (playlist.m3u8)
</pre>
<img width="1642" height="851" alt="image" src="https://github.com/user-attachments/assets/abb0711d-a74e-48d8-be86-f50aa46d2fa6" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Ecossistema do Fluxo Crítico:** `VideoPlayerClient`, `AuthService`, `VideoCatalogService` e `ICDNProvider`.
* **Responsabilidade:** Executar uma verificação superficial de integridade (*health check*) para garantir que os serviços essenciais do sistema estão no ar e respondendo.

##### Operação do Teste
* **Fluxo Testado:** `autenticarUsuario` -> `obterMetadadosVideo` -> `requisitarManifestoHLS`.
* **Modo de Execução:** Automação disparada imediatamente após o deploy no pipeline de CI/CD.
* **Isolamento:** Teste funcional de ponta a ponta cobrindo apenas o "caminho feliz" principal.

##### Detecção de Defeitos
O teste visa identificar:
1. **Regressões Catastróficas:** Impossibilidade total de logar ou carregar vídeos após uma nova implantação.
2. **Falhas de Configuração de Ambiente:** URLs de serviços incorretas, variáveis de ambiente ausentes ou serviços offline.
3. **Quebra de Integração Crítica:** Falhas que inviabilizam a execução da suíte completa de testes regressivos.

---

### 2.5 Teste de Regressão

#### Diagrama de Componentes UML

<pre class="mermaid">
graph LR
    subgraph "Módulo Modificado (v2.1)"
        RecEngine[Novo Algoritmo de Recomendação]
    end

    subgraph "Camada de Negócio Existente"
        CatalogService[VideoCatalogService]
    end

    subgraph "Suíte de Regressão Automatizada"
        Suite[Suíte de Testes de Contrato e Busca]
    end

    RecEngine -->|Integrado a| CatalogService
    Suite -->|Re-executa cenários salvos| CatalogService
    Suite -->|Valida compatibilidade| RecEngine
</pre>
<img width="1897" height="511" alt="image" src="https://github.com/user-attachments/assets/299b15e2-2eb7-436e-aa6b-5576ffc253ff" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Módulos Afetados:** `VideoCatalogService` e `Novo Algoritmo de Recomendação`.
* **Responsabilidade:** Garantir que a introdução do novo motor de recomendação não quebrou funcionalidades existentes no serviço de catálogo.

##### Operação do Teste
* **Métodos Testados:** Conjunto automatizado de testes cobrindo busca por título, listagem por categoria e paginação.
* **Modo de Execução:** Re-execução integral da suíte de testes do módulo sempre que o código é alterado ou refatorado.
* **Isolamento:** Teste automatizado comparando o comportamento atual do sistema com os resultados históricos esperados.

##### Detecção de Defeitos
O teste visa identificar:
1. **Efeitos Colaterais Indesejados:** Quebras em funcionalidades legadas causadas por alterações em dependências compartilhadas.
2. **Degradação de Contrato:** Modificações acidentais nos formatos de resposta do `VideoCatalogService`.
3. **Bugs Reincidentes:** Retorno de defeitos que já haviam sido corrigidos em versões anteriores.

---

## 3. Teste de Validação (Validation Testing)

### 3.1 Teste Baseado em Casos de Uso (Use Case Testing)

#### Diagrama de Casos de Uso UML

<pre class="mermaid">
graph LR
    actor Usuario as Usuário Assinante
    
    subgraph "StreamFlix - Caso de Uso: Reproduzir Vídeo em Alta Definição"
        UC1((UC01: Autenticar no Player))
        UC2((UC02: Selecionar Título do Catálogo))
        UC3((UC03: Validar Licença DRM))
        UC4((UC04: Iniciar Reprodução HLS))
        
        UC1 --> UC2
        UC2 --> UC3
        UC3 --> UC4
    end

    Usuario --> UC1
    Usuario --> UC2
</pre>
<img width="1157" height="477" alt="image" src="https://github.com/user-attachments/assets/b09c4bc8-b2fd-495e-afca-d88bb420852c" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Caso de Uso:** `UC04: Reproduzir Vídeo em Alta Definição (1080p/4K)`
* **Atores Envolvidos:** Usuário Assinante (Ativo) e o ecossistema `StreamFlix` (`VideoPlayerClient`, `AuthService`, `DRMTokenValidator`, `ICDNProvider`).
* **Responsabilidade:** Validar se a plataforma atende às expectativas de uso do cliente final durante o fluxo completo de visualização de conteúdo digital.

##### Operação do Teste
* **Pré-condições:** Usuário com assinatura ativa e conectividade de rede superior a 15 Mbps.
* **Fluxo Principal Exercitado:**
  1. O usuário acessa o aplicativo e seleciona um filme do catálogo.
  2. O player solicita autorização DRM e recebe a licença de exibição.
  3. A mídia inicia a reprodução em menos de 2 segundos no perfil de alta resolução (1080p), sincronizando áudio e legenda.
* **Modo de Execução:** Teste funcional executado sob a perspectiva do usuário (Black-Box / Validação de requisitos de negócio).
* **Isolamento:** Nenhum. Avaliação da jornada ponta a ponta do usuário.

##### Detecção de Defeitos
O teste visa identificar:
1. **Desacordo com Requisitos do Usuário:** Latência excessiva no carregamento inicial (*buffering* inicial > 3 segundos).
2. **Falhas na Experiência de Uso (UX):** Desincronização entre a faixa de áudio e as legendas durante a reprodução.
3. **Quebra de Regra de Negócio:** Bloqueio indevido de conteúdo HD para usuários que possuem planos elegíveis.

---

### 3.2 Teste Transversal de Desempenho (Performance Testing)

#### Diagrama de Sequência UML

<pre class="mermaid">
sequenceDiagram
    autonumber
    actor C1 as Cliente Simultâneo 1
    actor C2 as Cliente Simultâneo N (Carga)
    participant Gateway as API Gateway / Load Balancer
    participant Session as StreamingSessionService
    participant Cache as Redis Cache

    Note over C1, Cache: Teste de Carga e Tempo de Resposta sob Stress
    C1->>Gateway: POST /api/v1/session/start (contentId="filme-123")
    C2->>Gateway: POST /api/v1/session/start (contentId="filme-123")
    Gateway->>Session: Injetar requisições concorrentes
    Session->>Cache: Obter metadados em memória rápida
    Cache-->>Session: Retorno < 5ms
    Session-->>Gateway: 200 OK (Sessão Criada)
    Gateway-->>C1: Tempo de resposta total < 200ms
    Gateway-->>C2: Tempo de resposta total < 200ms
</pre>
<img width="1626" height="702" alt="image" src="https://github.com/user-attachments/assets/25c973ea-98fb-4369-9142-c87706162c49" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Módulos sob Carga:** `API Gateway`, `StreamingSessionService` e infraestrutura de `Redis Cache`.
* **Responsabilidade:** Avaliar a eficiência do sistema quanto ao tempo de resposta e consumo de recursos computacionais durante o pico de requisições de abertura de sessão.

##### Operação do Teste
* **Métrica Avaliada:** *Throughput* (requisições por segundo - RPS) e tempo médio de resposta (*latency* p95).
* **Modo de Execução:** Disparo automatizado de 5.000 requisições simultâneas de início de sessão via ferramenta de estresse (ex.: JMeter ou k6).
* **Cenário Limite:** Garantir que o tempo de resposta do endpoint permaneça abaixo do SLA estabelecido de 200ms para 95% das chamadas (p95).

##### Detecção de Defeitos
O teste visa identificar:
1. **Gargalos de Desempenho (*Bottlenecks*):** Consultas lentas ao banco de dados por falta de concorrência no cache.
2. **Vazamento de Memória (*Memory Leaks*):** Consumo desordenado de memória RAM nas instâncias do `StreamingSessionService` sob carga sustentada.
3. **Esgotamento de Conexões:** Exaustão do pool de conexões do banco de dados ou do gateway de entrada.

---

### 3.3 Teste Transversal de Segurança (Security Testing)

#### Diagrama de Componentes UML

<pre class="mermaid">
graph TD
    subgraph "Ataque / Execução de Teste de Penetrabilidade"
        Attacker[Cliente Não Autorizado / Script Malicioso]
    end

    subgraph "Barreira de Segurança StreamFlix"
        WAF[Web Application Firewall / WAF]
        Auth[AuthService & Validator]
    end

    subgraph "Recursos Protegidos"
        Media[MediaStorageS3 / Chunks de Vídeo]
    end

    Attacker -->|1. Tentativa de Bypass com Token Adulterado| WAF
    WAF -->|2. Encaminha Requisição| Auth
    Auth -->|3. Identifica Assinatura HMAC Inválida| Auth
    Auth --x|4. Bloqueia Acesso (401 Unauthorized)| Attacker
    Auth -.-x|5. Impede Acesso Direto à Mídia| Media
</pre>
<img width="985" height="748" alt="image" src="https://github.com/user-attachments/assets/347f2cce-3dfc-452a-826c-98f8a951e184" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Módulos Avaliados:** `DRMTokenValidator`, `AuthService` e regras do `Web Application Firewall (WAF)`.
* **Responsabilidade:** Garantir a proteção dos ativos digitais (vídeos) contra sequestro de URL (*hotlinking*), manipulação de tokens JWT e acesso não autorizado a conteúdos pagos.

##### Operação do Teste
* **Método Testado:** Injeção de tokens com assinaturas adulteradas, tentativa de reuso de tokens expirados e requisição direta de trechos de vídeo (chunks HLS) sem autenticação.
* **Modo de Execução:** Teste de Penetrabilidade (*Pen Test*) funcional de segurança estruturado.
* **Resultado Esperado:** Rejeição imediata de todas as chamadas ilegítimas com códigos HTTP `401 Unauthorized` ou `403 Forbidden`, acompanhada de log de auditoria de segurança.

##### Detecção de Defeitos
O teste visa identificar:
1. **Vulnerabilidades de Autenticação:** Falhas de validação sintática ou criptográfica que permitam contornar o controle de acesso.
2. **Exposição de URLs Privadas:** Links de streaming (S3/CDN) gerados sem tempo de expiração ou sem assinatura digital (*Pre-signed URLs*).
3. **Falta de Rate Limiting:** Ausência de controle de requisições por IP, permitindo ataques de força bruta contra o serviço de login.

---

## 4. Teste de Sistema e Aceitação (System & Acceptance Testing)

### 4.1 Teste de Sistema Ponta a Ponta (End-to-End - E2E)

#### Diagrama de Sequência UML

<pre class="mermaid">
sequenceDiagram
    autonumber
    actor Player as VideoPlayerClient
    participant Auth as AuthService
    participant DB as UserDatabase
    participant Catalog as VideoCatalogService
    participant Session as StreamingSessionService
    participant DRM as DRMTokenValidator
    participant CDN as ICDNProvider

    Note over Player, CDN: Fluxo E2E: Assinatura, Seleção e Reprodução em Ambiente Integrado
    Player->>Auth: 1. Realizar Login (credenciais)
    Auth->>DB: Consultar status de assinatura
    DB-->>Auth: Assinatura ativa
    Auth-->>Player: Retorna JWT Token
    
    Player->>Catalog: 2. Buscar filme "Batman"
    Catalog-->>Player: Detalhes do filme + ContentID
    
    Player->>Session: 3. Solicitar Abertura de Sessão (ContentID + JWT)
    Session->>DRM: Validar direitos de exibição
    DRM-->>Session: Licença Válida
    Session->>CDN: Requisitar Manifesto HLS
    CDN-->>Session: URL da Playlist (.m3u8)
    Session-->>Player: Retorna URL de Streaming + DRM Token
    
    Player->>CDN: 4. Baixar segmentos de vídeo (Chunks HLS)
    CDN-->>Player: Fluxo contínuo de bytes de mídia
</pre>
<img width="1651" height="805" alt="image" src="https://github.com/user-attachments/assets/ef382093-3e4f-4019-be27-3b7f55341655" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Sistema Completo:** Todo o ecossistema `StreamFlix` em ambiente homologado (Staging/Pre-production).
* **Responsabilidade:** Validar se a integração completa de software, banco de dados, gateways e redes funciona harmonicamente do início ao fim.

##### Operação do Teste
* **Cenário Executado:** Fluxo completo de um novo usuário: login -> navegação e busca -> autorização DRM -> inicialização do player -> consumo do manifesto de mídia HLS.
* **Modo de Execução:** Teste automatizado de interface e API disparado via Cypress ou Playwright.
* **Isolamento:** Nenhum. O teste roda em ambiente idêntico ao de produção com serviços reais operando integrados.

##### Detecção de Defeitos
O teste visa identificar:
1. **Falhas em Fronteiras do Sistema:** Incompatividades de contrato que ocorrem apenas quando todas as peças do sistema interagem sob uma mesma rede.
2. **Erros de Estado de Dados:** Dessincronização entre a alteração de status do usuário no banco de dados e a liberação de permissão na API.
3. **Falhas de Conectividade Extensiva:** Interrupções ou retries falhos na comunicação com CDN ou infraestrutura S3 durante o carregamento.

---

### 4.2 Teste Alpha (Alpha Testing)

#### Diagrama de Componentes UML

<pre class="mermaid">
graph TD
    subgraph "Ambiente Interno da Organização (Laboratório de QA)"
        DevTeam[Equipe de Desenvolvimento e QA]
        AlphaUsers[Funcionários / Testers Internos]
        
        subgraph "Plataforma StreamFlix (Build Alpha)"
            App[VideoPlayerClient - Alpha Release]
            Telemetry[Coletor de Crash & Logs Internos]
        end
    end

    AlphaUsers -->|1. Testam novos recursos| App
    DevTeam -->|2. Monitoram em tempo real| Telemetry
    App -->|3. Envia métricas e relatórios de falha| Telemetry
    DevTeam -->|4. Aplica correções imediatas| App
</pre>
<img width="1762" height="437" alt="image" src="https://github.com/user-attachments/assets/51ba7319-5979-4fc9-b972-970b244e91c0" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Versão sob Avaliação:** `StreamFlix Release Candidate (v2.0-Alpha)`.
* **Responsabilidade:** Avaliar a estabilidade global da versão candidata a lançamento sob a perspectiva de usuários internos da empresa antes da liberação ao público externo.

##### Operação do Teste
* **Perfil dos Testadores:** Desenvolvedores, analistas de qualidade, designers de produto e colaboradores da organização.
* **Modo de Execução:** Teste exploratório orientado a roteiros de uso reais em ambiente interno controlado (*In-house Testing*).
* **Foco:** Identificar falhas críticas de usabilidade, comportamentos inesperados de navegação e crashes de aplicação.

##### Detecção de Defeitos
O teste visa identificar:
1. **Crashes Inesperados da Aplicação:** Encerramentos abruptos do app ao alternar rapidamente entre vídeos ou telas.
2. **Falhas Severas de Usabilidade:** Layouts quebrados em resoluções específicas ou botões de controle de mídia sem resposta.
3. **Bugs de Regras Não Mapeadas:** Comportamentos anômalos identificados em cenários de uso não previstos nos casos de teste formais.

---

### 4.3 Teste Beta (Beta Testing)

#### Diagrama de Componentes UML

<pre class="mermaid">
graph LR
    subgraph "Ambiente Externo de Produção / Real"
        BetaUsers[Usuários Finais Convidados / Beta Testers]
    end

    subgraph "Infraestrutura StreamFlix (Beta Release)"
        AppBeta[App Mobile / Web StreamFlix Beta]
        Crashlytics[Serviço de Telemetria e Erros (Crashlytics)]
        FeedbackService[Canal de Feedback do Usuário]
    end

    BetaUsers -->|1. Utilizam o app no dia a dia| AppBeta
    AppBeta -->|2. Captura automática de exceções| Crashlytics
    BetaUsers -->|3. Submetem opiniões e bugs| FeedbackService
</pre>
<img width="1141" height="366" alt="image" src="https://github.com/user-attachments/assets/fac27141-d60f-4dac-adf9-e05cdf4f8115" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Versão sob Avaliação:** `StreamFlix Release Candidate (v2.0-Beta)`.
* **Responsabilidade:** Avaliar o comportamento do sistema quando submetido à diversidade real de dispositivos, sistemas operacionais e condições imprevisíveis de rede dos clientes finais.

##### Operação do Teste
* **Perfil dos Testadores:** Amostra selecionada de usuários finais reais atuando em seu próprio ambiente doméstico/móvel (*Field Testing*).
* **Modo de Execução:** Uso não supervisionado da plataforma no cotidiano.
* **Monitoramento:** Coleta passiva de métricas via telemetria e análise de relatórios de falhas (*Crash Reports*).

##### Detecção de Defeitos
O teste visa identificar:
1. **Incompatibilidades de Hardware/OS:** Falhas de decodificação de vídeo que ocorrem apenas em marcas ou versões específicas de Smart TVs ou smartphones.
2. **Comportamento em Redes Instáveis:** Falhas no algoritmo de *Adaptive Bitrate (ABR)* em conexões 3G/4G com alta perda de pacotes.
3. **Grau de Satisfação do Cliente:** Nível de aceitação das novas funcionalidades em relação às expectativas dos usuários.

---

### 4.4 Teste de Aceitação do Usuário (User Acceptance Testing - UAT)

#### Diagrama de Casos de Uso UML

<pre class="mermaid">
graph TD
    actor Client[Stakeholders / Cliente do Negócio]
    
    subgraph "Matriz de Critérios de Aceitação (UAT)"
        UAT1((UAT01: Playback em < 2s))
        UAT2((UAT02: Cobrança Recorrente Correta))
        UAT3((UAT03: Suporte a Múltiplos Perfis))
        
        UAT1 -->|Validado| Status[Homologado para Lançamento]
        UAT2 -->|Validado| Status
        UAT3 -->|Validado| Status
    end

    Client -->|Valida Termos do Contrato| UAT1
    Client -->|Valida Termos do Contrato| UAT2
    Client -->|Valida Termos do Contrato| UAT3
</pre>
<img width="1162" height="742" alt="image" src="https://github.com/user-attachments/assets/60961851-3efe-4da8-ac99-2a443475b216" />


#### Documentação do Cenário de Teste

##### Componente sob Teste
* **Produto de Software:** Módulo de Assinaturas e Reprodução da Plataforma `StreamFlix`.
* **Responsabilidade:** Validar formalmente se o software desenvolvido atende a todos os critérios de aceitação contratuais e requisitos de negócio definidos pelos *Stakeholders*.

##### Operação do Teste
* **Validadores:** Product Owner (PO), patrocinadores do projeto e representantes da área de negócios.
* **Modo de Execução:** Verificação rigorosa apoiada na Matriz de Rastreabilidade de Requisitos e Termos do Contrato.
* **Meta Final:** Obtenção do parecer formal de aceite (*Sign-off*) autorizando a implantação comercial do sistema em ambiente de produção.

##### Detecção de Defeitos
O teste visa identificar:
1. **Desconformidades de Contrato:** Divergências entre o comportamento implementado e o especificado nos requisitos de negócio.
2. **Incorreções de Regra de Faturamento:** Falhas na aplicação de cupons, taxas de conversão ou datas de cobrança de planos.
3. **Critérios de Aceitação Não Atingidos:** Indicadores de qualidade (como tempo máximo de carregamento) abaixo do exigido pelos patrocinadores.
