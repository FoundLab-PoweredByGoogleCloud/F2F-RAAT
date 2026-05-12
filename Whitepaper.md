# FoundLab Tecnologia Ltda.

## Whitepaper Técnico-Estratégico — Edição 2025/2026

**Versão:** 1.1 (revisão editorial · maio 2026)
**Classificação:** Confidencial — Circulação Controlada
**Localização:** São Paulo, Brasil
**Produto central:** REX Guard (anteriormente referido como *REX Gemini Guard*)
**Plataforma:** Google Cloud · região `southamerica-east1`

-----

# Trust by Physics

## Processamento de IA com não-persistência de conteúdo: uma arquitetura de confiança verificável para instituições financeiras reguladas

-----

## Sumário

1. Sumário Executivo
1. Anatomia da confiança e a engenharia da não-persistência
1. Auditabilidade e privacidade — endereçando uma tensão estrutural
1. Eficiência financeira via *Committed Use Discounts*
1. Arquitetura de referência
1. Cadeia de suprimentos de dados e superfície de risco residual
1. Posicionamento — uma camada de infraestrutura, não um produto de segurança
1. Apêndice técnico

-----

## 1. Sumário Executivo

A inteligência artificial passou a operar como infraestrutura crítica em instituições financeiras. Decisões de crédito, detecção de fraude, análise de risco e precificação são parcial ou integralmente executadas por modelos algorítmicos em janelas de milissegundos, em escala incompatível com supervisão humana caso a caso.

Esse deslocamento criou uma tensão estrutural entre duas obrigações regulatórias presentes em jurisdições maduras: a exigência de evidências auditáveis sobre decisões algorítmicas (no Brasil, materializada pela Resolução BCB 538/2025) e a exigência de minimização e eliminação de dados pessoais (LGPD, GDPR e equivalentes). Em arquiteturas convencionais, atender plenamente a uma das obrigações tende a comprometer a outra.

O paradigma **Trust by Physics** propõe um deslocamento da garantia: em vez de depender de políticas, controles de acesso ou compromissos contratuais sobre a manutenção e o uso de dados, a arquitetura busca **eliminar a persistência de conteúdo semântico** ao longo do ciclo de inferência, mantendo apenas evidências criptográficas verificáveis das decisões tomadas.

Este whitepaper descreve o produto REX Guard, a arquitetura subjacente e as propriedades verificáveis que ela busca oferecer. Os autores reconhecem que adequação regulatória plena depende de validação por jurídico da instituição contratante; este documento descreve **capacidade técnica de endereçamento de requisitos**, não declara conformidade unilateral.

**Propriedades que o desenho persegue:**

- Não-persistência de conteúdo semântico de prompts e respostas em substrato durável.
- Geração, para cada inferência, de uma evidência criptográfica encadeada (Veritas Ledger), assinada via Cloud KMS com proteção HSM.
- Verificabilidade independente do ledger por auditores autorizados, sem necessidade de confiança na palavra do fornecedor.
- Compatibilidade com Committed Use Discounts (CUDs) Google Cloud, permitindo absorção do custo de implementação por orçamento de cloud já comprometido.

-----

## 2. Anatomia da Confiança e a Engenharia da Não-Persistência

A maior parte da arquitetura de segurança digital construída no século XX assumiu que segurança é um problema de **controle de acesso**: quem pode ver o dado, quando e sob quais condições. Essa lógica produziu firewalls, criptografia em trânsito, gestão de identidade e controles baseados em *roles*.

A premissa do REX Guard é diferente: **reduzir a superfície sobre a qual o controle de acesso precisa operar**. Se o conteúdo semântico de uma inferência não é gravado em armazenamento durável em nenhuma etapa do pipeline operado pela FoundLab, vetores de ataque que dependem desse conteúdo persistente — exfiltração pós-processamento, comprometimento de backup, leitura indevida por operador — deixam de ter alvo.

**Dois mecanismos compõem o desenho:**

**Memória volátil como substrato de processamento.** A memória RAM alocada por um processo é, por construção, transitória: quando o processo termina, o sistema operacional libera as páginas para reuso. Não há mecanismo comercial ou operacional rotineiro, em ambientes serverless managed como Cloud Run, para recuperação semântica de páginas desalocadas por terceiros não privilegiados. Ataques contra memória residual existem em literatura forense (*cold boot*, análise de hibernação) mas dependem de acesso físico privilegiado ao host, ausente no modelo de serviço utilizado.

**Imutabilidade criptográfica como substituto da retenção de conteúdo.** Um hash SHA-256 prova que um determinado conteúdo existiu, sem revelar qual era. Combinado com assinatura ECDSA P-256 e encadeamento de blocos (hash-chaining), produz uma linha do tempo verificável que dispensa armazenamento do conteúdo original.

### Analogia operacional

O dispositivo de registro de voo de uma aeronave não grava conversas dos passageiros nem dados pessoais. Registra exclusivamente parâmetros operacionais relevantes para análise de eventos: altitude, atitude, posição de comandos. O resultado é evidência de auditoria utilizável, com pegada de dados pessoais nula por construção. O REX Guard opera segundo lógica equivalente: registra a evidência criptográfica do evento de inferência, não o conteúdo da interação.

### Fluxo de processamento

```
Cliente / Sistema → Requisição de inferência
        ↓
REX Guard (proxy fail-closed em Cloud Run)
        ↓
Validação de consentimento (cache Redis · OPIN)
        ↓
Gates de segurança pré-inferência (PII / OFAC / prompt injection)
        ↓
Inferência em RAM volátil (Vertex AI · Gemini)
        ↓
Validação pós-inferência
        ↓
Geração de evidência: SHA-256 (input, output, policy)
        ↓
Assinatura ECDSA P-256 via Cloud KMS HSM
        ↓
Persistência da evidência no Veritas Ledger (BigQuery append-only)
        ↓
Resposta ao cliente · contexto desalocado · container reciclado
```

O conteúdo semântico do prompt e da resposta nunca é gravado pela FoundLab em substrato durável. O que persiste é o conjunto de hashes, a assinatura e os metadados operacionais.

-----

## 3. Auditabilidade e Privacidade — Endereçando uma Tensão Estrutural

Nas conversas internas entre áreas de Compliance e de Proteção de Dados em grandes instituições financeiras, a mesma tensão se repete: o regulador exige que cada decisão algorítmica seja rastreável e justificável; a lei de proteção de dados exige minimização e eliminação. Em arquiteturas convencionais, atender plenamente a uma exigência implica concessões à outra.

Essa tensão é **estrutural na arquitetura, não na regulação**. Os reguladores não exigem que o dado pessoal seja retido para fins de auditoria — exigem que a *evidência sobre a decisão* seja retida. A confusão entre os dois é uma propriedade do desenho convencional, que armazena conteúdo como subproduto de logging.

### Mapa regulatório de referência

|Norma                          |Obrigação relevante                                                                                                                                    |
|-------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
|LGPD (Lei 13.709/2018, art. 18)|Eliminação de dados pessoais ao fim da finalidade que justificou seu tratamento.                                                                       |
|BCB 538/2025                   |Registros auditáveis sobre decisões de modelos algorítmicos com impacto em operações financeiras: rastreabilidade, justificabilidade e disponibilidade.|
|GDPR · EU Data Act · NIS2      |Minimização, portabilidade, transparência e requisitos de segurança para infraestruturas críticas, com escopo específico por instrumento.              |
|FCA / PRA (Reino Unido)        |Frameworks de governança algorítmica com requisitos de *explainability* e trilha de auditoria verificável.                                             |

A FoundLab não declara conformidade unilateral em nenhuma dessas normas. A leitura institucional desses requisitos, a aplicabilidade caso a caso, a articulação com sistemas legados e a sustentação jurídica permanecem responsabilidade da instituição contratante e de seu corpo jurídico.

### Como o REX Guard endereça cada requisito

|Requisito regulatório                 |Mecanismo técnico no REX Guard                                                        |Natureza da garantia                     |
|--------------------------------------|--------------------------------------------------------------------------------------|-----------------------------------------|
|Rastreabilidade de decisões           |Hash de entrada + hash de saída + assinatura ECDSA + encadeamento                     |Verificável por auditor autorizado       |
|Não retenção de conteúdo pessoal      |Processamento exclusivo em RAM; ausência de gravação semântica pelo REX Guard         |Arquitetural                             |
|Justificabilidade algorítmica         |*Policy snapshot hash* incluído em cada bloco; encadeamento entre blocos              |Imutável após selagem                    |
|Atendimento ao direito ao esquecimento|Ausência de dado pessoal persistente no escopo operado pela FoundLab                  |Arquitetural, dentro do escopo           |
|Auditoria regulatória independente    |Veritas Ledger acessível por *role* específica do auditor (provisionamento contratual)|Verificabilidade controlada por permissão|

O escopo dessas garantias é o pipeline operado pela FoundLab. Decisões da instituição contratante sobre logging em sistemas legados, retenção em camadas pré-proxy ou pós-resposta, e tratamento de dados em sistemas adjacentes permanecem sob governança da instituição.

-----

## 4. Eficiência Financeira via Committed Use Discounts

Instituições financeiras de grande porte costumam manter *Committed Use Discounts* (CUDs) plurianuais com Google Cloud, com descontos negociados sobre o preço de lista. Em muitos casos, parte do compromisso de consumo não é convertida em valor operacional dentro do ciclo de vigência.

O REX Guard pode ser contratado via **Private Offer** no Google Cloud Marketplace, modalidade na qual o consumo é debitado contra o CUD existente da instituição. O efeito é que a implantação de uma camada de governança de IA não requer abertura de novo *procurement* ou de orçamento incremental: a despesa é absorvida por capital já aprovado e comprometido.

**Cuidado de linguagem.** O modelo não elimina custo — desloca o custo para uma rubrica orçamentária já aprovada. A vantagem é de eficiência financeira e de velocidade de aprovação, não de gratuidade.

### Modelo de canal

|Canal       |Produto                                     |Modelo comercial                                                       |
|------------|--------------------------------------------|-----------------------------------------------------------------------|
|NetSecurity |REX Guard — mercado financeiro              |*Revenue share* 30% · contratação via Private Offer com absorção em CUD|
|Century Data|Umbrella Platform — compliance governamental|*Revenue share* 30% · licenciamento                                    |

-----

## 5. Arquitetura de Referência

### Princípios

**Zero trust por padrão.** Nenhum componente assume confiança em outro. Cada interação é autenticada, autorizada e registrada.

**Compute efêmero.** A computação sensível ocorre em containers reciclados ao fim de cada ciclo. Não há estado persistente em nível de aplicação no proxy.

**Responsabilidade criptográfica.** Cada decisão é vinculada a uma evidência verificável, assinada por chave gerenciada em Cloud KMS com proteção HSM.

**Desenho com regulação em vista.** A arquitetura foi concebida para endereçar requisitos da BCB 538/2025 e da LGPD desde o primeiro princípio, em vez de ser adaptada a posteriori.

### Topologia

```
Clientes externos
        │
        ▼
Cloud Armor (WAF · rate limit · proteção DDoS)
        │
        ▼
VPC — região southamerica-east1 (São Paulo)
        │
        ├── REX Guard (Cloud Run · container efêmero)
        │       ├── Camada de interceptação (Fastify + TypeScript)
        │       ├── Processamento em RAM volátil
        │       └── Geração de hash + assinatura
        │
        ├── Vertex AI — Gemini (inferência)
        ├── Cloud KMS — chave ECDSA P-256 com proteção HSM
        └── BigQuery — Veritas Ledger (append-only, particionado)
```

O Veritas Ledger reside em projeto e dataset do cliente, sob suas próprias chaves CMEK quando aplicável. A FoundLab opera o pipeline; o cliente mantém soberania sobre o repositório de evidências.

### Esquema do bloco de auditoria

```typescript
interface AuditBlock {
  transaction_id: string;        // UUID v4 — identificador da inferência
  timestamp: string;             // RFC 3339 — Spanner commit timestamp
  input_hash: string;            // SHA-256 do payload de entrada
  output_hash: string;           // SHA-256 do payload de saída
  policy_snapshot_hash: string;  // SHA-256 da política vigente
  prev_hash: string;             // Hash do bloco anterior (encadeamento)
  model_version: string;         // Identificador imutável do modelo
  signature: string;             // Assinatura ECDSA P-256 (raw r||s, base64)
  key_version: string;           // Versão da chave KMS utilizada
  latency_ms: number;            // Latência total da inferência
}
```

### Encadeamento determinístico

```
Block[N].prev_hash = SHA256(Block[N-1])

Block[N].block_hash = SHA256(
  Block[N].transaction_id ||
  Block[N].timestamp ||
  Block[N].input_hash ||
  Block[N].output_hash ||
  Block[N].policy_snapshot_hash ||
  Block[N].prev_hash
)
```

Qualquer alteração retroativa de um bloco invalida a cadeia a partir daquele ponto, em uma verificação de complexidade O(n) que pode ser executada por auditor com acesso de leitura ao ledger e à chave pública correspondente.

### Nota sobre marcação temporal externa (TSA)

A marcação temporal independente baseada em RFC 3161 (Time-Stamping Authority externa, do tipo FreeTSA, DigiCert ou TSA aprovada pelo regulador) é tratada como **componente em integração**, gateado por configuração. A versão de PoC opera com marcação temporal local. A integração com TSA externa é parte do roteiro até go-live e está documentada como dependência regulatória.

-----

## 6. Cadeia de Suprimentos de Dados e Superfície de Risco Residual

Arquiteturas modernas de IA operam por integração: provedor de modelos, dados de mercado, plataformas de identidade, sistemas legados. Cada vínculo é canal de trânsito de dados e, portanto, superfície de ataque. Frameworks tradicionais de gestão de terceiros frequentemente não cobrem esse **efeito cascata de visibilidade**.

### Vetores mapeados em arquiteturas convencionais

- Logs de API do provedor de modelos, com termos de serviço variáveis quanto a uso para *debugging* ou treinamento.
- Cache em *gateways* de API, que pode persistir conteúdo de inferências anteriores.
- Sistemas de APM e *tracing* distribuído, que capturam fragmentos de payload em traces de performance.
- Datastores de sessão, que persistem histórico de interações por desenho.

### Comparativo de exposição

|Vetor                                                 |Arquitetura convencional              |REX Guard (no escopo operado pela FoundLab)                                      |
|------------------------------------------------------|--------------------------------------|---------------------------------------------------------------------------------|
|Exfiltração pós-processamento de conteúdo semântico   |Risco material                        |Mitigado por ausência de gravação                                                |
|Comprometimento de parceiro na cadeia                 |Risco material por cascata            |Mitigado: parceiros recebem hashes, não conteúdo                                 |
|Ataque a backup ou snapshot de conteúdo               |Risco material                        |Mitigado: não há backup de conteúdo                                              |
|Acesso indevido de operador de cloud a logs semânticos|Risco moderado                        |Reduzido: logs operacionais contêm hashes e metadados                            |
|Resposta a requisição legal estrangeira por conteúdo  |Risco jurídico                        |Reduzido no escopo FoundLab; conteúdo em sistemas do cliente segue sua governança|
|Retenção indevida sob LGPD no pipeline da FoundLab    |Risco material em desenho convencional|Eliminado por desenho, dentro do escopo                                          |

A coluna direita descreve o comportamento do **pipeline operado pela FoundLab**. Decisões de logging, retenção e governança fora desse escopo permanecem responsabilidade da instituição contratante.

### O que persiste por desenho

Hashes SHA-256 de entradas e saídas, assinaturas ECDSA, *policy snapshot hashes*, encadeamento entre blocos, metadados operacionais (timestamp, latência, versão do modelo, versão da chave).

### O que não persiste no escopo FoundLab

Conteúdo do prompt, conteúdo da resposta, dados pessoais do usuário, contexto semântico da interação, identificadores reversíveis de conteúdo.

### Riscos residuais reconhecidos

- **Memória residual em hipervisor.** Ataques contra memória residual em ambientes managed dependem de acesso privilegiado ao host, fora do alcance dos modelos de ameaça realistas para clientes do Cloud Run, mas não são logicamente impossíveis. A garantia é operacional e contratual com o provedor de cloud, não absoluta.
- **Operações pré-proxy e pós-resposta.** Decisões de logging do cliente antes da requisição entrar no REX Guard e depois da resposta retornar permanecem sob sua governança.
- **Integrações com sistemas adjacentes do cliente.** Estão fora do escopo desta arquitetura e devem ser tratadas separadamente.

-----

## 7. Posicionamento — Uma Camada de Infraestrutura

A FoundLab não se posiciona como empresa de segurança. O produto não é uma defesa adicional contra os mesmos vetores de ataque que firewalls e WAFs já endereçam. O REX Guard é uma camada de infraestrutura que busca **reduzir a superfície sobre a qual decisões de segurança e privacidade precisam ser tomadas**, eliminando, no escopo operado pela FoundLab, a persistência do conteúdo que é alvo desses ataques.

A distinção é deliberada. Vendedores de segurança vendem proteção contra ameaças identificadas. Uma camada de infraestrutura de confiança verificável transforma uma classe específica de ameaças em problemas estruturalmente ausentes — não por melhoria incremental, mas por reorganização do desenho.

### Endereçamento por jurisdição

|Jurisdição    |Normas relevantes endereçadas  |Natureza do endereçamento                                                                                 |
|--------------|-------------------------------|----------------------------------------------------------------------------------------------------------|
|Brasil        |BCB 538/2025 · LGPD · CMN 4.557|Operação em região soberana (`southamerica-east1`); arquitetura desenhada com BCB 538 como vetor principal|
|União Europeia|GDPR · EU Data Act · NIS2      |Não-persistência reduz a superfície de obrigações relativas a tratamento de dados pessoais                |
|Reino Unido   |FCA · PRA                      |Trilha criptográfica endereça requisitos de *explainability* e auditabilidade                             |

Em todas as jurisdições, **adequação plena requer parecer jurídico local** sobre o caso de uso específico da instituição. A FoundLab fornece os mecanismos técnicos; a leitura institucional permanece com Compliance e Jurídico do cliente.

-----

## 8. Apêndice Técnico

### A.1 — Especificação criptográfica

|Componente                |Especificação                                             |
|--------------------------|----------------------------------------------------------|
|Algoritmo de hash         |SHA-256 (FIPS 180-4)                                      |
|Algoritmo de assinatura   |ECDSA P-256 (FIPS 186-4)                                  |
|Curva                     |NIST P-256 (secp256r1)                                    |
|Codificação de assinatura |Raw `r‖s` (64 bytes), serializado em base64 URL-safe      |
|Provedor de chaves        |Google Cloud KMS — nível de proteção HSM                  |
|Marcação temporal interna |Cloud Spanner commit timestamp (TrueTime)                 |
|Marcação temporal externa |RFC 3161 TSA — em integração antes do go-live             |
|Persistência de evidências|BigQuery Storage Write API — modo `COMMITTED`, append-only|
|Região de dados           |`southamerica-east1` (São Paulo)                          |

### A.2 — Stack de referência

|Camada                     |Tecnologia                                                          |
|---------------------------|--------------------------------------------------------------------|
|Runtime                    |Node.js 22 LTS + TypeScript 5.x (modo estrito)                      |
|Framework backend          |Fastify 4.x                                                         |
|Compute                    |Google Cloud Run (serverless, container efêmero)                    |
|Modelo                     |Vertex AI — Gemini                                                  |
|KMS                        |Cloud KMS — ECDSA P-256 com proteção HSM                            |
|Cadeia atômica de auditoria|Cloud Spanner — `runTransactionAsync`                               |
|Ledger                     |BigQuery Storage Write API                                          |
|Cache de consentimento     |Cloud Memorystore Redis                                             |
|Infraestrutura como código |Terraform                                                           |
|CI/CD                      |Cloud Build + GitHub Actions (OIDC via Workload Identity Federation)|
|Gestão de segredos         |Secret Manager                                                      |
|WAF                        |Cloud Armor                                                         |
|Observabilidade            |Cloud Monitoring + Cloud Logging (com redação de PII no logger)     |

### A.3 — Glossário

**Não-persistência de conteúdo semântico.** Propriedade arquitetural pela qual o conteúdo de prompts e respostas existe exclusivamente em RAM durante o ciclo de vida da requisição e não é gravado pela FoundLab em substrato durável.

**Trust by Physics.** Posicionamento de produto: substituir garantias baseadas em política, controle de acesso ou compromisso contratual por propriedades verificáveis da arquitetura — não-persistência, encadeamento criptográfico, assinatura HSM.

**Veritas Ledger.** Repositório append-only em BigQuery contendo as evidências encadeadas das inferências processadas, sem conteúdo semântico, acessível por *role* designada do auditor.

**ECDSA P-256.** Algoritmo de assinatura digital baseado em curva elíptica NIST P-256, padronizado em FIPS 186-4. Utilizado para selagem de cada bloco via Cloud KMS HSM.

**Hash-chaining.** Técnica em que cada bloco inclui o hash do bloco anterior, tornando alterações retroativas detectáveis em tempo linear.

**CUD Arbitrage.** Modelo comercial pelo qual a contratação do REX Guard via Private Offer no Google Cloud Marketplace é debitada contra Committed Use Discounts existentes da instituição, absorvendo o custo em orçamento de cloud já comprometido.

**CMEK.** *Customer-Managed Encryption Keys.* Modalidade na qual o cliente mantém controle direto sobre as chaves de criptografia, hospedadas em seu próprio projeto Cloud KMS.

**REX Guard.** Middleware fail-closed que atua como proxy reverso entre aplicações regulamentadas e a API Vertex AI / Gemini, implementando não-persistência de conteúdo semântico e geração de evidência criptográfica por inferência. Nomenclatura anterior em alguns documentos: *REX Gemini Guard*.

-----

**Não pedimos confiança. Oferecemos verificabilidade.**

*Trust by Physics · FoundLab Tecnologia Ltda.*

© 2025–2026 FoundLab Tecnologia Ltda. — Todos os direitos reservados — foundlab.com.br