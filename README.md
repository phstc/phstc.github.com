Repository for my [blog](https://pablocantero.com).

## Notes to self

### Local server

For running it locally, you will need to install Jekyll dependencies:

```shell
gem install jekyll
gem install jekyll-redirect-from
gem install jekyll-paginate
```

Then

```shell
rake server
```

### New post

For creating new draft:

```shell
rake draft:new
What's the name for your next post?
```
When the post is ready:

```shell
rake draft:ready
What's the name for your next post?
Posts in _drafts:
1 - post name 1
2 - post name 2
What's the number of the draft to post?
```

## All options

```shell
rake -T

rake draft:new    # Creating a new draft for post/entry
rake draft:ready  # Copy draft to production post
rake server       # Start jekyll in development mode
```
