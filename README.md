# 🌸 Magnolia — AI-агент для осознанного дня

Интерактивное приложение для анализа планетарных транзитов Human Design. Система помогает пользователям прожить энергию дня, предлагая диалог с ИИ.

## Описание

Magnolia — это мультиагентная система, объединяющая RAG-базу знаний канонических описаний транзитов с продвинутой логикой проверки ответов. Проект направлен на то, чтобы превратить абстрактные астрологические данные в персональные, практические советы для реальной жизни.

## 🚀 Как это работает

### 1. Выбор энергии дня
На главной странице пользователь видит актуальный список транзитов в формате `Планета в Ворота.Линия`. Каждая карточка содержит краткую суть энергии, чтобы вы сразу могли выбрать то, что откликается.

<p align="center">
  <img src="screenshots/01-main-page.png" alt="Главная страница со списком транзитов" width="800">
</p>

### 2. Диалог с ИИ-наставником
После выбора темы открывается чат. Вы общаетесь с агентом на базе легкой модели (8B), описываете свои ситуации и ощущения. Агент помогает структурировать мысли и контекст.

<p align="center">
  <img src="screenshots/02-chat-dialogue.png" alt="Интерфейс чата с агентом" width="800">
</p>

### 3. Администрирование и Логи
Для прозрачности работы системы предусмотрен административный дашборд. Здесь можно в реальном времени отслеживать логи работы моделей, запускать тесты (Benchmark) и проверять корректность ответов.

<p align="center">
  <img src="screenshots/03-dashboard-logs.png" alt="Дашборд администратора с логами" width="800">
</p>

## 🏗️ Архитектура системы

### 4. Текущая схема
Поток данных выстроен от пользователя через чат к цепочке моделей (8B → 14B), затем проходит валидацию через MCP-сервер и сохраняется в базе данных (SQLite).

Архитектура построена на **LangGraph** (детерминированный поток) + **MCP-сервис** (интеллектуальная валидация).
Система использует два уровня моделей:
1.  **8B (Dialog):** Для быстрого общения и сбора контекста.
2.  **14B (Analysis):** Для генерации финального совета и его оценки.

Ниже представлена полная схема взаимодействия узлов графа, инструментов и баз данных:

```mermaid
graph TB
    subgraph "🖥️ Web Interface (Port 8001)"
        UI[Главная страница]
        CHAT[Чат с агентом]
        DASH[Dashboard]
        SET[Настройки]
    end
    
    subgraph "🧠 LangGraph Workflow"
        GET_NUM[get_numbers<br/>Чтение CSV]
        GET_DESC[get_descriptions<br/>RAG из JSON]
        VALIDATE[validate<br/>Проверка чисел]
        FINAL[final<br/>Сборка контекста]
        ERROR[error<br/>Обработка ошибок]
    end
    
    subgraph "🛠️ Tools (LangChain)"
        TOOL1[get_numbers_from_db]
        TOOL2[get_descriptions_for_numbers]
        TOOL3[validate_numbers]
        TOOL4[save_advice_tool]
    end
    
    subgraph "🤖 LLM Models"
        LLM8B[qwen3-vl-8b<br/>Диалог]
        LLM14B[eva-qwen2.5-14b<br/>Генерация совета]
    end
    
    subgraph "🔧 MCP Server (Port 8002)"
        MCP_UNIQUE[check_uniqueness<br/>7 дней]
        MCP_EVAL[evaluate_with_llm<br/>0-100 баллов]
        MCP_ENRICH[enrich_with_llm<br/>Бытовые действия]
    end
    
    subgraph "💾 Data Layer"
        CSV[data/transits.csv]
        RAG[data/rag.json]
        DB[(tips.db<br/>SQLite)]
        LANG[(LangFuse<br/>Observability)]
    end
    
    UI --> GET_NUM
    CHAT --> LLM8B
    GET_NUM --> GET_DESC
    GET_NUM --> ERROR
    GET_DESC --> VALIDATE
    VALIDATE --> FINAL
    VALIDATE --> ERROR
    FINAL --> LLM8B
    LLM8B --> LLM14B
    LLM14B --> MCP_UNIQUE
    MCP_UNIQUE --> MCP_EVAL
    MCP_EVAL --> MCP_ENRICH
    MCP_ENRICH --> DB
    TOOL1 -.-> CSV
    TOOL2 -.-> RAG
    TOOL4 -.-> DB
    LLM8B -.-> LANG
    
    style UI fill:#e1f5ff,color:#333333
    style CHAT fill:#e1f5ff,color:#333333
    style GET_NUM fill:#fff4e1,color:#333333
    style GET_DESC fill:#fff4e1,color:#333333
    style VALIDATE fill:#fff4e1,color:#333333
    style FINAL fill:#fff4e1,color:#333333
    style ERROR fill:#ffe1e1,color:#333333
    style LLM8B fill:#ffe1e1,color:#333333
    style LLM14B fill:#ffe1e1,color:#333333
    style MCP_UNIQUE fill:#e1ffe1,color:#333333
    style MCP_EVAL fill:#e1ffe1,color:#333333
    style MCP_ENRICH fill:#e1ffe1,color:#333333
    style DB fill:#f0e1ff,color:#333333
    style LANG fill:#f0e1ff,color:#333333
    style TOOL1 fill:#f5f5f5,color:#333333
    style TOOL2 fill:#f5f5f5,color:#333333
    style TOOL3 fill:#f5f5f5,color:#333333
    style TOOL4 fill:#f5f5f5,color:#333333
```
### Настройки и кастомизация
Приложение позволяет гибко настраивать URL моделей и переключаться между разными нейросетями (например, Qwen и Eva-Qwen) без изменения кода.

<p align="center">
  <img src="screenshots/05-settings.png" alt="Настройки моделей" width="800">
</p>

### Планы развития (Roadmap)
Система готова к масштабированию. В следующей версии планируется интеграция **ComfyUI** для генерации визуальных метафор по транзиту (pic) и модуля **Text-to-Speech** для озвучивания советов (Audio).

<p align="center">
  <img src="screenshots/06-upcoming add to.png" alt="Схема будущей архитектуры с аудио и картинками" width="800">
</p>

## ⚙️ Технический стек

| Компонент | Технология / Модель | Назначение |
| :--- | :--- | :--- |
| **LLM (Dialog)** | `qwen/qwen3-vl-8b` | Общение и сбор контекста |
| **LLM (Analysis)** | `eva-qwen2.5-14b-v0.2` | Генерация советов и анализ |
| **Backend** | MCP Server (Port 8002) | Валидация, оценка, фильтрация |
| **Database** | SQLite (`tips.db`) | Хранение уникальных советов |
| **UI** | Web Interface | Интерактивный фронтенд |

## 🔧 Быстрый старт

```bash
# 1. Клонируйте репозиторий
git clone https://github.com/Jui4an/hd-transit-agent.git
cd hd-transit-agent

# 2. Установите зависимости
pip install -r requirements.txt

# 3. Запустите приложение
python main.py
