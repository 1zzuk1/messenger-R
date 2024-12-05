# messenger-R
# Messenger Application

This repository contains a lightweight **Messenger Application** built using **R**, leveraging `Shiny` for the front-end interface and a simple RESTful backend. The application allows users to register, log in, send messages, and view conversations with others. It also features local storage for chat persistence and periodic updates for real-time functionality.

---

## Features

### Front-End
- **User Authentication**: Register and log in with a username and password.
- **Messaging**: Send messages to other registered users.
- **Chat Tabs**: Conversations are displayed dynamically as tabs for each user.
- **Real-Time Updates**: Automatically fetches new messages periodically.
- **Local Persistence**: Saves chat history locally to reload on app restart.

### Back-End
- **User Management**: Supports user registration and authentication with hashed passwords.
- **Message Queue**: Manages message storage and retrieval using JSON files.
- **Endpoints**:
  - `/register`: Register a new user.
  - `/authenticate`: Authenticate an existing user.
  - `/send`: Send a message.
  - `/fetch`: Fetch messages for a user.

---

## Installation

### Prerequisites
- R installed on your system.
- Required R packages: `shiny`, `httr`, `jsonlite`, `httpuv`, `openssl`.

### Steps
1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/messenger-app.git
   cd messenger-app
   ```

2. Install required R packages:
   ```R
   install.packages(c("shiny", "httr", "jsonlite", "httpuv", "openssl"))
   ```

3. Start the backend server:
   - Navigate to the backend code (`R-messengerserver.R`).
   - Run the server script:
     ```R
     source("R-messengerserver.R")
     ```
   - The server will be available at `http://localhost:8080`.

4. Start the front-end Shiny app:
   - Navigate to the front-end code (`R-messengerclient.R`).
   - Run the app script:
     ```R
     shiny::runApp("app.R")
     ```
   - The app will open in your default browser.

---

## Usage

### Logging In or Registering
1. Enter a username and password in the sidebar.
2. Click "Login/Register".
   - If the username does not exist, it will automatically register a new user.
   - If the username exists, it will authenticate the user.

### Sending Messages
1. Enter the recipient's username and the message in the sidebar.
2. Click "Send Message".
3. The message will appear in the chat tab for the recipient.

### Viewing Conversations
- Conversations with different users are displayed in separate tabs.
- The app fetches new messages periodically, ensuring real-time updates.

---

## File Structure

```
.
├── server.R                  # Backend server handling user and message logic
├── app.R                     # Front-end Shiny application
├── messenger_config.json     # Local file for storing user conversations
├── server_credentials.json   # JSON file for storing user credentials (backend)
├── server_messages.json      # JSON file for storing messages (backend)
```

---

## Endpoints

The backend server exposes the following RESTful endpoints:

| Endpoint       | Method | Description                           |
|----------------|--------|---------------------------------------|
| `/register`    | POST   | Register a new user                  |
| `/authenticate`| POST   | Authenticate an existing user        |
| `/send`        | POST   | Send a message to another user       |
| `/fetch`       | POST   | Fetch messages for a specific user   |

---

## Enhancements

This project can be extended with:
- **Secure Authentication**: Add session tokens for secure communication.
- **UI Improvements**: Enhance the interface with better styling and UX features.
- **Encryption**: Encrypt locally stored chat data.
- **Deployment**: Host the app on a public server for shared access.

---

## License

This project is licensed under the [MIT License](LICENSE).

---

## Contributions

Contributions are welcome! Please create an issue or submit a pull request for bug fixes or new features.

---

## Acknowledgments

This project uses:
- [Shiny](https://shiny.rstudio.com/)
- [httr](https://httr.r-lib.org/)
- [jsonlite](https://cran.r-project.org/web/packages/jsonlite/index.html)
- [httpuv](https://cran.r-project.org/web/packages/httpuv/index.html)

---

Enjoy chatting! 😊
