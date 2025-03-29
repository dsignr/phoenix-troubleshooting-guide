# LiveView Troubleshooting Guide
## Non-streams

### Inserts
1. If your element insertion isn't working correctly, check if you've mistakenly added `phx-update="stream"` in the parent.
2. Check if you have a unique ID on each iterated element.


## Streams

## Stream Insert
1. If your streams are being replaced, but not being appended/inserted, then check if the stream array you use has an ID for each element.
This is particularly to be noted when manually creating entities to be inserted into a stream:

```elixir
task = %My.Task{
      id: Ecto.UUID.generate(), #Notice this line
      name: "some_name",
      description: "some_description",
      status: "pending",
    }
{
      :ok,
      socket
      |> stream_insert(:tasks, task)
}
```

2. Ensure you've added `phx-update="stream` to the parent div/container.
 ```
 <div id="tasks-stream" phx-update="stream" class="flex flex-1 flex-col gap-y-2 h-full min-h-0 overflow-y-scroll no-scrollbar px-2">
  <div :for={{id, task} <- @streams.tasks} id={id} class="flex gap-2 w-full">
    <.task_info_card task={task} />
  </div>
</div>
```
   
