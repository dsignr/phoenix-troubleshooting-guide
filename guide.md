# Phoenix Troubleshooting Guide
## Associations
### Nested Associations

**1. Problem:** When you submit a nested (create/update) form, nothing happens. The data isn't inserted, changesets aren't being invoked.

**Solution:** Check your params inside your controller. They should be of the form:
```elixir
%{ "model" => model_params}
```
It is very likely that you're passing `params` directly from your controller to your changeset. For example:

```elixir
# The wrong approach. Everything will be filtered out by your changeset 
# and you won't see any warnings in your logs.
def create(conn, params) do
    case update_city(params) do
    .
    .
    end
end
```

```elixir
# The right approach.
def create(conn, %{"city" => city_params}) do
    case update_city(city_params) do
    .
    .
    end
end
```

**2. Problem:** You get an error that says: ```invalid association :user. When using the :through option, the schema should not be passed as second argument```

**Solution:** This simply means that somewhere in your model, you're doing something like:
```elixir
has_one :user, Accounts.User, through: [:user_address, :user]
```
Instead, it should be:
```elixir
has_one :user, through: [:user_address, :user]
```

**3. Problem:** You get an error that says: ```Protocol Enumerable not implemented for MyApp.MyStruct
``` while using `form_for`.

**Solution:** This means that you're trying to generate a form with an ID attribute while your router doesn't hasn't defined any. Example, in your `router.ex`:
```elixir
get "/account/edit", UserController, :edit
```

But, whereas, inside your form.html.eex, you may be trying to do something like:
```elixir
<%= form_for user_path(@conn, :update, @current_user) %>
```
The problem is, your route doesn't have anywhere to capture the ID from the `@current_user`. So, it's actually expecting a route like this:
```elixir
get "/account/:id/edit", UserController, :edit
```
## Ecto
### Querying

**1. Problem:** You get an error that says: ```cannot use ^some_id outside of match clauses```

**Solution:**  This means you're trying to use Ecto's querying syntax such as `from x in SomeStruct` inside your module without importing Ecto.Query first. Include these lines:

```elixir
import Ecto.Query, warn: false
import Ecto.Changeset
```
## Changesets
### Forms

**1. Problem:** Your changeset errors aren't showing up.  ```form.errors is nil or []. Phoenix.HTML.Form does not take errors from changeset```
**Solution:** 
1. Inspect the form object (usually `f`). Check if `f.errors` has a non-empty array. If it is non-empty, it is very likely something to do with your error helpers. Check them in `views/error_helpers`. If the array is empty, then check the `form_for` tag. You may be passing a `@conn` object instead of `@changeset`.

### Nested forms
When you need to have a form that needs to allow both adding of existing entries from database as well as creating new ones from the same form, use the logic below to avoid creating / overwriting (replace) new entries everytime the parent is updated.

Post -> has_many `tags` through `post_tags`
PostTag -> belongs_to `tags`, also allows casting for tags `cast_assoc(:tags)`

Form -> contains `inputs_for` for `post_tags`
and `post_tags` contains `inputs_for` for `tags`

```
%{
  "content" => "",
  "excerpt" => "awdawdwdawda",
  "post_categories" => %{
    "0" => %{
      "category_id" => "1",
      "name" => "Politics",
      "post_id" => "",
      "slug" => "politics"
    }
  },
  "post_tags" => %{
    "0" => %{
      "post_id" => "", #ignored
      "tag" => %{"id" => "3", "name" => "asdf", "slug" => "asdf"}, #takes precedence
      "tag_id" => "3" #ignored
    }
  },
  "published" => "false",
  "published_at" => %{
    "day" => "17",
    "hour" => "17",
    "minute" => "2",
    "month" => "12",
    "year" => "2022"
  },
  "slug" => "awdwdawad",
  "title" => "wdaddwawdada"
}
```
The above will actually create a new tag and will error out if (and most likely will) a tag with the same ID already exists in the database. The changeset action here will be `insert`.

What you need to do - if you wish to associate an existing `Tag` to the `Post`:

```
%{
  "content" => "",
  "excerpt" => "awdawdwdawda",
  "post_categories" => %{
    "0" => %{
      "category_id" => "1",
      "name" => "Politics",
      "post_id" => "",
      "slug" => "politics"
    }
  },
  "post_tags" => %{
    "0" => %{
      "post_id" => "", #optional
      "tag_id" => "3"
    }
  },
  "published" => "false",
  "published_at" => %{
    "day" => "17",
    "hour" => "17",
    "minute" => "2",
    "month" => "12",
    "year" => "2022"
  },
  "slug" => "awdwdawad",
  "title" => "wdaddwawdada"
}
```
This will ensure you create the join association `PostTag` and not the `Tag` itself.

If you however wish to create a new `Tag` then you would do:
```
%{
  "content" => "",
  "excerpt" => "awdawdwdawda",
  "post_categories" => %{
    "0" => %{
      "category_id" => "1",
      "name" => "Politics",
      "post_id" => "",
      "slug" => "politics"
    }
  },
  "post_tags" => %{
    "0" => %{
      "tag" => %{"name" => "asdf", "slug" => "asdf"},
    }
  },
  "published" => "false",
  "published_at" => %{
    "day" => "17",
    "hour" => "17",
    "minute" => "2",
    "month" => "12",
    "year" => "2022"
  },
  "slug" => "awdwdawad",
  "title" => "wdaddwawdada"
}
```
Make sure you don't cast `id`. That's it.

## Accept arrays in API
There are two methods to do this. 
1. Given a model like below:
```elixir
defmodule Model do
    field :some_array, {:array, :integer}
end
```
Then. you are able to pass params like this:
```
form[]some_array = 1
form[]some_array = 2
form[]some_array = 3
```
This will be seen in the backend as:
```
form[some_array] => [1,2,3]
```
2. The other way is to create a virtual field:
```elixir
defmodule Model do
    field :virtual_field, :string. virtual: true
    field :some_array, {:array, :integer}
end
```
Pass a stringified array in params:
```
form[virtual_field] => "[1,2,3]"
```
Cast the changeset for changes
```
model_changeset
|> cast(...)
|> validate_virtual_field()

defp validate_virtual_field() do
    <check with regex>
    <add to the original some_array field using Enum.map>
    <add_error or do put_change>
end
```

## LiveView
**1. Problem:** Your application page keeps refreshing in production, but not on local. Your socket connections aren't working or are throwing errors.  ```Firefox can’t establish a connection to the server at wss://example.com/live/websocket?_csrf_token=AW8fZ2IsfQE5BCMyHSBBAkEeXDA-RmIs5Zp3RyDOpkAxOyx75QoUm35A&_track_static%5B0%5D=https%3A%2F%2Fexample.com%2Fcss%2Fapp-13404a0906796b323ae87ffa39743021.css%3Fvsn%3Dd&_track_static%5B1%5D=https%3A%2F%2Fexample.com%2Fjs%2Fapp-6483dc03ab95788286683ef9acb37e34.js%3Fvsn%3Dd&_mounts=0&vsn=2.0.0.```
**Solution:** 
This could happen due to many reasons. Here's the plan of attack:
1) Are you running a dockerized setup? If yes, run the docker container locally to see if sockets are able to connect locally. If not, then you will see the exact error that's causing this in the terminal.
2) Are you running on a serverless cloud environment like Google Cloud Run or AppEngine? These may have some limitations w.r.t websockets. Check their documentation. Check if a load balancer is blocking out connections, and disable it if so temporarily to debug.
    2.1) Cloud Run - Disable HTTP/2 support in settings to see if it fixes your issue.
4) Check your config files (`config.exs` and `config/prod.exs`). Particularly, ensure `check_origin` has an array of allowed origins set correctly. Almost always, this is the cause. 
