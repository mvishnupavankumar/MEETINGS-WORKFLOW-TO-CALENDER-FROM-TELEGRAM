# MEETINGS-WORKFLOW-TO-CALENDER-FROM-TELEGRAM
📂 Suggested Repo Name:
telegram-to-google-calendar-manager

📘 README.md Content:
📅 Telegram to Google Calendar Meeting Manager using n8n
📖 Description
This n8n workflow acts as your personal AI assistant to manage meetings directly from Telegram! 🤖 It intelligently understands your requests to create new meetings on your Google Calendar or fetch existing meeting schedules. Perfect for beginners, this workflow leverages OpenRouter's AI capabilities for natural language processing and integrates seamlessly with Google Calendar and Telegram.

⚙️ How It Works (step-by-step with emojis)
This workflow listens for your messages on Telegram and then intelligently decides whether you want to create a new meeting or check your existing schedule.

🔹 1. Takes Message 💬
Simple: This is the starting point! The workflow waits for any message you send to your Telegram bot.

Tech: Uses a Telegram Trigger node, configured to listen for incoming messages.

🔹 2. Switches Based on Message 🚦
Simple: The workflow checks your message for keywords like "check," "free," "find" (to search for meetings) or "set," "arrange," "create" (to add a new meeting).

Tech: A Switch node evaluates the incoming Telegram message text (converted to lowercase) to route the workflow to either the "search" path or the "create" path based on keyword detection.

🔹 3. Takes Searching Details (AI - Date Extractor) 🧠
Simple: If you asked to check meetings, this step uses AI to understand the date or date range you're asking about (e.g., "today," "next week," "July 15th").

Tech: An HTTP Request node sends your message to the OpenRouter AI (mistralai/mistral-7b-instruct model) with a system prompt specifically designed to extract single dates or date ranges in a structured DD-MM-YYYY format.

🔹 4. Converts That Calendar Understand (Code) 🧑‍💻
Simple: This step transforms the AI-extracted date information into a format Google Calendar can understand (ISO 8601 timestamps).

Tech: A Code node takes the DATE or DATE_RANGE output from the previous AI node, converts it into ISO 8601 timeMin and timeMax values, and ensures the correct IST timezone offset.

🔹 5. Get Many Events 📆
Simple: This is where the workflow actually queries your Google Calendar for meetings within the specified date range.

Tech: A Google Calendar node with the getAll operation retrieves events from your specified calendar, using the timeMin and timeMax values from the previous step.

🔹 6. Send Dummy / Take Dummy Data (Set / Code) 🧱
Simple: These steps ensure the workflow doesn't break if no events are found. They provide a placeholder if your calendar is empty for the requested period.

Tech: The Send Dummy Set node creates a default empty event structure. The Take Dummy Data Code node then parses this dummy data to ensure a consistent input for subsequent nodes, even when no real events are found.

🔹 7. Checks Data (Code) ✅
Simple: This step makes sure you're not trying to look up meetings too far in the past (only checks up to 3 months ago).

Tech: A Code node filters events based on a 3-month lookback period from the current IST date.

🔹 8. Merge (Merge) 🧩
Simple: This step combines the actual Google Calendar events with the dummy data, if applicable, to ensure all relevant information flows forward.

Tech: A Merge node combines outputs from "Checks Data" and "Take Dummy Data" to provide a unified input for the next step, ensuring that the workflow always has event data (even if it's empty).

🔹 9. Checks Event Present or Not (Code) ✨
Simple: This node determines if any real meetings were found by the Google Calendar query.

Tech: A Code node processes the merged event data, checking if the events array is populated. It sets a hasEvents boolean flag and passes the events if they exist.

🔹 10. Switches Based Event Present or Not 🚦
Simple: Based on whether meetings were found or not, the workflow decides whether to list them or tell you there are none.

Tech: A Switch node routes the workflow based on the hasEvents flag from the previous node.

🔹 11. Edit Events (Code) 📝
Simple: If meetings were found, this step formats all the meeting details (name, date, time, link) into a readable message for you.

Tech: A Code node iterates through the retrieved Google Calendar events and constructs a human-readable text message containing the event summary, date, start/end times (or "All Day Event"), and HTML link.

🔹 12. Send a Text Message (Telegram) 📤
Simple: This sends the list of found meetings (or the "no meetings" message) back to you on Telegram.

Tech: A Telegram node sends the formatted text message back to your specified chatId.

🔹 13. Takes Meeting Details1 (AI - Meeting Extractor) 🧠
Simple: If you asked to create a meeting, this step uses AI to extract all the important bits like the meeting subject, date, time, and any meeting link from your message.

Tech: An HTTP Request node sends your message to the OpenRouter AI (mistralai/mistral-7b-instruct model) with a system prompt tailored to extract meeting subject, date (DD-MM-YYYY), start/end times (12-hour format), and meeting link.

🔹 14. Clean Meeting Details (Code) 🧹
Simple: This step tidies up the AI-extracted meeting details, ensuring the time format is correct and consistent.

Tech: A Code node parses the AI response, normalizes the time format (e.g., "10PM" to "10:00PM"), and extracts the meeting name, date, start time, end time, and link into a structured JSON object.

🔹 15. Set Timings as Calendar Understand (Code) 🕰️
Simple: This step converts the cleaned meeting times and dates into a specific format that Google Calendar requires to create an event.

Tech: A Code node converts the date and time strings into ISO 8601 formatted dateTime strings with the "Asia/Kolkata" timezone, ready for Google Calendar. It also sets the summary and description for the event.

🔹 16. Create an Event (Google Calendar) ➕
Simple: This is the magic step! It takes all the prepared information and creates the new meeting on your Google Calendar.

Tech: A Google Calendar node with the create operation adds a new event to your calendar using the start, end, summary, description, and location provided by the previous node.

🔹 17. Takes Confirmation Message (Set) ✅
Simple: After creating the event, this step prepares a "Meeting Set" confirmation message to send back to you.

Tech: A Set node constructs a confirmation message string using the meeting summary and the start/end datetimes of the newly created event.

🔹 18. Edit Message (Code) 📝
Simple: This step formats the confirmation message to be easy to read and understand before it's sent to you.

Tech: A Code node parses the confirmation message string, extracts relevant details, and formats them into a clean, user-friendly message with the meeting subject, start time, and end time.

🔹 19. Merge1 (Merge) 🧩
Simple: This merges the confirmation message data with the date range data, ensuring all necessary details are available for the final message.

Tech: A Merge node combines the output from the "Converts That Calendar Understand" node (which provided the date range) and the "Switches Based Event Present or Not" node to create a comprehensive data item for the final message.

🔹 20. Send Message (Code) ✉️
Simple: This is a final check and preparation step to combine all relevant information before sending the message to Telegram.

Tech: A Code node takes the merged data (containing event status, date range, and possibly events) and formats it to be sent as a Telegram message.

🔁 Replace the following before running:
🔐 YOUR_GMAIL_ADDRESS
Replace this with your personal Gmail address in the "Get many events" and "Create an event" nodes for calendar access.

🔐 Google Calendar account
Configure your Google Calendar OAuth2 API credentials in n8n under the Credentials section. Look for the REPLACE_ME placeholder in the JSON and update the credential ID.

🔐 YOUR_OPENROUTER_BEARER_TOKEN
Replace this with your OpenRouter Bearer token in the Authorization header of both "TAKES MEETING DETAILS1" and "TAKES SEARCHING DETAILS" HTTP Request nodes.

💬 YOUR_TELEGRAM_CHAT_ID
Insert your personal Telegram chat ID in the "Send a text message" and "Send a text message1" nodes.

🤖 YOUR_TELEGRAM_BOT_TOKEN
Add your bot token to the Telegram credentials in n8n under the Credentials section. Look for the REPLACE_ME placeholder in the JSON and update the credential ID.

🧪 Testing Instructions
To get this workflow up and running:

✅ Open n8n: Make sure your n8n instance is running, typically accessible at http://localhost:5678.
🔧 Import Workflow:
1. Click on "Workflows" in the left sidebar.
2. Click the "New" button and select "Import from JSON".
3. Paste the entire JSON content of this workflow and click "Import".
⚙️ Configure Credentials:
1. Click on the "Credentials" tab in n8n.
2. Set up your Google Calendar OAuth2 API and Telegram API credentials. Make sure the names match what is specified in the workflow's JSON (e.g., "Google Calendar account", "Telegram account").
3. Update the REPLACE_ME placeholders in the workflow's JSON with the actual IDs of your configured credentials.
4. Fill in YOUR_GMAIL_ADDRESS, YOUR_OPENROUTER_BEARER_TOKEN, and YOUR_TELEGRAM_CHAT_ID directly in the relevant nodes as mentioned in the "Replace These Values" section.
🚀 Activate Workflow: Toggle the workflow to "Active" in the top right corner.
🧪 Test Execution:
1. Send a message to your Telegram bot.
2. To create an event: Send a message like "set up a meeting about marketing presentation tomorrow at 10 am to 11 am with link meet.google.com/xyz".
3. To check events: Send a message like "check my meetings for today" or "find free slots this week".
4. Check the Telegram response for confirmation or the list of events.
5. Verify on Google Calendar if the event was created or check your calendar for the listed events.
🪵 Debug Errors: If you encounter issues, click on the "Executions" tab for the workflow. Each run will show you a detailed log of what happened at each node, helping you pinpoint and troubleshoot any errors.

💡 Tips for Beginners
💡 Use the “Credentials” tab: Always set up your API keys and accounts (like Gmail and Telegram) under the "Credentials" tab in n8n first. This keeps your sensitive information secure and reusable across workflows.
💡 Check node outputs: After each step, you can click on a node during a test execution to see its output data. This is incredibly helpful for verifying that the data is flowing as expected and for debugging.
💡 Adjust messages and prompts: Feel free to tweak the system prompts in the "TAKES MEETING DETAILS1" and "TAKES SEARCHING DETAILS" HTTP Request nodes if you want to modify how the AI interprets your requests or extracts information.
💡 Telegram Webhook URL: Ensure your Telegram Trigger's webhook is correctly set up. n8n will typically provide this when you activate the workflow.












Canvas


