import os
import json
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, MessageHandler, filters, ContextTypes

# Define the start command
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    keyboard = [
        [InlineKeyboardButton("Create Quiz", callback_data='create_quiz')],
        [InlineKeyboardButton("Help", callback_data='help')]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text('Welcome to the Quiz Bot! Use the buttons below to navigate:', reply_markup=reply_markup)

# Handle button clicks
async def button(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    query = update.callback_query
    await query.answer()
    if query.data == 'create_quiz':
        await query.edit_message_text(text="Send your quiz in the format: [{\"Question\": \"Question 1\", \"Options\": [\"Option 1\", \"Option 2\", \"Option 3\", \"Option 4\"], \"Answer\": \"Answer 1\"}]")
    elif query.data == 'help':
        await query.edit_message_text(text="This bot helps you create polls. Use the /start command to begin.")

# Handle received messages
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    text = update.message.text
    if text.startswith("[") and text.endswith("]"):
        try:
            quiz_data = json.loads(text)
            if isinstance(quiz_data, list):
                for item in quiz_data:
                    if isinstance(item, dict) and "Question" in item and "Options" in item and "Answer" in item:
                        question = item["Question"]
                        options = item["Options"]
                        correct_answer = item["Answer"]

                        if len(options) >= 2:
                            # Get the index of the correct answer
                            try:
                                correct_answer_index = options.index(correct_answer)
                            except ValueError:
                                await update.message.reply_text(f"Answer '{correct_answer}' is not in the options for question: {question}")
                                continue

                            # Send poll as a quiz with correct answer marked
                            await context.bot.send_poll(
                                chat_id=update.effective_chat.id,
                                question=question,
                                options=options,
                                type="quiz",  # Important: set poll type to quiz
                                correct_option_id=correct_answer_index,  # Mark the correct answer
                                is_anonymous=False,
                                allows_multiple_answers=False
                            )
                        else:
                            await update.message.reply_text("Each question must have at least two options.")
                    else:
                        await update.message.reply_text("Invalid format for one or more questions. Please check your input.")
            else:
                await update.message.reply_text("Please send a list of questions in the correct format.")
        except json.JSONDecodeError:
            await update.message.reply_text("Invalid format. Please use the correct JSON format.")
        except Exception as e:
            await update.message.reply_text(f"An error occurred: {e}")

def main():
    token = "TELEGRAM_BOT_TOKEN"  # Replace with your actual bot token
    application = ApplicationBuilder().token(token).build()

    # Add handlers
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CallbackQueryHandler(button))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    print("Bot is starting...")
    application.run_polling()

if __name__ == '__main__':
    main()



    
