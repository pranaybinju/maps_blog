# Slack Bot in Elixir

1. Slack Authentication.
2. Using Elixir-Slack plugin to establish connection with Slack and linking requests with Trackive users.
3. Creating supervisor process to handle Slack connection independently.
4. Creating Slack bot commands.
5. Auto-restart Slack connection on crash/failure.
6. Disconnecting User after Slack bot removal.



Article
Slack Bot In Elixir/Phoenix

There are many Team messaging apps available in the market, but Slack is not only preferred for its simplicity but also for wide variety of Slack apps. Slack apps make our lives a lot easier if designed carefully. 
We use Trello to create our Story boards and track issues. Trello has excellent Slack app which helps us add and organize our cards and boards very easily. 

This helps us manage Trello right from out chat app, No more app/browser tab switching atleast for Trello. There are alot other examples like Google Drive integration which lets us grant permission to view some file to someone else, right in the Slack. 

We have our application written in Elixir/Phoenix, and wanted Slack integration for the purpose of adding tasks and viewing them across projects. Our app was for organizing our day by add accomplishments, help needed and meetings. We could also mention anyone in the our organisation.
Coming from Rails, it was natural to expect someone else to do the heavy lifting of creating a library for Slack and [Elixir/Slack](https://github.com/BlakeWilliams/Elixir-Slack) was the one we found noteworthy.

