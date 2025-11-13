# Flowise AI Prompts Documentation

Полная документация всех AI промтов, используемых в приложении Flowise.

## Содержание

1. [AGENTFLOW PROMPTS](#agentflow-prompts) - Промты для управления потоком агентов
2. [CHAIN PROMPTS](#chain-prompts) - Промты для цепочек обработки
3. [AGENT PROMPTS](#agent-prompts) - Системные промты для различных типов агентов
4. [RETRIEVER PROMPTS](#retriever-prompts) - Промты для поиска и извлечения информации
5. [API & TEMPLATE PROMPTS](#api--template-prompts) - Промты для API и шаблонов

---

## AGENTFLOW PROMPTS

Промты для управления потоком работы агентов, включая суммаризацию диалогов, обработку пользовательского ввода и маршрутизацию между сценариями.

**Файл:** `packages/components/nodes/agentflow/prompt.ts`

### 1. DEFAULT_SUMMARIZER_TEMPLATE

**Назначение:** Используется для прогрессивной суммаризации диалогов между пользователем и AI в процессе работы агента. Промт помогает создавать краткое резюме беседы, которое обновляется по мере продолжения диалога.

**Применение:** Применяется в узлах AgentFlow для сохранения контекста длинных диалогов без необходимости хранить всю историю сообщений.

**Переменные:**
- `{conversation}` - текущая беседа для суммаризации

**Промт:**
```
Progressively summarize the conversation provided and return a new summary.

EXAMPLE:
Human: Why do you think artificial intelligence is a force for good?
AI: Because artificial intelligence will help humans reach their full potential.

New summary:
The human asks what the AI thinks of artificial intelligence. The AI thinks artificial intelligence is a force for good because it will help humans reach their full potential.
END OF EXAMPLE

Conversation:
{conversation}

New summary:
```

### 2. DEFAULT_HUMAN_INPUT_DESCRIPTION

**Назначение:** Промт для суммаризации беседы с последующим запросом обратной связи от пользователя. Используется для создания контрольных точек в диалоге, где пользователь может подтвердить направление работы или внести коррективы.

**Применение:** Применяется в узлах с человеческим участием (Human-in-the-loop), чтобы пользователь мог контролировать процесс работы агента.

**Переменные:** Нет явных переменных, промт работает с контекстом текущей беседы.

**Промт:**
```
Summarize the conversation between the user and the assistant, reiterate the last message from the assistant, and ask if user would like to proceed or if they have any feedback.
- Begin by capturing the key points of the conversation, ensuring that you reflect the main ideas and themes discussed.
- Then, clearly reproduce the last message sent by the assistant to maintain continuity. Make sure the whole message is reproduced.
- Finally, ask the user if they would like to proceed, or provide any feedback on the last assistant message

## Output Format The output should be structured in three parts in text:
- A summary of the conversation (1-3 sentences).
- The last assistant message (exactly as it appeared).
- Ask the user if they would like to proceed, or provide any feedback on last assistant message. No other explanation and elaboration is needed.
```

### 3. CONDITION_AGENT_SYSTEM_PROMPT

**Назначение:** Системный промт для Condition Agent - специализированного агента, который анализирует входные данные и выбирает подходящий сценарий из предоставленного набора. Используется для маршрутизации запросов между различными обработчиками.

**Применение:** Применяется в условных узлах (Condition Nodes) для создания разветвленных агентских систем, где входящий запрос направляется к соответствующему обработчику на основе его содержания.

**Переменные:**
- `input` - входная строка с запросом пользователя
- `scenarios` - список предопределенных сценариев
- `instruction` - инструкция для выбора сценария

**Формат вывода:** JSON объект вида `{"output": "<selected_scenario_name>"}`

**Промт:**
```html
<p>You are part of a multi-agent system designed to make agent coordination and execution easy. Your task is to analyze the given input and select one matching scenario from a provided set of scenarios.</p>
    <ul>
        <li><strong>Input</strong>: A string representing the user's query, message or data.</li>
        <li><strong>Scenarios</strong>: A list of predefined scenarios that relate to the input.</li>
        <li><strong>Instruction</strong>: Determine which of the provided scenarios is the best fit for the input.</li>
    </ul>
    <h2>Steps</h2>
    <ol>
        <li><strong>Read the input string</strong> and the list of scenarios.</li>
        <li><strong>Analyze the content of the input</strong> to identify its main topic or intention.</li>
        <li><strong>Compare the input with each scenario</strong>: Evaluate how well the input's topic or intention aligns with each of the provided scenarios and select the one that is the best fit.</li>
        <li><strong>Output the result</strong>: Return the selected scenario in the specified JSON format.</li>
    </ol>
    <h2>Output Format</h2>
    <p>Output should be a JSON object that names the selected scenario, like this: <code>{"output": "<selected_scenario_name>"}</code>. No explanation is needed.</p>
    <h2>Examples</h2>
    <ol>
       <li>
            <p><strong>Input</strong>: <code>{"input": "Hello", "scenarios": ["user is asking about AI", "user is not asking about AI"], "instruction": "Your task is to check if the user is asking about AI."}</code></p>
            <p><strong>Output</strong>: <code>{"output": "user is not asking about AI"}</code></p>
        </li>
        <li>
            <p><strong>Input</strong>: <code>{"input": "What is AIGC?", "scenarios": ["user is asking about AI", "user is asking about the weather"], "instruction": "Your task is to check and see if the user is asking a topic about AI."}</code></p>
            <p><strong>Output</strong>: <code>{"output": "user is asking about AI"}</code></p>
        </li>
        <li>
            <p><strong>Input</strong>: <code>{"input": "Can you explain deep learning?", "scenarios": ["user is interested in AI topics", "user wants to order food"], "instruction": "Determine if the user is interested in learning about AI."}</code></p>
            <p><strong>Output</strong>: <code>{"output": "user is interested in AI topics"}</code></p>
        </li>
    </ol>
    <h2>Note</h2>
    <ul>
        <li>Ensure that the input scenarios align well with potential user queries for accurate matching.</li>
        <li>DO NOT include anything other than the JSON in your response.</li>
    </ul>
```

---

