# Proposta experimental revisada — Artigo 1
## Multimodal diabetic retinopathy grading with external clinical validation

## 1. Objetivo científico

O artigo avaliará se informações clínicas rotineiramente disponíveis acrescentam informação útil à classificação de retinopatia diabética baseada em imagens de fundo de olho e se esse ganho se mantém quando o modelo é transferido para uma coorte clínica externa.

A hipótese central será:

> **A combinação de representação visual e variáveis clínicas melhora a classificação de retinopatia diabética e reduz a perda de desempenho observada sob mudança de domínio, quando comparada a modelos exclusivamente baseados em imagem ou exclusivamente baseados em dados clínicos.**

O estudo será organizado em três famílias de modelos:

- **Modelo A — Image-only**
- **Modelo B — Clinical-only**
- **Modelo C — Multimodal image + clinical**

O objetivo não será apenas comparar três arquiteturas finais, mas estudar de forma controlada:
1. o efeito da origem dos dados visuais;
2. o efeito da adaptação ao BRSET;
3. o efeito do tratamento de desbalanceamento;
4. o valor incremental das variáveis clínicas;
5. a robustez sob mudança de domínio;
6. o comportamento em diferentes níveis de qualidade da imagem;
7. o tipo de erro observado clinicamente.

---

# 2. Papel das bases de dados

## RetinaMNIST

Função principal:
- fornecer uma base pública padronizada para desenvolvimento e comparação inicial de modelos visuais;
- permitir reutilização dos resultados e checkpoints produzidos no projeto anterior;
- funcionar como domínio visual de referência.

Não será usada como coorte clínica principal.

## APTOS 2019

Função principal:
- aumentar a heterogeneidade visual no treinamento;
- acrescentar imagens obtidas em condições de aquisição distintas;
- permitir verificar se treinamento visual em múltiplos domínios melhora a generalização.

## BRSET

Função principal:
- principal base de desenvolvimento do artigo;
- treinamento e teste interno dos modelos A, B e C;
- fornecer simultaneamente imagens, diagnóstico e dados clínicos;
- servir como ponte entre bases públicas e dados clínicos brasileiros.

Todos os splits do BRSET serão feitos em nível de paciente.

## UNESP

Função principal:
- validação externa clínica;
- análise de domain shift;
- análise médica dos erros;
- avaliação da utilidade real do modelo em uma população independente.

A base UNESP não será utilizada para seleção de hiperparâmetros, escolha de arquitetura, early stopping ou definição de threshold.

---

# 3. Endpoints

## Endpoint primário

**Referable diabetic retinopathy (RDR)**

Definição provisória:

- grades 0–1: non-referable;
- grades 2–4: referable.

A definição final será validada pelos oftalmologistas antes dos testes finais.

O modelo continuará sendo treinado como classificador de cinco classes. A probabilidade de RDR será derivada por:

\[
P(RDR) = P(2)+P(3)+P(4)
\]

Isso evita manter dois modelos distintos para classificação binária e ordinal.

## Endpoint secundário

Classificação em cinco graus:

- 0 — no DR;
- 1 — mild;
- 2 — moderate;
- 3 — severe;
- 4 — proliferative DR.

---

# 4. Organização geral dos experimentos

A parte computacional será dividida em cinco blocos:

1. **Visual representation experiments**
2. **Clinical-model experiments**
3. **Multimodal-fusion experiments**
4. **Cross-domain and robustness experiments**
5. **Clinical error analysis**

O desenho permite um número de experimentos suficientemente robusto para um artigo completo, mas evita uma exploração excessiva de dezenas de arquiteturas.

---

# 5. Modelo A — Image-only

O Modelo A será mais do que um único baseline. Ele será utilizado para estudar como a origem dos dados e a adaptação ao BRSET afetam a generalização.

## A0 — Existing project checkpoint

Usar o melhor checkpoint visual já disponível no projeto, treinado com RetinaMNIST/APTOS, desde que passe pela auditoria de proveniência.

Objetivo:
- estabelecer uma referência sem novo treinamento;
- medir diretamente a transferência para BRSET e UNESP.

Avaliações:
- RetinaMNIST;
- APTOS;
- BRSET;
- UNESP.

Esse teste fornecerá uma medida explícita da perda de desempenho entre domínio público e domínio clínico.

---

## A1 — BRSET from scratch / ImageNet initialization

Treinar o backbone escolhido diretamente no BRSET.

Objetivo:
- estabelecer quanto desempenho pode ser obtido apenas com BRSET;
- separar o efeito da representação pública anterior do efeito do próprio BRSET.

Inicialização:
- ImageNet pretrained.

Treino:
- BRSET train.

Seleção:
- BRSET validation.

Teste:
- BRSET internal test;
- UNESP external test.

---

## A2 — Public retinal pretraining → BRSET fine-tuning

Inicialização:
- checkpoint RetinaMNIST + APTOS.

Depois:
- fine-tuning no BRSET.

Objetivo:
- testar se pré-treinamento específico em retina melhora adaptação ao BRSET em relação à inicialização genérica ImageNet.

Comparação principal:

A1 vs A2.

---

## A3 — Imbalance-aware image model

A2 será repetido usando uma estratégia padrão de compensação de desbalanceamento.

Para evitar transformar o artigo em continuação direta de FERS/FUFA, serão usadas apenas técnicas consolidadas.

Proposta:

- A2a — standard cross-entropy;
- A2b — class-weighted cross-entropy;
- A2c — focal loss.

O objetivo é determinar se a vantagem multimodal permanece quando o image-only baseline já foi adequadamente tratado para class imbalance.

Não serão incluídos FERS ou FUFA neste artigo.

---

## A4 — Alternative visual representation

Para tornar a comparação de representação mais convincente, será incluído um segundo tipo de encoder.

Proposta preferencial:

- EfficientNetV2B0 — continuidade com os experimentos anteriores;
- DINOv2 ou outro foundation encoder visual — representação alternativa.

O objetivo não é realizar um benchmark de dezenas de backbones. O objetivo é verificar se os resultados multimodais dependem de um único tipo de representação visual.

Configuração:

- A4-EFF: melhor EfficientNetV2B0;
- A4-FM: melhor foundation-model encoder.

Ambos serão adaptados no BRSET sob o mesmo protocolo de split e avaliação.

Se o custo computacional ou prazo se tornar limitante, o segundo encoder poderá ficar como experimento suplementar.

---

# 6. Modelo B — Clinical-only

O Modelo B será utilizado para determinar quanto da classificação pode ser explicada por informações clínicas sem acessar a imagem.

Variáveis candidatas:

- age;
- sex;
- diabetes duration;
- insulin use;
- hypertension.

Serão utilizadas somente variáveis que possam ser harmonizadas entre BRSET e UNESP.

---

## B0 — Logistic regression baseline

Modelo simples e interpretável.

Objetivos:
- estabelecer baseline clínico;
- medir a contribuição linear dos metadados;
- permitir interpretação de coeficientes.

---

## B1 — Random Forest / Gradient Boosting

Um modelo não linear tabular será incluído.

Preferência:

- XGBoost, LightGBM ou CatBoost;
- se dependências adicionais forem indesejadas, Random Forest.

Objetivo:
- verificar se interações não lineares entre variáveis clínicas aumentam o desempenho.

---

## B2 — Clinical MLP

Arquitetura neural compacta:

clinical variables
→ Dense 32
→ dropout
→ Dense 16
→ output 5 classes.

Objetivo:
- fornecer a representação clínica usada posteriormente no Modelo C;
- comparar desempenho neural com modelos tabulares clássicos.

---

## B3 — Metadata ablation

A contribuição das variáveis será analisada através de ablação.

Configurações:

- B3a — age + sex;
- B3b — diabetes variables only;
- B3c — comorbidity variables;
- B3d — complete clinical set.

Não será necessário treinar todas as combinações possíveis.

O objetivo é identificar grupos de variáveis que fornecem informação incremental.

---

## B4 — Missingness analysis

Dois esquemas serão comparados:

- imputation only;
- imputation + missingness indicators.

Isso é importante porque ausência de informação clínica pode ser sistemática e não aleatória.

---

# 7. Modelo C — Multimodal

O Modelo C será o núcleo do artigo.

A pergunta central será:

> A informação clínica fornece ganho além de uma representação visual já otimizada?

Serão avaliadas diferentes formas de fusão, mas mantendo o desenho interpretável.

---

## C0 — Simple late fusion

Arquitetura principal:

image encoder
→ image embedding

clinical MLP
→ clinical embedding

concatenation
→ dense fusion layer
→ DR prediction

O encoder visual ficará inicialmente congelado.

Objetivo:
- medir o ganho puramente associado à adição dos metadados.

---

## C1 — Late fusion with adapted visual encoder

Utilizar o melhor image encoder do Modelo A após adaptação ao BRSET.

Esse será provavelmente o principal modelo multimodal do artigo.

Comparação:

A-best vs C1.

Essa comparação mede diretamente o valor incremental dos dados clínicos.

---

## C2 — Joint fine-tuning

Depois do treinamento inicial da fusão:

- liberar o último bloco do encoder visual;
- continuar o treinamento com learning rate reduzido.

Objetivo:
- verificar se permitir co-adaptação entre representação visual e clínica aumenta o desempenho.

Esse experimento evita a crítica de que o modelo multimodal foi artificialmente limitado por um encoder completamente congelado.

---

## C3 — Gated multimodal fusion

Será testada uma segunda estratégia de fusão além da concatenação simples.

Exemplo:

\[
z = \alpha z_{image} + (1-\alpha) z_{clinical}
\]

ou um pequeno gate aprendido que controla a contribuição relativa das duas modalidades.

Objetivo:
- verificar se o modelo aprende a ajustar a contribuição da informação clínica e visual.

A implementação deverá permanecer pequena e interpretável.

Não será usado mecanismo complexo de cross-attention neste artigo.

---

## C4 — Modality ablation

Na inferência serão executados:

- imagem + clínica;
- imagem sem clínica;
- clínica sem imagem.

Essa análise permite verificar se o modelo multimodal realmente utiliza as duas modalidades.

---

## C5 — Metadata-group ablation

Usar o melhor Modelo C e remover seletivamente:

- demographic variables;
- diabetes-related variables;
- comorbidity variables.

Objetivo:
- entender quais grupos clínicos fornecem ganho ao componente visual.

---

# 8. Estratégias de treinamento e repetição

## Seeds

Todos os experimentos finais principais usarão:

- 42;
- 7;
- 123.

O split permanecerá fixo.

Isso permite separar:

- variabilidade de otimização;
- variabilidade causada por diferentes partições.

Experimentos exploratórios de seleção poderão inicialmente usar apenas seed 42.

Depois de escolhida a configuração, os três seeds serão executados.

Essa estratégia reduz significativamente o tempo total de GPU.

---

# 9. Métricas

## Five-class grading

- Macro-F1;
- Balanced Accuracy;
- Quadratic Weighted Kappa;
- per-class recall;
- confusion matrix.

## Referable DR

- AUROC;
- AUPRC;
- sensitivity;
- specificity;
- F1;
- Balanced Accuracy.

## Calibration

Para os modelos finais:

- Brier score;
- calibration curve;
- Expected Calibration Error, se estatisticamente estável.

---

# 10. Cross-domain experiments

A generalização será explicitamente quantificada.

Para cada modelo final:

### Internal performance
BRSET test.

### External performance
UNESP.

Será calculado:

\[
\Delta_{domain} =
Metric_{BRSET} - Metric_{UNESP}
\]

Isso permitirá responder não apenas qual modelo tem maior desempenho, mas qual modelo sofre menor degradação fora do domínio de desenvolvimento.

Comparações principais:

- A-best internal vs external;
- B-best internal vs external;
- C-best internal vs external.

Hipótese:

\[
\Delta_{domain,C} < \Delta_{domain,A}
\]

ou seja, multimodalidade poderá reduzir a perda sob domain shift.

---

# 11. Image-quality experiments

Image quality será analisada como fator de robustez.

BRSET possui informações de qualidade e UNESP terá avaliação clínica de gradabilidade.

Os modelos finais A e C serão avaliados em:

- good-quality images;
- borderline-quality images;
- poor/ungradable images, quando clinicamente apropriado.

Pergunta:

> O ganho multimodal aumenta quando a informação visual se torna menos confiável?

Esse é um experimento particularmente interessante porque fornece uma hipótese clínica clara para o valor da segunda modalidade.

---

# 12. Class-imbalance analysis

Além das métricas globais, o desempenho será analisado por frequência de classe.

Serão definidos:

- head classes;
- intermediate classes;
- tail classes.

Comparar:

- recall por classe;
- macro-F1;
- severe-class recall.

Essa análise reaproveita a experiência metodológica acumulada no projeto anterior sem transformar FERS/FUFA na contribuição principal.

---

# 13. Clinical error analysis

Depois que todos os modelos estiverem bloqueados, os oftalmologistas revisarão:

- todos os false-negative referable DR;
- todos os erros com diferença de pelo menos dois graus;
- amostra de false positives;
- amostra de classificações corretas.

Categorias:

1. image-quality problem;
2. subtle lesion;
3. borderline grade;
4. possible reference-label disagreement;
5. lesion obscured by acquisition;
6. coexisting retinal disease;
7. clinically plausible adjacent-grade disagreement;
8. unexplained model error.

Essa etapa será tratada como resultado científico principal.

---

# 14. Matriz experimental proposta

## Model A — Image-only

| ID | Experiment | Purpose |
|---|---|---|
| A0 | Existing public retinal checkpoint | Direct transfer baseline |
| A1 | ImageNet → BRSET | BRSET-only image baseline |
| A2 | RetinaMNIST+APTOS → BRSET | Effect of retinal pretraining |
| A3a | A2 + standard CE | Standard imbalance condition |
| A3b | A2 + class weighting | Imbalance-aware baseline |
| A3c | A2 + focal loss | Hard-example loss baseline |
| A4 | Alternative foundation encoder | Representation robustness |

## Model B — Clinical-only

| ID | Experiment | Purpose |
|---|---|---|
| B0 | Logistic regression | Linear clinical baseline |
| B1 | Gradient boosting / RF | Non-linear tabular baseline |
| B2 | Clinical MLP | Neural clinical representation |
| B3 | Feature-group ablation | Clinical contribution analysis |
| B4 | Missingness ablation | Missing-data robustness |

## Model C — Multimodal

| ID | Experiment | Purpose |
|---|---|---|
| C0 | Frozen late fusion | Clean multimodal baseline |
| C1 | Best A encoder + late fusion | Main multimodal model |
| C2 | Joint fine-tuning | Co-adaptation test |
| C3 | Gated fusion | Alternative fusion mechanism |
| C4 | Modality ablation | Verify use of both modalities |
| C5 | Metadata-group ablation | Identify clinical contribution |

---

# 15. Priorização para manter o prazo razoável

## Tier 1 — obrigatório

- A0
- A1
- A2
- A3b
- B0
- B1
- B2
- C0
- C1
- C2
- BRSET internal test
- UNESP external test
- clinical error analysis

## Tier 2 — altamente desejável

- A3c
- A4
- B3
- B4
- C3
- C4
- image-quality stratification

## Tier 3 — somente se houver tempo

- C5
- extensive calibration experiments
- alternative foundation encoders beyond one additional model
- extensive subgroup analyses

Assim, o artigo pode ser concluído com Tier 1, mas existe uma expansão planejada caso os resultados iniciais indiquem necessidade de mais evidência.

---

# 16. Resultados principais esperados no artigo

O artigo deverá responder quatro perguntas:

1. Quanto o desempenho visual cai ao transferir para a coorte UNESP?
2. Quanto da informação de DR pode ser inferida apenas dos dados clínicos?
3. A combinação multimodal melhora o desempenho externo em relação ao melhor image-only model?
4. Em quais condições — classes raras, baixa qualidade de imagem ou determinados perfis clínicos — o ganho multimodal é maior?

---

# 17. Contribuição científica pretendida

A contribuição não será simplesmente “um modelo multimodal para DR”.

O artigo será estruturado como um estudo de **generalização clínica multimodal**:

> **A controlled evaluation of whether structured clinical information complements retinal image representations and improves robustness to cross-dataset domain shift in diabetic retinopathy grading.**

O trabalho combina:

- múltiplos domínios públicos de retina;
- um grande dataset multimodal brasileiro;
- uma coorte clínica externa;
- comparação image-only / clinical-only / multimodal;
- análise de qualidade;
- análise de class imbalance;
- avaliação clínica dos erros.

Isso fornece um escopo computacional mais completo sem conflitar com o segundo artigo metodológico sobre FUFA.
