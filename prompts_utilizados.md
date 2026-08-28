# Registro do Uso Responsável de IA Generativa
**Projeto:** Laboratório 02 — Estratégias e Níveis de Teste na Prática  
**Domínio:** Plataforma de Streaming de Vídeo sob Demanda (StreamFlix)  
**Disciplina:** Verificação, Validação e Testes de Software  
**Autor:** Ícaro Pontes  

---

## 📄 Introdução e Declaração de Uso

Este documento registra de forma transparente a utilização de ferramentas de Inteligência Artificial Generativa como suporte para a estruturação, documentação e elaboração dos diagramas UML do repositório `lab02-vvts-streaming-tests`.

O uso da IA Generativa foi pautado pela **Validação Crítica Obrigatória**, garantindo que nenhum diagrama, componente ou documento fosse incorporado ao repositório sem prévia revisão, adaptação e validação técnica em conformidade com o domínio de *Streaming de Vídeo sob Demanda*.

---

## 🛠️ 1. Mapeamento dos 12 Prompts Essenciais do Projeto

Abaixo estão registrados os 12 prompts fundamentais utilizados durante todo o ciclo de desenvolvimento: desde a concepção do domínio, geração dos 13 diagramas de teste, refinamentos arquiteturais até as correções de sintaxe Mermaid.

### 1. Concepção do Domínio e Arquitetura Base
> "Preciso criar um relatório de testes para a disciplina de Engenharia de Software. O domínio será uma plataforma de streaming de vídeo de alta escala chamada StreamFlix. Monte a arquitetura base do sistema mapeando componentes como cliente, autenticação, catálogo, sessão de streaming, DRM e CDN."

### 2. Mapeamento de Testes de Unidade e Isolamento
> "Gere a documentação e os diagramas para os Testes de Unidade focados no módulo de DRM (`DRMTokenValidator`), demonstrando o teste atômico e o isolamento de componentes sem dependências externas."

### 3. Diagramação das Estratégias de Integração (Big Bang vs. Incremental)
> "Crie os diagramas Mermaid que explicam as abordagens de Testes de Integração: o modelo Big Bang e as abordagens incrementais Top-Down (utilizando Stubs) e Bottom-Up (utilizando Drivers)."

### 4. Detalhamento de Mocks, Stubs e Drivers no Contexto de Mídia
> "Explique a diferença técnica entre o uso de um Stub para simular a resposta da CDN no teste Top-Down e o uso de um Driver para simular a chamada ao repositório de vídeo no teste Bottom-Up."

### 5. Cobertura de Testes de Regressão em Alterações de Código
> "Gere a representação dos testes de regressão automatizados para quando alteramos o algoritmo do motor de recomendação no `VideoCatalogService`, garantindo a compatibilidade com contratos anteriores."

### 6. Execução de Testes de Fumaça (Smoke Testing)
> "Crie o diagrama do Smoke Test para validar o health check crítico do build de streaming, garantindo a execução do caminho básico de autenticação, catálogo e manifesto HLS."

### 7. Validação de Casos de Uso em Testes de Sistema
> "Elabore um diagrama e uma explicação para o Teste de Sistema Funcional focado no Caso de Uso 'Reproduzir Vídeo em Alta Definição', integrando desde o login até a reprodução HLS."

### 8. Testes Não Funcionais: Carga, Desempenho e Stress
> "Modele um cenário de Teste de Carga e Stress no `StreamingSessionService` usando Mermaid para ilustrar múltiplas requisições concorrentes e a validação do tempo de resposta via Redis Cache."

### 9. Testes Não Funcionais: Segurança e Penetrabilidade (PenTest)
> "Crie o diagrama de PenTest simulando uma tentativa de ataque com token JWT adulterado para tentar acessar fragmentos de vídeo no storage S3, mostrando o bloqueio pelo WAF e AuthService."

### 10. Mapeamento dos Ambientes de Aceitação (Alpha e Beta)
> "Modele o fluxo do Teste Alpha em ambiente de laboratório controlado e do Teste Beta em ambiente real de produção com telemetria via Crashlytics para captura automática de exceções."

### 11. Estruturação dos Critérios de Homologação (UAT)
> "Elabore a matriz de critérios de aceitação pelo cliente do negócio (UAT), validando métricas como playback em menos de 2 segundos, cobrança recorrente exata e suporte a múltiplos perfis."

### 12. Padronização e Correção de Sintaxe Mermaid para o GitHub
> "Os blocos de diagrama Mermaid não estão renderizando no README do GitHub. Reconverta todos os diagramas aplicando estritamente a sintaxe nativa com três crases (```mermaid) para exibição correta."

---

## 🔄 2. Prompts de Refinamento e Formatação Técnica

Durante o desenvolvimento do laboratório, foram utilizados prompts de refinamento suplementares para garantir o alinhamento arquitetural e a consistência visual no GitHub:

1. **Padronização da Arquitetura Base:**
   * *"Defina uma arquitetura padronizada no item 0 contendo cliente, serviços de backend, interfaces e banco de dados para garantir a rastreabilidade dos 13 cenários de teste."*
2. **Consistência na Estrutura de Documentação:**
   * *"Garanta que todos os 13 cenários possuam exatamente as subseções: Componente sob Teste, Operação do Teste e Detecção de Defeitos."*

---

## 🛡️ 3. Declaração de Validação Crítica pelo Autor

Eu, **Ícaro Pontes**, declaro que:

1. **Acompanhamento Técnico:** Todos os conceitos de Verificação, Validação e Testes de Software apresentados (Stubs, Drivers, Big Bang, Top-Down, Bottom-Up, Smoke, Regressão, E2E, Alpha, Beta e UAT) foram analisados e compreendidos.
2. **Coerência de Domínio:** A arquitetura do ecossistema *StreamFlix* foi revisada para refletir um fluxo realista de streaming de vídeo sob demanda (autenticação JWT, manifestos HLS, validação de DRM e distribuição via CDN).
3. **Autonomia e Defesa:** Todo o conteúdo gerado e refinado com o auxílio da IA está pronto para ser apresentado e defendido perante o professor da disciplina.
