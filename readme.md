# Silly Chat

### Quick Start

Prerequisites:

- Python3.10 or newer: https://www.python.org/downloads/
- Poetry: https://python-poetry.org/

- `cd silly-chat`
- `poetry env use python3.10`
- `poetry install`
- `poetry shell`
- `poetry run serve` to run the service with hot reload
- Open http://localhost:8000/docs to see the API
- `poetry run test` to run the tests

### Milestone 0

Basic multi-user chat:

- `User1` creates a chat URL
- At that point, `User1` can post messages into the chat and read the chat
- `User1` creates a link URL and shares it with `User2` (e.g. by email)
- `User2` exchanges the link for the chat URL
- At that point, both `User1` and `User2` can post messages into the chat
- Both `User1` and `User2` can read the chat and they see all the messages

Assumptions and simplifications:

- The chat URL contains an id that is a large opaque random string
- There’s no authentication or authorization, other than the chat id
- Once the message is posted, it cannot be deleted
- Messages are kept in memory
- We’ll assume that chats don’t grow too large
- A chat message is just text, there's no formal author field
- Chat clients can freely decide on the format of the messages

Endpoints:

- `POST /v1/chats` creates a new chat and returns its URL
- `POST /v1/chats/<opaque>` creates a new message in the chat
- `GET /v1/chats/<opaque>` returns all messages in the chat
- `POST /v1/links` with chat URL, create and returns a link URL for this chat
- `GET /v1/links/<opaque>` returns the chat URL for this link URL

### Milestone 1

Chat join approval system:

- `User1` (owner) already created a chat, a link and passed the link to `User2` (guest)
- Only the owner can post and read messages in this chat at this point
- When the guest tries to exchange their link URL for the chat URL:
  - the owner receives a real-time notification of the request
  - the owner may approve the join request, then
  - the guest receives a real-time notification of approval
  - guest link exchange now succeeds
- Both the owner and the guest can post and read messages in this chat now

Assumptions and simplifications:

- Assume that both users are online and their UIs are open
- The owner can’t decline the guest, but they may leave the request pending forever
- If the request is left pending, we’ll trust that the guest gives up eventually

Endpoints:

- `GET /v1/approval/` Client/User2 can check Status of Approval Status by supply link URL, `Approval_Status` is return (`bool`)
- `POST /v1/approval/` Chat creator change link URL to approved (`True`), this depend on if User2 had request for approval, approval status is update after change
  
-- Modificatio to Existing Endpoint in order to accomodate some additional feature, such as user ID (based on UUID4)
- `POST /v1/chats` creates a new chat and returns its URL and author_id for the chat creator
- `POST /v1/chats/<opaque>` creates a new message in the chat. The API endpoint require User ID, both chat creator and guest will submit this information. For user2/guest to post, approval system is check
- `GET /v1/links/<opaque>` A guest hitting this ENDPOINT trigger approval process by sending request to chat creator/author, The ENDPOINT returns link URL(Pending Approval Status), `Request_sent_Status`, `Approval_Status`, `chat`, `client_id`
