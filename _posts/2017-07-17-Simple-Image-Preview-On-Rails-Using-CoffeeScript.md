---
layout:   post
title:    "Simple Image Preview On Rails Using CoffeeScript"
date:     2017-07-17 14:46:00 +0900
---

### new.html.erb

```erb
  <div class="field">
    <div id="image_preview"></div>
  </div>

  <div class="field">
    <%= f.label :image %>
    <%= f.file_field :image, id: :image_upload, accept:"image/*" %>
  </div>
```

### preview.coffee

In this case, used with **turbolinks**. If you don't use **turbolinks**, use `DOMContentLoaded` instead of `turbolinks:load`.

```coffee
document.addEventListener "turbolinks:load", (e) -> 
  input = document.querySelector("#image_upload")
  if input
    input.addEventListener "change", (e) ->
      set_image_to_preview(this)

    set_image_to_preview = (input) ->
      if (input.files && input.files[0])
        reader = new FileReader()
        reader.addEventListener "load", (e) ->
          preview = document.querySelector("#image_preview")
          preview.style.backgroundImage = "url('#{e.target.result}')"
        reader.readAsDataURL(input.files[0]);
```
