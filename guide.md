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
