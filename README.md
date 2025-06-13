# tarefa 4 - Relatório


# ✅ **Exercício 1 – Observação de Comportamento com Semáforo Contador**

> Objetivo: Avaliar o funcionamento de tarefas que compartilham um semáforo contador.
> 

---

### 🧾 **Resumo da implementação:**

- Foi criado um **semáforo contador com valor inicial = 2** .
- Três tarefas (`Task1`, `Task2`, `Task3`) tentam adquirir (`osSemaphoreAcquire`) o semáforo, **sem nunca liberá-lo** (`osSemaphoreRelease` não é chamado).
- Todas as tarefas são configuradas com **mesma prioridade** e realizam um `osDelay(500)` após adquirir.

---

### 🔍 **Comportamento observado:**

- Ao iniciar a execução, o semáforo possui **2 unidades disponíveis**.
- A primeira tarefa a ser agendada (`Task3`, no experimento) faz `Acquire` → semáforo: 2 → 1.
- A segunda tarefa (`Task1`, no experimento) também faz `Acquire` → semáforo: 1 → 0.
- A terceira tarefa (`Task2`) **bloqueia** no `osSemaphoreAcquire`, pois o semáforo está com valor 0.
- A tarefa que chegou por último nunca mais avança, pois nenhuma outra faz `Release()`.

---

### 🛠️ **Depuração com `osSemaphoreGetCount()`**

- Foi utilizado `osSemaphoreGetCount()` após o `Acquire()` para **monitorar o valor do semáforo**.
- Observou-se que:
    - Quando a primeira tarefa entra: `cont = 0`
    - Quando a primeira tarefa sai: `cont = 1`
    - Quando a segunda tarefa entra: `cont = 1`
    - Quando a segunda tarefa sai: `cont = 0`
    - A terceira tarefa bloqueia no `Acquire()` e nunca chega a imprimir o valor

---

### 💡 **Conclusão técnica:**

- O semáforo contador **permite acesso concorrente controlado**, até o limite configurado (`MaxCount`).
- Quando o valor do semáforo atinge 0, **tarefas adicionais bloqueiam até que alguém libere (release)**.
- Como nenhuma tarefa faz `Release()`, o sistema não executa mais nenhuma tarefa.

# ✅ **Exercício 2 – Liberação de Semáforo via Interrupção**

> Objetivo: Sincronizar tarefas com liberação do semáforo por interrupção externa (botão USER).
> 

---

### 🧾 **Resumo da implementação:**

- Criado um **semáforo binário com valor inicial 0**, usando `osSemaphoreNew(1, 0, NULL)`.
- Três tarefas foram implementadas (`Task1`, `Task2`, `Task3`) e **todas fazem `osSemaphoreAcquire()`**, aguardando indefinidamente (`osWaitForever`).
- O botão USER (PC13) foi configurado para gerar uma **interrupção externa** .
- Dentro da função `HAL_GPIO_EXTI_Callback()`, foi inserido `osSemaphoreRelease(semHandle)` para liberar o semáforo sempre que o botão for pressionado.

---

### 🔍 **Comportamento observado:**

- Ao iniciar a execução, **nenhuma tarefa é executada**, pois o semáforo está em 0.
- Quando o botão é pressionado:
    - **Uma tarefa é desbloqueada**, executa seu código e imprime no terminal.
- A cada nova pressão do botão:
    - **Mais uma tarefa é desbloqueada**, em ordem de espera ou conforme escalonador decidir.
- O sistema se comporta como esperado: **as tarefas só executam quando o semáforo é liberado por uma interrupção externa**.

---

### 🛠️ **Depuração com `osSemaphoreGetCount()`**

- O uso de `osSemaphoreGetCount()` permitiu confirmar que o valor do semáforo:
    - Está inicialmente em `0`
    - Vai para `1` no momento do `Release()` no callback
    - Volta para `0` logo após o `Acquire()` de uma tarefa
- Isso confirma que o semáforo binário está operando como **gatilho controlado via evento externo**.

---

### 💡 **Conclusão técnica:**

- A liberação de tarefas com base em evento externo (interrupção) é uma **forma eficiente e comum de sincronização** em sistemas embarcados.
- Um **único semáforo binário** foi suficiente para controlar o desbloqueio de múltiplas tarefas, **uma de cada vez**, ao pressionar o botão.
- Isso garante que as tarefas só executem **quando há uma solicitação explícita do ambiente externo** (pressão do botão).
- O sistema demonstrou que semáforos binários são úteis para **sincronização por eventos**, enquanto semáforos contadores são usados para **controle de recursos**.

---

# ✅ **Exercício 3 – Teste de Disponibilidade e Liberação Manual de Semáforo**

> Objetivo: Demonstrar que o semáforo contador não garante exclusividade no uso do recurso, permitindo que qualquer tarefa libere o semáforo, mesmo sem tê-lo adquirido anteriormente.
> 

---

### 🧾 **Resumo da implementação:**

- Foi utilizado um **semáforo contador** com valor inicial `0`, criado com `osSemaphoreNew(2, 0, NULL)`.
- A tarefa `Task1` realiza a seguinte lógica:
    - Verifica o valor do semáforo com `osSemaphoreGetCount()`.
    - Se o contador estiver em 0, **ela mesma faz `osSemaphoreRelease()`**, mesmo sem ter adquirido o recurso antes.
    - Em seguida, faz `osSemaphoreAcquire()` e executa sua ação.
- As tarefas `Task2` e `Task3` apenas aguardam com `osSemaphoreAcquire()` e executam sua ação se conseguirem adquirir o semáforo.

---

### 🔍 **Comportamento observado:**

- O semáforo inicia com valor `0`, então nenhuma tarefa conseguiria prosseguir inicialmente.
- `Task1`, ao detectar que o valor do semáforo está em zero, **faz `Release()` manualmente**, e depois imediatamente faz `Acquire()`.
- Isso permite que `Task1` acesse o recurso **mesmo sem ter "posse legítima"**, simulando um acesso forçado.
- Enquanto isso, `Task2` e `Task3` **bloqueiam no `Acquire()`**, pois não há outras liberações ocorrendo.

---

### 🛠️ **Depuração com `osSemaphoreGetCount()`**

- O uso de `osSemaphoreGetCount()` dentro da `Task1` mostrou que:
    - O semáforo inicia com valor `0`
    - A `Task1` percebe isso, imprime no terminal e faz `Release()`
    - O contador sobe para `1`, permitindo que ela imediatamente execute o `Acquire()` com sucesso
- O ciclo se repete indefinidamente, com `Task1` simulando uma liberação e retomando o controle.

---

### 💡 **Conclusão técnica:**

- O experimento demonstra claramente que **semáforos contadores não implementam controle de posse**: qualquer tarefa pode fazer `osSemaphoreRelease()`, independentemente de ter feito `Acquire()` antes.
- Isso **pode gerar comportamento indevido** se o acesso ao recurso depender de controle rigoroso de exclusividade.
- Em contraste, um **mutex** possui regras de posse: **apenas a tarefa que fez `Acquire()` pode fazer `Release()`**. Se outra tentar, o sistema retorna erro (`osErrorResource`).
- Logo, **semáforo contador é adequado para controle de quantidade de recursos**, mas **não garante exclusividade nem proteção contra liberações indevidas**.

---

### 🔄 **Comparação entre o semáforo contador e o mutex:**

| Característica | **Semáforo Contador** | **Mutex** |
| --- | --- | --- |
| Liberação por outra tarefa | ✅ Qualquer tarefa pode chamar `osSemaphoreRelease()` | ❌ Apenas a **tarefa que fez `Acquire()`** pode chamar `Release()` |
| Controle de exclusividade | ❌ Não garante exclusividade real | ✅ Garante exclusividade: só **um dono por vez** |
| Uso típico | Controle de **quantidade de recursos simultâneos** | Controle de **acesso exclusivo a um recurso compartilhado** |
| Verificação de erro | ❌ Nenhum erro ao liberar sem ter adquirido | ✅ `osMutexRelease()` retorna `osErrorResource` se não for o dono |
| Contador interno | ✅ Sim (permite múltiplas unidades disponíveis) | ❌ Não (apenas bloqueado ou disponível) |
| Inversão de prioridade (priority inheritance) | ❌ Não implementado | ✅ Implementado, útil para evitar *priority inversion* |

---

## ✅ **Relatório – Questão 4**

### 🧩 **Objetivo da Questão**

Demonstrar o funcionamento e a **restrição de uso do recurso mutex (mutual exclusion)** no FreeRTOS, simulando uma tentativa inválida de liberar o mutex por uma tarefa que não o possui.

---

### ⚙️ **Recursos RTOS utilizados**

- **2 tarefas** com a **mesma prioridade**:
    - `Task01`: adquire o mutex.
    - `Task02`: tenta liberar o mutex sem ter adquirido.
- **1 mutex**: `Mutex01Handle`.

---

### 🧪 **Comportamento implementado**

### **Task 1 (Task01)**

- Executa primeiro e **adquire o mutex** com `osMutexAcquire`.
- Após adquirir, envia pela UART a mensagem:
    
    ```
    Task1: mutex adquirido.
    
    ```
    
- Permanece em loop com `osDelay`, **mantendo o mutex em posse**.

### **Task 2 (Task02)**

- Aguarda 2 segundos para garantir que `Task01` já adquiriu o mutex.
- Tenta **liberar o mutex sem tê-lo adquirido**, o que é **proibido** pelo FreeRTOS.
- O código tenta detectar esse comportamento via:
    
    ```c
    result = osMutexRelease(Mutex01Handle);
    
    ```
    
    e imprime o valor de retorno.
    

---

### ❌ **Problema identificado**

- **O sistema trava** após a tentativa de `Task02` liberar o mutex.
- Isso ocorre porque, conforme a documentação do CMSIS-RTOS2 e FreeRTOS, **não é permitido liberar um mutex que não foi adquirido pela tarefa corrente**.
- **Ao violar essa regra, o sistema pode entrar em estado de erro irreversível** (comportamento indefinido).

---

### 🧠 **Conceitos explorados**

- O **mutex** é um recurso de exclusão mútua que garante que **apenas uma tarefa possa acessar uma região crítica por vez**.
- **Somente a tarefa que adquiriu o mutex pode liberá-lo**.
- Isso é diferente dos **semáforos binários**, que podem ser adquiridos por uma tarefa e liberados por outra.

---

### 🔎 **Observações adicionais**

- O sistema poderia ter identificado o erro usando o valor de retorno `osErrorResource`, mas o travamento impediu a finalização da função.
- Para testes futuros, recomenda-se:
    - **Não liberar mutex sem adquiri-lo**.
    - Utilizar logs antes da chamada para facilitar a depuração.
    - Monitorar com `osMutexGetOwner` (se disponível) ou estratégias alternativas.

---

## ✅ **Relatório – Questão 5**

### 🧩 **Objetivo da Questão**

Implementar um sistema sequencial de aquisição, processamento e envio de dados usando **semáforos binários** e **FreeRTOS**, em que:

- O ciclo é reiniciado pelo **botão USER**;
- Dados são simulados pelo ADC;
- Após 10 ciclos, o sistema aguarda nova interação do usuário para reinício.

---

### 🔄 **Fluxo das tarefas**

1. **Botão pressionado**
    - Interrupção libera `semStartHandle`.
    - Zera o contador `envio_contador` e sinaliza `processo_ativo = true`.
2. **`TaskADC`**
    - Aguarda `semStartHandle`.
    - Simula 100 leituras do ADC.
    - Libera `semCalcHandle`.
3. **`TaskCalc`**
    - Aguarda `semCalcHandle`.
    - Calcula a média das 100 leituras.
    - Libera `semSendHandle`.
4. **`TaskSend`**
    - Aguarda `semSendHandle`.
    - Envia a média pela UART.
    - Incrementa `envio_contador`.
    
    **Se `envio_contador < 10`:**
    
    - Libera `semStartHandle` para um novo ciclo automático.
    
    **Se `envio_contador == 10`:**
    
    - Exibe mensagem de fim de processo.
    - Sistema aguarda nova pressão do botão para reiniciar.

---

### ⚙️ **Recursos RTOS utilizados**

- **3 tasks** com mesma prioridade: `TaskADC`, `TaskCalc`, `TaskSend`
- **3 semáforos binários**:
    - `semStartHandle`
    - `semCalcHandle`
    - `semSendHandle`
- **1 interrupção externa** no botão USER (GPIO 13) acionando o ciclo

---

### 🧪 **Simulação do ADC**

- Os valores do ADC são simulados com `rand() % 4096`.
- O `osDelay(10)` simula o tempo de leitura de hardware real.

---

### 🧠 **Conceitos explorados**

- Coordenação de tarefas usando semáforos binários.
- Controle de fluxo sequencial e reinicialização por evento externo.
- Sincronização entre tarefas sem uso de prioridades diferentes.
- Uso de `osSemaphoreRelease` e `osSemaphoreAcquire` como gatilhos.

---

# Exercicio 6:

---

### ✅ **Questão 6 – Coleta, Processamento e Histórico de Leituras ADC com 4 Tarefas**

---

### 🔧 **Objetivo da Questão**

Implementar uma aplicação em FreeRTOS na qual quatro tarefas cooperam utilizando **semáforos binários** para realizar, em sequência:

1. Leitura de 100 amostras simuladas de ADC.
2. Cálculo da média dessas amostras.
3. Armazenamento da média em um vetor circular com 10 posições.
4. Envio da média pela UART.

---

### ⚙️ **Descrição do Funcionamento**

- A execução é iniciada quando o **botão USER (GPIO13)** é pressionado.
- A **TaskADC** simula as leituras de ADC (valores randômicos entre 0 e 4095).
- A **TaskCalc** aguarda a liberação via `semCalcHandle`, calcula a média das 100 amostras e sinaliza `semSaveHandle`.
- A **TaskSave** grava a média no vetor `historico[10]`, atualizando com índice circular, e libera o semáforo `semSendHandle`.
- A **TaskSend** converte a média (float) em inteiro + decimal para evitar `%f`, e envia via UART.

---

### 🧵 **Threads (Tarefas) Criadas**

| Tarefa | Função associada | Prioridade | Ação realizada |
| --- | --- | --- | --- |
| TaskADC | `TaskADCfun` | `osPriorityNormal` | Simula leitura ADC e aciona semáforo de cálculo |
| TaskCalc | `TaskCalcfun` | `osPriorityNormal` | Calcula média e aciona semáforo de salvar |
| TaskSave | `TaskSavefun` | `osPriorityNormal` | Armazena no vetor histórico e aciona sem enviar |
| TaskSend | `TaskSendfun` | `osPriorityNormal` | Envia resultado pela UART |

---

### 🔁 **Sincronização entre tarefas**

Utiliza 4 semáforos binários criados com `count = 0`:

| Semáforo | Usado para sinalizar | Quando é liberado? |
| --- | --- | --- |
| `semStart` | Início do ciclo (ADC) | Pressionar botão USER |
| `semCalc` | Após leitura ADC | TaskADC → TaskCalc |
| `semSave` | Após cálculo da média | TaskCalc → TaskSave |
| `semSend` | Após salvar histórico | TaskSave → TaskSend |

---

### 🧠 **Simulação de Leitura ADC**

Como o ADC real não pode ser lido sem entrada analógica:

```c
bufferADC[i] = rand() % 4096;

```

Cada valor simula uma conversão de 12 bits.

---

### 🔠 **Tratamento do float na UART**

Evita `%f` usando divisão manual:

```c
int parte_inteira = (int)mediaADC;
int parte_decimal = (int)((mediaADC - parte_inteira) * 100);
sprintf(msg, "Media enviada: %d.%02d\r\n", parte_inteira, parte_decimal);

```

---

### 📌 **Pontos importantes**

- Todas as tarefas têm **mesma prioridade**, não há preempção.
- A estrutura sequencial das tarefas depende **exclusivamente dos semáforos**.
- Código pode ser expandido futuramente com botão de reset do histórico ou transmissão completa do vetor `historico`.

---
