# Personal Website

Made using Hugo.

### Run locally:

```shell
hugo server -D
```
Go to [localhost:1313](http://localhost:1313) to view website.

### Deploy

Generate html files inside `docs` folder as github pages serve either root or docs folder by default.
```shell
hugo -d docs
```

Commit and push it to github. Site is currently being built from the `/docs` folder in the `master` branch
