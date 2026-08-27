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

## 🛠️ 1. Mapeamento e Adaptação dos Prompts Base

Abaixo estão registrados os prompts originais propostos no guia de laboratório (originalmente focados no domínio de *Gateway de Pagamentos Pix*) e como foram adaptados para a arquitetura da *StreamFlix*.

### 1.1 Teste de Unidade (Unit Testing)
* **Prompt Adaptado:** *"Atue como um Engenheiro de QA Sênior. Crie o cenário 1.1 (Verificação de Lógica Atômica) para uma plataforma de streaming (StreamFlix). Foque na classe `DRMTokenValidator`. Gere o Diagrama de Classes em Mermaid e preencha as seções: Componente sob Teste, Operação do Teste e Detecção de Defeitos."*

### 1.2 Teste de Integração (Integration Testing)
* **Prompt Adaptado:** *"Gere os cenários de Teste de Integração (2.1 a 2.5) para a StreamFlix. Adapte a integração Big Bang para `AuthService`, `VideoCatalogService`, `StreamingSessionService` e `AkamaiCDNProvider`. No Top-Down, utilize um `CDNProviderStub`. No Bottom-Up, crie um `VideoRepositoryDriver`. Para o Smoke Test, utilize um diagrama de sequência cobrindo login e obtenção de manifesto HLS. No Teste de Regressão, modele o impacto do novo motor de recomendação."*

### 1.3 Teste de Validação (Validation Testing)
* **Prompt Adaptado:** *"Elabore os cenários de Teste de Validação (3.1 a 3.3). Modele o Caso de Uso de reprodução em alta definição (1080p/4K), o Teste de Desempenho com foco no tempo de resposta do API Gateway sob carga (<200ms) e o Teste de Segurança cobrindo a validação contra adulteração de tokens DRM."*

### 1.4 Teste de Sistema e Aceitação (System & Acceptance Testing)
* **Prompt Adaptado:** *"Desenvolva os cenários de Teste de Sistema e Aceitação (4.1 a 4.4) para a StreamFlix. Crie o teste E2E cobrindo a jornada completa do usuário, o Teste Alpha no ambiente interno da empresa, o Teste Beta com telemetria via Crashlytics em dispositivos reais e o Teste de Aceitação (UAT) com critérios de homologação do negócio."*

---

## 🔄 2. Prompts de Refinamento e Formatação Técnica

Durante o desenvolvimento do laboratório, foram utilizados prompts de refinamento para garantir rigor técnico e compatibilidade com o renderizador de Markdown do GitHub:

1. **Padronização da Arquitetura Base:**
   * *"Defina uma arquitetura padronizada no item 0 contendo cliente, serviços de backend, interfaces e banco de dados para garantir a rastreabilidade dos 13 cenários de teste."*
2. **Ajuste na Renderização dos Diagramas Mermaid:**
   * *"Ajuste o formato dos blocos de código Mermaid para utilizar tags `<pre class="mermaid">` de modo a evitar quebras de sintaxe no leitor do repositório."*
3. **Consistência na Estrutura de Documentação:**
   * *"Garanta que todos os 13 cenários possuam exatamente as subseções: Componente sob Teste, Operação do Teste e Detecção de Defeitos."*

---

## 🛡️ 3. Declaração de Validação Crítica pelo Autor

Eu, **Ícaro Pontes**, declaro que:

1. **Acompanhamento Técnico:** Todos os conceitos de Verificação, Validação e Testes de Software apresentados (Stubs, Drivers, Big Bang, Top-Down, Bottom-Up, Smoke, Regressão, E2E, Alpha, Beta e UAT) foram analisados e compreendidos.
2. **Coerência de Domínio:** A arquitetura do ecossistema *StreamFlix* foi revisada para refletir um fluxo realista de streaming de vídeo sob demanda (autenticação JWT, manifestos HLS, validação de DRM e distribuição via CDN).
3. **Autonomia e Defesa:** Todo o conteúdo gerado e refinado com o auxílio da IA está pronto para ser apresentado e defendido perante o professor da disciplina.
