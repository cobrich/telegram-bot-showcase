# 🤖 Go2Study Bot - Portfolio demonstration

**Go2Study Bot** — this is a multifunctional Telegram bot for learning mathematics, created to prepare 5th and 6th grade students for exams. The bot uses an adaptive learning system and AI integration to personalize the learning process.

> **Note:** The source code for this project is private, as the bot is currently in active use (production). This repository serves to demonstrate its functionality, architecture, and technology stack.

---

## 🎥 Demonstration of operation

This section will feature a GIF animation or short video (15-30 seconds) demonstrating the main scenario for using the bot: launching, selecting a topic, taking the test, and receiving the results.

*(Вставьте сюда вашу GIF или видео)*

---

## ✨ Key features

-   🎯 **Adaptive testing**: The system focuses on topics where the student makes mistakes, suggesting them for review.
-   🤖 **Integration with AI (Google Gemini)**: 
    -   Automatic generation of explanations for test questions.
    -   Import and processing of questions from PDF files.
-   📊 **Progress analytics**: Users can track their progress on various topics by viewing statistics on completed tests.
-   👥 **Administration system**: A full-featured panel for administrators that allows them to manage students, topics, questions, and view general statistics.
-   🌐 **Multilingualism**: Full support for Russian and Kazakh languages with the ability to switch between them on the fly.
-   🔐 **Access control**: A whitelist-based system ensures that only authorized students can use the bot.
-   🔄 **Working on mistakes**: Mechanisms for repeating and reinforcing material where mistakes were made.
-   💬 **Optimized interface**: Key messages in the chat (e.g., “My progress,” “Help”) are not duplicated but update previous ones, making the interface cleaner.

---

## 🏗️ Architecture and technology stack

The project has a modular architecture, where each component is responsible for its own part of the logic. This ensures flexibility and ease of maintenance.

### Architecture diagram

```mermaid
graph TD
    A[Telegram user] --> B{Telegram Bot API};
    B --> C["Python Backend (aiogram)"];
    C --> D{Command and callback handlers};
    D --> E[Services (business logic)];
    E --> F[Database (PostgreSQL)];
    E --> G[AI Model (Google Gemini API)];
    F --> C;
    G --> C;

    subgraph "Application"
        C
        D
        E
    end
```

### Technology stack

-   **Programming language**: `Python 3.9`
-   **Main framework for Telegram**: `python-telegram-bot`
-   **Database**: `PostgreSQL` (via `SQLAlchemy` and `psycopg2`)
-   **AI integration**: `google-generativeai` (Google Gemini)
-   **Web server for health checks**: `Flask`
-   **Deployment**: `Docker`, `Railway`

---

## 📄 Example of Code

Below is a code snippet demonstrating the processing of the `/reset` command. This function allows the user to reset their current state (for example, if they are “stuck” in a test), which is an important part of a resilient user experience in a bot.

The code is written in asynchronous style using the `python-telegram-bot` library.

```python
# src/handlers/command_handlers.py

# ... (imports and class initialization)

async def reset(self, update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Reset user state if stuck in a test."""
    user_id = update.effective_user.id
    user_language = self.db.get_user_language(user_id)
    
    # Verifying user access
    if not self.db.check_user_access(user_id, update.effective_user.username):
        await update.message.reply_text(
            get_message('no_access', user_language, 
                        user_id=user_id, 
                        username=update.effective_user.username or 'не указан')
        )
        return
    
    # FORCIBLY clear the user's status (even if they are in a test)
    # Clear user activity (is_active status)
    self.db.set_user_inactive(user_id)
    
    # Clear current test topic
    self.db.clear_user_test_activity(user_id)
    
    # Clear context data
    context.user_data.clear()
    
    reset_text = get_message('state_reset', user_language)
    await update.message.reply_text(
        reset_text,
        reply_markup=get_main_menu_markup(user_id)
    )

```

