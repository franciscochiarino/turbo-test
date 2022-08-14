# Understanding Turbo

The aim of this little project is to get a good understanding of how turbo works. My source is [this video tutorial](https://www.youtube.com/watch?v=0CSGsHnci2I) and in this README I'm going to write down my notes. Thanks [Webcrunch](https://www.youtube.com/c/Webcrunch) for the explanation.

## Notes

### Turbo Stream

Notice that we have a div with a "todos" `id` in index.haml.erb. We have this `id` there because this is what we want to update when creating a todo without reloading the page:

```
<div id="todos">
  <% @todos.reverse.each do |todo| %>
    <%= render todo %>
  <% end %>
</div>
```

In the create action of TodosController, we're interpreting our response with `format.turbo_stream`. If no arguments are passed, Rails it's going to assume that the response will be views/todos/create.turbo_stream.rb. This means that the `format.html` response is **not** goin to run, unless there's something wrong with the `format.turbo_stream` response.

Inside this create.turbo_stream.erb we're telling `turbo_stream` to `prepend` something in "todos". This is where the `id` we were talking about in the beginning plays it's part. This is how we tell `turbo_stream` which parts of the page we want to modify.

```
<%= turbo_stream.prepend "todos" do %>
  <%= render "todo", todo: @todo %>
<% end %>
```

Since what we want to `prepend` is a todo, we are going to render the todo partial inside this block. Keep in mind we can write whatever we want in this block. We're just using the todo partial for convenience.

Notice that afterwards we're running another `turbo_stream` method in create.turbo_stream.erb. We can do as much as we want in this file. In this case we're going to replace the current form with a new instance of the form.

```
<%= turbo_stream.replace "#{dom_id(Todo.new)}_form" do %>
  <%= render "form", todo: Todo.new %>
<% end %>
```

### Turbo Frame Tag

Now we want to use `turbo_frame_tag` to update and delete our todos without reloading the page. To make this possible we're gonna have to add an `id` to each todo, so we always know which todo is the one we want to update/delete.

```
<%= turbo_frame_tag dom_id(todo) do %>
  <%= link_to todo.title, edit_todo_path(todo) %>
<% end %>
```

Cool, we have our `edit_todo_path` wrapped in a `turbo_frame_tag`. Now we need to tell Rails what are we going to render in this `turbo_frame_tag` when the user clicks the `edit_todo_path` link.

As we already know, the `edit_todo_path` is going to create a `GET` request that's going to trigger the `edit` action on out TodosController. Since we have nothing inside this action, this is going to render edit.html.erb. This is the file where we want to add our other `turbo_frame_tag`:

```
<%= turbo_frame_tag dom_id(@todo) do %>
  <%= render "form", todo: @todo %>
<% end %>
```

So now when we click the update link, the edit form is going to be rendered inside the `turbo_frame_tag`. Notice that if we update our todo, the `turbo_frame_tag` is going to go back to the updated todo automagically, without us having to specify anything on the update action. It works even though our update action looks like this:

```
if @todo.update(todo_params)
  format.html { redirect_to todo_url(@todo), notice: "Todo was successfully updated." }
else
  format.html { render :edit, status: :unprocessable_entity }
end
```

Personally I don't like this behaviour, because what's happening differs from what's written in the code. If you update a todo and your update action looks like the one above, you'll see that the response under the network tab is the HTML for the show action, even though we're neither rendering the show partial nor redirecting to the show action. So I'm going to change the update action to look like this:

```
if @todo.update(todo_params)
  format.html { render partial: '/todos/todo', locals: { todo: @todo } }
else
  format.html { render :edit, status: :unprocessable_entity }
end
```

Now we're specifying that the partial that's going to be rendered is _todo.html.erb, and we'll also see this reflected in our response under the network tab.

## Other

* Ruby: 2.7.2
* Rails: 7.0.0
