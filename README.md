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
O motor de inferência é composto por 6 regras que combinam os sintomas. A justificação baseia-se em princípios básicos de triagem:

* **R1:** SE a saturação é `critica` ENTÃO a prioridade é `vermelha`.
  * *Justificação:* A falta de oxigenação no sangue é um risco de vida imediato e anula qualquer outra variável. O paciente precisa de suporte respiratório imediato.
* **R2:** SE a temperatura é `alta` E a saturação está em `alerta` ENTÃO a prioridade é `vermelha`.
  * *Justificação:* Febre muito alta combinada com dificuldade respiratória inicial sugere um quadro de infeção pulmonar grave com rápida evolução.
* **R3:** SE a temperatura é `alta` E a saturação é `normal` ENTÃO a prioridade é `amarela`.
  * *Justificação:* A febre é preocupante e exige medicação intravenosa (urgente), mas como o paciente respira bem, não há risco imediato de paragem respiratória.
* **R4:** SE a temperatura é `febril` E a saturação está em `alerta` ENTÃO a prioridade é `amarela`.
  * *Justificação:* O paciente está num estado limítrofe em ambas as variáveis, exigindo vigilância moderada para evitar que o quadro piore.
* **R5:** SE a temperatura é `febril` E a saturação é `normal` ENTÃO a prioridade é `verde`.
  * *Justificação:* É um quadro clássico de constipação leve. O paciente pode aguardar na sala de espera sem grandes riscos.
* **R6:** SE a temperatura é `normal` E a saturação é `normal` ENTÃO a prioridade é `verde`.
  * *Justificação:* O paciente não apresenta nenhum dos dois sinais vitais alterados; atendimento de rotina.

---

## Casos de Teste e Análise de Inferência

### Teste 1: Paciente Saudável / Rotina
* **Valores de Entrada:** Temperatura = 36.8 ºC | Saturação = 99%
* **Análise:** O sistema mapeia os valores com 100% de pertinência nos conjuntos `normal`. A **Regra 6** é ativada em pleno.
* **Saída:** Score de prioridade baixo (~22.50). O paciente recebe a classificação **Verde** e aguarda o fluxo normal de atendimento.

### Teste 2: Paciente no Limiar (O Poder da Lógica Fuzzy)
* **Valores de Entrada:** Temperatura = 37.6 ºC | Saturação = 94%
* **Análise:** Este é o cenário onde a lógica binária falharia. O paciente não está perfeitamente normal, nem criticamente doente. O sistema fuzzy ativa os conjuntos `normal` e `febril` para a temperatura, e `alerta` e `normal` para o oxigénio de forma fracionada. Múltiplas regras (R4, R5 e R6) são ativadas simultaneamente com forças diferentes (ex: 0.20, 0.40). O método do centroide agrega estas ativações para gerar uma saída justa.
* **Saída:** Score de prioridade intermédio (~31.00), puxando a decisão para a transição entre Verde e Amarela.

### Teste 3: Paciente Crítico (Emergência)
* **Valores de Entrada:** Temperatura = 39.0 ºC | Saturação = 85%
* **Análise:** A saturação de 85% pertence integralmente ao conjunto `critica`. Isto faz com que a **Regra 1** dispare com força máxima (1.0).
* **Saída:** Score de prioridade extremamente alto (~77.50). O paciente é classificado como **Vermelho** e encaminhado diretamente para a sala de reanimação/emergência.
