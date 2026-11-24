# Literess MCP server

[![Literess MCP connector](https://glama.ai/mcp/connectors/com.literess/literess/badges/score.svg)](https://glama.ai/mcp/connectors/com.literess/literess)

[Literess](https://literess.com) is a writing companion for long manuscripts: she reads the whole book, comments in the margin and suggests edits the writer accepts or rejects. This remote MCP server lets Claude, ChatGPT, Cursor or any MCP client work in a writer's Literess workspace: read documents, leave comments, suggest tracked edits, keep todos and talk with Literess in the chat.

- **Endpoint:** `https://write.literess.com/mcp` (Streamable HTTP)
- **Sign-in:** OAuth 2.1 with dynamic client registration. The first connection opens a browser window where the writer signs in and approves access. `initialize` and `tools/list` answer without sign-in, so you can see the tools first.
- **Registry:** `com.literess/literess` in the [official MCP Registry](https://registry.modelcontextprotocol.io)
- **Price:** the connector comes with every Literess account; there's a free plan with no card.

This repository holds the registry entry (`server.json`) and this documentation. The server itself is hosted by Literess; there is nothing to install.

## Connect

**Claude (web and desktop):** Settings → Connectors → Add custom connector, and paste:

```
https://write.literess.com/mcp
```

**ChatGPT:** Settings → Connectors (developer mode) → Create, with the same URL.

**Claude Code:**

```bash
claude mcp add --transport http literess https://write.literess.com/mcp
```

**Cursor, VS Code and other clients that read `mcp.json`:**

```json
{
  "mcpServers": {
    "literess": { "url": "https://write.literess.com/mcp" }
  }
}
```

## What an agent can and can't do

- Edits arrive as **tracked suggestions** by default: the writer accepts or rejects each one in the editor.
- Projects locked with a PIN stay closed until the writer gives the agent the PIN.
- Every connection introduces itself first (`introduce`), so the writer sees which agent wrote what.
- Call `guide` for how to read, edit and follow the writer's work.

## Tools

- `introduce`: Call this FIRST, on every connection: every other tool refuses until you have.
- `list_documents`: List documents with titles, summaries, word counts and the folder each is kept in (folderId; list_folders names them).
- `search_documents`: Semantic search across documents using natural language.
- `get_document`: Fetch the content of a document as markdown.
- `query_document`: Ask a natural language question about a specific document.
- `create_document`: Create a new document from markdown given as content — or from a file you uploaded with upload_file, as from_file: a .md (markdown), .txt (plain text, kept literally), or a manuscript (.docx, .epub as one document with its chapters in order, .odt) converted into Literess's own formatting: headings, emphasis, links, nested lists, quotes, tables, pictures, footnotes as notes (a long text never has to pass through your reply).
- `update_document`: Replace a document's entire content and/or title.
- `edit_document`: Make a targeted edit without resending the whole document.
- `insert_blocks`: Insert new content into the document.
- `replace_blocks`: Replace a contiguous range of blocks with new markdown.
- `delete_blocks`: Delete a block, or a contiguous range of blocks, as a tracked suggestion.
- `move_block`: Move a block to a different location in the document.
- `indent_blocks`: Change the nesting depth of a block — a list item OR a paragraph/todo/code block/callout/comment note.
- `batch_edits`: Several block edits in one go, all or nothing: every step is checked first and one refused step writes nothing.
- `list_chats`: List chat sessions with titles, summaries, and linked documents.
- `search_chats`: Semantic search across chat conversations.
- `query_chat`: Ask a natural language question about a specific chat conversation.
- `get_chat_messages`: Fetch the message history of a chat session.
- `list_projects`: List projects with document counts, descriptions and colors, most recently active first.
- `get_project`: Get project details including instructions.
- `unlock_project`: Unlock a PIN-protected project for the rest of this connection.
- `list_folders`: How a project is organized: its folders (nested, outermost first, with each one's own instructions) and the documents kept in each and at the project's top.
- `create_folder`: Make a folder in a project — at its top, or inside another folder of the same project (parent_folder_id).
- `update_folder`: Rename a folder, set its own instructions (the whole text; "" clears them), paint it a colour (#rrggbb; "" unpainted), or move it into another folder of the same project ("" for the project's top).
- `delete_folder`: Delete a folder.
- `move_document_to_folder`: Keep a document in a folder ("" for its project's top).
- `get_story_map`: The story map of a fiction-mode project: the canon reading order with statuses, takes, pots and cold-read counts.
- `get_story_world_state`: Effective world state of a fiction-mode project at a scene (default: end of the canon chain): every established fact, beliefs kept beside truth.
- `guide`: How to read, edit and follow the user's work in Literess over MCP.
- `get_workspace`: What the user has on screen in Literess right now: the pane in focus (usually a document) and the other open panes.
- `get_activity`: What the user has been doing in Literess (edits, comments, suggestions accepted or rejected, going idle, moving around the story map), grouped by pane.
- `get_observations`: What Literess's watcher made of the user's activity: its recent verdicts (worth a look or not, and why) and the situation it keeps across them.
- `list_comments`: The comment threads on a document: each comment's id, the block it was left on (the ids get_document returns with include_ids:true), the words it was left on (`quote`), every block its highlight reaches now (`covers`), its text, author, replies, who it is addressed to (`to`), and whose move it is (`waitsOn`: "agent" when the user had the last word).
- `add_comment`: Leave a comment on one block of a document (the user sees it in the margin and in the Review panel, credited to you).
- `mark_pending`: Show the user where you are about to write before the text exists: a "⏳ Drafting: <intent>" note on that block, in their margin.
- `reply_to_comment`: Reply in an open comment thread (ids from list_comments).
- `react_to_comment`: React to a comment thread with one emoji (ids from list_comments), or to one reply in it (reply_id) — shown on it, credited to you.
- `resolve_comment`: Resolve an open comment thread once it is dealt with (ids from list_comments).
- `reject_suggestion`: Withdraw one of YOUR pending suggestions — ids from an edit's suggestionIds or get_document's yourPendingSuggestions.
- `watch_document`: Follow a document: get_watched_changes then reports what changed in it since you last looked.
- `unwatch_document`: Stop following a document.
- `get_watched_changes`: What changed in each watched document since you last looked (the new text quoted, who wrote last), then counts from now.
- `list_versions`: A document's saved versions, newest first: versionId, rev (the hash get_document reports), who wrote it, when, size, the user's name for it.
- `restore_version`: Put a saved version back as the document's content (by rev or versionId from list_versions).
- `recall`: Read what Literess remembers: about herself ("self"), a person (the user, you, another agent) or a project.
- `message_literess`: Say something to Literess, the in-app writing assistant, in the user's workspace chat.
- `say_in_chat`: Answer the user in the workspace chat — e.g.
- `react_to_message`: React to a chat message with one emoji (message ids from get_chat_messages or waiting_for_me) — shown under it, credited to you.
- `pass`: Seen it, nothing to add — like Literess ending her turn without a word.
- `waiting_for_me`: Everything waiting on your answer now: comment threads (in any document) addressed to you ("@YourName …" or `to`) by the user or Literess, or answering a comment you left; @mentions of you in the workspace chat, by the user or Literess; and open todos handed to you.
- `wait_for_me`: Wait until something NEW waits on you (an @mention in the chat, a comment thread addressed to you, an answer to your comment, a todo handed to you) and return it — or return empty when the time is up; then call it again.
- `stay_reachable`: How to stay reachable without holding a call open, here and while your session is closed.
- `upload_file`: Bring a file into Literess: a picture (png, jpg, gif, webp), a recording (mp3, wav, m4a, ogg, flac, aac), a video (mp4, webm, mov), a text (.md, .txt) or a manuscript (.docx, .epub, .odt).
- `list_todos`: The todo list the user, Literess and agents share: open todos by default (or done, or all), in every project you may see — or only the ones for you (mine:true).
- `create_todo`: Add a todo: one thing left to do in the writing — for the user, Literess, you, or another agent — pointing at what it is about.
- `update_todo`: Change a todo: its title, notes, what it points at, or who it is for (hand it over — `note` says why).
- `complete_todo`: Mark a todo done, with what was done in one line — the user reads it, and Literess may look at the work.
- `reopen_todo`: Open a done todo again, with why in one line (the work isn't there).

## Links

- Website: https://literess.com
- How to connect an assistant: https://literess.com/bring-your-own-assistant/
- Support: support@literess.com
- Privacy: https://literess.com/privacy.html
