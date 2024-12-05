library(jsonlite)
library(openssl)
library(httpuv)

# File paths
credentials_file <- "server_credentials.json"
message_queue_file <- "server_messages.json"

# Ensure files exist
if (!file.exists(credentials_file)) {
  write_json(list(users = list()), credentials_file, auto_unbox = TRUE, pretty = TRUE)
}
if (!file.exists(message_queue_file)) {
  write_json(list(messages = list()), message_queue_file, auto_unbox = TRUE, pretty = TRUE)
}

# Helper functions to load and save JSON
load_data <- function(file) {
  tryCatch({
    fromJSON(file, simplifyVector = TRUE)
  }, error = function(e) {
    list()
  })
}

save_data <- function(file, data) {
  write_json(data, file, auto_unbox = TRUE, pretty = TRUE)
}

# User management
register_user <- function(username, password) {
  data <- load_data(credentials_file)
  if (username %in% names(data$users)) return(FALSE)
  data$users[[username]] <- digest(password, algo = "sha256")
  save_data(credentials_file, data)
  TRUE
}

authenticate_user <- function(username, password) {
  data <- load_data(credentials_file)
  if (!username %in% names(data$users)) return(FALSE)
  hashed_password <- digest(password, algo = "sha256")
  data$users[[username]] == hashed_password
}

# Message queue management
add_message_to_queue <- function(sender, recipient, message) {
  data <- load_data(message_queue_file)
  if (!is.list(data$messages)) {
    data$messages <- list()
  }
  new_message <- list(
    sender = sender,
    recipient = recipient,
    message = message,
    timestamp = Sys.time()
  )
  data$messages <- append(data$messages, list(new_message))
  save_data(message_queue_file, data)
}


fetch_messages <- function(username) {
  data <- load_data(message_queue_file)
  if (!is.list(data$messages)) {
    data$messages <- list()
  }
  user_messages <- Filter(function(x) is.list(x) && x$recipient == username, data$messages)
  remaining_messages <- Filter(function(x) is.list(x) && x$recipient != username, data$messages)
  data$messages <- remaining_messages
  save_data(message_queue_file, data)
  return(user_messages)
}


# HTTP server
server <- function(request) {
  tryCatch({
    path <- request$PATH_INFO
    method <- request$REQUEST_METHOD
    body <- if (request$REQUEST_METHOD == "POST") fromJSON(request$rook.input$read_lines(), simplifyDataFrame = FALSE) else list()
    
    
    # Registration endpoint
    if (path == "/register" && method == "POST") {
      username <- body$username
      password <- body$password
      if (is.null(username) || username == "" || is.null(password) || password == "") {
        stop("Username and password cannot be empty.")
      }
      if (register_user(username, password)) {
        return(list(status = 200, headers = list("Content-Type" = "application/json"), body = toJSON(list(message = "Registration successful"))))
      } else {
        return(list(status = 400, headers = list("Content-Type" = "application/json"), body = toJSON(list(message = "Username already exists"))))
      }
    }
    
    # Authentication endpoint
    if (path == "/authenticate" && method == "POST") {
      username <- body$username
      password <- body$password
      if (is.null(username) || username == "" || is.null(password) || password == "") {
        stop("Username and password cannot be empty.")
      }
      if (authenticate_user(username, password)) {
        return(list(status = 200, headers = list("Content-Type" = "application/json"), body = toJSON(list(message = "Authentication successful"))))
      } else {
        return(list(status = 401, headers = list("Content-Type" = "application/json"), body = toJSON(list(message = "Invalid username or password"))))
      }
    }
    
    # Send message endpoint
    if (path == "/send" && method == "POST") {
      sender <- body$sender
      recipient <- body$recipient
      message <- body$message
      if (is.null(sender) || sender == "" || is.null(recipient) || recipient == "" || is.null(message) || message == "") {
        stop("Sender, recipient, and message cannot be empty.")
      }
      if (!recipient %in% names(load_data(credentials_file)$users)) {
        return(list(status = 400, headers = list("Content-Type" = "application/json"), body = toJSON(list(message = "Recipient does not exist"))))
      }
      add_message_to_queue(sender, recipient, message)
      return(list(status = 200, headers = list("Content-Type" = "application/json"), body = toJSON(list(message = "Message sent"))))
    }
    
    # Fetch messages endpoint
    if (path == "/fetch" && method == "POST") {
      username <- body$username
      if (is.null(username) || username == "") {
        stop("Username is required.")
      }
      messages <- fetch_messages(username)
      return(list(status = 200, headers = list("Content-Type" = "application/json"), body = toJSON(list(messages = messages))))
    }
    
    # Unknown endpoint
    list(status = 404, headers = list("Content-Type" = "application/json"), body = toJSON(list(message = "Endpoint not found")))
  }, error = function(e) {
    print(paste("Server error:", e$message))
    list(status = 500, headers = list("Content-Type" = "application/json"), body = toJSON(list(message = "Internal Server Error", details = e$message)))
  })
}

cat("Server running at http://localhost:8080\n")
runServer("0.0.0.0", 8080, list(call = server))

