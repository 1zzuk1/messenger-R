library(shiny)
library(httr)
library(jsonlite)

server_url <- "http://localhost:8080"
config_file <- "messenger_config.json"
ssend_request <- function(endpoint, body) {
  tryCatch({
    url <- paste0(server_url, endpoint)
    response <- POST(url, body = toJSON(body, auto_unbox = TRUE), encode = "json")
    if (status_code(response) >= 200 && status_code(response) < 300) {
      return(fromJSON(content(response, "text", encoding = "UTF-8"), simplifyVector = TRUE))
    } else {
      return(list(error = TRUE, message = paste("HTTP error:", status_code(response))))
    }
  }, error = function(e) {
    return(list(error = TRUE, message = e$message))
  })
}


# Helper to load and save local configuration
load_config <- function() {
  if (file.exists(config_file)) {
    return(fromJSON(config_file, simplifyVector = TRUE))
  }
  list()
}

save_config <- function(config) {
  write_json(config, config_file, auto_unbox = TRUE, pretty = TRUE)
}

# Initialize configuration
config <- load_config()

# UI
ui <- fluidPage(
  titlePanel("Messenger with Local Config"),
  sidebarLayout(
    sidebarPanel(
      textInput("username", "Username:"),
      passwordInput("password", "Password:"),
      actionButton("login", "Login/Register"),
      hr(),
      textInput("recipient", "Send to:"),
      textAreaInput("message", "Message:", rows = 3),
      actionButton("send", "Send Message"),
      width = 4
    ),
    mainPanel(
      uiOutput("chatTabs")
    )
  )
)

# Server
server <- function(input, output, session) {
  user_authenticated <- reactiveVal(FALSE)
  current_user <- reactiveVal(NULL)
  all_chats <- reactiveVal(list())
  
  send_request <- function(endpoint, data) {
    response <- POST(
      url = paste0(server_url, endpoint),
      body = toJSON(data, auto_unbox = TRUE),
      encode = "json",
      content_type("application/json")
    )
    if (http_error(response)) {
      list(error = TRUE, message = http_status(response)$message)
    } else {
      fromJSON(content(response, as = "text", encoding = "UTF-8"), simplifyVector = TRUE)
    }
  }
  
  observeEvent(input$login, {
    username <- input$username
    password <- input$password
    
    if (username == "" || password == "") {
      showNotification("Username and password cannot be empty.", type = "error")
      return()
    }
    
    response <- send_request("/authenticate", list(username = username, password = password))
    if (!is.null(response$error)) {
      response <- send_request("/register", list(username = username, password = password))
      if (!is.null(response$error)) {
        showNotification("Registration failed.", type = "error")
        return()
      }
      showNotification("Registered successfully!", type = "message")
    } else {
      showNotification("Logged in successfully!", type = "message")
    }
    
    current_user(username)
    user_authenticated(TRUE)
    all_chats(list())
    
    config <- load_config()
    if (!is.null(config[[username]])) {
      all_chats(config[[username]])
    } else {
      response <- send_request("/fetch", list(username = username))
      if (!is.null(response$error)) {
        showNotification("Error fetching messages.", type = "error")
        return()
      }
      
      messages <- response$messages
      chats <- list()
      for (msg in messages) {
        other_user <- ifelse(msg$sender == username, msg$recipient, msg$sender)
        if (is.null(chats[[other_user]])) {
          chats[[other_user]] <- data.frame(Sender = character(), Message = character(), Timestamp = character())
        }
        chats[[other_user]] <- rbind(
          chats[[other_user]],
          data.frame(Sender = msg$sender, Message = msg$message, Timestamp = msg$timestamp)
        )
      }
      all_chats(chats)
      config[[username]] <- chats
      save_config(config)
    }
  })
  
  observeEvent(input$send, {
    if (!user_authenticated()) {
      showNotification("Please log in first!", type = "error")
      return()
    }
    
    recipient <- input$recipient
    message <- input$message
    
    if (recipient == "" || message == "") {
      showNotification("Recipient and message cannot be empty.", type = "error")
      return()
    }
    
    response <- send_request("/send", list(sender = current_user(), recipient = recipient, message = message))
    if (!is.null(response$error)) {
      showNotification("Error sending message.", type = "error")
      return()
    }
    showNotification("Message sent!", type = "message")
    
    chats <- all_chats()
    if (is.null(chats[[recipient]])) {
      chats[[recipient]] <- data.frame(Sender = character(), Message = character(), Timestamp = character())
    }
    chats[[recipient]] <- rbind(
      chats[[recipient]],
      data.frame(Sender = current_user(), Message = message, Timestamp = as.character(Sys.time()))
    )
    all_chats(chats)
    
    config <- load_config()
    config[[current_user()]] <- chats
    save_config(config)
  })
  
  output$chatTabs <- renderUI({
    chats <- all_chats()
    if (length(chats) == 0) {
      tabsetPanel(tabPanel("Welcome", h4("Log in and start messaging!")))
    } else {
      tabs <- lapply(names(chats), function(user) {
        tabPanel(user, tableOutput(paste0("chat_", user)))
      })
      do.call(tabsetPanel, tabs)
    }
  })
  
  observe({
    chats <- all_chats()
    for (user in names(chats)) {
      local({
        partner <- user
        output[[paste0("chat_", partner)]] <- renderTable({
          chats[[partner]]
        }, rownames = FALSE)
      })
    }
  })
  
  autoInvalidate <- reactiveTimer(3000)
  observe({
    autoInvalidate()
    if (user_authenticated()) {
      response <- send_request("/fetch", list(username = current_user()))
      if (!is.null(response$error)) {
        showNotification("Error fetching messages.", type = "error")
        return()
      }
      
      messages <- response$messages
      chats <- all_chats()
      for (msg in messages) {
        other_user <- ifelse(msg$sender == current_user(), msg$recipient, msg$sender)
        if (is.null(chats[[other_user]])) {
          chats[[other_user]] <- data.frame(Sender = character(), Message = character(), Timestamp = character())
        }
        chats[[other_user]] <- rbind(
          chats[[other_user]],
          data.frame(Sender = msg$sender, Message = msg$message, Timestamp = msg$timestamp)
        )
      }
      all_chats(chats)
      
      config <- load_config()
      config[[current_user()]] <- chats
      save_config(config)
    }
  })
}

shinyApp(ui = ui, server = server)

