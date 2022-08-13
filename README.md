# Understanging Tubro

The aim of this little project is to get a good understanding of how turbo works. My source is [this video tutorial](https://www.youtube.com/watch?v=0CSGsHnci2I) and in this README I'm going to write down my notes. Thanks [Webcrunch](https://www.youtube.com/c/Webcrunch) for the explanation.

## Notes

### Create Action

Notice that we have a div with a "todos" `id` in index.haml.erb. We have this `id` there because this is what we want to update when creating a todo without reloading the page:

```
<div id="todos">
  <% @todos.reverse.each do |todo| %>
    <%= render todo %>
  <% end %>
</div>
```

In the create action of TodosController, we're interepting our response with `format.turbo_stream`. If no arguments are passed, Rails it's going to asume that the response will be views/todos/create.turbo_stream.rb. This means that the `format.html` response is **not** goin to run, unless there's something wrong with the `format.turbo_stream` response.

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

## Other

* Ruby version

 