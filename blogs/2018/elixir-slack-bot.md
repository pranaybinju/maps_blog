# Developing a Slack Bot in Elixir Phoenix

1. Slack Authentication.
2. Using Elixir-Slack plugin to establish connection with Slack and linking requests with Trackive users.
3. Creating supervisor process to handle Slack connection independently.
4. Creating Slack bot commands.
5. Auto-restart Slack connection on crash/failure.
6. Disconnecting User after Slack bot removal.

Article
Developing a Slack Bot in Elixir Phoenix

There are many team messaging apps available in the market, but Slack is not
only preferred for its simplicity but also for wide variety of Slack apps. Slack
apps make our lives a lot easier if designed carefully.

We use Trello to create our Story boards and track issues. Trello has excellent Slack app which helps us add and organize our cards and boards very easily.

This helps us manage Trello right from our chat app, No more app/browser tab switching atleast for Trello. There are alot other examples like Google Drive integration which lets us grant permission to view some file to someone else, right in the Slack.

At Kiprosh, we use an in-house task management tool which is written in Elixir + Pheonix. We use this tool heavily to collaborate with our team members. It helps us to work effectively across projects. We decided to write a Slack integration for this tool to collaborate more effectively and save time.

Coming from Rails, it was natural to expect someone else to do the heavy lifting of creating a library for Slack and [Elixir/Slack](https://github.com/BlakeWilliams/Elixir-Slack) was the one we found noteworthy.

I would like to split this post into following sections as it would be more clear and we'll set a direction for this blog post:

1. Slack Authentication.
2. Using Elixir-Slack plugin to establish connection with Slack and linking requests with Trackive users.
3. Creating supervisor process to handle Slack connection independently.
4. Creating Slack bot commands.
5. Disconnecting User after Slack bot removal.

## Slack Authentication

1. To get started with creating Slack app, we will need to signup to slack
   and create a dummy workspace to test our app. This is all very documented by
   Slack on [this page](https://api.slack.com/slack-apps#creating_apps).

   - First create a dummy work space
   - create a slack from [this page](https://api.slack.com/apps?new_app=1)
   - After creating the Slack app, you will get client ID and client secret
     which we will be using for developing the Slack app.

2. Now that we have registered our app on Slack we need to register the User
   wanting to use our app to Slack. This can be done by going to [`Manage Distributions Page`](https://api.slack.com/apps/YOUR_APP_ID/distribute?).
   You should be able to see `Add to Slack` button which we will use to embed
   in our Html page. This will help user to willing to use our app to grant
   access to his workspace. In return users will be able to access this App
   inside their workspace.

3. Once the Slack authorizes user's request for use an App, user is
   redirected back to one of our server URI. This is done by setting
   `Redirect URLs` in "OAuth and Permission" page. Also we can pas this
   redirect url along with Slack authorization URL so the URL is verified by
   Slack and post authorization we are sent to that URL.

4. When user is redirected to our server endpoint, Slack also provides us
   with unique access Token which is used as identifier by Slack to
   recongnize our App. This acess token has appropriate permissions
   associated with it based on the type of 'scope' request that we sent while
   requesting for authorization. So our authorization request looks like
   this
   `https://slack.com/oauth/authorize?scope=bot&client_id=YOUR_CLIENT_ID&redirect_uri=YOUR_SERVER_ENDPOINT&state=abcd`
   In the above URL,
   _scope:_ it tells what type of permission we are requesting from Slack. You
   can read more about this [here](https://api.slack.com/docs/oauth-scopes)

   _client_id:_ this is the id you generated while creating slack app.

   _redirect_uri:_ Post Slack authorization , this is the place Slack will
   point and grant acess_token. This must be the place you save user's access
   token so tha you could access the user workspace everytime you want to
   communicate with the Slack. If you do not save this access token you'll
   need to authorize your request again and will need to generate acess token
   again. Also you need to make sure your access token is stored safely as it
   will be gateway to misuse user's workspace information. We make ours as `http://localhost:4000/api/v1/auth/slack`

   _state:_ this can be some extra information you want to supply for extra
   security per user authorization. Slack says:

   > The state parameter should be used to avoid forgery attacks by passing in
   > a value that's unique to the user you're authenticating and checking it
   > when auth completes.

   All these can be found in depth on this page:
   https://api.slack.com/docs/oauth

## Using Elixir-Slack plugin to establish connection with Slack

1. Now we have access token, we are ready to establish connection with Slack.
   We are going to user Elixir/Slack to establish and manage connection from
   Elixir to Slack and vice versa. You can learn more about it on [Elixir/Slack
   github page](https://github.com/BlakeWilliams/Elixir-Slack).

2. If you check ReadMe.md of Elixir/Slack, you'll see we are required to run
   `Slack.Bot.start_link(SlackRtm, [], "TOKEN_HERE")` to set the token values.
   This is exactly where we left in authorization section. You might be
   wondering what all code sits inside controller of
   `http://localhost:4000/api/v1/auth/slack` , so here we are with the answer.

   Controller code will have:

   1. Check `state` send during authorization request(yes, you guessed right
      even `state` is sent back along with `access_token`)if sent from the
      request while authoriation, else move to next step.
   2. Save the `access_token` granted by slack in the DB (you can encypt this to
      make it extra safe).
   3. Then we use the `access_token` to establish the connection with Slack
      using Elixir/Slack. It uses [Slack RTM API](https://api.slack.com/rtm).

   Also to do this we'll need to set our config for Slack as below:

   config.mxs

   ```elixir
    config :slack,
      client_id: System.get_env("SLACK_CLIENT_ID"),
      client_secret: System.get_env("SLACK_CLIENT_SECRET"),
      root_url: System.get_env("SLACK_ROOT_URL")
   ```

   Our controller looks like:

   slack_auth_controller.mxs

   ```elixir

   def request(conn, %{"provider" => _provider, "code" => code, "state" => _state}) do
      #1 fetch details using Slack.Web.Oauth.html#access
      slack_details = get_slack_details(code)

      #2 if Slack return "ok" status and has no errors save the information in DB
      if !slack_details["error"] && slack_details["ok"] do
        case BotHelper.insert_bot_settings(slack_details) do
          {:ok, changeset} ->
            Logger.info("Registered Slack user #{slack_details["user_id"]}")

          {:error, changeset} ->
            Logger.info("Bot for this user already started")
        end
        #3 redirect user to Slack workspace post auth sucess.
        redirect(conn, external: "https://#{domain}.slack.com/")
      else
      #3 there was some error in authorization of Slack token, redirect user with  401
        Logger.info(":error Slack reponded with : \n#{inspect(slack_details["error"])}")

        conn
        |> put_status(401)
        |> json(%{
          message: "Slack error: #{inspect(slack_details["error"])}"
        })
      end
   end

   def get_slack_details(code) do
    if is_slack_setup?() do
      root_url = Application.get_env(:slack, :root_url)
      client_id = Application.get_env(:slack, :client_id)
      client_secret = Application.get_env(:slack, :client_secret)
      Logger.info("slack redirect url: #{root_url}")
      Slack.Web.Oauth.access(client_id, client_secret, code, %{redirect_uri: root_url})
    end
   end

   defp is_slack_setup?() do
    Application.get_env(:slack, :root_url) && Application.get_env(:slack, :client_id) &&
    Application.get_env(:slack, :client_secret)
   end

   ```

   1. Slack.Web.Oauth.access is used to generate a API token from https://hexdocs.pm/slack/Slack.Web.Oauth.html#access/4
   2. if Slack return "ok" status
      1. Check if user already in DB and Slack connection is established
         1. If no, save all details in the DB and start connection. This is done
            in using BotHelper.insert_bot_settings. Check https://blog.kiprosh.com/managing-slack-workspace-credentials-using-slack-bot-secret-key-and-id/
         2. If yes, inform user
   3. Inform user about error

3. Now that connection is established, we need to test if the our Slack app is
   able to send the messages to workspace and works correctly. To do that, you
   must have a file which handles RTM connection. If you read
   [Elixir-Slack](https://github.com/BlakeWilliams/Elixir-Slack), we have
   `SlackRtm` module which does exactly this. So create `SlackRtm` with
   following code

   ```elixir

    defmodule SlackBot do
      use Slack
      alias SlackCommands
      require Logger

      def handle_connect(slack, state) do
        Logger.info("Connected as #{slack.me.name}")
        {:ok, state}
      end

      def handle_event(message = %{type: "message"}, slack, state) do
        #1 check type of message incoming from Slack
        message = handle_message_subtypes(message)

        #2 check if the request if specific to our App/Bot and prevent responding to self messages
        if is_bot_request?(message, slack) do
          Logger.info("Incoming message for Bot.... \n#{inspect(message)}")
          text = String.replace(message.text, ~r{“|”}, "\"")

          message =
            message
            |> Map.put(:text, text)
          #3 use regex to match input and reply with appropriate response.
          SlackCommands.reply(message, slack)
        end

        {:ok, state}
      end

      def handle_event(_, _, state), do: {:ok, state}

      def handle_info({:message, text, channel}, slack, state) do
        Logger.info("Sending message from handle_nfo")
        send_message(text, channel, slack)
        {:ok, state}
      end

      def handle_info(_, _, state), do: {:ok, state}

      defp handle_message_subtypes(data)do
      # handles text edits
        if data[:subtype] == "message_changed" do
          data
          |> Map.put(:user, data.message.user)
          |> Map.put(:text, data.message.text)
        else
          data
        end
      end

      def is_bot_request?(message, slack) do
        # is message for bot only if the message contains text else it is some other command like deleting a message.
        if message[:text] && message[:user] do
          mentioned_user_id =
            message.text
            |> String.split(["<", "@", ">"], trim: true)
            |> Enum.at(0)

          mentioned_user_id == slack.me.id && slack.me.id != message.user
        else
          false
        end
      end
    end

   ```

- Here I have created a separate module for handling Slack incoming requests as
  `SlackCommands`. It basically pattern matches the input string and supplies
  appropriate response.
- Coming back to code, here file is taken as it is from Elixir/Slack, so we
  wont go into details of each method. We will focus on `handle_event` method.
  Here we are doin following things:

  1. Fist check the type of incoming message. Slack sends all the messages to
     us from any channel in which we invites this Bot, so we would first want
     to filter out the messages to focus on request specific to our App. Also
     when we edit our message in Slack then the request is sent with
     `data[:subtype] == "message_changed"` and text is sent in a different
     format. So we sure that handle_event `message` map is consistent each
     time and we have `user` and `text` keys available in any case. So use
     `handle_message_subtypes` to return `message` map in formatted form. You
     can learn more about message subtypes [here](https://api.slack.com/rtm)

  2. As we dicussed that we need to filter out the requests by checking
     if request has text and user, only then we be sure that incoming
     message is text request arriving from Slack app. We first check if
     incoming request text had our Slack app mention in it. So for text
     `@slack_bot how are you` we extract `@slack_bot` which is sent in the
     format `<@SLACK_BOT_ID>`, so we extract SLACK_BOT_ID which is
     `mentioned_user_id` here and check if id matches our Slack app. Our Slack
     app info in stored in `slack`. So we check
     `mentioned_user_id == slack.me.id`
     Then we check who is originator of the message by matching `message.user`
     which is id of the user with `slack.me.id` which is the app's user id. This
     will ensure we do not send request to our selves in any case.
     `slack.me.id != message.user`

  3. Once we have verified if the request is correct and is intended for our
     app, we proceed with processing the input and send appropriate output. We
     do that in `SlackCommands.reply` this command will be send appropriate response.

## Creating supervisor process to handle Slack connection independently

- Now the most tricky part of this was to make the Slack app crash resistent.
  By crash resistant, I mean it should not halt the whole system when it
  crashes and also it should manage to restart in such case because no one will
  Slack app which doesnt stay online all the time.

- We can do that by creating another Supervisor process under main Supervisor
  tree and that will make this connection independent of our main Supervisor
  process. So we create new Supervisor for each connection we make with the
  workspace.
  Our flow for this looks like:
  `MyApp.Application -> MyApp.Bot -> MyApp.BotSupervisor`

- Our `application.ex` file looks like this:

  ```elixir
  defmodule MyApp.Application do
    use Application

    def start(_type, _args) do
      import Supervisor.Spec
      Envy.load([".env"])
      Envy.reload_config()
      children = [
        supervisor(MyApp.Repo, []),
        supervisor(MyApp.Endpoint, []),
        supervisor(MyApp.Bot, [], id: :slack_bot, name: Slack.Supervisor)
      ]

      opts = [strategy: :one_for_one, name: MyApp.Supervisor]
      Supervisor.start_link(children, opts)
    end

    def config_change(changed, _new, removed) do
      MyApp.Endpoint.config_change(changed, removed)
      :ok
    end
  end
  ```

  bot.ex

  ```elixir
  defmodule MyApp.Bot do
    use Application
    use GenServer
    alias MyApp.{Repo, SlackWorkspaces}
    import Logger
    @delay 90_000
    def start(_, state), do: {:ok, state}

    def start_link do, GenServer.start_link(__MODULE__, MapSet.new())

    def init(state) do
      poll(100)
      {:ok, state}
    end

    def handle_info(:start_bots, state) do, start_bots(state)

    defp start_bots(_state) do
      process_ids = start_all_bots()
      poll()
      {:noreply, process_ids}
    end

    defp poll(delay \\ @delay) do, Process.send_after(self(), :start_bots, delay)


    defp start_all_bots() do
      SlackWorkspaces
      |> Repo.all()
      |> Enum.map(fn bot ->
        %{team_name: bot.team_name, token: bot.bot_access_token}
        atomized_name = String.to_atom(bot.team_name)

        processes =
          if :erlang.whereis(atomized_name) == :undefined do
            Logger.info("Starting Slack bot for following team: #{bot.team_name}")
            MyApp.BotSupervisor.start_link(bot)
          end

        processes
      end)
    end
  end
  ```

  bot_supervisor.ex

  ```elixir
  defmodule MyApp.BotSupervisor do
    import Logger
    use GenServer
    alias MyApp.Repo

    def child_spec(team_info) do
      %{
        id: __MODULE__,
        start: {__MODULE__, :start_link, [team_info]},
        type: :supervisor
      }
    end

    def start_link(team_info) do
      GenServer.start_link(__MODULE__, team_info, name: String.to_atom(team_info.team_name))
    end

    def init(team_info) do
      Slack.Bot.start_link(MyApp.SlackBot, team_info, team_info.bot_access_token)
      |> handle_errors(team_info)
    end

    defp handle_errors({:ok, _} = response, team) do
      # Logger.info("Worker running for #{team.team_name}")
      response
    end

    defp handle_errors({:error, "Slack API returned an error `invalid_auth" <> _ = message}, team),
      do: reset(team, message)

    defp handle_errors(
          {:error, "Slack API returned an error `account_inactive" <> _ = message},
          team
        ),
        do: reset(team, message)

    defp handle_errors(response, team) do
      Logger.warn("UNEXPECTED response from start_link for #{team.team_name}")
      Logger.warn("#{inspect(response)}")
      :ignore
    end

    defp reset(team, message) do
      Logger.warn("Starting team #{team.team_name} failed: #{message}")
      Logger.warn("Deleting this team from DB now...")
      Repo.delete!(team)
      :ignore
    end
  end
  ```



## Delete Slack data on App removal, manually remove Slack data
- Automatic deletion
  - ```elixir
    defp reset(team, message) do
      Logger.warn("Starting team #{team.team_name} failed: #{message}")
      Logger.warn("Deleting this team from DB now...")
      Repo.delete!(team)
      :ignore
    end
    ```
- manual delete
  -
  ```elixir
    def delete(conn, %{"id" => id}) do
      current_user = current_user(conn)
      bot_setting = Repo.get(BotSetting, id)

      if bot_setting do
        case BotSetting.get_bot_setting_permission(bot_setting, current_user) do
          "write" ->
            Logger.warn("Revoking Slack token for team : #{bot_setting.team_name}")
            # send `test: 1` as params to test this
            Logger.warn("#{inspect(Slack.Web.Auth.revoke(%{token: bot_setting.access_token}))}")
            Repo.delete!(bot_setting)
            TaskHelpMeetingHelper.delete_success_message(conn, bot_setting)

          "read" ->
            TaskHelpMeetingHelper.permission_denied_message(conn, bot_setting, "delete")
        end
      else
        TaskHelpMeetingHelper.not_found_message(conn, "BotSetting")
      end
    end
  ```

