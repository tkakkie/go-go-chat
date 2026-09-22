# 0006. Channels and topics, with a feed-first view and reply-creates-topic

Date: 2026-09-22. Status: accepted.

## Context

Three products informed this. Chatwork has no notion of a conversation, so
topics interleave and cannot be separated afterwards. Google Chat and Slack
have threads, but a reply is stored under its parent, so someone watching the
main view does not see it. Zulip separates channels from topics and interleaves
topics chronologically, but asks the author to name a topic up front, which
in practice means topics go unused.

The first proposal here was three levels — channel › topic › thread — with
threads promotable to topics. That doubles the UI complexity and the data
model, and Zulip's insight is that the topic *is* the thread.

## Decision

- Two levels: **channel › topic**. A message has a `topic_id` and nothing
  else structural. No threads inside topics.
- The default view of a channel is the **feed**: every topic's messages
  interleaved by time, each labelled with its topic. A reply always appears
  at the bottom of the feed, whatever topic it belongs to.
- When posting, the author may pick an existing topic or name a new one.
  A message posted without a topic goes to the channel's **default topic**.
  Its name is not the same as the default channel's name (both being
  "general" was confusing).
- **Replying to a message in the default topic creates a topic by
  default**: the original message and the reply move into a new topic
  whose name is derived from the original's first words. The author may
  **explicitly** keep the reply in the default topic instead (for a one-word
  acknowledgement); the API expresses this as an explicit field on the
  reply, never as a heuristic, and its name is fixed in the API issue. A
  topic does not have to be named up front; it may emerge from a reply.
  Because the feed shows all topics, nothing disappears from view.
- People with the capability can move messages between topics, rename
  topics, and move a topic to another channel **owned by the same
  organisation**. The destination is authorised like any other write.
- A "recent conversations" list shows active topics across channels.

Invariants:

1. A message's id never changes when it moves.
2. Creating the topic, moving the original and inserting the reply is one
   transaction.
3. A move is a domain event that realtime, unread counts, notifications and
   audit can follow.
4. Two replies to the same default-topic message at the same time converge
   on one topic: the first to commit creates it; a later reply whose
   original has already moved is stored as a reply in that topic, not as a
   second topic and not as an error. This is the operation's own
   concurrency contract and is exercised by a genuinely concurrent test.

## Consequences

- The schema is small: `channel`, `topic`, `message`.
- The reply UI exposes the "keep in default topic" choice; the default is
  to create the topic.
- Later: AI-suggested topic names, and suggested splitting of long default
  topic runs.

## Open questions (realtime issue)

- Message ordering (a per-channel sequence in the style of fukulow's
  `channel_seq`, or something else). Must be decided before the WebSocket
  work starts: it governs reconnect catch-up, history/live races, concurrent
  posts and multi-server ordering.
- The boundary between the database transaction and event delivery: at
  minimum, persist the state change and the event record together and
  deliver after commit; grow into a transactional outbox if needed.
