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

## CHAIN PROMPTS

Промты для цепочек обработки (Chains) - последовательностей операций для работы с документами, базами данных, API и диалогами.

**Основные файлы:**
- `packages/components/nodes/chains/ConversationalRetrievalQAChain/prompts.ts`
- `packages/components/nodes/chains/ConversationChain/ConversationChain.ts`

### 1. CUSTOM_QUESTION_GENERATOR_CHAIN_PROMPT

**Назначение:** Преобразует follow-up вопросы пользователя в самостоятельные (standalone) вопросы, которые можно понять без контекста предыдущей беседы. Это критически важно для корректного поиска информации в векторных базах данных.

**Применение:** Используется в ConversationalRetrievalQAChain для переформулирования вопросов перед отправкой в retriever.

**Переменные:**
- `{chat_history}` - история предыдущих сообщений
- `{question}` - текущий вопрос пользователя

**Промт:**
```
Given the following conversation and a follow up question, rephrase the follow up question to be a standalone question, answer in the same language as the follow up question. include it in the standalone question.

Chat History:
{chat_history}
Follow Up Input: {question}
Standalone question:
```

### 2. RESPONSE_TEMPLATE

**Назначение:** Системный промт для взаимодействия с документами в режиме вопрос-ответ. Определяет поведение AI ассистента при работе с предоставленным контекстом.

**Применение:** Используется в ConversationalRetrievalQAChain для генерации ответов на основе найденных документов.

**Переменные:**
- `{context}` - релевантный контекст из документов

**Особенности:**
- AI отвечает только на основе предоставленного контекста
- При отсутствии релевантной информации отвечает "Hmm, I'm not sure"
- Запрещает выдумывать информацию

**Промт:**
```
I want you to act as a document that I am having a conversation with. Your name is "AI Assistant". Using the provided context, answer the user's question to the best of your ability using the resources provided.
If there is nothing in the context relevant to the question at hand, just say "Hmm, I'm not sure" and stop after that. Refuse to answer any question not about the info. Never break character.
------------
{context}
------------
REMEMBER: If there is no relevant information within the context, just say "Hmm, I'm not sure". Don't try to make up an answer. Never break character.
```

### 3. QA_TEMPLATE

**Назначение:** Базовый шаблон для простого вопрос-ответа без сохранения истории диалога.

**Применение:** Используется для простых QA задач, где не требуется поддержание контекста беседы.

**Переменные:**
- `{context}` - фрагменты текста для ответа
- `{question}` - вопрос пользователя

**Промт:**
```
Use the following pieces of context to answer the question at the end.

{context}

Question: {question}
Helpful Answer:
```

### 4. REPHRASE_TEMPLATE

**Назначение:** Альтернативный шаблон для переформулирования вопросов. Более короткая версия CUSTOM_QUESTION_GENERATOR_CHAIN_PROMPT.

**Применение:** Используется когда нужна более лаконичная переформулировка без дополнительных инструкций.

**Переменные:**
- `{chat_history}` - история беседы
- `{question}` - текущий вопрос

**Промт:**
```
Given the following conversation and a follow up question, rephrase the follow up question to be a standalone question.

Chat History:
{chat_history}
Follow Up Input: {question}
Standalone Question:
```

### 5. CONVERSATION_CHAIN_SYSTEM_MESSAGE

**Назначение:** Системное сообщение по умолчанию для ConversationChain - определяет характер обычного диалога с AI.

**Применение:** Используется в ConversationChain для создания дружелюбного разговорного интерфейса.

**Файл:** `packages/components/nodes/chains/ConversationChain/ConversationChain.ts:32`

**Особенности:**
- Определяет AI как разговорчивого и детального ассистента
- AI честно признаёт незнание, если не знает ответа
- Создаёт дружелюбную атмосферу беседы

**Промт:**
```
The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.
```

---

## AGENT PROMPTS

Системные промты для различных типов агентов - автономных AI систем, которые могут использовать инструменты, планировать действия и выполнять сложные задачи.

**Основные файлы:**
- `packages/components/nodes/agents/ConversationalAgent/ConversationalAgent.ts`
- `packages/components/nodes/agents/ToolAgent/ToolAgent.ts`
- `packages/components/nodes/agents/XMLAgent/XMLAgent.ts`
- `packages/components/nodes/agents/BabyAGI/core.ts`
- `packages/components/nodes/agents/AutoGPT/AutoGPT.ts`
- `packages/components/nodes/agents/CSVAgent/core.ts`
- `packages/components/nodes/multiagents/Supervisor/Supervisor.ts`

### 1. CONVERSATIONAL_AGENT_DEFAULT_PREFIX

**Назначение:** Базовый системный промт для ConversationalAgent - определяет роль и возможности AI ассистента, обученного OpenAI.

**Применение:** Используется как префикс промта для создания разговорных агентов, которые могут работать с различными инструментами.

**Файл:** `packages/components/nodes/agents/ConversationalAgent/ConversationalAgent.ts:27-33`

**Особенности:**
- Описывает AI как большую языковую модель от OpenAI
- Подчеркивает широкий спектр возможностей ассистента
- Указывает на способность понимать и генерировать человекоподобный текст
- Отмечает постоянное обучение и улучшение модели

**Промт:**
```
Assistant is a large language model trained by OpenAI.

Assistant is designed to be able to assist with a wide range of tasks, from answering simple questions to providing in-depth explanations and discussions on a wide range of topics. As a language model, Assistant is able to generate human-like text based on the input it receives, allowing it to engage in natural-sounding conversations and provide responses that are coherent and relevant to the topic at hand.

Assistant is constantly learning and improving, and its capabilities are constantly evolving. It is able to process and understand large amounts of text, and can use this knowledge to provide accurate and informative responses to a wide range of questions. Additionally, Assistant is able to generate its own text based on the input it receives, allowing it to engage in discussions and provide explanations and descriptions on a wide range of topics.

Overall, Assistant is a powerful system that can help with a wide range of tasks and provide valuable insights and information on a wide range of topics. Whether you need help with a specific question or just want to have a conversation about a particular topic, Assistant is here to assist.
```

### 2. TEMPLATE_TOOL_RESPONSE

**Назначение:** Шаблон для форматирования ответов инструментов в agent scratchpad. Используется для обработки результатов выполнения инструментов и формирования следующего шага агента.

**Применение:** Применяется в ConversationalAgent для структурирования вывода после использования инструментов.

**Файл:** `packages/components/nodes/agents/ConversationalAgent/ConversationalAgent.ts:35-42`

**Переменные:**
- `{observation}` - результат работы инструмента

**Особенности:**
- Напоминает агенту, что он "забыл" ответы инструментов
- Требует явного упоминания информации из инструментов без названия самих инструментов
- Ожидает ответ в формате JSON blob с одним действием

**Промт:**
```
TOOL RESPONSE:
---------------------
{observation}

USER'S INPUT
--------------------

Okay, so what is the response to my last comment? If using information obtained from the tools you must mention it explicitly without mentioning the tool names - I have forgotten all TOOL RESPONSES! Remember to respond with a markdown code snippet of a json blob with a single action, and NOTHING else.
```

### 3. TOOL_AGENT_SYSTEM_MESSAGE

**Назначение:** Минималистичное системное сообщение для ToolAgent - агента, работающего через нативный вызов функций (function calling) в современных LLM.

**Применение:** Используется в ToolAgent для моделей с встроенной поддержкой tool calling (GPT-4, Claude 3, и т.д.).

**Файл:** `packages/components/nodes/agents/ToolAgent/ToolAgent.ts:84`

**Особенности:**
- Очень краткий промт
- Полагается на встроенные возможности модели для работы с инструментами
- Не требует сложных инструкций по форматированию

**Промт:**
```
You are a helpful AI assistant.
```

### 4. XML_AGENT_DEFAULT_SYSTEM_MESSAGE

**Назначение:** Системный промт для XMLAgent - агента, использующего XML-теги для структурированного вызова инструментов. Особенно эффективен для reasoning моделей вроде Claude.

**Применение:** Используется в XMLAgent для моделей, которые хорошо работают с XML-структурированным выводом.

**Файл:** `packages/components/nodes/agents/XMLAgent/XMLAgent.ts:25-47`

**Переменные:**
- `{tools}` - список доступных инструментов
- `{chat_history}` - история предыдущих сообщений
- `{input}` - текущий запрос пользователя
- `{agent_scratchpad}` - рабочая область агента с промежуточными шагами

**Особенности:**
- Использует XML-теги: `<tool>`, `<tool_input>`, `<observation>`, `<final_answer>`
- Предоставляет четкие примеры использования
- Явное указание на завершение работы через `<final_answer>`

**Промт:**
```
You are a helpful assistant. Help the user answer any questions.

You have access to the following tools:

{tools}

In order to use a tool, you can use <tool></tool> and <tool_input></tool_input> tags. You will then get back a response in the form <observation></observation>
For example, if you have a tool called 'search' that could run a google search, in order to search for the weather in SF you would respond:

<tool>search</tool><tool_input>weather in SF</tool_input>
<observation>64 degrees</observation>

When you are done, respond with a final answer between <final_answer></final_answer>. For example:

<final_answer>The weather in SF is 64 degrees</final_answer>

Begin!

Previous Conversation:
{chat_history}

Question: {input}
{agent_scratchpad}
```

### 5. BABYAGI_TASK_CREATION_PROMPT

**Назначение:** Промт для создания новых задач в системе BabyAGI на основе результатов выполнения предыдущих задач.

**Применение:** Используется в BabyAGI для автономной генерации новых подзадач на пути к достижению цели.

**Файл:** `packages/components/nodes/agents/BabyAGI/core.ts:13-21`

**Переменные:**
- `{objective}` - главная цель
- `{result}` - результат последней выполненной задачи
- `{task_description}` - описание последней выполненной задачи
- `{incomplete_tasks}` - список незавершенных задач

**Особенности:**
- Создаёт задачи, которые не пересекаются с существующими
- Возвращает массив новых задач
- Учитывает контекст предыдущих результатов

**Промт:**
```
You are a task creation AI that uses the result of an execution agent to create new tasks with the following objective: {objective}, The last completed task has the result: {result}. This result was based on this task description: {task_description}. These are incomplete tasks list: {incomplete_tasks}. Based on the result, create new tasks to be completed by the AI system that do not overlap with incomplete tasks. Return the tasks as an array.
```

### 6. BABYAGI_TASK_PRIORITIZATION_PROMPT

**Назначение:** Промт для приоритизации списка задач в BabyAGI с учётом общей цели.

**Применение:** Используется для переупорядочивания задач по важности для достижения цели.

**Файл:** `packages/components/nodes/agents/BabyAGI/core.ts:38-45`

**Переменные:**
- `{task_names}` - список названий задач
- `{objective}` - главная цель команды
- `{next_task_id}` - номер, с которого начинать нумерацию

**Особенности:**
- Не удаляет задачи, только переупорядочивает
- Возвращает пронумерованный список
- Начинает нумерацию с указанного ID

**Промт:**
```
You are a task prioritization AI tasked with cleaning the formatting of and reprioritizing the following task list: {task_names}. Consider the ultimate objective of your team: {objective}. Do not remove any tasks. Return the result as a numbered list, like:
 #. First task
 #. Second task
 Start the task list with number {next_task_id}.
```

### 7. BABYAGI_EXECUTION_PROMPT

**Назначение:** Промт для выполнения конкретной задачи в BabyAGI с учётом контекста предыдущих выполненных задач.

**Применение:** Используется для генерации ответа на конкретную задачу.

**Файл:** `packages/components/nodes/agents/BabyAGI/core.ts:60-64`

**Переменные:**
- `{objective}` - главная цель
- `{context}` - контекст из ранее выполненных задач
- `{task}` - текущая задача для выполнения

**Промт:**
```
You are an AI who performs one task based on the following objective: {objective}. Take into account these previously completed tasks: {context}. Your task: {task}. Response:
```

### 8. AUTOGPT_REPHRASE_PROMPT

**Назначение:** Промт для переформулирования вывода AutoGPT в более читаемую форму.

**Применение:** Используется для постобработки ответов AutoGPT.

**Файл:** `packages/components/nodes/agents/AutoGPT/AutoGPT.ts:217-220`

**Переменные:**
- `{sentence}` - предложение для переформулирования

**Промт:**
```
You are a helpful Assistant that rephrase a sentence: {sentence}
```

### 9. CSV_AGENT_PREFIX_PROMPT

**Назначение:** Системный промт для CSV Agent, который анализирует данные в pandas dataframe.

**Применение:** Используется для работы с CSV файлами через pandas, генерирует Python код для анализа данных.

**Файл:** `packages/components/nodes/agents/CSVAgent/core.ts:18-29`

**Переменные:**
- `{dict}` - словарь с названиями столбцов и типами данных
- `{question}` - вопрос пользователя

**Особенности:**
- Работает с pandas dataframe с именем `df`
- Генерирует только Python код без объяснений
- Получает структуру данных в виде Python словаря

**Промт:**
```
You are working with a pandas dataframe in Python. The name of the dataframe is df.

The columns and data types of a dataframe are given below as a Python dictionary with keys showing column names and values showing the data types.
{dict}

I will ask question, and you will output the Python code using pandas dataframe to answer my question. Do not provide any explanations. Do not respond with anything except the output of the code.

Question: {question}
Output Code:
```

### 10. CSV_AGENT_FINAL_ANSWER_PROMPT

**Назначение:** Промт для переформулирования технического ответа CSV Agent в самостоятельный ответ.

**Применение:** Используется для финальной обработки результатов анализа данных.

**Файл:** `packages/components/nodes/agents/CSVAgent/core.ts:28`

**Переменные:**
- `{question}` - исходный вопрос
- `{answer}` - технический ответ

**Промт:**
```
You are given the question: {question}. You have an answer to the question: {answer}. Rephrase the answer into a standalone answer.
Standalone Answer:
```

### 11. SUPERVISOR_SYSTEM_PROMPT

**Назначение:** Системный промт для Supervisor агента в мульти-агентных системах. Координирует работу нескольких worker агентов.

**Применение:** Используется в мульти-агентных системах для управления последовательностью работы агентов.

**Файл:** `packages/components/nodes/multiagents/Supervisor/Supervisor.ts:26-30`

**Переменные:**
- `{team_members}` - список доступных worker агентов

**Особенности:**
- Выбирает следующего агента для работы
- Минимизирует количество шагов
- Отвечает "FINISH" когда задача выполнена

**Промт:**
```
You are a supervisor tasked with managing a conversation between the following workers: {team_members}.
Given the following user request, respond with the worker to act next.
Each worker will perform a task and respond with their results and status.
When finished, respond with FINISH.
Select strategically to minimize the number of steps taken.
```

---

## RETRIEVER PROMPTS

Промты для ретриверов - компонентов, которые извлекают релевантную информацию из векторных баз данных и других источников. Эти промты помогают улучшить качество поиска и расширить запросы пользователей.

**Основные файлы:**
- `packages/components/nodes/retrievers/MultiQueryRetriever/MultiQueryRetriever.ts`
- `packages/components/nodes/retrievers/HydeRetriever/HydeRetriever.ts`
- `packages/components/nodes/retrievers/PromptRetriever/PromptRetriever.ts`
- `packages/components/nodes/retrievers/ExtractMetadataRetriever/ExtractMetadataRetriever.ts`

### 1. MULTI_QUERY_RETRIEVER_PROMPT

**Назначение:** Генерирует несколько альтернативных формулировок пользовательского запроса для улучшения поиска в векторных базах данных. Помогает преодолеть ограничения distance-based similarity search.

**Применение:** Используется в MultiQueryRetriever для создания 3 различных версий одного вопроса, что увеличивает шансы найти все релевантные документы.

**Файл:** `packages/components/nodes/retrievers/MultiQueryRetriever/MultiQueryRetriever.ts:5-20`

**Переменные:**
- `{question}` - исходный вопрос пользователя

**Особенности:**
- Генерирует ровно 3 альтернативные версии вопроса
- Результат оборачивается в XML теги `<questions>`
- Каждая альтернатива на отдельной строке

**Промт:**
```
You are an AI language model assistant. Your task is
to generate 3 different versions of the given user
question to retrieve relevant documents from a vector database.
By generating multiple perspectives on the user question,
your goal is to help the user overcome some of the limitations
of distance-based similarity search.

Provide these alternative questions separated by newlines between XML tags. For example:

<questions>
Question 1
Question 2
Question 3
</questions>

Original question: {question}
```

### 2. HYDE_RETRIEVER_PROMPTS

**Назначение:** Набор специализированных промтов для HyDE (Hypothetical Document Embeddings) retriever. Генерирует гипотетические документы, которые затем используются для улучшения поиска в различных доменах.

**Применение:** HyDE создаёт "воображаемый" документ, который теоретически мог бы ответить на вопрос, и использует его эмбеддинг для поиска реальных документов.

**Файл:** `packages/components/nodes/retrievers/HydeRetriever/HydeRetriever.ts:54-109`

**Переменные:**
- `{question}` - вопрос или утверждение пользователя

#### 2.1 Web Search Prompt

**Назначение:** Для общего веб-поиска и информационных запросов.

**Промт:**
```
Please write a passage to answer the question
Question: {question}
Passage:
```

#### 2.2 SciFact Prompt

**Назначение:** Для научных утверждений и проверки фактов в научной литературе.

**Промт:**
```
Please write a scientific paper passage to support/refute the claim
Claim: {question}
Passage:
```

#### 2.3 Arguana Prompt

**Назначение:** Для аргументации и дебатов, генерирует контраргументы.

**Промт:**
```
Please write a counter argument for the passage
Passage: {question}
Counter Argument:
```

#### 2.4 TREC-COVID Prompt

**Назначение:** Для медицинских и научных запросов, особенно связанных с COVID-19.

**Промт:**
```
Please write a scientific paper passage to answer the question
Question: {question}
Passage:
```

#### 2.5 FIQA (Financial) Prompt

**Назначение:** Для финансовых вопросов и анализа финансовой информации.

**Промт:**
```
Please write a financial article passage to answer the question
Question: {question}
Passage:
```

#### 2.6 DBPedia-Entity Prompt

**Назначение:** Для поиска информации о сущностях (люди, места, организации).

**Промт:**
```
Please write a passage to answer the question.
Question: {question}
Passage:
```

#### 2.7 TREC-News Prompt

**Назначение:** Для новостных тем и актуальных событий.

**Промт:**
```
Please write a news passage about the topic.
Topic: {question}
Passage:
```

#### 2.8 Mr-Tydi (Multilingual) Prompt

**Назначение:** Для мультиязычного поиска (Swahili, Korean, Japanese, Bengali).

**Промт:**
```
Please write a passage in Swahili/Korean/Japanese/Bengali to answer the question in detail.
Question: {question}
Passage:
```

### 3. PROMPT_RETRIEVER_EXAMPLE

**Назначение:** Пример промта для PromptRetriever - демонстрирует, как можно создать специализированного ассистента для конкретной области знаний.

**Применение:** Используется как шаблон для создания доменно-специфичных промтов.

**Файл:** `packages/components/nodes/retrievers/PromptRetriever/PromptRetriever.ts:40-44`

**Особенности:**
- Определяет роль эксперта (профессор физики)
- Устанавливает стиль ответов (краткие и понятные)
- Указывает поведение при незнании ответа

**Промт:**
```
You are a very smart physics professor. You are great at answering questions about physics in a concise and easy to understand manner. When you don't know the answer to a question you admit that you don't know.
```

### 4. EXTRACT_METADATA_RETRIEVER_PROMPT

**Назначение:** Извлекает ключевые слова из запроса пользователя для улучшения метаданных поиска.

**Применение:** Используется для создания структурированных метаданных из естественного языка запроса.

**Файл:** `packages/components/nodes/retrievers/ExtractMetadataRetriever/ExtractMetadataRetriever.ts:9`

**Переменные:**
- `{{query}}` - запрос пользователя (обратите внимание на двойные фигурные скобки)

**Промт:**
```
Extract keywords from the query: {{query}}
```

---

