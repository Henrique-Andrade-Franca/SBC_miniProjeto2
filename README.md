# Triagem Médica de Urgência

<a href="https://colab.research.google.com/github/Henrique-Andrade-Franca/SBC_miniProjeto2/blob/main/mini_projeto_2.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>  

**Disciplina:** Sistemas Baseados em Conhecimento (SBC)  
**Autor:** Henrique de Andrade França

## Descrição do Domínio
Este projeto implementa um Sistema Baseado em Conhecimento utilizando **Lógica Fuzzy** (Inocência de Mamdani) para realizar a **Triagem Médica** de pacientes num serviço de urgência.

O domínio médico é o cenário perfeito para a lógica fuzzy porque as medições corporais não operam em lógicas binárias (*crisp*). Existe uma transição gradual e imprecisa. O sistema captura essa imprecisão avaliando duas variáveis de entrada contínuas: a **Temperatura Corporal** e a **Saturação de Oxigénio (SpO2)**, gerando como saída um **Score de Prioridade** suave e proporcional (de 0 a 100), assemelhando-se ao raciocínio de um enfermeiro a aplicar o Protocolo de Manchester.

O projeto foi construído em Python utilizando a biblioteca `scikit-fuzzy`.

---

## Variáveis e Conjuntos Fuzzy
O modelo é composto por pelo menos 3 conjuntos fuzzy por variável, garantindo transições suaves:
* **Entrada 1 (Temperatura):** `normal` (até 37.5 ºC), `febril` (pico nos 37.8 ºC), `alta` (acima de 38 ºC).
* **Entrada 2 (Saturação de O2):** `critica` (abaixo de 92%), `alerta` (90% a 95%), `normal` (acima de 94%).
* **Saída (Prioridade):** `verde` (baixo risco, pode aguardar), `amarela` (urgência moderada), `vermelha` (emergência imediata).

---

## Base de Regras e Justificação Clínica
O motor de inferência é composto por 9 regras que cobrem exaustivamente o espaço de estados combinando os sintomas de Temperatura e Saturação. A justificação baseia-se em princípios de triagem clínica e no Protocolo de Manchester:

### Submatriz 1: Paciente Sem Febre (Temperatura Normal)
* **R1:** SE a temperatura é `normal` E a saturação é `normal` ENTÃO a prioridade é `verde`.
  * *Justificação:* Caso ambulatorial simples. O paciente não apresenta nenhum dos dois sinais vitais alterados; atendimento de rotina.
* **R2:** SE a temperatura é `normal` E a saturação está em `alerta` ENTÃO a prioridade é `amarela`.
  * *Justificação:* Necessita investigação. Embora esteja sem febre, a queda inicial da oxigenação indica um quadro potencialmente instável que precisa ser monitorado.
* **R3:** SE a temperatura é `normal` E a saturação é `critica` ENTÃO a prioridade é `vermelha`.
  * *Justificação:* Hipóxia grave isolada (ex: crise de asma aguda, engasgo ou embolia). Há risco de vida imediato por falta de oxigenação, independente da temperatura.

### Submatriz 2: Paciente Febril Leve (Temperatura Febril)
* **R4:** SE a temperatura é `febril` E a saturação é `normal` ENTÃO a prioridade é `verde`.
  * *Justificação:* Sintoma leve (ex: virose comum ou resfriado). O paciente apresenta uma linha sutil de febre, mas como a respiração está perfeita, pode aguardar atendimento com segurança.
* **R5:** SE a temperatura é `febril` E a saturação está em `alerta` ENTÃO a prioridade é `amarela`.
  * *Justificação:* Risco moderado de evolução para desconforto respiratório. O paciente encontra-se num estado limítrofe em ambas as variáveis, exigindo vigilância ativa.
* **R6:** SE a temperatura é `febril` E a saturação é `critica` ENTÃO a prioridade é `vermelha`.
  * *Justificação:* Emergência médica. O comprometimento respiratório evidente associado a um processo febril indica uma resposta inflamatória ou infecciosa aguda que exige intervenção imediata.

### Submatriz 3: Paciente com Febre Alta (Temperatura Alta)
* **R7:** SE a temperatura é `alta` E a saturação é `normal` ENTÃO a prioridade é `amarela`.
  * *Justificação:* Exige medicação intravenosa urgente para controle térmico (risco de convulsão ou desidratação), mas o pulmão e a mecânica respiratória ainda estão estáveis.
* **R8:** SE a temperatura é `alta` E a saturação está em `alerta` ENTÃO a prioridade é `vermelha`.
  * *Justificação:* Quadro infeccioso sistêmico avançado. A febre alta somada à dificuldade respiratória inicial sugere um forte comprometimento pulmonar com risco de rápida piora.
* **R9:** SE a temperatura é `alta` E a saturação é `critica` ENTÃO a prioridade é `vermelha`.
  * *Justificação:* Estado gravíssimo de urgência (ex: pneumonia severa ou choque séptico iminente). O paciente apresenta falência respiratória severa combinada com hipertermia, correndo risco iminente de morte.

---

## Casos de Teste e Análise de Inferência

### Teste 1: Paciente Saudável / Rotina
* **Valores de Entrada:** Temperatura = 36.8 ºC | Saturação = 99%
* **Análise:** O sistema mapeia os valores com 100% de pertinência nos conjuntos `normal`. A **Regra 1** é ativada em pleno.
* **Saída:** Score de prioridade baixo (~15.0). O paciente recebe a classificação **Verde** e aguarda o fluxo normal de atendimento.

### Teste 2: Paciente no Limiar (O Poder da Lógica Fuzzy)
* **Valores de Entrada:** Temperatura = 37.2 ºC | Saturação = 94.2%
* **Análise:** Este é o cenário onde a lógica binária falharia. O paciente não está perfeitamente normal, nem criticamente doente. O sistema fuzzy ativa os conjuntos `normal` e `febril` para a temperatura, e `alerta` e `normal` para o oxigénio de forma fracionada. Múltiplas regras (R1, R2, R4 e R5) são ativadas simultaneamente com forças diferentes. O método do centroide agrega estas ativações para gerar uma saída justa.
* **Saída:** Score de prioridade intermédio (~46.30), puxando a decisão para a transição entre Verde e Amarela.

### Teste 3: Paciente Crítico (Emergência)
* **Valores de Entrada:** Temperatura = 39.0 ºC | Saturação = 85%
* **Análise:** A saturação de 85% pertence integralmente ao conjunto `critica`. Isto faz com que a **Regra 9** dispare.
* **Saída:** Score de prioridade extremamente alto (~83.50). O paciente é classificado como **Vermelho** e encaminhado diretamente para a sala de reanimação/emergência.
