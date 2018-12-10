# Slack Bot in Elixir

1. Slack Authentication.
2. Using Elixir-Slack plugin to establish connection with Slack and linking requests with Trackive users.
3. Creating supervisor process to handle Slack connection independently.
4. Creating Slack bot commands.
5. Auto-restart Slack connection on crash/failure.
6. Disconnecting User after Slack bot removal.

Article
Slack Bot In Elixir/Phoenix

There are many Team messaging apps available in the market, but Slack is not
only preferred for its simplicity but also for wide variety of Slack apps. Slack
apps make our lives a lot easier if designed carefully.

We use Trello to create our Story boards and track issues. Trello has excellent Slack app which helps us add and organize our cards and boards very easily.

This helps us manage Trello right from out chat app, No more app/browser tab switching atleast for Trello. There are alot other examples like Google Drive integration which lets us grant permission to view some file to someone else, right in the Slack.

We have our application written in Elixir/Phoenix, and wanted Slack integration for the purpose of adding tasks and viewing them across projects. Our app was for organizing our day by add accomplishments, help needed and meetings. We could also mention anyone in the our organisation.
Coming from Rails, it was natural to expect someone else to do the heavy lifting of creating a library for Slack and [Elixir/Slack](https://github.com/BlakeWilliams/Elixir-Slack) was the one we found noteworthy.

I would like to split this post into following sections as it would be more clear and we'll set a direction for this blog post:

1. Slack Authentication.
2. Using Elixir-Slack plugin to establish connection with Slack and linking requests with Trackive users.
3. Creating supervisor process to handle Slack connection independently.
4. Creating Slack bot commands.
5. Auto-restart Slack connection on crash/failure.6. Disconnecting User after Slack bot removal.

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
   client_id: System.get_env("SLACK_CLIENT_ID") || "242215032151.295905216179",
   client_secret: System.get_env("SLACK_CLIENT_SECRET") || "12ca1f9c1e690455dcfc35542cdb8858",
   root_url:
   System.get_env("SLACK_ROOT_URL") || "https://server.trackive-staging.com/api/v1/auth/slack"
   ```

   Our controller looks like:

   slack_auth_controller.mxs

   ```elixir

   def request(conn, %{"provider" => _provider, "code" => code, "state" => _state}) do
      #1 fetch detials
      slack_details = get_slack_details(code)
      if !slack_details["error"] && slack_details["ok"] do
        #fetches workspace domain from slack
        domain =
          BotHelper.slack_team_info(slack_details["bot"]["bot_access_token"])["team"]["domain"]

        #stores slack access token and other information like domain, workspace name etc derived from slack into DB
        case BotHelper.insert_bot_settings(slack_details) do
          {:ok, changeset} ->
            Logger.info("Registered Slack user #{slack_details["user_id"]}")

          {:error, changeset} ->
            Logger.info(":error Changeset: \n#{inspect(changeset)}")
            Logger.info("Bot for this user already started")
        end

        redirect(conn, external: "https://#{domain}.slack.com/")
      else
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
