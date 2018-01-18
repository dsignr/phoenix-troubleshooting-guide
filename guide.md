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
