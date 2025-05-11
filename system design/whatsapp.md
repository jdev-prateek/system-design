User can have one or more contacts. So there is one to many relationship between user and contacts.

## contacts table

```sql
CREATE TABLE contacts (
  user_id UUID,
  contact_id UUID,
  contact_name TEXT,
  contact_number TEXT,
  PRIMARY KEY ((user_id), contact_id)
);
```

Partition key: `user_id` → so all of a user's contacts go to one shard.

## chats table (one to one)

```sql
CREATE TABLE chats (
  chat_id UUID,
  message_id timeuuid,
  message_text TEXT,
  sender_id TEXT,
  is_delivered boolean,
  is_seen boolean,
  created_at timestamp,
  updated_at timestamp,
  PRIMARY KEY ((chat_id), message_id)
);
```

chat_id = hash(sort(user1_id, user2_id))

## 2. groups table 

```sql
CREATE TABLE groups (
  group_id TIMEUUID,  
  group_name UUID,  
  created_at timestamp,
  updated_at timestamp,
  PRIMARY KEY (group_id, created_at)
);
```

## 2. groups_members table

```sql
CREATE TABLE group_members (
  group_id UUID,
  user_id UUID,
  role TEXT,        -- e.g., 'admin', 'member'
  joined_at TIMESTAMP,
  PRIMARY KEY ((group_id), joined_id desc)
);
```


## user_conversations table

```sql
CREATE TABLE user_inbox (
  user_id UUID,
  chat_id UUID,
  last_message_id TIMEUUID,
  last_message_text TEXT,
  last_message_time TIMESTAMP,
  unread_count INT,
  PRIMARY KEY (chat_id, user_id)
);
```

How to use websockets with kafka to scale:

https://chatgpt.com/c/681f06a2-2c20-8008-a2ee-b9c6bb389e68


## Use flow demo - one to one messages

1. user_1 is offline
1. user_1 comes online and whatsapp establishes a web socket connection.
1. there will be multiple instances of web server and user_1 can connect to any one.
1. once websocket connection is established. This information is saved to the redis like this:
    user_1 -> server_1
1. this information is used to forward incoming messages to user_1
1. user_1 wants to send message to user_2.
1. user_1 types the message and hits the enter key
1. calculate `chat_id = hash(sort(user_1, user_2))`
1. the `chat_id` decides in which db partition this chat will be stored.
1. a new row is added to db: 
    ```sql
        insert into chats (chat_id, message_id , message_text, sender_id, created_at, updated_at)
        values (hash(sort(user_1, user_2)), '12121', 'hello', 'sender_id', now(), now())
    ```
1. next, we also save it to the `user_inbox`. This table is responsible to displaying recent messages on the receiving side.
    ```sql
    insert into user_inbox ( user_id, chat_id, last_message_id, last_message_text, last_message_time,unread_count)
    values (`user_2`, 'some_chat_id', '1212', 'hello', now(), 1)
    ```

1. once message is saved to db, we push a message containing (chat_id, user_id) to kafka consumer groups.
1. one of the consumer then picks up the message, queries the database and fetches the row(s). Next, we perform redis lookup, to find the server user_2 is connected to. Assuming, mapping exists, the consumer send the message to user_2. Once the message is sent, we mark the message as delivered.

1. Now assuming, redis lookup failed or no such mapping exists. We don't mark mark the message delivered.
    1. Lets say after some time user_2 comes online and establishes a websocket connection to one of the server. This places a new key-value pair (`user_2 -> server_2`) in the redis. Next, we query `user_inbox` table and sends the pending messages to user.
